前些日子发现容器内时区老是不对... 

姑且把目前镜像打包的整个命令放出来吧. 

- 更换了apk源
- 调整了时区
- 记录了打包时间
- 进行了简单jvm参数调节
- 兼容了tomcat的推荐启动参数
- 留出了部署到Kubernetes的环境变量口子 

```
FROM java:8-jre-alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/' /etc/apk/repositories
RUN apk update && apk add ca-certificates
RUN apk add tzdata
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone
VOLUME /tmp
ADD app-1.0-exec.jar app.jar
RUN sh -c 'touch /app.jar'
RUN echo $(date) > /image_built_at
EXPOSE 8080
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-Xms4096m","-Xmx4096m","-Xmn2048m","-XX:MaxDirectMemorySize=2048m","-XX:MetaspaceSize=128m","-XX:MaxMetaspaceSize=512m","-XX:ReservedCodeCacheSize=240M","-jar","/app.jar","--spring.profiles.active=${SPRING_PROFILES_ACTIVE}"]
``` 

> 实际容器是Undertow,sessionId也是托管到spring session或者shiro的 

> 不过当初自己实现分布式session时候的确用的是java.security.Random就是了

- - - - -- 

如何测试容器内容 

我们目前就是简单的起一个alpine,生命周期很快就随着exit结束了. 

`docker run -it alpine /bin/sh` 

done. 
