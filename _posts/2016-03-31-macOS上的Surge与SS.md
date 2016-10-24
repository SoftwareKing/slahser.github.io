### intro 

接近两年的日子里我一直在用ybb.  

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-03-31%20at%2014.40.40.png?imageView2/2/h/100) 

在系统中开启了一个SOCKS5代理,代理全局,同时支持shell环境与ssh的代理 
 
稳定,好吃不贵.  

不过目前已经停止注册,我也只是在停止注册前给自己跟老婆买了两份,一直续费至今. 

最近买挂耳咖啡,店家赠送了几十G的SS流量,索性拿出之前的Surge for Mac Beta操作一番. 

>Surge与SS二者均需单独购买

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-03-31%20at%2014.51.24.png?imageView2/2/h/100) 

### install 

进入Surge for iOS -> More -> 摇一摇手机 -> Surge for Mac ->输入 step1 的[链接](http://surge.run/Surge-Mac.zip)下载app 

打开mac上Surge.app,而后点击手机Surge for Mac页面的step4进行激活 

### config 

Mac上编写配置文件`vim ~/.surge.conf`,内容如文末所示  

打开 Network->advanced->proxies 勾选如图所示http与https代理,填入`127.0.0.1:6152` 

在同一页面下方bypass列表填入`*.local, 169.254/16, 127.0.0.1, 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12, localhost` 

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-03-31%20at%2014.54.51.png?imageView2/2/h/300) 

### usage 

打开Surge.app ,勾选DIRECT是本机直连,其他选项为普通海外代理/ShadowSocks代理等等.  

速度虽然不如ybb快但是也够用了: 

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-03-31%20at%2013.32.32.png?imageView2/2/h/400) 

done. 

osx 上配置文件模板如下: 

