![](https://o4dyfn0ef.qnssl.com/image/2017-05-14-Screen%20Shot%202017-05-08%20at%2015.10.40.png?imageView2/2/h/400) 

16年9月我们有[基于Gitlab与Docker的CI](https://www.slahser.com/2016/09/07/基于Gitlab与Docker的CI/),并且在17年3月我更新了一版. 

最近优化了一下流程,并且偶尔的折腾中,有一些新的点. 

以下组件并不推荐放在 docker 中运行: 

- gitlab
- jira
- confluence
- sonarqube 

没错,基本上是所有. 

> 当然了,官方文档在[这里](https://docs.gitlab.com/ce/ci/yaml/README.html). 

- - - - -- 

### 安装Sonarqube
 

依照官网[文档](https://docs.sonarqube.org/display/SONAR/Installing+the+Server)进行安装 

调解下字符集,有条件的话最好开启 https. 

管控不赘述,插件安装的话主要是几个你喜欢的: 

- 语言包
- 系统接入
- checkstyle

- - - - -- 

### 插件 

插件下载失败的话可以手动去去 github 下载 release jar 放入安装目录 plugin 下. 

重启即可生效,另外最新6.3暂时没有中文包,可以降级使用6.2. 

以下这几个我装了: 

- [sonar-checkstyle](https://github.com/checkstyle/sonar-checkstyle)
- [sonar-gitlab-plugin](https://github.com/gabrie-allaigre/sonar-gitlab-plugin)
- [sonar-auth-gitlab-plugin](https://github.com/gabrie-allaigre/sonar-auth-gitlab-plugin)

前两个来支持本文,第三个需要 sonarqube 是 https 访问的. 

- - - - -- 

### 配置 

于是最新的 gitlab-ci.yml 

```
image: maven:3.5.0-jdk-8-alpine
before_script:
  - mvn clean
stages:
  - install
  - test
  - analysis
mvn_install:
  script:
    - mvn install -Dmaven.test.skip=true
  stage: install
  only:
    - develop
    - master
    - ^hotfix\/.+$
    - ^release\/.+$
unit_test:
  script:
    - mvn test
  stage: test
  only:
    - develop
    - master
    - ^hotfix\/.+$
    - ^release\/.+$
integration_test:
  script:
    - mvn integration-test
  stage: test
  only:
    - develop
    - master
    - ^hotfix\/.+$
    - ^release\/.+$
# 'External is fail because you have critical or blocker new issue.'
# 还是得开启,否则每次提交校验就没意义了
sonarqube_preview:
  script:
    - git config --global user.email "sonar@gogen.com.cn"
    - git config --global user.name "Sonar"
    - git checkout origin/master
    - git merge $CI_COMMIT_SHA --no-commit --no-ff
    - mvn --batch-mode verify sonar:sonar -Dsonar.host.url=$SONAR_URL -Dsonar.analysis.mode=preview -Dsonar.gitlab.project_id=$CI_PROJECT_PATH -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHA -Dsonar.gitlab.ref_name=$CI_COMMIT_REF_NAME
  stage: analysis
  except:
    - develop
    - master
    - ^hotfix\/.+$
    - ^release\/.+$
sonarqube:
  script:
    - mvn --batch-mode verify sonar:sonar -Dsonar.host.url=$SONAR_URL
  stage: analysis
  only:
    - develop
    - master
    - ^hotfix\/.+$
    - ^release\/.+$
after_script:
  - mvn clean
```

另外: 

```
项目-> Setting -> General -> 记录ID
项目-> Setting -> CI/CD pipeline -> Secret Variables -> 设置如下: 第一个填ID,第二个填Sonar地址
``` 

即可达成题图效果. 

也就是说: 

- 功能分支: 每次提交都会触发Sonar,并且本次提交剩余的问题状态的问题会自动写在你本次提交的Comments中
- 关键分支: 提交会将报表汇报至Sonar. 

- - - - --

done. 
