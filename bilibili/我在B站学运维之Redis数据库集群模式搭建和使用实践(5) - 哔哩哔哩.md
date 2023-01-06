**本章目录:**

-   Redis Cluster 模式
    

-   1.基础介绍
    
-   2.集群架构
    
-   3.原理分析
    
-   4.集群搭建
    
-   5.集群命令
    

-   5.1 命令行式
    
-   5.2 交互式命令
    

-   6.集群补充
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 在大型项目中Redis Cluster 集群模式被广泛应用，哨兵解决和主从不能自动故障恢复的问题，但是同时也存在难以扩容以及单机存储、读写能力受限的问题，并且集群之前都是一台redis的全量的数据，导致所有的redis都冗余一份，就会大大消耗内存空间,所以此时我们需要引入Redis 集群模式。

描述: **什么是Redis集群？**

> Redis集群是一个多Redis实例的集合，用于通过对数据库分区来扩展数据库，使其更具有弹性。集群中的每个成员，无论是主副本还是次级副本，都管理哈希槽的一个子集。如果一个主服务器出现不能访问的故障，那么它的从属服务器会提升为主服务器。在由三个主节点组成的最小的Redis集群中，每个主节点都有一个从属节点（为了至少能保证最低程度的故障转移），每个主节点分配一个范围在0至16383之间的哈希槽。节点A包含哈希槽范围为从0到5000，节点B为5001到10000，节点C从10001到18383。集群内部的通信则通过内部总线进行，使用gossip协议来传播关于集群的信息或者发现新节点。

Redis 集群采用了P2P的模式完全去中心化(分布式存储), 实现数据的分片把所有的 Key 分成了 16384 个 slot，每个 Redis 实例负责其中一部分slot, 并根据算法每个redis节点存储不同的内容; 集群中的所有信息（节点、端口、slot等），都通过节点之间定期的数据交换而更新，同时解决了在线的节点收缩（下线）和扩容（上线）问题。

例如：Redis 客户端可以在任意一个 Redis 实例发出请求（读、写），如果所需数据不在该实例中，通过重定向命令引导客户端访问所需的实例。

![WeiyiGeek.RedisCluster](https://i0.hdslb.com/bfs/article/c22f77fad12b6853a3275f9620b5d3105dbe8e25.png@942w_672h_progressive.webp)

**cluster 特性(已测试):**

-   1): 节点自动发现
    
-   2): slave->master 选举,集群容错
    
-   3): Hot resharding:在线分片
    
-   4): 集群管理:cluster xxx
    
-   5): 基于配置(nodes-port.conf)的集群管理
    
-   6): ASK 转向/MOVED 转向机制.
    

**优点**

1.  集群模式是一个无中心的架构模式，将数据进行分片，分布到对应的槽中，每个节点存储不同的数据内容，通过路由能够找到对应的节点负责存储的槽，能够实现高效率的查询。
    
2.  并且集群模式增加了横向和纵向的扩展能力，实现节点加入和收缩，集群模式是哨兵的升级版，哨兵的优点集群都有。
    

**缺点**

1.  cluster环境下slave默认不接受任何读写操作，在slave执行readonly命令后可只能执行读操作
    
2.  client端不支持多key操作(mget,mset等)，但当keys集合对应的slot相同时支持mget操作见:hash\_tag
    
3.  不支持多数据库，只有一个`db select 0`(无多个db)
    
4.  JedisCluster 没有针对byte\[\]的API，需要自己扩展(附件是我加的基于byte\[\]的BinaryJedisCluster-api)
    
5.  缓存的最大问题就是带来数据一致性问题，在平衡数据一致性的问题时，兼顾性能与业务要求，大多数都是以最终一致性的方案进行解决，而不是强一致性。
    
6.  并且集群模式带来节点数量的剧增，一个集群模式最少要6台机，因为要满足半数原则的选举方式，所以也带来了架构的复杂性。
    

Tips: 集群模式真正意义上实现了系统的高可用和高性能，但是集群同时进一步使系统变得越来越复杂，接下来我们来详细的了解集群的运作原理。

Q:什么是redis-cluster选举容错?

-   (1)选举过程是集群中所有master参与, 如果半数以上master节点与故障节点通信超过(cluster-node-timeout)时间, 则认为该节点故障，自动触发故障转移操作.
    
-   (2) 什么时候整个集群不可用(cluster\_state:fail)?
    

```
a: 如果集群任意master挂掉,且当前master没有slave.集群进入fail状态,也可以理解成集群的slot映射[0-16383]不完整时进入fail状态.ps : redis-3.0.0.rc1 加入cluster-require-full-coverage 参数,默认关闭,打开集群兼容部分失败.
b: 如果集群超过半数以上master挂掉，无论是否有slave集群进入fail状态.
ps: 当集群不可用时,所有对集群的操作做都不可用，收到((error) CLUSTERDOWN The cluster is down)错误
```

描述: redis-cluter 架构细节浅析

-   所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽.
    
-   节点的fail是通过集群中超过半数的节点检测失效时才生效.
    
-   客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
    
-   redis-cluster把所有的物理节点映射到`\[0-16383]slot`上, cluster 负责维护 `node<->slot<->key`
    
-   Redis集群预分好16384个桶，当需要在 Redis 集群中放置一个key-value 时，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中。
    

