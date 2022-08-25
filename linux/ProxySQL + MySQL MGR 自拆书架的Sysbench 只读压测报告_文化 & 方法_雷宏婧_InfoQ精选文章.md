在大量并行读请求、读多写少的业务场景下，本文利用 Sysbench 性能测试工具，研究基于【负载均衡+ProxySQL Cluster MGR 的读写分离架构】能够有效地利用横向扩展 MySQL 实例的读物能力，并最终提高应用系统 QPS。

## 前言

在大量并行读请求、读多写少的业务场景下，本文利用 Sysbench 性能测试工具，研究基于【负载均衡+ProxySQL Cluster MGR 的读写分离架构】能够有效地利用横向扩展 MySQL 实例的读物能力，并最终提高应用系统 QPS。

1.  MySQL Group Replication（M GR）于 2016 年 12 月被推出，了高可用、高扩展、高可靠的 MySQL 提供服务。但仅解决数据同步问题和其内部的自动故障转移。应用系统可能需要修改数据库连接地址，才能保证服务的可用性。为解决上述问题，可在 MRG 上层增加代理层，例如 ProxySQL。
    
2.  ProxySQL 于 2015 年被推出，是一个开源、高可用性、协议探索的 MySQL 代理。
    

1）可通过节点的只读值，自动调整它们是属于读组还是写组；

2）可定制的基于用户、基于 schema、基于语句的规则对 SQL 语句进行访问，实现读写分离；

3) 支持搭建 ProxySQL Cluster 来达到高可用，节点之间的配置可自动同步。

3.  负载均衡是将流量分发至多台节点设备上的服务。
    

1) 可通过删除单点故障，提升应用系统的可用性；

2) 可降低大量的综合访问，提高应用系统的处理能力。

4.  Sysbench 是一个开源的、线程探索的测试、跨平台的多性能工具。
    

## 一、压测目的

基于 Sysbench 的 oltpreadonly 压测模式，对比【负载均衡+ProxySQL Cluster+MGR 的读写分离】和【应用直连 MySQL Master】这两种架构的只读性能：

1.  建立读写读写性能数据；
    
2.  验证读写架构在大量并发读请求场景下的宣讲；
    
3.  分析各模块和参数对拆解资产业绩的影响。
    

## 二、压测结论

2.1. 读写读写读写性能数据

在 Sysbench oltpreadonly 压测模式下，【4 层负载均衡+ProxySQL Cluster+MGR 读写分离】架构的 QPS 与并发线程数关系如下表所示。

