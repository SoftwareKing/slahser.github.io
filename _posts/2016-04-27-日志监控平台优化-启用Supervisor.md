nohup启动起来的应用切换了shell就看不到jobs了,总是搞得心惊肉跳. 

启用Supervisor来进行进程管理与重新拉起 

## 安装

centos6.5 

```shell
pip install //在我这儿老是出依赖相关的错
yum install python-setuptools 
easy_install supervisor
```  

测试 

```shell
python
>>>  import supervisor
``` 

## 配置 

```shell
echo_supervisord_conf > /etc/supervisord.conf
vim /etc/supervisord.conf
// 修改最后一行: vim->shift+G
    [include]
    files = /etc/supervisor/conf.d/*.conf 
mkdir /etc/supervisor/conf.d
vim tt.conf
```

tt.conf 

```conf
[program:elasticsearch]
command=/home/cluster/elasticsearch/bin/elasticsearch
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = cluster       ; 用哪个用户启动

[program:kibana]
command=/home/cluster/kibana/bin/kibana
autostart = true     
startsecs = 5        
autorestart = true   
startretries = 3     
user = root       

[program:logstash]
command=/home/cluster/logstash/bin/logstash -f /home/cluster/data/logstash/conf/kafka-logstash-es.conf
autostart = true     
startsecs = 5        
autorestart = true   
startretries = 3     
user = root 
log_stdout=ture
log_stderr=true
logfile=/home/cluster/data/logstash/log/logstash.log

[program:flume]
command=/home/stack/flume/bin/flume-ng agent -n ag1  -c conf -f /home/stack/flume/conf/flume-kafka.properties
autostart = false     
startsecs = 5        
autorestart = true   
startretries = 3     
user = root

[program:zookeeper]
command=/home/cluster/zookeeper/bin/zkServer.sh start
autostart = true     
startsecs = 5        
autorestart = true   
startretries = 3     
user = root

[program:kafka]
command=/home/cluster/kafka/bin/kafka-server-start.sh /home/cluster/kafka/config/server.properties
autostart = false     
startsecs = 5        
autorestart = true   
startretries = 3     
user = root
``` 

> 我们这里要注意,像elasticsearch这种启动方式自带deamon进程的形式,最好利用`bin/elasticsearch -d` 启动,不要使用Supervisor管理,否则supervisor> status 会提示：BACKOFF  Exited too quickly.

## 启动  

```
启动服务
supervisord -c /etc/supervisord.conf
启动客户端
supervisorctl -c /etc/supervisord.conf 
修改配置后
supervisorctl reload 
```

## 使用 

```
supervisorctl -c /etc/supervisord.conf
以上命令进入客户端shell
status    # 查看程序状态
stop tt   # 关闭 tt 程序
start tt  # 启动 tt 程序
restart tt    # 重启 tt 程序
reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
update    ＃ 重启配置文件修改过的程序

也可以如下在shell直接进行
supervisorctl status
supervisorctl stop tt
supervisorctl start tt
supervisorctl restart tt
supervisorctl reread
supervisorctl update
``` 

## web管理界面

将/etc/supervisord.conf中[inet_http_server]部分做相应配置，在supervisorctl中reload即可启动web管理界面,聊胜于无吧. 

## 效果 

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-04-27%20at%2013.35.13.png?imageView2/2/h/200)
