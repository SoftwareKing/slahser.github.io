
>![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-fasdkfgasuriaubrt.jpeg?imageView2/2/h/200)

本文致力于适当的解决gitflow存在的一些问题. 

总之gitflow算是跌跌撞撞的在推广中.在前东家没推起来算是我的遗憾之一. 

我们熟知的gitflow是下图这样的. 

单位里的各位貌似是没有test就没有安全感的样子. 

所以姑且多了一条test分支.起到一部分gitflow模型里release的作用.  

![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-Screen%20Shot%202016-12-26%20at%2016.00.45.png?imageView2/2/h/600) 

那么,简单的升级包迭代没问题. 

可是当两个长期开发的升级包任务并行开发的时候,我们这条develop就备受污染,没法保证干净. 

困扰了我个人很久,我看到知乎上有人时隔一年还是没找到答案.... 

直到我最近看了题图这本书. 

![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-IMG_0325.JPG?imageView2/2/h/600) 

所以在RD不习惯做功能开关的情况下,我们可以 

![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-Screen%20Shot%202016-12-26%20at%2016.05.47.png?imageView2/2/h/600)  

> 请忽略我写错的单词... 

同时CI文件变成如下: 

```
image: maven:3.3.9-jdk-8
before_script:
  - mvn clean
stages:
  - install
  - ut
  - it
install:
  script:
    - mvn install -Dmaven.test.skip=true
  stage: install
  only:
    - develop
    - master
    - test
ut:
  script:
    - mvn test
  stage: ut
  only:
    - develop
    - master
    - test
it:
  script:
    - mvn integration-test
  stage: it
  only:
    - develop
    - master
    - test
after_script:
  - mvn clean
```

done. 


