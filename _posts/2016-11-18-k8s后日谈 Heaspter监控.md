![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-kube7-logo.png?imageView2/2/h/200) 

那么对于资源的监控,比较流行的选型有两种: 

- Heaspter + Influxdb + Grafana
- Prometheus + Alarmmanager + Grafana

两者当然都要尝试一下才进行选型确定,那么本篇是前者的试验. 

> 2017-03更新,因为Heapster的羸弱,所以只安装Heapster来充实面板. 
> 
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

那么完全版是下图这个样子: 

![](https://o4dyfn0ef.qnssl.com/image/2016-12-01-CA851A29-F5F5-463D-982B-AFB97B5EAB9D.png?imageView2/2/h/400) 

- - - - -- 

### minimal配置  

当然了,随着2017-03我的幡然醒悟,我们只是安装Heapster来充实一下面板. 

```shell
kubectl create -f heaspter-deployment.yaml
kubectl create -f heaspter-service.yaml
``` 

同时删除掉deployment里`--sink`相关的内容. 

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

- - - - --- 

done. 