　　　　　　　　　　　　　　**《Apache Kafka实战》读书笔记-调优Kafka集群**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.确定调优目标**

**1>.常见的非功能性要求**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
一.性能（performance）
    最重要的非功能性需求之一。大多数生产环境对集群性能都有着严格的要求。不同的系统对于性能有着不同的诉求。比如对数据库系统来说，最重要的性能是请求的响应时间（response time）。用户总是希望一条查询或更新操作的整体响应时间越短越好；而对kafak而言，性能一般指的是吞吐量和延时两个方面
1>.吞吐量（throughput/TPS）
    broker或clients应用程序每秒能处理多少字节（或消息）。
2>.延时（latency）
    通常指代producer端发送消息到broker端持久化保存消息之间的时间间隔。该概念也用于统计端到端（end-to-end，E2E）的延时，比如producer端发送一条消息到consumer端消费这条消息的时间间隔。

二.可用性
    在某段时间内，系统或组建正常运行的概率或在时间上的比率。业界一般用N个9来量化可用性，比如常见的“年度4个9”即指系统可用性要达到99.99%, 即一年中系统宕机的时间不能超过53分钟（365 x 24 x 60 x 0.01% = 53）.

三.持久性
    确保了已提交操作系统产生的消息需要被持久化的保持，即使系统出现崩溃。对于kafka来说，持久性通常意味着已提交的消息需要持久化到broker底层的物理日志而不能发送丢失。
    
    由于篇幅有限，上述仅仅罗列出了对Kafka非常重要的的非功能性需求，接下来我们会进行详细的展开来讨论如何从这些方面调优kafka集群。
     
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.只有明确目标才能明确调优的方向**

 　　如上所述，我们需要从四个方面来考虑调优目标：吞吐量，延时，持久性和可用性。为了明确用户生产环境中的目标，用户需要结合业务仔细思考Kafka集群的使用场景和初衷。比如使用kafka集群的目的是什么？是作为一个消息队列使用，还是作为数据存储，抑或是用作流失数据处理，更或是以上所有？

明确这个目标有两个重要的作用：

　　 2.1>.万物皆有度，世界上没有十全十美的事情。你不可能同时最大化上述四个目标。它们彼此之间可能是矛盾的，到底看重哪个方面实际上是一个权衡选择（trade-off）。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
举个例子：
    假设producer每秒发送一条消息需要花费2毫秒（即延时是2毫秒），那么显然producer的吞吐量就应该是500条/秒，因为1秒可以发送1/0.002=500条消息。因此，吞吐量和延时的关系似乎可以使用共识来表示：TPS=1000/Latency（毫秒）。

    其实，两者的关系远非上面公司表示的那么简单。我们依然以kafka producer来举例子，假设它仍然 以2毫秒的延时来发送消息。如果每秒之发送一条消息，那么TPS自然就是500条/秒。由此可见，延时增大了4倍，但TPS却增大的了将近200倍。

    上面的场景解释了目前为什么批次化（batchingi）以及微批次话（micro-batchingh）流行的原因。实际环境中用户几乎总是愿意用增加较小延时的待见去换取TPS的显著提升。毕竟从2毫秒到10毫秒的延迟增加通常是可以忍受的。值得一提的是，kafka producer也采取了这样的理念，这里的9毫秒就是producer等待8毫秒积攒出的消息数远远多于同等时间内producer能够发送的消息数。

    有的读者会问：producer等待8毫秒就能积累1000条消息吗？不是发送一条消息就需要2毫秒吗？这里需要解释一下，producer累积消息一般仅仅是将消息发送到内存中的缓冲区，而发送消息却需要设计网络I/O传输。内存操作和I/O操作的时间量级不是不同的，前者通常是基百纳秒级别，而后者从毫秒到秒级别不等，故producer等待8毫秒积攒出的消息数远远多于同等时间内producer能够发送的消息数。
    
    说了这么多其实就想强调上面4个调优目标统筹规划时互相关联，互相制约的。当然，这种制约觉得不是完全互斥，即提高一个目标一定会使其他目标降低。换句话说，就是用户不能同时使这4个目标达到最优成都。这就是在调优前明确优化目标的第一个含义。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　 2.2>.用户需要明确调优的重点才能针对性的调整不同的Kafka参数。Kafka提供的各类参数已达几百个之多。只有我们明确要调优哪些方面才能确定适合的参数。比如如果要为集群中所有topic进行优化，那么就需要调整broker的参数，而如果是只是为某些topic进行优化，

则需要调整topic级别的参数。

**二.集群基础调优**

