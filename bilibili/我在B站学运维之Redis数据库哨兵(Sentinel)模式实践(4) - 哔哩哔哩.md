**本文目录:**

0x01 Redis 运行模式

-   Redis 哨兵模式
    

-   Sentinel 介绍
    
-   Sentinel 高可用性
    
-   Sentinel 配置步骤
    
-   Sentinel 配置文件
    
-   Sentinel 常用命令
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 哨兵模式是主从的升级版，因为主从的出现故障后，不会自动恢复，需要人为干预，这就很蛋疼啊。在主从的基础上，实现哨兵模式就是为了监控主从的运行状况，对主从的健壮进行监控，就好像哨兵一样，只要有异常就发出警告，对异常状况进行处理。

描述: Redis从2.8开始发布了一个稳定版本的`Redis Sentinel` 。当前版本的 Sentinel 称为Sentinel 2。它是使用更强大和更简单的预测算法来重写初始Sentinel实现。

Redis Sentinel 是一个分布式系统，Redis Sentinel为Redis提供高可用性。可以在没有人为干预的情况下 阻止某种类型的故障。

Redis Sentinel 是redis的高可用实现方案，在实际生产环境中，对提高整个系统可用性是非常有帮助的。

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：

-   `监控(Monitoring)`: Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
    
-   `提醒(Notification)`: 当被监控的某个 Redis 服务器出现问题时，Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
    
-   `自动故障迁移（Automatic failover）`：如果 master 没有按预期工作，Sentinel 可以启动一个故障转移过程，其中一个副本被提升为 master，其他额外的副本被重新配置为使用新的 master，并且使用 Redis 服务器的应用程序会被告知要使用的新地址连接时
    
-   `配置提供程序`: Sentinel 充当客户端服务发现的权威来源：客户端连接到 Sentinel 以请求负责给定服务的当前 Redis 主节点的地址。如果发生故障转移，Sentinels 将报告新地址。
    

默认情况下，Sentinel 使用 TCP 端口 26379 运行（请注意，6379 是普通的 Redis 端口）。Sentinel 接受使用 Redis 协议的命令，因此您可以使用`redis-cli`或任何其他未修改的 Redis 客户端来与 Sentinel 对话。并且可以直接查询 Sentinel 以从其角度检查受监控 Redis 实例的状态，查看它知道的其他 Sentinel，等等。

描述: 当主节点出现故障时redis sentinel 能自动完成故障发现和故障转移，并通知客户端从而实现真正的高可用。

Redis Sentinel是一个分布式架构，其中包含N个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和其他Sentinel节点进行监控， 当它返现节点不可达时，会对节点做下线标识，如果被标识的是主节点，它还会和其他Sentinel及诶单进行“协商”，当大多数节点都认为主 节点不可达时，会选举出一个Sentinel节点来完成自动故障转移的工作，同时会将这个变化实时通知给redis的客户端，整个过程是自动的不需要人工干预，有效的解决了redis的高可用问题。

![Redis 主从和Redis Sentinel 区别](https://i0.hdslb.com/bfs/article/1818ed19246a189b7548908333f69a39044e94d1.jpg@942w_554h_progressive.webp)

同时看出，Redis Sentinel包含多个Sentinel节点，这样做带来两个好处:

-   对于节点的故障判断是由多个节点共同完成，这样可以有效的防止误判断
    
-   多个Sentinel节点出现个别节点不可用，也不会影响客户端的访问
    

**Sentinel的仲裁会**  
描述: 我们master被sentinel集群监控时，需要为它指定一个参数，该参数指定了当需要判决master为不可用，并且进行failover时，所需要的sentinel数量。

不过，当failover主备切换真正被触发后，failover并不会马上进行，还需要sentinel中的大多数sentinel授权后才可以进行failover。  
当`Objectively Down`时，failover被触发后将尝试去进行failover的sentinel会去获得“大多数”sentinel的授权（如果票数比大多数还要大的时候，则询问更多的sentinel)

例如，当集群中有 5个sentinel 票数被设置为2，当2个sentinel认为一个master已经不可用了以后，将会触发failover，但是，进行failover的那个sentinel必须先获得至少3个sentinel的授权才可以实行failover。

如果票数被设置为5，要达到ODOWN状态，必须所有5个sentinel都主观认为master为不可用，要进行failover，那么得获得所有5个sentinel的授权。

