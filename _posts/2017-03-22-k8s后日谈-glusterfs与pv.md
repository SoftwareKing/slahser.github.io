![](https://o4dyfn0ef.qnssl.com/image/2017-03-26-k8s-glusterfs-scaleway.png?imageView2/2/h/300) 

在分布式存储方面了解的不多,比较倾向于: 

- 块存储 - Glusterfs
- 小文件 - FastDFS

这次做的demo主要是如下流程: 

`efk->存储es数据->做块网络硬盘`. 

> 2017-04 补充 [heketi/heketi](https://github.com/heketi/heketi)也值得关注 

## 安装Glusterfs 

Centos7+的版本一般都自带Glusterfs.  

> `rpm -qa | grep gluster`查看各个组件 
> 
> gluster-lib/common/server/client啊之类的

另外也可以按照[这里](https://wiki.centos.org/SpecialInterestGroup/Storage/gluster-Quickstart)安装. 

关键步骤: 

- `/etc/fstab`里如果已经挂过硬盘,直接修改即可. 
- [firewalld开启](http://www.cnblogs.com/moxiaoan/p/5683743.html)tcp/24007端口
- 测试挂载后直接[umount](http://man.chinaunix.net/linux/mandrake/101/zh_cn/Command-Line.html/fs-and-mntpoints-mount.html)即可. 

## 创建PV 

Kubernetes关于Storage的文档在[这里](https://kubernetes.io/docs/concepts/storage/volumes/). 

> 在dashboard上我们可以清楚地看到PV是admin scope下的. 

那么: 

1. k8s node上有[gluster-client](http://gluster.readthedocs.io/en/latest/Administrator%20Guide/Setting%20Up%20Clients/?highlight=client). 
2. 创建Endpoint
3. 创建PV给所有namespace来申请

> 安装不好gluster-client可以[自己解决一下](https://buildlogs.centos.org/centos/7/storage/x86_64/gluster-3.10/) 

官方的Glusterfs[示例](https://github.com/kubernetes/kubernetes/tree/master/examples/volumes/glusterfs)在这里,不过是json形式的,而且引入形式比较陈旧.  

于是我用了如下工具来转换与验证一下: 

- [json2yaml](https://www.json2yaml.com)
- [yamllint](http://www.yamllint.com) 

于是gluster-endpoint.yaml 

```
kind: Endpoints
apiVersion: v1
metadata:
  name: glusterfs-cluster
subsets:
- addresses:
  - ip: your-gluster-ip1
  ports:
  - port: 1
- addresses:
  - ip: your-gluster-ip2
  ports:
  - port: 1
```

> 这块ip不能用域名形式,只能是ip 

而后gluster-pv.yaml 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gluster-volume2
spec:
  capacity:
    storage: 120Gi
  accessModes:
    - ReadWriteMany
  glusterfs:
    endpoints: "glusterfs-cluster"
    path: "glusterfs-data"
    readOnly: false
```

> 当然这个创建可分配容量是有限制的,无法超过当时创建的brick大小. 

## 创建PVC 

官方文档在[这里](https://kubernetes.io/docs/user-guide/persistent-volumes/#persistentvolumeclaims). 

> 再次观察dashboard会发现PVC是namespace scope下的. 

简单写下,gluster-pvc.yaml 

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: gluster-es-pvc
  namespace: kube-system
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 80Gi
``` 

那么分享一个我的测试: 

- 我用同一个240G的brick创建了一块100G的PV,一块120G的PV
- 我在`-n kube-system`下申请了80G的PVC
- 那么其中一块PV会从ready状态变为bound状态,并且不可继续被申请. 

感受一下. 

- - - - -- 

后续肯定会搞得有: 

- PVC申请这个到底怎么回事 
- 几种读写模式只取决于NFS方案的支持程度还是其他
- Glusterfs扩容

- - - - - 

done. 



