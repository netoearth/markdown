　　　　　　　　　　　　　　　**企业级Apache Kafka部署实战篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.安装zookeeper集群**

**1>.下载zookeeper**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# yum -y install wget
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.jdcloud.com
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
base                                                                                                                                                                                                                                     | 3.6 kB  00:00:00     
extras                                                                                                                                                                                                                                   | 3.4 kB  00:00:00     
updates                                                                                                                                                                                                                                  | 3.4 kB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package wget.x86_64 0:1.14-18.el7_6.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================================================================================================================================================================
 Package                                                   Arch                                                        Version                                                               Repository                                                    Size
================================================================================================================================================================================================================================================================
Installing:
 wget                                                      x86_64                                                      1.14-18.el7_6.1                                                       updates                                                      547 k

Transaction Summary
================================================================================================================================================================================================================================================================
Install  1 Package

Total download size: 547 k
Installed size: 2.0 M
Downloading packages:
wget-1.14-18.el7_6.1.x86_64.rpm                                                                                                                                                                                                          | 547 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : wget-1.14-18.el7_6.1.x86_64                                                                                                                                                                                                                  1/1 
  Verifying  : wget-1.14-18.el7_6.1.x86_64                                                                                                                                                                                                                  1/1 

Installed:
  wget.x86_64 0:1.14-18.el7_6.1                                                                                                                                                                                                                                 

Complete!
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# yum -y install wget　　　　　　　　#安装下载工具

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz
--2019-07-11 10:43:34--  https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz
Resolving mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)... 101.6.8.193, 2402:f000:1:408:8100::1
Connecting to mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)|101.6.8.193|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10622522 (10M) [application/x-gzip]
Saving to: ‘apache-zookeeper-3.5.5-bin.tar.gz’

100%[======================================================================================================================================================================================================================>] 10,622,522  1.98MB/s   in 5.0s   

2019-07-11 10:43:39 (2.04 MB/s) - ‘apache-zookeeper-3.5.5-bin.tar.gz’ saved [10622522/10622522]

[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ll
total 10376
-rw-r--r-- 1 root root 10622522 May 20 18:40 apache-zookeeper-3.5.5-bin.tar.gz
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz

**2>.解压zookeeper**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# tar -zxf apache-zookeeper-3.5.5-bin.tar.gz -C /home/softwares/
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ll /home/softwares/apache-zookeeper-3.5.5-bin/
total 32
drwxr-xr-x 2 2002 2002   232 Apr  9 19:13 bin
drwxr-xr-x 2 2002 2002    77 Apr  2 21:05 conf
drwxr-xr-x 5 2002 2002  4096 May  3 20:07 docs
drwxr-xr-x 2 root root  4096 Jul 11 10:44 lib
-rw-r--r-- 1 2002 2002 11358 Feb 15 20:55 LICENSE.txt
-rw-r--r-- 1 2002 2002   432 Apr  9 19:13 NOTICE.txt
-rw-r--r-- 1 2002 2002  1560 May  3 19:41 README.md
-rw-r--r-- 1 2002 2002  1347 Apr  2 21:05 README_packaging.txt
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# tar -zxf apache-zookeeper-3.5.5-bin.tar.gz -C /home/softwares/

**3>.**创建配置zookeeper的堆内存配置文件****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# cat /home/softwares/apache-zookeeper-3.5.5-bin/conf/java.env
#!/bin/bash
#@author :yinzhengjie
#blog:http://www.cnblogs.com/yinzhengjie
#EMAIL:y1053419035@qq.com

#指定JDK的安装路径
export JAVA_HOME=/home/softwares/jdk1.8.0_201

#指定zookeeper的heap内存大小
export JVMFLAGS="-Xms2048m -Xmx2048m $JVMFLAGS"
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# cat /home/softwares/apache-zookeeper-3.5.5-bin/conf/java.env

**4>.**修改zookeeper的配置文件zoo.cfg（需要手动创建，“/home/softwares/apache-zookeeper-3.5.5-bin/conf/zoo\_sample.cfg ”是一个配置模板）****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　Zookeeper的每个服务端的配置文件存放于安装目录的conf目录下，应命名为zoo.cfg，该目录下有一个zoo_sample.cfg文件是格式的简单样本。 

　　以下我们介绍zoo.cfg中可以配置的各种配置项。 
　　一.最小配置：以下三个配置是配置文件中必须存在的“最低消费”.
　　　　(1)clientPort：
　　　　　　　　zk服务器监听的端口，客户端通过该端口建立连接，每台 zk服务器也允许设置为不同的值。默认值2181 
　　　　(2)dataDir：
　　　　　　　　zk用于保存内存数据库的快照的目录，除非设置了 dataLogDir，否则这个目录也用来保存更新数据库的事务日志。在生产环境使用的zk集群，强烈建议设置dataLogDir，让dataDir只存放快照，因为写快照的开销很低，这样dataDir就可以和其他日志目录的挂载点合设.
　　　　(3)tickTime：
　　　　　　　　前面已提到过，zk使用的基本时间单位是tick，这个参数用于配置一个tick的长度，单位为毫秒，默认值3000，不建议修改.  


 　　二.高级配置：以下配置都是可选的，但如果要用好zk，有些几乎是一定要设的.  
　　　　(1)dataLogDir：
　　　　　　　　生产环境必须要设，注意，因为zk写事务日志是顺序地、 阻塞地，所以这个目录一定要单独划盘，禁止和其他高写操作的目录合设，否则严重影响性能。 
　　　　(2)globalOutstandingLimit：
　　　　　　　　因为zk服务器是需要对请求队列化的，当客户端很多，提交的写操作很频繁时，zk服务器可能在来得及处理前就OOM了。这个参数用于限制系统中还未处理的请求数阈值，默认为1000，一般不需要修改。  
　　　　(3)preAllocSize：
　　　　　　　　用于设置预分配的事务日志文件大小，单位为KB，默认值为64M。这个值明显太大，因为每次快照后都会生成一个新的事务日志文件，且一次的事务数是用snapCount限定住的，所以需要的日志文件大小可以准确估计，我使用时snapCount默认，单个事务大小不超过100字节，因此该值设为10240够了。  
　　　　(4)snapCount：
　　　　　　　　用于设置每次快照之间的事务数，默认为100000。因为写快照是消耗性能的，也没有必要频繁写，虽然该值看起来很大，一般不需要修改该值。  
　　　　(5)traceFile：
　　　　　　　　如果设置了该路径，将持续跟踪zk的操作并写入一个名为traceFile.year.month.day的跟踪日志中，该选项比较消耗性能，只在debug环境可以开启，生产环境不能开。 
　　　　(6)maxClientCnxns：
　　　　　　　　单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60，如果设置为0，那么表明不作任何限制。请注意这个限制的使用范围，仅仅是单台客户端机器与单台zk服务器之间的连接数限制，不是针对指定客户端IP，也不是zk集群的连接数限制，也不是单台zk对所有客户端的连接数限制。如果点对点连接数不高，建议改小这个值。  
　　　　(7)clientPortAddress：
　　　　　　　　用于配置zk的端口监听在本机哪个IP上生效，用 于服务器有多块网卡的情况，尤其是既有外网地址又有内网地址的集群， 只要开放内网地址即可。  
　　　　(8)minSessionTimeout：
　　　　　　　　客户端和zk服务器间的最小超时时间，单位为 tickTime的倍数，默认为2。  
　　　　(9)maxSessionTimeout：
　　　　　　　　客户端和zk服务器间的最大超时时间，单位为 tickTime的倍数，默认为20。客户端一般有自己的连接管理参数，如果客户端的参数不在minSessionTimeout--maxSessionTimeout这个范围内， 会被强制设为最接近的那个值。 
　　　　(10)fsync.warningthresholdms：
　　　　　　　　用于设置触发警告的存储同步时间阈值， 单位为毫秒，默认为1000，因为只是一个警告而已，无需修改。  
　　　　(11)autopurge.snapRetainCount：
　　　　　　　　3.4.0及之后版本zk提供了自动清理快照 文件和事务日志文件的功能，该参数指定了保留文件的个数，默认为3， 一般无需修改。 
　　　　(12)autopurge.purgeInterval：
　　　　　　　　和上一个参数配合使用，设置自动清理的 频率，单位为小时，默认为0表示不清理，建议设为6或12之类的值。  
　　　　(13)syncEnabled：
　　　　　　　　3.4.6之后新加的参数，在集群使用observer时，设置其 是否像leader和follower一样将快照和事务日志写到硬盘，默认为true。 我是没开observer的，如果你的集群开了，建议配成false，因为没有 必要。  


　　三.集群配置：用于zk集群模式的高级配置，单机模式是不需要管的。当然，没几个人会用单机的zk.
　　　　(1)electionAlg：
　　　　　　　　群首选举算法，默认为3代表FastLeaderElection，前面章节已经说过，除了这种算法外其余3种都已经被标注为deprecated了， 不要修改。  
　　　　(2)initLimit：
　　　　　　　　follower首次与leader连接并同步的超时值，因为集群启动或新leader选举完成后，follower需要从leader拉最新的数据，如果需要同步的数据很大还是可能超时的，单位为tickTime的倍数，没有默认值。  
　　　　(3)leaderServes：
　　　　　　　　默认为yes，此时leader也会接受客户端的连接提供读 写服务。如果设置为no，则leader不会和客户端连接，只负责处理 follower转发的写操作请求以及集群协调事务。改写该参数需要小心， 有可能会从leader很忙变成follower很忙，如果集群台数不多，leader 的CPU和IO也不高的话，就使用默认值。  
　　　　(4)server.x=[hostname]:nnnnn[:nnnnn][:observer]：
　　　　　　　　这里的x是一个数字，与myid文件中的id是一致的。接着可以配置两个端口，第一个端口 用于leader和follower之间的数据同步和其它通信，第二个端口用于leader选举过程中投票通信。另外可以在最后加上:observer标记，用于表示该服务器以observer模式运行。  
　　　　(5)syncLimit：
　　　　　　　　follower后续与leader同步的超时值，单位为tickTime的倍数。该参数与initLimit不同之处是，与数据量大小关系不大，只取决于网络质量，因此还起到心跳作用，如果follower超时了，leader就认为该follower落后太多需要放弃。如果网络为高延迟环境，可适当增加该值，但不宜加得太大，否则会掩盖某些故障。 
　　　　(6)group.x=nnnnn[:nnnnn]：
　　　　　　　　仲裁时除了默认的按每机1票计算，还有加权的方法和分组的方法，这个参数就是用来设置分组方法的。  
　　　　(7)weight.x=nnnnn：
　　　　　　　　见上一行。  
　　　　(8)cnxTimeout：
　　　　　　　　用于设置群首选举时打开一个新的连接的超时值，单位为毫秒，默认值为5000，不需要修改。  
　　　　(9)standaloneEnabled：
　　　　　　　　3.5.0及以后才有，默认为true，是为了向后兼容 的目的。如果设为false，单个server可以以集群模式启动，一个集群 可以通过重新配置降回单个node，也可以从单个node升回一个集群。  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# cat /home/softwares/apache-zookeeper-3.5.5-bin/conf/zoo.cfg 
# 滴答，计时的基本单位，默认是2000毫秒，即2秒。它是zookeeper最小的时间单位，用于丈量心跳时间和超时时间等，通常设置成默认2秒即可。
tickTime=2000

# 初始化限制是10滴答，默认是10个滴答，即默认是20秒。指定follower节点初始化是链接leader节点的最大tick次数。
initLimit=5

# 数据同步的时间限制，默认是5个滴答，即默认时间是10秒。设定了follower节点与leader节点进行同步的最大时间。与initLimit类似，它也是以tickTime为单位进行指定的。
syncLimit=2

# 指定zookeeper的工作目录，这是一个非常重要的参数，zookeeper会在内存中在内存只能中保存系统快照，并定期写入该路径指定的文件夹中。生产环境中需要注意该文件夹的磁盘占用情况。
dataDir=/home/data/zookeeper

# 监听zookeeper的默认端口。zookeeper监听客户端链接的端口，一般设置成默认2181即可。
clientPort=2181

# 这个操作将限制连接到 ZooKeeper 的客户端的数量，限制并发连接的数量，它通过 IP 来区分不同的客户端。此配置选项可以用来阻止某些类别的 Dos 攻击。将它设置为 0 或者忽略而不进行设置将会取消对并发连接的限制。
#maxClientCnxns=60
 
# 在上文中已经提到，3.4.0及之后版本，ZK提供了自动清理事务日志和快照文件的功能，这个参数指定了清理频率，单位是小时，需要配置一个1或更大的整数，默认是0，表示不开启自动清理功能。
#autopurge.purgeInterval=1

# 这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。
#autopurge.snapRetainCount=3

#server.x=[hostname]:nnnnn[:nnnnn]，这里的x是一个数字，与myid文件中的id是一致的。右边可以配置两个端口，第一个端口用于F和L之间的数据同步和其它通信，第二个端口用于Leader选举过程中投票通信。  
server.106=node106.yinzhengjie.org.cn:2888:3888
server.107=node107.yinzhengjie.org.cn:2888:3888
server.108=node108.yinzhengjie.org.cn:2888:3888
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# cat /home/softwares/apache-zookeeper-3.5.5-bin/conf/zoo.cfg

**5>.编写zookeeper的启动脚本**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# vi /usr/local/bin/zookeeper_manager.sh 
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# cat /usr/local/bin/zookeeper_manager.sh 
#!/bin/bash
#@author :yinzhengjie
#blog:http://www.cnblogs.com/yinzhengjie
#EMAIL:y1053419035@qq.com

#判断用户是否传参
if [ $# -ne 1 ];then
    echo "无效参数，用法为: $0  {start|stop|restart|status}"
    exit
fi

#获取用户输入的命令
cmd=$1

#定义函数功能
function zookeeperManger(){
    case $cmd in
    start)
        echo "启动服务"        
        remoteExecution start
        ;;
    stop)
        echo "停止服务"
        remoteExecution stop
        ;;
    restart)
        echo "重启服务"
        remoteExecution restart
        ;;
    status)
        echo "查看状态"
        remoteExecution status
        ;;
    *)
        echo "无效参数，用法为: $0  {start|stop|restart|status}"
        ;;
    esac
}