![WeiyiGeek.简单图示](https://i0.hdslb.com/bfs/article/402395a9b3ec461b20d6f232e15bddb529f27dac.png@942w_618h_progressive.webp)

**选举算法**  
Q: 当master被认为客观下线后，又是怎么进行故障恢复的呢?

> 原来哨兵中首先选举出一个老大哨兵来进行故障恢复，选举老大哨兵的算法叫做「Raft算法」：

-   发现master下线的哨兵（sentinelA）会向其它的哨兵发送命令进行拉票，要求选择自己为哨兵大佬。
    
-   若是目标哨兵没有选择其它的哨兵，就会选择该哨兵（sentinelA）为大佬。
    
-   若是选择sentinelA的哨兵超过半数（半数原则），该大佬非sentinelA莫属。
    
-   如果有多个哨兵同时竞选，并且可能存在票数一致的情况，就会等待下次的一个随机时间再次发起竞选请求，进行新的一轮投票，直到大佬被选出来。
    

选出大佬哨兵后，大佬哨兵就会对故障进行自动回复，从slave中选出一名slave作为主数据库，选举的规则如下所示：

-   1.所有的slave中slave-priority优先级最高的会被选中。
    
-   2.若是优先级相同，会选择偏移量最大的，因为偏移量记录着数据的复制的增量，越大表示数据越完整。
    
-   3.若是以上两者都相同，选择ID最小的。
    

Tips: 通过以上的层层筛选最终实现故障恢复，当选的slave晋升为master，其它的slave会向新的master复制数据，若是down掉的master重新上线，会被当作slave角色运行。

**配置版本号**  
为什么要先获得大多数sentinel的认可时才能真正去执行failover呢？

> 保证了活跃性：如果大多数sentinel能够互相通信，最终将会有一个被授权去进行failover.  
> 保证了安全性：每个试图去failover同一个master的sentinel都会得到一个独一无二的版本号。

当一个sentinel被授权后，它将会获得宕掉的master的一份最新配置版本号，当failover执行结束以后，这个版本号将会被用于最新的配置。因为大多数sentinel都已经知道该版本号已经被要执行failover的sentinel拿走了，所以其他的sentinel都不能再去使用这个版本号。这意味着每次failover都会附带有一个独一无二的版本号。我们将会看到这样做的重要性。

**节点通信**  
描述: 当哨兵启动后会与master建立一条连接，用于订阅master的`_sentinel_:hello`频道。

该频道用于获取监控该master的其它哨兵的信息,并且还会建立一条定时向master发送INFO命令获取master信息的连接。

「当哨兵与master建立连接后，定期会向（10秒一次）master和slave发送INFO命令，若是master被标记为主观下线，频率就会变为1秒一次。」

但是一个faiover要想被成功实行的前提，sentinel必须能够向选为master的slave发送`SLAVEOF NO ONE`命令，然后能够通过INFO命令看到新master的配置信息。

并且，定期`向_sentinel_:hello频道`发送自己的信息，以便其它的哨兵能够订阅获取自己的信息，发送的内容包含「哨兵的ip和端口、运行id、配置版本、master名字、master的ip端口还有master的配置版本」等信息。

以及，「定期的向master、slave和其它哨兵发送PING命令（每秒一次），以便检测对象是否存活」，若是对方接收到了PING命令，无故障情况下，会回复PONG命令。

所以，哨兵通过建立这两条连接、`通过定期发送INFO、PING命令来实现哨兵与哨兵、哨兵与master之间的通信`。

例子说明，假设有一个名为mymaster的地址为192.168.56.11:6379。一开始，集群中所有的sentinel都知道这个地址，于是为mymaster的配置打上版本号1。一段时候后mymaster死了，有一个sentinel被授权用版本号2对其进行failover。如果failover成功了，假设地址改为了192.168.56.12:6279，此时配置的版本号为2，进行failover的sentinel会将新配置广播给其他的sentinel，由于其他sentinel维护的版本号为1，发现新配置的版本号为2时，版本号变大了，说明配置更新了，于是就会采用最新的版本号为2的配置。

重要的事情说明三遍:

-   一个能够互相通信的sentinel集群最终会采用版本号最高且相同的配置。
    
-   一个能够互相通信的sentinel集群最终会采用版本号最高且相同的配置。
    
-   一个能够互相通信的sentinel集群最终会采用版本号最高且相同的配置。
    

Tips: 这里涉及到一些概念需要理解，INFO、PING、PONG等命令，后面还会有MEET、FAIL命令，以及主观下线，当然还会有客观下线，这里主要说一下这几个概念的理解：

-   INFO：该命令可以获取主从数据库的最新信息，可以实现新结点的发现
    
-   PING：该命令被使用最频繁，该命令封装了自身节点和其它节点的状态数据。
    
-   MEET：该命令在新结点加入集群的时候，会向老节点发送该命令，表示自己是个新人
    
-   PONG：当节点收到MEET和PING，会回复PONG命令，也把自己的状态发送给对方。
    
-   FAIL：当节点下线，会向集群中广播该消息。
    
-   SLAVEOF NO ONE: 不属于任何节点的从节点。
    

**上线和下线**

-   `主观下线（Subjectively Down， 简称 SDOWN）`指的是单个 Sentinel 实例对服务器做出的下线判断。
    
-   `客观下线（Objectively Down， 简称 ODOWN）`指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。 （一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线。）
    

Sentinel(哨兵) 与 master会定期一直保持联系，若是某一时刻哨兵发送的PING在指定时间内没有收到回复`（配置sentinel down-after-milliseconds master-name milliseconds 字段）`，那么发送PING命令的哨兵就会认为该master「主观下线」（Subjectively Down）。

当sentinel发送PING后，以下回复之一都被认为是合法的, `除此之外其它任何回复（或者根本没有回复）都是不合法的`。

```
* PING replied with +PONG.
* PING replied with -LOADING error.
* PING replied with -MASTERDOWN error.
```

有时 哨兵 与 master 之间的网络问题造成收发中断，而不是master本身的原因，所以哨兵同时会询问其它的哨兵是否也认为该master下线，若是认为该节点下线的哨兵达到一定的数量（`「配置的quorum字段」`），就会认为该节点「客观下线」（Objectively Down）。

从SDOWN切换到ODOWN不需要任何一致性算法，只需要一个gossip协议：如果一个sentinel收到了足够多的sentinel发来消息告诉它某个master已经down掉了，SDOWN状态就会变成ODOWN状态。如果之后master可用了，这个状态就会相应地被清理掉。

正如之前已经解释过了，真正进行failover需要一个授权的过程，但是所有的failover都开始于一个ODOWN状态。

ODOWN状态只适用于master，对于不是master的redis节点sentinel之间不需要任何协商，slaves和sentinel不会有ODOWN状态。

若是`没有足够数量的sentinel同意该master下线`，则该master客观下线的标识会被移除；`若是master重新向哨兵的PING命令`回复了客观下线的标识也会被移除。

**每个 Sentinel 都需要定期执行的任务**

-   每个 Sentinel 以每秒钟一次的频率向它所知的主服务器、从服务器以及其他 Sentinel 实例发送一个 PING 命令。
    
-   如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 那么这个实例会被 Sentinel 标记为主观下线。 一个有效回复可以是： +PONG 、 -LOADING 或者 -MASTERDOWN 。
    
-   如果一个主服务器被标记为主观下线， 那么正在监视这个主服务器的所有 Sentinel 要以每秒一次的频率确认主服务器的确进入了主观下线状态。
    
-   如果一个主服务器被标记为主观下线， 并且有足够数量的 Sentinel （至少要达到配置文件指定的数量）在指定的时间范围内同意这一判断， 那么这个主服务器被标记为客观下线。
    
-   在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有主服务器和从服务器发送 INFO 命令。 当一个主服务器被 Sentinel 标记为客观下线时， Sentinel 向下线主服务器的所有从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
    
-   当没有足够数量的 Sentinel 同意主服务器已经下线， 主服务器的客观下线状态就会被移除。 当主服务器重新向 Sentinel 的 PING 命令返回有效回复时， 主服务器的主管下线状态就会被移除。
    

**Sentinel之间和Slaves之间的自动发现机制**  
描述: 虽然sentinel集群中各个sentinel都互相连接彼此来检查对方的可用性以及互相发送消息。但是你不用在任何一个sentinel配置任何其它的sentinel的节点。

因为sentinel利用了master的发布/订阅机制(`__sentinel__:hello`)去自动发现其它也监控了统一master的sentinel节点。

同样，你也不需要在sentinel中配置某个master的所有slave的地址，sentinel会通过询问master来得到这些slave的地址的。

-   每个 Sentinel 会以每两秒一次的频率， 通过发布与订阅功能， 向被它监视的所有主服务器和从服务器的 `__sentinel__:hello` 频道发送一条信息， 信息中包含了 Sentinel 的 IP 地址、端口号和运行 ID （runid）。
    
-   每个 Sentinel 都订阅了被它监视的所有主服务器和从服务器的`__sentinel__:hello`频道， 查找之前未出现过的 sentinel （looking for unknown sentinels）。 当一个 Sentinel 发现一个新的 Sentinel 时， 它会将新的 Sentinel \* 添加到一个列表中， 这个列表保存了 Sentinel 已知的， 监视同一个主服务器的所有其他 Sentinel 。
    
-   Sentinel 发送的信息中还包括完整的主服务器当前配置（configuration）。 如果一个 Sentinel 包含的主服务器配置比另一个 Sentinel 发送的配置要旧， 那么这个 Sentinel 会立即升级到新配置上。
    
-   在将一个新 Sentinel 添加到监视主服务器的列表上面之前， Sentinel 会先检查列表中是否已经包含了和要添加的 Sentinel 拥有相同运行 ID 或者相同地址（包括 IP 地址和端口号）的 Sentinel ， 如果是的话， Sentinel 会先移除列表中已有的那些拥有相同运行 ID 或者相同地址的 Sentinel ， 然后再添加新 Sentinel 。
    

**网络隔离时的一致性**

描述: redis sentinel集群的配置的一致性模型为最终一致性，集群中每个sentinel最终都会采用最高版本的配置。然而，在实际的应用环境中，有三个不同的角色会与sentinel打交道: `Redis实例/Sentinel实例/客户端`

为了考察整个系统的行为我们必须同时考虑到这三个角色。

下面有个简单的例子，有三个主机，每个主机分别运行一个redis和一个sentinel:

```
             +-------------+
             | Sentinel 1  | <--- Client A
             | Redis 1 (M) |
             +-------------+
                     |
                     |
 +-------------+     |                     +------------+
 | Sentinel 2  |-----+-- / partition / ----| Sentinel 3 | <--- Client B
 | Redis 2 (S) |                           | Redis 3 (M)|
 +-------------+                           +------------+
```

在这个系统中，初始状态下redis3是master, redis1和redis2是slave。之后redis3所在的主机网络不可用了，sentinel1和sentinel2启动了failover并把redis1选举为master。

Sentinel集群的特性保证了sentinel1和sentinel2得到了关于master的最新配置。但是sentinel3依然持着的是就的配置，因为它与外界隔离了。

当网络恢复以后，我们知道sentinel3将会更新它的配置。但是，如果客户端所连接的master被网络隔离，会发生什么呢？

客户端将依然可以向redis3写数据，但是当网络恢复后，redis3就会变成redis的一个slave，那么，在网络隔离期间，客户端向redis3写的数据将会丢失。

也许你不会希望这个场景发生：

如果你把redis当做缓存来使用，那么你也许能容忍这部分数据的丢失。  
但如果你把redis当做一个存储系统来使用，你也许就无法容忍这部分数据的丢失了。  
因为redis采用的是异步复制，在这样的场景下，没有办法避免数据的丢失。然而,你可以通过以下配置来配置redis3和redis1，使得数据不会丢失。

```
# redis.conf
# min-replicas-to-write 3
# min-replicas-max-lag 10
# By default min-replicas-to-write is set to 0 (feature disabled) and

min-slaves-to-write 1
min-slaves-max-lag 10
```

通过上面的配置，当一个redis是master时，如果它不能向至少一个slave写数据(上面的min-slaves-to-write指定了slave的数量)，它将会拒绝接受客户端的写请求。由于复制是异步的，master无法向slave写数据意味着slave要么断开连接了，要么不在指定时间内向master发送同步数据的请求了(上面的min-slaves-max-lag指定了这个时间)。

**故障转移步骤**

一次故障转移操作由以下步骤组成：

-   发现主服务器已经进入客观下线状态。
    
-   对我们的当前纪元进行自增（详情请参考 Raft leader election ）， 并尝试在这个纪元中当选。
    
-   如果当选失败， 那么在设定的故障迁移超时时间的两倍之后， 重新尝试当选。 如果当选成功， 那么执行以下步骤。
    
-   选出一个从服务器，并将它升级为主服务器。
    
-   向被选中的从服务器发送 SLAVEOF NO ONE 命令，让它转变为主服务器。
    
-   通过发布与订阅功能， 将更新后的配置传播给所有其他 Sentinel ， 其他 Sentinel 对它们自己的配置进行更新。
    
-   向已下线主服务器的从服务器发送 SLAVEOF 命令， 让它们去复制新的主服务器。
    
-   当所有从服务器都已经开始复制新的主服务器时， 领头 Sentinel 终止这次故障迁移操作。
    

每当一个 Redis 实例被重新配置（reconfigured） —— 无论是被设置成主服务器、从服务器、又或者被设置成其他主服务器的从服务器 —— Sentinel 都会向被重新配置的实例发送一个 CONFIG REWRITE 命令， 从而确保这些配置会持久化在硬盘里。

Sentinel 使用以下规则来选择新的主服务器：

-   在失效主服务器属下的从服务器当中， 那些被标记为主观下线、已断线、或者最后一次回复 PING 命令的时间大于五秒钟的从服务器都会被淘汰。
    
-   在失效主服务器属下的从服务器当中， 那些与失效主服务器连接断开的时长超过 down-after 选项指定的时长十倍的从服务器都会被淘汰。
    
-   在经历了以上两轮淘汰之后剩下来的从服务器中， 我们选出复制偏移量（replication offset）最大的那个从服务器作为新的主服务器； 如果复制偏移量不可用， 或者从服务器的复制偏移量相同， 那么带有最小运行 ID 的那个从服务器成为新的主服务器。
    

**Sentinel状态持久化**

snetinel的状态会被持久化地写入sentinel的配置文件中。每次当收到一个新的配置时，或者新创建一个配置时，配置会被持久化到硬盘中，并带上配置的版本戳。这意味着，可以安全的停止和重启sentinel进程。

**优点**  
哨兵模式是主从模式的升级版，所以在系统层面提高了系统的可用性和性能、稳定性。当master宕机的时候，能够自动进行故障恢复，需不要人为的干预。

哨兵与哨兵之间、哨兵与master之间能够进行及时的监控，心跳检测，及时发现系统的问题，这都是弥补了主从的缺点。

**缺点**  
哨兵一主多从的模式同样也会遇到写的瓶颈，已经存储瓶颈，若是master宕机了，故障恢复的时间比较长，写的业务就会受到影响。

增加了哨兵也增加了系统的复杂度，需要同时维护哨兵模式。

描述: 哨兵模式的监控配置信息，是通过配置从数据库的`sentinel monitor <master-name> <ip> <redis-port> <quorum>`来指定的，比如：

```
sentinel monitor mymaster 10.20.172.108 6379 2

# 参数解析:
- mymaster 表示给master数据库定义了一个名字，
- master的ip 和 端口
- 2 : 表示至少需要一个Sentinel进程同意才能将master判断为失效，如果不满足这个条件，则自动故障转移（failover）不会执行
```

**运行哨兵的两种方式:**

```
# 如果您正在使用redis-sentinel可执行文件
redis-sentinel /path/to/sentinel.conf

# 或者直接使用redis-server在 Sentinel 模式下启动它的可执行文件：
redis-server /path/to/sentinel.conf --sentinel
```

Tips: 在运行 Sentinel 时必须使用配置文件，因为系统将使用该文件来保存在重启时将重新加载的当前状态。如果没有给出配置文件或配置文件路径不可写，Sentinel 将简单地拒绝启动。

Tips: Sentinel 默认运行侦听 TCP 端口 26379 的连接，因此要使Sentinel 工作，您的服务器的端口 26379必须打开以接收来自其他 Sentinel 实例的 IP 地址的连接。否则哨兵不能说话，也不能就该做什么达成一致，所以永远不会执行故障转移。

官方参考: https://redis.io/topics/sentinel

**测试环境**

```
10.20.172.108 Redis-Master 6379 |{Sentinel01} 26379
10.20.172.108 Redis-Slave 6380  |{Sentinel02} 36379
10.20.172.108 Redis-Slave 6381  |{Sentinel03} 46379
```

Tips: 此处演示只用了一台主机进行演示，在生产环境中搭建sentinel一定要在各redis服务器上配置监听，这才能真正的实现高可用。

**流程步骤**  
描述: 哨兵模式的搭建配置比较简单，在前面一章配置的主从模式的基础上，同时创建一个文件夹用于存放三个哨兵的配置文件：

-   Step 0.Redis主从服务配置文件并启动配置Redis主从服务。
    

```
# 配置&启动启动脚本
tee startRedisServer.sh <<'EOF'
#!/bin/bash
# cp /etc/redis/redis.conf /home/weiyigeek/redis/6379/redis-6379.conf
sed 's#6379#6380#g' /home/weiyigeek/redis/6379/redis-6379.conf > /home/weiyigeek/redis/6380/redis-6380.conf
sed 's#6379#6381#g' /home/weiyigeek/redis/6379/redis-6379.conf > /home/weiyigeek/redis/6381/redis-6381.conf
/usr/local/bin/redis-server /home/weiyigeek/redis/6379/redis-6379.conf
/usr/local/bin/redis-server /home/weiyigeek/redis/6380/redis-6380.conf
/usr/local/bin/redis-server /home/weiyigeek/redis/6381/redis-6381.conf
EOF

# 权限赋予与执行
chmod +x startRedisServer.sh && ./startRedisServer.sh

# 验证启动的redis服务
netstat -ano | egrep "(6379|6380|6381)"
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:6381            0.0.0.0:*               LISTEN

# 此步骤非常重要为了后续故障转移后Master->Slave服务能正常提供服务。
echo "CONFIG set masterauth 123456\nCONFIG REWRITE" | redis-cli -p 6379 -a 123456 

# 针对6380、6381服务端口设置Redis的Slave。
# 方式1
redis-cli -h 127.0.0.1 -p 6380 -a 123456
127.0.0.1:6380> SLAVEOF 10.20.172.108 6379   # OK
127.0.0.1:6380> CONFIG set masterauth 123456 # OK
127.0.0.1:6380> CONFIG REWRITE  # OK
# 方式2
➜ echo "SLAVEOF 10.20.172.108 6379\nCONFIG set masterauth 123456\nCONFIG REWRITE" | redis-cli -p 6381 -a 123456 
OK
OK
OK

# 在上面的进行CONFIG REWRITE命令执行后，将会更改其配置文件。
grep "Generated by CONFIG REWRITE" -A 5 ~/redis/6380/redis-6380.conf
.... # 在末尾加上如下几行
# Generated by CONFIG REWRITE
masterauth "123456"
user default on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all
replicaof 10.20.172.108 6379

grep "Generated by CONFIG REWRITE" -A 5 ~/redis/6381/redis-6381.conf
.... # 在末尾加上如下几行
# Generated by CONFIG REWRITE
masterauth "123456"
user default on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all
replicaof 10.20.172.108 6379

# 验证主从服务 {Master} 6379 / {Slave} 6380 8381
redis-cli -p 6379 -a 123456 info replication | head -n 5
  # role:master
  # connected_slaves:2
  # slave0:ip=10.20.172.108,port=6380,state=online,offset=448,lag=0
  # slave1:ip=10.20.172.108,port=6381,state=online,offset=448,lag=1
```

-   Step 1.Redis 哨兵Sentinel常用配置简单说明。
    

```
## 绑定主机、三个哨兵分别26379,36379,46379端口
bind 10.20.172.108
port 26379
daemonize yes

## 主目录与日志存放路径
dir "/home/weiyigeek/redis"
logfile "/home/weiyigeek/redis/sentinel_26379.log"

## 哨兵sentinel监控的redis主节点的  
## master-name：可以自己命名的主节点名字（只能由字母A-z、数字0-9 、这三个字符".-_"组成。） 
## quorum：当这些quorum个数sentinel哨兵认为master主节点失联 那么这时客观上认为主节点失联了 (最低通过票数为2) 
sentinel monitor mymaster 10.20.172.108 6379 2

## 当在Redis实例中开启了requirepass，所有连接Redis实例的客户端都要提供密码。 
sentinel auth-pass mymaster 123456

## 向master发送心跳PING来确认master是否存活，在这段时间（3s）内没有收到来自 ping 的任何回复，就会检测到主节点失败。
## 过需要注意的是，这个时候sentinel并不会马上进行failover主备切换，这个sentinel还需要参考sentinel集群中其他sentinel的意见，如果超过某个数量的sentinel也主观地认为该master死了，那么这个master就会被客观地(注意哦，这次不是主观，是客观，与刚才的subjectively down相对，这次是objectively down，简称为ODOWN)认为已经死了
sentinel down-after-milliseconds mymaster 3000

# # 指定了在发生failover主备切换时，最多可以有多少个slave同时对新的master进行同步。这个数字越小，完成failover所需的时间就越长；反之，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为1，来保证每次只有一个slave，处于不能处理命令请求的状态。 
sentinel parallel-syncs mymaster 1
# 故障转移的超时时间failover-timeout，默认三分钟，可以用在以下这些方面： 
## 1. 同一个sentinel对同一个master两次failover之间的间隔时间。   
## 2. 当一个slave从一个错误的master那里同步数据时开始，直到slave被纠正为从正确的master那里同步数据时结束。   
## 3. 当想要取消一个正在进行的failover时所需要的时间。 
## 4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来同步数据了 
sentinel failover-timeout mymaster 5000

# 当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本。一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。 
# 对于脚本的运行结果有以下规则：   
## 1. 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10。 
## 2. 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。   
## 3. 如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。 
# sentinel notification-script mymaster /var/redis/notify.sh 

# 这个脚本应该是通用的，能被多次调用，不是针对性的。 
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh 
```

-   Step 2.sentinel 配置文件配置分别生成26379、36379、46379的监听端口,以及分别启动三个端口的哨兵服务。
    

```
cat > /home/weiyigeek/redis/sentinel-26379.conf <<'EOF'
# 绑定主机、三个哨兵分别26379,36379,46379端口
bind 10.20.172.108
port 26379
daemonize yes
dir "/home/weiyigeek/redis"
logfile "/home/weiyigeek/redis/sentinel_26379.log"
sentinel monitor mymaster 10.20.172.108 6379 2
sentinel auth-pass mymaster 123456
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 10000
EOF

# 将上述 sentinel 文件配置再复制两份并更改服务端口
sed 's#26379#36379#g' /home/weiyigeek/redis/sentinel-26379.conf > /home/weiyigeek/redis/sentinel-36379.conf
sed 's#26379#46379#g' /home/weiyigeek/redis/sentinel-26379.conf > /home/weiyigeek/redis/sentinel-46379.conf

➜ ls && grep "port" sentinel-*
  # 6379  6380  6381  sentinel-26379.conf  sentinel-36379.conf  sentinel-46379.conf
  # sentinel-26379.conf:port 26379
  # sentinel-36379.conf:port 36379
  # sentinel-46379.conf:port 46379


# 服务启动脚本与启动哨兵服务
tee startSentinel.sh <<'EOF'
#!/bin/bash
/usr/local/bin/redis-server /home/weiyigeek/redis/sentinel-26379.conf --sentinel
/usr/local/bin/redis-server /home/weiyigeek/redis/sentinel-36379.conf --sentinel
/usr/local/bin/redis-server /home/weiyigeek/redis/sentinel-46379.conf --sentinel
EOF

chmod +x startSentinel.sh && ./startSentinel.sh

# 验证服务监听断开状态
redis netstat -ano | egrep "0 0.0.0.0:([234]){0,1}(6379|6380|6381)"
  # tcp        0      0 0.0.0.0:36379           0.0.0.0:*               LISTEN      关闭 (0.00/0/0)
  # tcp        0      0 0.0.0.0:46379           0.0.0.0:*               LISTEN      关闭 (0.00/0/0)
  # tcp        0      0 0.0.0.0:26379           0.0.0.0:*               LISTEN      关闭 (0.00/0/0)
  # tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      关闭 (0.00/0/0)
  # tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN      关闭 (0.00/0/0)
  # tcp        0      0 0.0.0.0:6381            0.0.0.0:*               LISTEN      关闭 (0.00/0/0)

# 也可以通过如下进行查看redis服务与哨兵启动的进程。
redis ps u -C "redis-server" 
```

-   Step 3.验证Redis Sentine服务并通过查看哨兵相关信息
    

```
# (1) Master(6379) 同步数据到 Slave(6380、6381)
$ cat /home/weiyigeek/redis/6379/logs/6379.log
#  10.20.172.108:6380
118725:M 05 Sep 2021 10:33:05.714 * Replica 10.20.172.108:6380 asks for synchronization
118725:M 05 Sep 2021 10:33:05.715 * Full resync requested by replica 10.20.172.108:6380
118725:M 05 Sep 2021 10:33:05.715 * Replication backlog created, my new replication IDs are '01bf2439d1606b8b74c2846ebb3dab02b1ab993c' and '0000000000000000000000000000000000000000'
118725:M 05 Sep 2021 10:33:05.715 * Starting BGSAVE for SYNC with target: disk
118725:M 05 Sep 2021 10:33:05.715 * Background saving started by pid 118816
118816:C 05 Sep 2021 10:33:05.756 * DB saved on disk
118816:C 05 Sep 2021 10:33:05.757 * RDB: 0 MB of memory used by copy-on-write
118725:M 05 Sep 2021 10:33:05.767 * Background saving terminated with success
118725:M 05 Sep 2021 10:33:05.767 * Synchronization with replica 10.20.172.108:6380 succeeded
118725:M 05 Sep 2021 10:34:58.126 - Accepted 10.20.172.108:36343
#  10.20.172.108:6380
118725:M 05 Sep 2021 10:34:58.152 * Replica 10.20.172.108:6381 asks for synchronization
118725:M 05 Sep 2021 10:34:58.152 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for '44d3228555abe04a02f979761bd534283d0d3613', my replication IDs are '01bf2439d1606b8b74c2846ebb3dab02b1ab993c' and '0000000000000000000000000000000000000000')
118725:M 05 Sep 2021 10:34:58.152 * Starting BGSAVE for SYNC with target: disk
118725:M 05 Sep 2021 10:34:58.152 * Background saving started by pid 118865
118865:C 05 Sep 2021 10:34:58.184 * DB saved on disk
118865:C 05 Sep 2021 10:34:58.185 * RDB: 0 MB of memory used by copy-on-write
118725:M 05 Sep 2021 10:34:58.232 * Background saving terminated with success
118725:M 05 Sep 2021 10:34:58.233 * Synchronization with replica 10.20.172.108:6381 succeeded

# (2) 查看 sentinel 日志
/redis$ more sentinel_26379.log 
10:50:43.944 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
10:50:43.944 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=119377, just started
10:50:43.944 # Configuration loaded
10:50:43.945 * Increased maximum number of open files to 10032 (it was originally set to 1024). 
10:50:43.945 * Running mode=sentinel, port=26379. # 当前模式与监听断开
10:50:44.026 # Sentinel ID is 875046c517f19aded2a5545457930ccd8ff35cb6
10:50:44.026 # +monitor master mymaster 10.20.172.108 6379 quorum 2
# 两个slave服务
10:50:44.027 * +slave slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6379
10:50:44.129 * +slave slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6379
# 其它两个sentinel实例
10:50:45.990 * +sentinel sentinel 2e53ed2e7407064d08f09b5547615144d6e34159 10.20.172.108 36379 @ mymaster 10.20.172.108 6379
10:50:46.917 * +sentinel sentinel 1c12264f05132a5627620588752a07429a635198 10.20.172.108 46379 @ mymaster 10.20.172.108 6379


# (3) 访问所有 sentinel 查看当前master节点信息
~/redis$ redis-cli -h 10.20.172.108 -p 26379 -a 123456 info | grep -n "master"
~/redis$ redis-cli -h 10.20.172.108 -p 36379 -a 123456 info | grep -n "master"
~/redis$ redis-cli -h 10.20.172.108 -p 46379 -a 123456 info | grep -n "master"
  # 85:sentinel_masters:1
  # 90:master0:name=mymaster,status=ok,address=10.20.172.108:6379,slaves=2,sentinels=3

# (4) 登陆 sentinel 26379 提供的服务
redis-cli -h 10.20.172.108 -p 26379 -a 123456
10.20.172.108:26379> sentinel masters              # 查看所有Master状态
10.20.172.108:26379> sentinel master mymaster      # 查看指定Master状态
10.20.172.108:26379> SENTINEL replicas mymaster    # 查看备选slave节点 
10.20.172.108:26379> SENTINEL sentinels mymaster   # 查看其它哨兵状态
1)  1) "name"
    2) "2e53ed2e7407064d08f09b5547615144d6e34159"
    3) "ip"
    4) "10.20.172.108"
    5) "port"
    6) "36379"
    7) "runid"
    8) "2e53ed2e7407064d08f09b5547615144d6e34159"
    省略.....
2)  1) "name"
    2) "1c12264f05132a5627620588752a07429a635198"
    3) "ip"
    4) "10.20.172.108"
    5) "port"
    6) "46379"
    7) "runid"
    8) "1c12264f05132a5627620588752a07429a635198"
    省略.....
  
# (5) 获取当前master的地址
10.20.172.108:26379> SENTINEL get-master-addr-by-name mymaster
1) "10.20.172.108"
2) "6379"
```

-   Step 4.在Master节点上添加一个Keys验证主从正常同步
    

```
# 1.在master 6379中添加一个key
10.20.172.108:6379> set username WeiyiGeek 
OK

# 2.在Slave 6380 中查询添加key
10.20.172.108:6380> get username    # "WeiyiGeek"
10.20.172.108:6380> set demo redis  # 注意从节点只能读不能写(主节点采有写)
  # (error) READONLY You cant write against a read only replica.  

# 3.在Slave 6381 中查询添加key
echo "get username" | redis-cli -p 6381 -a 123456 #  # "WeiyiGeek"
```

-   Step 5.从上面流程验证了redis主从同步以及哨兵服务正常运行，下面我们将进行演示哨兵模式故障转移(`验证其高可用`)。
    

```
# 1.模拟故障转移两种方式
# 方式1.此命令将使我们的 master 不再可访问，休眠 60 秒。它基本上模拟了出于某种原因挂起的主人。
# redis-cli -p 6379 DEBUG sleep 60
# 方式2.直接在哨兵服务中模拟故障转移（重新仲裁Master节点）
sentinel failover mymaster

# 2.查看 sentinel 日志
10:58:23.402 # Executing user requested FAILOVER of 'mymaster'
10:58:23.402 # +new-epoch 1
10:58:23.402 # +try-failover master mymaster 10.20.172.108 6379
10:58:23.457 # +vote-for-leader 875046c517f19aded2a5545457930ccd8ff35cb6 1
# step 1
10:58:23.457 # +elected-leader master mymaster 10.20.172.108 6379
# step 2
10:58:23.457 # +failover-state-select-slave master mymaster 10.20.172.108 6379
# Step 3.从slave中选举出 6380 服务的redis
10:58:23.534 # +selected-slave slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6379
# step 4.设置 6380 不为从节点
10:58:23.534 * +failover-state-send-slaveof-noone slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6379
10:58:23.589 * +failover-state-wait-promotion slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6379
10:58:24.522 # +promoted-slave slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6379
# step 5.将原master实例转为slave
10:58:24.522 # +failover-state-reconf-slaves master mymaster 10.20.172.108 6379
10:58:24.550 * +slave-reconf-sent slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6379
10:58:25.542 * +slave-reconf-inprog slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6379
10:58:25.542 * +slave-reconf-done slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6379
10:58:25.625 # +failover-end master mymaster 10.20.172.108 6379
# Step 6.将6379master服务切换到10.20.172.108 6380中
10:58:25.625 # +switch-master mymaster 10.20.172.108 6379 10.20.172.108 6380
# Step 7.切换后其它两个slave信息
10:58:25.625 * +slave slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6380
10:58:25.625 * +slave slave 10.20.172.108:6379 10.20.172.108 6379 @ mymaster 10.20.172.108 6380

# 3.此时看6379服务的日志（已将其配置为从节点）
grep "Generated by CONFIG REWRITE" -A 5 ~/redis/6379/redis-6379.conf
.... # 在末尾加上如下几行
# Generated by CONFIG REWRITE
user default on #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all
replicaof 10.20.172.108 6380
masterauth "123456"
```

-   Step 6.向 Sentinel 询问 master 的状态，在开始使用 Sentinel 最明显的事情是检查它正在监视的 master 是否运行良好
    

```
# (1) 在新的 Master 节点 6380 中新增一个keys并查询其包括的slave
$ redis-cli -h 10.20.172.108 -p 6380 -a 123456 info replication | head -n 5
# Replication
role:master
connected_slaves:2
slave0:ip=10.20.172.108,port=6379,state=online,offset=2396601,lag=0
slave1:ip=10.20.172.108,port=6381,state=online,offset=2396601,lag=0

$ redis-cli -h 10.20.172.108 -p 6380 -a 123456
127.0.0.1:6380> set demo1 sentinel
OK
127.0.0.1:6380> get username 
weiyigeek
127.0.0.1:6380> keys *
1) "username"
2) "demo1"

# (2) 哨兵服务选举出来的当前master节点信息
~/redis$ redis-cli -h 10.20.172.108 -p 26379 -a 123456 info | grep -n "master"
  # 85:sentinel_masters:1
  # 90:master0:name=mymaster,status=ok,address=10.20.172.108:6380,slaves=2,sentinels=3
```

-   Step 7.然后我们再来测试一下哨兵的自动故障恢复，直接kill掉6380服务端口的进程。
    

```
# (1) pid进程号查看并kill掉
~/redis$ ps uax | grep "6380"
weiyige+  119353  0.4  0.0 146544  6168 ?        Ssl  10:50   0:53 /usr/local/bin/redis-server 0.0.0.0:6380
~/redis$ kill 119353

# (2) 查哨兵新选举出来的Master地址
10.20.172.108:26379>  SENTINEL get-master-addr-by-name mymaster
1) "10.20.172.108"
2) "6381"

# (3) 查看sentinel日志(可以看到6380服务主动下线、与客观下线)
13:58:50.367 # +sdown master mymaster 10.20.172.108 6380
13:58:50.451 # +odown master mymaster 10.20.172.108 6380 #quorum 2/2
13:58:50.451 # +new-epoch 2
13:58:50.451 # +try-failover master mymaster 10.20.172.108 6380
13:58:50.504 # +vote-for-leader 875046c517f19aded2a5545457930ccd8ff35cb6 2
13:58:50.693 # 2e53ed2e7407064d08f09b5547615144d6e34159 voted for 875046c517f19aded2a5545457930ccd8ff35cb6 2
13:58:50.719 # 1c12264f05132a5627620588752a07429a635198 voted for 875046c517f19aded2a5545457930ccd8ff35cb6 2
13:58:50.727 # +elected-leader master mymaster 10.20.172.108 6380
13:58:50.727 # +failover-state-select-slave master mymaster 10.20.172.108 6380
13:58:50.803 # +selected-slave slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6380
13:58:50.803 * +failover-state-send-slaveof-noone slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6380
13:58:50.874 * +failover-state-wait-promotion slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6380
13:58:51.563 # +promoted-slave slave 10.20.172.108:6381 10.20.172.108 6381 @ mymaster 10.20.172.108 6380
13:58:51.563 # +failover-state-reconf-slaves master mymaster 10.20.172.108 6380
13:58:51.568 * +slave-reconf-sent slave 10.20.172.108:6379 10.20.172.108 6379 @ mymaster 10.20.172.108 6380
13:58:51.843 # -odown master mymaster 10.20.172.108 6380
13:58:52.529 * +slave-reconf-inprog slave 10.20.172.108:6379 10.20.172.108 6379 @ mymaster 10.20.172.108 6380
13:58:52.529 * +slave-reconf-done slave 10.20.172.108:6379 10.20.172.108 6379 @ mymaster 10.20.172.108 6380
13:58:52.582 # +failover-end master mymaster 10.20.172.108 6380
13:58:52.582 # +switch-master mymaster 10.20.172.108 6380 10.20.172.108 6381
13:58:52.582 * +slave slave 10.20.172.108:6379 10.20.172.108 6379 @ mymaster 10.20.172.108 6381
13:58:52.582 * +slave slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6381
13:58:57.611 # +sdown slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6381


# (4) 登陆6381进行验证角色及其权限(此时6379变成了从节点、6380不在线)
redis-cli -p 6381 -a 123456 info replication  | head -n 5
  # Replication
  role:master
  connected_slaves:1
  slave0:ip=10.20.172.108,port=6379,state=online,offset=2456277,lag=0

# (5) 在 6981 master节点上新增一个key 
echo "set newname weiyigeek" | redis-cli -h 10.20.172.108 -p 6381 -a 123456
OK

# (6) 在slave 6380 中查询上面插入的键
10.20.172.108:6380> keys *
1) "newname"
2) "username"
3) "demo1"
10.20.172.108:6380> get newname
"weiyigeek"

# (7) 现在恢复我们杀掉的6380服务
~/redis$ /usr/local/bin/redis-server /home/weiyigeek/redis/6380/redis-6380.conf 
# 查看其redis服务日志
~/redis$ tail -f 50 /home/weiyigeek/redis/6381/redis-6381.conf 
# Generated by CONFIG REWRITE
masterauth "123456"
user default on sanitize-payload #8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 ~* &* +@all

# (8) 查看 sentinel 服务可以看到6380以及上线且设置其连接的master实例节点为10.20.172.108 6381
14:04:23.511 # -sdown slave 10.20.172.108:6380 10.20.172.108 6380 @ mymaster 10.20.172.108 6381

# (9) 6381 redis 已切换为master角色
$ redis redis-cli -p 6381 -a 123456 info replication  | head -n 5
# Replication
role:master
connected_slaves:2
slave0:ip=10.20.172.108,port=6379,state=online,offset=2456277,lag=0
slave1:ip=10.20.172.108,port=6380,state=online,offset=2456277,lag=0

# (10) 在从故障恢复的6380服务加入到当前主从服务，并查询之前在master节点插入的newname键。
➜  redis redis-cli -p 6380 -a 123456 get newname                   
"weiyigeek"
```

-   Step 8.至此哨兵的故障恢复实践完毕，下面是一些注意事项:
    

-   sentinel 节点不要部署在同一台机器。
    
-   至少不是三个且奇数个的 sentinel 节点，增加选举的准确性因为领导者选举需要至少一半加1个节点。
    
-   sentinel节点集合可以只监控一个主节点，也可以监控多个主节点，尽量使用一套sentinel监控一个主节点。
    
-   sentinel的数据节点与普通的 redis 数据节点没有区别。
    
-   客户端初始化连接的是 Sentinel节点集合，不再是具体的redis节点，但是Sentinel 是配置中心不是代理。
    

描述：在redis源码包中自带一个`sentinel.conf`配置文件示例。

```
# Example sentinel.conf

# *** IMPORTANT ***
# 默认情况下，无法从不同于本地主机的接口访问Sentinel，请使用“bind”指令绑定到网络接口列表，或通过将其添加到此配置文件来禁用带有“protected mode no”的protected mode。
bind 127.0.0.1 192.168.1.1
protected-mode no

# port <sentinel-port> : The port that this sentinel instance will run on
port 26379

# By default Redis Sentinel does not run as a daemon. 
# 设置yes后将生成 /var/run/redis-sentinel.pid
daemonize yes

# 自定义 pid file 在本地路径
pidfile /var/run/redis-sentinel.pid

# 指定日志文件名。
# 如使用 "" 标准输出
# 如使用 daemonize 则日志将发送到 /dev/null.
logfile ""


# 由于NAT，Sentinel可以通过非本地地址从外部访问。当提供了通告ip时，Sentinel将在用于告知其存在的HELLO消息中声明指定的ip地址，而不是像通常那样自动检测本地地址。
# Example:
# sentinel announce-ip 1.2.3.4
# sentinel announce-port 26379

# sentinel 数据相关文件存储目录
# dir <working-directory>
dir /tmp

# 设置要得监控主控名称以及服务地址端口、并设置quorum(票数)
# 注意：主控名称不应包含特殊字符或空格，有效字符集为A-z 0-9和三个字符“-\”。
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2

# 设置用于向主机和副本进行身份验证的密码（与服务端的密码保持一致）。
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# 对于向具有ACL功能的实例进行身份验证非常有用，并且运行Redis 6.0或更高版本，当提供了just auth pass时 它将使用 “AUTH<user><pass>” (一般情况下设置的是密码)
# sentinel auth-user <master-name> <username>
# user sentinel-user >somepassword +client +subscribe +publish \
#                    +ping +info +multi +slaveof +config +client +exec on


# 主（或任何附加的副本或哨兵）应是不可达的（如在指定的时间段内连续不可接受的答复），以考虑它在Sy-DOWN状态（主观下线），默认30s.
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000

# IMPORTANT NOTE: starting with Redis 6.2 ACL capability is supported for Sentinel mode, 
# 参考地址: https://redis.io/topics/acl
# Sentinel's ACL users are defined in the following format:
#   user <username> ... acl rules ...
# For example:
#   user worker +@admin +@connection ~* on >ffa9203c493aa99
# For more information about ACL configuration please refer to the Redis
# website at https://redis.io/topics/acl and redis server configuration 
# template redis.conf.

# ACL日志跟踪失败的命令和与ACL关联的身份验证事件, 在下面定义ACL日志的最大条目长度：
acllog-max-len 128

# 使用外部ACL用户文件并且格式与redis.conf中用于描述用户的格式完全相同。
aclfile /etc/redis/sentinel-users.acl

# requirepass 与 aclfile选项和ACL LOAD命令不兼容，在使用acl如设置该字段将被忽略。
# requirepass <password>
# 建议新配置文件对传入连接（通过ACL）和传出连接（通过sentinel user和sentinel pass）使用单独的身份验证控制

# 将Sentinel配置为与具有特定用户名的其他Sentinel进行身份验证。
sentinel sentinel-user <username>

# Sentinel与其他Sentinel进行身份验证的密码。如果未配置sentinel用户，sentinel将使用具有sentinel pass的“默认”用户进行身份验证。
sentinel sentinel-pass <password>


# 在故障切换过程中，我们可以重新配置多少副本以同时指向新副本。如果使用副本来提供查询，请使用较低的数字，以避免在执行与主服务器的同步时，几乎同时无法访问所有副本。
# sentinel parallel-syncs <master-name> <numreplicas>
sentinel parallel-syncs mymaster 1

# 以毫秒为单位指定故障转移超时（默认三分钟)
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000


# SCRIPTS EXECUTION
# sentinel通知脚本和sentinel reconfig脚本用于配置在故障转移后被调用以通知系统管理员或重新配置客户端的脚本。使用以下错误处理规则执行脚本：
# 如果脚本以“1”退出，则稍后将重试执行（最大次数当前设置为10）。
# 如果脚本以“2”（或更高的值）退出，则不会重试脚本执行。
# 如果脚本因接收到信号而终止，则行为与退出代码1相同。
# 脚本的最大运行时间为60秒。达到此限制后，脚本将以SIGKILL终止，并重试执行。

# NOTIFICATION SCRIPT | 通知执行
# 为警告级别（例如-sdown、-odown等）中生成的任何sentinel事件调用指定的通知脚本
# Example:
# sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh


# CLIENTS RECONFIGURATION SCRIPT | 客户端重新配置脚本（可以多次调用）
# 由于故障切换而更改主机时，可以调用脚本来执行特定于应用程序的任务，以通知客户端配置已更改且主机位于不同的地址。
# ip、from port、to ip、to port 的参数用于传递主机的旧地址和所选副本（现在是主机）的新地址。
# The following arguments are passed to the script:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# <state> is currently always "failover"
# <role> is either "leader" or "observer"
# Example:
# sentinel client-reconfig-script <master-name> <script-path>
 sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# SECURITY
# 默认情况下，SENTINEL SET将无法在运行时更改通知脚本和客户端重新配置脚本。这避免了一个微不足道的安全问题，客户机可以将脚本设置为任何值，并触发故障转移以执行程序。
sentinel deny-scripts-reconfig yes

# REDIS COMMANDS RENAMING 
# 有时 Redis服务器会将Sentinel正常工作所需的某些命令重命名为不可用字符串。在将Redis作为服务提供的提供者的上下文中，CONFIG和SLAVEOF通常是这种情况，并且不希望客户在管理控制台之外重新配置实例。
# 在这种情况下，可以告诉Sentinel使用不同的命令名，而不是普通的命令名。
# 例如，如果主机“mymaster”和相关副本的“CONFIG”全部重命名为“GUESSME”，我可以使用如下配置，设置好这样的配置后，Sentinel每次使用CONFIG时都会使用GUESSME。
SENTINEL rename-command mymaster CONFIG GUESSME

# SENTINEL SET也可用于在运行时执行此配置。
# 为了将命令设置回其原始名称（撤消重命名），可以只将命令重命名为其自身：
SENTINEL rename-command mymaster CONFIG CONFIG

# HOSTNAMES SUPPORT
# 通过启用解析主机名来启用主机名支持。请注意，您必须确保DNS配置正确，并且DNS解析不会引入很长的延迟。
SENTINEL resolve-hostnames no

# 启用“解析主机名”时，Sentinel在向用户、配置文件等公开实例时仍使用IP地址。如果要在宣布时保留主机名，请启用下面的“宣布主机名”。
SENTINEL announce-hostnames no
```

描述: Sentinel 常用API命令(https://redis.io/topics/sentinel#Sentinel API),以下列出的是Sentinel接受的命令：

**常用命令**

(1) 出于连接管理和管理目的，Sentinel 支持以下 Redis 命令子集：

-   **PING**：返回PONG。
    
-   **COMMAND** ( `>= 6.2`) 此命令返回有关命令的信息。有关详细信息，请参阅COMMAND命令及其各种子命令。
    
-   **INFO** 返回有关 Sentinel 服务器的信息和统计信息。有关更多信息，请参阅INFO命令。
    
-   **ROLE** 此命令返回字符串“sentinel”和受监控主站的列表。有关更多信息，请参阅ROLE命令。
    
-   **AUTH** ( `>= 5.0.1`) 验证客户端连接。有关更多信息，请参阅AUTH命令和使用身份验证配置 Sentinel 实例部分。
    
-   **HELLO** ( `>= 6.0`) 切换连接的协议。有关更多信息，请参阅HELLO命令。
    
-   **CLIENT** 此命令管理客户端连接。有关更多信息，请参阅其子命令页面。
    
-   **ACL** ( >= 6.2) \`此命令管理 Sentinel 访问控制列表。有关详细信息，请参阅ACL文档页面和Sentinel 访问控制列表身份验证。
    
-   **SHUTDOWN** 关闭 Sentinel 实例。
    

(2) SENTINEL命令是 Sentinel 的主要 API

-   `SENTINEL masters`：用于查看监控的所有Redis Master信息，包括配置和状态等。
    
-   `SENTINEL master <master name>`： 显示指定主机的状态和信息。
    
-   `SENTINEL slaves`：列出给定主服务器的所有从服务器，以及这些从服务器的当前状态。
    
-   `SENTINEL replicas <master name>` ( >= 5.0) 显示此主服务器的副本列表及其状态。
    
-   `SENTINEL sentinels`：查看给定主服务器的Sentinel实例列表及其状态。
    
-   `SENTINEL sentinel <master name>`: 显示此 master 的哨兵实例列表及其状态。
    
-   `SENTINEL monitor`: 启动 Sentinel 的监控。参阅在运行时重新配置 Sentinel部分。
    
-   `SENTINEL MONITOR<name> <ip> <port> <quorum>`：此命令告诉 Sentinel 开始监视具有指定名称、IP、端口和仲裁的新主服务器。
    
-   `SENTINEL myid` ( >= 6.2): 返回 Sentinel 实例的 ID。
    
-   `SENTINEL get-master-addr-by-name`：返回给定名字的主服务器的IP地址和端口号。 如果这个主服务器正在执行故障转移操作，或者针对这个主服务器的故障转移操作已经完成，那么这个命令返回新的主服务器的IP地址和端口号。
    
-   `SENTINEL is-master-down-by-addr`：一个Sentinel可以通过向另一个Sentinel发送SENTINEL is-master-down-by-addr命令来询问对方是否认为给定的服务器已下线。
    
-   `SENTINEL failover`：强制发起一次某个Master的failover，如果该Master不可访问的话。
    
-   `SENTINEL failover <master name>`: 强制故障转移，就好像主节点不可访问一样，并且不要求其他 Sentinel 同意（但是，将发布新版本的配置，以便其他 Sentinel 更新其配置）。
    
-   `SENTINEL reset`：强制重设所有监控的Master状态，清除已知的Slave和Sentinel实例信息，重新获取并生成配置文件。
    
-   `SENTINEL reset <pattern>`：重置所有名字和给定模式pattern相匹配的主服务器。pattern 参数是一个Glob风格的模式。重置操作清除主服务器目前的所有状态，包括正在执行中的故障转移，并移除目前已经发现和关联的，主服务器的所有从服务器和Sentinel。
    
-   `SENTINEL ckquorum <master name>`: 检查当前 Sentinel 配置是否能够达到故障转移主服务器所需的法定人数，以及授权故障转移所需的大多数人数。此命令应用于监控系统以检查 Sentinel 部署是否正常。
    
-   `SENTINEL PENDING-SCRIPTS`: 此命令返回有关挂起脚本的信息。
    
-   `SENTINEL SIMULATE-FAILURE (crash-after-election|crash-after-promotion|help)` ( >= 3.2): 此命令模拟不同的 Sentinel 崩溃场景。
    

(3) 动态获取修改Sentinel配置

-   `SENTINEL CONFIG GET<name>` ( `>= 6.2`) 获取全局 Sentinel 配置参数的当前值。指定的名称可以是通配符，类似于Redis CONFIG GET命令。
    
-   `SENTINEL CONFIG <name> <value>`: SET命令与Redis的CONFIG SET命令非常相似，用于更改特定master 的配置参数。可以指定多个选项/值对（或根本不指定）。可以通过配置的所有配置参数也可以sentinel.conf使用 SET 命令进行配置。
    
-   `SENTINEL FLUSHCONFIG`: 强制 Sentinel 在磁盘上重写其配置，包括当前 Sentinel 状态。通常 Sentinel 会在每次其状态发生变化时重写配置（在状态子集的上下文中，该子集在重新启动时保留在磁盘上）。但是有时可能会因为操作错误、磁盘故障、包升级脚本或配置管理器而导致配置文件丢失。在这些情况下，强制 Sentinel 重写配置文件的方法很方便。即使之前的配置文件完全丢失此命令也能工作。
    

Tips 可以操作的全局参数包括：

-   resolve-hostnames, announce-hostnames. 请参阅IP 地址和 DNS 名称。
    
-   announce-ip, announce-port. 请参阅Sentinel、Docker、NAT 和可能的问题。
    
-   sentinel-user, sentinel-pass. 请参阅使用身份验证配置 Sentinel 实例。
    

(4) 移除master监控服务或者删除旧的 master 或无法访问的副本

-   `SENTINEL REMOVE <name>` : 用于移除指定的master：master将不再被监控（放弃对某个master的监听）。
    
-   `SENTINEL RESET mastername`: 从 Sentinels 监视的副本列表中永远删除一个副本（可能是旧的 master）。
    

(5) 添加或删除哨兵  
描述: 由于 Sentinel 实现了自动发现机制，因此将新 Sentinel 添加到您的部署中是一个简单的过程。您需要做的就是启动配置为监视当前活动主站的新 Sentinel。在 10 秒内，Sentinel 将获取其他 Sentinel 的列表以及附加到 master 的副本集。在该过程结束时，可以使用该命令 `SENTINEL MASTER mastername` 来检查所有 Sentinel 是否同意监视 master 的 Sentinel 总数。

移除 Sentinel 有点复杂：Sentinels 永远不会忘记已经看到的 Sentinels，即使它们很长时间无法访问，因为我们不想动态更改授权故障转移和创建新配置所需的多数数字。因此为了删除 Sentinel，应在没有网络分区的情况下执行以下步骤：

-   停止要移除的 Sentinel 的 Sentinel 进程。
    
-   `SENTINEL RESET *` 向所有其他 Sentinel 实例发送命令（\*如果您只想重置一个主服务器，则可以使用确切的主服务器名称）。一个接一个，实例之间至少等待 30 秒。
    
-   通过检查`SENTINEL MASTER mastername` 每个 Sentinel的输出，检查所有 Sentinel 是否同意当前活动的 Sentinel 数量。
    

**发布/订阅消息**  
描述: 客户端可以使用 Sentinel 作为与 Redis 兼容的发布/订阅服务器（但您不能使用PUBLISH），以便订阅或PSUBSCRIBE到频道并获得有关特定事件的通知。

以下是您可以使用此 API 接收的频道和消息格式的列表。第一个字是通道/事件名称，其余的是数据的格式。  
`<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>`  
注意：指定实例详细信息的地方意味着提供以下参数来标识目标实例：

标识 master 的部分（从 @ 参数到结尾）是可选的，并且仅在实例本身不是 master 时才指定。

```
+reset-master <instance details> -- 主机被重置。
+slave <instance details> -- 检测到并附加了一个新副本。
+failover-state-reconf-slaves <instance details> -- 故障转移状态更改为reconf-slavesstate。
+failover-detected <instance details> --检测到由另一个 Sentinel 或任何其他外部实体启动的故障转移（一个附加的副本变成了一个主）。
+slave-reconf-sent <instance details> -- 领导哨兵向该实例发送REPLICAOF命令，以便为新副本重新配置它。
+slave-reconf-inprog <instance details> -- 正在重新配置的副本显示为新主 ip:port 对的副本，但同步过程尚未完成。
+slave-reconf-done <instance details> -- 副本现在与新的主服务器同步。
-dup-sentinel <instance details> -- 指定 master 的一个或多个 sentinel 被删除为重复的（例如，当 Sentinel 实例重新启动时会发生这种情况）。
+sentinel <instance details> -- 检测到并附加了此主人的新哨兵。
+sdown <instance details> -- 指定的实例现在处于主观关闭状态。
-sdown <instance details> -- 指定的实例不再处于主观关闭状态。
+odown <instance details> -- 指定的实例现在处于 Objectively Down 状态。
-odown <instance details> -- 指定的实例不再处于 Objectively Down 状态。
+new-epoch <instance details> -- 当前纪元已更新。
+try-failover <instance details> -- 正在进行新的故障转移，等待多数人选举。
+elected-leader <instance details> -- 赢得指定时期的选举，可以进行故障转移。
+failover-state-select-slave <instance details> -- 新的故障转移状态是select-slave：我们正在尝试寻找合适的副本进行升级。
no-good-slave <instance details> -- 没有好的副本可以推广。目前我们会在一段时间后尝试，但可能这会改变，在这种情况下状态机将完全中止故障转移。
selected-slave <instance details> -- 我们找到了要提升的指定好的副本。
failover-state-send-slaveof-noone -- <instance details>我们正在尝试将提升的副本重新配置为主副本，等待它切换。
failover-end-for-timeout -- <instance details>故障转移因超时而终止，副本最终将被配置为与新的主节点进行复制。
failover-end <instance details> -- 故障转移成功终止。所有副本似乎都被重新配置为使用新的主副本进行复制。
switch-master <master name> <oldip> <oldport> <newip> <newport> -- 主新 IP 和地址是配置更改后指定的。这是大多数外部用户感兴趣的消息。
+tilt -- 进入倾斜模式。
-tilt -- 退出倾斜模式。
```

Tips ：-BUSY 状态的处理，当 Lua 脚本运行的时间超过配置的 Lua 脚本时间限制时，Redis 实例会返回 -BUSY 错误。在触发故障转移之前发生这种情况时，Redis Sentinel 将尝试发送SCRIPT KILL 命令，只有在脚本为只读时才会成功。如果在此尝试后实例仍处于错误状态，则最终将进行故障转移。

**命令使用演示:**

```
# 1.展示所有被监控的主节点状态以及相关的统计信息
> sentinel masters

# 2.展示指定<master name>的主节点状态以及相关的统计信息，例如：
> sentinel master mymaster

# 3.展示指定<master name>的从节点状态以及相关的统计信息
> sentinel slaves mymaster

# 4.展示指定<master name>的Sentinel节点集合（不包含当前Sentinel节点），例如：
> sentinel sentinels mymaster

# 5.返回指定<master name>主节点的IP地址和端口
> sentinel get-master-addr-by-name mymaster

# 6.当前Sentinel节点对符合 <pattern>（通配符风格）主节点的配置进行重置，包含清除主节点的相关状态（例如故障转移），重新发现从节点和 Sentinel节点。
# 例如，sentinel-1节点对mymaster-1节点重置状态如下：
> sentinel reset mymaster # integer 1

# 7.对指定<master name>主节点进行强制故障转移（没有和其他Sentinel节 点“协商”）
# 例如，对 mymaster 进行故障转移， 执行命令后mymaster由原来的一个从节点6380代替。
> sentinel failover mymaster

# 8.检测当前可达的Sentinel节点总数是否达到的个数。例如 quorum=3，而当前可达的Sentinel节点个数为2个，那么将无法进行故障转 移，Redis Sentinel的高可用特性也将失去。
> sentinel ckquorum mymaster

# 9.将Sentinel节点的配置强制刷到磁盘上，这个命令Sentinel节点自身用得 比较多，对于开发和运维人员只有当外部原因（例如磁盘损坏）造成配置文 件损坏或者丢失时，这个命令是很有用的。
> sentinel flushconfig


# 10.取消当前Sentinel节点对于指定 <master name>主节点的监控。
# 例如，sentinel-1当前对mymaster进行了监控，sentinel-1节点取消对mymaster节点的监控，但是要注意这 个命令仅仅对当前Sentinel节点有效。
> sentinel remove mymaster

# 11.通过命令的形式 来完成Sentinel节点对主节点的监控。
# 例如,命令sentinel-1节点重新监控mymaster节点：
> sentinel monitor mymaster 127.0.0.1 6379 2

# 12.Sentinel节点之间用来交换对主节点是否下线的判断。
> sentinel is-master-down-by-addr

# 13.动态修改Sentinel节点配置选项。
> sentinel set <master name>
```

![WeiyiGeek.sentinel相关命令实践](https://i0.hdslb.com/bfs/article/52ea6469342f80cc1252a5688ae52776393d70eb.png@942w_675h_progressive.webp)

![](https://i0.hdslb.com/bfs/article/db75225feabec8d8b64ee7d3c7165cd639554cbc.png@progressive.webp)

>     如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏，这将对我有很大帮助！  
>             文章来源: https://weiyigeek.top  
>             WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少。

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】