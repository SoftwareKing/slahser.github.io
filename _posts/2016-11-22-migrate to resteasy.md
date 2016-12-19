![](https://o4dyfn0ef.qnssl.com/image/2016-11-22-20160919113218_31916.gif?imageView2/2/h/300)

科普的东西就不多说了,JAX-RS(JavaAPI for RESTful Web Services)也就是实现了JSR-311. 

- Jersey - Sun家的,Spring Cloud中默认的就是Jersey
- Resteasy -JBoss家的,口碑极佳 

压测下来Undertow + Resteasy的表现真是好的过分...内网调用如果要是rest方式,那么首选这两个. 

同时也留了三个小口子可以供以后优化,毕竟优化要分批的来嘛. 

- - - - - 

## 依赖准备  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
<dependency>
    <groupId>javax.ws.rs</groupId>
    <artifactId>javax.ws.rs-api</artifactId>
    <version>2.0</version>
</dependency>
<dependency>
    <groupId>com.paypal.springboot</groupId>
    <artifactId>resteasy-spring-boot-starter</artifactId>
    <version>2.2.0-RELEASE</version>
    <scope>runtime</scope>
</dependency>
``` 

> lombok也是必要的 

- - - - - 

## app注册 

只要classpath中存在resteasy-starter的话就会默认接管所有rest服务 

而我们这期老旧代码的Spring MVC服务并不想进行替换,只是内部调用打算这么进行替换,那么 

```java
@Component
@ApplicationPath("/inner-api")
public class JaxrsApplication extends Application{
}
```

虽然配置文件中也可以这么手动去指定 

```
resteasy:
  jaxrs:
    app:
      registration: property
      classes: com.slahser.ss.JaxrsApplication
```

不过自动扫描这个app只是启动那一次,扫描就扫描一次吧,用起来干净了很多. 

> 当然这个app可以注册多个. 

- - - - - 

## Resource编写 

这样 

```java
@Path("/ttjsxrs")
@Component
public class TtJaxRs {
    @Path("/s2")
    @GET
//  @Produces({ MediaType.APPLICATION_JSON })
    @Produces({ MediaType.TEXT_PLAIN })
    public String s2() { return "s2"; }
}
```

- - - - - 

## 调用方 

那么顺理成章的可以使用RestTemplate: 

```java
@Bean
public RestTemplate restTemplate() { return new RestTemplate(); }
@Autowired
private RestTemplate restTemplate;

@GetMapping("/uss")
public String uss() {
    return restTemplate.
             getForObject("http://slahser-ss/inner-api/ttjsxrs/ss",
                          String.class);
}
```

当然这里你走Spring Cloud Eureka跟k8s自带的dns都ok. 

- - - - - 

## 留下的部分 

我们留下了一点可以优化的部分 

- 既然是rest方式,响应对象可以精简一点,利用一下http状态码也未尝不可
- 默认序列化是jsckson,我们也可以替换其他provider
- restTemplate用起来方便,但是效率有没有这么好呢? 





