![](https://o4dyfn0ef.qnssl.com/image/2017-03-26-Screen%20Shot%202017-03-27%20at%2000.02.49.png?imageView2/2/h/300) 

题图为cncf四大金刚吧算是,先后捐献出来的. 

很直接的把如下微服务中关心的部分都给解决了.  

- Tracing
- Logging
- Metrics

这边有一篇[Metrics, tracing, and logging](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html). 

![](https://o4dyfn0ef.qnssl.com/image/2017-03-26-03.png?imageView2/2/h/400) 

今儿来解决Metrics的事儿. 

> 本文写于2017-03-31,有些东西可能会随着时间变化. 

- - - - -- 

提纲: 

- prometheus in kubernetes
- monitoring kubernetes 
- monitoring mysql
- etc

- - - - --- 

### 安装 

> 显然prometheus不同版本之间配置文件是不兼容的. 

[这里](https://github.com/giantswarm/kubernetes-prometheus)有这么一个repo. 

在[manifests-all.yaml](https://github.com/giantswarm/kubernetes-prometheus/blob/master/manifests-all.yaml)里能清楚的看到它做了些什么. 

- 三份Deploy - 部署核心的东西
- 两份DaemonSets - 部署agent
- 两份ConfigMap - 配置导入
- 一个Job - dashboard导入
- 四个Service - 聚拢一下服务

当然有些需要调整: 

- image调节成私服
- requests与limits调节,量力而行. 
- 提前调节`prometheus.yml`,比如我提前添加了mysql的node与mysqld exporter进去. 
- 调节`storage.local.retention`到12h
- 调节grafanayu prometheus的svc到固定的NodePort.方便负载. 
- 调节VolumeMount到PVC(可选),示例里是存到hostPath上的.

另外一点就是这个文件需要上传到node去`kubectl create -f`,从dashboard上执行不认`----`. 

> 而且惊了,tiny-tools里居然有fish! 

再就是调节Nodeport这点. 

我看作者有计划做成ingress,不过这边算是内部服务,我觉得随便反代一下好了. 

- - - - -- 

### 使用nginx反代各位 

很久以前我们的dashboard就是这么暴露出来供访问的. 

在一台同网段机器上,docker运行nginx 

目录结构:   

```
docker-compose.yaml
nginx
    dashboard.conf //k8s dashboard
    grafana.conf //grafana面板
    prometheus.conf //prometheus查询面板
    passwd
    dashboard.htpasswd //给dashboard加一个http basic auth
```

内部配置: 

> 酌情修改.比如去掉两行auth就是给grafana等人的配置. 

```
upstream k8s-dashboard {
             #ip_hash;
             server [node1-ip]:[our-port];
             server [node2-ip]:[our-port];
             server [node3-ip]:[our-port];
         }

server {
        listen       80;
        server_name  k8s.dashboard.me;
        location / {
            auth_basic "Restricted";
            auth_basic_user_file /etc/nginx/conf.d/dashboard.htpasswd;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_buffering off;
            proxy_pass http://k8s-dashboard;
        }
}
```

修改过后,`docker exec -it nginx /bin/sh`执行`nginx -s reload`即可. 

- - - - -- 

### Grafana初见配置 

- 叉掉引导,手动添加用户.
- 调节默认k8s面板的一些跨度(标题->edit-> time range->12h).

### 安装exporters 

最基本的我们知道prometheus是把exporter当做agent来收集数据,并且是pull形式的. 

官方有[这些](https://prometheus.io/docs/instrumenting/exporters/)exporters. 

起码目前感兴趣的,可以短平快的接入的: 

- [prometheus/node_exporter](https://github.com/prometheus/node_exporter) - 监控机器基本状况. 
- [prometheus/mysqld_exporter](https://github.com/prometheus/mysqld_exporter)
- [oliver006/redis_exporter](https://github.com/oliver006/redis_exporter)
- [dcu/mongodb_exporter](https://github.com/dcu/mongodb_exporter)

说句大实话就是各自看文档就能安装好. 

拿mysql为例子,用`nohup ./*-exporter &`装上前两个 

暴露9100与9104端口来提供metrics数据. 

> 另外一点,主从的mysql,从库上的用户直接等主库同步过来就好了

同时如果你没做前文的修改prometheus.yml或者是安装新东西. 

那么你需要到k8s node上,然后删除prometheus pod来更新配置. 

`kubectl edit configmap/prometheus-core -n monitoring` 

vim手法添加如下内容:  

> 下面这个写法不保证简洁,但是用起来肯定没问题. 

``` 
- job_name: mysql-master-node
static_configs:
  - targets: ['33.73.192.14:9100']
    labels:
      instance: mysql-master

- job_name: mysql-master
static_configs:
  - targets: ['33.73.192.14:9104']
    labels:
      instance: mysql-master

- job_name: mysql-slave-node
static_configs:
  - targets: ['33.73.192.15:9100']
    labels:
      instance: mysql-slave

- job_name: mysql-slave
static_configs:
  - targets: ['33.73.192.15:9104']
    labels:
      instance: mysql-slave
```

- - - - -- 

### 添加面板 

Grafana里面目前还没有mysql相关的面板. 

那么我们的老朋友Percona有[这么一个](https://github.com/percona/grafana-dashboards)库,给了很多好面板. 

下载下来 

按道理来讲全部导入都没问题,但是数据源需要修改. 

我们的数据源是`prometheus`,repo里的数据源是`Prometheus`. 

另外挑几个重要的导入就好了. 

我暂时挑选了: 

- Cross_Server_Graphs
- Disk_Performance
- Disk_Space
- MySQL_Overview
- MySQL_Replication
- Prometheus
- Summary_Dashboard
- System_Overview
- Trends_Dashboard

修改数据源名称并在Grafana中导入,看情况修改下time range.  

- - - - -- 

日后肯定要做的: 

- alert
- 更多监控export
- 数据存储到pvc

另外这两个Grafana App怎么这么眼熟: 

- [percona-percona-app](https://grafana.com/plugins/percona-percona-app)
- [raintank-kubernetes-app](https://grafana.com/plugins/raintank-kubernetes-app) 

- - - - -- 

### 部分截图 

尴尬的是数据量大起来这个pod面板特别卡...下次重装时候一定要给大点. 

反正大家是按requests给资源的,不会影响其他分布. 

> 生产环境需要继续join机器了我觉得. 

![](https://o4dyfn0ef.qnssl.com/image/2017-03-31-Screen%20Shot%202017-03-30%20at%2023.15.27.png?imageView2/2/h/400) 

mysql主从同步状态,可以看到一次一次latency的毛尖 

![](https://o4dyfn0ef.qnssl.com/image/2017-04-01-Screen%20Shot%202017-04-01%20at%2014.08.50.png?imageView2/2/h/400)


mysql-slave的机器可以看到4点I/O陡增,因为当时在自动备份. 

![](https://o4dyfn0ef.qnssl.com/image/2017-04-01-Screen%20Shot%202017-04-01%20at%2014.10.32.png?imageView2/2/h/400)

![](https://o4dyfn0ef.qnssl.com/image/2017-04-01-Screen%20Shot%202017-04-01%20at%2014.10.57.png?imageView2/2/h/400)

![](https://o4dyfn0ef.qnssl.com/image/2017-04-01-Screen%20Shot%202017-04-01%20at%2014.02.23.png?imageView2/2/h/400) 

![](https://o4dyfn0ef.qnssl.com/image/2017-04-01-Screen%20Shot%202017-04-01%20at%2014.03.11.png?imageView2/2/h/400)

- - - - -- 

enjoy. 




 


