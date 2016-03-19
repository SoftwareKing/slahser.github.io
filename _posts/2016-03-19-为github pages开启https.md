![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2018.16.27.png)

昨天我接到了一封邮件.一名叫 @nubela 的小伙子在邮件里极尽诱惑,描述可以为我的博客开启Let's encrypt,并且具有一些其他dashboard功能. 

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2018.14.58.png)

文中链接在 [这里](https://kloudsec.com/github-pages)

特色功能是如下:
- 一键开启https
- 免费的cdn
- 其他的kcloudsec插件,比如定期的快照,防xss等等.

按如下链接填入前两阶信息,会生成两段dns解析配置,如下图新建两条解析记录

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2019.51.08.png)

稍等片刻,访问你的`https://host` 可以发现一切已经准备停当了.功能稍后介绍,先说说升级之后jekyll的调整

- 博客的url要调整
- 样式文件中`//`形式简写的部分,某些必要的要加上协议`http/https://` .
- 目前发现国内的google font镜像在启用https之后,会访问不到.

功能如下:
### 测速 

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2019.48.20.png)

### 网站可用性.提供离线快照

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2019.48.35.png)

### 网站防护

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2019.48.53.png)

### 其余特色插件

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-19%20at%2019.49.29.png)


done.