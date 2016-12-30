
>![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-fasdkfgasuriaubrt.jpeg?imageView2/2/h/200)

本文几经修改.. 

改到如今已经没多少原创内容了.总结下来还是Github上大项目开发这一套流程比较科学. 

> 我有印象的关于分支管理就开过两三次会,无比痛苦.  

![](https://o4dyfn0ef.qnssl.com/image/2016-12-30-Screen%20Shot%202016-12-30%20at%2015.34.29.png) 

以下部分摘自我在内网Confluence上的内容 

```
新功能的开发RD需要在develop基础上checkout出feature/xxx形式的新分支,直至功能开发结束,不要向develop上合并代码.

> 关系耦合严重的两个RD请共用同一条分支

提测后由QA在该feature分支基础上,创建release-v***分支,并在该release分支上进行测试

> 届时请修改CI文件,将最新的release分支也纳入CI范围 

测试结束后,将该release分支merge到master,并以v***作为标识打出tag

同时将tag打出后的master merge到develop上,完成一次功能的开发.

最后删除残留的feature/xxx分支.
``` 

CI文件修改部分 

```
it:
  script:
    - mvn integration-test
  stage: it
  only:
    - develop
    - master
    - release-v0.1
```

- - - - -- 

### 另一些 

我前些日子在读书的时候看到这么一段儿. 

![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-IMG_0325.JPG?imageView2/2/h/600) 

> 请忽略我写错的单词... 

实际如果开发情况没有复杂到`多功能同时开发,开发完的功能不一定发版`的话 

可以使用文中的思路进行另一层抽象 

你可以把主功能一直维护在develop上,周围的feature分支由一部分人进行开发,可以随时merge到develop上. 

而后另外一个功能需求入场,你可以把这个需求当做一次hotfix来进行处理.

从上一次tag出checkout出代码,保证了功能间没有干扰. 

那么... 

对吧. 

- - - - -- 

done. 


