最近需要搭建一套日志监控平台,参考了新浪与美团的一些东西.现在实录一下搭建与优化调整的过程 

> 目前把这几件放在一起的文档还不够多,其中相当一部分因为elk的升级配置也已经不能用了,更多的是单机版的配置,完全没有参考性. 

>优化的部分将等待项目与新平台正式上线在另一篇文章写出

## 架构图 

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-04-21%20at%2021.44.09.png?imageView2/2/h/200)

## 软硬件配置  

- 本机 ubuntu 14.04
- 线上 centos 6.5

|  host |   本地搭建 | 线上环境   |
| :-----: |:--------:| :-----:|
| c1     | 1core 1g | 4core 8g |
| c2     | 1core 1g | 4core 8g |
| c3     | 1core 1g | 4core 8g |
| c4     | 2core 4g | 8core 32g |
| c5     | 2core 4g | 4core 32g |


|  c1 |   c2 | c3   | c4 |   c5 |
| :-----: |:--------:| :-----:|:--------:| :-----:|
| jdk+scala+zk+kafka  | 同左 | 同左 | jdk+es+logstash+kibana | jdk+es |

## 搭建 

### basic 

新机器修改root密码`sudo passwd` 

创建用户 

```shell
useradd cluster
passwd cluster
chmod +w /etc/sudoers
vim  /etc/sudoers
cluster ALL=(root)NOPASSWD:ALL 
chmod -w /etc/sudoers
mkdir /home/cluster
mkdir /home/stack
chmod 777 /home/cluster
chmod 777 /home/stack
```  

> 以上部分/home/stack用于存储所需所有tar.gz包 
> 
> /home/cluster作为所有软件的安装目录 
> 
> 而后创建cluster目录下data目录,用于存放各组件配置,日志,数据 
> 
> scp推荐工具ZOC7 

同步各个机器的hosts  

```conf
127.0.0.1 localhost 
10.1.12.25 c1
10.1.12.23 c2
10.1.12.24 c3
10.1.12.27 c4
10.1.12.28 c5
``` 

分发各机器的rsa公钥 

```shell
ssh-keygen
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh/authorized_keys 
chmod 600 -R ~/.ssh/
```
各机器profile设置 

```shell
tar -zxvf 
sudo vim /etc/profile
export JAVA_HOME=/home/cluster/jdk
export ZK_HOME=/home/cluster/zookeeper
export KAFKA_HOME=/home/cluster/kafka
export SCALA_HOME=/home/cluster/scala
export PATH=$PATH:$JAVA_HOME/bin:$ZK_HOME/bin:$KAFKA_HOME/bin:$SCALA_HOME/bin   
替换系统默认java
sudo update-alternatives --install /usr/bin/java java /home/cluster/jdk/jre/bin/java 301 
sudo update-alternatives --config java 
java -version
```

前三台机器的cluster目录树 

```shell
drwxr-xr-x  4 root    root    4096  4月 20 14:34 data/
drwxr-xr-x  8 uucp        143 4096  3月 21 13:13 jdk/
drwxr-xr-x  7 root    root    4096  4月 20 14:30 kafka/
drwxrwxr-x  6 cluster cluster 4096  3月  4 23:30 scala/
drwxr-xr-x 10 zy      zy      4096  4月 20 14:13 zookeeper/  
```
后两台机器的cluster目录树 

```shell
drwxr-xr-x  4 zy      root    4096  4月 21 17:23 data/
drwxrwxrwx  7 zy      root    4096  4月 21 15:07 elasticsearch/
drwxr-xr-x  8 zy          143 4096  3月 21 13:13 jdk/
drwxr-xr-x 10 zy      staff   4096  3月 29 06:46 kibana/
drwxr-xr-x  5 zy      root    4096  4月 21 18:06 logstash/  
```
开发环境下关闭防火墙 

```shell
chkconfig  iptables off && service iptables status或者
ufw disable
```
非ubuntu机器关闭SELinux 

```
修改 /etc/selinux/config，将 SELINUX=enforcing 改为 SELINUX=disabled
selinux默认ubuntu不安装,iptables默认也是全开放的.可以用getenforce和iptables -L命令查看下两个组件的状态
```

