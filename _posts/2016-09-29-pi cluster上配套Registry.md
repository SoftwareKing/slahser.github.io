书接[上文](http://www.slahser.com/2016/09/29/pi-cluster上搭建kubernetes/),我们遇到了GFW无法下载gcr.io的镜像,而后遇到了国内容器云的事儿. 

解决问题: 

- pull速度.
- 传统/官方的Registry搭建方式没有使用nginx,通用性和稳定性都有问题. 
- 使用Compose增强可维护性. 
- 说不定他们网站哪天倒闭了呢.

> 本文暂时不考虑`harbor`或者`portus`这种大型私服管理工具. 

- - - - --- 

## 更换mirror 

这里我是用了[Daocloud的Mirror](https://dashboard.daocloud.io)来加速官方镜像下载.  

```shell
# 这样
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://[yours].m.daocloud.io
# 或者这样
vim /etc/default/docker
--registry-mirror=http://[yours].m.daocloud.io
# 而后重启docker
```

> 本步骤与本篇无关,可选. 

- - - - --- 

## 搭建Registry 

如下几种简易搭建方式,我们选择第三种:  

- 使用--insecure-rgistry=xxxx:5000
- 官方文档直接启动registry:2
- 配合nginx来使用 

### 同步hosts 

```shell
echo '192.168.31.100 index.slahser.com' >> /etc/hosts
``` 

### 自签证书准备 

```shell
cd ~/.ssh
openssl genrsa -out index.slahser.com.key 2048
# 倒数第二步的name设置成index.slahser.com
openssl req -newkey rsa:4096 -nodes -sha256 -keyout index.slahser.com.key -x509 -days 365 -out index.slahser.com.crt
``` 

### 宿主机与客户机证书信任 

```shell
mkdir -p /etc/docker/certs.d/index.slahser.com
# ansible copy或者单机cp 
ansible pis -m copy -a 'src=~/.ssh/index.slahser.com.crt dest=/etc/docker/certs.d/index.slahser.com/'
cp ~/.ssh/index.slahser.com.crt /etc/docker/certs.d/index.slahser.com/
```

### 创建compose环境与配置文件

创建基本目录结构: 

```shell
mkdir -p ~/Documents/repository/compose/registry/nginx
# 复制ssl必要文件到挂载目录
cp ~/.ssh/index.slahser.com.crt ~/Documents/repository/compose/registry/nginx
cp ~/.ssh/index.slahser.com.key ~/Documents/repository/compose/registry/nginx
# nginx配置文件
vim ~/Documents/repository/compose/registry/nginx/registry.conf
``` 

nginx配置文件registry.conf: 

```
upstream docker-registry {
  server registry:5000;
}

server {
  listen 443;
  server_name index.slahser.com;

  # SSL
  ssl on;
  ssl_certificate /etc/nginx/conf.d/index.slahser.com.crt;
  ssl_certificate_key /etc/nginx/conf.d/index.slahser.com.key;

  # disable any limits to avoid HTTP 413 for large image uploads
  client_max_body_size 0;

  # required to avoid HTTP 411: see Docker Issue #1486
  chunked_transfer_encoding on;

  location /v2/ {
    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
      return 404;
    }

    proxy_pass                          http://docker-registry;
    proxy_set_header  Host              $http_host; 
    proxy_set_header  X-Real-IP         $remote_addr;
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
  }
}
``` 

compose配置文件docker-compose.yaml

``` 
nginx:
    container_name : nginx
    image : nginx:1.11.4
    ports :
        - 443:443
    links :
        - registry:registry
    volumes:
        - /Users/Slahser/Documents/repository/compose/registry/nginx:/etc/nginx/conf.d

registry:
    container_name : registry
    image : registry:2.5.1
    ports:
        - 5000:5000
    volumes:
        - /Users/Slahser/Documents/repository/registry:/var/lib/registry
``` 

### 仓库启动与测试访问 

```shell
docker-compose up -d
curl -k https://index.slahser.com/v2/
```

- - - - --- 

## 怎么打TAG 

mac上操作: 

```shell
docker pull index.tenxcloud.com/google_containers/etcd-arm:2.2.5
docker tag [yourimageid] index.slahser.com/google_containers/etcd-arm:2.2.5
docker push index.slahser.com/google_containers/etcd-arm:2.2.5
```

pi上操作: 

```shell
docker pull index.slahser.com/google_containers/etcd-arm:2.2.5
docker tag [yourimageid] gcr.io/google_containers/etcd-arm:2.2.5
```

done. 


