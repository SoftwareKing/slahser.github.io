依然贯彻[新浪Motan那篇](http://www.slahser.com/2016/05/28/新浪的RPC框架Motan源码解读/)的风格来搞. 

项目地址:[sharding-jdbc](https://github.com/dangdangdotcom/sharding-jdbc) 

> 初见源码突然觉得自己之前写的集成测试不太好,Lombok用的也比我们单位统一多了...学一手. 

![2016-06-27_architecture.png](https://o4dyfn0ef.qnssl.com/image/2016-06-27_architecture.png?imageView2/2/h/300) 

实现来讲我个人觉得很难,起码难度要比其他协议层的实现难得多. 

## 协议重写 

既然是jdbc层的协议,那么从jdbc包看肯定没问题. 

![2016-07-03_Screen Shot 2016-07-03 at 22.31.04.png](https://o4dyfn0ef.qnssl.com/image/2016-07-03_Screen%20Shot%202016-07-03%20at%2022.31.04.png?imageView2/2/h/300) 

传统的组件:DataSource/Connection/Statement.各自都继承了抽象Wrapper来记录日志. 

`MasterSlaveDataSource`是为了支持后续更新的读写分离功能. 

那么既然是Wrap一定要具有原对象的所有必要方法,Getter系列,execute系列就不举例了. 

这样我们顺滑的实现实例就能接入Spring了,没什么太大的成本. 

点对点直连数据库,也没有Proxy层面机器挂掉的风险. 

> 写到这儿我必须吐槽一下,这个项目的源码读起来一点也不舒服,完全没有Motan的顺畅. 

比较核心的上下文是这样的: 

```java 
@RequiredArgsConstructor
@Getter
public final class ShardingContext {
    //约定的分库分表配置形式,包含配置解析,shard策略,别名绑定等内容
    private final ShardingRule shardingRule; 
    private final SQLRouteEngine sqlRouteEngine;
    //类似Fork/Join思想的异步线程池. 
    private final ExecutorEngine executorEngine;
}
``` 

适当的注释了一下. 

那么`SQLRouteEngine`内含的内容就是下一节的部分了,内含着SQL路由与解析相关的内容. 

## SQL解析/路由 

```java 
public SQLRouteResult route(final String logicSql, final List<Object> parameters) throws SQLParserException {
    return routeSQL(parseSQL(logicSql, parameters));
}
``` 

接收到一条原始SQL第一步就是进行解析,市面上我了解的解析引擎有`JSqlParser`和`Druid`,sharding-jdbc用的是后者. 

![2016-07-03_parse.png](https://o4dyfn0ef.qnssl.com/image/2016-07-03_parse.png?imageView2/2/h/300) 

根据传入的配置信息与原始SQL,获得对应的Parser,将语句归纳为增删改查的SQLStatement. 

其中Druid的SQLASTOutputVisitor使用SqlStatement.accept()来访问AST生成SQLObject,遍历过程中将条件对象维护在ParseContext存入ParsedResult. 

同时将解析结果相关部分塞入路由Context. 

![2016-07-03_route.png](https://o4dyfn0ef.qnssl.com/image/2016-07-03_route.png?imageView2/2/h/300) 

那么承接上一节的部分就在这里了. 

路由的部分将SQL与DataSource绑定在执行单元中,完成依照配置分库分表的核心功能. 

```java
private SQLRouteResult routeSQL(final SQLParsedResult parsedResult) {
    SQLRouteResult result = new SQLRouteResult(parsedResult.getRouteContext().getSqlStatementType(), parsedResult.getMergeContext());
    for (ConditionContext each : parsedResult.getConditionContexts()) {
        result.getExecutionUnits().addAll(routeSQL(each, Sets.newLinkedHashSet(Collections2.transform(parsedResult.getRouteContext().getTables(), new Function<Table, String>() {
            @Override
            public String apply(final Table input) {
                return input.getName();
            }
        })), parsedResult.getRouteContext().getSqlBuilder(), parsedResult.getRouteContext().getSqlStatementType()));
    }
    return result;
}
```  

获取执行单元的部分则是依照策略来进行,这部分解释起来不那么容易,因为我也没怎么看~. 

> 这里照着使用文档看看实际就能理解了.至于为什么我没写出来,谁知道呢. 

## SQL执行/结果归并 

![2016-07-03_execute.png](https://o4dyfn0ef.qnssl.com/image/2016-07-03_execute.png?imageView2/2/h/300) 

SQL语句统一归纳成执行单元去并发执行,涉及到事务的话就把执行单元发布到EventBus上,并注册一些Listener在上面. 

```java
@Getter
@Slf4j
@ToString
@EqualsAndHashCode
public class SQLExecutionUnit {
    private final String dataSource;
    private final String sql;
}
``` 

执行单元是如上对象的一层包装,划分为DML与DQL,最终扔到一个优化版的线程池`ExecutorEngine`,基于`ListeningExecutorService`. 

执行结束后得到ResultSet列表传入merge package进行结果归并. 

![2016-07-03_merge.png](https://o4dyfn0ef.qnssl.com/image/2016-07-03_merge.png?imageView2/2/h/300) 

如上图,归并中主要的功能是对一些内在函数(排序/分组/聚合函数)的分片支持. 

> 其中没有支持的部分就只能代码实现或者多个函数叠加起来,比如来个range内随机数.    

调用的入口在最开始的`ShardingStatement`中.静态Statement或者预编译Statement都是在这里.  

而MergeContext是随着上面解析部分过程中进行丰满,将结果归并配置留存到执行结束. 

具体实现就不说了,量很小,跟着上图到merger package走一走就可以了. 

## etc 

不知道是最近有其他事导致太监了3天,还是心态有点乱..这篇写的不是很满意,读的也不是很细.日后找个机会要重新补一下. 

done. 







