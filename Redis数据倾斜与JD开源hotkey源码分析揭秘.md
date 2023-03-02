## 1 前言

之前旁边的小伙伴问我热点数据相关问题，在给他粗略地讲解一波 redis 数据倾斜的案例之后，自己也顺道回顾了一些关于热点数据处理的方法论，同时也想起去年所学习 JD 开源项目 hotkey—— 专门用来解决热点数据问题的框架。在这里结合两者所关联到的知识点，通过几个小图和部分粗略的讲解，来让大家了解相关方法论以及 hotkey 的源码解析。

## 2 Redis 数据倾斜

### 2.1 定义与危害

先说说数据倾斜的定义，借用百度词条的解释:

对于集群系统，一般缓存是分布式的，即不同节点负责一定范围的缓存数据。我们把缓存数据分散度不够，导致大量的缓存数据集中到了一台或者几台服务节点上，称为数据倾斜。一般来说数据倾斜是由于负载均衡实施的效果不好引起的。

从上面的定义中可以得知，数据倾斜的原因一般是因为 LB 的效果不好，导致部分节点数据量非常集中。

**那这又会有什么危害呢？**

如果发生了数据倾斜，那么保存了大量数据，或者是保存了热点数据的实例的处理压力就会增大，速度变慢，甚至还可能会引起这个实例的内存资源耗尽，从而崩溃。这是我们在应用切片集群时要避免的。

### 2.2 数据倾斜的分类

#### 2.2.1 数据量倾斜 (写入倾斜)

1\. 图示

