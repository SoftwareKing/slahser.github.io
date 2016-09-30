书接[上文](http://www.slahser.com/2016/09/29/pi-cluster上搭建kubernetes/),我们遇到了GFW无法下载gcr.io的镜像,而后遇到了国内容器云的事儿. 

缺点: 

- 树莓派上网速不知道为什么pull速度总是不行.
- 说不定他们网站哪天倒闭了呢.. 

- - - - --- 

## 更换树莓派上docker mirror 

这里我是用了[Daocloud的Mirror](https://dashboard.daocloud.io)来加速官方镜像下载.  

```shell
# 这样
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://[yours].m.daocloud.io
# 或者这样
vim /etc/default/docker
--registry-mirror=http://[yours].m.daocloud.io
# 而后重启docker
```
- - - - --- 

## 搭建简易Registry 

### 同步hosts 

```shell
echo '192.168.31.100 index.slahser.com' >> /etc/hosts
``` 

### 自签证书准备 

```shell
cd ~/.ssh
openssl genrsa -out index.slahser.com.key 2048
# 倒数第二步的name设置成index.slahser.com
openssl req -newkey rsa:4096 -nodes -sha256 -keyout index.slahser.com.key -x509 -days 365 -out index.slahser.com.crt
``` 

### 宿主机与客户机证书信任 

```shell
mkdir -p /etc/docker/certs.d/index.slahser.com
# ansible copy或者单机cp 
ansible pis -m copy -a 'src=/Users/Slahser/.ssh/index.slahser.com.crt dest=/etc/docker/certs.d/index.slahser.com/'
cp ~/.ssh/index.slahser.com.crt /etc/docker/certs.d/index.slahser.com/
```

### 仓库启动 

```shell
docker run -d -p 443:5000 --restart=always --name registry \
  -v /Users/Slahser/.ssh:/certs \
  -v /Users/Slahser/Documents/repository/registry:/var/lib/registry \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/index.slahser.com.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/index.slahser.com.key \
  registry:2.5.1
``` 

> 个人小tips:不要轻易使用latest来偷懒. 

> 另外mac下1024一下端口是不开放的.所以可能会遇到其妙的问题,所以土一点save/load也是可以的. 

### 宿主机私服设置 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-30-Screen%20Shot%202016-09-30%20at%2002.29.37.png?imageView2/2/h/300) 

### 怎么打TAG 

mac上操作: 

```shell
docker pull index.tenxcloud.com/google_containers/etcd-arm:2.2.5
docker tag [yourimageid] index.slahser.com/google_containers/etcd-arm:2.2.5
docker push index.slahser.com/google_containers/etcd-arm:2.2.5
```

pi上操作: 

```shell
docker pull index.slahser.com/google_containers/etcd-arm:2.2.5
docker tag [yourimageid] gcr.io/google_containers/etcd-arm:2.2.5
```

done. 


