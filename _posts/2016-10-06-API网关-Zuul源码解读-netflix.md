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

## Zuul-Core实现 

### com.netflix.zuul.context.RequextContext 

`RequestContext extends ConcurrentHashMap<String, Object>`

可以从比较完备的单元测试中窥得功能,维护这http请求上下文,是一个threadlocal的Map. 

另含一个返回自身clone的copy()方法.其余就是写增删改查内存的操作.  

> 项目的测试写的也挺好的,框架都搭好了,我公司的同学为什么就不能好好写测试呢 

### com.netflix.zuul.contextContextLifecycleFilter

调用链外围finally中remove上文中threadlocal. 

### com.netflix.zuul.http.ZuulServlet 

那么有filter就有Wrapper,就有Request实现,看过Spring Session实现就知道这是必然的. 

- service() - 调用init将req,res置入上下文.获取并标记上下文此session已经通过进入zuul引擎
- init() - 注册一个ZuulRunner用于call起filter
- 通过ZuulRunner调用流程: pre/route/post/error

其中ZuulRunner内传入的req/res就会被替换为wrapper类,增强功能:

- HttpServletRequestWrapper - chunked
- HttpServletResponseWrapper - 独立出一个status属性

###  com.netflix.zuul.filters.FilterRegistry 

Map来维护filter列表. 

### com.netflix.zuul.filters.ZuulServletFilter

跟`ZuulServlet`是同一份代码.

### com.netflix.zuul.monitoring 

留了`CounterFactory`与`TracerFactory`的口子用来工人实现counter与timer. 

> 实现在`com.netflix.zuul.plugins`有,不过不在本篇就是了. 

### com.netflix.zuul.groovy 

提供了接受groovy脚本filter的能力,实现就是一个`GroovyClassLoader`来load Class. 

- - - - -- 

那么跳脱出来,外部调用就是下面这样的. 

### com.netflix.zuul.IZuulFilter 

几经修改之后现在只剩两个接口方法了: 

- shouldFilter()
- run()

而抽象实现类ZuulFilter内功能: 

- order
- disable
- 正常的run&返回result

那么整个调用的入口来了

### com.netflix.zuul.FilterProcessor 

内含FilterUsageNotifier将filter名字前加上`zuul.filter-`按tag记录调用次数(或者自己的其他实现). 

- preRoute
- postRoute
- route
- error

通过`FilterLoader`把注册在FilterRegistry中的filter按类别与order提取. 

> 这里order是因为`ZuulFilter`实现了Comparable<T>,所以子类可想而知. 

真正调用doFilter时记录调用时间置入上下文: 

```java
ctx.addFilterExecutionSummary(filterName, ExecutionStatus.FAILED.name(), execTime);
``` 

### com.netflix.zuul.FilterFileManager  

那最后这块是groovy的文件filter加载,这里有一个`FilterFileManager`来轮询某个目录(数据库也行) 

观测到脚本更新更新则通过`FilterLoader.putFilter()`置入`FilterRegistry`. 

> 推荐使用vfs重实现. 

那这块几个例子就是groovy目录下`com.netflix.zuul.filters`里的内容. 

- - - - -- 

## 总结 

Netflix-Core里面只是提供了一个filter系统,提供了pre/route/post/error的filterType标准. 

具体的调用部分还是要看各家实现. 

--------- 

## Spring-Cloud-Zuul 

Spring Cloud另外写了一份来接入这份filter系统.而并非`zuul-netflix`. 

篇幅会很长干脆挪到[这里](http://www.slahser.com/2016/10/08/API网关-Zuul源码解读-spring-cloud/)分开写好了. 

done. 







 






