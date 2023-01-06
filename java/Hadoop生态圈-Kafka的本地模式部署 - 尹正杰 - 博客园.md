　　　　　　　　　　　　　　　　　　　　**Hadoop生态圈-Kafka的本地模式部署**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.Kafka简介**

**1>.什么是JMS**

　　答：在Java中有一个角消息系统的东西，我们叫他Java Message Service，简称JMS。比如各种MQ。

**2>.JMS的两种工作模式**

　　第一种模式：点到点（point to point,简称P2P），典型的一对一模式（一个人发送数据的同时只有一个人接收数据），也有人称之为端到端（peer to peer）。

　　第二种模式：发布订阅模式（publish subscribe，简称P-S），典型的一对多模式（一个人发送数据的同时可以有多个人接收数据）。

**3>.Kafka的工作模式**

　　答：Kafka的工作模式可以把JMS的两种模式结合在一起，我们称之为消费者组模式**。**

**4>.什么是Kafka**

　　答：Kafka和flume以及Sqoop一样，他们都是中间件（不含有业务的技术组件）。Kafka在官方定义是分布式消息系统。当然Kafka还可以用在做分布式数据库，除此之外，它还可以当做分布式缓存。

**5>.ApacheKafka是一个分布式流媒体平台**

　　ApacheKafka是一个分布式流媒体平台，这到底是什么意思呢？接下来我们看一下流媒体平台有三个关键功能如下：

　　　　第一：发布和订阅记录流，类似于消息队列或企业消息传递系统。

　　　　第二：以容错持久的方式存储记录流。

　　　　第三：处理记录发生的流。

**6>.Kafka通常用于两大类应用**

　　第一：构建可在系统或应用程序之间可靠获取数据的实时流数据管道。

　　第二：构建实时流应用程序，用于转换或响应数据流。

**7>.kafka版本介绍**

 　　kafka起先由领英（linkedin创建）公司，开源后被Apache基金会纳入子项目。我们在下载Kafka时，你是如何区分它的版本呢？比如本篇博客下载kafka的版本是“kafka\_2.11-1.1.0”，这个“2.11”是scala（java语言脚本化）版本而“1.1.0”是kafka版本。

**二.Kafka本地模式**部署****

**1>.下载Kafaka**

 　　下载地址：http://kafka.apache.org/downloads。

**2>.解压并****创建软连接**

```
[yinzhengjie@s101 data]$ tar xzf kafka_2.11-1.1.0.tgz -C /soft/
[yinzhengjie@s101 data]$ ln -s /soft/kafka_2.11-1.1.0/ /soft/kafka
[yinzhengjie@s101 data]$ 
```

**4>.配置环境变量并使之生效**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[yinzhengjie@s101 data]$ sudo vi /etc/profile
[sudo] password for yinzhengjie: 
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ tail -3 /etc/profile
#ADD KafKa PATH
export KAFKA_HOME=/soft/kafka
PATH=$PATH:$KAFKA_HOME/bin
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ source /etc/profile
[yinzhengjie@s101 data]$ 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.修改配置文件**

```
[yinzhengjie@s101 data]$ sed -i 's@#listeners=PLAINTEXT://:9092@listeners=PLAINTEXT://s101:9092@g' /soft/kafka/config/server.properties
[yinzhengjie@s101 data]$ sed -i 's@log.dirs=/tmp/kafka-logs@log.dirs=/home/yinzhengjie/kafka/logs@g' /soft/kafka/config/server.properties
[yinzhengjie@s101 data]$ sed -i 's@zookeeper.connect=localhost:2181@zookeeper.connect=s102:2181,s103:2181,s104:2181@g' /soft/kafka/config/server.properties
[yinzhengjie@s101 data]$ 
```

