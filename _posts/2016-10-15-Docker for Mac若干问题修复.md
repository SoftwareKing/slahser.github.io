Docker for Mac一只小问题不断,直到今天实在有点block工作了,索性集中解决一下. 

问题列表: 

- 证书问题,提示X509
- 代理问题
- Compose端口占用问题

截止今天为止,Docker for Mac的

- stable版本是1.12.1
- beta版本是1.12.3-rc1-beta29.1 (13437) 

问题在issue与fourm中反馈的很集中,后两个问题是我自己发现的解决方法,暂时还没有贡献到社区去. 

- - - - --- 

### 证书问题,提示X509 

> 官方定义为bug. 

按道理来讲,我们搭建好私服,同步了证书过来,同时访问的是https的registry,应该是万无一失咯? 

linux机器上将证书cp至`/etc/docker/certs.d/[yourregistry]`就可以访问`curl -k https://[yourregistry]/v2/`了. 

mac的话经过几次的升级已经完全不支持这样的方式. 

解决方案: 

- 使用带端口的--insecure-registry
- 切换到Beta Channel
 
- - - - --- 

### 代理问题 

> 官方定义为bug 

如果你的机器上如下位置的代理是有内容的,比如下图 

![](https://o4dyfn0ef.qnssl.com/image/2016-10-27-Screen%20Shot%202016-10-27%20at%2016.24.44.png?imageView2/2/h/400) 

那么恭喜你的Docker for Mac会自动检测到这个设置并且将其设置进docker的启动参数中. 

而且,而且当这个代理是127.0.0.1的情况下,那么你将与任何服务起进行socket连接. 

解决方案: 

- 按住Option点击wifi图标,查到本机局域网ip,手动设置到http(s)的位置上. 

ps: 

感谢Surge的这个贴心功能,让我确信gcr的镜像如今我也下得到了. 

50刀花的值. 

![](https://o4dyfn0ef.qnssl.com/image/2016-10-27-Screen%20Shot%202016-10-27%20at%2016.27.38.png?imageView2/2/h/400) 

- - - - --- 

### Compose的端口占用问题 

启动`docker-compose up`时提示端口占用,一般发生在443/80上. 

这个问题反馈人数也是很多,但是官方没有认定为bug,我潜心解决了一下... 

解决方案: 

- 删除brew中的php7
- 删除本机的macOS server
- 确认机器上不存在apache 

- - - - --- 

done . 











