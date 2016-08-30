记录一下解决各路问题的过程: 

> 一般是随手搜不到的. 

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

## 类型转换错误 

上述项目我们引入了Mapper3与dev-tools,这导致每次启动线程可以很显著的看到线程名带着restart字样. 

而后使用selectOne()方法时出现ClassCast相关错误,一定是classloader不同导致,查阅issue发现dev-tools有bug,弃用解决. 

##  Test Suite与多Module 

这个我没有解决. 目前只能分着module跑. 

后续会把Gitlab CI配一下持续集成起来. 




