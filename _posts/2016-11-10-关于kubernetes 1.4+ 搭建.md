![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2012.26.11.png?imageView2/2/h/200) 

> 最近明显感觉到k8s-on-arm的作者不太上心..不过也促成了kubernetes本身在HypriotOS上的表现更好了. 

那么首先,我有这样的三台机器: 

| 内网ip |  hostname | 
| :-----: |:--------:| 
| 192.168.6.51 | dev.node1 | 
| 192.168.6.52 | dev.node2 | 
| 192.168.6.53 | dev.node3 | 

配置条件: 暂时联网而后断网. 

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
sudo yum install -y docker-engine git socat ebtables  
# 源于镜像中下载好的rpm,不清理直接升级安装也行实际..  
rpm -ivh *.rpm

systemctl enable docker.service
systemctl start docker
```

- - - - -- 

## 清理旧环境 

### 执行teardown 

`kubectl reset`或者 

```shell
systemctl stop kubelet;
docker rm -f -v $(docker ps -q);
find /var/lib/kubelet | xargs -n 1 findmnt -n -t tmpfs -o TARGET -T | uniq | xargs -r umount -v;
rm -r -f /etc/kubernetes /var/lib/kubelet /var/lib/etcd;
```

### remove旧yum  

```
yum remove kubelet kubeadm kubectl kubernetes-cni
# 或者卸载之前手动安装的 
rpm -qa | grep kube 
rpm -e --nodeps [component]
```

### 清理cni配置残余 

这步很多人忘掉,导致切换网络方案时候一直cni错误. 

来源依然是[这里](https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/repodata/primary.xml) 

其中`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 

我们看到kubelet的启动参数,`kubeadm reset`是不会清理cni配置的. 

在`kubeadm init`与`kubeadm join`前我们手动清空掉`/etc/cni/net.d`,所有节点上的残余. 

### 清理网卡 

```
ip link delete cni0 
ip link delete flannel.1
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

1. 复制`etcd`与`etcdctl`到`/usr/local/bin`
2. 在个节点上分别执行下方初始化命令
3. 数据存储就是在文件夹内`cluset.name`中

```shell
etcd --name dev.node1 --initial-advertise-peer-urls http://192.168.6.51:2380 \
  --listen-peer-urls http://192.168.6.51:2380 \
  --listen-client-urls http://192.168.6.51:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.6.51:2379 \
  --initial-cluster-token my-etcd-cluster \
  --initial-cluster dev.node1=http://192.168.6.51:2380,dev.node2=http://192.168.6.52:2380,dev.node3=http://192.168.6.53:2380 \
  --initial-cluster-state new &
```

这里是示例作用,目前kubeadm有bug,不支持master节点上运行etcd 

会无法通过端口检查.可以采取将etcd集群放在master之外. 

> 这步Optional. 

- - - - -- 

## 镜像下载 

> 这部分有修改,请看[k8s后日谈 源与镜像](http://www.slahser.com/2016/11/17/K8s后日谈-源与镜像/)私服部分 

版本来自上文. 

### 基础镜像 

```shell
images=(kube-proxy-amd64:v1.5.0-beta.2 kube-scheduler-amd64:v1.5.0-beta.2 kube-controller-manager-amd64:v1.5.0-beta.2 kube-apiserver-amd64:v1.5.0-beta.2)
for imageName in ${images[@]} ; do
  docker pull registry.yourcompany.com/$imageName
  docker tag registry.yourcompany.com/$imageName gcr.io/google_containers/$imageName
  docker rmi registry.yourcompany.com/$imageName
done
``` 

### 扩展部分 

```shell
images=(kube-discovery-amd64:1.0 kubedns-amd64:1.7 etcd-amd64:2.2.5 kube-dnsmasq-amd64:1.3 exechealthz-amd64:1.1 pause-amd64:3.0 kubernetes-dashboard-amd64:v1.5.0 heapster_grafana:v3.1.1 etcd:2.2.1)
for imageName in ${images[@]} ; do
  docker pull registry.yourcompany.com/$imageName
  docker tag registry.yourcompany.com/$imageName gcr.io/google_containers/$imageName
  docker rmi registry.yourcompany.com/$imageName
