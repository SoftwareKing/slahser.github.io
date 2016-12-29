
>![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-fasdkfgasuriaubrt.jpeg?imageView2/2/h/200)

本文几经修改.. 

改到如今已经没多少原创内容了.总结下来还是Github上大项目开发这一套流程比较科学. 

> 我有印象的关于分支管理就开过两三次会,无比痛苦.  

todo. 

![](https://o4dyfn0ef.qnssl.com/image/2016-12-26-IMG_0325.JPG?imageView2/2/h/600) 

> 请忽略我写错的单词... 

CI文件依然是:  

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
ut:
  script:
    - mvn test
  stage: ut
  only:
    - develop
    - master
it:
  script:
    - mvn integration-test
  stage: it
  only:
    - develop
    - master
after_script:
  - mvn clean
```

done. 


