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

## 单元/接口/集成测试填坑 

我们把时钟拨回[sharding-jdbc](http://www.slahser.com/2016/06/25/当当的sharding-jdbc源码解读/)那天,我感慨了一下人家测试写得好. 

那么就拿这个项目做单元测试和集成测试的例子. 

主要思路就是拿Test Suite串起来所有的UT. 

单元测试一般都是BDD的思路,期待出现期待值,那么其中复杂对象或者异步操作就需要Mock. 

这部分选型我更倾向于`TestNG+PowerMock+AssertJ`. 

> 不好用testNG实现的就用junit也没关系,反正后者也可以用前者来执行. 
>
> mock部分同理. 
  
![2016-07-08_Screen Shot 2016-07-08 at 23.34.49.png](https://o4dyfn0ef.qnssl.com/image/2016-07-08_Screen%20Shot%202016-07-08%20at%2023.34.49.png?imageView2/2/h/600) 

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

那么我们可以把这部分的接口测试也接入到上一节的UT中. 

## Travis CI/Circle CI 


 