![](https://static001.geekbang.org/infoq/62/62daa7eec675628cb178e9062dac1ae0.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

### 2.2. 只读场景下编写的文档

首先简单浏览下实验对比图和结果，

![](https://static001.geekbang.org/infoq/9d/9db6ee77057473dbeda57f87274d46f7.webp)

![](https://static001.geekbang.org/infoq/92/9204ea3110bead006cec1080cb45df25.webp)

![](https://static001.geekbang.org/infoq/f3/f383fcc59b9280aacf6d47d24b9c9b88.webp)

ps“/”表示：由于 Sysbench 机器 CPU 完成，完成测试，无实验结果实验结果证明：

1.  在不引入负载均衡、ProxySQL Cluster 中间件的理想情况下，【连 MGR 2 个只读实例】QPS 应用能达到 100w，为【应用连直 MySQL Master】的只读 QPS 等 37w 的 2.7 倍。该结果验证了 MGR 架构在大量并发读请求场景下的有效性。
    

2.  但实际上，如要保证应用系统高可用，则需要增加负载均衡、ProxySQL Cluster 等中间件，而这些中间件或多或少会带来性能损失。 MGR 读写出版】自己的只读 QPS 阅读为 89w，大约【应用直连 MySQL Master】的只读 QPS 报告 37w 的 2.4 倍，该结果验证了【4 层加载均衡+ProxySQL Cluster+MGR 读】写原稿】架构在大量并发读请求场景下的文章。
    

### 2.3. 各模块和参数对剥离架构绩效的影响

1.  【4 层负载均衡+ProxySQL Cluster+MGR 读写分离】框架的 QPS 滨海周边【直连 MGR 2 个只读实例】QPS 周边 100w 的 89%。其中，ProxySQL Cluster 带来约 11%的性能但 ProxySQL 的 CPU 占用率最高只有 57%，还需要 57% 的时间才能进一步有效利用 ProxySQL。
    

2.  根据 https://github.com/sysown/ProxySQL/issues/1724，参考 CPU 核数增加 ProxySQL 的 mysql-threads 变量值，即增加 ProxySQL 用于处理 MySQL 流量的后台线程数，能有效提升 QPS（如将）线程数 4 增加至 16，QPS 从提升了 3.3 倍），但目前尚未压测出 ProxySQL 的 CPU 持续提升到 100% 的场景。
    

3.  横向扩展 ProxySQL 实例数，能有效提升 QPS（实例数增加 1 至 2，QPS 提升 1 倍）。
    

4.  将 7 层负载均衡实际 4 层，由在应用层进行流量分配改成在传输层，降低网络性能，在实验中提升了 1 倍 QPS。
    

5.  可知 https://ProxySQL.com/blog/benchmarking-ProxySQL-144/，本身 ProxySQL 的 mysql-max\_stmts\_per\_connection 变量值（20 增加至 100），让单独可以处理更多的准备语句，但实验中影响 QPS。
    

## 三、压测详情

### 3.1. 压测环境

![](https://static001.geekbang.org/infoq/44/440bceeff0171c0c8b37e6ab2f4aff02.webp)

另外，还安装了 nodeexporter、mysqlexporter、proxysql\_exporter 来监控 OS、MySQL 和 ProxySQL，方便定位问题。

### 3.2. 压测指标

• 每秒执行请求数 QPS（Queries Per Second），数据库包含执行的 SQL 数 INSERT、SELECT、UPDATE、DELE、COMMIT 等。

• 95% Latency (ms) 95% 请求的延迟时间完全查询请求中的 95% 在发出到接收结果的平均时间。延迟越小延迟。

### 3.3. 实验设置

为减少干净，每轮实验重复 3 次。每次任务执行完之后，等待 300 秒，让系统及时处理未完成任务，进入下管道压测。压测后除了利用 Sysbench 自带的清理才清理数据，还额外把垃圾桶清理干净，清洁磁盘空间影响一次压测。模块设置下可以看到变。

#### 3.3.1 MySQL 设置

• MGR：单模式。共 3 个节点，其中 1 个只写节点，2 个只读节点。max\_connection 设为 3000。

• Master-Master：主主同步，其中只有 1 个 Master 提供读写服务。max\_connection 设为 3000。

#### 3.3.2 ProxySQL 设置

• mysql\_user 的 transaction\_persistent 表字段：设置为 1，表示在某节点内启动的事务将保留在该节点内，而与其他评论规则。用于避免以下问题的操作操作：事务有混合的读和写组成操作，事务未提交前，如果事务中的读操作和写操作访问到不同节点，读取到的结果是脏数据。

```
 insert into mysql_users(username,password,default_hostgroup,transaction_persistent) values('MGR','MGR',1,1);
```

复制代码

• mysqls 表的 max\_connections：允许连接到该实例的最大连接数，不能超过 MySQL 设置的 max\_connections，因此设为 3000。

• mysql\_group\_replication\_hostgroups 表：配置 MGR writerhostgroup、readerhostgroup 的 hostgroupidProxy。SQL 会通过视图来监控 MGR 节点等是否正常，是否开启了读取结果、挤压事务数调整了单个 MGR 节点所属的 hostgroupid，具体对应可在 runtime\_mysql\_servers 中查看。 insert into mysql\_group\_replication\_hostgroups(writer\_hostgroup,backup\_writer\_hostgroup,reader\_hostgroup,offline\_hostgroup,active,max\_writers,writer\_is\_also\_reader,max\_transactions\_behind,comment) values(1,2,3,4,1,0,'-test,1r ;

\-- 可以有写组有 1 个节点，读组有 2 个节点，均在正常工作

![](https://static001.geekbang.org/infoq/4a/4a9895d0f8e4ca68c7cfbe853b0dfc4c.webp)

•查询规则：根据 SQL 的正则表达式匹配，读请求评论至读组，写请求评论至写组。

_**——将写请求评论到写组**_

INSERT INTO mysql\_query\_rules (rule\_id,active,username,match\_digest,destination\_hostgroup,apply) VALUES (200,1,'mgr','^SELECT.\*FOR UPDATE ,1,1);

_**——将读请求评论到读组**_

INSERT INTO mysql\_query\_rules (rule\_id,active,username,match\_digest,destination\_hostgroup,apply) VALUES (201,1,'mgr','^SELECT',3,1);

• 变量变量 mysql-threads：ProxySQL 用于处理 MySQL 流量的后台线程数。默认值为 4，实验中发现，增加至 16 值可致命实验提升 QPS，因此除了变量该变量的参数调优，其他实验中该变量值 16。    

set mysql-threads=16;显示变量，如'mysql-threads';

![](https://static001.geekbang.org/infoq/51/511c5478693724fd73216af3437d869e.webp)

### 3.3.3 Sysbench 设置

• 实验基于 Sysbench 的 oltpreadonly 只读模式。该模式下，一个事务包含 14 个读 SQL（10 条主键点查询、4 条范围查询）。

• oltpreadonly 模式的压测命令

**准备数据：**

sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=dbtest --tables=1 -- table-size=10000000 --report-interval=1 --threads=XXX --rand-type=uniform --time=120 --auto-inc=on /usr/local/share/sysbench/oltp\_read\_only.lua prepare

**运行工作量：**

sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=dbtest --tables=1 --表大小=10000000 --report-interval=1 --threads=XXX --rand-type=uniform --time=120 --auto-inc=on --skip\_trx=on /usr/local/share/sysbench/ oltp\_read\_only.lua 运行

**清理数据：**

sysbench --db-driver=mysql --mysql-host=XXX --mysql-port=XXX --mysql-user=XXX --mysql-password=XXX --mysql-db=dbtest --tables=1 -- table-size=10000000 --report-interval=1 --threads=XXX --rand-type=uniform --time=120 --auto-inc=on /usr/local/share/sysbench/oltp\_read\_only.lua 清理

**普通变量：**

•    时间：压测总持续时间（秒），后任务未完也会被终止。

•    线程：一起压测的线程数。取值范围\[16,32,64,128,256,512,1024,1500,2048,2500,3000,3500,4096\]。

**重点变量：**

• skip\_trx\[=on|off\]：默认为 off，即启动显式事务；启动开启时，不显式事务，以 AUTOCOMMIT 模式执行所有查询。

• 压测时设置：sysbench --skiptrx=on；ProxySQL 的 mysqluser 表的 transaction\_persistent=1。原因如下：

• Proxy 的 mysqluser 表事务的持久字段设为 1 时，在某节点内事务将保留在该节点启动，而与其他转发规则内的。用于避免以下问题：一个事务有混合的读和写操作组成，事务未提交前，如果事务中的读操作和写操作访问到不同节点，读取到的结果是脏数据。因此，如果不开启 skip\_trx，sysbench 全部请求都被 ProxySQL 评论到写组，这样便便测不了分离的性能。

•  sysbench 默认使用准备好的语句，因为本实验需要测试使用准备好的语句的情况，因此在此不作关闭该功能的参数说明。

• 设置变量--mysql-host=\[host1,host2,...,hostN\]，对多个 MySQL 同时发起发起请求。可用于同时压测多个 MySQL 实例时的 QPS。

[https://github.com/akopytov/sysbench/issues/19](https://github.com/akopytov/sysbench/issues/19)  

#### 3.3.4 实验场景设置

共同设计了 6 个实验场景（架构图查看实验结果分析），实验目的如下：

![](https://static001.geekbang.org/infoq/37/378e3698c44765cf40630bbf378120b7.webp)

### 3.4. 实验结果分析

_**实验一：\[MGR\] vs \[Master-Master\]**_

![](https://static001.geekbang.org/infoq/d2/d2578af70f7b28b5caca521db0090548.webp)

**实验目的：**

获取应用（sysbench）直连 MGR 的 2 个只读实例数引发的 QPS 服务，确认该服务和应用直连 mysql Master-Master 中 1 台的 QPS 差异。

**实验结果：**

![](https://static001.geekbang.org/infoq/c6/c6237e70db25a5088d3f1cf41b70b61a.webp)

![](https://static001.geekbang.org/infoq/b7/b764394403d05fc669d0a4d97a561de0.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

• MGR QPS 物业约 100w，约 Master-Master 的 2.67 倍。

_**实验二：\[单 ProxySQL+MGR\] vs \[MGR\]**_

![](https://static001.geekbang.org/infoq/04/047b8f515b665518edd6673b28ebb17e.webp)

**实验目的：**

在典型应用和 ProxySQL 网络延迟的情况下，确认增加 ProxySQL 中间件会带来的性能损失

**实验结果：**

![](https://static001.geekbang.org/infoq/18/18475c7ea9a99af559288b8b0c76ed74.webp)

![](https://static001.geekbang.org/infoq/5a/5aaf89da635257ab94bb673f8aac2dc1.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

• 单个 ProxySQL+MGR QPS 最大 48w，大约 MGR 的 48%。

• 单个 ProxySQL+MGR QPS 最大时，ProxySQL 实例 CPU 占用率最高为 57%。

**实验结论：**

在该实验中，ProxySQL Cluster 带来了约 48% 的损失，但此时 ProxySQL 的 CPU 占用率还不算高，性能探索可能进一步有效利用 ProxySQL。

_**实验三：【ProxySQL Cluster+MGR】对比【单个 ProxySQL+MGR】**_

![](https://static001.geekbang.org/infoq/b3/b30f9212a9fceb71e0f4c49d819e07ce.webp)

**实验目的：**

确认横向扩展 ProxySQL 实例化可能进一步提升 QPS

**实验结果：**

![](https://static001.geekbang.org/infoq/9a/9aeeff47f8b72594ab56efa91a4bcba7.webp)

![](https://static001.geekbang.org/infoq/8f/8fbb1fc629f935101ff25c69a5b923a2.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

• ProxySQL+MGR QPS 集群约 89w，约为单个 ProxySQL+MGR 的 1.85 倍。

• ProxySQL Cluster+MGR QPS 最大时，ProxySQL 实例 CPU 占用率最高为 56%。

**实验结论：**

横向拓展代理 SQL 实例集合可以进一步提升 QPS 至 89w，相对接近 MGR 的小区 100w。

_**实验 4：【7 层负载均衡+ProxySQL Cluster+MGR】对比【ProxySQL Cluster+MGR】**_

![](https://static001.geekbang.org/infoq/57/578f64ba0294dfc727f01d5851012149.webp)

**实验目的：**

增加读写架构中的需求均衡服务，并确认其带来的性能损失

**实验结果：**

![](https://static001.geekbang.org/infoq/fb/fbcc34a35b3d0b654aaecddd139e8860.webp)

![](https://static001.geekbang.org/infoq/90/90577f709595f186fd985c760e7fa0e3.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

• 增加负载均衡后，QPS 时间为 42w，约 MGR 服务的 50%，1 主-主.135 倍。

**实验结论：**

增加负载均衡导致性能损失近 50%，可能是因为网络、配置问题，需要进一步排查。

_**实验 5：【4 层负载均衡+ProxySQL Cluster+MGR】对比【7 层负载均衡+ProxySQL Cluster+MGR】**_

**实验目的：**

4 层负载均衡工作在 OSI 模型的传输层（基于 IP+端口），7 层工作在应用层（基于 URL）。

晚上，7 层负载均衡会带来更多的网络性能。因此喂饱为 4 层负载均衡，以减少性能损失。

**实验结果：**

![](https://static001.geekbang.org/infoq/7b/7bd41c1f729f54ee091abde1b3dabd12.webp)

![](https://static001.geekbang.org/infoq/a0/a0e15198f36e032306806b91aa24ae5a.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

**实验结论：**

将 7 层负载均衡立即 4 层负载均衡后，QPS 观看为 89w，几乎没有带来性能损失。

_**实验 6: ProxySQL 参数调优：分析 mysql-threads 参数值对性能的影响**_

**实验目的：**

根据 [https://github.com/sysown/ProxySQL/issues/1724](https://github.com/sysown/ProxySQL/issues/1724)，mysql-threads 变量是 ProxySQL 用于处理 MySQL 流量的后台线程数，根据机器 CPU 核数来调整该变量，可提升 ProxySQL 性能。因此实验分析该参数对性能的影响。

**结实验果：**

![](https://static001.geekbang.org/infoq/f3/f395dc6080f1d3b0c71bca56d3157aab.webp)

![](https://static001.geekbang.org/infoq/07/0740b8399bcf9f9ddfc9ccb631a9dcd4.webp)

ps/”表示由于 Sysbench 机器 CPU 完成，完成测试，无实验结果。

**实验结论：**

可知机器 CPU 核数来增加 ProxySQL 的 mysql-thread 变量值，可更进一步提升 QPS。

## 四、总结

1.  【4 层负载均衡+代理 SQL 集群+MGR 读写分离】适用于大量读请求场景，只读 QPS 能达到 89w，左右【应用直连 MySQL Master】的只读 QPS 37w 的 2.4 倍。
    
2.  参考机器的 CPU 核数增加 ProxySQL 的 mysql-threads 变量值，即增加了用于处理 MySQL 流量的后台线程数的 ProxySQL，能有效提升 QPS（如将线程数从 4 增加至 16，QPS 提高了 3.3 倍）。
    
3.  横向扩展 ProxySQL 实例数，能有效提升 QPS（实例数增加 1 至 2，QPS 提升 1 倍）。
    
4.  将 7 层负载均衡按 4 层，由在应用层进行流量分配改成在传输层，能降低网络性能并提高 QPS。
    
5.  次次实验中，ProxySQL 集群带来约 11% 的性能损失，几乎没有带来性能损失。但是 ProxySQL 的 CPU 占用率最高只有 57%，还需要汽车探索才能进一步有效利用 ProxySQL。
    

**参考文献：**

1.  https://dev.MySQL.com/doc/refman/5.7/en/group-replication.html
    
2.  https://ProxySQL.com/documentation/ProxySQL-Threads/
    
3.  https://ProxySQL.com/blog/ProxySQL-vs-maxscale-persistent-connection-response-time/
    
4.  https://www.percona.com/blog/2020/08/28/ProxySQL-overhead-explained-and-measured/
    
5.  https://github.com/sysown/ProxySQL/issues/1724
    
6.  https://www.percona.com/blog/2017/04/10/ProxySQL-rules-do-i-have-too-many/
    

**作者简介：**

雷宏姧，网易游戏技术部数据库系统工程师。参与海量高级数据库生产环境故障排查和优化，热衷于 MySQL 技术原理、灾难失败和高研究可用方案。