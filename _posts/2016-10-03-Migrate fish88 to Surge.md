![](https://o4dyfn0ef.qnssl.com/image/2016-10-03-Screen%20Shot%202016-10-03%20at%2018.19.04.png?imageView2/2/h/150)  

- - - - - 

## 原因 

鱼摆摆应该是我用过最舒服的代理服务了,不排除以后我还会转回来. 

- 最近鱼摆摆速度一般
- 搬瓦工跟Vultr服务我觉得还挺舒服的
- 买了Surge for Mac,$50不想浪费. 

## 鱼摆摆  

几年前的日子里我一直在用ybb.  

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-03-31%20at%2014.40.40.png?imageView2/2/h/100) 

稳定,9元/月好吃不贵. 

最近貌似重新开放注册了,地址是[这里](https://ybb1024.com),悄咪咪分享一下.  

- - - - -- 

## Terminal  

从本行开始算是一个逐渐切换过程,因为ybb有一些配置我们要清掉. 

实际就是两家各种争夺`Network->Advanced->Proxies`里面

- http_proxy
- https_proxy
- socks5_proxy

那么: 

```
set -Ux http_proxy http://127.0.0.1:6152
set -Ux https_proxy http://127.0.0.1:6152
set -Ux socks_proxy http://127.0.0.1:6153
``` 

![](https://o4dyfn0ef.qnssl.com/image/2016-10-03-Screen%20Shot%202016-10-03%20at%2018.33.29.png?imageView2/2/h/300) 

## Surge for Mac  

1. 取消fish88内开机启动,系统网络设置权限. 
2. 取消login item内fish88. 
3. Surge set as system proxy 使三种代理生效
4. 拉取iCloud上surge.conf配置

## bugfix 

刚切换之后ssh无法使用真是把我吓尿了.. 

检查后调整`~/.ssh/config`: 

```
ProxyCommand /usr/bin/nc -x 127.0.0.1:9742 %h %p
到
ProxyCommand /usr/bin/nc -x 127.0.0.1:6153 %h %p
``` 

## 配置 

以上配置来自[这里](https://github.com/lhie1/Surge). 

屏蔽广告啊,apple服务加速啊什么的都有,挺好的. 

## enhance mode 

自从Surge开了enhance mode之后,上面这个配置文件需要修改一下才能正常活动 

enhance mode要求必须设置dns-override,第一个最好写成实际的内网dns192.168.0.1而不是system. 

因为system已经被替换成了240.0.0.2了. 

## 扩展阅读  

- [让人耳目一新的 Surge Mac 2.0](https://medium.com/@scomper/让人耳目一新的-surge-mac-2-0-bb7cf735b1b8#.q1vsnhi7r)
- [Surge -定制自己的规则配置](https://medium.com/@scomper/surge-定制自己的规则配置-34a6d74b0434?source=latest---)
- [用 Excel 整理配置文件中的规则](https://medium.com/@scomper/用-excel-整理配置文件中的规则-dd0e886c9b80?source=latest---)
- [Surge MitM 证书的创建和配置](https://medium.com/@scomper/surge-mitm-证书的创建和配置-d2d4aced4169)
- [局域网其他设备共享上网](https://medium.com/@scomper/局域网其他设备共享上网-dd29e18853da#.6c9uniiys)
- [如何屏蔽网页上的流量「飘浮球」](https://medium.com/@scomper/如何屏蔽网页上的流量-飘浮球-662a449e61e3#.osvrzz7hz)
- [Surge 网页广告拦截案例分析](https://medium.com/@scomper/surge-网页广告拦截案例分析-db88b8c8bbca)
- [Surge 的增强模式（TUN）](https://medium.com/@scomper/surge-的增强模式-tun-cc0aaad86ff5)

- - - - -- 

![](https://o4dyfn0ef.qnssl.com/image/2016-10-03-Screen%20Shot%202016-10-03%20at%2016.45.42.png?imageView2/2/h/300) 

- - - - -- 

done. 


