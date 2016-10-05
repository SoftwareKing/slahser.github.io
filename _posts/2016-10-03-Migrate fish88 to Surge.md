![](https://o4dyfn0ef.qnssl.com/image/2016-10-03-Screen%20Shot%202016-10-03%20at%2018.19.04.png?imageView2/2/h/150)  

- - - - - 

## 原因 

鱼摆摆应该是我用过最舒服的代理服务了,不排除以后我还会转回来. 

- 最近鱼摆摆速度一般
- 搬瓦工速度居然在一年80块的情况下速度还不错...我分享给了9个设备. 
- 买了Surge for Mac,$50不想浪费. 

- - - - -- 

## Terminal  

实际就是两家各种争夺`Network->Advanced->Proxies`里面

- http_proxy
- https_proxy
- socks5_proxy

那么命令行: 

```
set -Ux http_proxy http://127.0.0.1:6152
set -Ux https_proxy http://127.0.0.1:6152
set -Ux socks_proxy http://127.0.0.1:6153
``` 

![](https://o4dyfn0ef.qnssl.com/image/2016-10-03-Screen%20Shot%202016-10-03%20at%2018.33.29.png?imageView2/2/h/300) 

## App 

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

我更新一下我的配置文件:

```
[General]
dns-server = 223.6.6.6,223.5.5.5,114.114.114.114,114.114.115.115

loglevel = notify

skip-proxy = 127.0.0.1, 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12, 100.64.0.0/10, localhost, *.local, e.crashlytics.com

ipv6 = true

bypass-tun = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12
bypass-system = true

interface = 0.0.0.0
socks-interface = 0.0.0.0

port = 6152
socks-port = 6153

allow-wifi-access = 0

[Proxy]
💊 Direct = direct
🇯🇵 JP = custom,xxx.xxx.xxx.xxx,xxx,rc4-md5,xxxx,https://raw.githubusercontent.com/lhie1/Surge/master/SSEncrypt.module
🇺🇸 US = custom,xxx.xxx.xxx.xxx,xxx,aes-256-cfb,xxxx,https://raw.githubusercontent.com/lhie1/Surge/master/SSEncrypt.module

[Proxy Group]
✈️ Proxy = select, 💊 Direct, 🇯🇵 JP, 🇺🇸 US, 🏃 Auto
cn Proxy = select, 💊 Direct, ✈️ Proxy
🍎 Proxy = select, 💊 Direct, 🇯🇵 JP, 🇺🇸 US
🏃 Auto = url-test, 🇯🇵 JP, 🇺🇸 US, url = http://www.gstatic.com/generate_204

[Rule]
*** 
``` 
 
以上配置来自[这里](https://github.com/lhie1/Surge). 

屏蔽广告啊,apple服务加速啊什么的都有,挺好的. 

![](https://o4dyfn0ef.qnssl.com/image/2016-10-03-Screen%20Shot%202016-10-03%20at%2016.45.42.png?imageView2/2/h/300) 

- - - - -- 

done. 


