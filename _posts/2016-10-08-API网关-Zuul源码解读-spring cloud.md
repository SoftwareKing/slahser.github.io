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

## Spring-Cloud-Netflix--Core实现 


