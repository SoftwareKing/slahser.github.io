![](https://o4dyfn0ef.qnssl.com/image/2016-11-15-kube7-logo.png?imageView2/2/h/200) 

这篇简单分享两块内容: 

- 更换到 deploy + beats后配置文件
- multiline 相关的点

------- 

### deploy配置文件 

这份配置几经更新,因为 篇幅太长姑且放在文末,简单描述一下里面现在有什么 

- deploy 的部署方式来支持 rollout状态
- resource相关的限制
- readiness 与 liveness 的健康检查
- 基于 env 与 secret 来隐藏项目中敏感内容
- 同一 pod 种放置两个 container 来维持单进程
- 通过将日志输出到同一个Volume来收集日志

------- 

### beats 配置文件 

相关部署就不提了,能 hold 住什么就用什么. 

目前我打算最简单直接收集好发到 es cluster 就行了. 

有其他 filter 等需求我们再升级. 

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: production
data:
  filebeat.yml: |
    filebeat.prospectors:
    - input_type: log
      paths:
        - /log/*.log
        - /log/bizLog/*.log
      multiline.pattern: '^[[:space:]]+|^Caused by:|^#{3}|^;|^org|^com'
      multiline.negate: false
      multiline.match: after
    output.elasticsearch:
      hosts: ["ip1:9200","ip2:9200","ip3:9200"]
      index: "filebeat-your-app-%{+yyyy.MM.dd}"

``` 

官方的配置介绍在[这里](https://www.elastic.co/guide/en/beats/filebeat/5.4/configuring-howto-filebeat.html),后面的部分甚至还有一段在线的 go 代码来帮你验证是否 pattern. 

上面这份配置文件呢,用 cm 的形式存到了 k8s 里,用来给特定 container 读取. 

其中 multiline 的部分还是看情况来,针对 logback 输出的日志我客制化了一下这段正则. 


> 果然日志以后还是要规范一下,最理想莫过于全变成 ctx log 就好了. 

------ 

### 很长的 deploy 配置 

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: app
  namespace: production
  labels:
    name: app
spec:
  replicas: 2
  template:
    metadata:
      labels:
        name: app
    spec:
      containers:
      - name: app
        image: registry.yours.com/app-image:[yourvesion]
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: app-logs
          mountPath: /usr/local/app/logs
        resources:
          requests:
            cpu: 2
            memory: 8Gi
          limits:
            cpu: 3
            memory: 16Gi
        readinessProbe:
          httpGet:
            path: /readness
            port: 8080
            scheme: HTTP
          timeoutSeconds: 3
          initialDelaySeconds: 180
          periodSeconds: 15
          successThreshold: 1
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 180
          periodSeconds: 15
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
        env:
          - name: SPRING_PROFILES_ACTIVE
            value: prod
          - name: MYSQL_HOST
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mysqlHost
          - name: MYSQL_PORT
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mysqlPort
          - name: MYSQL_DB
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mysqlDb
          - name: MYSQL_USER
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mysqlUser
          - name: MYSQL_PWD
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mysqlPwd
          - name: REDIS_HOST
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: redisHost
          - name: REDIS_PORT
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: redisPort
          - name: REDIS_PWD
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: redisPwd
          - name: MONGO_HOST
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mongoHost
          - name: MONGO_PORT
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mongoPort
          - name: MONGO_DB
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mongoDb
          - name: MONGO_USER
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mongoUser
          - name: MONGO_PWD
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: mongoPwd
          - name: RABBIT_HOST
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: rabbitHost
          - name: RABBIT_PORT
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: rabbitPort
          - name: RABBIT_USER
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: rabbitUser
          - name: RABBIT_PWD
            valueFrom:
              secretKeyRef:
                name: yours-secret
                key: rabbitPwd
      - name: filebeat
        image: registry.yours.com/filebeat:5.4.0
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: app-logs
          mountPath: /log
        - name: filebeat-config
          mountPath: /etc/filebeat/
      volumes:
      - name: app-logs
        emptyDir: {}
      - name: filebeat-config
        configMap:
          name: filebeat-config
```

> 拆一拆就好了. 

对了,本文的镜像来自[这里](https://github.com/rootsongjc/docker-images),从官方自己打包的话需要调整一下外置配置文件就是了. 

------- 

done. 
