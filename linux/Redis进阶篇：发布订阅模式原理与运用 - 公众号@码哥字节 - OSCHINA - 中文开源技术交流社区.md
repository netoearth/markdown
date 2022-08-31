> “65 哥，如果你交了个漂亮小姐姐做女朋友，你会通过什么方式将这个消息广而告之给你的微信好友？“

> “那不得拍点女朋友的美照 + 亲密照弄一个九宫格图文消息在朋友圈发布大肆宣传，暴击单身狗。”

像这种 65 哥通过朋友圈发布消息，关注 65 哥的好友能收到通知的场景叫做「发布 / 订阅机制」。

今天不聊小姐姐，深入了解下 「Redis 发布 / 订阅机制」。的原理与实战运用。

Redis 通过 [`SUBSCRIBE`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Fcommands%2Fsubscribe)，[`UNSUBSCRIBE`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Fcommands%2Funsubscribe)和 [`PUBLISH`](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Fcommands%2Fpublish) 实现发布订阅消息传递模式，Redis 提供了两种模式实现，分别是「发布 / 订阅到频道」和「发布 \\ 订阅到模式」。

\[toc\]

## Redis 发布订阅简介

Redis 发布订阅（Pus/Sub）是一种消息通信模式：发送者通过 `PUBLISH` 发布消息，订阅者通过 `SUBSCRIBE` 订阅接收消息或通过 `UNSUBSCRIBE` 取消订阅。

主要包含三个部分组成：「发布者」、「订阅者」、「Channel」。

发布者和订阅者属于客户端，Channel 是 Redis 服务端，发布者将消息发布到频道，订阅这个频道的订阅者则收到消息。

如下图所示，三个「订阅者」订阅「ChannelA」频道：

![订阅](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/pub_sub.drawio.png)

这时候，小组长往「ChannelA」发布消息，这个消息的订阅者就会收到消息「关注码哥字节，提升技术」：

![发布/订阅](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220807161809.png)

## Pub/Sub 实战

废话不多说，知道基本概念以后，学习一个技术第一步先把它跑起来，接着才是探索原理，从而达到「知其然，知其所以然」的境界 。

一共有两种模式实现「发布 \\ 订阅」：

-   使用频道（Channel）的发布订阅；
-   使用模式（Pattern）的发布订阅。

**需要注意的是，发布订阅机制与 db 空间无关，比如在 db 10 发布， db0 的订阅者也会收到消息。**

### 通过频道（Channel）实现

三步走：

1.  订阅者订阅频道；
2.  发布者向「频道」发布消息；
3.  所有订阅「**频道**」的订阅者收到消息。

**订阅者订阅频道**

使用 `SUBSCRIBE channel [channel ...]` 订阅一个或者多个频道，O (n) 时间复杂度，n = 订阅的 Channel 数量。

```
SUBSCRIBE develop
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 消息类型
2) "develop" // 频道
3) (integer) 1 // 消息内容
```

执行该指令后，客户端进入订阅状态，订阅者只能使用 `subscribe`、`unsubscribe`、`psubscribe` 和 `punsubscribe` 这四个属于 "发布 / 订阅" 的指令。

客户端「肖菜鸡」订阅了 「develop」频道接受组长的消息，消息响应体分别表示：

-   消息类型：subscribe、message、unsubscribe
-   频道
-   消息内容：随着消息类型不同代表不同含义。

进入订阅后的客户端可以收到 3 种类型的消息回复：

1.  **subscribe**：订阅成功的反馈消息，第二个值是订阅成功的频道名称，第三个是当前客户端订阅的频道数量。
2.  **message**：客户端接收到消息，第二个值表示产生消息的频道名称，第三个值是消息的内容。
3.  **unsubscribe**：表示成功取消订阅某个频道。第二个值是对应的频道名称，第三个值是当前客户端订阅的频道数量，当此值为 0 时客户端会退出订阅状态，之后就可以执行其他非 "发布 / 订阅" 模式的命令了。

**发布者发布消息**

小组长使用 `PUBLISH channel message` 向指定 「develop」频道发布消息。

```
PUBLISH develop 'do job'
(integer) 1
```

