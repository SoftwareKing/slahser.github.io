![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-Unknown.png)
![](https://o4dyfn0ef.qnssl.com/image/2017-05-31-reading_title.png?imageView2/2/h/300) 
todo 

写了个工程Spring Cloud全家桶. 

截止目前为止,市面上写Spring Cloud的这几位基本上都是没上生产拿个demo开始搞事儿的.. 

工程相关的库冲突跟上生产路上遇到的问题我会慢慢总结. 

todo. 

> 这篇提交上来主要是不小心git push了一下...半成品都还不是,就开了头

下面这段儿是阿里云上的一篇文章,给的一段儿脚本. 

觉得挺好的摘抄过来一下. 

创建镜像

`export REGPREFIX=registry.aliyuncs.com/<username>/ docker tag $(docker build -t ${REGPREFIX}discovery-server -q .) ${REGPREFIX}discovery-server:$(date -ju "+%Y%m%d-%H%M%S")`

登录到镜像仓库，*** 用户名， *** 密码

`docker login -u *** -p *** registry.aliyuncs.com`

将创建好的镜像上传到镜像仓库，<username>替换为你的username

`export REGPREFIX=registry.aliyuncs.com/<username>/ docker push {REGPREFIX}discovery-server`
