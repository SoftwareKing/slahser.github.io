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

## 内置的filters 

> 前提:我并不觉得这块他们实现的有多好,真上生产肯定也要定制一下. 

> 可调节的部分多得很,也可以通过zuul配置取消以下内置filter的enable. 

另外就是我会写一点能想得到的filter功能实现. 

### org.springframework.cloud.netflix.zuul.filters 

> 大家都是在操作这个RequestContext,debug时候watch它就可以解决相当多的问题了. 

这块按下不表按了有几天了,感觉棺材板都按不住了... 

所有的filer系列都是继承`ZuulFilter`,上一篇netfilx我们也说了,该抽象类具有如下部分: 

- type
- shouldFilter
- run
- order
- disable

那么接下来的部分也是分成pre/route/post三类,以及一些暂时我能想到的filter功能实现思路. 

- - - - -- 

### pre 

#### ServletDetectionFilter 

- pre
- order=-3

简要就是把之前工具类里`RequestContext.getCurrentContext().getBoolean("isDispatcherServletRequest")`的内容拿出来,塞进上下文给所有filter使用. 

对所有request生效. 

#### Servlet30WrapperFilter 

- pre
- order=-2

为了解决zuul默认的wrapper拿不到正确的HttpServletRequest而写,没什么意义. 

对所有request生效. 

#### FormBodyWrapperFilter 

- pre
- order=-1 

当Content-Type是`org.springframework.http.MediaType`中的`multipart`常量类型是生效. 

效果大概可以这么形容,无论如何都将上下文中`ZuulRequestHeaders`的`content-type`设置成了带文件类型+boundary的形式. 

具体过程是: 

1. 获取上下文中HttpServletRequest
2. 判断类型,使用FormBodyRequestWrapper封装旧的HttpServletRequestWrapper
3. 封装过程中重新构造contentData,拆出请求内可能存在的多个文件.
4. 将上一步拆开的文件headers平铺到spring抽象的HttpEntity中
5. 在wrapper中抽出contentType. 

#### DebugFilter

- pre
- order=1 

请求中含有`zuul.debug.parameter=true`时生效,将很久以前的`RequestContext`

- ctx.setDebugRouting(true);
- ctx.setDebugRequest(true);

当时我们看源码时候没解释这块,其实就是放到Map里面这样一个kv对用作标识. 

#### PreDecorationFilter 

- pre
- order=5

当一个请求没有没转发过且并没有显式指定serviceId时生效. 

效果就是补充头信息.

> 实际里面写了一大堆,遇到问题debug一下RequestContext就好了. 

具体过程: 

1. 抽取上下文中requestURI转换为Route,并获取完整点对点服务地址(location)
2. 照顾配置中zuul.customSensitiveHeaders相关,将其置入`ProxyRequestHelper`. 
3. 补充retryable属性
4. 判断location是http(s)或者forward.to
5. 前者则添加`X-Zuul-Service`头,后者则添加`forward.to`头
6. 如果这个location是服务地址的话则加上`X-Zuul-ServiceId`头
7. 补充其他头信息:X-Forwarded-Host/X-Forwarded-Port/X-Forwarded-Prefix/X-Forwarded-For/Host

- - - - -- 

### route 

#### RibbonRoutingFilter 

- route
- order=10  

在上下文中没有routeHost且sendZuulResponse为true时生效 

主要效果就是组装一个`RibbonCommandContext`也就是Request里面内含

1. 经过preFilter过滤的存储在ProxyRequestHelper中的header列表 
2. 重新组织gQueryParams/Http Method/RequestBody/serviceId/retryable
3. 糅合以上生成RibbonCommandContext
4. 还记得上文的apache/okhttp/ribbon三选一么?根据配置创建这个客户端
5. 将这个请求(RibbonCommand)使用这个客户端进行转发.
6. 获得response,并将状态码与headers记录回ProxyRequestHelper
7. 将整个response也放入ProxyRequestHelper,涉及到response头与压缩相关的内容就在这儿了. 

> Spring自己还封了一个MultiValueMap用啊,实际就是k-list的map. 

> 读着读着发现ProxyRequestHelper变成核心了.. 

#### SimpleHostRoutingFilter 

- route
- order=100 

在上下文中存在routeHost且sendZuulResponse为true时生效 

主要功效是: 

主要过程发生在一个线程中: 

1. 同上,依然是helper中组装内容
2. if (request.getContentLength() < 0) context.setChunkedRequestBody();
3. 并没有拼装RibbonCommand,而是直接使用CloseableHttpClient转发掉这个请求了(forwardRequest=execute)
4. 就这么得到了一个response. 

Connection是在一个Timer中回收的: 

```java
private void initialize() {
	this.httpClient = newClient();
	SOCKET_TIMEOUT.addCallback(this.clientloader);
	CONNECTION_TIMEOUT.addCallback(this.clientloader);
    new Timer("tt", true).schedule(() -> {
    		if (this.connectionManager == null) return;
    		this.connectionManager.closeExpiredConnections();
    }, 30000, 5000);
}
```

同时可以看到上文中这个客户端创建在一个线程中,比较有意思: 

```java
private final Runnable clientloader = () -> {
		this.httpClient.close();
		this.httpClient = newClient();
};
``` 

其他就比较传统了,ConnectionManager/SSLContext之类的.只是这个绑定callback的方式是第一次见: 

```java
DynamicIntProperty SOCKET_TIMEOUT = DynamicPropertyFactory
		.getInstance()
		.getIntProperty(ZuulConstants.ZUUL_HOST_SOCKET_TIMEOUT_MILLIS, 10000);

DynamicIntProperty CONNECTION_TIMEOUT = DynamicPropertyFactory
		.getInstance()
		.getIntProperty(ZuulConstants.ZUUL_HOST_CONNECT_TIMEOUT_MILLIS, 2000);
``` 

内容来自Netflix,这次没兴趣看了. 

#### SendForwardFilter 

- route
- order=500 

当上下文中含有`forward.to`且没被转发(execute)过的时候生效. 

> 这块这个概念翻译成中文实际有混淆,对zuul来讲把请求转走和实际的redirect/forward还是不一样的. 

功效就是简单的获取该servlet的RequestDispatcher将请求转发掉并标记转发过的状态. 

- - - - -- 

### post 

####  SendErrorFilter 

- post
- order=0 

当上下文内含`error.status_code`且没经过本filter时生效

功效就是补充上下文内error信息并找到RequestDispatcher将这个servlet走下去迎来终结. 

#### SendResponseFilter 

- post
- order=1000 

反复确认上下文允许返回内容并且该返回的内容都存在时生效. 

功效就是返回内容咯,走完servlet这一程. 

步骤如下: 

1. 获取上下文以及上下文内response. 
2. 将之前留存的zuulResponseHeaders灌到response头中
3. 处理response与body的ByteArrayInputStream编码
4. 遇到response被gzipped的情况则是判断响应接受端是否接受压缩内容.不接受则解压,接受则直接发送. 
5. writeResponse() 

```java
private void writeResponse(InputStream zin, OutputStream out) {
	byte[] bytes = new byte[INITIAL_STREAM_BUFFER_SIZE.get()];
	int bytesRead = -1;
	while ((bytesRead = zin.read(bytes)) != -1) {
		out.write(bytes, 0, bytesRead);
		out.flush();
		// doubles buffer size if previous read filled it
		if (bytesRead == bytes.length) 
            bytes = new byte[bytes.length * 2];
	}
}
```

> 感觉DynamicIntProperty还是要看看,怎么被用了这么多. 

- - - - --- 

## 自定义filter思路 

 



