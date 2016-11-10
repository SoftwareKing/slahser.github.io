![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2012.26.11.png?imageView2/2/h/200) 

这篇文章基本上算是全文转载,内容来自[这里](https://mritd.me/2016/10/29/set-up-kubernetes-cluster-by-kubeadm/)和[这里](http://www.xf80.com/2016/10/31/kubernetes-update-1.4.5) 

这两位大兄弟的博客为何如此相似,内容也差不多...,不过内容都挺好的,nice. 

我实践了一番,可行. 

那么first,我有这样的三台机器: 

| 内网ip |  hostname | CPU | 内存 | 硬盘 |
| :-----: |:--------:| :-----:|:--------:| :-----:|
| 192.168.6.51 | prod.node1 | E5-2620 V3 | 128G | 4*500G SSD |
| 192.168.6.52 | prod.node2 | E5-2620 V3 | 128G | 4*500G SSD |
| 192.168.6.53 | prod.node3 | E5-2620 V3 | 128G | 4*500G SSD |

配置条件: 暂时联网而后断网. 

- - - - -- 

### 修改host相关 

可以这样 

```shell
echo "dev.node1" > /etc/hostname
sysctl kernel.hostname=dev.node1
```

也可以这样 

`hostnamectl --static set-hostname hostname`

```shell
echo "192.168.6.51 prod.node1" >> /etc/hosts
echo "192.168.6.52 prod.node2" >> /etc/hosts
echo "192.168.6.53 prod.node3" >> /etc/hosts
``` 

- - - - -- 

### 安全相关 

```shell 
# 关闭防火墙
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
# 关闭selinu
setenforce 0
```

- - - - -- 

### 系统组件准备 

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

# 网友k8s rpm镜像
cat <<EOF> /etc/yum.repos.d/k8s.repo
[kubelet]
name=kubelet
baseurl=http://files.rm-rf.ca/rpms/kubelet/
enabled=1
gpgcheck=0
EOF
```

- - - - -- 

### 安装一些有的没的 

```shell
sudo yum makecache
sudo yum install -y docker-engine git socat kubelet kubeadm kubectl kubernetes-cni ebtables
```

- - - - -- 

### 切换docker mirror 

```shell
sudo systemctl enable docker.service
sudo systemctl start docker
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://[你的token].m.daocloud.io
sudo systemctl restart docker
```

- - - - -- 

### gcr镜像下载 

批量执行一下

```shell
images=(kube-proxy-amd64:v1.4.5 kube-scheduler-amd64:v1.4.5 kube-controller-manager-amd64:v1.4.5 kube-apiserver-amd64:v1.4.5 kube-discovery-amd64:1.0 kubedns-amd64:1.7 etcd-amd64:2.2.5 kube-dnsmasq-amd64:1.3 exechealthz-amd64:1.1 pause-amd64:3.0 kubernetes-dashboard-amd64:v1.4.1)
for imageName in ${images[@]} ; do
  docker pull jicki/$imageName
  docker tag jicki/$imageName gcr.io/google_containers/$imageName
  docker rmi jicki/$imageName
done
```

或者稳扎稳打一步一步pull->tag->rmi...限于网速这次我是这么干的. 

```shell
docker pull jicki/kube-proxy-amd64:v1.4.5
docker pull jicki/kube-scheduler-amd64:v1.4.5
docker pull jicki/kube-controller-manager-amd64:v1.4.5
docker pull jicki/kube-apiserver-amd64:v1.4.5

docker pull jicki/kube-discovery-amd64:1.0
docker pull jicki/kubedns-amd64:1.7
docker pull jicki/etcd-amd64:2.2.5
docker pull jicki/kube-dnsmasq-amd64:1.3
docker pull jicki/exechealthz-amd64:1.1
docker pull jicki/pause-amd64:3.0
# 这个已经有新版了
docker pull jicki/kubernetes-dashboard-amd64:v1.4.1
```


### k8s安装 - 基本 

执行teardown,目的是清理/etc/kubernetes文件夹内容等等.. 

```shell
systemctl stop kubelet;
docker rm -f -v $(docker ps -q);
find /var/lib/kubelet | xargs -n 1 findmnt -n -t tmpfs -o TARGET -T | uniq | xargs -r umount -v;
rm -r -f /etc/kubernetes /var/lib/kubelet /var/lib/etcd;
``` 

启动kubelet 

```shell 
systemctl enable kubelet
systemctl start kubelet
``` 

kubeadm在master节点操作 

`kubeadm init --api-advertise-addresses=192.168.6.51 --use-kubernetes-version v1.4.5`

产生的这条数据kubeadm join --token=6cd5f8.2ca419916fb17bb3 192.168.6.51 要保存好,无法重现

kubeadm在slave节点操作

`kubeadm join --token=6cd5f8.2ca419916fb17bb3 192.168.6.51`


### k8s安装 - 创建网络 

本地下载再上传到master,观察内部weave-kube镜像版本 

`wget https://git.io/weave-kube -O weave-kube.yaml`

master节点操作

```
docker pull weaveworks/weave-kube:1.8.0
kubectl create -f weave-kube.yaml
```

等待一会儿后查看网络状态,直至所有kube-system相关组件都就绪

`kubectl get pods --all-namespaces`

![](https://o4dyfn0ef.qnssl.com/image/2016-11-10-Screen%20Shot%202016-11-10%20at%2018.52.28.png) 

> 上面这张图我特意没放压缩,列位可以new tab打开来看大图 

那么目前的管控工作只能在master上执行,如果想其他节点控制kube操作: 

```shell
# 备份master节点的 配置文件
/etc/kubernetes/admin.conf
# 保存至 其他电脑, 通过执行配置文件控制集群
kubectl --kubeconfig ./admin.conf get nodes
``` 

### k8s安装 - 配置dashboard


依然是本地下载

`wget https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml -O kubernetes-dashboard.yaml`

修改里面内容将镜像版本改为v1.4.1 并且将拉取策略改为`IfNotPresent`

> 修改为你能下到的版本,另外这个拉取策略,如果你看过我的[基于Gitlab与Docker的CI](http://www.slahser.com/2016/09/07/基于Gitlab与Docker的CI/)的话一定明白...  

而后上传到主节点

```shell 
kubectl create -f kubernetes-dashboard.yaml
# 可以看到dashboard已经在运行了
kubectl get pods --all-namespaces
# 查看dashboard外网访问端口NodePort
kubectl describe svc kubernetes-dashboard --namespace=kube-system
``` 

![](https://o4dyfn0ef.qnssl.com/image/2016-11-10-Screen%20Shot%202016-11-10%20at%2018.28.52.png?imageView2/2/h/400) 

啊,生命的大和谐... 

done. 

