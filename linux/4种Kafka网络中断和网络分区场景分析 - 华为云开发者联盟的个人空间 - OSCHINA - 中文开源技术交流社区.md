> **摘要：**本文主要带来 4 种 Kafka 网络中断和网络分区场景分析。

本文分享自华为云社区《[Kafka 网络中断和网络分区场景分析](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%2F363995%3Futm_source%3Doschina%26utm_medium%3Dbbs-ex%26utm_campaign%3Dpaas%26utm_content%3Dcontent)》，作者： 中间件小哥。

以 Kafka 2.7.1 版本为例，依赖 zk 方式部署

3 个 broker 分布在 3 个 az，3 个 zk（和 broker 合部），单分区 3 副本

![](https://pic2.zhimg.com/80/v2-4c4a0c1b99d129ffb83b982a95b5340d_720w.jpg)

## 1\. 单个 broker 节点和 leader 节点网络中断

网络中断前：

![](https://pic2.zhimg.com/80/v2-5df52adc81c758dd7377e9040417b97d_720w.jpg)

broker-1 和 broker-0（leader）间的网络中断后，单边中断，zk 可用（zk-1 为 leader，zk-0 和 zk-2 为 follower，zk-0 会不可用，但 zk 集群可用，过程中可能会引起原本连在 zk-0 上的 broker 节点会先和 zk 断开，再重新连接其他 zk 节点，进而引起 controller 切换、leader 选举等，此次分析暂不考虑这种情况），leader、isr、controller 都不变

![](https://pic2.zhimg.com/80/v2-dfc7cbb4697b125b93f91ab21d93a83d_720w.jpg)

az2 内的客户端无法生产消费（metadata 指明 leader 为 broker-0，而 az2 连不上 broker-0），az1/3 内的客户端可以生产消费，若 acks=-1，retries=1，则生产消息会失败，error\_code=7（REQUEST\_TIMED\_OUT）（因为 broker-1 在 isr 中，但无法同步数据），且会发两次（因为 retries=1），broker-0 和 broker-2 中会各有两条重复的消息，而 broker-1 中没有；由于 broker-0 没有同步数据，因此会从 isr 中被剔除，controller 同步 metadata 和 leaderAndIsr，isr 更新为 \[2,0\]

![](https://pic3.zhimg.com/80/v2-01de343f2ba1dde478284e04827528f6_720w.jpg)

网络恢复后，数据同步，更新 isr

## 2\. 单个 broker 节点和 controller 节点网络中断

broker 和 controller 断连，不影响生产消费，也不会出现数据不一致的情况

![](https://pic3.zhimg.com/80/v2-f9614449f13a5007f0be73a5d0846fd2_720w.jpg)

而当发生 leader 和 isr 变化时，controller 无法将 leader 和 isr 的变化更新给 broker，导致元数据不一致

![](https://pic1.zhimg.com/80/v2-6969c912d48d09cacd08631c5b275668_720w.jpg)

broker-0 故障时，controller（broker-2）感知，并根据 replicas 选举新的 leader 为 broker-1，但因为和 broker-1 网络中断，无法同步给 broker-1，broker-1 缓存的 leader 依然是 broker-0，isr 为 \[1,2,0\]；当客户端进行生产消费时，如果从 broker-2 拿到 metadata，认为 leader 为 1，访问 broker-1 会返回 NOT\_LEADER\_OR\_FOLLOWER；如果从 broker-1 拿到 metadata，认为 leader 为 0，访问 broker-0 失败，都会导致生产消费失败

## 3\. 非 controller 节点所在 az 被隔离（分区）

![](https://pic1.zhimg.com/80/v2-9c75e4301ccb833bc61dfd3175eee540_720w.jpg)

zk-0 和 zk-1、zk-2 不通，少于半数，az1 内 zk 不可用，broker-0 无法访问 zk，不会发生 controller 选举，controller 还是在 broker-1

网络恢复后，broker-0 加入集群，并同步数据

### 3.1 三副本 partition（replicas:\[1,0,2\]），原 leader 在 broker-1（或 broker-2）

![](https://pic2.zhimg.com/80/v2-8926273e6d06b88f296fb9f002ef41c1_720w.jpg)

az1 内：

broker-0 无法访问 zk，感知不到节点变化，metadata 不更新（leader：1，isr：\[1,0,2\]），依然认为自己是 follower，leader 在 1；az1 内的客户端无法生产消费

az2/3 内：

zk 可用，感知到 broker-0 下线，metadata 更新，且不发生 leader 切换（isr：\[1,0,2\] -> \[1,2\]，leader：1）；az2 和 az3 内的客户端可正常生产消费

### 3.2 三副本 partition（replicas:\[0,1,2\]），原 leader 在 broker-0

![](https://pic1.zhimg.com/80/v2-2807e2f073a1ed8e34d14217eeab3aa0_720w.jpg)

az1 内：

zk-0 和 zk-1、zk-2 连接中断，少于一半，az1 内 zk 集群不可用，Broker-0 连不上 zk，无法感知节点变化，且无法更新 isr，metadata 不变，leader 和 isr 都不变；az1 内客户端可以继续向 broker-0 生产消费

az2/3 内：

zk-1 和 zk-2 连通，zk 可用，集群感知到 broker-0 下线，触发 leader 切换，broker-1 成为新的 leader（时间取决于 zookeeper.session.timeout.ms），并更新 isr；az2/3 内的客户端可以向 broker-1 生产消费

此时，该分区出现了双主现象，replica-0 和 replica-1 均为 leader，均可以进行生产消费

若两个隔离域内的客户端都生产了消息，就会出现数据不一致的情况

示例：（假设网络隔离前有两条消息，leaderEpoch=0）

网络隔离前：

![](https://pic2.zhimg.com/80/v2-b6919a917c7d522da79c6d6b55167221_720w.jpg)

az1 隔离后，分区双主，az1 内的客户端写入 3 条消息：c、d、e，az2/3 内的客户端写入 2 条消息：f、g：

![](https://pic2.zhimg.com/80/v2-9a874e71ab90acb559f3417b618bee61_720w.jpg)

这里 leaderEpoch 增加 2，是因为有两次增加 leaderEpoch 的操作：一次是 PartitionStateMachine 的 handleStateChanges to OnlinePartition 时的 leader 选举，一次是 ReplicationStateMachine 的 handleStateChanges to OfflineReplica 时的 removeReplicasFromIsr

网络恢复后：

![](https://pic2.zhimg.com/80/v2-962d687f340ddfcdee888a8bf895f225_720w.jpg)

由于 controller 在 broker-2，缓存和 zk 中的 leader 都是 broker-1，controller 会告知 broker-0 makerFollower，broker-0 随即 add fetcher，会先从 leader（broker-1）获取 leaderEpoch 对应的 endOffset（通过 OFFSET\_FOR\_LEADER\_EPOCH），根据返回的结果进行 truncate，然后开始 FETCH 消息，并根据消息中的 leaderEpoch 进行 assign，以此和 leader 保持一致

![](https://pic1.zhimg.com/80/v2-47c4233dde1410f894a061ce9c7a8df4_720w.jpg)

待数据同步后，加入 isr，并更新 isr 为 \[1,2,0\]。之后在触发 preferredLeaderElection 时，broker-0 再次成为 leader，并增加 leaderEpoch 为 3

在网络隔离时，若 az1 内的客户端 acks=-1，retries=3，会发现生产消息失败，而数据目录中有消息，且为生产消息数的 4 倍（每条消息重复 4 次）

![](https://pic3.zhimg.com/80/v2-2bae030170274d0ecd9a74f8d6d88176_720w.jpg)

有前面所述可知，网络恢复后，offset2-13 的消息会被覆盖，但因为这些消息在生产时，acks=-1，给客户端返回的是生产失败的，因此也不算消息丢失

因此，考虑此种情况，建议客户端 acks=-1

## 4\. Controller 节点所在 az 被隔离（分区）

### 4.1 Leader 节点未被隔离

![](https://pic2.zhimg.com/80/v2-d5089c2033bbac3226fb5a2c43ff72b5_720w.jpg)

网络中断后，az3 的 zk 不可用，broker-2（原 controller）从 zk 集群断开，broker-0 和 broker-1 重新竞选 controller

![](https://pic4.zhimg.com/80/v2-d7fd1172253b663e6e5b700d0a795d63_720w.jpg)

最终 broker-0 选举为 controller，而 broker-2 也认为自己是 controller，出现 controller 双主，同时因连不上 zk，metadata 无法更新，az3 内的客户端无法生产消费，az1/2 内的客户端可以正常生产消费

![](https://pic3.zhimg.com/80/v2-5123220828008b531af9bf65b13ad536_720w.jpg)

故障恢复后，broker-2 感知到 zk 连接状态发生变化，会先 resign，再尝试竞选 controller，发现 broker-0 已经是 controller 了，放弃竞选 controller，同时，broker-0 会感知到 broker-2 上线，会同步 LeaderAndIsr 和 metadata 到 broker-2，并在 broker-2 同步数据后加入 isr

![](https://pic1.zhimg.com/80/v2-d74f517693e38996270309c8bbe72ab8_720w.jpg)

### 4.2 Leader 节点和 controller 为同一节点，一起被隔离

隔离前，controller 和 leader 都在 broker-0：

![](https://pic2.zhimg.com/80/v2-2adb6d2e74f8159f3ebc16be5a241525_720w.jpg)

隔离后，az1 网络隔离，zk 不可用，broker-2 竞选为 controller，出现 controller 双主，同时 replica-2 成为 leader，分区也出现双主

![](https://pic4.zhimg.com/80/v2-847b868c6c9fde24a6ac43710a90e84f_720w.jpg)

此时的场景和 3.2 类似，此时生产消息，可能出现数据不一致

![](https://pic3.zhimg.com/80/v2-ed81ebc7e845fbfe249004947f446cb2_720w.jpg)

网络恢复后的情况，也和 3.2 类似，broker-2 为 controller 和 leader，broker-0 根据 leaderEpoch 进行 truncate，从 broker-2 同步数据

![](https://pic1.zhimg.com/80/v2-dc6fbb559adaf0a8aa0edf0af019ed74_720w.jpg)

加入 isr，然后通过 preferredLeaderElection 再次成为 leader，leaderEpoch 加 1

## 5\. 补充：故障场景引起数据不一致

### 5.1 数据同步瞬间故障

初始时，broker-0 为 leader，broker-1 为 follower，各有两条消息 a、b：

![](https://pic2.zhimg.com/80/v2-c7fc507c5d427de386aac7db733e3c8d_720w.jpg)

leader 写入一条消息 c，还没来得及同步到 follower，两个 broker 都故障了（如下电）：

![](https://pic4.zhimg.com/80/v2-e925febdc93780b6a0cb04a210517dcf_720w.jpg)

之后 broker-1 先启动，成为 leader（0 和 1 都在 isr 中，无论 unclean.leader.election.enable 是否为 true，都能升主），并递增 leaderEpoch：

![](https://pic2.zhimg.com/80/v2-ea2048e6e0918a2a3f1038f03a6dadf9_720w.jpg)

然后 broker-0 启动，此时为 follower，通过 OFFSET\_FOR\_LEADER\_EPOCH 从 broker-1 获取 leaderEpoch=0 的 endOffset

![](https://pic4.zhimg.com/80/v2-586d203f115ae6df4a6839d60f1b648f_720w.jpg)

broker-0 根据 leader epoch endOffset 进行 truncate：

![](https://pic3.zhimg.com/80/v2-3005281c07dd11f8bc24f33ea5b11612_720w.jpg)

之后正常生产消息和副本同步：

![](https://pic1.zhimg.com/80/v2-f9f6d684fb1a232bef04d1a606023538_720w.jpg)

该过程，如果 acks=-1，则生产消息 c 时，返回客户端的是生产失败，不算消息丢失；如果 acks=0 或 1，则消息 c 丢失

### 5.2 unclean.leader.election.enable=true 引起的数据丢失

还是这个例子，broker-0 为 leader，broker-1 为 follower，各有两条消息 a、b，此时 broker-1 宕机，isr=\[0\]

![](https://pic2.zhimg.com/80/v2-856aa8923b31f59cd370a78423907ee5_720w.jpg)

在 broker-1 故障期间，生产消息 c，因为 broker-1 已经不在 isr 中了，所以即使 acks=-1，也能生产成功

![](https://pic3.zhimg.com/80/v2-d26fd02620432b15a2cc7478d24943ee_720w.jpg)

然后 broker-0 也宕机，leader=-1，isr=\[0\]

![](https://pic1.zhimg.com/80/v2-6775013576a5cd575035181cabe86830_720w.jpg)

此时 broker-1 先拉起，若 unclean.leader.election.enable=true，那么即使 broker-1 不在 isr 中，因为 broker-1 是唯一活着的节点，因此 broker-1 会选举为 leader，并更新 leaderEpoch 为 2

![](https://pic3.zhimg.com/80/v2-1e7333a835154cef4be1029c5f4832be_720w.jpg)

这时，broker-0 再拉起，会先通过 OFFSET\_FOR\_LEADER\_EPOCH，从 broker-1 获取 epoch 信息，并进行数据截断

![](https://pic2.zhimg.com/80/v2-dafa399ad57a5f1d089cca0eab5dac51_720w.jpg)

再进行生产消息和副本同步

![](https://pic1.zhimg.com/80/v2-d26e166e13a1fad4f504c18847f4c7f4_720w.jpg)

消息 c 丢失

[**点击关注，第一时间了解华为云新鲜技术～**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%3Futm_source%3Doschina%26utm_medium%3Dbbs-ex%26utm_campaign%3Dpaas%26utm_content%3Dcontent)