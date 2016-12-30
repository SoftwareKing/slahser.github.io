![](https://o4dyfn0ef.qnssl.com/image/2016-12-19-architecture.png?imageView2/2/h/300) 

需要的知识储备: 

- [Ingress](http://kubernetes.io/docs/user-guide/ingress/)
- [Traefik文档](https://docs.traefik.io/user-guide/kubernetes/)

Traefik扮演的是Ingress抽象中负载均衡器与Ingress Controller的角色 

路由与元数据访问都是Traefik来进行. 

- - - - - 

内容部分思路来自[这里](https://medium.com/@patrickeasters/using-traefik-with-tls-on-kubernetes-cb67fb43a948#.2u7hhq5my) 

### 证书签发 

`openssl req -newkey rsa:2048 -nodes -keyout tls.key -x509 -days 365 -out tls.crt`

### 创建traefik.toml 

```
defaultEntryPoints = ["http","https"]
[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
      entryPoint = "https"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      CertFile = "/ssl/tls.crt"
      KeyFile = "/ssl/tls.key"
```

### 创建Secret 

`kubectl create secret generic traefik-cert --from-file=tls.crt --from-file=tls.key -n kube-system` 

### 创建ConfigMap 

`kubectl create configmap traefik-conf --from-file=traefik.toml -n kube-system` 

### 更新deploy如下

```
apiVersion: v1
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  replicas: 2
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      volumes:
      - name: ssl
        secret:
          secretName: traefik-cert
      - name: config
        configMap:
          name: traefik-conf
      hostNetwork: true
      containers:
      - image: registry.gogen.com/traefik:v1.1.1
        name: traefik-ingress-lb
        volumeMounts:
        - mountPath: "/ssl"
          name: "ssl"
        - mountPath: "/config"
          name: "config"
        resources:
          limits:
            cpu: 200m
            memory: 30Mi
          requests:
            cpu: 100m
            memory: 20Mi
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: https
          containerPort: 443
          hostPort: 443
        - name: admin
          containerPort: 9002
        args:
        - --configfile=/config/traefik.toml
        - --web
        - --kubernetes
``` 

### Infress配置 

不需要变,该是http就是http,没有大碍. 

### 测试 

依然是`curl -k https:xxx`

- - - - -- 

done.


