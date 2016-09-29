![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2012.26.11.png?imageView2/2/h/200) 

> 看了[这个Session](https://youtu.be/0mIwhAJz2Gg?t=1809)后很激动,索性在pi cluster上装一下k8s. 

```
https://github.com/hypriot/flash
flash -n rpi3-4 https://downloads.hypriot.com/hypriotos-rpi-v1.0.0.img.zip
ssh-copy-id -i ~/.ssh/id_rsa.pub pirate@rpi3-4
sudo passwd root
sudo passwd --unlock root
nano /etc/ssh/sshd_config 当然了,其他修改也是在这里,比如密码重置,端口设置
PermitRootLogin yes
Control + X /Y/Enter
reboot

修改源
https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/
nano /etc/apt/sources.list

deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ jessie main non-free contrib

apt-get update

注意rpi.local与rpi的区别,同时不断维护/etc/hosts与~/.ssh/known_hosts

apt-get install -y vim


修改dns

vim /etc/resolv.conf
添加nameserver 8.8.8.8 解决问题

kube-config install 

Asia/Shanghai
开启1G的SWAP

重启后

kube-config info

搭建私服
```