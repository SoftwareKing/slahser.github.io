![](https://o4dyfn0ef.qnssl.com/image/2016-10-07-Richardson-microservices-part2-3_api-gateway.png?imageView2/2/h/300) 

挖个坑,最近需要自建一个网关系统,顺手读一下下面这两位的源码. 

- [zuul-core](https://github.com/Netflix/zuul) 
- [spring-cloud-netflix-zuul](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/zuul) 

Spring Cloud更新频繁得很,版本号是根据字母排序确定稳定性的. 

我在[这里](https://spring.io/blog/category/releases)看所有项目的更新进度. 

目前的话

- B系列基于Spring Boot 1.3
- C系列基于Spring Boot 1.4  

> git chekcout v[tab]到你喜欢的版本. 

- - - - - 

## 絮絮叨叨  

网关干嘛的: 

- 调用入口
- 协议协调
- 组合&过滤 

基于zuul可以实现什么: 

- 认证&鉴权
- 数据统计
- 服务路由
- 协助单点压测
- 限流
- 静态响应 

猜测实现: 

- filter系统
- 获取服务层级列表
- 负载均衡客户端
- 与Spring Cloud全家桶对接的口子

- - - - -- 

## Spring-Cloud-Netflix-Core实现 

本文是接着[上篇](http://www.slahser.com/2016/10/06/API网关-Zuul源码解读-netflix/)来进行食用的. 

阅读本篇需要懂起码的Spring Boot auto-configure原理.起码翻一翻: 

- Meta Annotation与Composed Annotation
- ConfigurationProperties
- Conditional Annotation
- ConfigurationSelector

> 随着版本的更迭,处理请求的一些配置慢慢外移到了server配置上,具体可以看`org.springframework.boot.autoconfigure.web.ServerProperties`

### @EnableZuulProxy与@EnableZuulServer 

因为启用一个zuul服务是在太简单,配合eureka的情况下一个反代网关只需要几行代码就搭建完成了..

- @EnableZuulServer - 不带有反代功能的网关,只支持基本的route与filter系统.
- @EnableZuulProxy - 配合上服务发现与熔断开关的上面这位增强版,具有反向代理功能. 

我们先从不带reverse proxy功能的网关看起. 

### org.springframework.cloud.netflix.zuul.ZuulConfiguration 

从这份java config中可以拿到Simple模式下所有bean. 

- zuulFeature 告知actuator监控当前模式:Simple/Discovery
- ZuulProperties 配置文件内容,`zuul.ignoredServices`/`zuul.routes`形式,其中后者对应着这样的内部类定义`ZuulRoute`. 
- ServerProperties 配置文件内容
- RouteLocator 解析`ZuulProperties`来获取路由表,命中路由表等等内容.内部悄然把配置中ZuulRoute转化成自己抽象的`Route` 
- ZuulController 通过继承`ServletWrappingController`接管了上文[](http://www.slahser.com/2016/10/06/API网关-Zuul源码解读-netflix/))定义的ZuulServlet.
- ZuulHandlerMapping 响应器模式,其实目前就是把所有路径的请求导入到ZuulController上.另外的功效是当觉察RouteLocator路由表变更,则更新自己dirty状态,重新注册所有Route到ZuulController. 
- ZuulRefreshListener Simple模式下注册`RoutesRefreshedEvent`,Endpoint模式下又添加了`HeartbeatEvent`. 
- 另外就是些`ZuulFilter`子类,我们后续一个一个来说. 

可以看到Simple模式下并没有什么内容,主要内容就是: 

- 解析配置文件
- 维护路由表并监听变化
- 将请求都导向ZuulController去历经filters

### org.springframework.cloud.netflix.zuul.ZuulProxyConfiguration 

现在切换到Discovery模式. 








