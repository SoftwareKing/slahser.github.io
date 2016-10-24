记录一下解决各路问题的过程: 

> 一般是随手搜不到的. 

- - - - -- 

## 接入activiti配置 

在网上我还没有搜索到boot+mybatis+activiti的科学运行方式,我fix了一下: 

从boot自带的事务管理器配置里可以看到默认的PlatformTransactionManager配置,文件位置`package org.springframework.boot.autoconfigure.jdbc` 

相关配置都在这里,那么拿jpa之外的数据源配置就如下: 

> activiti全家桶starter省略 

```java
import org.activiti.spring.SpringProcessEngineConfiguration;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;
import javax.sql.DataSource;

@Configuration
public class ActivitiConfig {
    @Autowired
    public PlatformTransactionManager transactionManager;
    @Autowired
    public DataSource druidDataSource;

    @Bean
    public SpringProcessEngineConfiguration getProcessEngineConfiguration() {
        SpringProcessEngineConfiguration engineConfiguration = new SpringProcessEngineConfiguration();
        engineConfiguration.setDataSource(druidDataSource);
        engineConfiguration.setTransactionManager(transactionManager);
        return engineConfiguration;
    }
}
``` 

> 事实情况是你尊重一下编辑器的提醒.逆变一下把类型写的够窄就不会有红线提示找不到bean了. 

另外就是starter可供配置的部分在[这里](https://github.com/Activiti/Activiti/blob/master/modules/activiti-spring-boot/spring-boot-starters/activiti-spring-boot-starter-basic/src/main/java/org/activiti/spring/boot/ActivitiProperties.java),按照规范开发的starter大抵都是这种位置.  

所以官方给的模板配置并不是全部,对starter感兴趣可以自己去翻一下.  

- - - - -- 

## 类型转换错误 

上述项目我们引入了Mapper3与dev-tools,这导致每次启动线程可以很显著的看到线程名带着restart字样. 

而后使用selectOne()方法时出现ClassCast相关错误,一定是classloader不同导致,查阅issue发现dev-tools有bug,弃用解决. 

> 你说它在selectAll()时候就不重现..懒得看了

- - - - -- 

##  Test Suite与多Module 

这个我没有解决. 目前只能分着module跑. 

后续会把Gitlab CI配一下持续集成起来. 

- - - - -- 

## activiti rest的安全设置 

因为历史遗留,权限验证目前项目里是shiro. 

查看了依赖树activiti res依赖了spring security并触发了在activiti basic starter中的Condition. 

这就导致多了一层乱七八糟的权限验证. 

翻了一下官方文档,尽管rest starter已经释出,但是文档完全没有更新,rest访问还是依赖@RestController. 

于是尝试autoConfig里面exclude掉相关设置,结果问题重重,官方forum里回答也都是含糊其辞,失望... 

那解决办法: 

- 弃用starter,使用activiti-spring方式继续使用
- 弃用activiti rest,按照官方使用@RestController 来使用. 

诸君自取. 

- - - - -- 

## @Value注入属性 

最近听到不少反馈说运行时实例化的bean直接@Value拿不到值. 

之前测试了在Configuration里面直接这么做是不行的,需要放到参数中直接来初始化bean属性. 

这个暂时挂在这里,改天验证下. 

- - - - --- 

## 插件的repackage 

多module下的这段儿我一直没成功过,生成可执行jar也是用`<configuration>里的<executable>`搞定. 

```
<execution>
    <goals>
        <goal>repackage</goal>
    </goals>
    <configuration>
        <classifier>exec</classifier>
    </configuration>
</execution>
```

> 另外如果你用了surfire和failsafe的话,那么这个插件的pre-integration-test这一对儿命令就是个笑话. 

- - - - --- 

## 文件上传的一些 

这个不是spring boot的坏味道了,文件上传时如果同时传递json数据与multipart到mvc 

那么`Content-Type:Mixed`,同时由于ajax生成的bondary切割字段应该是不带引号的形式,出处来自RFC的标准. 

- - - - --- 





