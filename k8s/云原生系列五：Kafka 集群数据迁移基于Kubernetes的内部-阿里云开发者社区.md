## 1.概述

Kafka的使用场景非常广泛，一些实时流数据业务场景，均依赖Kafka来做数据分流。而在分布式应用场景中，数据迁移是一个比较常见的问题。关于Kafka集群数据如何迁移，今天叶秋学长将为大家详细介绍。

## 2.内容

本篇博客为大家介绍两种迁移场景，分别是同集群数据迁移、跨集群数据迁移。如下图所示：

  ![](https://img-blog.csdnimg.cn/0824fe8aa5bd4b7e95391b33b10e4502.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/a8558c18d4de4a65a3e6be98f24eafe1.gif "image.gif")编辑

## 2.1 同集群迁移

同集群之间数据迁移，比如在已有的集群中新增了一个Broker节点，此时需要将原来集群中已有的Topic的数据迁移部分到新的集群中，缓解集群压力。

将新的节点添加到Kafka集群很简单，只需为它们分配一个唯一的Broker ID，并在新服务器上启动Kafka。但是，这些新服务器节点不会自动分配任何数据分区，因此除非将分区移动到新增的节点，否则在创建新Topic之前新节点不会执行任何操作。因此，通常在将新服务器节点添加到Kafka集群时，需要将一些现有数据迁移到这些新的节点。

迁移数据的过程是手动启动的，执行过程是完全自动化的。在Kafka后台服务中，Kafka将添加新服务器作为其正在迁移的分区的Follower，并允许新增节点完全复制该分区中的现有数据。当新服务器节点完全复制此分区的内容并加入同步副本（ISR）时，其中一个现有副本将删除其分区的数据。

Kafka系统提供了一个分区重新分配工具（kafka-reassign-partitions.sh），该工具可用于在Broker之间迁移分区。理想情况下，将确保所有Broker的数据和分区均匀分配。分区重新分配工具无法自动分析Kafka群集中的数据分布并迁移分区以实现均匀的负载均衡。因此，管理员在操作的时候，必须弄清楚应该迁移哪些Topic或分区。

分区重新分配工具可以在3种互斥模式下运行：

-   \--generate：在此模式下，给定Topic列表和Broker列表，该工具会生成候选重新分配，以将指定Topic的所有分区迁移到新Broker中。此选项仅提供了一种方便的方法，可在给定Topic和目标Broker列表的情况下生成分区重新分配计划。
-   \--execute：在此模式下，该工具将根据用户提供的重新分配计划启动分区的重新分配。 （使用--reassignment-json-file选项）。由管理员手动制定自定义重新分配计划，也可以使用--generate选项提供。
-   \--verify：在此模式下，该工具将验证最后一次--execute期间列出的所有分区的重新分配状态。状态可以有成功、失败或正在进行等状态。

### 2.1.1 迁移过程实现

分区重新分配工具可用于将一些Topic从当前的Broker节点中迁移到新添加的Broker中。这在扩展现有集群时通常很有用，因为将整个Topic移动到新的Broker变得更容易，而不是一次移动一个分区。当执行此操作时，用户需要提供已有的Broker节点的Topic列表，以及到新节点的Broker列表（源Broker到新Broker的映射关系）。然后，该工具在新的Broker中均匀分配给指定Topic列表的所有分区。在迁移过程中，Topic的复制因子保持不变。

现有如下实例，将Topic为ke01，ke02的所有分区从Broker1中移动到新增的Broker2和Broker3中。由于该工具接受Topic的输入列表作为JSON文件，因此需要明确迁移的Topic并创建json文件，如下所示：

\> cat topic-to-move.json

{"topics": \[{"topic": "ke01"},

           {"topic": "ke02"}\],

"version":1

}

准备好JSON文件，然后使用分区重新分配工具生成候选分配，命令如下：

\> bin/kafka-reassign-partitions.sh --zookeeper dn1:2181 --topics-to-move-json-file topics-to-move.json --broker-list "1,2" --generate

执行命名之前，Topic（ke01、ke02）的分区如下图所示：

![](https://img-blog.csdnimg.cn/img_convert/370279b8c20fb47e7d569b00fad2d84f.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/7f44f5a8223e459987a4b6135c41642d.gif "image.gif")编辑

![](https://img-blog.csdnimg.cn/img_convert/26f933005bd3c89ad8f0d468d7b6ca01.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/a6eec7fe8a1c4382884f69b108ec1aad.gif "image.gif")编辑

执行完成命令之后，控制台出现如下信息：

![](https://img-blog.csdnimg.cn/img_convert/f632fdffd5bfaac94f81a09d18f45360.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/e4aac11997b84e508f742b7ce41b08ef.gif "image.gif")编辑

该工具生成一个候选分配，将所有分区从Topic ke01，ke02移动到Broker1和Broker2。需求注意的是，此时分区移动尚未开始，它只是告诉你当前的分配和建议。保存当前分配，以防你想要回滚它。新的赋值应保存在JSON文件（例如expand-cluster-reassignment.json）中，以使用--execute选项执行。JSON文件如下：

{"version":1,"partitions":\[{"topic":"ke02","partition":0,"replicas":\[2\]},{"topic":"ke02","partition":1,"replicas":\[1\]},{"topic":"ke02","partition":2,"replicas":\[2\]},{"topic":"ke01","partition":0,"replicas":\[2\]},{"topic":"ke01","partition":1,"replicas":\[1\]},{"topic":"ke01","partition":2,"replicas":\[2\]}\]}

执行命令如下所示：

\> ./kafka-reassign-partitions.sh --zookeeper dn1:2181 --reassignment-json-file expand-cluster-reassignment.json --execute

最后，--verify选项可与该工具一起使用，以检查分区重新分配的状态。需要注意的是，相同的expand-cluster-reassignment.json（与--execute选项一起使用）应与--verify选项一起使用，执行命令如下：

\> ./kafka-reassign-partitions.sh --zookeeper dn1:2181 --reassignment-json-file expand-cluster-reassignment.json --verify

执行结果如下图所示：

![](https://img-blog.csdnimg.cn/img_convert/1a7ec6d6510a971661c388055fb04bb2.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/500b9c348b334452a0661a12024247c5.gif "image.gif")编辑

同时，我们可以通过[Kafka Eagle](https://blog.csdn.net/m0_63722685?spm=1011.2124.3001.5343)工具来查看Topic的分区情况。

![](https://img-blog.csdnimg.cn/img_convert/69115a3a6e3b16e908a492ebf275f146.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/285235abaeb14bc6b93562be535c8822.gif "image.gif")编辑

![](https://img-blog.csdnimg.cn/img_convert/30978d276b06619d11d084156b3ccdc6.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/9906a487ecdb4dd0ade2a99a41827a58.gif "image.gif")编辑

## 2.2 跨集群迁移

这里跨集群迁移，我们指的是在Kafka多个集群之间复制数据“镜像”的过程，以避免与单个集群中的节点之间发生的复制混淆。 Kafka附带了一个用于在Kafka集群之间镜像数据的工具。该工具从源集群使用并生成到目标集群。这种镜像的一个常见用例是在另一个数据中心提供副本。

另外，你可以运行许多此类镜像进程以提高吞吐量和容错（如果一个进程终止，其他进程将占用额外负载）。将从源集群中的Topic读取数据，并将其写入目标集群中具有相同名称的主题。事实上，“镜像”数据只不过是一个Kafka将消费者和生产者联系在了一起。

源集群和目标集群是完全独立的实体，它们可以具有不同数量的分区，并且偏移量将不相同。出于这个原因，镜像集群并不是真正意图作为容错机制（因为消费者的位置会有所不同）;为此，建议使用正常的集群内复制。但是，镜像进程将保留并使用消息Key进行分区，因此可以按Key保留顺序。

下面是一个跨集群的单Topic实例，命令如下：

\> ./kafka-mirror-maker.sh --consumer.config consumer.properties --producer.config producer.properties --whitelist ke03

需要注意的是，consumer.properties文件配置源Kafka集群Broker地址，producer.properties文件配置目标Kafka集群地址。如果需要迁移多个Topic，可以使用 --whitelist 'A|B'，如果需要迁移所有的Topic，可以使用 --whitelist '\*'。

## 3.结果预览

执行跨集群迁移命令后，目标集群中使用Kafka Eagle中查看Topic Size大小看是否与源集群的Topic Size大小相等，或者使用SQL语句，验证是否有数据迁移过来，结果如下图所示：

![](https://img-blog.csdnimg.cn/img_convert/4d7cfbfee03c2f497e11b160230afe44.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/44f1d9cb4564496c99544b637ee8bc16.gif "image.gif")编辑

## 4.总结

跨集群迁移数据的本质是，Kafka启动了消费者读取源集群数据，并将消费后的数据写入到目标集群，在迁移的过程中，可以启动多个实例，提供迁出的吞吐量。

> ## 本期分享到此为止，关注博主不迷路，叶秋学长带你上高速