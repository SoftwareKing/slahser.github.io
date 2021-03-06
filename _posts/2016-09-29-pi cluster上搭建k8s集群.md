![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2012.26.11.png?imageView2/2/h/200) 

> 看了[这个Session](https://youtu.be/0mIwhAJz2Gg?t=1809)后很激动,索性在pi cluster上装一下k8s. 

> 文中命令大部分可以用ansible同步执行.  

- - - - -- 

## 烧录存储卡 

访问[flash工具](https://github.com/hypriot/flash)了解用途

```
flash -n [youthostname] https://github.com/hypriot/image-builder-rpi/releases/download/v1.1.0/hypriotos-rpi-v1.1.0.img.zip
```

- - - - -- 

## ip查询 & 本机hosts添加

- 通过小米路由器后台查看所有机器ip

本机hosts添加: 

```shell
vim /etc/hosts 
# raspberry
192.168.31.222 rpi2.node
192.168.31.104 rpi3.node1
192.168.31.130 rpi3.node2
192.168.31.228 rpi3.node3
192.168.31.126 rpi3.node4
192.168.31.132 rpi3.node5
192.168.31.127 rpi3.node6
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

> 本步骤不推荐

```shell
sudo passwd root
sudo passwd --unlock root
sudo nano /etc/ssh/sshd_config # 当然了,其他修改也是在这里,比如密码重置,端口设置
PermitRootLogin yes
Control + X -> Y -> Enter
su -
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

当然也可以ansible执行

```
echo "deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib" > /etc/apt/sources.list  
echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib" >> /etc/apt/sources.list 
``` 
- - - - --- 

## 搭建私服 

详见[这里](https://www.slahser.com/2016/09/29/pi-cluster上配套简易Registry/) 

- - - - -- 

## 修改dns 

```shell
vim /etc/resolv.conf
nameserver 119.29.29.29
``` 

- - - - --- 

## 集群搭建 

后续可以看[关于kubernetes-1.5+-搭建](https://www.slahser.com/2016/11/10/关于kubernetes-1.5+-搭建/),因为不只是在pi cluster上玩玩而已了. 

将教程内amd64修改为arm64就可以了. 

- - - - - 

done. 



