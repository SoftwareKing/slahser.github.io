![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-d257631f59976b1fa0845c5aac413147.png?imageView2/2/h/200) 

很久以前出于对Github Avtivity图的虚荣感,我的博客是架设在Github Pages上的. 

直到...  

直到今晚一时兴起,把项目的pushUrl设置成了国内国外两个仓库,同步更新两个仓库进行渲染. 

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

那么最新的DIG: 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-Screen%20Shot%202016-09-13%20at%2023.08.50.png?imageView2/2/h/400) 

最新的测速也是一片绿,页面全部秒开,爽:  

![](https://o4dyfn0ef.qnssl.com/image/2016-09-13-Screen%20Shot%202016-09-13%20at%2023.11.30.png?imageView2/2/h/400) 

当然了,初心不变,两个库的ctivity图是一起更新的. 

另外的好处是可以被Baidu Spider爬到了,也不知道算是好还是坏. 

done. 