done
```

### add-on  

```shell
images=(kubernetes/heapster:canary kubernetes/heapster_influxdb:v0.6 calico/kube-policy-controller:v0.4.0 calico/cni:v1.4.3 calico/ctl:v0.23.0 weaveworks/weave-npc:1.8.1 weaveworks/weave-kube:1.8.1)
for imageName in ${images[@]} ; do
  docker pull registry.yourcompany.com/$imageName
  docker tag registry.yourcompany.com/$imageName $imageName
  docker rmi registry.yourcompany.com/$imageName
done
``` 

### 遗漏部分 

```shell
images=(calico/node:v0.23.0 flannel-git:v0.6.1-28-g5dde68d-amd64)
for imageName in ${images[@]} ; do
  docker pull registry.gogen.com/$imageName
  docker tag registry.gogen.com/$imageName quay.io/$imageName
  docker rmi registry.gogen.com/$imageName
done
``` 

- - - - -- 

## kubeadm  

执行teardown,目的是清理/etc/kubernetes文件夹内容等等.. 

```shell
kubeadm reset
```

或者

```shell
systemctl stop kubelet;
docker rm -f -v $(docker ps -q);
find /var/lib/kubelet | xargs -n 1 findmnt -n -t tmpfs -o TARGET -T | uniq | xargs -r umount -v;
rm -r -f /etc/kubernetes /var/lib/kubelet /var/lib/etcd;
``` 

> 它会停止所有目前正在运行的容器. 

启动kubelet 

```shell 
systemctl enable kubelet
systemctl start kubelet
``` 

kubeadm在master节点操作 

`kubeadm init --api-advertise-addresses=192.168.6.51 --use-kubernetes-version v1.5.0-beta.2 --token=xxxxxx.xxxxxxxxxxxxxxxx --pod-network-cidr 10.244.0.0/16 --external-etcd-endpoints xxx,xxx`

> 这里要指定版本,否则那四个核心组件与永远是1.4.4..  

> 更多的kubeadm文档可以看[这里](http://kubernetes.io/docs/admin/kubeadm/) 

`在执行完下一步网络创建之后`,再用kubeadm在minion节点操作

`kubeadm join --token=xxxxxx.xxxxxxxxxxxxxxxx 192.168.6.51`

- - - - -- 

## 网络创建

网络方案对比可以参考[这里](https://seanzhau.com/blog/post/seanzhau/d35e8ffeafe4) 和[这里](http://blog.dataman-inc.com/shurenyun-docker-133/). 

各种网络框架的对比,传说中CNM与CNI之争..  

### Weave 

虽然我压测下来Weave表现并不好,但是下面这两种高性能方案的DNS问题我暂时没有搞定.  

所以暂时选型Weave,在其他方面找回一点性能,另外两种作为探索. 

yml来自[这里](https://git.io/weave-kube),修改拉取策略. 

而你的机器上会多出来无数的网卡,伴随着服务的增多. 

### Calico    

优点: 

- 比较好做网络策略,进行隔离
- 性能好  

yml来自[Calico文档](http://docs.projectcalico.org/v1.6/getting-started/kubernetes/installation/hosted/kubeadm/)的[这个文件](http://docs.projectcalico.org/v1.6/getting-started/kubernetes/installation/hosted/kubeadm/calico.yaml). 

其中镜像我们可以看镜像篇,或者直接看这个yml内容进行准备. 

- - - - -- 

### Flannel 

那么[Flannel的yaml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml)

Backend修改为host-gw

执行完这一步ifconfig能看到网卡cni0
如果依然是vxlan的话会看到另一张flannel.1的网卡创建

- - - - -- 

## dashboard

yaml内容源码在[这里](https://github.com/kubernetes/dashboard/blob/master/src/deploy/kubernetes-dashboard.yaml) 

也可以wget下载[这个](https://github.com/kubernetes/dashboard/blob/master/src/deploy/kubernetes-dashboard.yaml)

将拉取策略改为`IfNotPresent`. 

> 修改为你能下到的版本,另外这个拉取策略,如果你看过我的[基于Gitlab与Docker的CI](http://www.slahser.com/2016/09/07/基于Gitlab与Docker的CI/)的话一定明白...  

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

done. 