同步各机器时区 

```shell
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### zookeeper配置 

修改conf目录下模板配置为zoo.cfg 

```conf
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/cluster/data/zookeeper
clientPort=2181
server.1= c1:2888:3888
server.2= c2:2888:3888
server.3= c3:2888:3888  
``` 

而后 

在配置的dataDir目录下创建一个myid文件，里面写入一个0-255之间的一个随意数字，每个zk上这个文件的数字要是不一样的，这些数字应该是从1开始，依次写每个服务器。 

> 文件中序号要与dn节点下的zk配置序号一直，如：server.1=c1:2888:3888，那么dn1节点下的myid配置文件应该写上1

- 各节点启动:`bin/zkServer.sh start`
- 查看节点状态与leader与否`bin/zkServer.sh status`
- 查看java进程`jps`

### kafka配置

配置目录下 

zookeeper.properties 

```conf
dataDir=/home/cluster/data/zookeeper
clientPort=2181
maxClientCnxns=0  
``` 

server.properties 

```conf
############### Server Basics ###############
broker.id=0
############# Socket Server Settings #############
listeners=PLAINTEXT://:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
##############Log Basics ##############
log.dirs=/home/cluster/data/kafka/log
num.partitions=1
num.recovery.threads.per.data.dir=1
############ Log Retention Policy ###############
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
############## Zookeeper ###############
zookeeper.connect=localhost:2181
zookeeper.connection.timeout.ms=6000  
```

producer.properties 

```conf
############### Producer Basics ##############
metadata.broker.list=c1:9092,c2:9092,c3:9092
producer.type=sync
compression.codec=none
serializer.class=kafka.serializer.DefaultEncoder    
```

consumer.properties 

```conf
zookeeper.connect=c1:2181,c2:2181,c3:2181
zookeeper.connection.timeout.ms=6000
group.id=cluster-consumer-group 
```

- 启动`nohup bin/kafka-server-start.sh config/server.properties & `
- 依然jps`jps`

测试 

> 本部分的分片partitions有待调整,分布式的logstash需要分片的消息,否则会出现消息顺序错误

leader上执行 

```shell
bin/kafka-topics.sh --zookeeper c1:2181,c2:2181,c3:2181 --topic tt_topic --replication-factor 3 --partitions 3 --create
bin/kafka-topics.sh --zookeeper c1:2181,c2:2181,c3:2181 --topic tt_topic --describe
``` 

leader消息发送 

```shell
bin/kafka-console-producer.sh --broker-list c1:9092,c2:9092,c3:9092 --topic tt_topic
```

从者消息消费 

```conf
bin/kafka-console-consumer.sh --zookeeper c1:2181,c2:2181,c3:2181 --from-beginning --topic tt_topic
```

### elasticsearch配置 

修改es文件夹所属用户组 `chown -R cluster /home/cluster/elasticsearch`
修改data文件夹所属用户组 `chown -R cluster /home/cluster/data`

es配置 

```conf
cluster.name: elasticsearch
node.name: c4
path.data: /home/cluster/data/elasticsearch/data
path.logs: /home/cluster/data/elasticsearch/log
bootstrap.mlockall: true
//这行一定要填ip
network.host: 10.1.12.27
http.port: 9200
discovery.zen.ping.unicast.hosts: ["c4", "c5"]
discovery.zen.minimum_master_nodes: 2 
```  

安装插件 

```shell
 bin/plugin install mobz/elasticsearch-head
 bin/plugin install lmenezes/elasticsearch-kopf
 bin/plugin install lukas-vlcek/bigdesk
```

- 运行`bin/elasticsearch -d`
- 状态查看`http://10.1.12.27:9200/_plugin/head/`

### kibana配置 

基本设置需要修改的部分 

```conf
server.port: 5601
server.host: "c4"
elasticsearch.url: "http://c4:9200"  
```

- 启动`nohup bin/kibana &`
- 访问`http://10.1.12.27:5601`

### logstash配置

修改ruby源,修改Gemfile文件`https://ruby.taobao.org` 

安装插件  

```shell
bin/logstash-plugin install logstash-input-kafka
bin/logstash-plugin install logstash-output-elasticsearch
```
新建kafka-logstash-es.conf  

