![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-d257631f59976b1fa0845c5aac413147.png?imageView2/2/h/200) 

很久以前出于如此这般的原因,我的博客是架设在Github Pages上的. 

直到...  

直到今晚一时兴起,把项目的pushurl设置成了国内国外两个仓库,同步更新两个仓库进行渲染. 

```
vim yourproject/.git/config
//修改成如下形式,push时指向两个远程库
[remote "origin"]
    url = git@github.com:Slahser/slahser.github.io.git
    fetch = +refs/heads/*:refs/remotes/origin/*
    pushurl = git@git.coding.net:slahser/slahser.git
    pushurl = git@github.com:Slahser/slahser.github.io.git
``` 

同时修改了域名解析,将一级域名指向了Coding.net. 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-Screen%20Shot%202016-09-13%20at%2023.08.20.png?imageView2/2/h/300) 

> 更新:这个解析后来又有修改,因为github pages不太推荐CNAME为二级域名...而Coding.net支持..所以调换一个位置. 

那么最新的DIG: 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-Screen%20Shot%202016-09-13%20at%2023.08.50.png?imageView2/2/h/400) 

最新的测速也是一片绿,配合预加载页面全部秒开,爽:  

![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-Screen%20Shot%202016-09-13%20at%2023.11.30.png?imageView2/2/h/400) 

当然了,初心不变,两个库的Activity图是一起更新的. 

另外的好处是可以被Baidu Spider爬到了.  

> 反代或者DaoCloud这种暂时不考虑,写博客就是要简单便捷才能坚持下去,不想再折腾了. 

done. 



