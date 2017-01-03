![](https://o4dyfn0ef.qnssl.com/image/2016-09-07-Screen%20Shot%202016-09-07%20at%2015.49.25.png?imageView2/2/h/400) 

> 2016-09-08写就 2017-01-03重写  

- - - - -- 

## 从持续集成到持续交付 

在[也说说DevOps](https://www.slahser.com/2016/09/27/也说说DevOps/)中转载了这么一张图. 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-27-a7da8a5bfdc951862afc8f63f1650.png?imageView2/2/h/300) 

换成人话就是: 

- 持续集成 - 把代码变成代码包
- 持续交付 - 把代码包变成可执行包

这么短短的一步之差在我更新这篇文章之前,我们的部署方式会有这些顾虑: 

- 单位是不是有OpenStack
- 没有的话单机性能如何
- 单机多实例的话,日志/pid/编排的记录怎么做
- rsync的脚本,审批流程
- 累不累

- - - - -- 

## 后来我们怎么做的 

- Continuous Integration - Gitlab CI
- Continuous Delivery - webhook & docker registry
- Continuous Deploy - kubernetes

- - - - -- 

## 不言而喻的一些 

其实有了基础设施之后我们很多问题就不要考虑了. 

关于Kubernetes的一些可以看我的[首页](https://www.slahser.com),自己搜一下相关. 

在国内算是写的比较详细,版本很新的一批内容. 

我们省了哪些心呢?基本上上一小节所有的顾虑都不存在了. 

1. 从release-xxx到master上打tag - 来自[这里](https://www.slahser.com/2016/11/26/Enhance-Gitflow/)
2. 触发webhook执行`mvn clean install docker:build docker:push` - 来自[这里](https://www.slahser.com/2017/01/03/日后的测试代码要怎么写/)
3. 我们的docker私服搭建 - 来自[这里](https://www.slahser.com/2016/09/29/pi-cluster上配套Registry/) 
4. k8s部署应用与滚动更新 - 来自[这里(未更新)](https://www.slahser.com/2016/11/15/k8s后日谈-概念与设计/)
5. 从内网暴露服务到外部 - 来自[这里](https://www.slahser.com/2016/11/19/k8s后日谈-负载均衡traefik/)

日后的健康检查,日志收集,资源监控有的已经完成,有的还在进行中,敬请期待. 

- - - - - 

done. 






