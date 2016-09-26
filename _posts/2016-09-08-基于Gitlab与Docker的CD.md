![](https://o4dyfn0ef.qnssl.com/image/2016-09-07-Screen%20Shot%202016-09-07%20at%2015.49.25.png?imageView2/2/h/400) 

> 因为一些莫名其妙的原因单位的服务器80端口未开,所以本篇暂时是简易手法以及解决初创团队部署的思路分享 

> 这次不会太监,后续一定会补基于docker的持续交付

- - - - -- 

## 几条选择 

- 手撕脚本
- webhook触发监听应用自动部署
- ansible手动同步操作机器
- docker cd

- - - - -- 

## 手撕脚本 

之前的用的云平台在我眼里跟手撕脚本差不多了,进web terminal翻一翻密密麻麻的py交易.. 

觉得不够优雅,我也不太会写这东西. 

- - - - -- 

## webhook 

gitlab一直是有webhook这么个东西,我也一直惦念着,直到我写了这个程序.  

`http://yourgitlab/yourgroup/yourproject/hooks` 可以勾选什么时候触发这个hook. 

那么在git flow中我们在master上打tag,通过Tag Push Event触发hook. 

那么这个hook原理就是向指定地址发送POST请求,带有特定的Header属性来判断不同行为. 

那么验证`X-Gitlab-Token`和`X-Gitlab-Event`就ok了. 

同时另一方面需要编写&部署另一套应用来提供rest服务供hook调用,验证上面的Header属性,而后执行部署流程. 

两种套路: 

- `java -jar app.jar -Dxxx`记录运行pid写入文件.用来控制生命周期 
- exec jar注册成linux系统服务,`service app restart` 

详情可以看:

- [java -jar](http://www.cnblogs.com/vipsoft/p/5252306.html) 
- [注册服务](http://www.jianshu.com/p/44ef43b282f0) 

- - - - -- 

## ansible 

> mac上直接互通证书实现免登陆貌似不科学,还是直接写账号密码吧. 

这个相对来讲农耕一些,不过我觉得比上一种可操作性强一点. 

简单来说步骤: 

1. 同步信息
2. 下载release
3. 运行
4. 挂负载均衡(或者网关)

playbook各家都不同,这个就不细说了,目前公司我觉得适合这种方式. 

- - - - -- 

## docker cd 

todo. 