**6>.启动kafka**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[yinzhengjie@s101 data]$ kafka-server-start.sh -daemon /soft/kafka/config/server.properties
[yinzhengjie@s101 data]$ echo $?
0
[yinzhengjie@s101 data]$ jps | grep Kafka
10145 Kafka
[yinzhengjie@s101 data]$ netstat -untalp | grep 9092
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 172.30.100.101:9092     :::*                    LISTEN      10145/java          
tcp6       0      0 172.30.100.101:37648    172.30.100.101:9092     ESTABLISHED 10145/java          
tcp6       0      0 172.30.100.101:9092     172.30.100.101:37642    TIME_WAIT   -                   
tcp6       0      0 172.30.100.101:37646    172.30.100.101:9092     TIME_WAIT   -                   
tcp6       0      0 172.30.100.101:9092     172.30.100.101:37648    ESTABLISHED 10145/java          
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**7>.停止kafka**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[yinzhengjie@s101 data]$ jps | grep Kafka
10145 Kafka
[yinzhengjie@s101 data]$ netstat -untalp | grep 9092
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 172.30.100.101:9092     :::*                    LISTEN      10145/java          
tcp6       0      0 172.30.100.101:37648    172.30.100.101:9092     ESTABLISHED 10145/java          
tcp6       0      0 172.30.100.101:9092     172.30.100.101:37642    TIME_WAIT   -                   
tcp6       0      0 172.30.100.101:37646    172.30.100.101:9092     TIME_WAIT   -                   
tcp6       0      0 172.30.100.101:9092     172.30.100.101:37648    ESTABLISHED 10145/java          
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ kafka-server-stop.sh
[yinzhengjie@s101 data]$ jps | grep Kafka
[yinzhengjie@s101 data]$ netstat -untalp | grep 9092
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 172.30.100.101:37649    172.30.100.101:9092     TIME_WAIT   -                   
tcp6       0      0 172.30.100.101:9092     172.30.100.101:37648    TIME_WAIT   -                   
[yinzhengjie@s101 data]$ 
[yinzhengjie@s101 data]$ 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.kafka初体验**

**1>.启动kafka（提供服务）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[yinzhengjie@s101 data]$ kafka-server-start.sh -daemon /soft/kafka/config/server.properties
[yinzhengjie@s101 data]$ netstat -untalp | grep 9092
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 172.30.100.101:9092     :::*                    LISTEN      10516/java          
tcp6       0      0 172.30.100.101:9092     172.30.100.101:37657    ESTABLISHED 10516/java          
tcp6       0      0 172.30.100.101:37657    172.30.100.101:9092     ESTABLISHED 10516/java          
[yinzhengjie@s101 data]$ jps | grep Kafka
10516 Kafka
[yinzhengjie@s101 data]$ 
```

\[yinzhengjie@s101 data\]$ kafka-server-start.sh -daemon /soft/kafka/config/server.properties

**2>.创建主题（**消息保管者**）  
**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[yinzhengjie@s101 data]$ kafka-topics.sh --zookeeper s102:2181 --create  --partitions 2 --replication-factor 1  --topic yinzhengjie
Created topic "yinzhengjie".
[yinzhengjie@s101 data]$ 
```

\[yinzhengjie@s101 data\]$ kafka-topics.sh --zookeeper s102:2181 --create --partitions 2 --replication-factor 1 --topic yinzhengjie

**3>.启动生产者（**消息发送方**）  
**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[yinzhengjie@s101 data]$ kafka-console-producer.sh --broker-list s101:9092 --topic yinzhengjie
>My name is yinzhengjie , I love Beijing!   
>
```

\[yinzhengjie@s101 data\]$ kafka-console-producer.sh --broker-list s101:9092 --topic yinzhengjie

![](https://images2018.cnblogs.com/blog/795254/201806/795254-20180621151907064-487165719.png)

**4>.启动消费者（**消息接收方**）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[yinzhengjie@s101 lib]$ kafka-console-consumer.sh --zookeeper s102:2181 --topic yinzhengjie --from-beginning
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
My name is yinzhengjie , I love Beijing!
```

\[yinzhengjie@s101 lib\]$ kafka-console-consumer.sh --zookeeper s102:2181 --topic yinzhengjie --from-beginning

 ![](https://images2018.cnblogs.com/blog/795254/201806/795254-20180621152058712-1627731653.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186