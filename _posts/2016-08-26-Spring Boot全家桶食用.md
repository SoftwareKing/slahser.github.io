![2016-08-29_Screen Shot 2016-08-29 at 19.45.33.png](https://o4dyfn0ef.qnssl.com/image/2016-08-29_Screen%20Shot%202016-08-29%20at%2019.45.33.png?imageView2/2/h/400) 

在[这一篇](http://www.slahser.com/2016/06/24/框架系列-他们怎么用开发框架/)的末尾说到了Spring Boot全家桶的事儿,太监了好久今儿补上. 

内容涉及: 

- 多Module的Spring Boot项目搭建
- 配置方式变更
- 技术栈引入
- 打包部署 with docker
- 健康检查与指标度量
- 测试
- 文档生成 with swagger 
- 微服务

> spring boot版本1.4.0,有一些地方跟之前的版本不太一样. 

> BUG: spring-boot-dev-tools现在是自带bug,不建议引入. 

那我做了一次系统重设计的分享,具体的Slide在[这里](http://slides.com/slahser/nirvana/fullscreen). 

## 多Module的Spring Boot项目搭建 

正常的部分没有区别,parent啊,modules啊,relativePath啊,managementa啊都照常. 

打包的部分随着module数增多就开始不对劲起来,本来期待`mvn spring-boot:repackage`来生成fat jar,但后来只能`mvnpa`来完成打包. 

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>xxx</mainClass>
        <layout>WAR</layout>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 配置方式变更 

这部分就是随便找个Spring Changelog就可以了解了. 

主要是把所有xml文件平移进了java Config.我认为Condition Annotation算是很显著的好处,另外避免大家乱抄配置. 

## 技术栈引入 

主要体现在各种Starter上面,目前用到的有: 

```xml
<!--spring boot 基本-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-velocity</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<!--<dependency>-->
    <!--<groupId>org.springframework.boot</groupId>-->
    <!--<artifactId>spring-boot-devtools</artifactId>-->
    <!--<optional>true</optional>-->
<!--</dependency>-->
<!--hibernate validator-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<!--websocket-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!--ssh登录-->
<!--<dependency>-->
    <!--<groupId>org.springframework.boot</groupId>-->
    <!--<artifactId>spring-boot-starter-remote-shell</artifactId>-->
<!--</dependency>-->
<!--可视化监控-->
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>${boot.admin.version}</version>
</dependency>
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
    <version>1.3.4</version>
</dependency>
<!--文档生成-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>${swagger.version}</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>${swagger.version}</version>
    <scope>compile</scope>
</dependency>
<!--Spring session-->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
``` 

其中喜闻乐见的遇到了dev-tools与data-redis的bug,故而弃用.我就说我不爱用热部署的玩意... 

加入了以上starter之后,参考[官方](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)最下方的配置模板进行客制化配置,即可迅速的在上下问中使用各种技术栈了. 

当然,如果不喜欢你可以自定义`factoryBean`啊,`xxTempalte`啊之类的进行覆盖. 

比如这样: 

```java
@Configuration
public class RedisConfig {
    
    @Bean(name = "fastJsonJsonRedisSerializer")
    public RedisSerializer getFastJsonJsonRedisSerializer() {
        return new FastJsonJsonRedisSerializer<Object>(Object.class);
    }
    
    @Bean(name = "redisTemplate")
    public RedisTemplate<String, String> getRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(getFastJsonJsonRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(getFastJsonJsonRedisSerializer());
        template.afterPropertiesSet();
        return template;
    }
}
``` 

> 虽然我知道fastJson有坑,但是同事比较习惯这个.. 

## 打包部署 

几种方式: 

- 生成fat jar,利用生成的脚本注册成linux服务
- 做成带有jdk的docker镜像
- 生成war包置入传统servlet容器

喜欢什么用什么. 

## 健康检查与指标度量 

程序启动后可供访问的常用部分有: 

- beans
- dump
- health
- metrics  
- mappings
- *shutdown 

顾名思义,我就不写注释了,当然我们可以通过post shutdown进行服务摘除,比较方便. 

自定义以上health/metrics也比较简易,参照文档即可,主要思路是把上下文中关注部分注入,来进行参考. 

![2016-08-29_Screen_Shot_2016-08-21_at_14.36.15.png](https://o4dyfn0ef.qnssl.com/image/2016-08-29_Screen_Shot_2016-08-21_at_14.36.15.png?imageView2/2/h/400) 

更有累加器与程序耗时统计服务供使用,比如: 

```java
@Aspect
@Component
public class ServiceMonitor {

    @Autowired
    private CounterService counterService;

    @Autowired
    private GaugeService gaugeService;

    @Before("execution(* com.xxx.*.*(..))")
    public void countServiceInvoke(JoinPoint joinPoint) {
        counterService.increment(joinPoint.getSignature() + "");
    }

    @Around("execution(* com.xxx.*.*(..))")
    public void latencyService(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        pjp.proceed();
        long end = System.currentTimeMillis();
        gaugeService.submit(pjp.getSignature().toString(), end - start);
    }
}
``` 

> 那么以上的部分想要可视化也可以通过搜索"Spring Boot Admin"或者联系我来获取更多,它提供了上述部分的可视化与jmx相关的更好支持. 

![2016-08-29_Screen_Shot_2016-08-21_at_14.36.15.png](https://o4dyfn0ef.qnssl.com/image/2016-08-29_Screen_Shot_2016-08-21_at_14.36.15.png?imageView2/2/h/400) 

## 测试 

参考之前的`sharding-jdbc`终于搞了一次比较满意的test base. 

主要思路:利用JUnit test Suite串起所有的用例,在一次集成测试中跑完. 

以下是一份简单的示例,现实朋友可以微信联系我,一起聊聊细节. 

```java
@RunWith(Suite.class)
@Suite.SuiteClasses({
        AllControllerTests.class,
        AllServiceTests.class
})
public class AllAppTests {
}
```  

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TtServiceTest {
    @MockBean
    private CardInfoMapper mapper;
    @Autowired
    private TtService ttService;

    @Test
    public void getListTest() {
        // RemoteService has been injected into the reverser bean
        given(this.mapper.selectAll()).willReturn(Lists.<CardInfo>newArrayList());
        List list = ttService.getPageList();
        assertThat(list.size()).isEqualTo(0);
    }
}
```

> 以上程序是Spring Boot1.4的新写法. 之前的版本可能不太兼容,因为官方也在不断的更新复合注解.. 

> 覆盖率的生成有传统的Cobertura,那这部分我还没实践,也就先不写出来了. 

## 文档生成 

虽然这次文档生成使用的也是`swagger-ui`,但是同事们比较惊喜的还是这部分 

我的原话是"可以试想测试环境下,系统间数据交互,你又不想写文档..." 

对吧,那么可以把自动生成的,可供测试的API文档地址提供给对方. 

而我们的开发成本仅仅是写Controller时顺手写个注解. 

引入方式: 

1. 引入上面的代码生成依赖
2. 添加如下配置
3. 在controller方法上增加@ApiOperation
4. done. 

```java 
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.xxx.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("xxx项目API文档")
                .description("来自xxx")
                .termsOfServiceUrl("http://localhost:8080/")
                .version("1.0")
                .build();
    }
}
```  

效果真是好的过分. 

![image_2016-08-29_Screen_Shot_2016-08-22_at_00-1.05.17.png](https://o4dyfn0ef.qnssl.com/image/image_2016-08-29_Screen_Shot_2016-08-22_at_00-1.05.17.png?imageView2/2/h/400)

![2016-08-29_Screen_Shot_2016-08-22_at_00.05.41.png](https://o4dyfn0ef.qnssl.com/image/2016-08-29_Screen_Shot_2016-08-22_at_00.05.41.png?imageView2/2/h/400) 

## 微服务 

说是全家桶其实这部分还是值得斟酌,一是业务没有多到需要切分,二是选型的问题. 

以小公司来讲,生产环境上的docker编排,hold得住么? 

只是留下了Slide上的一些选型空间放在那儿思考. 

日后有实践也会来进行分享. 

done. 


