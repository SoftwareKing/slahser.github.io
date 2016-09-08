![](https://o4dyfn0ef.qnssl.com/image/2016-09-07-Screen%20Shot%202016-09-07%20at%2015.49.25.png?imageView2/2/h/400) 

> G20终于过去了..放了个莫名其妙的假去所谓的景点伤筋动骨了一番.. 

最近需要实践一下Gitlab based的CI与CD 

- 持续集成在先,先把写好的模块化单元测试跑起来
- 持续交付在后,通过PR生成镜像到私服 

> Gitlab 8.0以后的版本自带CI,不需要到Services里面去启用了. 

> 如何找到Runenr/文档什么的我就不说了,大家都很忙. 

- - - - -- 

## 准备工作 

1. 安装Docker
2. 安装[Runner - Docker](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/install/docker.md) : 出于安全与效率原因. 
3. `mkdir -p /srv/gitlab-runner/config`
4. `docker pull gitlab/gitlab/gitlab-runner`

## 运行 

```sh
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner:Z \
  gitlab/gitlab-runner:latest
``` 

> 这条命令用来解决SELinux问题,直接关闭SELinux好了. 

## 注册到gitlab 

```
docker exec -it gitlab-runner gitlab-runner register
``` 

- executor选docker来启用`docker-in-docker`模式 
- image写maven:3-jdk-7 

## 写脚本 

到`http://yourgitlab/ci/lint`检测自己的脚本是否科学. 

下面是我的脚本 

```yaml
image: maven:3-jdk-7

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

ut:
  script:
    - mvn test
  stage: ut

it:
  script:
    - mvn integration-test
  stage: it

after_script:
  - mvn clean
```

## 工程里如何配合 

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

> 各自的文档各位受累自己去看. 

## 遇到什么问题 

runner默认的镜像中maven是从中央库拉jar包,这不行!(尔康脸 

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

我们在准备工作里的`/srv/gitlab-runner/config`派上了用处,里面在运行初始化配置后会生成
`config.toml` 

那么修改如下来利用重复镜像与maven配置和repo:   

```
\[runners.docker\]
    volumes = ["/cache","/root/m2:/root/.m2"]
    pull_policy = "if-not-present"
```

> 另外当持续集成进行时你在宿主机`docker ps`就能看到生成的`docker-in-docker`实例. 

## 成果 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-08-Screen%20Shot%202016-09-08%20at%2015.33.28.png?imageView2/2/h/400) 

每次push或者merge request都会触发,这部分可以自行设置. 

另外merge request在持续集成成功之前是无法完成的. 

![](https://o4dyfn0ef.qnssl.com/image/2016-09-08-Screen%20Shot%202016-09-08%20at%2014.50.39.png?imageView2/2/h/400) 

这部分也可以通过`docker logs`命令来看到,不过这样方便点对吧~ 

那么持续集成的部分结束 

- - - - --  

持续交付择日继续~ 