置于cluster/data/logstash/conf目录下 

```conf
input {
    kafka {
        zk_connect => "c1:2181,c2:2181,c3:2181"
        group_id => "cluster-consumer-group"
        topic_id => "tt_topic"
        reset_beginning => false 
        consumer_threads => 5  
        decorate_events => true 
        codec => "plain"
        }
    }
output {
    elasticsearch {
        hosts => ["c4:9200","c5:9200"]
        index => "logstash-log-%{+YYYY.MM.dd}"
        workers => 5
        codec => "json"
		  }
	 }
``` 

测试配置文件 

```shell
bin/logstash -f /home/cluster/data/logstash/conf/kafka-logstash-es.conf --configtest
``` 

运行 

```shell
nohup bin/logstash -f /home/cluster/data/logstash/conf/kafka-logstash-es.conf  &
``` 

这个平台搭建的后期我遇见了新的需求,对flume的定制需求越来越多,如果你不想面对这种情况,那么可以这样: 

```shell 
bin/logstash-plugin install logstash-input-log4j 
bin/logstash-plugin install logstash-output-kafka
```

把flume的部分替换成使用logstash来进行: 

```conf
input{
    log4j {
        mode => "server"
        host => "[c1/c2/c3]"
        port => 4560
    }
}

output{
    kafka {
        bootstrap_servers => "c1:9092,c2:9092,c3:9092"
        topic_id => "tt_topic"
        workers => 5
        codec => "plain"
    }
}
``` 

同样在log4j中配置新的SocketAppender指向挂在logstash集群前的负载均衡. 

```
<appender name="LOGSTASH-APPENDER" class="org.apache.log4j.net.SocketAppender">
    <param name="remoteHost" value="lb1" />
    <param name="port" value="4560" />
    <param name="Threshold" value="INFO" />
    <param name="ReconnectionDelay" value="1000" />
    <param name="LocationInfo" value="true" />
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%-d{yyyy-MM-dd HH:mm:ss}-[%p]-[%l]-%m%n" />
    </layout>
  </appender>
``` 


> 想要使用多个 logstash 端协同消费同一个 topic 的话，那么需要把两个或是多个 logstash 消费端配置成相同的 group_id 和 topic_id 
> 
> 但是前提是要把相应的 topic 分多个 partitions (区)，多个消费者消费是无法保证消息的消费顺序性的 
> 
> 查看后台任务 jobs 
> 
> 杀掉后台任务 kill %number 

### flume配置 

- 确定jdk
- 上传stack
- 同步hosts

flume-env.sh 

```shell
export JAVA_HOME=/opt/software/java 
export JAVA_OPTS="-Xms1024m -Xmx2048m" 
```

flume-kafka.properties 

```properties
ag1.sources=src1 src2
ag1.sinks=sink1
ag1.channels=chn1

ag1.sources.src1.type = exec
ag1.sources.src1.shell = /bin/bash -c
ag1.sources.src1.command = tail -F tt.log
ag1.sources.src1.channels = chn1

ag1.sources.src2.type = exec
ag1.sources.src2.shell = /bin/bash -c
ag1.sources.src2.command = tail -F tt.log
ag1.sources.src2.channels = chn1

ag1.channels.chn1.type = memory
agent.channels.chn1.keep-alive = 60  
ag1.channels.chn1.capacity = 1000
ag1.channels.chn1.transactionCapacity = 100


ag1.sinks.sink1.type = org.apache.flume.sink.kafka.KafkaSink
ag1.sinks.sink1.topic = tt_topic
ag1.sinks.sink1.brokerList = c1:9092:c2:9092:c2:9092
ag1.sinks.sink1.channel = chn1
```

启动 

```
bin/flume-ng agent -n ag1  -c conf -f conf/flume-kafka.properties
```

### 初步成果 

初步成果如图,我们成功把22台机器的agent安好,获得了应用集群上的指定日志内容,后续的优化包括 

- 性能
- 日志filter
- 存储/压缩/备份
- visualize与dashboard配置
- 更多可视化插件安装 

![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-04-30%20at%2015.58.57.png?imageView2/2/h/600)




