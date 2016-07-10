其实这篇文章本来我是打算只贴一张SQL小抄儿的...后来觉得写了一点点心里有愧,对不起Google进来的父老乡亲. 

## 原本的内容 

其实我觉得SQL跟正则似的...你懂得. 

听说有人有哲学思路理解正则...等改天去见识一下.  

![2016-07-09_RebelLabs-SQL-cheat-sheet.png](https://o4dyfn0ef.qnssl.com/image/2016-07-09_RebelLabs-SQL-cheat-sheet.png) 


## 附加的内容 

ZeroTurnaround家的东西首页上写的清清楚楚: 

- JRebel - Reload Code Changes Instantly
- XRebel - Profiler for Java web applications
- Optimizer - IDE性能优化 

> 第三个实际用处实在太小..按着我之前写的调比这个科学.  

### JRebel 

> 原来一直支持Docker..这科技真是黑又硬.. 

- [本机测试](http://zeroturnaround.com/software/jrebel/quickstart/intellij/#!/server-configuration)
- [maven plugin](http://manuals.zeroturnaround.com/jrebel/standalone/maven.html)
- [remote server](http://manuals.zeroturnaround.com/jrebel/remoteserver/intellij.html) 
- [单独使用](http://zeroturnaround.com/software/jrebel/quickstart/standalone/#!/server-configuration)

> maven plugin是同事告诉我这么用的..只不过他的配置不太科学,嘻嘻. 

 
### XRebel 

用的人不多,主要因为付费.. 

虽然自己写Metrics也能实现.但开发阶段我会选择用这个,免得还要上VisualVM/JProfiler去看性能问题.  

- 能监控指标与告警
- web调用栈在页面中就看得到
- 链路与中间件调用耗时展示
- 支持微服务调用链

所以有什么理由不用一下呢. 

###  热部署与ASM 

我是参考的这篇[方法体热部署](http://www.ibm.com/developerworks/cn/java/j-lo-hotdeploy/)与这篇[服务器应用热部署](http://blog.csdn.net/chenjie19891104/article/details/42807959) 

> 说到上文为什么我不太用JRebel,因为修改多了他不准. 

####  方法体热部署 

思路: 

自定义CLassLoader来加载特定的类,实现一些加载的策略. 

而面对双亲委派机制,就那么个classpath,抢在上层ClassLoader前加载也不可能. 

那么换个方向,系统实例化某对象时,使其指向自定义loader加载出来的带版本的类. 

那么工作就变成了文中的: 

- 自定义ClassLoader - 加载需要监听改变的类.在.class文件发生改变的时候,重新加载该类,并增加版本. 
- 改变创建对象的行为 - 使他们在创建时使用自定义ClassLoader加载的带版本的Class. 

之所以说带版本是因为同一个名字的Class没法被自定义ClassLoader加载两次.. 

> 下面的部分是我第一次接触到ASM代码怎么写.以前没有应用场景. 

```java
ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS); 
ClassReader cr = null;     
String enhancedClassName = classSource.getEnhancedName(); 
try { 
    cr = new ClassReader(new FileInputStream(classSource.getFile())); 
} catch (IOException e) { 
    e.printStackTrace(); 
    return null; 
} 
ClassVisitor cv = new EnhancedModifier(cw, 
        className.replace(".", "/"), 
        enhancedClassName.replace(".", "/")); 
cr.accept(cv, 0);
``` 

> ASM 修改字节码文件的流程是一个责任链模式,首先使用一个 ClassReader 读入字节码,然后利用 ClassVisitor 做个性化的修改,最后利用 ClassWriter 输出修改后的字节码. 

所以自定义ClassLoader的主要功能实现: 

1. 将原始类转变为没有实现的接口. 
2. 将之前的内容复制到派生类中,以后每次更新都实现一个新的.
3. ClassWriter写出的时候将带版本的className写出
4. 定时检查.class文件是否有改变

因为原始类已经变成了接口,新的实现类还没有被感知.所以对应 

- new Instance()
- Class.forName() & 反射

实例化对象要有不同的实现. 

- 前者要通过ASM修改字节码,将所有new的部分替换成自定义ClassLoader寻找并实例化的形式. 
- 后者同理通过hack Class.forName() 即可. 

#### 服务器应用热部署 

实现思路跟上面都是一样的.只是主体换成了目录. 

其中监控文件变化用的VFS的DefaultFileMonitor,监听器实现FileLisetner回调设成reloadApp(). 

while(true)启动起来,将上一个app生命周期结束,启动新的即可. 

done. 

 






