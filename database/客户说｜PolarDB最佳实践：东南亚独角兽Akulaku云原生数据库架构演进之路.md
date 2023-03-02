![图片](https://mmbiz.qpic.cn/sz_mmbiz_gif/Hk1ceA7cQUD7GbcKYEvDRyV2mk9BjtCKpOEBDQWicOEiaakmzadeBw2POcVPXXsVXX27GpT2SNJIK75lkmwFTyRQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1)

> 作者：张君泉，Akulaku数据库团队高级DBA**1**

**1.引言**

云数据库实现了计算存储分离，支持计算与存储的独立扩展，高可用性和灵活性，同时提供按量计费，增强成本的可控性，因此应用已非常普遍。但各家云数据库也有会各自的特性与瓶颈，比如传统的类MySQL云数据库服务在面对几亿甚至几十亿数据量变更时，会显得十分吃力，影响业务迭代速度。我司在调研解决此问题时对阿里云原生数据库PolarDB有所研究与实践，通过本文做一分享，希望能够为行业提供一些参考。

**2.背景**

Akulaku是一家领先的金融科技公司，为东南亚市场提供数字金融解决方案。随着公司的快速发展，业务量的持续增加，部分采用传统的数据库服务已不能满足业务增长需求。这些业务拥有数百套的数据库，在日常工作中数据库的扩容，以及在高负载下频繁的数据变更 ，维护成本上升较快，因此我们决定投入资源调研方案以解决此问题。

针对目前传统云数据库面临的问题，主要包括以下几个方面:

1.核心数据库负载普遍较高，对于高负载能力不足，同时节点扩容不便，而且读写常常都配置在主库。

2.日常变更，超过一亿行大表变更变得越来越多，对数据库性能和业务影响较大。

3.主从延时，因为延迟导致从库接口查不到数据而报错。

4.磁盘容量上限，不足以支持业务数据扩展。

经过收集业务痛点和数据库架构综合选型，我们最终决定把该部分业务迁移到PolarDB上，原因在于PolarDB的计算节点和共享存储分离，便于快速扩展，底层使用物理日志，主从同步在毫秒级，以及支持100T的超大磁盘容量，让业务不需要担心磁盘空间问题，同时在测试中PolarDB的性能和容灾方面也有很出色的表现。

架构改动，如图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfarGfAH0rDLPcsY02zhxTJxHG3xS2pZ2Kw3JwRE05nedFyCGibiaas48Csw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**3.迁移**

下面介绍从传统云数据库至PolarDB的迁移环节。整个迁移主要由4个阶段组成：

-   原数据库信息收集，用于判断是否满足迁移需求
    
-   针对不同业务编写迁移方案
    
-   数据同步和迁移过程的保障
    
-   迁移后的成果验收和性能表现
    

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfar3qPm51OIMDujMnk3YrHLeibfiaoAUeaPib4WLw03ibsPPibOomWzg0TvGFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**3.1 业务梳理**

收集需要迁移的数据库实例，我们主要围绕日常变更频繁和性能优化较大的实例。对于迁移方案，需要结合实际情况调研，比如不仅需考虑主业务本身，还需考虑上下游业务，例如数据分析平台，数仓报表，Canal同步等。

**3.2 上下游异构数据**

依赖业务需要在主业务迁移前先迁移到PolarDB，避免主业务迁移后，出现依赖业务数据缺失的情况。特别是异构数据源的场景，这里以Canal迁移为例：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfarfO8HTWF0MhiboRSU71lUbwB6Cv6GC7wJ0gGGm9yvvT4P0mmNIDcmiaCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Canal需要提前切换到PolarDB上，这里需要注意两个问题。

1\. 切换PolarDB时，因为binlog的位点信息变更，需要提前修改好Canal的配置文件。

2\. 确认PolarDB上对应Canal的同步账号，权限和密码是否和原数据库上的一致。

**3.3 自动化运维平台**

由于业务日常变更量大，Akulaku自主研发数据库运维平台，集成实例迁移，SQL审核，系统巡检等功能。本次迁移主要依赖数据库自动化平台的实例迁移功能，同时能避免误操作造成的业务影响。切换完后，也可以根据平台的资源展示和监控功能，实时查看PolarDB的性能状态。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfarMJaW6icLcI3MUR2ZFkMBia3S2cAhlZCFTfWpZgkdQ9ulwEricf5BKWJEg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

数据库自动化运维平台的数据同步功能，主要是调用了DTS接口来做数据的实时同步，只需要在平台上填好配置即可，后面的账号迁移和域名变更都是通过平台一键操作。

要注意几个事项：

1\. 数据同步时需要确认源端是否有触发器，可能会导致源和目标数据不一致，需要删除被迁移到目标库中的触发器。

2\. 域名切换前确保依赖业务已经全部迁移到PolarDB，防止依赖业务数据丢失。

3\. 域名切换前确保业务无写入，避免业务双写。

**3.4 迁移结果验收**  

本章节对迁移后的情况做一介绍，同时探讨一下PolarDB相关的相关特性。

_**3.4.1 性能优势**_

随着业务量上涨，在电商大促的场景，能体验到迁移后带来的好处。比如我们有一个数据库的QPS有几十万，以前数据库做支撑明显感觉到吃力，而且有些业务读写都在主库，加大对主库的负载压力。

PolarDB 计算节点和存储分离，能快速扩充计算节点，能更好地应付突发情况。自带Proxy节点，可以支持读写分离和业务切割，负载均衡，能更好地应付高负载的环境。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfarLtXviaT75HSy2NCjstZ1PwD0NKFSicmbBLMJicbw7LxguTH9K5apiaGpLQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在大促高负载的场景下，我们收集了一些QPS和往年的数据库做了对比，看到PolarDB性能有明显的提升。**在相同配置下，数据库负载越高，PolarDB的优势越大，**优势不仅仅是性能上的优化，同时也体现在数据库的稳定性上。

**PolarDB的功能特点**

**1\. 读写分离**

因为业务没有做读写分离，常常把连接都打到主库。而PolarDB只需要提供集群地址， 在保证读写一致性的前提下，对一些读写的事务，自动分流到主从库上，而且基于负载的自动调度策略，按照活跃连接数自动调度，实现多个节点间的负载均衡。

**2\. 快速添加从节点**

在一些大促期间，某些从库负载瞬时飙升，导致业务查询过慢。这时PolarDB可以快速添加从节点。因为PolarDB是共享的磁盘，添加节点不需要拷贝数据。另外计算节点和存储分离，单独添加从节点时，不影响整个集群的运行。

**3\. 高速链路互联**

一般MySQL的服务针对单个服务器，数据从写入到落盘，都是要经过OS的缓存的，即OS内核和用户数据的交互。PolarDB采用远程直接数据存取（Remote Direct Memory Access，以下简称RDMA）高速网络互连的众多区块服务器（Chunk Server）一起向上层计算节点提供块设备服务。摆脱传统的io模式，让数据读写更上一层楼，QPS可突破50w。

_**3.4.2 日常大表变更**_

对于大表加列，特别是涉及到数据分析平台的业务，表的基数都特别大，亿级别的表不在少数。以往在MySQL做变更时，为了把业务影响降低到最小，一般都是使用的PT或者Ghost工具进行大表的变更。但是耗时很长，而且常常会因为网络或者负载原因中断。迁移到PolarDB后，对于那些大表加列的DDL变更，现在可以直接通过自动化平台操作，利用PolarDB的新特性直接添加。

PolarDB 5.7入了MySQL 8.0的新特性—Instant Add Column ，快速加列采用的是 instant 算法，使得添加列时不再需要 rebuild 整个表，只需要短暂的获取MDL锁并在表的 metadata 中记录新增列的基本信息即可。而对于加索引， PolarDB支持并行DDL和DDL物理复制优化功能。

**秒级加字段**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfar2OYJY21jm3n8UQTicZpVOthW0fQWlom2lA2gXAlnV1fMicUrxlA1lJxw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，一亿多的大表，加列变更时，耗时都是在1秒内完成， 只需变更表定义信息，无需修改已有数据。基于该特性，大大降低业务变更时间和风险，对于我们日常维护有巨大的帮助，提升数据库SLA。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfar8qr5YsdmCjS1k31eO4xOib9MwUUkorJ6IPdUmWBKsjt3Hw0AtJ1azYg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

由上图可知，在添加列时，PolarDB耗时一直稳定在1秒内，但是原MySQL数据库的耗时却随着数据量的增长而增长。特别是几亿行的表变更时，原数据库耗时需要十几个小时。PolarDB执行效率有明显的提升，对比起来有质的飞跃。  

针对PolarDB MySQL引擎5.7版本的集群，需要开启以下参数来使用秒级加字段功能：

innodb\_support\_instant\_add\_column

**并行创建索引**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfarmBuH0nSibW4X9x7mhadCnPib9gNLFn8PaDhCFLlklP28KLCQBjJ890lA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfarmvyPS0KIibdM2CvAia4L2iad1EcmZQWNxzROC2qN1icMCtgBxXzblUibmQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，包含4.4亿行数据、接近3TB大小的表，使用并行创建索引，只用了不到20分钟的时间，极大地缩短了大表创建索引的耗时。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfar9QHe9Dg2pyiaNLP6BPa1AqSDu96svM9d0A0aiaG10lFbYfsJiccESbmWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

由上图可见，与原MySQL对比，随着表的大小的提升，PolarDB创建索引的性能优势越来越明显。

_**3.4.3 主从延迟**_

对于Akulaku业务系统，有一些业务场景对于主从延时要求特别严格。在使用原MySQL数据库时，往往因为延迟而触发业务告警，也是我们比较头疼的一个问题。迁移到PolarDB后，主从同步的效率有明显的提升。

PolarDB采用物理日志进行同步，因为共享存储原因，主节点通过RDMA网络将数据实时更新到共享存储，其它计算节点通过高性能的RDMA网络实时读取redo日志去修改Buffer Pool中的Page，同步B+Tree及事务信息。不同于逻辑复制自上而下的复制方式，物理复制方式是自下而上的，能把主要延时控制在毫秒级别。

**物理同步**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/Hk1ceA7cQUBxPic6mY7ku9zbMfkXoibfaryk7iaylfOJxmzhictIl0cLVA5gIPnUwupYQPodRgWkrJHvlKMddCRORw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如架构图所示，Primary和Replica节点共享同一个PFS（PolarStore File System），复用数据文件和日志文件，RO节点直接读取PFS上的Redo Log进行解析，并将其修改应用到自己Buffer Pool中的Page上，当用户的请求到达Replica节点后，就可以访问到最新的数据了。同时Replica和Primary节点间也会保持RPC通信，用于同步Replica当前日志的Apply位点，以及ReadView等信息。  

_**3.4.4 磁盘容量**_

随着业务量增长，原MySQL数据库服务的磁盘需频繁进行扩容操作，对于一些区域磁盘剩余不足的情况，还会导致实例切换，严重影响业务。数据分析平台相关业务，存储的数据至少都是T级别，而PolarDB可以支持到100T的容量，较大程度上缓解了这个问题，由于其底层的架构的优化，在IO性能方面得到较大的提升。

PolarDB的存储是怎么样做到高容错，容量大，而且加载速度快的？我们查询了相关资料，了解到这是PolarFS起了作用，如感兴趣可参阅以下文档：

_https://www.vldb.org/pvldb/vol11/p1849-cao.pdf_

**4.结语**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

初步统计，本次迁移跨公司多个业务线，涉及系统有几十个，从开始调研到最后正式上线耗时数月。经过大半年的实践，稳定性、兼容性、性能等均符合预期，能够满足我们现阶段发展需求。期望阿里云以后可以推出更多好的产品，共同迎接未来的挑战。

  **/ End /**  

**推荐阅读**

  

[![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)](http://mp.weixin.qq.com/s?__biz=MzIxNTQ0MDQxNg==&mid=2247510264&idx=1&sn=86c8730b56f998f55ef53e173c4511d7&chksm=979aa5b7a0ed2ca153da7b6c52892e68f4427daa0c97d6a517f20263c6011626184e13af3dde&scene=21#wechat_redirect)

[![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)](http://mp.weixin.qq.com/s?__biz=MzIxNTQ0MDQxNg==&mid=2247512974&idx=1&sn=6efb16f3d5d8eb88caba68da7c7ff6a1&chksm=979abec1a0ed37d75c391c730d78bb3518b27839470f0c7286381d0ed28ad4a6d070d83c80dc&scene=21#wechat_redirect)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)点击**「**阅读原文」****查看 **PolarDB 5周年** 更多信息