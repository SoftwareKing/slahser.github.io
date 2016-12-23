![](https://o4dyfn0ef.qnssl.com/image/2016-11-21-migrating-gitlab-infrastructur-into-Docker.png?imageView2/2/h/300) 

因为公司错(shi)综(qian)复(ju)杂(keng)的网络环境,测试数据库的迁移到了内网,而nexus跟gitlab-ci-runner在外网. 

又迟迟拿不出一个能同时连通内外网的机器,一步一步做吧. 

- 将gitlab迁移到内网
- 将nexus迁移到意义访问外网&&可以被内网访问的机器
- 将gitlab-ci-runner迁移到内网环境

正好之前的gitlab是omnibus方式安装的,那么这次干脆运行在docker中好了 

我翻之前的`/etc/gitlab/gitlab.rb`真是受够了..  

- - - - -- 

## 下载Compose  

下载[本项目](https://github.com/sameersbn/docker-gitlab),checkout到与你之前gitlab相同的tag 

> Optional: 查看内含的redis/pg/docker-gitlab镜像版本,pull之.  

- - - - -- 

## 微调配置  

将compose中环境变量进行微调,比如我调节了:

- GITLAB_BACKUP_SCHEDULE - daily
- GITLAB_BACKUP_EXPIRY - 备份过期时间,避免磁盘占满,单位秒
- GITLAB_HOST - gitlab.yourcompany.com
- GITLAB_PORT - 挂nginx的话就无所谓了,否则请设置个奇奇怪怪的端口
- GITLAB_SSH_PORT - 挂nginx的话就转发过来
- GITLAB_SECRETS_DB_KEY_BASE - `pwgen -Bsv1 64`生成一个
- GITLAB_SECRETS_OTP_KEY_BASE - `pwgen -Bsv1 64`生成一个
- GITLAB_SECRETS_SECRET_KEY_BASE - `pwgen -Bsv1 64`生成一个
- GITLAB_TIMEZONE - Beijing
- GITLAB_ROOT_PASSWORD - 初次启动修改密码的那个预设值
- TZ=Asia/Shanghai

另外调节volumes到靠谱的硬盘上. 

- - - - -- 

## 运行compose 

```shell
docker-compose up -d 
```

- - - - -- 

## 备份老版本 

```shell
sudo gitlab-rake gitlab:backup:create
``` 

默认的备份目录在`/var/opt/gitlab/backups` 

- - - - -- 

## restore 

```shell
docker exec -it gitlab sudo -HEu git bundle exec rake gitlab:backup:restore BACKUP=1417624827
```

备份版本号是备份tar文件的前缀

- - - - -- 

done. 