　　 可参考我之前的笔记：[**《Kafka权威指南》读书笔记-操作系统调优篇（https://www.cnblogs.com/yinzhengjie/p/9993719.html）**](https://www.cnblogs.com/yinzhengjie/p/9993719.html)

 　　关于内核调优的参数，我只是微调了一下，仅仅调试了12个内核参数，如下：（仅供参考，大家可以根据自己的实际环境调试相应的内核参数）

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@yinzhengjie ~]# tail -13 /etc/sysctl.conf    
#Add by yinzhengjie
net.ipv6.conf.all.disable_ipv6=1
vm.dirty_ratio = 80
vm.dirty_background_ratio = 5
vm.swappiness = 1
net.core.wmem_default = 256960
net.core.rmem_default = 256960
net.ipv4.tcp_wmem = 8760  256960  4088000
net.ipv4.tcp_rmem = 8760  256960  4088000
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.core.netdev_max_backlog = 2000
vm.max_map_count=262144
[root@yinzhengjie ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.调优吞吐量**

 　　若要调优吞吐量（TPS），producer，broker和consumer都需要进行调整，以便让他们在相同的时间内传输更多的数据。众所周知，Kafka基本的并行单元就是分区。producer在设计时就被要求能够同时向多个分区发送消息，这些消息也要能够被写入到多个broker中供多个consumer同时消费。因此通常来说，分区数越多TPS越高。那么这是否意味着我们每次创建topic时都要创建大量的分区呢？答案显然是否定的。这依然是一个权衡（trade-off）的问题：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
一.过多的分区可能有哪些弊端呢？
1>.server/clients端将占用更多的内存。
    producer默认使用缓冲区为每个分区缓存消息，一旦满足条件producer便会批量发出缓存的消息。看上去这是一个提升性能的设计，不过由于该参数时分区级别的，因此如果分区很多，这部分缓存的内存占用也会变大；而在broker端，每个broker内部都维护了很多分区的元数据，比如controller，副本管理器，分区管理器等等。显然，分区数越多，缓存的成本越大。

2>.分区属越多，系统文件句柄书也越多
    每个分区在底层文件系统都有专属目录。该目录下厨了3个索引文件之外还会保存日志段文件。通常生产环境下的日志短文件可能有多个，因此保守估计一个分区就可能要占用十几个甚至几十个文件句柄。当kafka一旦打开文件并不会显式关闭该文件，故文件句柄书时不会释放的。那么随着分区数的增加，系统文件句柄书也会相应地增长。


3>.分区数过多会导致controller处理周期过长
    每个分区通常都有若干个副本而副本保存在不同的broker上。当leader副本挂掉了，controller回自动监测到，然后在zookeeper的帮助下选择新的leader。虽然在大部分情况下leader选举只有很短的延时，但若分区数很多，当broker挂掉后，需要进行leader选举的分区数就会很多。当前controller时单线程处理事件的，所以controller只能一个个地处理leader的变更请求，可能会拉长整体系统恢复的时间。



二.设置合理的分区数：
    基于以上3点，分区数的选择绝不是多多益善的。用户需要结合自身的实际环境基于吞吐量等指标进行一系列测试来确定需要选择多少分区数。有的用户总是抱怨自身环境无法达到官网给出的性能结果。其实官网的基准测试对用户的实际意义不大，因为不同的硬件，软件，网络，负载情况必然会带来不同的测试结果。比如用户使用1KB大小的啊消息进行测试，最后发现吞吐量才1MB/是，而官网说的每秒能达到10MB/s。这是因为官网使用的是100字节的消息体进行测试的，故根本没有可比性。
    虽然没法给出统一的分区数，但用户基本上可以遵循下面的步骤来尝试确定分区数：
        1>.创建单分区的topic，然后在实际生产机器上分别测试producer和consumer的TPS，分别为T(p) 和T(c)。
        2>.假设目标TPS是T(t),那么分区数大致可以确定为T(t)/max(T(p),T(c))。
    Kafka提供了专门的脚本kafka-producer-perf-test.sh和Kafka-consumer-perf-test.sh用于计算T(p) 和T(c)。这两个脚本的具体使用方啊可参考我笔记：https://www.cnblogs.com/yinzhengjie/p/9953212.html。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　好了，关于吞吐量的调优我就不废话了，直接总结一下调优TPS的一些参数清单和要点：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
一.broker端
1>.适当增加num.replica.fetchers（该值控制了broker端follower副本从leader副本处获取消息的最大线程数，默认值1表示follower副本只使用一个线程去实时拉取leader处的最新消息。对于设置了acks=all的producer而言，主要的延时可能都耽误在follower与leader同步的过程，故增加该值通常能够缩短同步的时间间隔，从而间接地提升producer端的TPS）,但不要超过CPU核数

2>.调优GC避免经常性的Full GC


二.producer端
1>.适当增加batch.size，比如100～512KB

2>.适当增加limger.ms,比如10～100ms

3>.设置compression.type=lz4（当前Kafka支持GZIP，Snappy和LZ4，但由于目前一些固有配置等原因，Kafka+LZ4组合的性能是最好的，因此推荐在那些CPU资源充足的环境中启用producer端压缩。）

4>.ack=0或1

5>.retries=0

6>.若干线程共享producer或分区数很多，增加buffer.memory


三.consumer端
1>.采用多consumer实例


2>.增加fetch.min.bytes（该参数控制了leader副本每次返回consumer的最小数据字节数。通过增加该参数值，默认值是1，Kafka会为每个FETCH请求的repsonse填入更多的数据，从而减少了网络开销并提升了TPS。）,比如10000
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**四.调优延时**

　　针对不同的组件，延时（latency）的定义可能不同。对于producer而言，延时主要是消息发送的延时，即producer发送PRODUCE请求到broker端返回请求response的时间间隔；对consumer而言，延时衡量了consumer发送FETCH请求到broker端返回请求resonse的时间间隔。还有只用延时定义表示的是集群的端到端延时（end-to-end latency），即producer端发送消息到consumer端“看到”这条消息的时间间隔。不论是那种延时，他们调优的思想大致是相同的。

　　下面总结一下调优延时的一些参数清单：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
一.broker端

1>.适度增加num.replica.fetchers(该值控制了broker端follower副本从leader副本处获取消息的最大线程数，默认值1表示follower副本只使用一个线程去实时拉取leader处的最新消息。对于设置了acks=all的producer而言，主要的延时可能都耽误在follower与leader同步的过程，故增加该值通常能够缩短同步的时间间隔，从而间接地提升producer端的TPS）

2>.避免创建过多topic分区


二.producer端
1>.设置linger.ms=0

2>.设置compression.type=none


3>.设置acks=1或者0



三.consumer端

1>.设置fetch.min.bytes=1(该参数控制了leader副本每次返回consumer的最小数据字节数,默认值是1）
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**五.调优持久性**

 　　顾名思义，持久性（durability）定义了Kafka集群中消息不容易丢失的程度。持久性越高表面kafka越不会丢失消息。持久性通常由冗余的手段就是备份机制（reolication）。它保证每条Kafka消息最终会保存在多台broker上。这样即使单个broker崩溃，数据依然是可用的。

　　下面是总结调优丑就行的参数清单和要点：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
一.broker端
1>.设置unclean.leader.election.enable=false(0.11.0.0之前版本需要显式设置。0.11.0.0版本开始unclean.leader.election.enable参数的默认值由原来的true改为false，可以关闭unclean leader election，也就是不在ISR(IN-Sync Replica)列表中的replica，不会被提升为新的
leader partition。kafka集群的持久化力大于可用性，如果ISR中没有其它的replica，会导致这个partition不能读写。)


2>.设置auto.create.topics.enable=false(否允许自动创建topic，默认值是true，若是false，就需要通过命令创建topic,推荐设置为false)


3>.设置replication.factor=3(指定分区的副本数,默认是1),min.insync.replicas（指定最少分区数据同步的个数，默认是1.）=replication.factor -1

4>.default.replication.factor=3（指定默认的分区数，默认是1）


5>.设置broker.rack属性分区数据到不同几家

6>.设置log.flush.interval.messages（指定了kafka写入多少条消息后执行一次消息“落盘”，是频率纬度上的参数）和log.flush.interval.ms（ 指定款Kafka多长时间执行一次消息“落盘”，是时间维度上的参数）为一个较小的值


二.producer端
1>.设置acks=all

2>.设置retries为一个较大的值，比如10～30。


3>.设置max.in.flight.requests.per.connection=1(客户端在阻塞之前在单个连接上发送的未确认请求的最大数量。注意，如果该设置设置大于1，并且发送失败，则存在由于重试（即，如果启用了重试）而导致消息重新排序的风险。默认值是5。简单的来说，设置为1可以规避消息乱序的风险。)

4>.设置enable.idempotence=true（保证同一条消息只被broker写入一次）启用幂等性


三.consumer端
1>.设置auto.commit.enable=false（关闭自动提交位移）
2>.消息消费成功后调用commitSync提交位移（设置auto.commit.enable=false既然是手动提交，用户需要调用commitSync方法来提位移，而不是使用commitAsync方法）。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**六.调优可用性**

　　所谓可用性（availability）反应的是Kafka集群应对崩溃的能力。无论broker，producer或consumer出现崩溃，kafka服务器依然保持可用状态而不会因为failure就中断服务。调优可用性就是要让Kafka 更快地从崩溃中恢复。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
一.broker端
1>.避免创建过多地分区

2>.设置unclean.leader.election.enable=true

3>.设置min.insync.replicas=1

4>.设置num.recovery.threads.per.data.dir=broker端参数log.dirs中设置的目录数。


二.producer端

1>.设置acks=1，若一定要设置为all，则遵循上面broker端的min.insync.replicas配置


三.consumer端
1>.设置session.timeout.ms为较低的值，比如5～10秒。

2>.设置max.poll.interval.ms（默认是300000ms，即5分钟。该参数赋予了consumer实例更多的时间来处理消息）为此消息平均处理时间稍大的值。（0.10.1.0及之后版本）

3>.设置max.poll.records和max.partition.fetch.bytes减少consumer处理消息的总时长，避免频繁rebalance。（0.10.1.0之前版本）
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186