![](https://img1.jcloudcs.com/developer.jdcloud.com/d9db88d0-6c4c-4c5f-9777-5594f94bef7620220811163709.png)

如图，在某些情况下，实例上的数据分布不均衡，某个实例上的数据特别多。

2.bigkey 导致倾斜

某个实例上正好保存了 bigkey。bigkey 的 value 值很大（String 类型），或者是 bigkey 保存了大量集合元素（集合类型），会导致这个实例的数据量增加，内存资源消耗也相应增加。

**应对方法**

-   在业务层生成数据时，要尽量避免把过多的数据保存在同一个键值对中。
-   如果 bigkey 正好是集合类型，还有一个方法，就是把 bigkey 拆分成很多个小的集合类型数据，分散保存在不同的实例上。

3.Slot 分配不均导致倾斜

先简单的介绍一下 slot 的概念，slot 其实全名是 Hash Slot (哈希槽)，在 Redis Cluster 切片集群中一共有 16384 个 Slot，这些哈希槽类似于数据分区，每个键值对都会根据它的 key，被映射到一个哈希槽中。Redis Cluster 方案采用哈希槽来处理数据和实例之间的映射关系。

一张图来解释，数据、哈希槽、实例这三者的映射分布情况。

![](https://img1.jcloudcs.com/developer.jdcloud.com/2817c2e3-6229-4810-bf8a-62c3bda5a55420220811163745.png)

这里的 CRC16 (city)%16384 可以简单的理解为将 key1 根据 CRC16 算法取 hash 值然后对 slot 个数取模，得到的就是 slot 位置为 14484，他所对应的实例节点是第三个。

运维在构建切片集群时候，需要手动分配哈希槽，并且把 16384 个槽都分配完，否则 Redis 集群无法正常工作。由于是手动分配，则可能会导致部分实例所分配的 slot 过多，导致数据倾斜。

**应对方法**

使用 CLUSTER SLOTS 命令来查

看 slot 分配情况，使用 CLUSTER SETSLOT，CLUSTER GETKEYSINSLOT，MIGRATE 这三个命令来进行 slot 数据的迁移，具体内容不再这里细说，感兴趣的同学可以自行学习一下。

4.Hash Tag 导致倾斜

-   Hash Tag 定义：指当一个 key 包含 {} 的时候，就不对整个 key 做 hash，而仅对 {} 包括的字符串做 hash。
-   假设 hash 算法为 sha1。对 user:{user1}:ids 和 user:{user1}:tweets，其 hash 值都等同于 sha1 (user1)。
-   Hash Tag 优势：如果不同 key 的 Hash Tag 内容都是一样的，那么，这些 key 对应的数据会被映射到同一个 Slot 中，同时会被分配到同一个实例上。
-   Hash Tag 劣势：如果不合理使用，会导致大量的数据可能被集中到一个实例上发生数据倾斜，集群中的负载不均衡。

#### 2.2.2 数据访问倾斜 (读取倾斜 - 热 key 问题)

一般来说数据访问倾斜就是热 key 问题导致的，如何处理 redis 热 key 问题也是面试中常会问到的。所以了解相关概念及方法论也是不可或缺的一环。

1\. 图示

![](https://img1.jcloudcs.com/developer.jdcloud.com/f91df4ba-cc7d-4bef-945f-54baf97fe2ac20220811163828.png)

如图，虽然每个集群实例上的数据量相差不大，但是某个实例上的数据是热点数据，被访问得非常频繁。

但是为啥会有热点数据的产生呢？

2\. 产生热 key 的原因及危害

1) 用户消费的数据远大于生产的数据（热卖商品、热点新闻、热点评论、明星直播）。

在日常工作生活中一些突发的事件，例如：双十一期间某些热门商品的降价促销，当这其中的某一件商品被数万次点击浏览或者购买时，会形成一个较大的需求量，这种情况下就会造成热点问题。

同理，被大量刊发、浏览的热点新闻、热点评论、明星直播等，这些典型的读多写少的场景也会产生热点问题。

2) 请求分片集中，超过单 Server 的性能极限。

在服务端读数据进行访问时，往往会对数据进行分片切分，此过程中会在某一主机 Server 上对相应的 Key 进行访问，当访问超过 Server 极限时，就会导致热点 Key 问题的产生。

如果热点过于集中，热点 Key 的缓存过多，超过目前的缓存容量时，就会导致缓存分片服务被打垮现象的产生。当缓存服务崩溃后，此时再有请求产生，会缓存到后台 DB 上，由于 DB 本身性能较弱，在面临大请求时很容易发生请求穿透现象，会进一步导致雪崩现象，严重影响设备的性能。

3\. 常用的热 key 问题解决办法:

**解决方案一：备份热 key**

可以把热点数据复制多份，在每一个数据副本的 key 中增加一个随机后缀，让它和其它副本数据不会被映射到同一个 Slot 中。

这里相当于把一份数据复制到其他实例上，这样在访问的时候也增加随机前缀，将对一个实例的访问压力，均摊到其他实例上

**例如:** 我们在放入缓存时就将对应业务的缓存 key 拆分成多个不同的 key。如下图所示，我们首先在更新缓存的一侧，将 key 拆成 N 份，比如一个 key 名字叫做”good\_100”，那我们就可以把它拆成四份，“good\_100\_copy1”、“good\_100\_copy2”、“good\_100\_copy3”、“good\_100\_copy4”，每次更新和新增时都需要去改动这 N 个 key，这一步就是拆 key。

![](https://img1.jcloudcs.com/developer.jdcloud.com/8a9c5918-1bac-4a68-9dfc-74a3de44009a20220811163918.png)

对于 service 端来讲，我们就需要想办法尽量将自己访问的流量足够的均匀。

如何给自己即将访问的热 key 上加入后缀？几种办法，根据本机的 ip 或 mac 地址做 hash，之后的值与拆 key 的数量做取余，最终决定拼接成什么样的 key 后缀，从而打到哪台机器上；服务启动时的一个随机数对拆 key 的数量做取余。

伪代码如下:

```
const M = N * 2
//生成随机数
random = GenRandom(0, M)
//构造备份新key
bakHotKey = hotKey + “_” + random
data = redis.GET(bakHotKey)
if data == NULL {
  data = GetFromDB()
  redis.SET(bakHotKey, expireTime + GenRandom(0,5))
}
```

**解决方案二：本地缓存 + 动态计算自动发现热点缓存**

![](https://img1.jcloudcs.com/developer.jdcloud.com/73b5f97e-6c8a-4f1a-a9ab-c6cb41565ab820220811164001.png)

该方案通过主动发现热点并对其进行存储来解决热点 Key 的问题。首先 Client 也会访问 SLB，并且通过 SLB 将各种请求分发至 Proxy 中，Proxy 会按照基于路由的方式将请求转发至后端的 Redis 中。

在热点 key 的解决上是采用在服务端增加缓存的方式进行。具体来说就是在 Proxy 上增加本地缓存，本地缓存采用 LRU 算法来缓存热点数据，后端节点增加热点数据计算模块来返回热点数据。

Proxy 架构的主要有以下优点：

-   Proxy 本地缓存热点，读能力可水平扩展
-   DB 节点定时计算热点数据集合
-   DB 反馈 Proxy 热点数据
-   对客户端完全透明，不需做任何兼容

热点数据的发现与存储

![](https://img1.jcloudcs.com/developer.jdcloud.com/e693d524-b98e-443b-bda4-d986506fa2f420220811164459.png)

对于热点数据的发现，首先会在一个周期内对 Key 进行请求统计，在达到请求量级后会对热点 Key 进行热点定位，并将所有的热点 Key 放入一个小的 LRU 链表内，在通过 Proxy 请求进行访问时，若 Redis 发现待访点是一个热点，就会进入一个反馈阶段，同时对该数据进行标记。

可以使用一个 etcd 或者 zk 集群来存储反馈的热点数据，然后本地所有节点监听该热点数据，进而加载到本地 JVM 缓存中。

**热点数据的获取**

![](https://img1.jcloudcs.com/developer.jdcloud.com/cfb4b0f8-580d-445b-8da6-ab07ee0aeb5920220811164514.png)

在热点 Key 的处理上主要分为写入跟读取两种形式，在数据写入过程当 SLB 收到数据 K1 并将其通过某一个 Proxy 写入一个 Redis，完成数据的写入。

假若经过后端热点模块计算发现 K1 成为热点 key 后， Proxy 会将该热点进行缓存，当下次客户端再进行访问 K1 时，可以不经 Redis。

最后由于 proxy 是可以水平扩充的，因此可以任意增强热点数据的访问能力。

**最佳成熟方案: JD 开源 hotKey** 这是目前较为成熟的自动探测热 key、分布式一致性缓存解决方案。原理就是在 client 端做洞察，然后上报对应 hotkey，server 端检测到后，将对应 hotkey 下发到对应服务端做本地缓存，并且能保证本地缓存和远程缓存的一致性。

在这里咱们就不细谈了，这篇文章的第三部分：JD 开源 hotkey 源码解析里面会带领大家了解其整体原理。

## 3 JD 开源 hotkey— 自动探测热 key、分布式一致性缓存解决方案

### 3.1 解决痛点

从上面可知，热点 key 问题在并发量比较高的系统中 (特别是做秒杀活动) 出现的频率会比较高，对系统带来的危害也很大。

那么针对此，hotkey 诞生的目的是什么？需要解决的痛点是什么？以及它的实现原理。

**在这里引用项目上的一段话来概述:** 对任意突发性的无法预先感知的热点数据，包括并不限于热点数据（如突发大量请求同一个商品）、热用户（如恶意爬虫刷子）、热接口（突发海量请求同一个接口）等，进行毫秒级精准探测到。然后对这些热数据、热用户等，推送到所有服务端 JVM 内存中，以大幅减轻对后端数据存储层的冲击，并可以由使用者决定如何分配、使用这些热 key（譬如对热商品做本地缓存、对热用户进行拒绝访问、对热接口进行熔断或返回默认值）。这些热数据在整个服务端集群内保持一致性，并且业务隔离。

核心功能：热数据探测并推送至集群各个服务器

### 3.2 集成方式

集成方式在这里就不详述了，感兴趣的同学可以自行搜索。

### 3.3 源码解析

#### 3.3.1 架构简介

1\. 全景图一览

![](https://img1.jcloudcs.com/developer.jdcloud.com/ec29cd29-0e62-44b6-aa3a-37f80051216d20220811164621.png)

流程介绍:

-   客户端通过引用 hotkey 的 client 包，在启动的时候上报自己的信息给 worker，同时和 worker 之间建立长连接。定时拉取配置中心上面的规则信息和 worker 集群信息。
-   客户端调用 hotkey 的 ishot () 的方法来首先匹配规则，然后统计是不是热 key。
-   通过定时任务把热 key 数据上传到 worker 节点。
-   worker 集群在收取到所有关于这个 key 的数据以后（因为通过 hash 来决定 key 上传到哪个 worker 的，所以同一个 key 只会在同一个 worker 节点上），在和定义的规则进行匹配后判断是不是热 key，如果是则推送给客户端，完成本地缓存。

2\. 角色构成

这里直接借用作者的描述:

**1) etcd 集群** etcd 作为一个高性能的配置中心，可以以极小的资源占用，提供高效的监听订阅服务。主要用于存放规则配置，各 worker 的 ip 地址，以及探测出的热 key、手工添加的热 key 等。

**2) client 端 jar 包**就是在服务中添加的引用 jar，引入后，就可以以便捷的方式去判断某 key 是否热 key。同时，该 jar 完成了 key 上报、监听 etcd 里的 rule 变化、worker 信息变化、热 key 变化，对热 key 进行本地 caffeine 缓存等。

**3) worker 端集群** worker 端是一个独立部署的 Java 程序，启动后会连接 etcd，并定期上报自己的 ip 信息，供 client 端获取地址并进行长连接。之后，主要就是对各个 client 发来的待测 key 进行累加计算，当达到 etcd 里设定的 rule 阈值后，将热 key 推送到各个 client。

**4) dashboard 控制台**控制台是一个带可视化界面的 Java 程序，也是连接到 etcd，之后在控制台设置各个 APP 的 key 规则，譬如 2 秒 20 次算热。然后当 worker 探测出来热 key 后，会将 key 发往 etcd，dashboard 也会监听热 key 信息，进行入库保存记录。同时，dashboard 也可以手工添加、删除热 key，供各个 client 端监听。

3.hotkey 工程结构

![](https://img1.jcloudcs.com/developer.jdcloud.com/bbe0e4b4-53ac-4f2a-a957-d1a62cf0baf920220811164717.png)

#### 3.3.2 client 端

主要从下面三个方面来解析源码:

![](https://img1.jcloudcs.com/developer.jdcloud.com/3db2a6df-607e-4232-833d-6d601a2d39d320220811164733.png)

1\. 客户端启动器

1）启动方式

```
@PostConstruct
public void init() {
    ClientStarter.Builder builder = new ClientStarter.Builder();
    ClientStarter starter = builder.setAppName(appName).setEtcdServer(etcd).build();
    starter.startPipeline();
}
```

appName: 是这个应用的名称，一般为 ${spring.application.name} 的值，后续所有的配置都以此为开头

etcd: 是 etcd 集群的地址，用逗号分隔，配置中心。

还可以看到 ClientStarter 实现了建造者模式，使代码更为简介。

2）核心入口  
com.jd.platform.hotkey.client.ClientStarter#startPipeline

```

public void startPipeline() {
    JdLogger.info(getClass(), "etcdServer:" + etcdServer);
    
    Context.CAFFEINE_SIZE = caffeineSize;

    
    EtcdConfigFactory.buildConfigCenter(etcdServer);
    
    PushSchedulerStarter.startPusher(pushPeriod);
    PushSchedulerStarter.startCountPusher(10);
    
    WorkerRetryConnector.retryConnectWorkers();

    registEventBus();

    EtcdStarter starter = new EtcdStarter();
    
    starter.start();
}
```

该方法主要有五个功能:

![](https://img1.jcloudcs.com/developer.jdcloud.com/53e02843-6985-4ee2-a928-1383e6fbaab220220811164826.png)

① 设置本地缓存 (caffeine) 的最大值，并创建 etcd 实例

```

Context.CAFFEINE_SIZE = caffeineSize;


EtcdConfigFactory.buildConfigCenter(etcdServer);
```

caffeineSize 是本地缓存的最大值，在启动的时候可以设置，不设置默认为 200000。  
etcdServer 是上面说的 etcd 集群地址。

Context 可以理解为一个配置类，里面就包含两个字段:

```
public class Context {
    public static String APP_NAME;

    public static int CAFFEINE_SIZE;
}
```

EtcdConfigFactory 是 ectd 配置中心的工厂类

```
public class EtcdConfigFactory {
    private static IConfigCenter configCenter;

    private EtcdConfigFactory() {}

    public static IConfigCenter configCenter() {
        return configCenter;
    }

    public static void buildConfigCenter(String etcdServer) {
        
        configCenter = JdEtcdBuilder.build(etcdServer);
    }
}
```

通过其 configCenter () 方法获取创建 etcd 实例对象，IConfigCenter 接口封装了 etcd 实例对象的行为 (包括基本的 crud、监控、续约等)

![](https://img1.jcloudcs.com/developer.jdcloud.com/b74c63fc-574d-4af8-9451-d97a58830a5a20220811164923.png)

② 创建并启动定时任务：PushSchedulerStarter

```

PushSchedulerStarter.startPusher(pushPeriod);
PushSchedulerStarter.startCountPusher(10);
```

pushPeriod 是推送的间隔时间，可以再启动的时候设置，最小为 0.05s，推送越快，探测的越密集，会越快探测出来，但对 client 资源消耗相应增大

PushSchedulerStarter 类

```

    public static void startPusher(Long period) {
        if (period == null || period <= 0) {
            period = 500L;
        }
        @SuppressWarnings("PMD.ThreadPoolCreationRule")
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor(new NamedThreadFactory("hotkey-pusher-service-executor", true));
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            
            IKeyCollector<HotKeyModel, HotKeyModel> collectHK = KeyHandlerFactory.getCollector();
            
            
            List<HotKeyModel> hotKeyModels = collectHK.lockAndGetResult();
            if(CollectionUtil.isNotEmpty(hotKeyModels)){
                
                KeyHandlerFactory.getPusher().send(Context.APP_NAME, hotKeyModels);
                collectHK.finishOnce();
            }

        },0, period, TimeUnit.MILLISECONDS);
    }
    
    public static void startCountPusher(Integer period) {
        if (period == null || period <= 0) {
            period = 10;
        }
        @SuppressWarnings("PMD.ThreadPoolCreationRule")
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor(new NamedThreadFactory("hotkey-count-pusher-service-executor", true));
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            IKeyCollector<KeyHotModel, KeyCountModel> collectHK = KeyHandlerFactory.getCounter();
            List<KeyCountModel> keyCountModels = collectHK.lockAndGetResult();
            if(CollectionUtil.isNotEmpty(keyCountModels)){
                
                KeyHandlerFactory.getPusher().sendCount(Context.APP_NAME, keyCountModels);
                collectHK.finishOnce();
            }
        },0, period, TimeUnit.SECONDS);
    }
```

从上面两个方法可知，都是通过定时线程池来实现定时任务的，都是守护线程。

咱们重点关注一下 KeyHandlerFactory 类，它是 client 端设计的一个比较巧妙的地方，从类名上直译为 key 处理工厂。具体的实例对象是 DefaultKeyHandler:

```
public class DefaultKeyHandler {
    
    private IKeyPusher iKeyPusher = new NettyKeyPusher();
    
    private IKeyCollector<HotKeyModel, HotKeyModel> iKeyCollector = new TurnKeyCollector();
    
    private IKeyCollector<KeyHotModel, KeyCountModel> iKeyCounter = new TurnCountCollector();

    public IKeyPusher keyPusher() {
        return iKeyPusher;
    }
    public IKeyCollector<HotKeyModel, HotKeyModel> keyCollector() {
        return iKeyCollector;
    }
    public IKeyCollector<KeyHotModel, KeyCountModel> keyCounter() {
        return iKeyCounter;
    }
}
```

这里面有三个成员对象，分别是封装推送消息到 netty 的 NettyKeyPusher、待测 key 收集器 TurnKeyCollector、数量收集器 TurnCountCollector，其中后两者都实现了接口 IKeyCollector，能对 hotkey 的处理起到有效的聚合，充分体现了代码的高内聚。  
先来看看封装推送消息到 netty 的 NettyKeyPusher:

```

public class NettyKeyPusher implements IKeyPusher {
    @Override
    public void send(String appName, List<HotKeyModel> list) {
        
        long now = System.currentTimeMillis();

        Map<Channel, List<HotKeyModel>> map = new HashMap<>();
        for(HotKeyModel model : list) {
            model.setCreateTime(now);
            Channel channel = WorkerInfoHolder.chooseChannel(model.getKey());
            if (channel == null) {
                continue;
            }
            List<HotKeyModel> newList = map.computeIfAbsent(channel, k -> new ArrayList<>());
            newList.add(model);
        }

        for (Channel channel : map.keySet()) {
            try {
                List<HotKeyModel> batch = map.get(channel);
                HotKeyMsg hotKeyMsg = new HotKeyMsg(MessageType.REQUEST_NEW_KEY, Context.APP_NAME);
                hotKeyMsg.setHotKeyModels(batch);
                channel.writeAndFlush(hotKeyMsg).sync();
            } catch (Exception e) {
                try {
                    InetSocketAddress insocket = (InetSocketAddress) channel.remoteAddress();
                    JdLogger.error(getClass(),"flush error " + insocket.getAddress().getHostAddress());
                } catch (Exception ex) {
                    JdLogger.error(getClass(),"flush error");
                }
            }
        }
    }
    @Override
    public void sendCount(String appName, List<KeyCountModel> list) {
        
        long now = System.currentTimeMillis();
        Map<Channel, List<KeyCountModel>> map = new HashMap<>();
        for(KeyCountModel model : list) {
            model.setCreateTime(now);
            Channel channel = WorkerInfoHolder.chooseChannel(model.getRuleKey());
            if (channel == null) {
                continue;
            }
            List<KeyCountModel> newList = map.computeIfAbsent(channel, k -> new ArrayList<>());
            newList.add(model);
        }
        for (Channel channel : map.keySet()) {
            try {
                List<KeyCountModel> batch = map.get(channel);
                HotKeyMsg hotKeyMsg = new HotKeyMsg(MessageType.REQUEST_HIT_COUNT, Context.APP_NAME);
                hotKeyMsg.setKeyCountModels(batch);
                channel.writeAndFlush(hotKeyMsg).sync();
            } catch (Exception e) {
                try {
                    InetSocketAddress insocket = (InetSocketAddress) channel.remoteAddress();
                    JdLogger.error(getClass(),"flush error " + insocket.getAddress().getHostAddress());
                } catch (Exception ex) {
                    JdLogger.error(getClass(),"flush error");
                }
            }
        }
    }
}
```

send(String appName, List list)  
主要是将 TurnKeyCollector 收集的待测 key 通过 netty 推送给 worker，HotKeyModel 对象主要是一些热 key 的元数据信息 (热 key 来源的 app 和 key 的类型和是否是删除事件，还有该热 key 的上报次数)

sendCount(String appName, List list)  
主要是将 TurnCountCollector 收集的规则所对应的 key 通过 netty 推送给 worker，KeyCountModel 对象主要是一些 key 所对应的规则信息以及访问次数等

WorkerInfoHolder.chooseChannel(model.getRuleKey())  
根据 hash 算法获取 key 对应的服务器，分发到对应服务器相应的 Channel 连接，所以服务端可以水平无限扩容，毫无压力问题。

再来分析一下 key 收集器：TurnKeyCollector 与 TurnCountCollector:  
实现 IKeyCollector 接口:

```

public interface IKeyCollector<T, V> {
    
    List<V> lockAndGetResult();
    
    void collect(T t);

    void finishOnce();
}
```

lockAndGetResult()  
主要是获取返回 collect 方法收集的信息，并将本地暂存的信息清空，方便下个统计周期积攒数据。

collect(T t)  
顾名思义他是收集 api 调用的时候，将收集的到 key 信息放到本地存储。

finishOnce()  
该方法目前实现都是空，无需关注。

待测 key 收集器：TurnKeyCollector

```
public class TurnKeyCollector implements IKeyCollector<HotKeyModel, HotKeyModel> {
    
    private ConcurrentHashMap<String, HotKeyModel> map0 = new ConcurrentHashMap<>();
    private ConcurrentHashMap<String, HotKeyModel> map1 = new ConcurrentHashMap<>();
    private AtomicLong atomicLong = new AtomicLong(0);

    @Override
    public List<HotKeyModel> lockAndGetResult() {
        
        atomicLong.addAndGet(1);
        List<HotKeyModel> list;
        
        
        if (atomicLong.get() % 2 == 0) {
            list = get(map1);
            map1.clear();
        } else {
            list = get(map0);
            map0.clear();
        }
        return list;
    }
    private List<HotKeyModel> get(ConcurrentHashMap<String, HotKeyModel> map) {
        return CollectionUtil.list(false, map.values());
    }
    @Override
    public void collect(HotKeyModel hotKeyModel) {
        String key = hotKeyModel.getKey();
        if (StrUtil.isEmpty(key)) {
            return;
        }
        if (atomicLong.get() % 2 == 0) {
            
            HotKeyModel model = map0.putIfAbsent(key, hotKeyModel);
            if (model != null) {
                
                model.add(hotKeyModel.getCount());
            }
        } else {
            HotKeyModel model = map1.putIfAbsent(key, hotKeyModel);
            if (model != null) {
                model.add(hotKeyModel.getCount());
            }
        }
    }
    @Override
    public void finishOnce() {}
}
```

可以看到该类中有两个 ConcurrentHashMap 和一个 AtomicLong，通过对 AtomicLong 来自增，然后对 2 取模，来分别控制两个 map 的读写能力，保证每个 map 都能做读写，并且同一个 map 不能同时读写，这样可以避免并发集合读写不阻塞，这一块无锁化的设计还是非常巧妙的，极大的提高了收集的吞吐量。

key 数量收集器：TurnCountCollector  
这里的设计与 TurnKeyCollector 大同小异，咱们就不细谈了。值得一提的是它里面有个并行处理的机制，当收集的数量超过 DATA\_CONVERT\_SWITCH\_THRESHOLD=5000 的阈值时，lockAndGetResult 处理是使用 java Stream 并行流处理，提升处理的效率。

③ 开启 worker 重连器

```

WorkerRetryConnector.retryConnectWorkers();
public class WorkerRetryConnector {

    
    public static void retryConnectWorkers() {
        @SuppressWarnings("PMD.ThreadPoolCreationRule")
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor(new NamedThreadFactory("worker-retry-connector-service-executor", true));
        
        scheduledExecutorService.scheduleAtFixedRate(WorkerRetryConnector::reConnectWorkers, 30, 30, TimeUnit.SECONDS);
    }

    private static void reConnectWorkers() {
        List<String> nonList = WorkerInfoHolder.getNonConnectedWorkers();
        if (nonList.size() == 0) {
            return;
        }
        JdLogger.info(WorkerRetryConnector.class, "trying to reConnect to these workers :" + nonList);
        NettyClient.getInstance().connect(nonList);
    }
}
```

也是通过定时线程来执行，默认时间间隔是 30s，不可设置。  
通过 WorkerInfoHolder 来控制 client 的 worker 连接信息，连接信息是个 List，用的 CopyOnWriteArrayList，毕竟是一个读多写少的场景，类似与元数据信息。

```

private static final List<Server> WORKER_HOLDER = new CopyOnWriteArrayList<>();
```

④ 注册 EventBus 事件订阅者

```
private void registEventBus() {
    
    EventBusCenter.register(new WorkerChangeSubscriber());
    
    EventBusCenter.register(new ReceiveNewKeySubscribe());
    
    EventBusCenter.register(new KeyRuleHolder());
}
```

使用 guava 的 EventBus 事件消息总线，利用发布 / 订阅者模式来对项目进行解耦。它可以利用很少的代码，来实现多组件间通信。

基本原理图如下:

![](https://img1.jcloudcs.com/developer.jdcloud.com/f380d56f-9a2a-43af-9037-dcc4580f80a120220811165302.png)

监听 worker 信息变动：WorkerChangeSubscriber

```

@Subscribe
public void connectAll(WorkerInfoChangeEvent event) {
    List<String> addresses = event.getAddresses();
    if (addresses == null) {
        addresses = new ArrayList<>();
    }

    WorkerInfoHolder.mergeAndConnectNew(addresses);
}

@Subscribe
public void channelInactive(ChannelInactiveEvent inactiveEvent) {
    
    Channel channel = inactiveEvent.getChannel();
    InetSocketAddress socketAddress = (InetSocketAddress) channel.remoteAddress();
    String address = socketAddress.getHostName() + ":" + socketAddress.getPort();
    JdLogger.warn(getClass(), "this channel is inactive : " + socketAddress + " trying to remove this connection");

    WorkerInfoHolder.dealChannelInactive(address);
}
```

![](https://img1.jcloudcs.com/developer.jdcloud.com/5ff1dadf-e44e-447f-89aa-4e202e5b2dae20220811165333.png)

监听热 key 回调事件：ReceiveNewKeySubscribe

```
private ReceiveNewKeyListener receiveNewKeyListener = new DefaultNewKeyListener();

@Subscribe
public void newKeyComing(ReceiveNewKeyEvent event) {
    HotKeyModel hotKeyModel = event.getModel();
    if (hotKeyModel == null) {
        return;
    }
    
    if (receiveNewKeyListener != null) {
        receiveNewKeyListener.newKey(hotKeyModel);
    }
}
```

该方法会收到新的热 key 订阅事件之后，会将其加入到 KeyHandlerFactory 的收集器里面处理。

核心处理逻辑：DefaultNewKeyListener#newKey:

```
@Override
public void newKey(HotKeyModel hotKeyModel) {
    long now = System.currentTimeMillis();
    
    if (hotKeyModel.getCreateTime() != 0 && Math.abs(now - hotKeyModel.getCreateTime()) > 1000) {
        JdLogger.warn(getClass(), "the key comes too late : " + hotKeyModel.getKey() + " now " +
                +now + " keyCreateAt " + hotKeyModel.getCreateTime());
    }
    if (hotKeyModel.isRemove()) {
        
        deleteKey(hotKeyModel.getKey());
        return;
    }
    
    if (JdHotKeyStore.isHot(hotKeyModel.getKey())) {
        JdLogger.warn(getClass(), "receive repeat hot key ：" + hotKeyModel.getKey() + " at " + now);
    }
    addKey(hotKeyModel.getKey());
}
private void deleteKey(String key) {
        CacheFactory.getNonNullCache(key).delete(key);
}
private void addKey(String key) {
  ValueModel valueModel = ValueModel.defaultValue(key);
  if (valueModel == null) {
      
      deleteKey(key);
      return;
  }
  
   JdHotKeyStore.setValueDirectly(key, valueModel);
}
```

1.  如果该 HotKeyModel 里面是删除事件，则获取 RULE\_CACHE\_MAP 里面该 key 超时时间对应的 caffeine，然后从中删除该 key 缓存，然后返回 (这里相当于删除了本地缓存)。
2.  如果不是删除事件，则在 RULE\_CACHE\_MAP 对应的 caffeine 缓存中添加该 key 的缓存。
3.  这里有个注意点，如果不为删除事件，调用 addKey () 方法在 caffeine 增加缓存的时候，value 是一个魔术值 0x12fcf76，这个值只代表加了这个缓存，但是这个缓存在查询的时候相当于为 null。

监听 Rule 的变化事件：KeyRuleHolder

![](https://img1.jcloudcs.com/developer.jdcloud.com/6d8acd7e-83f1-4c6a-a35e-b0dd2fe2820d20220811165436.png)

可以看到里面有两个成员属性：RULE\_CACHE\_MAP，KEY\_RULES

```

private static final ConcurrentHashMap<Integer, LocalCache> RULE_CACHE_MAP = new ConcurrentHashMap<>();

private static final List<KeyRule> KEY_RULES = new ArrayList<>();
```

ConcurrentHashMap RULE\_CACHE\_MAP:

-   保存超时时间和 caffeine 的映射，key 是超时时间，value 是 caffeine \[(String,Object)\]。
-   巧妙的设计：这里将 key 的过期时间作为分桶策略，这样同一个过期时间的 key 就会在一个桶 (caffeine) 里面，这里面每一个 caffeine 都是 client 的本地缓存，也就是说 hotKey 的本地缓存的 KV 实际上是存储在这里面的。

List KEY\_RULES:

-   这里 KEY\_RULES 是保存 etcd 里面该 appName 所对应的所有 rule。

具体监听 KeyRuleInfoChangeEvent 事件方法:

```
@Subscribe
public void ruleChange(KeyRuleInfoChangeEvent event) {
    JdLogger.info(getClass(), "new rules info is :" + event.getKeyRules());
    List<KeyRule> ruleList = event.getKeyRules();
    if (ruleList == null) {
        return;
    }

    putRules(ruleList);
}
```

核心处理逻辑：KeyRuleHolder#putRules:

```

public static void putRules(List<KeyRule> keyRules) {
    synchronized (KEY_RULES) {
        
        if (CollectionUtil.isEmpty(keyRules)) {
            KEY_RULES.clear();
            RULE_CACHE_MAP.clear();
            return;
        }
        KEY_RULES.clear();
        KEY_RULES.addAll(keyRules);
        Set<Integer> durationSet = keyRules.stream().map(KeyRule::getDuration).collect(Collectors.toSet());
        for (Integer duration : RULE_CACHE_MAP.keySet()) {
            
            if (!durationSet.contains(duration)) {
                RULE_CACHE_MAP.remove(duration);
            }
        }
        
        for (KeyRule keyRule : keyRules) {
            int duration = keyRule.getDuration();
            
            
            
            if (RULE_CACHE_MAP.get(duration) == null) {
                LocalCache cache = CacheFactory.build(duration);
                RULE_CACHE_MAP.put(duration, cache);
            }
        }
    }
}
```

-   使用 synchronized 关键字来保证线程安全；
-   如果规则为空，清空规则表 (RULE\_CACHE\_MAP、KEY\_RULES);
-   使用传递进来的 keyRules 来覆盖 KEY\_RULES;
-   清除掉 RULE\_CACHE\_MAP 里面在 keyRules 没有的映射关系；
-   遍历所有的 keyRules，如果 RULE\_CACHE\_MAP 里面没有相关的超时时间 key，则在里面赋值；

⑤ 启动 EtcdStarter (etcd 连接管理器)

```
EtcdStarter starter = new EtcdStarter();

starter.start();

public void start() {
fetchWorkerInfo();
fetchRule();
startWatchRule();

startWatchHotKey();
}
```

fetchWorkerInfo()

从 etcd 里面拉取 worker 集群地址信息 allAddress，并更新 WorkerInfoHolder 里面的 WORKER\_HOLDER

```

private void fetchWorkerInfo() {
    ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
    
    scheduledExecutorService.scheduleAtFixedRate(() -> {
        JdLogger.info(getClass(), "trying to connect to etcd and fetch worker info");
        fetch();

    }, 0, 30, TimeUnit.SECONDS);
}
```

-   使用定时线程池来执行，单线程。
-   定时从 etcd 里面获取，地址 /jd/workers/+$appName 或 default，时间间隔不可设置，默认 30 秒，这里面存储的是 worker 地址的 ip+port。
-   发布 WorkerInfoChangeEvent 事件。
-   备注：地址有 $appName 或 default，在 worker 里面配置，如果把 worker 放到某个 appName 下，则该 worker 只会参与该 app 的计算。

fetchRule()

定时线程来执行，单线程，时间间隔不可设置，默认是 5 秒，当拉取规则配置和手动配置的 hotKey 成功后，该线程被终止 (也就是说只会成功执行一次)，执行失败继续执行

```
private void fetchRule() {
    ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
    
    scheduledExecutorService.scheduleAtFixedRate(() -> {
        JdLogger.info(getClass(), "trying to connect to etcd and fetch rule info");
        boolean success = fetchRuleFromEtcd();
        if (success) {
            
            fetchExistHotKey();
            
            scheduledExecutorService.shutdown();
        }
    }, 0, 5, TimeUnit.SECONDS);
}
```

fetchRuleFromEtcd()

-   从 etcd 里面获取该 appName 配置的 rule 规则，地址 /jd/rules/+$appName。
-   如果查出来规则 rules 为空，会通过发布 KeyRuleInfoChangeEvent 事件来清空本地的 rule 配置缓存和所有的规则 key 缓存。
-   发布 KeyRuleInfoChangeEvent 事件。

fetchExistHotKey()

-   从 etcd 里面获取该 appName 手动配置的热 key，地址 /jd/hotkeys/+$appName。
-   发布 ReceiveNewKeyEvent 事件，并且内容 HotKeyModel 不是删除事件。

startWatchRule()

```

private void startWatchRule() {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    executorService.submit(() -> {
        JdLogger.info(getClass(), "--- begin watch rule change ----");
        try {
            IConfigCenter configCenter = EtcdConfigFactory.configCenter();
            KvClient.WatchIterator watchIterator = configCenter.watch(ConfigConstant.rulePath + Context.APP_NAME);
            
            while (watchIterator.hasNext()) {
                
                WatchUpdate watchUpdate = watchIterator.next();
                List<Event> eventList = watchUpdate.getEvents();
                JdLogger.info(getClass(), "rules info changed. begin to fetch new infos. rule change is " + eventList);

                
                fetchRuleFromEtcd();
            }
        } catch (Exception e) {
            JdLogger.error(getClass(), "watch err");
        }
    });
}
```

-   异步监听 rule 规则变化，使用 etcd 监听地址为 /jd/rules/+$appName 的节点变化。
-   使用线程池，单线程，异步监听 rule 规则变化，如果有事件变化，则调用 fetchRuleFromEtcd () 方法。

startWatchHotKey()  
异步开始监听热 key 变化信息，使用 etcd 监听地址前缀为 /jd/hotkeys/+$appName

```

private void startWatchHotKey() {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    executorService.submit(() -> {
        JdLogger.info(getClass(), "--- begin watch hotKey change ----");
        IConfigCenter configCenter = EtcdConfigFactory.configCenter();
        try {
            KvClient.WatchIterator watchIterator = configCenter.watchPrefix(ConfigConstant.hotKeyPath + Context.APP_NAME);
            
            while (watchIterator.hasNext()) {
                WatchUpdate watchUpdate = watchIterator.next();

                List<Event> eventList = watchUpdate.getEvents();
                KeyValue keyValue = eventList.get(0).getKv();
                Event.EventType eventType = eventList.get(0).getType();
                try {
                    
                    String key = keyValue.getKey().toStringUtf8().replace(ConfigConstant.hotKeyPath + Context.APP_NAME + "/", "");
                    
                    if (Event.EventType.DELETE == eventType) {
                        HotKeyModel model = new HotKeyModel();
                        model.setRemove(true);
                        model.setKey(key);
                        EventBusCenter.getInstance().post(new ReceiveNewKeyEvent(model));
                    } else {
                        HotKeyModel model = new HotKeyModel();
                        model.setRemove(false);
                        String value = keyValue.getValue().toStringUtf8();
                        
                        JdLogger.info(getClass(), "etcd receive new key : " + key + " --value:" + value);
                        
                        
                        
                        
                        if (Constant.DEFAULT_DELETE_VALUE.equals(value)) {
                            continue;
                        }
                        
                        model.setCreateTime(Long.valueOf(keyValue.getValue().toStringUtf8()));
                        model.setKey(key);
                        EventBusCenter.getInstance().post(new ReceiveNewKeyEvent(model));
                    }
                } catch (Exception e) {
                    JdLogger.error(getClass(), "new key err ：" + keyValue);
                }

            }
        } catch (Exception e) {
            JdLogger.error(getClass(), "watch err");
        }
    });

}
```

-   使用线程池，单线程，异步监听热 key 变化
-   使用 etcd 监听前缀地址的当前节点以及子节点的所有变化值
-   删除节点动作
-   发布 ReceiveNewKeyEvent 事件，并且内容 HotKeyModel 是删除事件
-   新增 or 更新节点动作
-   事件变化的 value 值为删除标记 #\[DELETE\]#
-   如果是删除标记的话，代表是 worker 自动探测或者 client 需要删除的指令。
-   如果是删除标记则什么也不做，直接跳过 (这里从 HotKeyPusher#push 方法可以看到，做删除事件的操作时候，他会给 /jd/hotkeys/+$appName 的节点里面增加一个值为删除标记的节点，然后再删除相同路径的节点，这样就可以触发上面的删除节点事件，所以这里判断如果是删除标记直接跳过)。
-   不为删除标记
-   发布 ReceiveNewKeyEvent 事件，事件内容 HotKeyModel 里面的 createTime 是 kv 对应的时间戳

疑问：这里代码注释里面说只监听手工添加或者删除的 hotKey，难道说 /jd/hotkeys/+$appName 地址只是手工配置的地址吗？

解疑：这里确实只监听手工配置的 hotKey，etcd 的 /jd/hotkeys/+$appName 该地址只是手动配置 hotKey，worker 自动探测的 hotKey 是直接通过 netty 通道来告知 client 的

2.API 解析

1）流程图示  
① 查询流程

![](https://img1.jcloudcs.com/developer.jdcloud.com/6edda0cf-9fa2-4ed1-a93a-ac86b389d99720220811170207.png)

② 删除流程:

![](https://img1.jcloudcs.com/developer.jdcloud.com/b5bbb066-1682-4005-988d-e98357f9b3dd20220811170220.png)

从上面的流程图中，大家应该知道该热点 key 在代码中是如何扭转的，这里再给大家讲解一下核心 API 的源码解析，限于篇幅的原因，咱们不一个个贴相关源码了，只是单纯的告诉你它的内部逻辑是怎么样的。

2）核心类：JdHotKeyStore

![](https://img1.jcloudcs.com/developer.jdcloud.com/74375688-df6b-482d-8e02-dba92994b30d20220811170236.png)

JdHotKeyStore 是封装 client 调用的 api 核心类，包含上面 10 个公共方法，咱们重点解析其中 6 个重要方法:

① isHotKey(String key)  
判断是否在规则内，如果不在，返回 false  
判断是否是热 key，如果不是或者是且过期时间在 2s 内，则给 TurnKeyCollector#collect 收集  
最后给 TurnCountCollector#collect 做统计收集

② get(String key)  
从本地 caffeine 取值  
如果取到的 value 是个魔术值，只代表加入到 caffeine 缓存里面了，查询的话为 null

③ smartSet(String key, Object value)  
判断是否是热 key，这里不管它在不在规则内，如果是热 key，则给 value 赋值，如果不为热 key 什么也不做

④ forceSet(String key, Object value)  
强制给 value 赋值  
如果该 key 不在规则配置内，则传递的 value 不生效，本地缓存的赋值 value 会被变为 null

⑤ getValue(String key, KeyType keyType)  
获取 value，如果 value 不存在则调用 HotKeyPusher#push 方法发往 netty  
如果没有为该 key 配置规则，就不用上报 key，直接返回 null  
如果取到的 value 是个魔术值，只代表加入到 caffeine 缓存里面了，查询的话为 null

⑥ remove(String key)  
删除某 key (本地的 caffeine 缓存)，会通知整个集群删除 (通过 etcd 来通知集群删除)

3）client 上传热 key 入口调用类：HotKeyPusher  
核心方法:

```
public static void push(String key, KeyType keyType, int count, boolean remove) {
    if (count <= 0) {
        count = 1;
    }
    if (keyType == null) {
        keyType = KeyType.REDIS_KEY;
    }
    if (key == null) {
        return;
    }
    
    
    LongAdder adderCnt = new LongAdder();
    adderCnt.add(count);

    HotKeyModel hotKeyModel = new HotKeyModel();
    hotKeyModel.setAppName(Context.APP_NAME);
    hotKeyModel.setKeyType(keyType);
    hotKeyModel.setCount(adderCnt);
    hotKeyModel.setRemove(remove);
    hotKeyModel.setKey(key);


    if (remove) {
        
        
        
        
        
        EtcdConfigFactory.configCenter().putAndGrant(HotKeyPathTool.keyPath(hotKeyModel), Constant.DEFAULT_DELETE_VALUE, 1);
        EtcdConfigFactory.configCenter().delete(HotKeyPathTool.keyPath(hotKeyModel));
        
        EtcdConfigFactory.configCenter().delete(HotKeyPathTool.keyRecordPath(hotKeyModel));
    } else {
        
        if (KeyRuleHolder.isKeyInRule(key)) {
            
            KeyHandlerFactory.getCollector().collect(hotKeyModel);
        }
    }
}
```

从上面的源码中可知:

-   这里之所以用 LongAdder 是为了保证多线程计数的线程安全性，虽然这里是在方法内调用的，但是在 TurnKeyCollector 的两个 map 里面，存储了 HotKeyModel 的实例对象，这样在多个线程同时修改 count 的计数属性时，会存在线程安全计数不准确问题。
-   如果是 remove 删除类型，在删除手动配置的热 key 配置路径的同时，还会删除 dashboard 展示热 key 的配置路径。
-   只有在规则配置的 key，才会被积攒探测发送到 worker 内进行计算。

3\. 通讯机制 (与 worker 交互)

1）NettyClient:netty 连接器

```
public class NettyClient {
    private static final NettyClient nettyClient = new NettyClient();

    private Bootstrap bootstrap;

    public static NettyClient getInstance() {
        return nettyClient;
    }

    private NettyClient() {
        if (bootstrap == null) {
            bootstrap = initBootstrap();
        }
    }

    private Bootstrap initBootstrap() {
        
        EventLoopGroup group = new NioEventLoopGroup(2);

        Bootstrap bootstrap = new Bootstrap();
        NettyClientHandler nettyClientHandler = new NettyClientHandler();
        bootstrap.group(group).channel(NioSocketChannel.class)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .option(ChannelOption.TCP_NODELAY, true)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) {
                        ByteBuf delimiter = Unpooled.copiedBuffer(Constant.DELIMITER.getBytes());
                        ch.pipeline()
                                .addLast(new DelimiterBasedFrameDecoder(Constant.MAX_LENGTH, delimiter))
                                .addLast(new MsgDecoder())
                                .addLast(new MsgEncoder())
                                
                                .addLast(new IdleStateHandler(0, 0, 30))
                                .addLast(nettyClientHandler);
                    }
                });
        return bootstrap;
    }
}
```

-   使用 Reactor 线程模型，只有 2 个工作线程，没有单独设置主线程
-   长连接，开启 TCP\_NODELAY
-   netty 的分隔符”$( )$”，类似 TCP 报文分段的标准，方便拆包
-   Protobuf 序列化与反序列化
-   30s 没有消息发给对端的时候，发送一个心跳包判活
-   工作线程处理器 NettyClientHandler

JDhotkey 的 tcp 协议设计就是收发字符串，每个 tcp 消息包使用特殊字符 $( )$ 来分割  
优点：这样实现非常简单。

获得消息包后进行 json 或者 protobuf 反序列化。

缺点：是需要，从字节流 -》反序列化成字符串 -》反序列化成消息对象，两层序列化损耗了一部分性能。

protobuf 还好序列化很快，但是 json 序列化的速度只有几十万每秒，会损耗一部分性能。

2）NettyClientHandler: 工作线程处理器

```
@ChannelHandler.Sharable
public class NettyClientHandler extends SimpleChannelInboundHandler<HotKeyMsg> {
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent) {
            IdleStateEvent idleStateEvent = (IdleStateEvent) evt;
            
            if (idleStateEvent.state() == IdleState.ALL_IDLE) {
                
                ctx.writeAndFlush(new HotKeyMsg(MessageType.PING, Context.APP_NAME));
            }
        }

        super.userEventTriggered(ctx, evt);
    }
    
    
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        JdLogger.info(getClass(), "channelActive:" + ctx.name());
        ctx.writeAndFlush(new HotKeyMsg(MessageType.APP_NAME, Context.APP_NAME));
    }
    

    
    
    
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        super.channelInactive(ctx);
        
        
        notifyWorkerChange(ctx.channel());
    }
    private void notifyWorkerChange(Channel channel) {
        EventBusCenter.getInstance().post(new ChannelInactiveEvent(channel));
    }
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, HotKeyMsg msg) {
        if (MessageType.PONG == msg.getMessageType()) {
            JdLogger.info(getClass(), "heart beat");
            return;
        }
        if (MessageType.RESPONSE_NEW_KEY == msg.getMessageType()) {
            JdLogger.info(getClass(), "receive new key : " + msg);
            if (CollectionUtil.isEmpty(msg.getHotKeyModels())) {
                return;
            }
            for (HotKeyModel model : msg.getHotKeyModels()) {
                EventBusCenter.getInstance().post(new ReceiveNewKeyEvent(model));
            }
        }
    }
}
```

userEventTriggered

-   收到对端发来的心跳包，返回 new HotKeyMsg (MessageType.PING, Context.APP\_NAME)

channelActive

-   在 Channel 注册 EventLoop、绑定 SocketAddress 和连接 ChannelFuture 的时候都有可能会触发 ChannelInboundHandler 的 channelActive 方法的调用
-   类似 TCP 三次握手成功之后触发，给对端发送 new HotKeyMsg (MessageType.APP\_NAME, Context.APP\_NAME)

channelInactive

-   类似 TCP 四次挥手之后，等待 2MSL 时间之后触发 (大概 180s)，比如 channel 通道关闭会触发 (channel.close ()) 该方法，发布 ChannelInactiveEvent 事件，来 10s 后重连

channelRead0

-   接收 PONG 消息类型时，打个日志返回
-   接收 RESPONSE\_NEW\_KEY 消息类型时，发布 ReceiveNewKeyEvent 事件

#### 3.3.3 worker 端

1\. 入口启动加载：7 个 @PostConstruct

1）worker 端对 etcd 相关的处理：EtcdStarter  
① 第一个 @PostConstruct:watchLog ()

```
@PostConstruct
public void watchLog() {
    AsyncPool.asyncDo(() -> {
        try {
            
            String loggerOn = configCenter.get(ConfigConstant.logToggle);
            LOGGER_ON = "true".equals(loggerOn) || "1".equals(loggerOn);
        } catch (StatusRuntimeException ex) {
            logger.error(ETCD_DOWN);
        }
        
        KvClient.WatchIterator watchIterator = configCenter.watch(ConfigConstant.logToggle);
        while (watchIterator.hasNext()) {
            WatchUpdate watchUpdate = watchIterator.next();
            List<Event> eventList = watchUpdate.getEvents();
            KeyValue keyValue = eventList.get(0).getKv();
            logger.info("log toggle changed : " + keyValue);
            String value = keyValue.getValue().toStringUtf8();
            LOGGER_ON = "true".equals(value) || "1".equals(value);
        }
    });
}
```

-   放到线程池里面异步执行
-   取 etcd 的是否开启日志配置，地址 /jd/logOn，默认 true
-   监听 etcd 地址 /jd/logOn 是否开启日志配置，并实时更改开关
-   由于有 etcd 的监听，所以会一直执行，而不是执行一次结束

② 第二个 @PostConstruct:watch ()

```

@PostConstruct
public void watch() {
    AsyncPool.asyncDo(() -> {
        KvClient.WatchIterator watchIterator;
        if (isForSingle()) {
            watchIterator = configCenter.watch(ConfigConstant.rulePath + workerPath);
        } else {           
            watchIterator = configCenter.watchPrefix(ConfigConstant.rulePath);
        }
        while (watchIterator.hasNext()) {
            WatchUpdate watchUpdate = watchIterator.next();
            List<Event> eventList = watchUpdate.getEvents();
            KeyValue keyValue = eventList.get(0).getKv();
            logger.info("rule changed : " + keyValue);
            try {
                ruleChange(keyValue);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
}

    private synchronized void ruleChange(KeyValue keyValue) {
        String appName = keyValue.getKey().toStringUtf8().replace(ConfigConstant.rulePath, "");
        if (StrUtil.isEmpty(appName)) {
            return;
        }
        String ruleJson = keyValue.getValue().toStringUtf8();
        List<KeyRule> keyRules = FastJsonUtils.toList(ruleJson, KeyRule.class);
        KeyRuleHolder.put(appName, keyRules);
    }
```

通过 etcd.workerPath 配置，来判断该 worker 是否为某个 app 单独服务的，默认为”default”，如果是默认值，代表该 worker 参与在 etcd 上所有 app client 的计算，否则只为某个 app 来服务计算

使用 etcd 来监听 rule 规则变化，如果是共享的 worker，监听地址前缀为”/jd/rules/“，如果为某个 app 独享，监听地址为”/jd/rules/“+$etcd.workerPath

如果规则变化，则修改对应 app 在本地存储的 rule 缓存，同时清理该 app 在本地存储的 KV 缓存

KeyRuleHolder:rule 缓存本地存储

-   Map> RULE\_MAP，这个 map 是 concurrentHashMap，map 的 kv 分别是 appName 和对应的 rule
-   相对于 client 的 KeyRuleHolder 的区别:worker 是存储所有 app 规则，每个 app 对应一个规则桶，所以用 map

CaffeineCacheHolder:key 缓存本地存储

-   Map> CACHE\_MAP，也是 concurrentHashMap，map 的 kv 分别是 appName 和对应的 kv 的 caffeine
-   相对于 client 的 caffeine，第一是 worker 没有做缓存接口比如 LocalCache，第二是 client 的 map 的 kv 分别是超时时间、以及相同超时时间所对应 key 的缓存桶

放到线程池里面异步执行，由于有 etcd 的监听，所以会一直执行，而不是执行一次结束

③ 第三个 @PostConstruct:watchWhiteList ()

```

@PostConstruct
public void watchWhiteList() {
    AsyncPool.asyncDo(() -> {
        
        fetchWhite();
        KvClient.WatchIterator watchIterator = configCenter.watch(ConfigConstant.whiteListPath + workerPath);
        while (watchIterator.hasNext()) {
            WatchUpdate watchUpdate = watchIterator.next();
            logger.info("whiteList changed ");
            try {
                fetchWhite();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
}
```

-   拉取并监听 etcd 白名单 key 配置，地址为 /jd/whiteList/+$etcd.workerPath
-   在白名单的 key，不参与热 key 计算，直接忽略
-   放到线程池里面异步执行，由于有 etcd 的监听，所以会一直执行，而不是执行一次结束

④ 第四个 @PostConstruct:makeSureSelfOn ()

```

@PostConstruct
public void makeSureSelfOn() {
    
    ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
    scheduledExecutorService.scheduleAtFixedRate(() -> {
        try {
            if (canUpload) {
                uploadSelfInfo();
            }
        } catch (Exception e) {
            
        }
    }, 0, 5, TimeUnit.SECONDS);
}
```

-   在线程池里面异步执行，定时执行，时间间隔为 5s
-   将本机 woker 的 hostName，ip+port 以 kv 的形式定时上报给 etcd，地址为 /jd/workers/+$etcd.workPath+”/“+$hostName，续期时间为 8s
-   有一个 canUpload 的开关来控制 worker 是否向 etcd 来定时续期，如果这个开关关闭了，代表 worker 不向 etcd 来续期，这样当上面地址的 kv 到期之后，etcd 会删除该节点，这样 client 循环判断 worker 信息变化了

2）将热 key 推送到 dashboard 供入库：DashboardPusher

① 第五个 @PostConstruct:uploadToDashboard ()

```
@Component
public class DashboardPusher implements IPusher {
    
    private static LinkedBlockingQueue<HotKeyModel> hotKeyStoreQueue = new LinkedBlockingQueue<>();

    @PostConstruct
    public void uploadToDashboard() {
        AsyncPool.asyncDo(() -> {
            while (true) {
                try {
                    
                    List<HotKeyModel> tempModels = new ArrayList<>();
                    Queues.drain(hotKeyStoreQueue, tempModels, 1000, 1, TimeUnit.SECONDS);
                    if (CollectionUtil.isEmpty(tempModels)) {
                        continue;
                    }

                    
                    DashboardHolder.flushToDashboard(FastJsonUtils.convertObjectToJSON(tempModels));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

-   当热 key 的数量达到 1000 或者每隔 1s，把热 key 的数据通过与 dashboard 的 netty 通道来发送给 dashboard，数据类型为 REQUEST\_HOT\_KEY
-   LinkedBlockingQueue hotKeyStoreQueue：worker 计算的给 dashboard 热 key 的集中营，所有给 dashboard 推送热 key 存储在里面

3）推送到各客户端服务器：AppServerPusher

① 第六个 @PostConstruct:batchPushToClient ()

```
public class AppServerPusher implements IPusher {
    
    private static LinkedBlockingQueue<HotKeyModel> hotKeyStoreQueue = new LinkedBlockingQueue<>();

    
    @PostConstruct
    public void batchPushToClient() {
        AsyncPool.asyncDo(() -> {
            while (true) {
                try {
                    List<HotKeyModel> tempModels = new ArrayList<>();
                    
                    Queues.drain(hotKeyStoreQueue, tempModels, 10, 10, TimeUnit.MILLISECONDS);
                    if (CollectionUtil.isEmpty(tempModels)) {
                        continue;
                    }
                    Map<String, List<HotKeyModel>> allAppHotKeyModels = new HashMap<>();
                    
                    for (HotKeyModel hotKeyModel : tempModels) {
                        List<HotKeyModel> oneAppModels = allAppHotKeyModels.computeIfAbsent(hotKeyModel.getAppName(), (key) -> new ArrayList<>());
                        oneAppModels.add(hotKeyModel);
                    }
                    
                    for (AppInfo appInfo : ClientInfoHolder.apps) {
                        List<HotKeyModel> list = allAppHotKeyModels.get(appInfo.getAppName());
                        if (CollectionUtil.isEmpty(list)) {
                            continue;
                        }
                        HotKeyMsg hotKeyMsg = new HotKeyMsg(MessageType.RESPONSE_NEW_KEY);
                        hotKeyMsg.setHotKeyModels(list);

                        
                        appInfo.groupPush(hotKeyMsg);
                    }
                    
                    allAppHotKeyModels = null;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

-   会按照 key 的 appName 来进行分组，然后通过对应 app 的 channelGroup 来推送
-   当热 key 的数量达到 10 或者每隔 10ms，把热 key 的数据通过与 app 的 netty 通道来发送给 app，数据类型为 RESPONSE\_NEW\_KEY
-   LinkedBlockingQueue hotKeyStoreQueue：worker 计算的给 client 热 key 的集中营，所有给 client 推送热 key 存储在里面

4）client 实例节点处理：NodesServerStarter  
① 第七个 @PostConstruct:start ()

```
public class NodesServerStarter {
    @Value("${netty.port}")
    private int port;
    private Logger logger = LoggerFactory.getLogger(getClass());
    @Resource
    private IClientChangeListener iClientChangeListener;
    @Resource
    private List<INettyMsgFilter> messageFilters;

    @PostConstruct
    public void start() {
        AsyncPool.asyncDo(() -> {
            logger.info("netty server is starting");
            NodesServer nodesServer = new NodesServer();
            nodesServer.setClientChangeListener(iClientChangeListener);
            nodesServer.setMessageFilters(messageFilters);
            try {
                nodesServer.startNettyServer(port);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
    }
}
```

-   线程池里面异步执行，启动 client 端的 nettyServer
-   iClientChangeListener 和 messageFilters 这两个依赖最终会被传递到 netty 消息处理器里面，iClientChangeListener 会作为 channel 下线处理来删除 ClientInfoHolder 下线或者超时的通道，messageFilters 会作为 netty 收到事件消息的处理过滤器 (责任链模式)

② 依赖的 bean:IClientChangeListener iClientChangeListener

```
public interface IClientChangeListener {
    
    void newClient(String appName, String channelId, ChannelHandlerContext ctx);
    
    void loseClient(ChannelHandlerContext ctx);
}
```

对客户端的管理，新来 (newClient)(会触发 netty 的连接方法 channelActive)、断线 (loseClient)(会触发 netty 的断连方法 channelInactive ()) 的管理  
client 的连接信息主要是在 ClientInfoHolder 里面

-   List apps，这里面的 AppInfo 主要是 appName 和对应的 channelGroup
-   对 apps 的 add 和 remove 主要是通过新来 (newClient)、断线 (loseClient)

③ 依赖的 bean:List messageFilters

```

public interface INettyMsgFilter {
    boolean chain(HotKeyMsg message, ChannelHandlerContext ctx);
}
```

对 client 发给 worker 的 netty 消息，进行过滤处理，共有四个实现类，也就是说底下四个过滤器都是收到 client 发送的 netty 消息来做处理

④ 各个消息处理的类型：MessageType

```
APP_NAME((byte) 1),
REQUEST_NEW_KEY((byte) 2),
RESPONSE_NEW_KEY((byte) 3),
REQUEST_HIT_COUNT((byte) 7), 
REQUEST_HOT_KEY((byte) 8), 
PING((byte) 4), PONG((byte) 5),
EMPTY((byte) 6);
```

顺序 1:HeartBeatFilter

-   当消息类型为 PING，则给对应的 client 示例返回 PONG

顺序 2:AppNameFilter

-   当消息类型为 APP\_NAME，代表 client 与 worker 建立连接成功，然后调用 iClientChangeListener 的 newClient 方法增加 apps 元数据信息

顺序 3:HotKeyFilter

-   处理接收消息类型为 REQUEST\_NEW\_KEY
-   先给 HotKeyFilter.totalReceiveKeyCount 原子类增 1，该原子类代表 worker 实例接收到的 key 的总数
-   publishMsg 方法，将消息通过自建的生产者消费者模型 (KeyProducer，KeyConsumer)，来把消息给发到生产者中分发消费
-   接收到的消息 HotKeyMsg 里面 List
-   首先判断 HotKeyModel 里面的 key 是否在白名单内，如果在则跳过，否则将 HotKeyModel 通过 KeyProducer 发送

顺序 4:KeyCounterFilter

-   处理接收类型为 REQUEST\_HIT\_COUNT
-   这个过滤器是专门给 dashboard 来汇算 key 的，所以这个 appName 直接设置为该 worker 配置的 appName
-   该过滤器的数据来源都是 client 的 NettyKeyPusher#sendCount (String appName, List list)，这里面的数据都是默认积攒 10s 的，这个 10s 是可以配置的，这一点在 client 里面有讲
-   将构造的 new KeyCountItem (appName, models.get (0).getCreateTime (), models) 放到阻塞队列 LinkedBlockingQueue COUNTER\_QUEUE 中，然后让 CounterConsumer 来消费处理，消费逻辑是单线程的
-   CounterConsumer：热 key 统计消费者
-   放在公共线程池中，来单线程执行
-   从阻塞队列 COUNTER\_QUEUE 里面取数据，然后将里面的 key 的统计数据发布到 etcd 的 /jd/keyHitCount/+ appName + “/“ + IpUtils.getIp () + “-“ + System.currentTimeMillis () 里面，该路径是 worker 服务的 client 集群或者 default，用来存放客户端 hotKey 访问次数和总访问次数的 path，然后让 dashboard 来订阅统计展示

2\. 三个定时任务：3 个 @Scheduled

1）定时任务 1:EtcdStarter#pullRules ()

```

@Scheduled(fixedRate = 60000)
public void pullRules() {
    try {
        if (isForSingle()) {
            String value = configCenter.get(ConfigConstant.rulePath + workerPath);
            if (!StrUtil.isEmpty(value)) {
                List<KeyRule> keyRules = FastJsonUtils.toList(value, KeyRule.class);
                KeyRuleHolder.put(workerPath, keyRules);
            }
        } else {
            List<KeyValue> keyValues = configCenter.getPrefix(ConfigConstant.rulePath);
            for (KeyValue keyValue : keyValues) {
                ruleChange(keyValue);
            }
        }
    } catch (StatusRuntimeException ex) {
        logger.error(ETCD_DOWN);
    }
}
```

每隔 1 分钟拉取一次 etcd 地址为 /jd/rules/ 的规则变化，如果 worker 所服务的 app 或者 default 的 rule 有变化，则更新规则的缓存，并清空该 appName 所对应的本地 key 缓存

2）定时任务 2:EtcdStarter#uploadClientCount ()

```

    @Scheduled(fixedRate = 10000)
    public void uploadClientCount() {
        try {
            String ip = IpUtils.getIp();
            for (AppInfo appInfo : ClientInfoHolder.apps) {
                String appName = appInfo.getAppName();
                int count = appInfo.size();
                
                
                configCenter.putAndGrant(ConfigConstant.clientCountPath + appName + "/" + ip, count + "", 13);
            }
            configCenter.putAndGrant(ConfigConstant.caffeineSizePath + ip, FastJsonUtils.convertObjectToJSON(CaffeineCacheHolder.getSize()), 13);
            
            String totalCount = FastJsonUtils.convertObjectToJSON(new TotalCount(HotKeyFilter.totalReceiveKeyCount.get(), totalDealCount.longValue()));
            configCenter.putAndGrant(ConfigConstant.totalReceiveKeyCount + ip, totalCount, 13);
            logger.info(totalCount + " expireCount:" + expireTotalCount + " offerCount:" + totalOfferCount);
            
            if (openMonitor) {
                checkReceiveKeyCount();
            }

        } catch (Exception ex) {
            logger.error(ETCD_DOWN);
        }
    }
```

-   每个 10s 将 worker 计算存储的 client 信息上报给 etcd，来方便 dashboard 来查询展示，比如 /jd/count/ 对应 client 数量，/jd/caffeineSize/ 对应 caffeine 缓存的大小，/jd/totalKeyCount/ 对应该 worker 接收的 key 总量和处理的 key 总量
-   可以从代码中看到，上面所有 etcd 的节点租期时间都是 13s，而该定时任务是每 10s 执行一次，意味着如果 full gc 或者说上报给 etcd 的时间超过 3s，则在 dashboard 查询不到 client 的相关汇算信息
-   长时间不收到 key，判断网络状态不好，断开 worker 给 etcd 地址为 /jd/workers/+$workerPath 节点的续租，因为 client 会循环判断该地址的节点是否变化，使得 client 重新连接 worker 或者断开失联的 worker

3）定时任务 3:EtcdStarter#fetchDashboardIp ()

```

@Scheduled(fixedRate = 30000)
public void fetchDashboardIp() {
    try {
        
        List<KeyValue> keyValues = configCenter.getPrefix(ConfigConstant.dashboardPath);
        
        if (CollectionUtil.isEmpty(keyValues)) {
            logger.warn("very important warn !!! Dashboard ip is null!!!");
            return;
        }
        String dashboardIp = keyValues.get(0).getValue().toStringUtf8();
        NettyClient.getInstance().connect(dashboardIp);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

每隔 30s 拉取一次 etcd 前缀为 /jd/dashboard/ 的 dashboard 连接 ip 的值，并且判断 DashboardHolder.hasConnected 里面是否为未连接状态，如果是则重新连接 worker 与 dashboard 的 netty 通道

3\. 自建的生产者消费者模型 (KeyProducer，KeyConsumer)

一般生产者消费者模型包含三大元素：生产者、消费者、消息存储队列  
这里消息存储队列是 DispatcherConfig 里面的 QUEUE，使用 LinkedBlockingQueue，默认大小为 200W

1）KeyProducer

```
@Component
public class KeyProducer {
    public void push(HotKeyModel model, long now) {
        if (model == null || model.getKey() == null) {
            return;
        }
        
        if (now - model.getCreateTime() > InitConstant.timeOut) {
            expireTotalCount.increment();
            return;
        }
        try {
            QUEUE.put(model);
            totalOfferCount.increment();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

判断接收到的 HotKeyModel 是否超出”netty.timeOut” 配置的时间，如果是将 expireTotalCount 纪录过期总数给自增，然后返回

2）KeyConsumer

```
public class KeyConsumer {
    private IKeyListener iKeyListener;
    public void setKeyListener(IKeyListener iKeyListener) {
        this.iKeyListener = iKeyListener;
    }
    public void beginConsume() {
        while (true) {
            try {
                
                HotKeyModel model = QUEUE.take();
                if (model.isRemove()) {
                    iKeyListener.removeKey(model, KeyEventOriginal.CLIENT);
                } else {
                    iKeyListener.newKey(model, KeyEventOriginal.CLIENT);
                }
                
                totalDealCount.increment();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
@Override
public void removeKey(HotKeyModel hotKeyModel, KeyEventOriginal original) {
   
   String key = buildKey(hotKeyModel);
   hotCache.invalidate(key);
   CaffeineCacheHolder.getCache(hotKeyModel.getAppName()).invalidate(key);
   
   hotKeyModel.setCreateTime(SystemClock.now());
   logger.info(DELETE_KEY_EVENT + hotKeyModel.getKey());
   for (IPusher pusher : iPushers) {
       
       pusher.remove(hotKeyModel);
   }
}
    @Override
    public void newKey(HotKeyModel hotKeyModel, KeyEventOriginal original) {
        
        String key = buildKey(hotKeyModel);
        
        
        
        Object o = hotCache.getIfPresent(key);
        if (o != null) {
            return;
        }

        
        
        
        
        
        
        
        

        
        

        
        SlidingWindow slidingWindow = checkWindow(hotKeyModel, key);
        
        boolean hot = slidingWindow.addCount(hotKeyModel.getCount());

        if (!hot) {
            
            CaffeineCacheHolder.getCache(hotKeyModel.getAppName()).put(key, slidingWindow);
        } else {
            
            
            
            hotCache.put(key, 1);

            
            
            CaffeineCacheHolder.getCache(hotKeyModel.getAppName()).invalidate(key);

            
            hotKeyModel.setCreateTime(SystemClock.now());

            
            if (EtcdStarter.LOGGER_ON) {
                logger.info(NEW_KEY_EVENT + hotKeyModel.getKey());
            }

            
            for (IPusher pusher : iPushers) {
                pusher.push(hotKeyModel);
            }

        }

    }
```

“thread.count” 配置即为消费者个数，多个消费者共同消费一个 QUEUE 队列  
生产者消费者模型，本质上还是拉模式，之所以不使用 EventBus，是因为需要队列来做缓冲  
根据 HotKeyModel 里面是否是删除消息类型

-   删除消息类型
-   根据 HotKeyModel 里面的 appName+keyType+key 的名字，来构建 caffeine 里面的 newkey，该 newkey 在 caffeine 里面主要是用来与 slidingWindow 滑动时间窗对应
-   删除 hotCache 里面 newkey 的缓存，放入的缓存 kv 分别是 newKey 和 1，hotCache 作用是用来存储该生成的热 key，hotCache 对应的 caffeine 有效期为 5s，也就是说该 key 会保存 5s，在 5s 内不重复处理相同的 hotKey。毕竟 hotKey 都是瞬时流量，可以避免在这 5s 内重复推送给 client 和 dashboard，避免无效的网络开销
-   删除 CaffeineCacheHolder 里面对应 appName 的 caffeine 里面的 newKey，这里面存储的是 slidingWindow 滑动窗口
-   推送给该 HotKeyModel 对应的所有 client 实例，用来让 client 删除该 HotKeyModel
-   非删除消息类型
-   根据 HotKeyModel 里面的 appName+keyType+key 的名字，来构建 caffeine 里面的 newkey，该 newkey 在 caffeine 里面主要是用来与 slidingWindow 滑动时间窗对应
-   通过 hotCache 来判断该 newkey 是否刚热不久，如果是则返回
-   根据滑动时间窗口来计算判断该 key 是否为 hotKey (这里可以学习一下滑动时间窗口的设计)，并返回或者生成该 newKey 对应的滑动窗口
-   如果没有达到热 key 的标准
-   通过 CaffeineCacheHolder 重新 put，cache 会自动刷新过期时间
-   如果达到了热 key 标准
-   向 hotCache 里面增加 newkey 对应的缓存，value 为 1 表示刚为热 key。
-   删除 CaffeineCacheHolder 里面对应 newkey 的滑动窗口缓存。
-   向该 hotKeyModel 对应的 app 的 client 推送 netty 消息，表示新产生 hotKey，使得 client 本地缓存，但是推送的 netty 消息只代表为热 key，client 本地缓存不会存储 key 对应的 value 值，需要调用 JdHotKeyStore 里面的 api 来给本地缓存的 value 赋值
-   向 dashboard 推送 hotKeyModel，表示新产生 hotKey

3）计算热 key 滑动窗口的设计  
限于篇幅的原因，这里就不细谈了，直接贴出项目作者对其写的说明文章：Java 简单实现滑动窗口

#### 3.3.4 dashboard 端

这个没啥可说的了，就是连接 etcd、mysql，增删改查，不过京东的前端框架很方便，直接返回 list 就可以成列表。

## 4 总结

文章第二部分为大家讲解了 redis 数据倾斜的原因以及应对方案，并对热点问题进行了深入，从发现热 key 到解决热 key 的两个关键问题的总结。

文章第三部分是热 key 问题解决方案 ——JD 开源 hotkey 的源码解析，分别从 client 端、worker 端、dashboard 端来进行全方位讲解，包括其设计、使用及相关原理。

希望通过这篇文章，能够使大家不仅学习到相关方法论，也能明白其方法论具体的落地方案，一起学习，一起成长。

___

作者：李鹏