需要注意的是，发布的**消息并不会持久化，消息发布之后还有新「开发」靓仔订阅的话，只能接收后续发布到该频道的消息。**

**好一个「不问过往，只争当下」。**

**订阅者接受消息**

关注了「develop」频道的订阅者将会收到「do job」消息。

```
// 订阅 develop 频道
SUBSCRIBE develop
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 订阅频道成功
2) "develop" // 频道
3) (integer) 1
// 当发布者发布消息，订阅者读取到的消息如下
1) "message" // 接受到消息
2) "develop" // 频道名称
3) "do job" // 消息内容
```

**退订频道**

订阅的反向操作，「65 哥」天天在朋友圈秀恩爱，受不了了，取消订阅他的朋友圈。

使用 [UNSUBSCRIBE](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fredis.readthedocs.org%2Fen%2Flatest%2Fpub_sub%2Funsubscribe.html%23unsubscribe) 命令可以退订指定的「模式」不会影响通过 \`subscribe 命令订阅的频道。

同样 `unsubscribe` 命令也不会影响通过 `psubscribe` 命令订阅的规则。

### 通过模式（Pattern）实现

接下来看另一种方式实现发布订阅，如下图表示当「匹配模式」与这个频道匹配的话，当消息向频道发布消息，该消息还会发布到与这个频道匹配的「模式」上，订阅这个模式的客户端也会收到消息。

`smile.girl.*` 模式表示「你微笑时好美」pattern，与这个模式匹配的两个频道是 `smile.girls.Tina`、`smile.girls.maggi` ，分别表示喜欢「微笑的 Tina」 和喜欢「微笑的 maggi」的粉丝。

如下图：

![模式匹配](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814112343.png)

现在 Tina 发布动态将消息发送到 `smile.girls.Tina` 频道的时候，除了订阅了 `smile.girls.Tina` 这个频道的粉丝收到消息以外，这 个消息还会发送给订阅 `smile.girl.*` 模式的粉丝（因为频道与模式匹配）。

这些粉丝比较贪心，所有「微笑时好美的 girls」都关注了，LSP~~，码哥可不是这样的人。

![模式匹配发布](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814114904.png)

使用匹配模式，用 `PUBLISH` 将消息发布到订阅 `smile.girls.Tina` 客户端之外，还会将该「频道」与「pub/sub pattern」中的模式进行对比，如果 Channel 与某个模式匹配的话，也将这个消息发布到订阅这个模式的客户端。

#### 订阅模式

订阅模式的指令是 `PSUBSCRIBE`，如下表示 LSP 订阅「smile.girl.\*」模式：

```
PSUBSCRIBE smile.girls.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe" // 消息类型
2) "smile.girls.*"// 模式
3) (integer) 1 //订阅数
```

对应的反向取消模式订阅的指令是 `PUNSUBSCRIBE smile.girl.*`。

**订阅 「smile.girls.Tina」频道**：

```
SUBSCRIBE smile.girls.Tina
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "smile.girls.Tina"
3) (integer) 1
```

**订阅「smile.girls.maggi」频道**：

```
SUBSCRIBE smile.girls.maggi
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "smile.girls.maggi"
3) (integer) 1
```

Tina 发布消息，\*\* 关注「smile.girls.Tina」\*\* 的粉丝和订阅了与该频道匹配的「smile.girls.\*」模式的粉丝收到消息。

关注 「smile.girls.\*」模式的粉丝收到消息

```
PSUBSCRIBE smile.girls.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "smile.girls.*"
3) (integer) 1
//进入订阅状态，接收到消息
1) "pmessage" 消息类型
2) "smile.girls.*"
3) "smile.girls.Tina"
4) "love u" // 消息内容
```

关注「smile.girls.Tina」的粉丝收到消息

```
127.0.0.1:6379> SUBSCRIBE smile.girls.Tina
Reading messages... (press Ctrl-C to quit)
// 订阅成功
1) "subscribe"
2) "smile.girls.Tina"
3) (integer) 1
// 接收消息
1) "message"
2) "smile.girls.Tina"
3) "love u"
```

需要注意的是，**如果一个客户端订阅了与模式匹配的模式和频道，那么客户端会收到多次消息。**

比如，65 哥 订阅了「smile.girls.Tina」频道和「smile.girls.\*」模式，那么当 Tina 发布动态到频道的时候，65 哥会收到两条票消息，一条消息类型是 `message`，一条类型是 `pmessage`。

### Redisson 与 SpringBoot 实战

官方文档：[https://github.com/redisson/redisson/wiki/6.-distributed-objects/#67-topic](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fredisson%2Fredisson%2Fwiki%2F6.-distributed-objects%2F%2367-topic)

**生产者代码**

```
/**
 * 发布消息到 Topic
 * @param message 消息
 * @return 接收消息的客户端数量
 */
