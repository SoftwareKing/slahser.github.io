![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-kube7-logo.png?imageView2/2/h/200) 

那么对于资源的监控,比较流行的选型有两种: 

- Heaspter + Influxdb + Grafana
- Prometheus 

两者当然都要尝试一下才进行选型确定,那么本篇是前者的试验. 

> 看了下release note,Grafana 4已经支持告警了. 

## Heapster  

![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-Screen%20Shot%202016-11-15%20at%2016.40.42.png?imageView2/2/h/400) 

```shell
git clone https://github.com/kubernetes/heapster.git
cd heapster
# 修改yaml,添加registry
kubectl create -f deploy/kube-config/influxdb/
``` 

实际看到文件夹内含6个 yaml,deploy 与 svc 各三份. 

- heapster - agent 与 k8s add-on 的作用
- influxdb - db
- grafana - ui

只是想达到上图效果很简单了,只要部署一个 heapster-svc 上来就够了,但是 metrics 数据也可以被记录到 influxdb 中配合 grafana 进行展现. 

```shell
kubectl create -f heaspter-deployment.yaml
kubectl create -f heaspter-service.yaml
``` 

那么完全版是下图这个样子: 

![](https://o4dyfn0ef.qnssl.com/image/2016-12-01-CA851A29-F5F5-463D-982B-AFB97B5EAB9D.png?imageView2/2/h/400) 

爽到. 

- - - - -- 

### minimal配置  

```
# 查看 nodeport
kubectl describe svc monitoring-grafana --namespace=kube-system
```

1. 访问 masterip:port 进入 grafana 管理界面,左侧 admin - admin 登陆 
2. 剩下的什么新建 influxdb 的步骤都不需要了,默认自带一个`http://monitoring-influxdb:8086`
3. 这个数据源是否靠谱还不确定,因为不知道你是否已经是 dns 生效过,那么 test connection 进行测试. 
4. 实际上一步换成你的 influsdb-svc 的 ip 也 ok. 

![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-Screen%20Shot%202016-11-15%20at%2017.29.54.png?imageView2/2/h/400) 

- - - - -- 

### influxdb 配置 

> 经过更新,这个章节已经没用了...被去掉了. 

同时如果你对 influxdb 的数据结构感兴趣的话,这里有一个 ui 界面供使用. 

修改[这里](https://github.com/kubernetes/heapster/tree/master/deploy/kube-config/influxdb),修改`influxdb-service.yaml` 

解开以下部分的注释: 

```
spec:
  # InfluxDB has a UI that can be used to query, uncomment the `http` port and expose
  # to access this service.
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 8083
  - name: api
    port: 8086
    targetPort: 8086
  selector:
    k8s-app: influxdb
```

而后重新 create deploy 与 service. 

那么describe 一下 monitoring-influxdb 得到下图的 nodeport

![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-Screen%20Shot%202016-11-15%20at%2020.10.52.png?imageView2/2/h/400) 

- 80 - 3xxxx对应 ui - masterip:3xxxx 用于访问 ui
- 8086 - 3xxxx 对应数据访问 - internal ip用于访问数据

![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-Screen%20Shot%202016-11-15%20at%2020.11.03.png?imageView2/2/h/400) 

可以通过这个查询界面来看数据结构与数据是否正确落入. 

- - - - -- 

### 图表配置 

[这里](https://github.com/kubernetes/heapster/blob/master/docs/storage-schema.md)是 heaspter 的 metrics 列表. 

grafana dashboard 配置纷繁复杂,我这一期肯定不会在这上面耽误太长时间,应该只是浅尝辄止就是了. 

根据上面列表的具体示意以及 grafana 配置查询语句时的联想功能 

基本上简单的数据图/表应该也能顺藤摸瓜搞个出来. 


- - - - --- 

done. 