![WeiyiGeek.架构图](https://i0.hdslb.com/bfs/article/50ec34a43004a0fc9da82086f66e3f8cf3eb1a18.png@732w_839h_progressive.webp)

**3.1 数据分区原理**  
描述: 集群的原理图还是很好理解的，在Redis集群中采用的是虚拟槽分区算法，会把redis集群分成 16384 个槽（0-16383）。

比如：下图所示三个master会把`0-16383`范围的槽可能分成三部分（0-5000）、（5001-11000）、（11001-16383）分别数据三个缓存节点的槽范围。

当客户端请求过来，会首先通过对key进行CRC16 校验并对 16384 取模（CRC16(key)%16383）计算出key所在的槽，然后再到对应的槽上进行取数据或者存数据，这样就实现了数据的访问更新。

![WeiyiGeek.数据分区原理](https://i0.hdslb.com/bfs/article/5033af670e5e44bd5ca6713f902bd4f4ce9e08ea.png@942w_275h_progressive.webp)

Tips: 进行分槽存储是将一整堆的数据进行分片，防止单台的redis数据量过大，影响性能的问题。

**3.2 节点通信**  
描述: 节点之间实现了将数据进行分片存储，那么节点之间又是怎么通信的呢？

这个和前面哨兵模式讲的命令基本一样, 首先新上线的节点, 会通过 Gossip 协议向老成员发送Meet消息，表示自己是新加入的成员。

老成员收到Meet消息后，在没有故障的情况下会恢复PONG消息，表示欢迎新结点的加入，除了第一次发送Meet消息后，之后都会发送定期PING消息，实现节点之间的通信。

流程示例：

```
缓存节点2  
-> Meet 消息
-< Pong 消息
-> Ping 消息
-< Pong 消息
缓存节点1
```

通信的过程中会为每一个通信的节点开通一条tcp通道，之后就是定时任务，不断的向其它节点发送PING消息，这样做的目的就是为了了解节点之间的元数据存储情况，以及健康状况，以便即使发现问题。

**3.3 数据请求**  
上面说到了槽信息，在Redis的底层维护了`unsigned char myslots[CLUSTER_SLOTS/8]`一个数组存放每个节点的槽信息。

因为他是一个二进制数组，只有存储0和1值，如下所示：

```
数组下标: 0 1 2 3 4 ..... 16382 16383
二进制值: 1 1 1 1 0 ..... 0     0
```

这样数组只表示自己是否存储对应的槽数据，若是1表示存在该数据，0表示不存在该数据，这样查询的效率就会非常的高，类似于布隆过滤器(二进制存储)。

比如：集群节点1负责存储0-5000的槽数据，但是此时只有0、1、2存储有数据，其它的槽还没有存数据，所以0、1、2对应的值为1。

并且，每个redis底层还维护了一个clusterNode数组，大小也是16384，用于储存负责对应槽的节点的ip、端口等信息，这样每一个节点就维护了其它节点的元数据信息，便于及时的找到对应的节点。

当新结点加入或者节点收缩，通过PING命令通信，及时的更新自己clusterNode数组中的元数据信息，这样有请求过来也就能及时的找到对应的节点。

![WeiyiGeek.clusterNode数组中的元数据信息](https://i0.hdslb.com/bfs/article/c62556b756a7d906b78c6bad7f1a8d7fe317a8bd.png@942w_719h_progressive.webp)

有两种其它的情况就是：

1.  若是请求过来发现，数据发生了迁移，比如新节点加入，会使旧的缓存节点数据迁移到新结点。
    
2.  请求过来发现旧节点已经发生了数据迁移并且数据被迁移到新结点，由于每个节点都有clusterNode信息，通过该信息的ip和端口。此时旧节点就会向客户端发一个MOVED 的重定向请求，表示数据已经迁移到新结点上，你要访问这个新结点的ip和端口就能拿到数据，这样就能重新获取到数据。
    

倘若正在发正数据迁移呢？

> 答: 旧节点就会向客户端发送一个ASK 重定向请求，并返回给客户端迁移的目标节点的ip和端口，这样也能获取到数据。

**扩容和收缩**

描述: 扩容和收缩也就是节点的上线和下线，可能节点发生故障了，故障自动回复的过程（节点收缩）。

节点的收缩和扩容时，会重新计算每一个节点负责的槽范围，并发根据虚拟槽算法，将对应的数据更新到对应的节点。

还有前面的讲的新加入的节点会首先发送Meet消息，详细可以查看前面讲的内容，基本一样的模式。

以及发生故障后，哨兵老大节点的选举，master节点的重新选举，slave怎样晋升为master节点，可以查看前面哨兵模式选举过程。

**高可用主从切换原理**  
描述: 集群中当`slave`发现自己的 master(主节点)变为 FAIL 状态时，便尝试进行 Failover 指向其它正常 master 节点。

有时由于挂掉的 master 可能会有多个slave 节点，从而存在多个 slave 节点 竞争成为 master节点 的过程，其过程如下：

1.slave 发现自己的 master 变为FAIL

2.将自己记录的集群 `currentEpoch` 进行 +1，并广播 `FAILOVER_AUTH_REQUEST` 信息.

3.其他节点收到该信息只有 master 响应判断请求者的合法性， 并发送 `FAILOVER_AUTH_ACK`，对每一个 epoch 只发送一次 ack

4.尝试 failover 的 slave 收集 `FAILOVER_AUTH_ACK`.

5.超过半数后变成新Master.

6.广播 Pong 通知其他集群节点。

所以 Redis Cluster 既能够实现主从的角色分配，又能够实现主从切换，相当于`集成了Replication 和Sentinal` 的功能。

描述: Redis集群中要求奇数节点,至少要有三个节点，并且每个节点至少有一备份节点，所以至少需要6个redis服务实例。

**4.1 安装环境**

```
# 系统版本与redis版本
OS: Centos7.10 / Centos6.10
Reids版本：5.0.4

# 3台服务器，每台起3个服务，共9个节点
Centos7：192.168.1.99/24 {7000,7001,7002}
Centos6-1：192.168.1.100/24 {7000,7001,7002}
Centos6-2：192.168.1.101/24 {7000,7001,7002}

# CentOS 依赖库更新，并下载或更新额外需要的依赖
$ yum update -y && yum uograde -y
lib / zlib / ruby / rubugems  #述均安装最新版本，否则会和redis版本不匹配
```

**4.2 安装流程**  
Step 1.分别在三台服务器安装编译配置redis安装与上面一致(注意更新依赖)。

Step 2.准备目录结构和redis配置。

```
# 每台机器上执行如下命令
$ mkdir -p /data/redis/redis-cluster/{7000,7001,7002}
$ touch /data/redis/redis-cluster/7000/redis.conf
$ mkdir -p /etc/redis/cluster/run/ #7建立集群配置文件cluster-config-file路径

tee /data/redis/redis-cluster/7000/redis.conf <<'EOF'
## redis 通用配置 ## (注意注意：注释不能放在配置文件后)
# 指定监听地址与端口
port 7000
bind 0.0.0.0

# 修改安全模式(我们设置requirepass字段，所以保护模式可以关闭了)
protected-mode no

# 后台运行
daemonize yes
masterauth 123456
requirepass 123456

# 指定数据库文件存放路径
dir /data/redis/redis-cluster/7000

# 指定进程pid文件
pidfile /var/run/redis_7000.pid

# 指定日志文件记录文件(notice / verbose)
logfile /var/log/redis_7000.log
loglevel verbose  

# 最大客户端连接数
maxclients 10000

# 客户端连接空闲多久后断开连接，单位秒，0表示禁用
timeout 60

# Redis 数据持久化(rdb/aof)配置
# RDB 文件名
dbfilename "dump.rdb"
# 数据自动保存脚本条件例如300s中有10key发生变化
save 300 10
save 60 10000
# 对RDB文件进行压缩，建议以（磁盘）空间换（CPU）时间。
rdbcompression yes
# 版本5的RDB有一个CRC64算法的校验和放在了文件的最后。这将使文件格式更加可靠。
rdbchecksum yes

# AOF开启
appendonly yes
# AOF文件名
appendfilename "appendonly.aof"
# 可选值 always， everysec，no，建议设置为everysec
appendfsync everysec

# 重命名Redis风险命令
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
rename-command FLUSHDB b840fc02d524045429941cc15f59e41cb7be6c53
rename-command FLUSHALL b840fc02d524045429941cc15f59e41cb7be6c54
rename-command EVAL b840fc02d524045429941cc15f59e41cb7be6c55
rename-command DEBUG b840fc02d524045429941cc15f59e41cb7be6c56
# rename-command SHUTDOWN SHUTDOWN

# 启用集群
cluster-enabled yes
# 指定集群配置文件名称，该文件记录集群运行时信息
# 注意：可以用绝对路径，但必须保证路径已经存在，否则启动失败；如果只有文件名称的话，文件生成在工作路径下
cluster-config-file /etc/redis/cluster/run/nodes-7000.conf
# 集群超时时间
cluster-node-timeout 5000
EOF
```

Step 3.将配置文件分别复制到其他目录并修改端口号。

```
cp /data/redis/redis-cluster/7000/redis.conf /data/redis/redis-cluster/7001/redis.conf
cp /data/redis/redis-cluster/7000/redis.conf /data/redis/redis-cluster/7002/redis.conf 
sed -i 's/7000/7001/g' /data/redis/redis-cluster/7001/redis.conf
sed -i 's/7000/7002/g' /data/redis/redis-cluster/7002/redis.conf
# 注意，上述有些配置项要对应服务和目录，三个目录按照上述配置好后，启动服务
```

Step 4.编写开启与关闭集群脚本 cluster-control.sh，并启动集群

```
tee cluster-control.sh <<'EOF'
#!/bin/bash
function start_cluster()
{
  for i in {0..2}
  do 
    /usr/bin/redis-server /data/redis/redis-cluster/700$i/redis.conf; 
  done
}

function stop_cluster()
{
  # -c 连接到集群
  for i in {0..2}
  do 
    /usr/bin/redis-cli -a 123456 -c -p 700${i} shutdown save; 
  done
}

if [ $# -eq 0 ];then
  echo "Usage: cluster-control.sh [start|sop]"
else
  case $1 in
    start) echo "开启redis集群" 
      start_cluster
    ;;
    stop) echo "停止Redis集群"
      stop_cluster
    ;;
  esac
fi
EOF

# 开启集群
$ chmod +x ./cluster_control.sh && ./cluster_control.sh start

# 集群服务的进程查看
$ ps -ef | grep "redis"
# root      6171     1  0 11:25 ?        00:00:00 /usr/bin/redis-server 0.0.0.0:7000 [cluster]
# root      6178     1  0 11:25 ?        00:00:00 /usr/bin/redis-server 0.0.0.0:7001 [cluster]
# root      6182     1  0 11:25 ?        00:00:00 /usr/bin/redis-server 0.0.0.0:7002 [cluster]
# root      6198  3732  0 11:26 pts/0    00:00:00 grep --color=auto redis
```

Step 5.Redis 官方提供了 redis-trib.rb 工具(`/opt/redis/redis-5.0.4/utils/create-cluster`)，我们可以进行手动进行创建Redis集群。

```
# 注意：在任意一台上运行即可, 不要在每台机器上都运行，一台就够了我们就选用CentOS7作为主。
# redis-4.0.10 版本
./redis-trib.rb create --replicas 1 192.168.1.99:7000 192.168.1.99:7001 192.168.1.99:7002 192.168.1.100:7000 192.168.1.100:7001 192.168.1.100:7002 192.168.1.101:7000 192.168.1.101:7001 192.168.1.101:7002 -a [密码]       
# redis-5.0.4 版本
redis-cli --cluster create 192.168.1.99:7000 192.168.1.99:7001 192.168.1.99:7002 192.168.1.100:7000 192.168.1.100:7001 192.168.1.100:7002 192.168.1.101:7000 192.168.1.101:7001 192.168.1.101:7002 --cluster-replicas 1 -a [密码] 

# 正在9个节点上执行哈希插槽分配
>>> Performing hash slots allocation on 9 nodes... 
Master[0] -> Slots 0 - 4095
Master[1] -> Slots 4096 - 8191
Master[2] -> Slots 8192 - 12287
Master[3] -> Slots 12288 - 16383
# 四台 master 、五台 slave (选择：YES)
Adding replica 192.168.1.101:7001 to 192.168.1.99:7000
Adding replica 192.168.1.99:7002 to 192.168.1.100:7000
Adding replica 192.168.1.100:7002 to 192.168.1.101:7000
Adding replica 192.168.1.101:7002 to 192.168.1.99:7001
Adding replica 192.168.1.100:7001 to 192.168.1.99:7000
M: 8c5a037999975a052e422bc030c3fc72053d059e 192.168.1.99:7000
   slots:[0-4095] (4096 slots) master
M: 4877856114a8efc62325dc0d32adede6256eb9ef 192.168.1.99:7001
   slots:[12288-16383] (4096 slots) master
S: 3bd40f9a2d95a9865f58706b55f4559e6c43dd7e 192.168.1.99:7002
   replicates 6df7e2ad828ba148239dd4465235d17ce0e0af10
M: 6df7e2ad828ba148239dd4465235d17ce0e0af10 192.168.1.100:7000
   slots:[4096-8191] (4096 slots) master
S: 35756a64813b68bf5fea92b6ee2b1a7020b8cc7a 192.168.1.100:7001
   replicates 8c5a037999975a052e422bc030c3fc72053d059e
S: f467098bffa3761868842556eb4cfff7cc8bc363 192.168.1.100:7002
   replicates 660590807a4417e901c6d2277b29a306c1ea07e3
M: 660590807a4417e901c6d2277b29a306c1ea07e3 192.168.1.101:7000
   slots:[8192-12287] (4096 slots) master
S: cd7fb4d0a48a114629c1f34e0f75da5235910995 192.168.1.101:7001
   replicates 8c5a037999975a052e422bc030c3fc72053d059e
S: 8d1cab296fe88e1809e550ecd482f6df67c61e7b 192.168.1.101:7002
   replicates 4877856114a8efc62325dc0d32adede6256eb9ef

# 显示了集群和slot分配结果 4主/5从
>>> Performing Cluster Check (using node 192.168.1.99:7000)
M: 8c5a037999975a052e422bc030c3fc72053d059e 192.168.1.99:7000
   slots:[0-4095] (4096 slots) master
   2 additional replica(s)
S: cd7fb4d0a48a114629c1f34e0f75da5235910995 192.168.1.101:7001
   slots: (0 slots) slave
   replicates 8c5a037999975a052e422bc030c3fc72053d059e
S: 35756a64813b68bf5fea92b6ee2b1a7020b8cc7a 192.168.1.100:7001
   slots: (0 slots) slave
   replicates 8c5a037999975a052e422bc030c3fc72053d059e
M: 4877856114a8efc62325dc0d32adede6256eb9ef 192.168.1.99:7001
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
M: 6df7e2ad828ba148239dd4465235d17ce0e0af10 192.168.1.100:7000
   slots:[4096-8191] (4096 slots) master
   1 additional replica(s)
S: 3bd40f9a2d95a9865f58706b55f4559e6c43dd7e 192.168.1.99:7002
   slots: (0 slots) slave
   replicates 6df7e2ad828ba148239dd4465235d17ce0e0af10
S: f467098bffa3761868842556eb4cfff7cc8bc363 192.168.1.100:7002
   slots: (0 slots) slave
   replicates 660590807a4417e901c6d2277b29a306c1ea07e3
S: 8d1cab296fe88e1809e550ecd482f6df67c61e7b 192.168.1.101:7002
   slots: (0 slots) slave
   replicates 4877856114a8efc62325dc0d32adede6256eb9ef
M: 660590807a4417e901c6d2277b29a306c1ea07e3 192.168.1.101:7000
   slots:[8192-12287] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 如果要重新配置集群先停止集群，然后 将cluster-config-file 配置的所有文件删除，再重新启动集群，就可以重新配置集群。
# 当出现集群无法启动时，删除集群配置文件，再次重新启动每一个redis服务，然后重新构件集群环境。
```

Step 6.验证Redis集群是否正常工作

```
$ redis-cli --cluster check 192.168.1.99:7000 #查看节点7000端口号的集群信息

$ redis-cli -c -p 7000 -h 192.168.1.99  # 连接集群（-c参数得重要特性）
192.168.1.99:7000> auth 123456
OK
# 如果该redis实例中没有key则切换到其他得机器得redis实例中进行获取。
192.168.1.99:7000> get name  
# -> Redirected to slot [5798] located at 192.168.1.100:7000 (error) NOAUTH Authentication required.
192.168.1.100:7000> auth 123456
OK
192.168.1.100:7000> get name
"weiyi"

# 查看集群状态
192.168.1.100:7000> cluster info 
  # cluster_state:ok
  # cluster_slots_assigned:16384
  # cluster_slots_ok:16384
  # cluster_slots_pfail:0
  # cluster_slots_fail:0
  # cluster_known_nodes:9
  # cluster_size:4
  # cluster_current_epoch:9
  # cluster_my_epoch:4
  # cluster_stats_messages_ping_sent:1100
  # cluster_stats_messages_pong_sent:984
  # cluster_stats_messages_sent:2084
  # cluster_stats_messages_ping_received:984
  # cluster_stats_messages_pong_received:1087
  # cluster_stats_messages_received:2071

# 集群节点
192.168.1.100:7000> cluster nodes 
  # 8d1cab296fe88e1809e550ecd482f6df67c61e7b 192.168.1.101:7002@17002 slave 4877856114a8efc62325dc0d32adede6256eb9ef 0 1555559362575 9 connected
  # f467098bffa3761868842556eb4cfff7cc8bc363 192.168.1.100:7002@17002 slave 660590807a4417e901c6d2277b29a306c1ea07e3 0 1555559363077 7 connected
  # 8c5a037999975a052e422bc030c3fc72053d059e 192.168.1.99:7000@17000 master - 0 1555559362172 1 connected 0-4095
  # cd7fb4d0a48a114629c1f34e0f75da5235910995 192.168.1.101:7001@17001 slave 8c5a037999975a052e422bc030c3fc72053d059e 0 1555559363582 8 connected
  # 660590807a4417e901c6d2277b29a306c1ea07e3 192.168.1.101:7000@17000 master - 0 1555559362575 7 connected 8192-12287
  # 6df7e2ad828ba148239dd4465235d17ce0e0af10 192.168.1.100:7000@17000 myself,master - 0 1555559362000 4 connected 4096-8191
  # 35756a64813b68bf5fea92b6ee2b1a7020b8cc7a 192.168.1.100:7001@17001 slave 8c5a037999975a052e422bc030c3fc72053d059e 0 1555559363000 5 connected
  # 4877856114a8efc62325dc0d32adede6256eb9ef 192.168.1.99:7001@17001 master - 0 1555559362070 2 connected 12288-16383
  # 3bd40f9a2d95a9865f58706b55f4559e6c43dd7e 192.168.1.99:7002@17002 slave 6df7e2ad828ba148239dd4465235d17ce0e0af10 0 1555559363078 4 connected

# 查看集群数据槽（划分给每个节点的分片Slots范围。）
192.168.1.100:7000> CLUSTER SLOTS   
1) 1) (integer) 0
   2) (integer) 4095
   3) 1) "192.168.1.99"
      2) (integer) 7000
      3) "8c5a037999975a052e422bc030c3fc72053d059e"
   4) 1) "192.168.1.101"
      2) (integer) 7001
      3) "cd7fb4d0a48a114629c1f34e0f75da5235910995"
   5) 1) "192.168.1.100"
      2) (integer) 7001
      3) "35756a64813b68bf5fea92b6ee2b1a7020b8cc7a"
2) 1) (integer) 8192
   2) (integer) 12287
   3) 1) "192.168.1.101"
      2) (integer) 7000
      3) "660590807a4417e901c6d2277b29a306c1ea07e3"
   4) 1) "192.168.1.100"
      2) (integer) 7002
      3) "f467098bffa3761868842556eb4cfff7cc8bc363"
3) 1) (integer) 4096
   2) (integer) 8191
   3) 1) "192.168.1.100"
      2) (integer) 7000
      3) "6df7e2ad828ba148239dd4465235d17ce0e0af10"
   4) 1) "192.168.1.99"
      2) (integer) 7002
      3) "3bd40f9a2d95a9865f58706b55f4559e6c43dd7e"
4) 1) (integer) 12288
   2) (integer) 16383
   3) 1) "192.168.1.99"
      2) (integer) 7001
      3) "4877856114a8efc62325dc0d32adede6256eb9ef"
   4) 1) "192.168.1.101"
      2) (integer) 7002
      3) "8d1cab296fe88e1809e550ecd482f6df67c61e7b"
```

**语法格式:**

```
# Help | 获取帮助
$ redis-cli --cluster help

# Cluster Manager Commands:
# - 集群创建
  create         host1:port1 ... hostN:portN 
                 --cluster-replicas <arg>    # 指定集群副本主节点数/从节点数的比例，如果为0表示不为集群的master节点创建slave。
# - 集群检查
  check          host:port 
                 --cluster-search-multiple-owners      # 检查是否有槽同时被分配给了多个节点
# - 集群信息
  info           host:port
# - 集群修复
  fix            host:port
                 --cluster-search-multiple-owners       # 说明：修复集群和槽的重复分配问题
                 --cluster-fix-with-unreachable-masters # 不可访问主机的群集修复
# - 添加节点        
  add-node       new_host:new_port existing_host:existing_port # 默认为主节点
                 --cluster-slave           # 按比例分配到master上作为副本Slave
                 --cluster-master-id id    # 指定MasterID上作为其副本Slave
# - 删除节点                 
  del-node       host:port node_id 

# - 分片迁移              
  reshard        host:port
                 --cluster-from <arg>  # 需要从哪些源master节点（id）上迁移slot，可从多个源节点完成迁移，以逗号隔开，传递的是节点的node id，还可以直接传递--from all，这样源节点就是集群的所有节点，不传递该参数的话，则会在迁移过程中提示用户输入
                 --cluster-to <arg>    # slot需要迁移的目的节点的node id
                 --cluster-slots <arg> # 需要迁移的slot数量
                 --cluster-yes         # 指定迁移时的确认输入
                 --cluster-timeout <arg>  # 设置migrate命令的超时时间
                 --cluster-pipeline <arg> # 定义cluster getkeysinslot命令一次取出的key数量，不传的话使用默认值为10
                 --cluster-replace       # 是否直接replace到目标节点
# - 集群重新分片 
  rebalance      host:port
                 --cluster-weight <node1=w1...nodeN=wN>   # 节点权重配置默认1，如为多个节点分配权重则 master1id=5 master2id=2 需要保证前缀数能唯一区别该节点。
                 --cluster-use-empty-masters  # 让没有分配的slots的主机节点参与到rebalance 分片中
                 --cluster-timeout <arg>      # 超时时间设置
                 --cluster-simulate           # 模拟执行
                 --cluster-pipeline <arg>     # 定义cluster getkeysinslot命令一次取出的key数量，不传的话使用默认值为10
                 --cluster-threshold <arg>    # 迁移的slot阈值超过threshold，执行rebalance操作
                 --cluster-replace            # 是否直接replace到目标节点
# 将外部redis数据导入集群
  import         host:port
                 --cluster-from <arg>      # 将指定实例IP
                 --cluster-from-user <arg>
                 --cluster-from-pass <arg>
                 --cluster-from-askpass
                 --cluster-copy        # 复制
                 --cluster-replace     # 替换

#  在集群的所有节点执行相关命令
  call           host:port command arg arg .. arg
                 --cluster-only-masters
                 --cluster-only-replicas
              
# 备份集群数据
  backup         host:port backup_directory

# 设置集群节点间心跳连接的超时时间
  set-timeout    host:port milliseconds

# Cluster Manager Options:
  --cluster-yes  Automatic yes to cluster commands prompts (在脚本实现自动化的时候非常有用)
```

Tips: 对于check、fix、reshard、del node、set timeout，您可以指定群集中任何工作节点的主机和端口。

**实践使用:**

```
# (1) Redis集群创建提供6台配置一样的Redis服务，--cluster-replicas 1 : 表示我们希望为集群中的每个主节点创建一个从节点，此处为三主三从。
/usr/local/bin/redis-cli -h 192.168.1.2 -a weiyigeek.top --cluster create --cluster-replicas 1 [192.168.1.2:6379 ~ 192.168.1.7:6379]
  # [OK] All nodes agree about slots configuration.
  # >>> Check for open slots...
  # >>> Check slots coverage...
  # [OK] All 16384 slots covered.

# (2) 集群信息各工作节点以及卡槽使用检查(2Master/5从节点)
redis-cli -h 172.16.243.97 -a weiyigeek --cluster check 172.16.243.97:6379 --cluster-search-multiple-owners
  # >>> Performing Cluster Check (using node 172.16.243.97:6379)
  # M: d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 172.16.243.97:6379  slots:[0-4723],[8192-16383] (12916 slots) master
  #   3 additional replica(s)
  # S: 436c6a1d7e4c5f782e1e0620b831211ebb0a41a4 172.16.100.116:6379 slots: (0 slots) slave
  #   replicates d97cb5b15b7130ca0bd5322758e0c2dce061fd7b
  # M: 94b8d3748dc47053454e657da8d6bb90e0081f2c 172.16.183.95:6379  slots:[4724-8191] (3468 slots) master
  #   1 additional replica(s)
  # S: 6af3ad6a8691b57393aeb6b4cc5334382aa5b392 172.16.243.98:6379  slots: (0 slots) slave
  #   replicates d97cb5b15b7130ca0bd5322758e0c2dce061fd7b
  # S: d69c99fd6605d747da03874aafaa283a2b5614ea 172.16.24.215:6379  slots: (0 slots) slave
  #   replicates d97cb5b15b7130ca0bd5322758e0c2dce061fd7b
  # S: 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379  slots: (0 slots) slave
  #   replicates 94b8d3748dc47053454e657da8d6bb90e0081f2c


# (3) 集群Master节点信息以及占用的Keys数量和卡槽数量以及有几个从节点
redis-cli -h 172.16.243.97 -a weiyigeek --cluster info 172.16.24.214:6379
  # 172.16.183.95:6379 (94b8d374...) -> 30327 keys | 3468 slots | 1 slaves.
  # 172.16.243.97:6379 (d97cb5b1...) -> 194497 keys | 12916 slots | 3 slaves.
  # [OK] 224824 keys in 2 masters.
  # 13.72 keys per slot on average.


# (4) 集群节点删除实践，例如从redis集群中删除上面的 172.16.100.116:6379 从节点。
redis-cli -h 172.16.243.97 -a weiyigeek --cluster del-node 172.16.100.116:6379 436c6a1d7e4c5f782e1e0620b831211ebb0a41a4
  # >>> Removing node 436c6a1d7e4c5f782e1e0620b831211ebb0a41a4 from cluster 172.16.100.116:6379 
redis-cli -h 172.16.243.97 -a weiyigeek --cluster del-node 172.16.24.214:6379 2674f21a88a9573f51ec46f9dc248ad4a5c5974d
  # >>> Removing node 2674f21a88a9573f51ec46f9dc248ad4a5c5974d from cluster 172.16.24.214:6379
  # >>> Sending CLUSTER FORGET messages to the cluster...
  # >>> Sending CLUSTER RESET SOFT to the deleted node.


# (5) 集群节点添加实践，前者按照比例添加到其master节点下，我们可以使用后者将节点添加为master节点
redis-cli -h 172.16.243.97 -a weiyigeek --cluster add-node 172.16.100.116:6379 172.16.243.97:6379 --cluster-slave  # slave 节点添加
  # [OK] All nodes agree about slots configuration.
  # >>> Check for open slots...
  # >>> Check slots coverage...
  # [OK] All 16384 slots covered.
  # Automatically selected master 172.16.183.95:6379
  # >>> Send CLUSTER MEET to node 172.16.100.116:6379 to make it join the cluster.
  # Waiting for the cluster to join
  # >>> Configure node as replica of 172.16.183.95:6379.
  # [OK] New node added correctly.
redis-cli -h 172.16.243.97 -a weiyigeek --cluster add-node 172.16.24.214:6379 172.16.243.97:6379 --cluster-master-id 94b8d3748dc47053454e657da8d6bb90e0081f2c  # master节点


# (6) 说明：修复集群和槽的重复分配问题（将所有slots迁移到一处）
redis-cli -h 172.16.243.97 -a weiyigeek --cluster fix --cluster-fix-with-unreachable-masters 172.16.24.214:6379 
  # 172.16.24.214:6379 (2674f21a...) -> 0 keys | 0 slots | 0 slaves.
  # [OK] 0 keys in 1 masters.
  # 0.00 keys per slot on average.
  # >>> Performing Cluster Check (using node 172.16.24.214:6379)
  # M: 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379
  #   slots: (0 slots) master
  # [OK] All nodes agree about slots configuration.
  # >>> Check for open slots...
  # >>> Check slots coverage...
  # [ERR] Not all 16384 slots are covered by nodes.
  # >>> Fixing slots coverage...
  # The following uncovered slots have no keys across the cluster:
  # [0-16383]
  # Fix these slots by covering with a random node? (type 'yes' to accept): yes
  # >>> Covering slot 660 with 172.16.24.214:6379
  .....
~$ redis-cli -h 172.16.243.97 -a weiyigeek -c cluster nodes  # 可以看见现在只有一个Master
  # 436c6a1d7e4c5f782e1e0620b831211ebb0a41a4 172.16.100.116:6379@16379 slave 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 0 1631268716040 10 connected
  # 94b8d3748dc47053454e657da8d6bb90e0081f2c 172.16.183.95:6379@16379 slave 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 0 1631268715000 10 connected
  # 6af3ad6a8691b57393aeb6b4cc5334382aa5b392 172.16.243.98:6379@16379 slave 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 0 1631268716541 10 connected
  # d69c99fd6605d747da03874aafaa283a2b5614ea 172.16.24.215:6379@16379 slave 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 0 1631268716000 10 connected
  # d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 172.16.243.97:6379@16379 myself,slave 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 0 1631268715000 10 connected
  # 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379@16379 master - 0 1631268715537 10 connected 0-16383 ... (关键点)


# (6) redis集群重新分配slots给加入的主节点，此时按照如下步骤进行
# - 将两个Slave节点移除
redis-cli -h 172.16.24.214 -a weiyigeek --cluster del-node 172.16.183.95:6379  94b8d3748dc47053454e657da8d6bb90e0081f2c
redis-cli -h 172.16.24.214 -a weiyigeek --cluster del-node 172.16.243.97:6379  d97cb5b15b7130ca0bd5322758e0c2dce061fd7b

# - 将刚才移除的两个slave节点加入到集群并设置为master(此时我们的三主Master也回来了)
redis-cli -h 172.16.24.214 -a weiyigeek --cluster add-node 172.16.183.95:6379 172.16.24.214:6379
redis-cli -h 172.16.24.214 -a weiyigeek --cluster add-node 172.16.243.97:6379 172.16.24.214:6379
  # 94b8d3748dc47053454e657da8d6bb90e0081f2c 172.16.183.95:6379@16379 master - 0 1631276731560 8 connected
  # 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379@16379 myself,master - 0 1631276729000 10 connected 0-16383
  # d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 172.16.243.97:6379@16379 master - 0 1631276730000 9 connected
# - 重新分配卡槽给两个空的master节点
redis-cli -h 172.16.24.214 -a weiyigeek --cluster rebalance --cluster-threshold 1 --cluster-use-empty-masters --cluster-pipeline 10 172.16.243.97:6379
  # >>> Performing Cluster Check (using node 172.16.243.97:6379)
  # [OK] All nodes agree about slots configuration.
  # >>> Check for open slots...
  # >>> Check slots coverage...
  # [OK] All 16384 slots covered.
  # >>> Rebalancing across 3 nodes. Total weight = 3.00
  # Moving 5462 slots from 172.16.24.214:6379 to 172.16.243.97:6379
  # #######
  # Moving 5461 slots from 172.16.24.214:6379 to 172.16.183.95:6379
  # #######
redis-cli -h 172.16.24.214 -a weiyigeek -c cluster nodes  # 验证分配
  # 94b8d3748dc47053454e657da8d6bb90e0081f2c 172.16.183.95:6379@16379 master - 0 1631277376000 12 connected 5462-10922
  # 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379@16379 myself,master - 0 1631277377000 10 connected 10923-16383
  # d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 172.16.243.97:6379@16379 master - 0 1631277377248 11 connected 0-5461


# (7) 迁移 slot 到指定nodeID张，在线把集群的一些slot从集群原来slot节点迁移到新的节点，即可以完成集群的在线横向扩容和缩容。（非常注意卡槽分别分配的数量为 5462 、5461）
# 交互式 - 直接连接到集群的任意一节点，根据选择进行操作
redis-cli -a weiyigeek --cluster reshard 172.16.24.214:6379 # 
# 命令行式 - 连接到集群的任意一节点来对指定节点指定数量的slot进行迁移到指定的节点
redis-cli -a weiyigeek --cluster reshard 172.16.24.214:6379 --cluster-from d97cb5b15b7130ca0bd5322758e0c2dce061fd7b --cluster-to 2674f21a88a9573f51ec46f9dc248ad4a5c5974d --cluster-slots 50 --cluster-yes --cluster-timeout 5000 --cluster-pipeline 50 --cluster-replace
  # Ready to move 50 slots.
  #   Source nodes:
  #     M: d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 172.16.243.97:6379
  #       slots:[0-5461] (5462 slots) master
  #       1 additional replica(s)
  #   Destination node:
  #     M: 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379
  #       slots:[10923-16383] (5461 slots) master
  #       1 additional replica(s)
  #   Resharding plan:
  #     Moving slot 0 from d97cb5b15b7130ca0bd5322758e0c2dce061fd7b
  #     .....
  #     Moving slot 49 from d97cb5b15b7130ca0bd5322758e0c2dce061fd7b


# (8) 备份集群数据到本地目录中。
redis-cli -a weiyigeek --cluster backup 172.16.243.97:6379 .
  # [OK] All nodes agree about slots configuration.
  # >>> Check for open slots...
  # >>> Check slots coverage...
  # [OK] All 16384 slots covered.
  # >>> Node 172.16.243.97:6379 -> Saving RDB...
  # SYNC sent to master, writing 178 bytes to './redis-node-172.16.243.97-6379-d97cb5b15b7130ca0bd5322758e0c2dce061fd7b.rdb'
  # Transfer finished with success.
  # >>> Node 172.16.183.95:6379 -> Saving RDB...
  # SYNC sent to master, writing 178 bytes to './redis-node-172.16.183.95-6379-94b8d3748dc47053454e657da8d6bb90e0081f2c.rdb'
  # Transfer finished with success.
  # >>> Node 172.16.24.214:6379 -> Saving RDB...
  # SYNC sent to master, writing 178 bytes to './redis-node-172.16.24.214-6379-2674f21a88a9573f51ec46f9dc248ad4a5c5974d.rdb'
  # Transfer finished with success.
  # Saving cluster configuration to: ./nodes.json
  # [OK] Backup created into: .


# (9) 将外部redis数据导入集群，例如：把 192.168.75.187:7002上的数据导入到 172.16.24.214:6379 这个节点所在的集群
redis-cli --cluster import 192.168.75.187:7005 --cluster-from 192.168.75.187:7002

# (10) 设置集群的超时时间  
redis-cli -a weiyigeek --cluster set-timeout 172.16.24.214:6379 10000

# (11) 在集群全部节点上执行相同命令
redis-cli -a weiyigeek --cluster call 172.16.243.97:6379 info keyspace
  # >>> Calling info keyspace
  # 172.16.243.97:6379: # Keyspace
  # db0:keys=264260,expires=0,avg_ttl=0
  # 172.16.100.116:6379: # Keyspace
  # db0:keys=264260,expires=0,avg_ttl=0
  # 172.16.183.95:6379: # Keyspace
  # 172.16.243.98:6379: # Keyspace
  # 172.16.24.215:6379: # Keyspace
  # db0:keys=1,expires=0,avg_ttl=0
  # 172.16.24.214:6379: # Keyspace
  # db0:keys=1,expires=0,avg_ttl=0
```

Tips : 通过`--cluster import`从其它集群导入数据到当前集群中的流程说明。

-   1、通过load\_cluster\_info\_from\_node方法加载集群信息，check\_cluster方法检查集群是否健康；
    
-   2、连接外部redis节点，如果外部节点开启了cluster\_enabled，则提示错误；
    
-   3、通过scan命令遍历外部节点，一次获取1000条数据；
    
-   4、遍历这些key，计算出key对应的slot；
    
-   5、执行migrate命令,源节点是外部节点,目的节点是集群slot对应的节点，如果设置了–copy参数，则传递copy参数，如果设置了–replace，则传递replace参数；
    
-   6、不停执行scan命令，直到遍历完全部的key；
    
-   7、至此完成整个迁移流程。
    

描述: 在我们以`-c`集群模式连接到集群中的任意一台主机时，可以通过交互魔术执行如下命令。

**语法格式:**

```
# Help | 获取帮助
redis-cli --cluster help cluster help
# 集群相关操作主命令
 1) CLUSTER <subcommand> [<arg> [value] [opt] ...]. Subcommands are:
  CLUSTER BUMPEPOCH -              # 进行集群配置 epoch
  CLUSTER SET-CONFIG-EPOCH config-epoch # 在新节点中设置配置EPOCH 

  CLUSTER COUNT-FAILURE-REPORTS node-id  # 返回给定节点的活动故障报告数
  CLUSTER GETKEYSINSLOT slot count # 返回指定哈希槽中的本地密钥名称
  CLUSTER COUNTKEYSINSLOT slot           # 返回指定哈希槽中的本地密钥数
  CLUSTER SLOTS - # 获取群集插槽到节点映射的数组
  CLUSTER ADDSLOTS slot [slot ...] # 将新哈希槽分配给接收节点
  CLUSTER SETSLOT slot IMPORTING|MIGRATING|STABLE|NODE [node-id]  # 将哈希槽绑定到特定节点
  CLUSTER DELSLOTS slot [slot ...]       # 在接收节点中将哈希槽设置为未绑定
  CLUSTER FLUSHSLOTS -                   # 删除节点自己的插槽信息

  CLUSTER KEYSLOT key    # 返回指定键的哈希槽

  CLUSTER INFO -         # 提供有关Redis群集节点状态的信息
  CLUSTER MYID -         # 返回节点id
  CLUSTER NODES -        # 获取节点的群集配置

  CLUSTER FAILOVER [FORCE|TAKEOVER]      # 强制复制副本对其主副本执行手动故障切换。
  CLUSTER FORGET node-id # 从节点表中删除节点
  CLUSTER MEET ip port   # 强制节点群集与另一个节点握手
  CLUSTER RESET [HARD|SOFT] # 重置Redis群集节点

  CLUSTER SLAVES node-id  # 列出指定主节点的副本节点
  CLUSTER REPLICAS node-id  # 列出指定主节点的副本节点 (在5.x版本以上使用)
  CLUSTER REPLICATE node-id # 将节点重新配置为指定主节点的副本
  CLUSTER SAVECONFIG -      # 强制节点在磁盘上保存群集状态
```

**使用实践:**

```
$ redis-cli -h 172.16.243.97 -a weiyigeek -c
# //集群(cluster)
CLUSTER INFO  #打印集群的信息
  # cluster_state:ok
  # cluster_slots_assigned:16384
  # cluster_slots_ok:16384
  # cluster_slots_pfail:0
  # cluster_slots_fail:0
  # cluster_known_nodes:6
  # cluster_size:3
  # cluster_current_epoch:13
  # cluster_my_epoch:11
  # cluster_stats_messages_ping_sent:196626
  # cluster_stats_messages_pong_sent:244353
  # cluster_stats_messages_meet_sent:7
  # cluster_stats_messages_update_sent:179
  # cluster_stats_messages_sent:441165
  # cluster_stats_messages_ping_received:193415
  # cluster_stats_messages_pong_received:210335
  # cluster_stats_messages_meet_received:8
  # cluster_stats_messages_update_received:138
  # cluster_stats_messages_received:403896

CLUSTER NODES #列出集群当前已知的所有节点（node），以及这些节点的相关信息。
  # 436c6a1d7e4c5f782e1e0620b831211ebb0a41a4 172.16.100.116:6379@16379 slave d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 0 1631280922704 11 connected
  # 94b8d3748dc47053454e657da8d6bb90e0081f2c 172.16.183.95:6379@16379 master - 0 1631280921700 12 connected 5462-10922
  # 6af3ad6a8691b57393aeb6b4cc5334382aa5b392 172.16.243.98:6379@16379 slave 94b8d3748dc47053454e657da8d6bb90e0081f2c 0 1631280923706 12 connected
  # d69c99fd6605d747da03874aafaa283a2b5614ea 172.16.24.215:6379@16379 slave 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 0 1631280920697 13 connected
  # d97cb5b15b7130ca0bd5322758e0c2dce061fd7b 172.16.243.97:6379@16379 myself,master - 0 1631280922000 11 connected 50-5461
  # 2674f21a88a9573f51ec46f9dc248ad4a5c5974d 172.16.24.214:6379@16379 master - 0 1631280922000 13 connected 0-49 10923-16383


CLUSTER SLOTS #用于当前的集群状态，以数组形式展,返回值 IP/端口嵌套的列表数组
  # 1) 1) (integer) 0
  #   2) (integer) 49
  #   3) 1) "172.16.24.214"
  #       2) (integer) 6379
  #       3) "2674f21a88a9573f51ec46f9dc248ad4a5c5974d"
  #   4) 1) "172.16.24.215"
  #       2) (integer) 6379
  #       3) "d69c99fd6605d747da03874aafaa283a2b5614ea"
  # 2) 1) (integer) 50
  #   2) (integer) 5461
  #   3) 1) "172.16.243.97"
  #       2) (integer) 6379
  #       3) "d97cb5b15b7130ca0bd5322758e0c2dce061fd7b"
  #   4) 1) "172.16.100.116"
  #       2) (integer) 6379
  #       3) "436c6a1d7e4c5f782e1e0620b831211ebb0a41a4"
  # 3) 1) (integer) 5462
  #   2) (integer) 10922
  #   3) 1) "172.16.183.95"
  #       2) (integer) 6379
  #       3) "94b8d3748dc47053454e657da8d6bb90e0081f2c"
  #   4) 1) "172.16.243.98"
  #       2) (integer) 6379
  #       3) "6af3ad6a8691b57393aeb6b4cc5334382aa5b392"
  # 4) 1) (integer) 10923
  #   2) (integer) 16383
  #   3) 1) "172.16.24.214"
  #       2) (integer) 6379
  #       3) "2674f21a88a9573f51ec46f9dc248ad4a5c5974d"
  #   4) 1) "172.16.24.215"
  #       2) (integer) 6379
  #       3) "d69c99fd6605d747da03874aafaa283a2b5614ea"


# //节点(node)
CLUSTER MEET 172.16.24.215 6379    # 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
CLUSTER FORGET <node_id>           # 从 集群中移除指定的 node_id 节点。
CLUSTER REPLICATE <node_id>        # 将当前节点设置为指定的 node_id 节点的从节点。
CLUSTER SAVECONFIG                  # 将节点的配置文件保存到硬盘里面。

# //槽(slot)
CLUSTER ADDSLOTS <slot> [slot ...] # 将一个或多个槽（slot）指派（assign）给当前节点。
CLUSTER DELSLOTS <slot> [slot ...] # 移除一个或多个槽对当前节点的指派。
CLUSTER SETSLOT <slot> NODE <node_id> # 将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
CLUSTER SETSLOT <slot> MIGRATING <node_id> # 将本节点的槽 slot 迁移到 node_id 指定的节点中。
CLUSTER SETSLOT <slot> IMPORTING <node_id>#  从 node_id 指定的节点中导入槽 slot 到本节点。
CLUSTER SETSLOT <slot> STABLE # 取消对槽 slot 的导入（import）或者迁移（migrate）。
CLUSTER FLUSHSLOTS # 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点, 前提是该节点上无任何数据。

# //键 (key)
CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。
  # 172.16.243.97:6379> CLUSTER KEYSLOT weiyigeek66666
  # (integer) 11697

CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。
  # 172.16.243.97:6379> CLUSTER COUNTKEYSINSLOT 2604
  # (integer) 61

CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。
  # 172.16.243.97:6379> CLUSTER GETKEYSINSLOT 2604 8
  # 1) "weiyigeek108466"
  # 2) "weiyigeek119427"
  # 3) "weiyigeek12581"
  # 4) "weiyigeek201670"
  # 5) "weiyigeek204102"
  # 6) "weiyigeek210631"
  # 7) "weiyigeek215143"
  # 8) "weiyigeek21542"
```

**常用命令示例:**

```
# 0.集群创建
redis-cli -h 127.0.0.1 -a weiyigeek --cluster create --cluster-replicas 1 172.16.182.219:6379 172.16.183.72:6379 172.16.24.202:6379 172.16.100.65:6379 172.16.135.193:6379 172.16.243.65:6379

# 1.向集群中添加主节点(127.0.0.1:7000 将成为主节点)
redis-cli -a weiyigeek --cluster add-node 192.168.1.100:7000 192.168.1.99:7000 
  # >>> Adding node 192.168.1.100:7000 to cluster 192.168.1.99:7000
  # >>> Performing Cluster Check (using node 192.168.1.99:7000)

# 3.向集群中添加指定从节点(192.168.1.99:7001 将成为1.99:7000(MYID=a7b511330bffe28357cd21d6ee543e59f0a38dea) 从节点)
redis-cli -a weiyigeek --cluster add-node --cluster-slave --cluster-master-id a7b511330bffe28357cd21d6ee543e59f0a38dea  192.168.1.100:7000 

# 4.集群信息与节点查看
redis-cli -a weiyigeek --cluster check 192.168.1.100:7000 
redis-cli -h 192.168.1.100 -p 7000 -a weiyigeek -c cluster info
redis-cli -h 192.168.1.100 -p 7000 -a weiyigeek -c cluster nodes

# 5.删除节点
redis-cli --cluster del-node 127.0.0.1:7000 f467098bffa3761868842556eb4cfff7cc8bc363 -a 123456   #该节点将会被shutdown

# 6.平衡各节点槽数量
redis-cli --cluster rebalance -a 123456 --cluster-threshold 1 127.0.0.1:8007  

# 7.把7000节点的10个slots移到7001节点上，键入命令如下，
redis-cli -–cluster reshard 127.0.0.1:7000 -–cluster-from a7b511330bffe28357cd21d6ee543e59f0a38dea -–cluster-to a7ab1aa24c9030d1fb42bbac3ad72c15bf683ef4 –-cluster-slots 10 –cluster-yes
```

**(1) Redis集群和哨兵的区别**  
1、哨兵只提供一个主节点，同步时也是全量同步。集群的数据是分片放的，分片意味着数据是不重叠的。  
2、哨兵架构可以写主读从，但是集群架构读写都走主节点。  
3、哨兵架构收到单机数据量制约【一般不超过10个G】，集群架构可以很方便的进行水平扩展【当然操作起来还需要分配slots槽等等是有些麻烦】。  
4、哨兵架构中的哨兵角色只是起到监控的作用，但不对外提供服务，相当于占用了资源。集群的话每一台主机都是主/或从，就不会有哨兵角色的这种主机资源“浪费”。

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>         如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！  
>                 文章来源: https://weiyigeek.top  
>                 WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少。  

欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】