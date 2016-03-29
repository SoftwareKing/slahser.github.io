>这本书过于基础,不适宜购买

1.通过类型擦除来保证以下代码的兼容性: 

```
List someThings = getSomething();
List<String> strings = (List<String>) someThings;
``` 

2.协变与逆变 

臭名昭著的`<Scala编程>`中译本彻底把这段讲到让人困惑. 

```
List<?> objects =new ArrayList<String>();//让容器类型具有父子关系
List<? extends Object> objects =new ArrayList<String>();//协变
List<? super String> objects =new ArrayList<Object>();//逆变
```
俗话说得好 

> Producer Extends ,Consumer Super.

把方法视为一等公民时,则一个方法的定义同时具有生产者与消费者的身份 

- 面向输出生产->使用extends->返回值协变->向上转型
- 面向输入消费->使用super->参数逆变->方法重载

反映到泛型里,恰似`<java 核心编程>`里的这一句 

```
public static <T extends Comparable<? super T>> T min(T[] a)
``` 

3.GC算法与VisualVM使用 

我们都知道常用GC root有几下几种： 

- Class - 由system classloader加载的对象，这些类是不能够被回收的，他们以静态字段的方式持有其它对象。  

    - 通过自定义的类加载器加载的类，除非相应的java.lang.Class实例以其它的某种方式成为roots，否则它们并不是roots.
- Thread - 活着的线程
- Stack Local - Java方法的local变量或参数
- Monitor Used - 用于同步的monitor对象






