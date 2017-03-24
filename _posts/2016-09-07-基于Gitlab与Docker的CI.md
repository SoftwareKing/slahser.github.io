![](https://o4dyfn0ef.qnssl.com/image/2016-09-07-Screen%20Shot%202016-09-07%20at%2015.49.25.png?imageView2/2/h/400) 

> 2017-03-24更新,gitlab 9.0.0中这部分功能大家谨慎使用. 
> 
> 我回滚到了gitlab/gitlab-runner:v1.10.5才正常工作. 

最近需要实践一下Gitlab based的CI与CD 

- 持续集成在先,先把写好的模块化单元测试跑起来 
- 持续交付在后,通过PR生成镜像到私服 

> Gitlab 8.0以后的版本自带CI,不需要到Services里面去启用了. 

- - - - -- 

## 准备工作 

1. 安装Docker
2. 安装[Runner - Docker](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/install/docker.md). 

```
mkdir -p /srv/gitlab-runner/config
docker pull gitlab/gitlab-runner
```

## 运行Runner  

```sh
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner:Z \
  gitlab/gitlab-runner:latest
``` 

## 注册到gitlab 

```
docker exec -it gitlab-runner gitlab-runner register
``` 

- executor选docker来启用`docker-in-docker`模式 
- image写maven:3.3.9-jdk-8 

## 写yml脚本 

到`http://yourgitlab/ci/lint`检测自己的脚本是否科学. 

下面是我的脚本 

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

## 工程里如何配置  

插件是如下三个: 

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <mainClass>com.xxx.Application</mainClass>
            <layout>WAR</layout>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>repackage</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
            <excludes>
                <exclude>**/*IT.java</exclude>
            </excludes>
        </configuration>
    </plugin>

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-failsafe-plugin</artifactId>
        <configuration>
            <includes>
                <include>**/*IT.java</include>
            </includes>
        </configuration>
        <executions>
            <execution>
                <phase>integration-test</phase>
                <goals>
                    <goal>integration-test</goal>
                    <goal>verify</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

- spring-boot-maven-plugin : 基本生命周期套件
- maven-surefire-plugin : 自动化单元测试
- maven-failsafe-plugin : 自动化集成测试 

对RD的基本要求: 

- 集成测试复合插件规定,IT形式
- 单元测试不需要外部启动,全部mock,db_mock用hsqldb
- 运行时启动测试全部加上@Ignore避开持续集成  

## 遇到什么问题 

runner默认的镜像中maven是从中央库拉jar包,这不行.  

最开始思路有了点问题,尝试着自建一个mvn镜像push到docker hub,但是查看了maven官方镜像的Dockerfile是下面这样的:  

```
FROM openjdk:7-jdk

ARG MAVEN_VERSION=3.3.9
ARG USER_HOME_DIR="/root"

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL http://apache.osuosl.org/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz \
    | tar -xzC /usr/share/maven --strip-components=1 \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

COPY mvn-entrypoint.sh /usr/local/bin/mvn-entrypoint.sh
COPY settings-docker.xml /usr/share/maven/ref/

VOLUME "$USER_HOME_DIR/.m2"

ENTRYPOINT ["/usr/local/bin/mvn-entrypoint.sh"]
CMD ["mvn"]
``` 

那么我们-v把目录挂载到每个`docker-in-docker`容器中. 

换言之,将自己的`settings.xml`置入宿主机的`/root/.m2`即可解决问题. 

那么索性将jar包下载地址放入/root/.m2,这样只需要挂载一个目录 

也就是修改settings.xml 

```xml
 <localRepository>/root/.m2</localRepository>
```

我们在准备工作里的`/srv/gitlab-runner/config`派上了用处,里面在运行初始化配置后会生成
`config.toml` 

那么修改如下,来利用重复镜像与maven配置和repo:   

```
[runners.docker]
    volumes = ["/cache","/root/.m2:/root/.m2"]
    pull_policy = "if-not-present"
```

> 另外当持续集成进行时你在宿主机`docker ps`就能看到生成的`docker-in-docker`实例. 

## 成果 

![](https://o4dyfn0ef.qnssl.com/image/2017-01-03-Screen%20Shot%202017-01-03%20at%2013.57.59.png?imageView2/2/h/400) 

每次push或者merge request都会触发,这部分可以自行设置. 

另外merge request在持续集成成功之前是无法完成的. 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-08-Screen%20Shot%202016-09-08%20at%2014.50.39.png?imageView2/2/h/300) 

这部分也可以通过`docker logs`命令来看到 

![](https://o4dyfn0ef.qnssl.com/image/2017-01-03-Screen_Shot_2017-01-03_at_13_58_53.png?imageView2/2/h/400) 

## 添加构建状态徽标 

在CI/CD pipline选项里面看得到添加代码 

我目前是这么写的

```
## 构建状态
- master [![build status](http://yourgitlab/group/project/badges/master/build.svg)](http://yourgitlab/group/projec/commits/master)
- develop [![build status](http://yourgitlab/group/projec/badges/develop/build.svg)](http://yourgitlab/group/projec/commits/develop)
```

配合git flow,可以很清晰看到主要分支状态 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-09-Screen%20Shot%202016-09-09%20at%2012.27.54.png?imageView2/2/h/200) 

## 可以优化的点 

- on_failure的时候应该把单元测试与集成测试的report取出来发邮件,这个以后要加上
- spotify的docker-maven插件可以直接把镜像推到registry,那么hook触发后就简单了,见下一篇. 

- - - - --  

持续集成阶段拓扑,后续应该什么样? 

```
stages: 
    - install 
    - ut
    - it
    - build_image
    - push_image
    - deploy
```

大概是这样的规划

那么持续集成的部分结束,持续交付择日继续. 

> 2017-01-03更新 

上文我的思路没错,不过上了kubernetes之后.最后这一步deploy是不需要的 

我们运行到把镜像push到registry就算CD终止. 

- - - - -- 

done. 




