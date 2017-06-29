![2016-06-26_Screen Shot 2016-06-26 at 18.17.38.png](https://o4dyfn0ef.qnssl.com/image/2016-06-26_Screen%20Shot%202016-06-26%20at%2018.17.38.png?imageView2/2/h/200) 

今天我在写一段 Vertx 程序的时候发现 eventloop怎么都起不来 

提示的是`java.net.Inet6AddressImpl.lookupAllHostAddr`相关的堆栈 

耗时几千毫秒,导致框架认为我在阻塞主流程. 

于是我开始了如下的自责: 

- 目前用得是外置的千兆网卡,换成自带驱动的 wifi 呢?
- ipv6万恶之源关掉呢?
- vmoption 加上`-Djava.net.preferIPv4Stack=true`呢?
- 自己手动 dig 一下自己呢?
- 程序里写死某个网卡的ip 呢?

当然都不奏效...压根就没想到是 macOS Sierra 的问题.

> 我早早已经上了 high Sierra. 

直到我不甘心上 SO 换了又换关键字 

- [jvm-takes-a-long-time-to-resolve-ip-address-for-localhost](https://stackoverflow.com/questions/39636792/jvm-takes-a-long-time-to-resolve-ip-address-for-localhost/39698914#39698914)
- [macos-sierra-java](https://thoeni.io/post/macos-sierra-java/)

另外友情提示的是: 

mac 上的 hostname 不一定跟你在 sharing 里设置的名字一样. 

我的mac 几经 timemachine 恢复之后,我发现 hostname 居然来自我几年前的老电脑. 

> 按年代标记就是优越.. 

- - - - --- 

done. 
