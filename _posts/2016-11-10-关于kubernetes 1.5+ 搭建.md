![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2012.26.11.png?imageView2/2/h/200) 

> 最近明显感觉到k8s-on-arm的作者不太上心..不过也促成了kubernetes本身在HypriotOS上的表现更好了. 

折腾一个月来感触颇多,总之多看源码多翻issue吧,方案也进入了试运行的阶段,不再追新. 

> 2017-04-12更新 
> 
> 就搭建来讲,[follow-me-install-kubernetes-cluster](https://github.com/opsnull/follow-me-install-kubernetes-cluster)比我这里写的要详细科学很多. 
> 所以日后安装相关就不再更新

以下是k8s生态的一些快捷链接 

我每隔几天就要翻,姑且拿来备忘: 

- [Kubernetes-Github](https://github.com/kubernetes)
- [组件文档](http://kubernetes.io/docs/user-guide/annotations/)
- [配置教程](http://kubernetes.io/docs/user-guide/simple-nginx/)
- [k8s-release](https://github.com/kubernetes/release) - 手动编译
- [kubeadm changelog](https://github.com/kubernetes/kubeadm/blob/master/CHANGELOG.md)
- [Google rpm repo](https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/repodata/primary.xml)
- [gcr.io registry](https://console.cloud.google.com/kubernetes/images/list?location=GLOBAL&project=google-containers)
- [Heapster yaml](https://github.com/kubernetes/heapster/tree/master/deploy/kube-config/influxdb) 
- [使用Helm的CD流程](https://articles.microservices.com/gitlab-consumer-driven-contracts-helm-and-kubernetes-b7235a60a1cb#.f7hu5pld8)

- - - - -- 

## 硬件环境 

那么首先,我有这样的三台机器: 

| 内网ip |  hostname | 
| :-----: |:--------:| 
| 192.168.6.51 | dev.node1 | 
| 192.168.6.52 | dev.node2 | 
| 192.168.6.53 | dev.node3 | 

- - - - -- 

## 检查系统信息 

```shell 
# 7.2
cat /etc/system-release
# 3.10
uname -r
```

- - - - -- 

## 系统组件准备 

```shell
# 切换yum源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 清华docker镜像 
tee /etc/yum.repos.d/docker.repo << EOF
[dockerrepo]
name=Docker Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker/yum/repo/centos7
enabled=1
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker/yum/gpg
EOF
```

> 这部分有修改,请看[k8s后日谈 源与镜像](http://www.slahser.com/2016/11/17/K8s后日谈-源与镜像/)中rpm的部分. 

```shell
sudo yum makecache
sudo yum update -y
sudo yum install -y epel-release
sudo yum install -y docker-engine git socat ebtables vim sshpass lrzsz wget telnet bind-utils htop iotop iftop iptraf tofrodos lsof iperf traceroute git policycoreutils-python bash-completion net-tools iptables-services bridge-utils 
# 源于镜像中下载好的rpm,不清理直接升级安装也行实际..  
rpm -ivh *.rpm

systemctl enable docker.service
systemctl start docker.service
```

- - - - -- 

## 修改host相关 

```shell
# 修改hostname
hostnamectl set-hostname dev.node1
# 同步hosts
echo "192.168.6.51 dev.node1" >> /etc/hosts
echo "192.168.6.52 dev.node2" >> /etc/hosts
echo "192.168.6.53 dev.node3" >> /etc/hosts
echo "192.168.6.xx registry.yourcompany.com" >> /etc/hosts
``` 
 
- - - - - 

## 修改环境变量 

因为1.5开始的kubeadm支持了设置环境变量,所以再也不用tag来tag去了. 

感谢阿里云的[PR](https://github.com/kubernetes/kubernetes/pull/35948). 

```shell
echo "KUBE_REPO_PREFIX=registry.yourcompany.com" >> /etc/profile
source /etc/profile 
``` 

- - - - -- 

## 安全相关 

```shell 
# 关闭防火墙
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
# 关闭selinux
setenforce 0
``` 

- - - - -- 

## 私服搭建 

参考[pi-cluster上配套Registry](http://www.slahser.com/2016/09/29/pi-cluster上配套Registry/)来搭建一个. 

1. 那么我们保存证书 /cert/registry.yourcompany.com.crt 出来. 
2. 分发到所有k8s节点上`/etc/docker/certs.d/registry.yourcompany.com/`中. 
3. 同时所有节点同步hosts `echo 'xx.xx.xx.xx registry.yourcompany.com' >> /etc/hosts`
4. 测试 `curl -k https://registry.yourcompany.com/v2/`

- - - - -- 

## 清理旧环境 

### 执行teardown 

`kubeadm reset` 

> 这步在安装完kubelet后执行一次

### 清理旧yum  

```
yum remove kubelet kubeadm kubectl kubernetes-cni
# 或者卸载之前手动安装的 
rpm -qa | grep kube 
rpm -e --nodeps [component]
```

### 清理容器文件 

```
/var/lib/docker/containers 
``` 

### 清理cni配置残余 

这步很多人忘掉,导致切换网络方案时候一直cni错误. 

> 更新: 可以看到[这里](https://github.com/kubernetes/kubeadm/blob/master/CHANGELOG.md),更新了kubeadm reset的表现,将cni清理加入进来了. 

### 清理网卡 

```
ip link delete cni0 
ip link delete flannel.1
ip link delete weave
...
```

类似的该删除就删除. 

### 清理iptables 

```
iptables -L -n
iptables -t nat -S 

iptables -F
iptables -X
iptables -Z
iptables -t nat -F
iptables -t nat -X
iptables -t nat -Z
``` 

- - - - -- 

## 外部etcd集群安装 

[这里](https://github.com/coreos/etcd/releases) 下一个新版release. 

> 这步Optional.我的docker内etcd还算稳定.  

- - - - -- 

## 镜像下载 

> 版本请看[k8s后日谈 源与镜像](http://www.slahser.com/2016/11/17/K8s后日谈-源与镜像/)私服部分 

因为1.5+的版本支持了自定义kubeadm仓库地址 

所以以下的部分相比2016.12之前方便了大概几十倍.   

### 基础镜像 


- kube-proxy-amd64:v1.5.1 
- kube-scheduler-amd64:v1.5.1 
- kube-controller-manager-amd64:v1.5.1 
- kube-apiserver-amd64:v1.5.1

> 本部分设置环境变量 

### 扩展部分 

- kube-discovery-amd64:1.0 
- kubedns-amd64:1.9 
- etcd-amd64:3.0.14-kubeadm 
- kube-dnsmasq-amd64:1.4 
- exechealthz-amd64:1.2 
- dnsmasq-metrics-amd64:1.0 
- pause-amd64:3.0 

> 本部分设置环境变量 

### 网络组件 

- etcd:2.2.1
- calico/kube-policy-controller:v0.4.0
- calico/cni:v1.4.3
- calico/ctl:v0.23.0
- calico/node:v0.23.0
- weaveworks/weave-npc:1.8.2
- weaveworks/weave-kube:1.8.2
- flannel-git:v0.6.1-28-g5dde68d-amd64

> 本部分修改yaml文件 
 
### 可视化组件 

- kubernetes-dashboard-amd64:v1.5.0
- heapster_grafana:v3.1.1 
- kubernetes/heapster:canary
- kubernetes/heapster_influxdb:v0.6

> 本部分修改yaml文件 

- - - - -- 

## kubeadm  

执行teardown,目的是清理/etc/kubernetes文件夹内容等等.. 

```shell
kubeadm reset
```

> 它会停止所有目前正在运行的容器. 

启动kubelet 

```shell 
systemctl enable kubelet
systemctl start kubelet
``` 

kubeadm在master节点操作 

`kubeadm init --api-advertise-addresses=192.168.6.51 --use-kubernetes-version v1.5.1 --token=xxxxxx.xxxxxxxxxxxxxxxx --external-etcd-endpoints xxx,xxx`

> 这里要指定版本,否则那四个核心组件与永远是1.4.4..  

> 更多的kubeadm文档可以看[这里](http://kubernetes.io/docs/admin/kubeadm/) 

`在执行完下一步网络创建之后`,再用kubeadm在minion节点操作

`kubeadm join --token=xxxxxx.xxxxxxxxxxxxxxxx 192.168.6.51`

- - - - -- 

## 网络创建

网络方案对比可以参考[这里](https://seanzhau.com/blog/post/seanzhau/d35e8ffeafe4)和[这里](http://blog.dataman-inc.com/shurenyun-docker-133/),传说中CNM与CNI之争..  

> 修改yaml的image地址添加私服地址. 

### Calico  

优点: 

- 比较好做网络策略,进行隔离
- 性能好  

yml来自[Calico文档](http://docs.projectcalico.org/v1.6/getting-started/kubernetes/installation/hosted/kubeadm/)的[这个文件](http://docs.projectcalico.org/v1.6/getting-started/kubernetes/installation/hosted/kubeadm/calico.yaml). 

### Weave 

虽然我压测下来Weave表现并不好,但是下面这两种高性能方案的DNS问题我暂时没有搞定.  

所以暂时选型Weave,在其他方面找回一点性能,另外两种作为探索. 

yml来自[这里](https://git.io/weave-kube),修改拉取策略. 

而你的机器上会多出来无数的网卡,伴随着服务的增多. 

### Flannel 

那么[Flannel的yaml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml)

Backend修改为host-gw

执行完这一步ifconfig能看到网卡cni0
如果依然是vxlan的话会看到另一张flannel.1的网卡创建

### 配置后检查 

检查`node<->pod<->pod<->node`的连通性. 

`docker exec -it [cid] /bin/sh`来ping. 

- - - - -- 

## Dashboard

yaml内容源码在[这里](https://github.com/kubernetes/dashboard/blob/master/src/deploy/kubernetes-dashboard.yaml),也可以wget下载[这个](https://github.com/kubernetes/dashboard/blob/master/src/deploy/kubernetes-dashboard.yaml)

修改image地址添加私服. 

```shell 
kubectl create -f kubernetes-dashboard.yaml
# 查看dashboard外网访问端口NodePort,30000–32767
kubectl describe svc kubernetes-dashboard -n kube-system
``` 

> 这步注意的是创建了一个deploy一个svc,想彻底删除dashboard的话需要清理干净.  

> `kubectl delete deploy,svc,rc,po -l app=kubernetes-dashboard -n kube-system`

![](https://o4dyfn0ef.qnssl.com/image/2016-11-10-Screen%20Shot%202016-11-10%20at%2019.04.35.png?imageView2/2/h/400) 

啊,生命的大和谐... 

## 参考 

这篇文章参考了一些内容,来自: 

- [1](http://hustcat.github.io)  
- [2](https://seanzhau.com/blog/seanzhau)
- [3](https://mritd.me/)
- [4](http://www.cnblogs.com/openxxs/)

done. 