#定义执行的命令
function remoteExecution(){
    for (( i=106 ; i<=108 ; i++ )) ; do
            tput setaf 2
            echo ========== node${i}.yinzhengjie.org.cn zkServer.sh  $1 ================
            tput setaf 9
            ssh node${i}.yinzhengjie.org.cn  "source /etc/profile ; zkServer.sh $1"
    done
}

#调用函数
zookeeperManger
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# chmod +x /usr/local/bin/zookeeper_manager.sh
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ll /usr/local/bin/zookeeper_manager.sh 
-rwxr-xr-x 1 root root 1101 Jul 10 19:04 /usr/local/bin/zookeeper_manager.sh
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]#
```

\[root@node106.yinzhengjie.org.cn ~\]# cat /usr/local/bin/zookeeper\_manager.sh

****6>.配置zookeeper的环境变量****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node106.yinzhengjie.org.cn ~]# tail -3  /etc/profile
#ADD Zookeeper PATH BY yinzhengjie
ZOOKEEPER=/home/softwares/apache-zookeeper-3.5.5-bin
PATH=$PATH:$ZOOKEEPER/bin
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# source /etc/profile
[root@node106.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

****7>.配置管理节点与其他节点免密登录****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:1f6hE914tCJFEZoCoRk9OJGivY76rt+sSCjtmsXphAw root@node106.yinzhengjie.org.cn
The key's randomart image is:
+---[RSA 2048]----+
|     o=o.    +o  |
|   . ++o.  .+    |
|  o .o. ...o..  .|
| . .     ..... +.|
|E   .   S  .o.+.o|
|++ o        .+.o |
|+oO         o .  |
|oO +         .   |
|B*B.o            |
+----[SHA256]-----+
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# ssh-keygen -t rsa -P '' -f ~/.ssh/id\_rsa

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ssh-copy-id node106.yinzhengjie.org.cn
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'node106.yinzhengjie.org.cn (10.0.2.15)' can't be established.
ECDSA key fingerprint is SHA256:EN9dVOkMLQ/20rdlpb+dDXmI8Os43C51nuDxSc61laU.
ECDSA key fingerprint is MD5:c6:ce:a2:ea:01:5c:fb:f5:5a:56:4a:24:bb:d6:2a:7d.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@node106.yinzhengjie.org.cn's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'node106.yinzhengjie.org.cn'"
and check to make sure that only the key(s) you wanted were added.

[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# ssh-copy-id node106.yinzhengjie.org.cn

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ssh-copy-id node107.yinzhengjie.org.cn
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'node107.yinzhengjie.org.cn (172.30.1.107)' can't be established.
ECDSA key fingerprint is SHA256:gMWo+eGV/qjyhw7zK6eLCt4LYPFo1lAYLF56DoYcIuI.
ECDSA key fingerprint is MD5:9d:7d:6e:d7:7a:5a:bd:de:30:8a:20:ea:41:cf:0d:06.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@node107.yinzhengjie.org.cn's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'node107.yinzhengjie.org.cn'"
and check to make sure that only the key(s) you wanted were added.

[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# ssh-copy-id node107.yinzhengjie.org.cn

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ssh-copy-id node108.yinzhengjie.org.cn
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'node108.yinzhengjie.org.cn (172.30.1.108)' can't be established.
ECDSA key fingerprint is SHA256:ywuhrLI5g/YAS+OmkHX/YQMC8LoW8cCxuCShYi4F+To.
ECDSA key fingerprint is MD5:cc:ce:62:fe:27:e4:b9:bf:c9:ae:23:0b:d7:14:c0:77.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@node108.yinzhengjie.org.cn's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'node108.yinzhengjie.org.cn'"
and check to make sure that only the key(s) you wanted were added.

[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# ssh-copy-id node108.yinzhengjie.org.cn

****8>.将zookeeper二进制文件及环境变量拷贝至其他节点****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp -r /home/softwares/apache-zookeeper-3.5.5-bin/ node107.yinzhengjie.org.cn:/home/softwares/
......
log4j-1.2.17.LICENSE.txt                                                                                                                                                                                                      100%   11KB  15.3MB/s   00:00    
jline-2.11.LICENSE.txt                                                                                                                                                                                                        100% 1535     2.4MB/s   00:00    
slf4j-1.7.25.LICENSE.txt                                                                                                                                                                                                      100% 1133     1.8MB/s   00:00    
json-simple-1.1.1.LICENSE.txt                                                                                                                                                                                                 100%   11KB  16.1MB/s   00:00    
netty-all-4.1.29.Final.LICENSE.txt                                                                                                                                                                                            100%   11KB  17.2MB/s   00:00    
zookeeper-jute-3.5.5.jar                                                                                                                                                                                                      100%  363KB  76.3MB/s   00:00    
audience-annotations-0.5.0.jar                                                                                                                                                                                                100%   20KB  20.0MB/s   00:00    
zookeeper-3.5.5.jar                                                                                                                                                                                                           100%  956KB  88.3MB/s   00:00    
netty-all-4.1.29.Final.jar                                                                                                                                                                                                    100% 3771KB  68.1MB/s   00:00    
slf4j-api-1.7.25.jar                                                                                                                                                                                                          100%   40KB  34.1MB/s   00:00    
slf4j-log4j12-1.7.25.jar                                                                                                                                                                                                      100%   12KB  18.3MB/s   00:00    
log4j-1.2.17.jar                                                                                                                                                                                                              100%  478KB  83.9MB/s   00:00    
commons-cli-1.2.jar                                                                                                                                                                                                           100%   40KB  16.6MB/s   00:00    
jetty-server-9.4.17.v20190418.jar                                                                                                                                                                                             100%  632KB  84.9MB/s   00:00    
javax.servlet-api-3.1.0.jar                                                                                                                                                                                                   100%   94KB  17.0MB/s   00:00    
jetty-http-9.4.17.v20190418.jar                                                                                                                                                                                               100%  198KB  64.4MB/s   00:00    
jetty-util-9.4.17.v20190418.jar                                                                                                                                                                                               100%  514KB  67.1MB/s   00:00    
jetty-io-9.4.17.v20190418.jar                                                                                                                                                                                                 100%  153KB  51.6MB/s   00:00    
jetty-servlet-9.4.17.v20190418.jar                                                                                                                                                                                            100%  118KB  55.2MB/s   00:00    
jetty-security-9.4.17.v20190418.jar                                                                                                                                                                                           100%  114KB  59.5MB/s   00:00    
jackson-databind-2.9.8.jar                                                                                                                                                                                                    100% 1316KB  70.9MB/s   00:00    
jackson-annotations-2.9.0.jar                                                                                                                                                                                                 100%   65KB  50.5MB/s   00:00    
jackson-core-2.9.8.jar                                                                                                                                                                                                        100%  318KB  75.7MB/s   00:00    
json-simple-1.1.1.jar                                                                                                                                                                                                         100%   23KB  27.1MB/s   00:00    
jline-2.11.jar                                                                                                                                                                                                                100%  204KB  73.1MB/s   00:00    
LICENSE.txt                                                                                                                                                                                                                   100%   11KB  14.1MB/s   00:00    
README.md                                                                                                                                                                                                                     100% 1560     2.6MB/s   00:00    
README_packaging.txt                                                                                                                                                                                                          100% 1347     2.4MB/s   00:00    
NOTICE.txt                                                                                                                                                                                                                    100%  432   798.8KB/s   00:00    
configuration.xsl                                                                                                                                                                                                             100%  535   850.1KB/s   00:00    
zoo_sample.cfg                                                                                                                                                                                                                100%  922     1.5MB/s   00:00    
log4j.properties                                                                                                                                                                                                              100% 2712     3.7MB/s   00:00    
java.env                                                                                                                                                                                                                      100%  259   395.0KB/s   00:00    
zoo.cfg                                                                                                                                                                                                                       100% 2147     3.3MB/s   00:00    
README.txt                                                                                                                                                                                                                    100%  232   435.6KB/s   00:00    
zkTxnLogToolkit.cmd                                                                                                                                                                                                           100%  996     2.0MB/s   00:00    
zkTxnLogToolkit.sh                                                                                                                                                                                                            100% 1385     2.8MB/s   00:00    
zkCli.cmd                                                                                                                                                                                                                     100% 1154     1.9MB/s   00:00    
zkServer.cmd                                                                                                                                                                                                                  100% 1286     1.8MB/s   00:00    
zkEnv.sh                                                                                                                                                                                                                      100% 3690     5.5MB/s   00:00    
zkCleanup.sh                                                                                                                                                                                                                  100% 2067     3.8MB/s   00:00    
zkCli.sh                                                                                                                                                                                                                      100% 1621     3.6MB/s   00:00    
zkEnv.cmd                                                                                                                                                                                                                     100% 1766     4.1MB/s   00:00    
zkServer-initialize.sh                                                                                                                                                                                                        100% 4573     8.5MB/s   00:00    
zkServer.sh                                                                                                                                                                                                                   100% 9372    11.7MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp -r /home/softwares/apache-zookeeper-3.5.5-bin/ node107.yinzhengjie.org.cn:/home/softwares/

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp -r /home/softwares/apache-zookeeper-3.5.5-bin/ node108.yinzhengjie.org.cn:/home/softwares/
......
log4j-1.2.17.LICENSE.txt                                                                                                                                                                                                      100%   11KB  13.4MB/s   00:00    
jline-2.11.LICENSE.txt                                                                                                                                                                                                        100% 1535     3.4MB/s   00:00    
slf4j-1.7.25.LICENSE.txt                                                                                                                                                                                                      100% 1133     2.5MB/s   00:00    
json-simple-1.1.1.LICENSE.txt                                                                                                                                                                                                 100%   11KB   2.8MB/s   00:00    
netty-all-4.1.29.Final.LICENSE.txt                                                                                                                                                                                            100%   11KB  13.8MB/s   00:00    
zookeeper-jute-3.5.5.jar                                                                                                                                                                                                      100%  363KB  81.4MB/s   00:00    
audience-annotations-0.5.0.jar                                                                                                                                                                                                100%   20KB  20.5MB/s   00:00    
zookeeper-3.5.5.jar                                                                                                                                                                                                           100%  956KB  88.4MB/s   00:00    
netty-all-4.1.29.Final.jar                                                                                                                                                                                                    100% 3771KB  71.5MB/s   00:00    
slf4j-api-1.7.25.jar                                                                                                                                                                                                          100%   40KB  38.7MB/s   00:00    
slf4j-log4j12-1.7.25.jar                                                                                                                                                                                                      100%   12KB  15.3MB/s   00:00    
log4j-1.2.17.jar                                                                                                                                                                                                              100%  478KB  83.8MB/s   00:00    
commons-cli-1.2.jar                                                                                                                                                                                                           100%   40KB  38.8MB/s   00:00    
jetty-server-9.4.17.v20190418.jar                                                                                                                                                                                             100%  632KB  86.0MB/s   00:00    
javax.servlet-api-3.1.0.jar                                                                                                                                                                                                   100%   94KB  41.8MB/s   00:00    
jetty-http-9.4.17.v20190418.jar                                                                                                                                                                                               100%  198KB  30.7MB/s   00:00    
jetty-util-9.4.17.v20190418.jar                                                                                                                                                                                               100%  514KB  52.4MB/s   00:00    
jetty-io-9.4.17.v20190418.jar                                                                                                                                                                                                 100%  153KB  58.1MB/s   00:00    
jetty-servlet-9.4.17.v20190418.jar                                                                                                                                                                                            100%  118KB  57.1MB/s   00:00    
jetty-security-9.4.17.v20190418.jar                                                                                                                                                                                           100%  114KB  58.5MB/s   00:00    
jackson-databind-2.9.8.jar                                                                                                                                                                                                    100% 1316KB  75.6MB/s   00:00    
jackson-annotations-2.9.0.jar                                                                                                                                                                                                 100%   65KB  48.1MB/s   00:00    
jackson-core-2.9.8.jar                                                                                                                                                                                                        100%  318KB  42.1MB/s   00:00    
json-simple-1.1.1.jar                                                                                                                                                                                                         100%   23KB  24.6MB/s   00:00    
jline-2.11.jar                                                                                                                                                                                                                100%  204KB  60.9MB/s   00:00    
LICENSE.txt                                                                                                                                                                                                                   100%   11KB  15.5MB/s   00:00    
README.md                                                                                                                                                                                                                     100% 1560     3.3MB/s   00:00    
README_packaging.txt                                                                                                                                                                                                          100% 1347     2.6MB/s   00:00    
NOTICE.txt                                                                                                                                                                                                                    100%  432   867.6KB/s   00:00    
configuration.xsl                                                                                                                                                                                                             100%  535   795.3KB/s   00:00    
zoo_sample.cfg                                                                                                                                                                                                                100%  922     1.3MB/s   00:00    
log4j.properties                                                                                                                                                                                                              100% 2712     4.6MB/s   00:00    
java.env                                                                                                                                                                                                                      100%  259   596.8KB/s   00:00    
zoo.cfg                                                                                                                                                                                                                       100% 2147     4.0MB/s   00:00    
README.txt                                                                                                                                                                                                                    100%  232   437.5KB/s   00:00    
zkTxnLogToolkit.cmd                                                                                                                                                                                                           100%  996     1.7MB/s   00:00    
zkTxnLogToolkit.sh                                                                                                                                                                                                            100% 1385     1.8MB/s   00:00    
zkCli.cmd                                                                                                                                                                                                                     100% 1154     2.1MB/s   00:00    
zkServer.cmd                                                                                                                                                                                                                  100% 1286     2.4MB/s   00:00    
zkEnv.sh                                                                                                                                                                                                                      100% 3690     7.2MB/s   00:00    
zkCleanup.sh                                                                                                                                                                                                                  100% 2067     3.6MB/s   00:00    
zkCli.sh                                                                                                                                                                                                                      100% 1621     3.1MB/s   00:00    
zkEnv.cmd                                                                                                                                                                                                                     100% 1766     3.3MB/s   00:00    
zkServer-initialize.sh                                                                                                                                                                                                        100% 4573     5.7MB/s   00:00    
zkServer.sh                                                                                                                                                                                                                   100% 9372    12.9MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp -r /home/softwares/apache-zookeeper-3.5.5-bin/ node108.yinzhengjie.org.cn:/home/softwares/

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp /etc/profile node107.yinzhengjie.org.cn:/etc/profile
profile                                                                                                                                                                                                                       100% 2016     2.8MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp /etc/profile node107.yinzhengjie.org.cn:/etc/profile

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp /etc/profile node108.yinzhengjie.org.cn:/etc/profile
profile                                                                                                                                                                                                                       100% 2016     2.3MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp /etc/profile node108.yinzhengjie.org.cn:/etc/profile

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.30.1.101 node101.yinzhengjie.org.cn
172.30.1.102 node102.yinzhengjie.org.cn
172.30.1.103 node103.yinzhengjie.org.cn
172.30.1.104 node104.yinzhengjie.org.cn
172.30.1.105 node105.yinzhengjie.org.cn
172.30.1.106 node106.yinzhengjie.org.cn
172.30.1.107 node107.yinzhengjie.org.cn
172.30.1.108 node108.yinzhengjie.org.cn
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# cat /etc/hosts

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp /etc/hosts node107.yinzhengjie.org.cn:/etc/hosts
hosts                                                                                                                                                                                                                         100%  479   284.7KB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp /etc/hosts node107.yinzhengjie.org.cn:/etc/hosts

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp /etc/hosts node108.yinzhengjie.org.cn:/etc/hosts
hosts                                                                                                                                                                                                                         100%  479   352.2KB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp /etc/hosts node108.yinzhengjie.org.cn:/etc/hosts

****9>.创建数据目录并生产myid文件****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# yum -y install ansible
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package ansible.noarch 0:2.4.2.0-2.el7 will be installed
--> Processing Dependency: sshpass for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python2-jmespath for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-six for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-setuptools for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-passlib for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-paramiko for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-jinja2 for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-httplib2 for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: python-cryptography for package: ansible-2.4.2.0-2.el7.noarch
--> Processing Dependency: PyYAML for package: ansible-2.4.2.0-2.el7.noarch
--> Running transaction check
---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
--> Processing Dependency: libyaml-0.so.2()(64bit) for package: PyYAML-3.10-11.el7.x86_64
---> Package python-httplib2.noarch 0:0.9.2-1.el7 will be installed
---> Package python-jinja2.noarch 0:2.7.2-3.el7_6 will be installed
--> Processing Dependency: python-babel >= 0.8 for package: python-jinja2-2.7.2-3.el7_6.noarch
--> Processing Dependency: python-markupsafe for package: python-jinja2-2.7.2-3.el7_6.noarch
---> Package python-paramiko.noarch 0:2.1.1-9.el7 will be installed
--> Processing Dependency: python2-pyasn1 for package: python-paramiko-2.1.1-9.el7.noarch
---> Package python-passlib.noarch 0:1.6.5-2.el7 will be installed
---> Package python-setuptools.noarch 0:0.9.8-7.el7 will be installed
--> Processing Dependency: python-backports-ssl_match_hostname for package: python-setuptools-0.9.8-7.el7.noarch
---> Package python-six.noarch 0:1.9.0-2.el7 will be installed
---> Package python2-cryptography.x86_64 0:1.7.2-2.el7 will be installed
--> Processing Dependency: python-idna >= 2.0 for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-cffi >= 1.4.1 for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-ipaddress for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-enum34 for package: python2-cryptography-1.7.2-2.el7.x86_64
---> Package python2-jmespath.noarch 0:0.9.0-3.el7 will be installed
---> Package sshpass.x86_64 0:1.06-2.el7 will be installed
--> Running transaction check
---> Package libyaml.x86_64 0:0.1.4-11.el7_0 will be installed
---> Package python-babel.noarch 0:0.9.6-8.el7 will be installed
---> Package python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7 will be installed
--> Processing Dependency: python-backports for package: python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch
---> Package python-cffi.x86_64 0:1.6.0-5.el7 will be installed
--> Processing Dependency: python-pycparser for package: python-cffi-1.6.0-5.el7.x86_64
---> Package python-enum34.noarch 0:1.0.4-1.el7 will be installed
---> Package python-idna.noarch 0:2.4-1.el7 will be installed
---> Package python-ipaddress.noarch 0:1.0.16-2.el7 will be installed
---> Package python-markupsafe.x86_64 0:0.11-10.el7 will be installed
---> Package python2-pyasn1.noarch 0:0.1.9-7.el7 will be installed
--> Running transaction check
---> Package python-backports.x86_64 0:1.0-8.el7 will be installed
---> Package python-pycparser.noarch 0:2.14-1.el7 will be installed
--> Processing Dependency: python-ply for package: python-pycparser-2.14-1.el7.noarch
--> Running transaction check
---> Package python-ply.noarch 0:3.4-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================================================================================================================================================================================================
 Package                                                                          Arch                                                Version                                                        Repository                                            Size
================================================================================================================================================================================================================================================================
Installing:
 ansible                                                                          noarch                                              2.4.2.0-2.el7                                                  extras                                               7.6 M
Installing for dependencies:
 PyYAML                                                                           x86_64                                              3.10-11.el7                                                    base                                                 153 k
 libyaml                                                                          x86_64                                              0.1.4-11.el7_0                                                 base                                                  55 k
 python-babel                                                                     noarch                                              0.9.6-8.el7                                                    base                                                 1.4 M
 python-backports                                                                 x86_64                                              1.0-8.el7                                                      base                                                 5.8 k
 python-backports-ssl_match_hostname                                              noarch                                              3.5.0.1-1.el7                                                  base                                                  13 k
 python-cffi                                                                      x86_64                                              1.6.0-5.el7                                                    base                                                 218 k
 python-enum34                                                                    noarch                                              1.0.4-1.el7                                                    base                                                  52 k
 python-httplib2                                                                  noarch                                              0.9.2-1.el7                                                    extras                                               115 k
 python-idna                                                                      noarch                                              2.4-1.el7                                                      base                                                  94 k
 python-ipaddress                                                                 noarch                                              1.0.16-2.el7                                                   base                                                  34 k
 python-jinja2                                                                    noarch                                              2.7.2-3.el7_6                                                  updates                                              518 k
 python-markupsafe                                                                x86_64                                              0.11-10.el7                                                    base                                                  25 k
 python-paramiko                                                                  noarch                                              2.1.1-9.el7                                                    updates                                              269 k
 python-passlib                                                                   noarch                                              1.6.5-2.el7                                                    extras                                               488 k
 python-ply                                                                       noarch                                              3.4-11.el7                                                     base                                                 123 k
 python-pycparser                                                                 noarch                                              2.14-1.el7                                                     base                                                 104 k
 python-setuptools                                                                noarch                                              0.9.8-7.el7                                                    base                                                 397 k
 python-six                                                                       noarch                                              1.9.0-2.el7                                                    base                                                  29 k
 python2-cryptography                                                             x86_64                                              1.7.2-2.el7                                                    base                                                 502 k
 python2-jmespath                                                                 noarch                                              0.9.0-3.el7                                                    extras                                                39 k
 python2-pyasn1                                                                   noarch                                              0.1.9-7.el7                                                    base                                                 100 k
 sshpass                                                                          x86_64                                              1.06-2.el7                                                     extras                                                21 k

Transaction Summary
================================================================================================================================================================================================================================================================
Install  1 Package (+22 Dependent packages)

Total download size: 12 M
Installed size: 60 M
Downloading packages:
(1/23): PyYAML-3.10-11.el7.x86_64.rpm                                                                                                                                                                                                    | 153 kB  00:00:00     
(2/23): python-backports-1.0-8.el7.x86_64.rpm                                                                                                                                                                                            | 5.8 kB  00:00:00     
(3/23): python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch.rpm                                                                                                                                                                     |  13 kB  00:00:00     
(4/23): libyaml-0.1.4-11.el7_0.x86_64.rpm                                                                                                                                                                                                |  55 kB  00:00:00     
(5/23): python-enum34-1.0.4-1.el7.noarch.rpm                                                                                                                                                                                             |  52 kB  00:00:00     
(6/23): python-idna-2.4-1.el7.noarch.rpm                                                                                                                                                                                                 |  94 kB  00:00:00     
(7/23): python-ipaddress-1.0.16-2.el7.noarch.rpm                                                                                                                                                                                         |  34 kB  00:00:00     
(8/23): python-httplib2-0.9.2-1.el7.noarch.rpm                                                                                                                                                                                           | 115 kB  00:00:00     
(9/23): python-cffi-1.6.0-5.el7.x86_64.rpm                                                                                                                                                                                               | 218 kB  00:00:01     
(10/23): python-jinja2-2.7.2-3.el7_6.noarch.rpm                                                                                                                                                                                          | 518 kB  00:00:00     
(11/23): python-markupsafe-0.11-10.el7.x86_64.rpm                                                                                                                                                                                        |  25 kB  00:00:00     
(12/23): python-babel-0.9.6-8.el7.noarch.rpm                                                                                                                                                                                             | 1.4 MB  00:00:02     
(13/23): python-ply-3.4-11.el7.noarch.rpm                                                                                                                                                                                                | 123 kB  00:00:00     
(14/23): python-pycparser-2.14-1.el7.noarch.rpm                                                                                                                                                                                          | 104 kB  00:00:00     
(15/23): python-six-1.9.0-2.el7.noarch.rpm                                                                                                                                                                                               |  29 kB  00:00:00     
(16/23): python-setuptools-0.9.8-7.el7.noarch.rpm                                                                                                                                                                                        | 397 kB  00:00:00     
(17/23): python-passlib-1.6.5-2.el7.noarch.rpm                                                                                                                                                                                           | 488 kB  00:00:00     
(18/23): python2-jmespath-0.9.0-3.el7.noarch.rpm                                                                                                                                                                                         |  39 kB  00:00:00     
(19/23): sshpass-1.06-2.el7.x86_64.rpm                                                                                                                                                                                                   |  21 kB  00:00:00     
(20/23): python2-cryptography-1.7.2-2.el7.x86_64.rpm                                                                                                                                                                                     | 502 kB  00:00:00     
(21/23): python-paramiko-2.1.1-9.el7.noarch.rpm                                                                                                                                                                                          | 269 kB  00:00:01     
(22/23): python2-pyasn1-0.1.9-7.el7.noarch.rpm                                                                                                                                                                                           | 100 kB  00:00:01     
(23/23): ansible-2.4.2.0-2.el7.noarch.rpm                                                                                                                                                                                                | 7.6 MB  00:00:08     
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                           1.4 MB/s |  12 MB  00:00:08     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python2-pyasn1-0.1.9-7.el7.noarch                                                                                                                                                                                                           1/23 
  Installing : python-ipaddress-1.0.16-2.el7.noarch                                                                                                                                                                                                        2/23 
  Installing : python-six-1.9.0-2.el7.noarch                                                                                                                                                                                                               3/23 
  Installing : python-httplib2-0.9.2-1.el7.noarch                                                                                                                                                                                                          4/23 
  Installing : python-enum34-1.0.4-1.el7.noarch                                                                                                                                                                                                            5/23 
  Installing : libyaml-0.1.4-11.el7_0.x86_64                                                                                                                                                                                                               6/23 
  Installing : PyYAML-3.10-11.el7.x86_64                                                                                                                                                                                                                   7/23 
  Installing : python-backports-1.0-8.el7.x86_64                                                                                                                                                                                                           8/23 
  Installing : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch                                                                                                                                                                                    9/23 
  Installing : python-setuptools-0.9.8-7.el7.noarch                                                                                                                                                                                                       10/23 
  Installing : python-babel-0.9.6-8.el7.noarch                                                                                                                                                                                                            11/23 
  Installing : python-passlib-1.6.5-2.el7.noarch                                                                                                                                                                                                          12/23 
  Installing : python-ply-3.4-11.el7.noarch                                                                                                                                                                                                               13/23 
  Installing : python-pycparser-2.14-1.el7.noarch                                                                                                                                                                                                         14/23 
  Installing : python-cffi-1.6.0-5.el7.x86_64                                                                                                                                                                                                             15/23 
  Installing : python-markupsafe-0.11-10.el7.x86_64                                                                                                                                                                                                       16/23 
  Installing : python-jinja2-2.7.2-3.el7_6.noarch                                                                                                                                                                                                         17/23 
  Installing : python-idna-2.4-1.el7.noarch                                                                                                                                                                                                               18/23 
  Installing : python2-cryptography-1.7.2-2.el7.x86_64                                                                                                                                                                                                    19/23 
  Installing : python-paramiko-2.1.1-9.el7.noarch                                                                                                                                                                                                         20/23 
  Installing : sshpass-1.06-2.el7.x86_64                                                                                                                                                                                                                  21/23 
  Installing : python2-jmespath-0.9.0-3.el7.noarch                                                                                                                                                                                                        22/23 
  Installing : ansible-2.4.2.0-2.el7.noarch                                                                                                                                                                                                               23/23 
  Verifying  : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch                                                                                                                                                                                    1/23 
  Verifying  : python2-jmespath-0.9.0-3.el7.noarch                                                                                                                                                                                                         2/23 
  Verifying  : sshpass-1.06-2.el7.x86_64                                                                                                                                                                                                                   3/23 
  Verifying  : python-setuptools-0.9.8-7.el7.noarch                                                                                                                                                                                                        4/23 
  Verifying  : python-jinja2-2.7.2-3.el7_6.noarch                                                                                                                                                                                                          5/23 
  Verifying  : python-six-1.9.0-2.el7.noarch                                                                                                                                                                                                               6/23 
  Verifying  : python-idna-2.4-1.el7.noarch                                                                                                                                                                                                                7/23 
  Verifying  : python-markupsafe-0.11-10.el7.x86_64                                                                                                                                                                                                        8/23 
  Verifying  : python-ply-3.4-11.el7.noarch                                                                                                                                                                                                                9/23 
  Verifying  : python-passlib-1.6.5-2.el7.noarch                                                                                                                                                                                                          10/23 
  Verifying  : python-paramiko-2.1.1-9.el7.noarch                                                                                                                                                                                                         11/23 
  Verifying  : python-babel-0.9.6-8.el7.noarch                                                                                                                                                                                                            12/23 
  Verifying  : python-backports-1.0-8.el7.x86_64                                                                                                                                                                                                          13/23 
  Verifying  : python-cffi-1.6.0-5.el7.x86_64                                                                                                                                                                                                             14/23 
  Verifying  : python-pycparser-2.14-1.el7.noarch                                                                                                                                                                                                         15/23 
  Verifying  : libyaml-0.1.4-11.el7_0.x86_64                                                                                                                                                                                                              16/23 
  Verifying  : ansible-2.4.2.0-2.el7.noarch                                                                                                                                                                                                               17/23 
  Verifying  : python-ipaddress-1.0.16-2.el7.noarch                                                                                                                                                                                                       18/23 
  Verifying  : python-enum34-1.0.4-1.el7.noarch                                                                                                                                                                                                           19/23 
  Verifying  : python-httplib2-0.9.2-1.el7.noarch                                                                                                                                                                                                         20/23 
  Verifying  : python2-pyasn1-0.1.9-7.el7.noarch                                                                                                                                                                                                          21/23 
  Verifying  : PyYAML-3.10-11.el7.x86_64                                                                                                                                                                                                                  22/23 
  Verifying  : python2-cryptography-1.7.2-2.el7.x86_64                                                                                                                                                                                                    23/23 

Installed:
  ansible.noarch 0:2.4.2.0-2.el7                                                                                                                                                                                                                                

Dependency Installed:
  PyYAML.x86_64 0:3.10-11.el7               libyaml.x86_64 0:0.1.4-11.el7_0       python-babel.noarch 0:0.9.6-8.el7   python-backports.x86_64 0:1.0-8.el7    python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7 python-cffi.x86_64 0:1.6.0-5.el7      
  python-enum34.noarch 0:1.0.4-1.el7        python-httplib2.noarch 0:0.9.2-1.el7  python-idna.noarch 0:2.4-1.el7      python-ipaddress.noarch 0:1.0.16-2.el7 python-jinja2.noarch 0:2.7.2-3.el7_6                       python-markupsafe.x86_64 0:0.11-10.el7
  python-paramiko.noarch 0:2.1.1-9.el7      python-passlib.noarch 0:1.6.5-2.el7   python-ply.noarch 0:3.4-11.el7      python-pycparser.noarch 0:2.14-1.el7   python-setuptools.noarch 0:0.9.8-7.el7                     python-six.noarch 0:1.9.0-2.el7       
  python2-cryptography.x86_64 0:1.7.2-2.el7 python2-jmespath.noarch 0:0.9.0-3.el7 python2-pyasn1.noarch 0:0.1.9-7.el7 sshpass.x86_64 0:1.06-2.el7           

Complete!
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# yum -y install ansible

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node106.yinzhengjie.org.cn ~]# tail -2 /etc/ansible/hosts 
[zk]
node[106:108].yinzhengjie.org.cn
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ansible --list-hosts zk
  hosts (3):
    node106.yinzhengjie.org.cn
    node107.yinzhengjie.org.cn
    node108.yinzhengjie.org.cn
[root@node106.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ansible zk -m shell -a 'mkdir -pv /home/data/zookeeper'
 [WARNING]: Consider using file module with state=directory rather than running mkdir

node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
mkdir: created directory ‘/home/data’
mkdir: created directory ‘/home/data/zookeeper’

node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
mkdir: created directory ‘/home/data’
mkdir: created directory ‘/home/data/zookeeper’

node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
mkdir: created directory ‘/home/data’
mkdir: created directory ‘/home/data/zookeeper’

[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# ansible zk -m shell -a 'mkdir -pv /home/data/zookeeper'

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node106.yinzhengjie.org.cn ~]# for (( i=106;i<=108;i++ )) do ssh node${i}.yinzhengjie.org.cn "echo -n $i > /home/data/zookeeper/myid" ;done
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ansible zk -m shell -a 'cat /home/data/zookeeper/myid'
node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
108

node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
107

node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
106

[root@node106.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

****10>.**启动zookeeper并查看状态******

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# zookeeper_manager.sh start
启动服务
========== node106.yinzhengjie.org.cn zkServer.sh start ================
ZooKeeper JMX enabled by default
Using config: /home/softwares/apache-zookeeper-3.5.5-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
========== node107.yinzhengjie.org.cn zkServer.sh start ================
ZooKeeper JMX enabled by default
Using config: /home/softwares/apache-zookeeper-3.5.5-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
========== node108.yinzhengjie.org.cn zkServer.sh start ================
ZooKeeper JMX enabled by default
Using config: /home/softwares/apache-zookeeper-3.5.5-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# zookeeper\_manager.sh start

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# zookeeper_manager.sh status
查看状态
========== node106.yinzhengjie.org.cn zkServer.sh status ================
ZooKeeper JMX enabled by default
Using config: /home/softwares/apache-zookeeper-3.5.5-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
========== node107.yinzhengjie.org.cn zkServer.sh status ================
ZooKeeper JMX enabled by default
Using config: /home/softwares/apache-zookeeper-3.5.5-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
========== node108.yinzhengjie.org.cn zkServer.sh status ================
ZooKeeper JMX enabled by default
Using config: /home/softwares/apache-zookeeper-3.5.5-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# zookeeper\_manager.sh status

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ansible zk -m shell -a 'jps'
node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
3859 QuorumPeerMain
4020 Jps

node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
4055 Jps
3887 QuorumPeerMain

node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
4000 QuorumPeerMain
4218 Jps

[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# ansible zk -m shell -a 'jps'

**二.使用zkWeb健康zookeeper状态**

**1>.下载“ZkWeb For Zookeeper”的jar包(**下载地址：[https://github.com/zhitom/zkweb/releases](https://github.com/zhitom/zkweb/releases)。**)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# wget https://github.com/zhitom/zkweb/releases/download/zkWeb-v1.2.1/zkWeb-v1.2.1.jar
--2019-07-11 11:11:19--  https://github.com/zhitom/zkweb/releases/download/zkWeb-v1.2.1/zkWeb-v1.2.1.jar
Resolving github.com (github.com)... 52.74.223.119
Connecting to github.com (github.com)|52.74.223.119|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/138690727/1450b800-5d72-11e9-9cd3-d62def1384d2?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20190711%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190711T0
31121Z&X-Amz-Expires=300&X-Amz-Signature=e3e489bba28072ce240b99ff7745d5a55baf9831669e3ef7111c41ffa5d47fc8&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3DzkWeb-v1.2.1.jar&response-content-type=application%2Foctet-stream [following]--2019-07-11 11:11:20--  https://github-production-release-asset-2e65be.s3.amazonaws.com/138690727/1450b800-5d72-11e9-9cd3-d62def1384d2?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20190711%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-
Date=20190711T031121Z&X-Amz-Expires=300&X-Amz-Signature=e3e489bba28072ce240b99ff7745d5a55baf9831669e3ef7111c41ffa5d47fc8&X-Amz-SignedHeaders=host&actor_id=0&response-content-disposition=attachment%3B%20filename%3DzkWeb-v1.2.1.jar&response-content-type=application%2Foctet-streamResolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)... 54.231.32.83
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e65be.s3.amazonaws.com)|54.231.32.83|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 28541920 (27M) [application/octet-stream]
Saving to: ‘zkWeb-v1.2.1.jar’

100%[======================================================================================================================================================================================================================>] 28,541,920  33.4KB/s   in 14m 34s

2019-07-11 11:25:56 (31.9 KB/s) - ‘zkWeb-v1.2.1.jar’ saved [28541920/28541920]

[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# wget https://github.com/zhitom/zkweb/releases/download/zkWeb-v1.2.1/zkWeb-v1.2.1.jar

**![](https://img2018.cnblogs.com/blog/795254/201907/795254-20190711111031753-1077637047.png)**

****2>.运行ZkWeb的jar包（默认启动端口是8099）****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# java -jar zkWeb-v1.2.1.jar 
11:41:54.393 [main] INFO com.yasenagat.zkweb.ZkWebSpringBootApplication - applicationYamlFileName(application-zkweb.yaml)=file:/root/zkWeb-v1.2.1.jar!/BOOT-INF/classes!/application-zkweb.yaml

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.2.RELEASE)

[2019-07-11 11:41:56 INFO  main  StartupInfoLogger.java:50] c.y.zkweb.ZkWebSpringBootApplication --> Starting ZkWebSpringBootApplication vv1.2.1 on node106.yinzhengjie.org.cn with PID 4470 (/root/zkWeb-v1.2.1.jar started by root in /root)
[2019-07-11 11:41:56 INFO  main  SpringApplication.java:663] c.y.zkweb.ZkWebSpringBootApplication --> The following profiles are active: local
[2019-07-11 11:41:56 INFO  main  AbstractApplicationContext.java:590] o.s.b.w.s.c.AnnotationConfigServletWebServerApplicationContext --> Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@30c7da1e: st
artup date [Thu Jul 11 11:41:56 CST 2019]; root of context hierarchy[2019-07-11 11:41:58 INFO  main  DefaultListableBeanFactory.java:824] o.s.b.f.s.DefaultListableBeanFactory --> Overriding bean definition for bean 'requestMappingHandlerAdapter' with a different definition: replacing [Root bean: class [null]; scope=; abstr
act=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=zkSpringBootConfiguration; factoryMethodName=requestMappingHandlerAdapter; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/yasenagat/zkweb/util/ZkSpringBootConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration; factoryMethodName=requestMappingHandlerAdapter; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]][2019-07-11 11:41:59 INFO  main  TomcatWebServer.java:91] o.s.b.w.e.tomcat.TomcatWebServer --> Tomcat initialized with port(s): 8099 (http)
[2019-07-11 11:41:59 INFO  main  DirectJDKLog.java:180] o.a.coyote.http11.Http11NioProtocol --> Initializing ProtocolHandler ["http-nio-8099"]
[2019-07-11 11:41:59 INFO  main  DirectJDKLog.java:180] o.a.catalina.core.StandardService --> Starting service [Tomcat]
[2019-07-11 11:41:59 INFO  main  DirectJDKLog.java:180] o.a.catalina.core.StandardEngine --> Starting Servlet Engine: Apache Tomcat/8.5.31
[2019-07-11 11:41:59 INFO  localhost-startStop-1  DirectJDKLog.java:180] o.a.c.core.AprLifecycleListener --> The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/us
r/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib][2019-07-11 11:42:00 INFO  localhost-startStop-1  DirectJDKLog.java:180] o.a.c.c.C.[Tomcat].[localhost].[/] --> Initializing Spring embedded WebApplicationContext
[2019-07-11 11:42:00 INFO  localhost-startStop-1  ServletWebServerApplicationContext.java:285] o.s.web.context.ContextLoader --> Root WebApplicationContext: initialization completed in 3493 ms
[2019-07-11 11:42:00 INFO  localhost-startStop-1  ServletRegistrationBean.java:185] o.s.b.w.s.ServletRegistrationBean --> Servlet dispatcherServlet mapped to [/]
[2019-07-11 11:42:00 INFO  localhost-startStop-1  ServletRegistrationBean.java:185] o.s.b.w.s.ServletRegistrationBean --> Servlet webServlet mapped to [/console/*]
[2019-07-11 11:42:00 INFO  localhost-startStop-1  ServletRegistrationBean.java:185] o.s.b.w.s.ServletRegistrationBean --> Servlet cacheServlet mapped to [/cache/*]
[2019-07-11 11:42:00 INFO  localhost-startStop-1  AbstractFilterRegistrationBean.java:244] o.s.b.w.s.FilterRegistrationBean --> Mapping filter: 'characterEncodingFilter' to: [/*]
[2019-07-11 11:42:00 INFO  localhost-startStop-1  AbstractFilterRegistrationBean.java:244] o.s.b.w.s.FilterRegistrationBean --> Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
[2019-07-11 11:42:00 INFO  localhost-startStop-1  AbstractFilterRegistrationBean.java:244] o.s.b.w.s.FilterRegistrationBean --> Mapping filter: 'httpPutFormContentFilter' to: [/*]
[2019-07-11 11:42:00 INFO  localhost-startStop-1  AbstractFilterRegistrationBean.java:244] o.s.b.w.s.FilterRegistrationBean --> Mapping filter: 'requestContextFilter' to: [/*]
[2019-07-11 11:42:00 INFO  MLog-Init-Reporter  Slf4jMLog.java:212] com.mchange.v2.log.MLog --> MLog clients using slf4j logging.
[2019-07-11 11:42:00 INFO  main  Slf4jMLog.java:212] com.mchange.v2.c3p0.C3P0Registry --> Initializing c3p0-0.9.5.2 [built 08-December-2015 22:06:04 -0800; debug? true; trace: 10]
[2019-07-11 11:42:00 INFO  main  Slf4jMLog.java:212] c.m.v.c.i.AbstractPoolBackedDataSource --> Initializing c3p0 pool... com.mchange.v2.c3p0.ComboPooledDataSource [ acquireIncrement -> 3, acquireRetryAttempts -> 30, acquireRetryDelay -> 1000, autoCommitOn
Close -> false, automaticTestTable -> null, breakAfterAcquireFailure -> false, checkoutTimeout -> 0, connectionCustomizerClassName -> null, connectionTesterClassName -> com.mchange.v2.c3p0.impl.DefaultConnectionTester, contextClassLoaderSource -> caller, dataSourceName -> 1br8b56a31pwpvwk1gvtu9d|34b7ac2f, debugUnreturnedConnectionStackTraces -> false, description -> null, driverClass -> org.h2.Driver, extensions -> {}, factoryClassLocation -> null, forceIgnoreUnresolvedTransactions -> false, forceSynchronousCheckins -> false, forceUseNamedDriverClass -> false, identityToken -> 1br8b56a31pwpvwk1gvtu9d|34b7ac2f, idleConnectionTestPeriod -> 60, initialPoolSize -> 10, jdbcUrl -> jdbc:h2:file:~/.h2/zkweb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=TRUE;FILE_LOCK=SOCKET, maxAdministrativeTaskTime -> 0, maxConnectionAge -> 0, maxIdleTime -> 60, maxIdleTimeExcessConnections -> 0, maxPoolSize -> 20, maxStatements -> 100, maxStatementsPerConnection -> 0, minPoolSize -> 5, numHelperThreads -> 3, preferredTestQuery -> null, privilegeSpawnedThreads -> false, properties -> {user=******, password=******}, propertyCycle -> 0, statementCacheNumDeferredCloseThreads -> 0, testConnectionOnCheckin -> false, testConnectionOnCheckout -> false, unreturnedConnectionTimeout -> 0, userOverrides -> {}, usesTraditionalReflectiveProxies -> false ][2019-07-11 11:42:01 ERROR main  ZkCfgManagerImpl.java:406] c.y.zkweb.util.ZkCfgManagerImpl --> isTableOk Failed,A problem occurred while trying to acquire a cached PreparedStatement in a background thread.
[2019-07-11 11:42:01 ERROR main  ZkCfgManagerImpl.java:70] c.y.zkweb.util.ZkCfgManagerImpl --> create table (CREATE TABLE IF NOT EXISTS ZK(ID VARCHAR PRIMARY KEY, DESC VARCHAR, CONNECTSTR VARCHAR, SESSIONTIMEOUT VARCHAR))...
[2019-07-11 11:42:01 ERROR main  ZkCfgManagerImpl.java:76] c.y.zkweb.util.ZkCfgManagerImpl --> create table OK !ret=0
[2019-07-11 11:42:01 ERROR main  ZkCfgManagerImpl.java:81] c.y.zkweb.util.ZkCfgManagerImpl --> table select check OK!
[2019-07-11 11:42:01 INFO  main  ZkCache.java:41] com.yasenagat.zkweb.util.ZkCache --> zk info size=0
[2019-07-11 11:42:01 INFO  main  ZkCfgManagerImpl.java:436] c.y.zkweb.util.ZkCfgManagerImpl -->  afterPropertiesSet init 0 zk instance
[2019-07-11 11:42:01 INFO  main  RequestMappingHandlerAdapter.java:574] o.s.w.s.m.m.a.RequestMappingHandlerAdapter --> Looking for @ControllerAdvice: org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@30c7da1e: 
startup date [Thu Jul 11 11:41:56 CST 2019]; root of context hierarchy[2019-07-11 11:42:02 INFO  main  AbstractUrlHandlerMapping.java:373] o.s.w.s.h.SimpleUrlHandlerMapping --> Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zkcfg/queryZkCfgById]}" onto public java.util.Map<java.lang.String, java.lang.Object> com.yasenagat.zkweb.web.ZkCfgController.
queryZkCfg(java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zkcfg/queryZkCfg]}" onto public java.util.Map<java.lang.String, java.lang.Object> com.yasenagat.zkweb.web.ZkCfgController.quer
yZkCfg(int,int,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zkcfg/addZkCfg],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkCfgController.addZ
kCfg(java.lang.String,java.lang.String,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zkcfg/updateZkCfg],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkCfgController.u
pdateZkCfg(java.lang.String,java.lang.String,java.lang.String,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zkcfg/delZkCfg],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkCfgController.delZ
kCfg(java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/queryZKOk]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkController.queryZKOk(org.springframework.ui.Model,java.
lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/queryZKJMXInfo],produces=[application/json;charset=UTF-8]}" onto public java.util.List<com.yasenagat.zkweb.util.ZkManager$P
ropertyPanel> com.yasenagat.zkweb.web.ZkController.queryZKJMXInfo(java.lang.String,java.lang.String,javax.servlet.http.HttpServletResponse)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/saveData],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkController.saveData(j
ava.lang.String,java.lang.String,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/createNode],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkController.createNo
de(java.lang.String,java.lang.String,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/deleteNode],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkController.deleteNo
de(java.lang.String,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/queryZnodeInfo],produces=[text/html;charset=UTF-8]}" onto public java.lang.String com.yasenagat.zkweb.web.ZkController.quer
yzNodeInfo(java.lang.String,org.springframework.ui.Model,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/zk/queryZnode]}" onto public java.util.List<com.yasenagat.zkweb.model.Tree> com.yasenagat.zkweb.web.ZkController.query(java.la
ng.String,java.lang.String,java.lang.String)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.
web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)[2019-07-11 11:42:02 INFO  main  AbstractHandlerMethodMapping.java:547] o.s.w.s.m.m.a.RequestMappingHandlerMapping --> Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springfram
ework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)[2019-07-11 11:42:02 INFO  main  AbstractUrlHandlerMapping.java:360] o.s.w.s.h.SimpleUrlHandlerMapping --> Root mapping to handler of type [class org.springframework.web.servlet.mvc.ParameterizableViewController]
[2019-07-11 11:42:02 INFO  main  AbstractUrlHandlerMapping.java:373] o.s.w.s.h.SimpleUrlHandlerMapping --> Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2019-07-11 11:42:02 INFO  main  AbstractUrlHandlerMapping.java:373] o.s.w.s.h.SimpleUrlHandlerMapping --> Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2019-07-11 11:42:02 INFO  main  AbstractUrlHandlerMapping.java:373] o.s.w.s.h.SimpleUrlHandlerMapping --> Mapped URL path [/resources/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[2019-07-11 11:42:02 INFO  main  MBeanExporter.java:433] o.s.j.e.a.AnnotationMBeanExporter --> Registering beans for JMX exposure on startup
[2019-07-11 11:42:02 INFO  main  DirectJDKLog.java:180] o.a.coyote.http11.Http11NioProtocol --> Starting ProtocolHandler ["http-nio-8099"]
[2019-07-11 11:42:02 INFO  main  DirectJDKLog.java:180] o.a.tomcat.util.net.NioSelectorPool --> Using a shared selector for servlet write/read
[2019-07-11 11:42:03 INFO  main  DirectJDKLog.java:180] o.a.c.c.C.[Tomcat].[localhost].[/] --> Initializing Spring FrameworkServlet 'dispatcherServlet'
[2019-07-11 11:42:03 INFO  main  FrameworkServlet.java:494] o.s.web.servlet.DispatcherServlet --> FrameworkServlet 'dispatcherServlet': initialization started
[2019-07-11 11:42:03 INFO  main  FrameworkServlet.java:509] o.s.web.servlet.DispatcherServlet --> FrameworkServlet 'dispatcherServlet': initialization completed in 21 ms
[2019-07-11 11:42:03 INFO  main  TomcatWebServer.java:206] o.s.b.w.e.tomcat.TomcatWebServer --> Tomcat started on port(s): 8099 (http) with context path ''
[2019-07-11 11:42:03 INFO  main  StartupInfoLogger.java:59] c.y.zkweb.ZkWebSpringBootApplication --> Started ZkWebSpringBootApplication in 8.271 seconds (JVM running for 9.277)
```

