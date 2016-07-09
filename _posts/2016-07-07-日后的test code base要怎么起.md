本篇算是心血来潮,规划一下日后公司里的测试代码应该怎么搞下去. 

目前觉得自己短短的职业生涯内呆的几个大的小的公司写的测试代码都不是我心里想要的那样,总结一点下班后的东西算是. 

本篇涉及的实践内容: 

- 单元/接口/集成测试怎么去填坑
- 数据驱动的接口测试
- Travis CI/Circle CI
- git flow简述
- App/UI自动化
- 从Mock Server到Mock平台
- 从测试到线上

优先级从上到下,有一部分我也在摸索,随缘更新,随时太监. 

## 单元/接口/集成测试填坑m 

我们把时钟拨回[sharding-jdbc](http://www.slahser.com/2016/06/25/当当的sharding-jdbc源码解读/)那天,我感慨了一下人家测试写得好. 

那么就拿这个项目做单元测试和集成测试的例子. 

主要思路就是拿Test Suite串起来所有的UT. 

单元测试一般都是BDD的思路,期待出现期待值,那么其中复杂对象或者异步操作就需要Mock. 

这部分选型我更倾向于`TestNG+PowerMock+AssertJ`. 

> 不好用testNG实现的就用junit也没关系,反正后者也可以用前者来执行. 
>
> mock部分同理. 
  
![2016-07-08_Screen Shot 2016-07-08 at 23.34.49.png](https://o4dyfn0ef.qnssl.com/image/2016-07-08_Screen%20Shot%202016-07-08%20at%2023.34.49.png?imageView2/2/h/500) 

几点: 

- 普通pojo也会测试scope与getter/setter.
- 异常情况会`@Test(expected = xxException.class)`
- 断言优化可以用AssertJ或者Hamcrest,这些静态导入没关系的. 
- Lombok这种ASM级别的库用用没关系的,别写的又臭又长. 
- 异步情况最好也要覆盖,PowerMock做得到


## 数据驱动的接口测试 

俗称DDD. 

自动化程度高一点我更喜欢csv做数据源做BDD的自动化. 

TestNG是自带`@DataProvider`的数据源配置的,可以自行实现数据源对象.  

今天的主角就是这方面的扩展[feed4testng](http://databene.org/feed4testng.html). 

一言概之: 

```java
@Test(dataProvider = "feeder")
@Source("xxxx.csv")
``` 

> 更浮夸一点的直接拿数据表做数据源它也实现了,自行看看咯 
>
> 再比如[这种](https://github.com/superproxy/test-data-provider)也都可以瞄一眼看看实现,自定义一些功能. 

那么我们可以把这部分的接口测试也接入到上一节的UT中. 

## Travis CI/Circle CI 

实际这两个持续集成功能上差距不多,我喜欢后者. 

大体思路是给你一个云虚机,写yml配置文件来 

- 初始化环境 - 设置环境变量,设置工作目录
- 执行task - mvn test或者gradle task等等
- 生成报告 - 默认或者plugin
- hook - 包括触发hook与报告hook

> 有时候就觉得有点穿越到Dockerfile.. 

关键点在于这个yml怎么写,反而trigger之类的我觉得不太重要,翻翻文档就有了.  

[这](https://circleci.com/docs/config-sample/)是官方的例子,替换一下,把

- dependency -> `mvn dependency:resolve`
- test -> `mvn test -Dtest=com.slahser.AllTests`

这样就很轻易的跟上两节的测试集成在一起了. 

> 详细的配置[这里](https://circleci.com/docs/configuration/)有

## git flow简述 

[这里](https://www.atlassian.com/git/tutorials/comparing-workflows/)有一篇工作流对比的文章,是Atlassian的. 

- Centralized Workflow - 当SVN用,遇到冲突扯出分支来解决再合回去. 
- Feature Branch Workflow - 小作坊,大家扯feature分支,交流怎么合. 
- Gitflow Workflow - 我比较喜欢,做了一些规范.但是无法解决一部分问题,后续会谈. 
- Forking Workflow - 提patch,github的主要交流手段. 

gitflow模型有很多插件,我手头已经安装的: 

- fish-gitflow - 提供`git flow`命令对应开发生命周期
- fish-completion - 提供了上述命令的提示功能
- idea插件
- SourceTree插件

选择安装即可. 

gitflow模型 

![2016-07-09_Screen Shot 2016-07-09 at 20.19.06.png](https://o4dyfn0ef.qnssl.com/image/2016-07-09_Screen%20Shot%202016-07-09%20at%2020.19.06.png?imageView2/2/h/300) 

前面提到的问题是: 

怎么才能优雅的解决多个release并行开发的问题呢?同时带来的问题是测试是不是应该在gitflow模型上拓展. 

没得到什么好的解决,gitflow我也在组里推过一次,不过没什么成效,大家喜欢小作坊就先这样吧. 

另外关于merge与rebase,他们还有[一篇质量很高的文章](https://www.atlassian.com/git/tutorials/merging-vs-rebasing/conceptual-overview). 

![2016-07-09_Screen Shot 2016-07-09 at 20.27.26.png](https://o4dyfn0ef.qnssl.com/image/2016-07-09_Screen%20Shot%202016-07-09%20at%2020.27.26.png?imageView2/2/h/300)

![2016-07-09_Screen Shot 2016-07-09 at 20.27.16.png](https://o4dyfn0ef.qnssl.com/image/2016-07-09_Screen%20Shot%202016-07-09%20at%2020.27.16.png?imageView2/2/h/300)

说白了,rebase只在本地分支用用就算了,优点是分支会很干净.突然的rebase就算不在master上也会导致被合并方莫名其妙. 

科学点的套路:将被合并方merge到合并方解决冲突,再merge回去.  

> 另外这里有官方的[github flow](https://guides.github.com/introduction/flow/)教程. 

## App/UI自动化 












 


