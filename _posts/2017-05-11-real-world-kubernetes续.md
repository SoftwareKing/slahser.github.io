![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-kube7-logo.png?imageView2/2/h/200) 

[上文](https://www.slahser.com/2017/05/02/real-world-kubernetes/)只是基本的集群搭建,里面还什么东西都没有. 

我整理了一下把一个集群做成可用的状态需要多少工作 

于是建立了一个项目[real-world-kubernetes](https://github.com/Slahser/real-world-kubernetes). 

做了几方面的工作: 

- 修复了全部的 RBAC 权限,而且大部分是比较 clear 的状态
- 添加了大部分需要暴露服务的 ingress
- resource 使用进行了调优
- 相应的存储引擎还没动
- 操作顺序是起亲手 tuning 过的. 

> 有些项目内部镜像是写死的,所以使用 private registry 会有问题 
> 
> 这时只能打 tag 来解决了,或者提个 PR. 

- - - - --- 

### calico 

[原地址](http://docs.projectcalico.org/v2.2/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml) 

在上一文 join 之后执行 

官方已经修改了 RBAC 相关内容 

内置的也是 http 的 etcd,所以没什么可调节的. 

### heapster 

[原地址](https://github.com/kubernetes/heapster/blob/master/deploy/kube-config/standalone/heapster-controller.yaml) 

大部分人喜欢这部分用 influxdb 的那个版本,这并不好. 

官方内置了 RBAC. 

### traefik 

[原地址](https://github.com/containous/traefik/blob/master/examples/k8s/traefik-with-rbac.yaml) 

这块可以调节的地方比较多,比如: 

- 外置的 toml
- 接入 prometheus
- 内存 cpu 申请

### dns-scale 

[原地址](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns-horizontal-autoscaler)

按照集群规模来伸缩 dns 副本数. 

挺有用的项目. 

### dashboard 

[原地址](https://github.com/kubernetes/dashboard)

不在 addon 里的单独的项目版本更新,自带了 rbac 

> v1.6.0带有一个比较严重的 bug, daemonset 里面不显示内容... 

### prometheus 

[原地址](https://github.com/giantswarm/kubernetes-prometheus)

比较均衡的一个项目,部署起来比官方舒服一点 

但是作者比较懒,到现在还没有 rbac 支持,我提交了一个 pr 过去. 


### redis sentinel 

[原地址](https://github.com/kubernetes/kubernetes/tree/master/examples/storage/redis) 

这个操作起来的确需要一些顺序... 

1. 执行redis1.yaml 等待,查看log
2. 执行redis2.yaml 等待,查看log
3. `kubectl scale rc redis --replicas=3 -n storage`
4. `kubectl scale rc redis-sentinel --replicas=3 -n storag`
5. 等待,查看log
6. `kubectl delete pods redis-master -n storage`

事实证明直接起两个 rc 的确无法将 sentinel 集群起来. 

两个点 : 

- redis 版本是2.8
- 调用本集群需要使用 sentinel 客户端,直接访问 svc 即可. 

想使用在 namespace: storage 中的应用,可以先: 

`kubectl exec -ti yourpod -n yournamespace -- nslookup redis-sentinel.storage` 

来试试看访问权限. 

### fission 

[原地址](https://github.com/fission/fission)

> 创建环境时候缺` fission/fetcher`这么个镜像得自己 tag目前. 

这块我帮着合并了 svc 跟 review 了一个 pr解决了一些 rbac 问题. 

```
fission --server http://[masterip]:31313 env create --name nodejs --image registry.yourcompany.com/node-env

fission  --server http://[masterip]:31313 function create --name hello --env nodejs --code hello.js

fission  --server http://[masterip]:31313 route create --method GET --url /hello --function hello

curl http://[masterip]:31314/hello
```

神奇吧. 

### kubenetic 