\[root@node106.yinzhengjie.org.cn ~\]# java -jar zkWeb-v1.2.1.jar

![](https://img2018.cnblogs.com/blog/795254/201907/795254-20190711114537826-388560751.png)

**3>.添加节点保存成功**

![](https://img2018.cnblogs.com/blog/795254/201907/795254-20190711114630235-112348105.png)

**4>.通过zookeeper的Web界面查看相应的数据**

![](https://img2018.cnblogs.com/blog/795254/201907/795254-20190711114918636-1608781729.png)

**三.**搭建kafka完全分布式集群****

**1>.官网下载kafka（kafka\_2.11-0.10.2.1.tgz）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# wget https://archive.apache.org/dist/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz
--2018-11-10 01:21:08--  https://archive.apache.org/dist/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz
Resolving archive.apache.org (archive.apache.org)... 163.172.17.199
Connecting to archive.apache.org (archive.apache.org)|163.172.17.199|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 37664956 (36M) [application/x-gzip]
Saving to: ‘kafka_2.11-0.10.2.1.tgz’

100%[====================================================================================================================================================================================================================================>] 37,664,956   228KB/s   in 2m 58s 

2018-11-10 01:24:07 (207 KB/s) - ‘kafka_2.11-0.10.2.1.tgz’ saved [37664956/37664956]

[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ll
total 36784
-rw-r--r-- 1 root root 37664956 Apr 27  2017 kafka_2.11-0.10.2.1.tgz
[root@node106.yinzhengjie.org.cn ~]# 

 
```

\[root@node106.yinzhengjie.org.cn ~\]# wget https://archive.apache.org/dist/kafka/0.10.2.1/kafka\_2.11-0.10.2.1.tgz

****2>.解压kafka并配置环境变量****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# tar -zxf kafka_2.11-0.10.2.1.tgz -C /home/softwares/
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ll /home/softwares/kafka_2.11-0.10.2.1/
total 48
drwxr-xr-x 3 root root  4096 Apr 22  2017 bin
drwxr-xr-x 2 root root  4096 Apr 22  2017 config
drwxr-xr-x 2 root root  4096 Jul 11 14:23 libs
-rw-r--r-- 1 root root 28824 Apr 22  2017 LICENSE
-rw-r--r-- 1 root root   336 Apr 22  2017 NOTICE
drwxr-xr-x 2 root root    47 Apr 22  2017 site-docs
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# tar -zxf kafka\_2.11-0.10.2.1.tgz -C /home/softwares/

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node106.yinzhengjie.org.cn ~]# tail -3 /etc/profile
#ADD kafka PATH BY yinzhengjie
KAFKA_HOME=/home/softwares/kafka_2.11-0.10.2.1
PATH=$PATH:$KAFKA_HOME/bin
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# source /etc/profile
[root@node106.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

****3>********.**修改kafka的启动脚本******

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node106.yinzhengjie.org.cn ~]# cat /home/softwares/kafka_2.11-0.10.2.1/bin/kafka-server-start.sh 
#!/bin/bash
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

if [ $# -lt 1 ];
then
    echo "USAGE: $0 [-daemon] server.properties [--override property=value]*"
    exit 1
fi
base_dir=$(dirname $0)

if [ "x$KAFKA_LOG4J_OPTS" = "x" ]; then
    export KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:$base_dir/../config/log4j.properties"
fi

if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then　　#默认的KAFKA的HEAP内存为1G，在实际生产环境中显然是不够的，在《kafka权威指南》书中说是配置5G，在《Apache Kafka实战》书中说配置6G，其实差距并不是很大，我们这里暂且配置6G吧，当时书中的知识是死的，如果Kafka配置了6G的Heap内存严重发现Full GC的话，到时候我们应该学会变通，将其在扩大，但在实际生产环境中，我就是这样配置的。注意力，这样配置如果你的虚拟机可用内存如果不足6G可能会直接抛出OOM异常哟~
    #export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
    export KAFKA_HEAP_OPTS="-Xmx6g -Xms6g -XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80"
fi

EXTRA_ARGS=${EXTRA_ARGS-'-name kafkaServer -loggc'}

COMMAND=$1
case $COMMAND in
  -daemon)
    EXTRA_ARGS="-daemon "$EXTRA_ARGS
    shift
    ;;
  *)
    ;;
esac
#从这行命令不难看出，该脚本会调用kafka-run-class.sh，如果我们在该配置文件中配置HEAP内存，就不要在Kafka-run-class.sh脚本里再去配置了哟，否则当前脚本配置的HEAP将无效！
exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka "$@"　　　　　　　　　　　　
[root@node106.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.分发kafka相关应用程序**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp /etc/profile node107.yinzhengjie.org.cn:/etc/profile
profile                                                                                                                                                                                                                       100% 2126     3.1MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp /etc/profile node107.yinzhengjie.org.cn:/etc/profile

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp /etc/profile node108.yinzhengjie.org.cn:/etc/profile
profile                                                                                                                                                                                                                       100% 2126     2.8MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp /etc/profile node108.yinzhengjie.org.cn:/etc/profile

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp -r /home/softwares/kafka_2.11-0.10.2.1/ node107.yinzhengjie.org.cn:/home/softwares/
......
jackson-databind-2.8.5.jar                                                                                                                                                                                                    100% 1207KB  83.6MB/s   00:00    
jackson-annotations-2.8.0.jar                                                                                                                                                                                                 100%   54KB  40.5MB/s   00:00    
jackson-core-2.8.5.jar                                                                                                                                                                                                        100%  274KB  32.3MB/s   00:00    
connect-api-0.10.2.1.jar                                                                                                                                                                                                      100%   55KB  35.7MB/s   00:00    
connect-runtime-0.10.2.1.jar                                                                                                                                                                                                  100%  297KB  78.8MB/s   00:00    
connect-transforms-0.10.2.1.jar                                                                                                                                                                                               100%   54KB  41.6MB/s   00:00    
jackson-jaxrs-json-provider-2.8.5.jar                                                                                                                                                                                         100%   15KB  15.7MB/s   00:00    
jersey-container-servlet-2.24.jar                                                                                                                                                                                             100%   18KB  21.0MB/s   00:00    
jetty-server-9.2.15.v20160210.jar                                                                                                                                                                                             100%  409KB  76.7MB/s   00:00    
jetty-servlet-9.2.15.v20160210.jar                                                                                                                                                                                            100%  113KB  61.3MB/s   00:00    
jetty-servlets-9.2.15.v20160210.jar                                                                                                                                                                                           100%  122KB  58.9MB/s   00:00    
reflections-0.9.10.jar                                                                                                                                                                                                        100%  127KB  60.0MB/s   00:00    
jackson-jaxrs-base-2.8.5.jar                                                                                                                                                                                                  100%   32KB  34.6MB/s   00:00    
jackson-module-jaxb-annotations-2.8.5.jar                                                                                                                                                                                     100%   34KB  28.9MB/s   00:00    
jersey-container-servlet-core-2.24.jar                                                                                                                                                                                        100%   65KB  50.0MB/s   00:00    
jersey-common-2.24.jar                                                                                                                                                                                                        100%  698KB  83.4MB/s   00:00    
jersey-server-2.24.jar                                                                                                                                                                                                        100%  919KB  59.9MB/s   00:00    
javax.ws.rs-api-2.0.1.jar                                                                                                                                                                                                     100%  113KB  55.9MB/s   00:00    
javax.servlet-api-3.1.0.jar                                                                                                                                                                                                   100%   94KB  47.1MB/s   00:00    
jetty-http-9.2.15.v20160210.jar                                                                                                                                                                                               100%  124KB  59.9MB/s   00:00    
jetty-io-9.2.15.v20160210.jar                                                                                                                                                                                                 100%  106KB  58.5MB/s   00:00    
jetty-security-9.2.15.v20160210.jar                                                                                                                                                                                           100%   94KB  44.0MB/s   00:00    
jetty-continuation-9.2.15.v20160210.jar                                                                                                                                                                                       100%   16KB  22.8MB/s   00:00    
jetty-util-9.2.15.v20160210.jar                                                                                                                                                                                               100%  360KB  78.7MB/s   00:00    
guava-18.0.jar                                                                                                                                                                                                                100% 2203KB  69.1MB/s   00:00    
javax.inject-2.5.0-b05.jar                                                                                                                                                                                                    100% 5952     7.6MB/s   00:00    
javax.annotation-api-1.2.jar                                                                                                                                                                                                  100%   26KB  23.6MB/s   00:00    
jersey-guava-2.24.jar                                                                                                                                                                                                         100%  949KB  61.9MB/s   00:00    
hk2-api-2.5.0-b05.jar                                                                                                                                                                                                         100%  174KB  71.1MB/s   00:00    
hk2-locator-2.5.0-b05.jar                                                                                                                                                                                                     100%  180KB  66.8MB/s   00:00    
osgi-resource-locator-1.0.1.jar                                                                                                                                                                                               100%   20KB  24.4MB/s   00:00    
jersey-client-2.24.jar                                                                                                                                                                                                        100%  165KB  22.4MB/s   00:00    
jersey-media-jaxb-2.24.jar                                                                                                                                                                                                    100%   71KB  47.4MB/s   00:00    
validation-api-1.1.0.Final.jar                                                                                                                                                                                                100%   62KB  49.6MB/s   00:00    
hk2-utils-2.5.0-b05.jar                                                                                                                                                                                                       100%  116KB  58.6MB/s   00:00    
aopalliance-repackaged-2.5.0-b05.jar                                                                                                                                                                                          100%   14KB  16.7MB/s   00:00    
javax.inject-1.jar                                                                                                                                                                                                            100% 2497     5.0MB/s   00:00    
jackson-annotations-2.8.5.jar                                                                                                                                                                                                 100%   54KB  43.8MB/s   00:00    
javassist-3.20.0-GA.jar                                                                                                                                                                                                       100%  733KB  77.1MB/s   00:00    
connect-json-0.10.2.1.jar                                                                                                                                                                                                     100%   42KB  36.3MB/s   00:00    
connect-file-0.10.2.1.jar                                                                                                                                                                                                     100%   18KB  20.5MB/s   00:00    
kafka-streams-0.10.2.1.jar                                                                                                                                                                                                    100%  593KB  79.4MB/s   00:00    
rocksdbjni-5.0.1.jar                                                                                                                                                                                                          100% 7232KB  74.7MB/s   00:00    
kafka-streams-examples-0.10.2.1.jar                                                                                                                                                                                           100%   36KB  33.5MB/s   00:00    
kafka_2.11-0.10.2.1-site-docs.tgz                                                                                                                                                                                             100% 1966KB  94.0MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp -r /home/softwares/kafka\_2.11-0.10.2.1/ node107.yinzhengjie.org.cn:/home/softwares/

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# scp -r /home/softwares/kafka_2.11-0.10.2.1/ node108.yinzhengjie.org.cn:/home/softwares/
......
jackson-databind-2.8.5.jar                                                                                                                                                                                                    100% 1207KB  90.6MB/s   00:00    
jackson-annotations-2.8.0.jar                                                                                                                                                                                                 100%   54KB  45.4MB/s   00:00    
jackson-core-2.8.5.jar                                                                                                                                                                                                        100%  274KB  73.3MB/s   00:00    
connect-api-0.10.2.1.jar                                                                                                                                                                                                      100%   55KB  36.5MB/s   00:00    
connect-runtime-0.10.2.1.jar                                                                                                                                                                                                  100%  297KB  34.4MB/s   00:00    
connect-transforms-0.10.2.1.jar                                                                                                                                                                                               100%   54KB  39.5MB/s   00:00    
jackson-jaxrs-json-provider-2.8.5.jar                                                                                                                                                                                         100%   15KB  18.1MB/s   00:00    
jersey-container-servlet-2.24.jar                                                                                                                                                                                             100%   18KB  18.5MB/s   00:00    
jetty-server-9.2.15.v20160210.jar                                                                                                                                                                                             100%  409KB  64.7MB/s   00:00    
jetty-servlet-9.2.15.v20160210.jar                                                                                                                                                                                            100%  113KB  64.9MB/s   00:00    
jetty-servlets-9.2.15.v20160210.jar                                                                                                                                                                                           100%  122KB  62.9MB/s   00:00    
reflections-0.9.10.jar                                                                                                                                                                                                        100%  127KB  61.6MB/s   00:00    
jackson-jaxrs-base-2.8.5.jar                                                                                                                                                                                                  100%   32KB  32.4MB/s   00:00    
jackson-module-jaxb-annotations-2.8.5.jar                                                                                                                                                                                     100%   34KB  35.3MB/s   00:00    
jersey-container-servlet-core-2.24.jar                                                                                                                                                                                        100%   65KB  47.4MB/s   00:00    
jersey-common-2.24.jar                                                                                                                                                                                                        100%  698KB  83.5MB/s   00:00    
jersey-server-2.24.jar                                                                                                                                                                                                        100%  919KB  59.3MB/s   00:00    
javax.ws.rs-api-2.0.1.jar                                                                                                                                                                                                     100%  113KB  55.1MB/s   00:00    
javax.servlet-api-3.1.0.jar                                                                                                                                                                                                   100%   94KB  52.5MB/s   00:00    
jetty-http-9.2.15.v20160210.jar                                                                                                                                                                                               100%  124KB  60.0MB/s   00:00    
jetty-io-9.2.15.v20160210.jar                                                                                                                                                                                                 100%  106KB  61.5MB/s   00:00    
jetty-security-9.2.15.v20160210.jar                                                                                                                                                                                           100%   94KB  51.1MB/s   00:00    
jetty-continuation-9.2.15.v20160210.jar                                                                                                                                                                                       100%   16KB  20.2MB/s   00:00    
jetty-util-9.2.15.v20160210.jar                                                                                                                                                                                               100%  360KB  79.1MB/s   00:00    
guava-18.0.jar                                                                                                                                                                                                                100% 2203KB  78.6MB/s   00:00    
javax.inject-2.5.0-b05.jar                                                                                                                                                                                                    100% 5952     6.2MB/s   00:00    
javax.annotation-api-1.2.jar                                                                                                                                                                                                  100%   26KB  25.8MB/s   00:00    
jersey-guava-2.24.jar                                                                                                                                                                                                         100%  949KB  61.6MB/s   00:00    
hk2-api-2.5.0-b05.jar                                                                                                                                                                                                         100%  174KB  65.6MB/s   00:00    
hk2-locator-2.5.0-b05.jar                                                                                                                                                                                                     100%  180KB  67.9MB/s   00:00    
osgi-resource-locator-1.0.1.jar                                                                                                                                                                                               100%   20KB  24.9MB/s   00:00    
jersey-client-2.24.jar                                                                                                                                                                                                        100%  165KB  58.3MB/s   00:00    
jersey-media-jaxb-2.24.jar                                                                                                                                                                                                    100%   71KB  48.4MB/s   00:00    
validation-api-1.1.0.Final.jar                                                                                                                                                                                                100%   62KB  50.6MB/s   00:00    
hk2-utils-2.5.0-b05.jar                                                                                                                                                                                                       100%  116KB  60.5MB/s   00:00    
aopalliance-repackaged-2.5.0-b05.jar                                                                                                                                                                                          100%   14KB  17.3MB/s   00:00    
javax.inject-1.jar                                                                                                                                                                                                            100% 2497     3.9MB/s   00:00    
jackson-annotations-2.8.5.jar                                                                                                                                                                                                 100%   54KB  45.2MB/s   00:00    
javassist-3.20.0-GA.jar                                                                                                                                                                                                       100%  733KB  86.3MB/s   00:00    
connect-json-0.10.2.1.jar                                                                                                                                                                                                     100%   42KB  32.7MB/s   00:00    
connect-file-0.10.2.1.jar                                                                                                                                                                                                     100%   18KB  21.9MB/s   00:00    
kafka-streams-0.10.2.1.jar                                                                                                                                                                                                    100%  593KB  48.3MB/s   00:00    
rocksdbjni-5.0.1.jar                                                                                                                                                                                                          100% 7232KB  71.9MB/s   00:00    
kafka-streams-examples-0.10.2.1.jar                                                                                                                                                                                           100%   36KB  35.2MB/s   00:00    
kafka_2.11-0.10.2.1-site-docs.tgz                                                                                                                                                                                             100% 1966KB  73.9MB/s   00:00    
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# scp -r /home/softwares/kafka\_2.11-0.10.2.1/ node108.yinzhengjie.org.cn:/home/softwares/

**5>.修改kafka的配置文件（server.properties）**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# cat /home/softwares/kafka_2.11-0.10.2.1/config/server.properties 
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

#每一个broker在集群中的唯一表示，要求是正数。当该服务器的IP地址发生改变时，broker.id没有变化，则不会影响consumers的消息情况
broker.id=106

#这就是说，这条命令其实并不执行删除动作，仅仅是在zookeeper上标记该topic要被删除而已，同时也提醒用户一定要提前打开delete.topic.enable开关，否则删除动作是不会执行的。
delete.topic.enable=true

#是否允许自动创建topic，若是false，就需要通过命令创建topic
auto.create.topics.enable=false

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092

#Socket服务器侦听的地址。如果没有配置，它将获得从Java.NET.InAddio.GETCANONICALITHAMEMENE（）返回的值
#listeners=PLAINTEXT://172.30.1.106:9092

#broker server服务端口
port=9092

#broker的主机地址，若是设置了，那么会绑定到这个地址上，若是没有，会绑定到所有的接口上，并将其中之一发送到ZK，一般不设置
host.name=node106.yinzhengjie.org.cn
# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().

#kafka 0.9.x以后的版本新增了advertised.listeners配置,kafka 0.9.x以后的版本不要使用 advertised.host.name 和 advertised.host.port 已经deprecated.如果配置的话，它使用 "listeners" 的值。否则，它将使用从java.net.InetAddress.getCanonicalHostName()返回的值。
#advertised.listeners=PLAINTEXT://your.host.name:9092


#将侦听器(listener)名称映射到安全协议，默认情况下它们是相同的。有关详细信息，请参阅配置文档。
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL


#处理网络请求的最大线程数
num.network.threads=30

#处理磁盘I/O的线程数
num.io.threads=30


#套接字服务器使用的发送缓冲区（SOYSNDBUF)
socket.send.buffer.bytes=5242880

#套接字服务器使用的接收缓冲区（SOYRCVBUF）
socket.receive.buffer.bytes=5242880

#套接字服务器将接受的请求的最大大小（对OOM的保护）
socket.request.max.bytes=104857600

#I/O线程等待队列中的最大的请求数，超过这个数量，network线程就不会再接收一个新的请求。应该是一种自我保护机制。
queued.max.requests=1000

############################# Log Basics #############################

#日志存放目录，多个目录使用逗号分割,如果你有多块磁盘，建议配置成多个目录，从而达到I/O的效率的提升。
log.dirs=/home/data/kafka/logs,/home/data/kafka/logs2,/home/data/kafka/logs3

#每个topic的分区个数，若是在topic创建时候没有指定的话会被topic创建时的指定参数覆盖
num.partitions=20

#在启动时恢复日志和关闭时刷盘日志时每个数据目录的线程的数量，默认1
num.recovery.threads.per.data.dir=30


# 默认副本数
default.replication.factor=2

#服务器接受单个消息的最大大小，即消息体的最大大小，单位是字节
message.max.bytes=104857600

# 自动负载均衡,如果设为true，复制控制器会周期性的自动尝试，为所有的broker的每个partition平衡leadership，为更优先(preferred)的replica分配leadership。
# auto.leader.rebalance.enable=false


############################# Log Flush Policy #############################

#在强制fsync一个partition的log文件之前暂存的消息数量。调低这个值会更频繁的sync数据到磁盘，影响性能。通常建议人家使用replication来确保持久性，而不是依靠单机上的fsync，但是这可以带来更多的可靠性，默认10000。
#log.flush.interval.messages=10000

#2次fsync调用之间最大的时间间隔，单位为ms。即使log.flush.interval.messages没有达到，只要这个时间到了也需要调用fsync。默认3000ms.
#log.flush.interval.ms=10000

############################# Log Retention Policy #############################


# 日志保存时间 (hours|minutes)，默认为7天（168小时）。超过这个时间会根据policy处理数据。bytes和minutes无论哪个先达到都会触发。
log.retention.hours=168

#日志数据存储的最大字节数。超过这个时间会根据policy处理数据。
#log.retention.bytes=1073741824

#控制日志segment文件的大小，超出该大小则追加到一个新的日志segment文件中（-1表示没有限制）
log.segment.bytes=536870912

# 当达到下面时间，会强制新建一个segment
#log.roll.hours = 24*7

# 日志片段文件的检查周期，查看它们是否达到了删除策略的设置（log.retention.hours或log.retention.bytes）
log.retention.check.interval.ms=600000

#是否开启压缩
#log.cleaner.enable=false

#日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被 topic创建时的指定参数覆盖
#log.cleanup.policy=delete

# 日志压缩运行的线程数
#log.cleaner.threads=2


# 压缩的日志保留的最长时间
#log.cleaner.delete.retention.ms=3600000


############################# Zookeeper #############################

#zookeeper集群的地址，可以是多个，多个之间用逗号分割.
zookeeper.connect=node106.yinzhengjie.org.cn:2181,node107.yinzhengjie.org.cn:2181,node108.yinzhengjie.org.cn:2181

#ZooKeeper的最大超时时间，就是心跳的间隔，若是没有反映，那么认为已经死了，不易过大
zookeeper.session.timeout.ms=180000

#指定多久消费者更新offset到zookeeper中。注意offset更新时基于time而不是每次获得的消息。一旦在更新zookeeper发生异常并重启，将可能拿到已拿到过的消息,连接zk的超时时间
zookeeper.connection.timeout.ms=6000

#请求的最大大小为字节,请求的最大字节数。这也是对最大记录尺寸的有效覆盖。注意：server具有自己对消息记录尺寸的覆盖，这些尺寸和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。
max.request.size=104857600

#每次fetch请求中，针对每次fetch消息的最大字节数。这些字节将会督导用于每个partition的内存中，因此，此设置将会控制consumer所使用的memory大小。这个fetch请求尺寸必须至少和server允许的最大消息尺寸相等，否则，producer可能发送的消息尺寸大于consumer所能消耗的尺寸
。fetch.message.max.bytes=104857600

#ZooKeeper集群中leader和follower之间的同步时间,换句话说：一个ZK follower能落后leader多久。
#zookeeper.sync.time.ms=2000


############################# Replica Basics #############################

# leader接收follower的"fetch请求"的超时时间,默认是10秒。
# replica.lag.time.max.ms=30000

# 如果relicas落后太多,将会认为此partition relicas已经失效。而一般情况下,因为网络延迟等原因,总会导致replicas中消息同步滞后。如果消息严重滞后,leader将认为此relicas网络延迟较大或者消息吞吐能力有限。在broker数量较少,或者网络不足的环境中,建议提高此值.follower落
后于leader的最大message数,这个参数是broker全局的。设置太大 了，影响真正“落后”follower的移除;设置的太小了，导致follower的频繁进出。无法给定一个合适的replica.lag.max.messages的值,因此不推荐使用，据说新版本的Kafka移除了这个参数。#replica.lag.max.messages=4000

# follower与leader之间的socket超时时间
#replica.socket.timeout.ms=30000

# follower每次fetch数据的最大尺寸
replica.fetch.max.bytes=104857600

# follower的fetch请求超时重发时间
replica.fetch.wait.max.ms=2000

# fetch的最小数据尺寸
#replica.fetch.min.bytes=1

#0.11.0.0版本开始unclean.leader.election.enable参数的默认值由原来的true改为false，可以关闭unclean leader election，也就是不在ISR(IN-Sync Replica)列表中的replica，不会被提升为新的leader partition。kafka集群的持久化力大于可用性，如果ISR中没有其它的replica，
会导致这个partition不能读写。unclean.leader.election.enable=false

# follower中开启的fetcher线程数, 同步速度与系统负载均衡
num.replica.fetchers=5

# partition leader与replicas之间通讯时,socket的超时时间
#controller.socket.timeout.ms=30000

# partition leader与replicas数据同步时,消息的队列尺寸.
#controller.message.queue.size=10

#指定将使用哪个版本的 inter-broker 协议。 在所有经纪人升级到新版本之后，这通常会受到冲击。升级时要设置
#inter.broker.protocol.version=0.10.1

#指定broker将用于将消息添加到日志文件的消息格式版本。 该值应该是有效的ApiVersion。 一些例子是：0.8.2，0.9.0.0，0.10.0。 通过设置特定的消息格式版本，用户保证磁盘上的所有现有消息都小于或等于指定的版本。 不正确地设置这个值将导致使用旧版本的用户出错，因为他们
将接收到他们不理解的格式的消息。#log.message.format.version=0.10.1
[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# cat /home/softwares/kafka\_2.11-0.10.2.1/config/server.properties

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node107.yizhengjie.org.cn ~]# cat /home/softwares/kafka_2.11-0.10.2.1/config/server.properties 
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

#每一个broker在集群中的唯一表示，要求是正数。当该服务器的IP地址发生改变时，broker.id没有变化，则不会影响consumers的消息情况
broker.id=107

#这就是说，这条命令其实并不执行删除动作，仅仅是在zookeeper上标记该topic要被删除而已，同时也提醒用户一定要提前打开delete.topic.enable开关，否则删除动作是不会执行的。
delete.topic.enable=true

#是否允许自动创建topic，若是false，就需要通过命令创建topic
auto.create.topics.enable=false

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092

#Socket服务器侦听的地址。如果没有配置，它将获得从Java.NET.InAddio.GETCANONICALITHAMEMENE（）返回的值
#listeners=PLAINTEXT://172.30.1.106:9092

#broker server服务端口
port=9092

#broker的主机地址，若是设置了，那么会绑定到这个地址上，若是没有，会绑定到所有的接口上，并将其中之一发送到ZK，一般不设置
host.name=node107.yinzhengjie.org.cn
# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().

#kafka 0.9.x以后的版本新增了advertised.listeners配置,kafka 0.9.x以后的版本不要使用 advertised.host.name 和 advertised.host.port 已经deprecated.如果配置的话，它使用 "listeners" 的值。否则，它将使用从java.net.InetAddress.getCanonicalHostName()返回的值。
#advertised.listeners=PLAINTEXT://your.host.name:9092


#将侦听器(listener)名称映射到安全协议，默认情况下它们是相同的。有关详细信息，请参阅配置文档。
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL


#处理网络请求的最大线程数
num.network.threads=30

#处理磁盘I/O的线程数
num.io.threads=30


#套接字服务器使用的发送缓冲区（SOYSNDBUF)
socket.send.buffer.bytes=5242880

#套接字服务器使用的接收缓冲区（SOYRCVBUF）
socket.receive.buffer.bytes=5242880

#套接字服务器将接受的请求的最大大小（对OOM的保护）
socket.request.max.bytes=104857600

#I/O线程等待队列中的最大的请求数，超过这个数量，network线程就不会再接收一个新的请求。应该是一种自我保护机制。
queued.max.requests=1000

############################# Log Basics #############################

#日志存放目录，多个目录使用逗号分割,如果你有多块磁盘，建议配置成多个目录，从而达到I/O的效率的提升。
log.dirs=/home/data/kafka/logs,/home/data/kafka/logs2,/home/data/kafka/logs3

#每个topic的分区个数，若是在topic创建时候没有指定的话会被topic创建时的指定参数覆盖
num.partitions=20

#在启动时恢复日志和关闭时刷盘日志时每个数据目录的线程的数量，默认1
num.recovery.threads.per.data.dir=30


# 默认副本数
default.replication.factor=2

#服务器接受单个消息的最大大小，即消息体的最大大小，单位是字节
message.max.bytes=104857600

# 自动负载均衡,如果设为true，复制控制器会周期性的自动尝试，为所有的broker的每个partition平衡leadership，为更优先(preferred)的replica分配leadership。
# auto.leader.rebalance.enable=false


############################# Log Flush Policy #############################

#在强制fsync一个partition的log文件之前暂存的消息数量。调低这个值会更频繁的sync数据到磁盘，影响性能。通常建议人家使用replication来确保持久性，而不是依靠单机上的fsync，但是这可以带来更多的可靠性，默认10000。
#log.flush.interval.messages=10000

#2次fsync调用之间最大的时间间隔，单位为ms。即使log.flush.interval.messages没有达到，只要这个时间到了也需要调用fsync。默认3000ms.
#log.flush.interval.ms=10000

############################# Log Retention Policy #############################


# 日志保存时间 (hours|minutes)，默认为7天（168小时）。超过这个时间会根据policy处理数据。bytes和minutes无论哪个先达到都会触发。
log.retention.hours=168

#日志数据存储的最大字节数。超过这个时间会根据policy处理数据。
#log.retention.bytes=1073741824

#控制日志segment文件的大小，超出该大小则追加到一个新的日志segment文件中（-1表示没有限制）
log.segment.bytes=536870912

# 当达到下面时间，会强制新建一个segment
#log.roll.hours = 24*7

# 日志片段文件的检查周期，查看它们是否达到了删除策略的设置（log.retention.hours或log.retention.bytes）
log.retention.check.interval.ms=600000

#是否开启压缩
#log.cleaner.enable=false

#日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被 topic创建时的指定参数覆盖
#log.cleanup.policy=delete

# 日志压缩运行的线程数
#log.cleaner.threads=2


# 压缩的日志保留的最长时间
#log.cleaner.delete.retention.ms=3600000


############################# Zookeeper #############################

#zookeeper集群的地址，可以是多个，多个之间用逗号分割.
zookeeper.connect=node106.yinzhengjie.org.cn:2181,node107.yinzhengjie.org.cn:2181,node108.yinzhengjie.org.cn:2181

#ZooKeeper的最大超时时间，就是心跳的间隔，若是没有反映，那么认为已经死了，不易过大
zookeeper.session.timeout.ms=180000

#指定多久消费者更新offset到zookeeper中。注意offset更新时基于time而不是每次获得的消息。一旦在更新zookeeper发生异常并重启，将可能拿到已拿到过的消息,连接zk的超时时间
zookeeper.connection.timeout.ms=6000

#请求的最大大小为字节,请求的最大字节数。这也是对最大记录尺寸的有效覆盖。注意：server具有自己对消息记录尺寸的覆盖，这些尺寸和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。
max.request.size=104857600

#每次fetch请求中，针对每次fetch消息的最大字节数。这些字节将会督导用于每个partition的内存中，因此，此设置将会控制consumer所使用的memory大小。这个fetch请求尺寸必须至少和server允许的最大消息尺寸相等，否则，producer可能发送的消息尺寸大于consumer所能消耗的尺寸
。fetch.message.max.bytes=104857600

#ZooKeeper集群中leader和follower之间的同步时间,换句话说：一个ZK follower能落后leader多久。
#zookeeper.sync.time.ms=2000


############################# Replica Basics #############################

# leader接收follower的"fetch请求"的超时时间,默认是10秒。
# replica.lag.time.max.ms=30000

# 如果relicas落后太多,将会认为此partition relicas已经失效。而一般情况下,因为网络延迟等原因,总会导致replicas中消息同步滞后。如果消息严重滞后,leader将认为此relicas网络延迟较大或者消息吞吐能力有限。在broker数量较少,或者网络不足的环境中,建议提高此值.follower落
后于leader的最大message数,这个参数是broker全局的。设置太大 了，影响真正“落后”follower的移除;设置的太小了，导致follower的频繁进出。无法给定一个合适的replica.lag.max.messages的值,因此不推荐使用，据说新版本的Kafka移除了这个参数。#replica.lag.max.messages=4000

# follower与leader之间的socket超时时间
#replica.socket.timeout.ms=30000

# follower每次fetch数据的最大尺寸
replica.fetch.max.bytes=104857600

# follower的fetch请求超时重发时间
replica.fetch.wait.max.ms=2000

# fetch的最小数据尺寸
#replica.fetch.min.bytes=1

#0.11.0.0版本开始unclean.leader.election.enable参数的默认值由原来的true改为false，可以关闭unclean leader election，也就是不在ISR(IN-Sync Replica)列表中的replica，不会被提升为新的leader partition。kafka集群的持久化力大于可用性，如果ISR中没有其它的replica，
会导致这个partition不能读写。unclean.leader.election.enable=false

# follower中开启的fetcher线程数, 同步速度与系统负载均衡
num.replica.fetchers=5

# partition leader与replicas之间通讯时,socket的超时时间
#controller.socket.timeout.ms=30000

# partition leader与replicas数据同步时,消息的队列尺寸.
#controller.message.queue.size=10

#指定将使用哪个版本的 inter-broker 协议。 在所有经纪人升级到新版本之后，这通常会受到冲击。升级时要设置
#inter.broker.protocol.version=0.10.1

#指定broker将用于将消息添加到日志文件的消息格式版本。 该值应该是有效的ApiVersion。 一些例子是：0.8.2，0.9.0.0，0.10.0。 通过设置特定的消息格式版本，用户保证磁盘上的所有现有消息都小于或等于指定的版本。 不正确地设置这个值将导致使用旧版本的用户出错，因为他们
将接收到他们不理解的格式的消息。#log.message.format.version=0.10.1
[root@node107.yizhengjie.org.cn ~]# 
```

\[root@node107.yizhengjie.org.cn ~\]# cat /home/softwares/kafka\_2.11-0.10.2.1/config/server.properties

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node108.yinzhengjie.org.cn ~]# cat /home/softwares/kafka_2.11-0.10.2.1/config/server.properties 
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

#每一个broker在集群中的唯一表示，要求是正数。当该服务器的IP地址发生改变时，broker.id没有变化，则不会影响consumers的消息情况
broker.id=108

#这就是说，这条命令其实并不执行删除动作，仅仅是在zookeeper上标记该topic要被删除而已，同时也提醒用户一定要提前打开delete.topic.enable开关，否则删除动作是不会执行的。
delete.topic.enable=true

#是否允许自动创建topic，若是false，就需要通过命令创建topic
auto.create.topics.enable=false

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092

#Socket服务器侦听的地址。如果没有配置，它将获得从Java.NET.InAddio.GETCANONICALITHAMEMENE（）返回的值
#listeners=PLAINTEXT://172.30.1.106:9092

#broker server服务端口
port=9092

#broker的主机地址，若是设置了，那么会绑定到这个地址上，若是没有，会绑定到所有的接口上，并将其中之一发送到ZK，一般不设置
host.name=node108.yinzhengjie.org.cn
# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().

#kafka 0.9.x以后的版本新增了advertised.listeners配置,kafka 0.9.x以后的版本不要使用 advertised.host.name 和 advertised.host.port 已经deprecated.如果配置的话，它使用 "listeners" 的值。否则，它将使用从java.net.InetAddress.getCanonicalHostName()返回的值。
#advertised.listeners=PLAINTEXT://your.host.name:9092


#将侦听器(listener)名称映射到安全协议，默认情况下它们是相同的。有关详细信息，请参阅配置文档。
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL


#处理网络请求的最大线程数
num.network.threads=30

#处理磁盘I/O的线程数
num.io.threads=30


#套接字服务器使用的发送缓冲区（SOYSNDBUF)
socket.send.buffer.bytes=5242880

#套接字服务器使用的接收缓冲区（SOYRCVBUF）
socket.receive.buffer.bytes=5242880

#套接字服务器将接受的请求的最大大小（对OOM的保护）
socket.request.max.bytes=104857600

#I/O线程等待队列中的最大的请求数，超过这个数量，network线程就不会再接收一个新的请求。应该是一种自我保护机制。
queued.max.requests=1000

############################# Log Basics #############################

#日志存放目录，多个目录使用逗号分割,如果你有多块磁盘，建议配置成多个目录，从而达到I/O的效率的提升。
log.dirs=/home/data/kafka/logs,/home/data/kafka/logs2,/home/data/kafka/logs3

#每个topic的分区个数，若是在topic创建时候没有指定的话会被topic创建时的指定参数覆盖
num.partitions=20

#在启动时恢复日志和关闭时刷盘日志时每个数据目录的线程的数量，默认1
num.recovery.threads.per.data.dir=30


# 默认副本数
default.replication.factor=2

#服务器接受单个消息的最大大小，即消息体的最大大小，单位是字节
message.max.bytes=104857600

# 自动负载均衡,如果设为true，复制控制器会周期性的自动尝试，为所有的broker的每个partition平衡leadership，为更优先(preferred)的replica分配leadership。
# auto.leader.rebalance.enable=false


############################# Log Flush Policy #############################

#在强制fsync一个partition的log文件之前暂存的消息数量。调低这个值会更频繁的sync数据到磁盘，影响性能。通常建议人家使用replication来确保持久性，而不是依靠单机上的fsync，但是这可以带来更多的可靠性，默认10000。
#log.flush.interval.messages=10000

#2次fsync调用之间最大的时间间隔，单位为ms。即使log.flush.interval.messages没有达到，只要这个时间到了也需要调用fsync。默认3000ms.
#log.flush.interval.ms=10000

############################# Log Retention Policy #############################


# 日志保存时间 (hours|minutes)，默认为7天（168小时）。超过这个时间会根据policy处理数据。bytes和minutes无论哪个先达到都会触发。
log.retention.hours=168

#日志数据存储的最大字节数。超过这个时间会根据policy处理数据。
#log.retention.bytes=1073741824

#控制日志segment文件的大小，超出该大小则追加到一个新的日志segment文件中（-1表示没有限制）
log.segment.bytes=536870912

# 当达到下面时间，会强制新建一个segment
#log.roll.hours = 24*7

# 日志片段文件的检查周期，查看它们是否达到了删除策略的设置（log.retention.hours或log.retention.bytes）
log.retention.check.interval.ms=600000

#是否开启压缩
#log.cleaner.enable=false

#日志清理策略选择有：delete和compact主要针对过期数据的处理，或是日志文件达到限制的额度，会被 topic创建时的指定参数覆盖
#log.cleanup.policy=delete

# 日志压缩运行的线程数
#log.cleaner.threads=2


# 压缩的日志保留的最长时间
#log.cleaner.delete.retention.ms=3600000


############################# Zookeeper #############################

#zookeeper集群的地址，可以是多个，多个之间用逗号分割.
zookeeper.connect=node106.yinzhengjie.org.cn:2181,node107.yinzhengjie.org.cn:2181,node108.yinzhengjie.org.cn:2181

#ZooKeeper的最大超时时间，就是心跳的间隔，若是没有反映，那么认为已经死了，不易过大
zookeeper.session.timeout.ms=180000

#指定多久消费者更新offset到zookeeper中。注意offset更新时基于time而不是每次获得的消息。一旦在更新zookeeper发生异常并重启，将可能拿到已拿到过的消息,连接zk的超时时间
zookeeper.connection.timeout.ms=6000

#请求的最大大小为字节,请求的最大字节数。这也是对最大记录尺寸的有效覆盖。注意：server具有自己对消息记录尺寸的覆盖，这些尺寸和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。
max.request.size=104857600

#每次fetch请求中，针对每次fetch消息的最大字节数。这些字节将会督导用于每个partition的内存中，因此，此设置将会控制consumer所使用的memory大小。这个fetch请求尺寸必须至少和server允许的最大消息尺寸相等，否则，producer可能发送的消息尺寸大于consumer所能消耗的尺寸
。fetch.message.max.bytes=104857600

#ZooKeeper集群中leader和follower之间的同步时间,换句话说：一个ZK follower能落后leader多久。
#zookeeper.sync.time.ms=2000


############################# Replica Basics #############################

# leader接收follower的"fetch请求"的超时时间,默认是10秒。
# replica.lag.time.max.ms=30000

# 如果relicas落后太多,将会认为此partition relicas已经失效。而一般情况下,因为网络延迟等原因,总会导致replicas中消息同步滞后。如果消息严重滞后,leader将认为此relicas网络延迟较大或者消息吞吐能力有限。在broker数量较少,或者网络不足的环境中,建议提高此值.follower落
后于leader的最大message数,这个参数是broker全局的。设置太大 了，影响真正“落后”follower的移除;设置的太小了，导致follower的频繁进出。无法给定一个合适的replica.lag.max.messages的值,因此不推荐使用，据说新版本的Kafka移除了这个参数。#replica.lag.max.messages=4000

# follower与leader之间的socket超时时间
#replica.socket.timeout.ms=30000

# follower每次fetch数据的最大尺寸
replica.fetch.max.bytes=104857600

# follower的fetch请求超时重发时间
replica.fetch.wait.max.ms=2000

# fetch的最小数据尺寸
#replica.fetch.min.bytes=1

#0.11.0.0版本开始unclean.leader.election.enable参数的默认值由原来的true改为false，可以关闭unclean leader election，也就是不在ISR(IN-Sync Replica)列表中的replica，不会被提升为新的leader partition。kafka集群的持久化力大于可用性，如果ISR中没有其它的replica，
会导致这个partition不能读写。unclean.leader.election.enable=false

# follower中开启的fetcher线程数, 同步速度与系统负载均衡
num.replica.fetchers=5

# partition leader与replicas之间通讯时,socket的超时时间
#controller.socket.timeout.ms=30000

# partition leader与replicas数据同步时,消息的队列尺寸.
#controller.message.queue.size=10

#指定将使用哪个版本的 inter-broker 协议。 在所有经纪人升级到新版本之后，这通常会受到冲击。升级时要设置
#inter.broker.protocol.version=0.10.1

#指定broker将用于将消息添加到日志文件的消息格式版本。 该值应该是有效的ApiVersion。 一些例子是：0.8.2，0.9.0.0，0.10.0。 通过设置特定的消息格式版本，用户保证磁盘上的所有现有消息都小于或等于指定的版本。 不正确地设置这个值将导致使用旧版本的用户出错，因为他们
将接收到他们不理解的格式的消息。#log.message.format.version=0.10.1
[root@node108.yinzhengjie.org.cn ~]# 
```

\[root@node108.yinzhengjie.org.cn ~\]# cat /home/softwares/kafka\_2.11-0.10.2.1/config/server.properties

**6>.启动kafka集群**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node106.yinzhengjie.org.cn ~]# kafka-server-start.sh /home/softwares/kafka_2.11-0.10.2.1/config/server.properties >> /dev/null  &　　　　　　　　#启动Broker
[1] 13760
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# jps
13760 Kafka
3745 QuorumPeerMain
14083 Jps
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# kafka-server-stop.sh 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　#停止Broker
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# jps
3745 QuorumPeerMain
14810 Jps
[root@node106.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ansible kafka -m shell -a 'jps'
node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
3705 QuorumPeerMain
12313 Jps

node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
12308 Jps
3708 QuorumPeerMain

node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
3745 QuorumPeerMain
15750 Jps

[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# kafka_manager.sh start
========== node106.yinzhengjie.org.cn start ================
node106.yinzhengjie.org.cn 服务已启动
========== node107.yinzhengjie.org.cn start ================
node107.yinzhengjie.org.cn 服务已启动
========== node108.yinzhengjie.org.cn start ================
node108.yinzhengjie.org.cn 服务已启动
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ansible kafka -m shell -a 'jps'
node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
3745 QuorumPeerMain
16218 Jps
15788 Kafka

node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
12338 Kafka
12725 Jps
3708 QuorumPeerMain

node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
12343 Kafka
3705 QuorumPeerMain
12730 Jps

[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# kafka\_manager.sh start

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node106.yinzhengjie.org.cn ~]# ansible kafka -m shell -a 'jps'
node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
3745 QuorumPeerMain
16218 Jps
15788 Kafka

node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
12338 Kafka
12725 Jps
3708 QuorumPeerMain

node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
12343 Kafka
3705 QuorumPeerMain
12730 Jps

[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# kafka_manager.sh stop　　　　　　　　#执行该命令，在生产集群数据量较大时，可能需要等待一段时间才能停止集群，此时不要使用kill命令去直接干掉broker进程，否则可鞥会造成数据丢失。
========== node106.yinzhengjie.org.cn stop ================
node106.yinzhengjie.org.cn 服务已停止
========== node107.yinzhengjie.org.cn stop ================
node107.yinzhengjie.org.cn 服务已停止
========== node108.yinzhengjie.org.cn stop ================
node108.yinzhengjie.org.cn 服务已停止
[root@node106.yinzhengjie.org.cn ~]# 
[root@node106.yinzhengjie.org.cn ~]# ansible kafka -m shell -a 'jps'
node107.yinzhengjie.org.cn | SUCCESS | rc=0 >>
3705 QuorumPeerMain
12846 Jps

node108.yinzhengjie.org.cn | SUCCESS | rc=0 >>
12841 Jps
3708 QuorumPeerMain

node106.yinzhengjie.org.cn | SUCCESS | rc=0 >>
16384 Jps
3745 QuorumPeerMain

[root@node106.yinzhengjie.org.cn ~]# 
```

\[root@node106.yinzhengjie.org.cn ~\]# kafka\_manager.sh stop　　　　　　　　#执行该命令，在生产集群数据量较大时，可能需要等待一段时间才能停止集群，此时不要使用kill命令去直接干掉broker进程，否则可鞥会造成数据丢失。

**四.Apache Kafka运维常用命令**

```
　　详情请参考:https://www.cnblogs.com/yinzhengjie/p/9210029.html
```

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186