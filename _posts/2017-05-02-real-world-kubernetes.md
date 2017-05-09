![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-kube7-logo.png?imageView2/2/h/200) 

最近单位要起一个新的k8s集群,大家说起来搞个k8s集群,都头头是道. 

```
外置etcd奇数节点集群,haproxy做vip,搞几个master. 
master上kube-apiserver,kube-controller-manager,kube-scheduler
minion上装上kubelet,kube-proxy
中间同步一下证书,同步一下kubeconfig认证,组件再RBAC填上SA.
然后添个dns三件套就完事了

再不就kargo配一下ansible怎么说也ok了. 
```

naive. 

个中缘由有很多,我目前还是倾向于用`kubeadm`来搭建集群. 

master的高可用先不做,但是k8s与calico的元数据会外置. 

> 1.6.1->1.6.2 之间调整了kubelet的cgroup相关内容,让人对k8s开发团队简直失去了信心. 
> 
> 他们只做了e2e test,绝对没有做手工测试才会产生这种事故. 

- k8s版本: [1.6.2](https://share.weiyun.com/ad36abaa3fe91d3ad5cce450d1b40adc) 
- Linux版本: CentOS 7.3.1611
- Linux Kernel: 4.10.13

rpm获取依然是[k8s后日谈 源与镜像](http://www.slahser.com/2016/11/17/K8s后日谈-源与镜像/)中RPM部分获取. 

> 当然也可以用vps自己编译. 

- - - - --- 

> 机器最开始还是联网吧

### 同步ssh公钥 

```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@[ip]
```

### 修改hostname 

```
hostnamectl set-hostname [*]
```

### 同步/etc/hosts

```
echo "[ip1] k8s-master-1" >> /etc/hosts
echo "[ip2] k8s-node-1" >> /etc/hosts
echo "[ip3] k8s-node-2" >> /etc/hosts
echo "[ip4] k8s-node-3" >> /etc/hosts
echo "[ip5] registry.[yourcompany].com" >> /etc/hosts
```

### 换源 

```
# 清华大学源,我这里用起来比阿里云的强一点 
https://mirrors.tuna.tsinghua.edu.cn/help/centos/
https://mirror.tuna.tsinghua.edu.cn/help/epel/

sudo yum makecache
sudo yum update -y
sudo yum install -y epel-release

# 偶尔yum会卡住,其余情况自行google
jobs -l 
lill -9 
```

### 关闭防火墙  

```
systemctl disable firewalld
systemctl stop firewalld
```

### 关闭selinux 

> 不推荐setenforce 0这种,多的就不解释了..是测试出来的. 

修改`/etc/selinux/config`中第一项为disabled. 

重启机器后`getenforce`查询. 

> 重启推荐和下一步一起做. 

### 升级内核 

```
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
# 内核不要4.11及以上,试了你就知道了. 
sudo yum --enablerepo=elrepo-kernel install kernel-ml-4.10.* -y

# 引导文件我知道有这两处,哪个行查哪个
sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
sudo egrep ^menuentry /boot/efi/EFI/centos/grub.cfg | cut -f 2 -d \'

sudo grub2-set-default [从零开始]
```

而后`sudo shutdown -r now`重启机器 

`uname -r`查询. 

### 调整必要 

```
vim /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

sudo sysctl --system
```

### 安装必要 

```
# 这里直接安装1.12.6,不要添加docker的源了
yum install docker socat
# 安装k8s那4个rpm 
yum -ivh *.rpm
```

### 同步私服证书 

参考 [pi-cluster上配套Registry](https://www.slahser.com/2016/09/29/pi-cluster上配套Registry/) 

### 调整配置文件 

[kubeadm配置文档](https://github.com/kubernetes/kubernetes.github.io/blob/master/docs/admin/kubeadm.md)

```
vim /etc/sysconfig/docker
调整 
OPTIONS='--log-driver=json-file --signature-verification=false'

vim /etc/sysconfig/docker-storage
调整 
DOCKER_STORAGE_OPTIONS="--storage-driver=overlay"

vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
添加 
Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.yourcompany.com/pause-amd64:3.0"

vim /etc/profile 
添加 
export KUBE_REPO_PREFIX=registry.yourcompany.com
export KUBE_ETCD_IMAGE=registry.yourcompany.com/etcd-amd64:3.0.17
source /etc/profile 

sudo systemctl daemon-reload
kubeadm reset
sudo systemctl enable docker
sudo systemctl enable kubelet
sudo systemctl start docker
sudo systemctl start kubelet
```


### 解决镜像问题 

> 我们暂时用k8s与calicp都用内置的etcd.接下来再不断解决问题. 

那我们列一下清单

```
# 核心组件,随着大版本走
kube-apiserver-amd64:v1.6.2
kube-controller-manager-amd64:v1.6.2
kube-scheduler-amd64:v1.6.2
kube-proxy-amd64:v1.6.2
# dns组件,可以替换成coredns
k8s-dns-kube-dns-amd64:1.14.1
k8s-dns-dnsmasq-nanny-amd64:1.14.1
k8s-dns-sidecar-amd64:1.14.1
# 空镜像
pause-amd64:3.0
# calico组件,etcd有内置外置两种部署方式
calico/node:v1.1.3
calico/cni:v1.8.0
calico/kube-policy-controller:v0.5.4
# 下面这两个可选
etcd-amd64:3.0.17
etcd:2.2.1
```

### 开始 

[kubeadm文档](https://kubernetes.io/docs/getting-started-guides/kubeadm/)下方可选步骤 

```
master 
kubeadm init 

# 也有人喜欢先生成token,然后再指定..
# kubeadm init --token=xxxxxx.xxxxxxxxxxxxxxxx

sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf

# 将conf内从拷贝至本机~/.kube/config文件内即可远程访问
```

```
minion 
kubeadm join --token 8ec45a.01782b1b4059a682 192.168.6.95:6443

# 实际这个token可以在master上自己指定
# 我一般将其设置为xxxxxx.xxxxxxxxxxxxxxxx
# 日后拓展就算忘了也没关系
# 实际它存在-n kube-system下secret/bootstrap-token
```

### 部署calico 

> 因为拷贝了kubeconfig,当然此步骤可以在本机上执行. 

```
kubectl apply -f http://docs.projectcalico.org/v2.1/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml

# 需要手动调整一下image. 
```

### 测试DNS 

参考 [这里](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) 

### 其他 

不是dashboard,heapster,prometheus就不说了,可以参考以前我的博客. 

[这里](https://kubernetes.io/docs/user-guide/kubectl/v1.6/)是kubectl的操作文档.别忘了我们可以在本机上操作了. 

或者说

![](https://o4dyfn0ef.qnssl.com/image/2017-05-04-E399F559-E814-4181-85F9-509DC1110BB1.png?imageView2/2/h/300) 

另外内置的etcd元数据目录分别在

- 3.0.17 /var/lib/etcd
- 2.2.1 /var/etcd

前者会被`kubeadm reset`清理,后者则需要手动清理,它们都是hostPath. 

单机上启动kubelet的日志都在`/var/log/message`里,这个是需要特别关注的. 


- - - - --- 

上面是初步的迅速启动一个集群,那么接下来我们需要将etcd外置并且接入进去. 

因为k8s 1.6开始全面启用etcd v3作为元数据存储 

但是calico暂时还只支持etcd v2的api. 

所以calico暂时还是内部的etcd好了. 

那么kubeadm之前有一个--external-etcd-endpoints的flag 

逐渐废除掉了,现在支持的是这样的形式 

```
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
  advertiseAddress: 192.168.6.95
  bindPort: 6443
etcd:
  endpoints:
  - http://192.168.6.95:2379
  - http://192.168.6.90:2379
  - http://192.168.6.91:2379
kubernetesVersion: v1.6.2
token: xxxxxx.xxxxxxxxxxxxxxxx
```

保存为kubeadm-init.yaml 

`kubeadm init --config kubeadm-init.yaml` 

即可. 

> 后来我的dns遇到了写入etcd集群有问题的情况...不过先调着吧 

### 另外的tips 

其实iptables并不是老是要清理,但是虚拟网卡需要 

如果机器网络情况切换过,那么calico创建的网卡

`ip link delete tunl0` 

同时在minion节点上,需要这样: 

`docker tag registry.yourcompany.com pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0`
- - - - ---- 


done. 