public long sendMessage(String message) {
    RTopic topic = redissonClient.getTopic(CHANNEL);
    long publish = topic.publish(message);
    log.info("生产者发送消息成功，msg = {}", message);
    return publish;
}
```

**消费者代码**

```
public void onMessage() {
  // in other thread or JVM
  RTopic topic = redissonClient.getTopic(CHANNEL);
  topic.addListener(String.class, (channel, msg) -> {
    log.info("channel: {} 收到消息 {}.",  channel, msg);
  });
}
```

**需要注意的是，发布消息与监听消息要运行在不同的 JVM，如果使用同一个 `redissonClient` 发布的话，不会监听到自己的消息。**

## 原理分析

我们通过上文知道了发布订阅的概念，一共两种模式实现发布订阅。并且运用原生指令和 Redisson 进行实战。

接下来，我们要深入理解 Redis 如何实现发布订阅机制，做到知其然知其所以然。

### 频道 (Channel) 的发布 / 订阅如何实现的？

> 65 哥，如果是你会使用什么数据结构来实现基于频道来定位对应客户端？

码哥，我觉得可以字典来实现，字典的 key 对应被订阅的频道，而字典的值可以使用一个链表，链表里面保存着订阅这个频道的所有客户端。

#### 数据结构

聪明，Redis 使用 `redis.h` 中有一个 `redisServer` 结构体维护每个服务器进程表示服务器状态，`pubsub_channels` 属性是一个字典，用于保存订阅频道的信息。

```
struct redisServer {
  ...
  /* Pubsub */
   dict *pubsub_channels;
  ...
}
```

如下图所示，「码哥」、「靓仔」订阅了「redis-channel」，「宅男」「LSP」订阅了「枝～藤￥由 \* 香 - 里」：

![频道订阅发布原理](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814181642.png)

#### 发送消息到频道

生产者调用 `PUBLISH channel messsage` 发送消息，程序先根据 channel 从 `pubsub_channels` 定位到字典的 key 所在的桶，接着把消息发送给这个 key 对应的 value 链表的所有客户端。

#### 退订频道

`UNSUBSCRIBE` 命令可以退订指定的频道：丢与字典操作来说，根据 key 找到关注链表，遍历链表，删除这个客户端，这样消息就不会发送给这个客户端了。

### 模式 (Pattern) 的发布 / 订阅如何实现的？

接下来，我们继续看基于模式实现的发布订阅原理……

当使用 `PUBLISH` 发布消息到某个频道的时候，不仅订阅这个频道的所有客户端会收到消息，与这个模式匹配的客户端也会收到消息。

源码在 `server.h` 文件中的 `redisServer.pubsub_patterns` 属性定义。

```
struct redisServer {
  ...
  /* A dict of pubsub_patterns */
dict *pubsub_patterns;
  ...
}
```

也是 dict 字典类型， key 对应「pattern」模式，value 是一个 链表类型的结构： `list *clients` 里面包含匹配个模式的客户端列表。

当执行 `PSUBSCRIBE smile.girls.*` 命令的时候，会执行 `pubsubSubscribePattern` 方法。

在这里我分享下如何定位关键源码，发布订阅我们根据经验搜索 `pubsub` 便能检索到 `pubsub.c`：

![pubsub.c](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814191326.png)

码哥使用 CLion 调试的 Redis 源码，跟我们 Java 开发用的 IDEA 出自于一家，所以快捷键都是一样的，接着使用 `Command + F12` 弹出方法搜索，找到 `pubsubSubscribePattern` 订阅模式的方法。

方法参数别分表示关注该模式的客户端 client \*c，和客户端想要关注的 \*pattern，方法主要逻辑如下：

1.  `listSearchKey(c->pubsub_patterns,pattern)`：根据 pattern 从 redisServer.pubsub\_patterns 字典查找是否已经存在该模式的 key，存在则调用 `addReplyPubsubPatSubscribed` 通知客户端已经订阅过了，否则继续执行以下逻辑。
2.  `dictFind(server.pubsub_patterns,pattern)`：根据模式 `pattern` 从字典 `server.pubsub_patterns` 找到 dictEntry 哈希桶，为空就调用 `listCreate()` 创建客户端链表 `list *clients`，并放到字典中，key = pattern，value = list \*clients 链表。
3.  哈希桶不为空，那么把当前客户端 `client *c` 添加到 `list *clients` 链表尾节点。

![订阅模式源码](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814210038.png)

所以模式实现的发布订阅也是通过字典来保存模式与客户端的关系，如下图所示：

![基于模式实现的发布订阅原理](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/20220814211411.png)

当使用 `PUBLISH` 发布消息的时候，除了发布到订阅 `channel` 的客户端以外，还会将该 channel 与 `pubsub_patterns` 字典中查找匹配模式 key 对应的 value 中的客户端链表，并执行消息发送。

#### 退订模式

使用 `PUNSUBSCRIBE` 命令可以退订指定的模式， 这个命令执行的是订阅模式的反操作：根据模式从 `pubsub_patterns` 字典中找到客户端链表，遍历链表将当前客户端删除。

## 总结

Redis 发布订阅功能，主要通过如下命令实现：

-   `subscribe channel [channel ...]`：订阅一个或者多个频道；
-   `unsubscribe channel` 退订指定频道；
-   `publish channel message` 向指定频道发送消息；
-   `psubscribe pattern` 订阅指定模式；
-   `punsubscribe pattern` 退订指定模式。

Pub/Sub 与数据库无关，比如在 `DB0` 上发布， `DB1` 的订阅者也将接收到。

基于频道实现的发布订阅信息是由服务器进程的 `redisServer.pubsub_channels` 字典保存，key = 被订阅的频道，value 是订阅频道的所有客户端链表。

当消息发布到频道的时候，遍历字典获取所有客户端并把消息发送到频道的客户端。

基于模式实现的发布订阅的信息保存在字典 `pubsub_patterns` 中，key = pattern，value 是客户端链表。

当消息发布到频道的时候，除了订阅该频道的客户端收到消息以外，所有订阅了与频道匹配的模式的客户端也会收到消息。

### 使用场景

说了这么多，Redis 发布订阅能在什么场景发挥作用呢？

**哨兵间通信**

哨兵集群中，每个哨兵节点利用 Pub/Sub 发布订阅实现哨兵之间的相互发现彼此和找到 Slave，详情点击 ->[《哨兵集群原理那些事》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fm3j2WZdFas8fjLRsykGcBQ)。

哨兵与 Master 建立通信后，利用 master 提供发布 / 订阅机制在`__sentinel__:hello` 发布自己的信息，比如身高体重、是否单身、IP、端口……，同时订阅这个频道来获取其他哨兵的信息，就这样实现哨兵间通信。

**消息队列**

之前「码哥」跟大家分享过如何利用 [Redis List](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FPv7IXUQQnhGCGMM9n0VfkA) 与 [Stream 实现消息队列](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FFC7n8KVQVejRzhjwlq0PMQ)。

我们也可以利用 Redis 发布订阅实现**轻量级简单的 MQ 功能**，实现上下游解耦，需要注意点是 Redis 发布订阅的消息不会被持久化，所以新订阅的客户端将收不到历史消息。

也不支持 ACK 机制，所以当前业务不能容忍这些缺点，那就使用专业的消息队列，如果能容忍那就能享受 Redis 快带来的优势。

最后，可以在评论区叫我一声「靓仔」么？为了写这个文章，码哥看了好多微笑时好美的 girl 才写出来，原创不易。

**朋友们点赞、分享、收藏支持我吧。**

参考资料

1.Redis 设计与实现

2.[https://redisbook.readthedocs.io/en/latest/feature/pubsub.html](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredisbook.readthedocs.io%2Fen%2Flatest%2Ffeature%2Fpubsub.html)