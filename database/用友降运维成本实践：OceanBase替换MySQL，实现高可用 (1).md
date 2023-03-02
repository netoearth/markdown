**导语：**随着业务模型的不断变化使运维难度越来越大，用友IT内部采用任务调度中心XXL-JOB和配置管理中心Nacos来实现公司IT分布式任务调度和微服务开发。但XXL-JOB和Nacos集群数量的增多又使其支撑系统MySQL难以招架。

  
为了寻找一款既能提供高可用又能统一管理集群的数据库，降低运维压力，用友IT内部开始了数据库选型的探索与实践。

以下为用友DBA的自述。

![图片](https://mmbiz.qpic.cn/mmbiz_png/DmLSlAr1Lx1dBwSXX57SyjfXicibwF9XzlmWEMITtHqnPJILqrrE0qycicErf4wt5hBbESibXbN3RvBLQjnIpUsT5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**业务面临挑战**

随着公司的发展，人员数量及IT系统的规模不断增长，日常的运维管理工作需要更多的投入，因此不得不引入一些自动化工具来助力运维管理，考虑到这些需求，我们引入了调度平台和配置管理中心来实现公司IT运维的标准化和自动化工作。

目前，公司IT团队任务调度平台部分采用XXL-JOB，其核心设计目标是开发迅速、学习简单、轻量级、易扩展，正是基于这些因素。

同时，为了解决团队内部配置管理和服务注册的问题，我们又引入了Nacos作为注册中心和配置管理中心，通过Nacos，一方面可以做服务发现和服务注册，另一方面可以使得配置标准化、格式统一化，当配置信息发生变动时，修改实时生效，无需重启服务器就能够自动感知相应的变化，并将新的变化统一发送到相应程序上，快速响应变化。使用配置中心后只需要相关人员在配置中心动态去调整参数，就基本上可以实时或准实时去调整对应的业务。另外，通过审计功能还可以追溯问题。

但随着XXL-JOB和Nacos集群数量的增多，给我们带来了很多其他烦恼。这些集群自身的运行维护也需要投入人力，同时这些系统都需要数据库来支撑，而我们使用的是单体数据库MySQL，一方面会导致MySQL实例数量增多，另一方面，MySQL实例无法提供HA高可用。当实例出现故障的时候，整个调度平台或配置管理中心就无法使用。如果要继续为MySQL创建高可用集群，管理成本和硬件成本都会非常高。

因此，我们想寻找一款既能提供高可用，又能统一管理集群的数据库，降低运维压力，减少运维成本。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**技术选型与落地效果**

在考察了目前市场上的各类数据库后，我们选择了两款分布式数据库作为候选，两款数据库都原生提供了高可用能力，在节点出现问题的时候，可以自动切换，并且业务不需要做任何改造。

其中一款数据库是 OceanBase，由于其提供多租户能力，因此我们最终选择了 OceanBase 作为调度中心和配置管理中心的数据库。利用这种多租户能力，我们可以将多个平台的数据库放在一套 OceanBase 集群中。大幅减少运维多套数据库所带来的工作量，同时租户的资源可以灵活配置，在某些集群业务大量增长时，可快速实现租户的扩容。

除了多租户能力外，我们通过阅读官方文档、浏览 OceanBase 问答社区以及与原厂工程师交流，并上手测试后，发现 OceanBase 还提供了以下四个关键能力。

**1\. 高扩展**

在我们使用MySQL时，其性能会随着数据量的不断增大而降低，导致平台变卡顿。而 OceanBase 这类分布式数据库，提供线性的扩展能力，在数据量增大之后，可通过增加服务器达到性能的线性扩展。

**2\. 高可用**

OceanBase 分布式数据库原生具备高可用能力，通过Paxos协议实现分区级别的高可用，在少数节点出现故障之后，依然能够提供服务，不需要再为数据库补充其他组件来实现高可用及自动故障转移。

**3\. 高压缩率**

基于 OceanBase 的LSM-Tree数据组织形式，全量数据=基线数据+增量数据，因为基线数据是静态数据，只有在下次合并的时候才会有变化，所以实现了很高的压缩率。我们从MySQL迁移到 OceanBase 之后，磁盘占用率只用到原先的1/3左右。

**4\. 高兼容性**

目前 OceanBase 基本兼容MySQL语法，因为Nacos以及XXL-JOB都需要在数据库中执行SQL的初始化，除了 OceanBase 在建表时由于XXL-JOB及Nacos官方提供的SQL运行需要ENGINE=InnoDB语法，其他都没有问题，解决方法很简单，去掉ENGINE=InnoDB就可以了。

决定采用 OceanBase 后，我们投入测试环境运行，效果如下。

XXL-JOB任务调度中心：初始化SQL完成之后直接运行比较顺利，兼容很不错，没有发现任何异常。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Nacos配置管理中心：初始化SQL完成之后，能够顺利运行。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**总结**

经过上述探索及测试，说明了 OceanBase 与XXL-JOB和Nacos的兼容度、集成度很高，我们利用 OceanBase 的多租户能力，灵活部署XXL-JOB任务调度及Nacos配置管理中心，实现了多租户的管理能力，减少了大量MySQL实例，大大简化了数据库的运维工作。下图是我们集群现在的一个示意图。 

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

对于本次数据库替换的初步尝试，我们认为非常顺利。基于 OceanBase 提供的Daas能力，可以更好地为XXL-JOB和Nacos这类依赖MySQL的工具提供服务，简化我们数据库的管理，让我们能够有更多的时间不断优化其他工具。

期待和你线上见的OB君

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![](https://wx.qlogo.cn/finderhead/2pm5Nb2cMaOROSZzX41Kqw3g8ayHyNUQxaZ58e4F74LqpEtmrJscow/0)

**OceanBase数据库**

，

已结束直播，可观看回放

对话ACE 第8期：如何做好分布式数据库的运维管理

视频号

数字化时代，主流的应用架构正由集中式向分布式发展，更多企业选择和开展了分布式数据库的探索和应用，关于分布式数据库的运维工作也面临着新的需求。12月13日（周二），OceanBase 技术专家鲍磊、Oracle ACE皇甫晓飞 ，将和大家一起直播探讨“如何做好分布式数据库的运维管理？”

欢迎大家预约直播，咱们线上见！

**点击文末“阅读原文”进入 OceanBase 博客专区**