```properties
[General]
loglevel = notify
skip-proxy = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12,127.0.0.0/24
bypass-tun = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12,127.0.0.0/24

// DNS OVERRIDE
dns-server = 223.6.6.6,223.5.5.5,114.114.114.114,114.114.115.115

[Proxy]
# Proxy = http,$IP,$PORT,$USERNAME,$PASSWORD
# Proxy = https,$IP,$PORT,$USERNAME,$PASSWORD
# Proxy = socks,$IP,$PORT,$USERNAME,$PASSWORD

# Shadowsocks 配置，代理方式一定要是 custom
# Proxy = custom,$IP,$PORT,$METHOD,$PASSWORD,$MODULE_URL 
# 
# 可用配置文件 http://proxy.sofi.sh/surge.conf
# 代理需要 Shadowsocks 的一个模块，这里我放一个 http://proxy.sofi.sh/SSEncrypt.module

DIRECT=direct
HTTP=http,$IP,$PORT,$USERNAME,$PASSWORD
HTTPS=https,$IP,$PORT,$USERNAME,$PASSWORD
SOCKS=socks,$IP,$PORT,$USERNAME,$PASSWORD
SS=custom,$IP,$PORT,$METHOD,$PASSWORD,$MODULE_URL

[Proxy Group]
Proxy = select,DIRECT,HTTP,HTTPS,SOCKS,SS

[Rule]

// REJECT RULES BLOCK SOME ADS
DOMAIN-SUFFIX,icloud-analysis.com,REJECT
DOMAIN-SUFFIX,x.jd.com,REJECT
DOMAIN-SUFFIX,sax.sina.cn,REJECT
DOMAIN-SUFFIX,tanx.com,REJECT
DOMAIN-SUFFIX,ads.genieessp.com,REJECT
DOMAIN-SUFFIX,ad.unimhk.com,REJECT
DOMAIN-SUFFIX,cpro.baidustatic.com,REJECT
DOMAIN-SUFFIX,m.simaba.taobao.com,REJECT
DOMAIN-SUFFIX,ads.yahoo.com,REJECT
DOMAIN-SUFFIX,ib.adnxs.com,REJECT
DOMAIN-SUFFIX,i-mobile.co.jp,REJECT
DOMAIN-SUFFIX,g.doubleclick.net,REJECT
DOMAIN-SUFFIX,adinfuse.com,REJECT
DOMAIN-SUFFIX,admob.com,REJECT
DOMAIN-SUFFIX,adwhirl.com,REJECT
DOMAIN-SUFFIX,appads.com,REJECT
DOMAIN-SUFFIX,inmobi.com,REJECT
DOMAIN-SUFFIX,smartadserver.com,REJECT
DOMAIN-SUFFIX,adsage.cn,REJECT
DOMAIN-SUFFIX,duomeng.cn,REJECT
DOMAIN-SUFFIX,duomeng.net,REJECT
DOMAIN-SUFFIX,duomeng.org,REJECT
DOMAIN-SUFFIX,googeadsserving.cn,REJECT
DOMAIN-SUFFIX,guomob.com,REJECT
DOMAIN-SUFFIX,immob.cn,REJECT
DOMAIN-SUFFIX,mobads.baidu.com,REJECT
DOMAIN-SUFFIX,mobads-logs.baidu.com,REJECT
DOMAIN-SUFFIX,tapjoyads.com,REJECT

// CN DIRECT
DOMAIN-SUFFIX,cn,DIRECT
DOMAIN-SUFFIX,jd.com,DIRECT
DOMAIN-SUFFIX,yhd.com,DIRECT
DOMAIN-SUFFIX,58cdn.com,DIRECT
DOMAIN-SUFFIX,qunarzz.com,DIRECT
DOMAIN-SUFFIX,avosapps.com,DIRECT
DOMAIN-SUFFIX,mob.com,DIRECT
DOMAIN-SUFFIX,same.com,DIRECT
DOMAIN-SUFFIX,toutiao.com,DIRECT
DOMAIN-SUFFIX,zaih.com,DIRECT
DOMAIN-SUFFIX,lantouzi.com,DIRECT
DOMAIN-SUFFIX,smzdm.com,DIRECT
DOMAIN-SUFFIX,126.net,DIRECT
DOMAIN-SUFFIX,163.com,DIRECT
DOMAIN-SUFFIX,alicdn.com,DIRECT
DOMAIN-SUFFIX,amap.com,DIRECT
DOMAIN-SUFFIX,bdimg.com,DIRECT
DOMAIN-SUFFIX,bdstatic.com,DIRECT
DOMAIN-SUFFIX,cnbeta.com,DIRECT
DOMAIN-SUFFIX,cnzz.com,DIRECT
DOMAIN-SUFFIX,douban.com,DIRECT
DOMAIN-SUFFIX,gtimg.com,DIRECT
DOMAIN-SUFFIX,hao123.com,DIRECT
DOMAIN-SUFFIX,haosou.com,DIRECT
DOMAIN-SUFFIX,ifeng.com,DIRECT
DOMAIN-SUFFIX,iqiyi.com,DIRECT
DOMAIN-SUFFIX,netease.com,DIRECT
DOMAIN-SUFFIX,qhimg.com,DIRECT
DOMAIN-SUFFIX,qq.com,DIRECT
DOMAIN-SUFFIX,sogou.com,DIRECT
DOMAIN-SUFFIX,sohu.com,DIRECT
DOMAIN-SUFFIX,soso.com,DIRECT
DOMAIN-SUFFIX,suning.com,DIRECT
DOMAIN-SUFFIX,tmall.com,DIRECT
DOMAIN-SUFFIX,tudou.com,DIRECT
DOMAIN-SUFFIX,weibo.com,DIRECT
DOMAIN-SUFFIX,youku.com,DIRECT
DOMAIN-SUFFIX,xunlei.com,DIRECT
DOMAIN-SUFFIX,zhihu.com,DIRECT
DOMAIN-SUFFIX,huofu.com,DIRECT
DOMAIN-SUFFIX,5wei.com,DIRECT

DOMAIN-KEYWORD,alipay,DIRECT
DOMAIN-KEYWORD,taobao,DIRECT
DOMAIN-KEYWORD,baidu,DIRECT
DOMAIN-KEYWORD,360buy,DIRECT
DOMAIN-KEYWORD,baidu,DIRECT

// Apple
DOMAIN-SUFFIX,appldnld.apple.com,DIRECT
DOMAIN-SUFFIX,adcdownload.apple.com,DIRECT
DOMAIN-SUFFIX,lcdn-registration.apple.com,DIRECT
DOMAIN-SUFFIX,swcdn.apple.com,DIRECT
DOMAIN-SUFFIX,phobos.apple.com,DIRECT
DOMAIN-SUFFIX,ls.apple.com,DIRECT
DOMAIN-SUFFIX,icloud.com,Proxy
DOMAIN-SUFFIX,instagram.com,Proxy
DOMAIN-SUFFIX,twimg.com,Proxy,force-remote-dns
DOMAIN-SUFFIX,kenengba.com,Proxy,force-remote-dns
DOMAIN-SUFFIX,apple.com,Proxy
DOMAIN-SUFFIX,amazonaws.com,Proxy
DOMAIN-SUFFIX,android.com,Proxy
DOMAIN-SUFFIX,angularjs.org,Proxy
DOMAIN-SUFFIX,appspot.com,Proxy
DOMAIN-SUFFIX,akamaihd.net,Proxy
DOMAIN-SUFFIX,amazon.com,Proxy
DOMAIN-SUFFIX,bit.ly,Proxy
DOMAIN-SUFFIX,bitbucket.org,Proxy
DOMAIN-SUFFIX,blog.com,Proxy
DOMAIN-SUFFIX,blogcdn.com,Proxy
DOMAIN-SUFFIX,blogger.com,Proxy
DOMAIN-SUFFIX,blogsmithmedia.com,Proxy
DOMAIN-SUFFIX,box.net,Proxy
DOMAIN-SUFFIX,bloomberg.com,Proxy
DOMAIN-SUFFIX,chromium.org,Proxy
DOMAIN-SUFFIX,cl.ly,Proxy
DOMAIN-SUFFIX,cloudfront.net,Proxy
DOMAIN-SUFFIX,cloudflare.com,Proxy
DOMAIN-SUFFIX,cocoapods.org,Proxy
DOMAIN-SUFFIX,crashlytics.com,Proxy
DOMAIN-SUFFIX,dribbble.com,Proxy
DOMAIN-SUFFIX,dropbox.com,Proxy
DOMAIN-SUFFIX,dropboxstatic.com,Proxy
DOMAIN-SUFFIX,dropboxusercontent.com,Proxy
DOMAIN-SUFFIX,docker.com,Proxy
DOMAIN-SUFFIX,duckduckgo.com,Proxy
DOMAIN-SUFFIX,digicert.com,Proxy
DOMAIN-SUFFIX,dnsimple.com,Proxy
DOMAIN-SUFFIX,edgecastcdn.net,Proxy
DOMAIN-SUFFIX,engadget.com,Proxy
DOMAIN-SUFFIX,eurekavpt.com,Proxy
DOMAIN-SUFFIX,fb.me,Proxy
DOMAIN-SUFFIX,fbcdn.net,Proxy
DOMAIN-SUFFIX,fc2.com,Proxy
DOMAIN-SUFFIX,feedburner.com,Proxy
DOMAIN-SUFFIX,fabric.io,Proxy
DOMAIN-SUFFIX,flickr.com,Proxy
DOMAIN-SUFFIX,fastly.net,Proxy
DOMAIN-SUFFIX,ggpht.com,Proxy
DOMAIN-SUFFIX,github.com,Proxy
DOMAIN-SUFFIX,github.io,Proxy
DOMAIN-SUFFIX,githubusercontent.com,Proxy
DOMAIN-SUFFIX,golang.org,Proxy
DOMAIN-SUFFIX,goo.gl,Proxy
DOMAIN-SUFFIX,gstatic.com,Proxy
DOMAIN-SUFFIX,godaddy.com,Proxy
DOMAIN-SUFFIX,gravatar.com,Proxy
DOMAIN-SUFFIX,imageshack.us,Proxy
DOMAIN-SUFFIX,imgur.com,Proxy
DOMAIN-SUFFIX,jshint.com,Proxy
DOMAIN-SUFFIX,ift.tt,Proxy
DOMAIN-SUFFIX,itunes.com,Proxy
DOMAIN-SUFFIX,j.mp,Proxy
DOMAIN-SUFFIX,kat.cr,Proxy
DOMAIN-SUFFIX,linode.com,Proxy
DOMAIN-SUFFIX,linkedin.com,Proxy
DOMAIN-SUFFIX,licdn.com,Proxy
DOMAIN-SUFFIX,lithium.com,Proxy
DOMAIN-SUFFIX,megaupload.com,Proxy
DOMAIN-SUFFIX,mobile01.com,Proxy
DOMAIN-SUFFIX,modmyi.com,Proxy
DOMAIN-SUFFIX,mzstatic.com,Proxy
DOMAIN-SUFFIX,nytimes.com,Proxy
DOMAIN-SUFFIX,name.com,Proxy
DOMAIN-SUFFIX,openvpn.net,Proxy
DOMAIN-SUFFIX,openwrt.org,Proxy
DOMAIN-SUFFIX,ow.ly,Proxy
DOMAIN-SUFFIX,pinboard.in,Proxy
DOMAIN-SUFFIX,Proxyl-images-amazon.com,Proxy
DOMAIN-SUFFIX,Proxytatic.net,Proxy
DOMAIN-SUFFIX,stackoverflow.com,Proxy
DOMAIN-SUFFIX,staticflickr.com,Proxy
DOMAIN-SUFFIX,squarespace.com,Proxy
DOMAIN-SUFFIX,symcd.com,Proxy
DOMAIN-SUFFIX,symcb.com,Proxy
DOMAIN-SUFFIX,symauth.com,Proxy
DOMAIN-SUFFIX,ubnt.com,Proxy
DOMAIN-SUFFIX,t.co,Proxy
DOMAIN-SUFFIX,thepiratebay.org,Proxy
DOMAIN-SUFFIX,tumblr.com,Proxy
DOMAIN-SUFFIX,twimg.com,Proxy
DOMAIN-SUFFIX,twitch.tv,Proxy
DOMAIN-SUFFIX,twitter.com,Proxy,force-remote-dns
DOMAIN-SUFFIX,wikipedia.com,Proxy
DOMAIN-SUFFIX,wikipedia.org,Proxy
DOMAIN-SUFFIX,wikimedia.org,Proxy
DOMAIN-SUFFIX,wordpress.com,Proxy
DOMAIN-SUFFIX,wsj.com,Proxy
DOMAIN-SUFFIX,wsj.net,Proxy
DOMAIN-SUFFIX,wp.com,Proxy
DOMAIN-SUFFIX,vimeo.com,Proxy
DOMAIN-SUFFIX,youtu.be,Proxy
DOMAIN-SUFFIX,ytimg.com,Proxy

// LAN
IP-CIDR,192.168.0.0/16,DIRECT
IP-CIDR,10.0.0.0/8,DIRECT
IP-CIDR,172.16.0.0/12,DIRECT
IP-CIDR,127.0.0.0/8,DIRECT

// Telegram
IP-CIDR,91.108.56.0/22,Proxy,no-resolve
IP-CIDR,91.108.4.0/22,Proxy,no-resolve
IP-CIDR,109.239.140.0/24,Proxy,no-resolve
IP-CIDR,149.154.160.0/20,Proxy,no-resolve


GEOIP,CN,DIRECT
FINAL,Proxy
``` 

done. 



