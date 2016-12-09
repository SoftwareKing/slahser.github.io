![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2015.00.30.png?imageView2/2/h/200)

很久很久以前我们写了[为树莓派加上web terminal](https://www.slahser.com/2016/03/23/为树莓派加上web-terminal/) 

这个[teleport项目](https://github.com/gravitational/teleport)提供了新的SSH管理思路.

- 带有webui且保持*同一个session*
- 鉴权与OpenID与OAuth与两步验证
- 支持k8s形式的labels筛选
- session回放

官网首页就有他们的[宣传视频](https://youtu.be/bprRpX-4R_0),看起来还是很诱人很适合集群与团队管理.  

![](https://o4dyfn0ef.qnssl.com/image/2016-09-29-Screen%20Shot%202016-09-29%20at%2015.06.25.png?imageView2/2/h/400) 

好东西. 

但有个问题,它配置起来太麻烦了,而且不适合配置太低的机器,后台运行的5个应用,不断发送心跳包...让低配机器捉襟见肘.  

所以这篇博客流产了,我真后悔把标题起出来. 

> 或许等我手头不是pi cluster而是银河cluster时候再更新本篇吧. 

done. 


