![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-23%20at%2020.47.50.png?imageView2/2/h/600) 

今天介绍一下如何为树莓派或者任何主机添加web terminal 

[butterfly项目主页](https://github.com/paradoxxxzero/butterfly)  

### 安装:
```
$ pip install butterfly
$ pip install libsass  # If you want to use themes
$ butterfly
```
依terminal提示访问url 

树莓派上安装可能会提示一些依赖编译不过,需要先手动解决一下间接依赖,stackoverflow可破.

### 作为server运行
如下命令`--host`填 

- `localhost` 本地运行
- 0.0.0.0 本地/公网
- ip 公网 

```
$ git clone https://github.com/paradoxxxzero/butterfly.git
$ cd butterfly
$ python butterfly.server.py --host=[myhost] --port=57575 --nosecure
``` 

### 作为linux service运行

```
$ cd /etc/systemd/system
# curl -O https://raw.githubusercontent.com/paradoxxxzero/butterfly/master/butterfly.service
# curl -O https://raw.githubusercontent.com/paradoxxxzero/butterfly/master/butterfly.socket
# systemctl enable butterfly.socket
# systemctl start butterfly.socket
``` 

### 效果图 

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/687474703a2f2f70.gif?imageView2/2/h/600)

> 其他功能比如主题设置,session时间调整自行探索. 
> ps:我对web terminal的认识开始于github上一个star,而对web terminal彻底绝望是在我使用[蚂蚁金融云](https://www.cloud.alipay.com)的时候,体验催人泪下.
