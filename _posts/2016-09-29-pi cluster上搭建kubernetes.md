![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2012.26.11.png?imageView2/2/h/200) 

> 看了[这个Session](https://youtu.be/0mIwhAJz2Gg?t=1809)后很激动,索性在pi cluster上装一下k8s. 

> 文中命令大部分可以用ansible同步执行.  

- - - - -- 

## 烧录存储卡 

访问[flash工具](https://github.com/hypriot/flash)了解用途

```
flash -n [youthostname] https://downloads.hypriot.com/hypriotos-rpi-v1.0.0.img.zip
```

- - - - -- 

## ip查询 & 本机hosts添加

- 通过小米路由器后台查看所有机器ip
- ping rpi3-1.local 

本机hosts添加: 

```shell
vim /etc/hosts 
# raspberry
192.168.31.222 rpi2-1
192.168.31.104 rpi3-1
192.168.31.130 rpi3-2
192.168.31.228 rpi3-3
192.168.31.126 rpi3-4
192.168.31.132 rpi3-5
192.168.31.127 rpi3-6
``` 

ansible hosts添加: 

```shell 
vim ~/.ansible/hosts
[pis]
192.168.31.222
192.168.31.104
192.168.31.130
192.168.31.228
192.168.31.126
192.168.31.132
192.168.31.127
[pis:vars]
ansible_ssh_user=pirate
ansible_ssh_pass=hypriot
``` 

测试 
 
```shell 
ansible pis -m ping -o
```

- - - - -- 

## 同步公钥免密码 

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub pirate@[youthostname] 
with password 'hypriot'
```

- - - - -- 

## 开启root用户 

```shell
sudo passwd root
sudo passwd --unlock root
nano /etc/ssh/sshd_config # 当然了,其他修改也是在这里,比如密码重置,端口设置
PermitRootLogin yes
Control + X -> Y -> Enter
reboot
```

- - - - -- 

## 修改镜像源

可以先看一下[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/)

> 注意rpi.local与rpi的区别,同时不断维护/etc/hosts与~/.ssh/known_hosts

```
nano /etc/apt/sources.list
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib

apt-get update
apt-get install -y vim
``` 

- - - - -- 

## 修改dns 

```shell
vim /etc/resolv.conf
nameserver 8.8.8.8 
```

- - - - -- 

## k8s安装 

可以看一下[kubernetes-on-arm](https://github.com/luxas/kubernetes-on-arm)  

这个项目的作者凭借这个项目加入了kubernetes项目组,他目前也是负责跨平台代码的开发.  

> 安了这个做好心理准备跨版本不兼容更新,也就是说你要重新烧录SD卡才能升级到k8s 1.4.0

```shell
wget https://github.com/luxas/kubernetes-on-arm/releases/download/v0.8.0/docker-multinode.deb
dpkg -i docker-multinode.deb
kube-config install 
# 时区填写 Asia/Shanghai
# 开启1G的SWAP
# 重启虽然在Hypriot上不需要,不过我还是选择重启
```

重点来了,重启后`kube-config info`会提示你`gcr.io/google_containers/etcd-arm:2.2.5`下不到.  

这当然是我大清特色了. 

> 不过令我惊讶的是我机器上的Socks5代理跟http代理都无法访问Google Container Registry...  

于是昨晚我四点多悻悻睡去. 

第二天上午换了个脑子找到国内的两家良心Docker云,比DaoCloud强到不知道哪儿去了. 

- [时速云hub](https://hub.tenxcloud.com)
- [灵雀云hub](https://hub.alauda.cn)

所以如下可以解决问题 

```shell 
docker pull index.tenxcloud.com/google_containers/etcd-arm:2.2.5
docker pull index.alauda.cn/googlecontainer/etcd-arm:2.2.5
``` 

但是我不放心,还是想搭建个私服打tag留住这两个镜像.  

- - - - --- 

## 搭建私服 

详见[这里](http://www.slahser.com/2016/09/29/pi-cluster上配套简易Registry/) 

- - - - -- 

## 