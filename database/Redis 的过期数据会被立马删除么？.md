> ❝
> 
> 码哥，当 key 达到过期时间，Redis 就会马上删除么？还有文末有送书福利么？

先说结论（ps：文末有福利）：**并不会立马删除。****Redis 有两种删除过期数据的策略：**

-   定期选取部分数据删除；
    
-   惰性删除；
    

该命令在 Redis 2.4 版本，过期时间并不是很精确，它可能在零到一秒之间。

从 Redis 2.6 开始，过期错误为 0 到 1 毫秒。

`EXPIRE key seconds [ NX | XX | GT | LT]` 指令可以将指定的 key 设置过期时间，如果没有设置过期时间， key 将一直存在，除非我们明确将其删除，比如执行 `DEL` 指令。

所谓”狡兔死，走狗烹“，没用了就干掉，跟 35 岁就“毕业”是一个道理。

好慌……

从 Redis 版本 7.0.0 开始：`EXPIRE` 添加了选项：`NX`、`XX`和`GT`、`LT` 选项。

-   NX：当 key 没有过期时才设置过期时间；
    
-   XX：只有 key 已过期的时候才设置过期时间；
    
-   GT：仅当**新的到期时间**大于当前到期时间时才设置过期时间；
    
-   LT：仅在新到期时间小于当前到期时间才设置到过期时间。
    

## 过期与持久化

> ❝
> 
> 主从或者集群架构中，两台机器的时钟严重不同步，会有什么问题么？

key 过期信息是用 **Unix 绝对时间戳**表示的。

**为了让过期操作正常运行，机器之间的时间必须保证稳定同步，否则就会出现过期时间不准的情况。**

比如两台时钟严重不同步的机器发生 RDB 传输， slave 的时间设置为未来的 2000 秒，假如在 master 的一个 key 设置 1000 秒存活，当 Slave 加载 RDB 的时候 key 就会认为该 key 过期（因为 slave 机器时间设置为未来的 2000 s），并不会等待 1000 s 才过期。

![图片](https://mmbiz.qpic.cn/mmbiz_png/EoJib2tNvVtcOv8BrjWQFzmr4jXnQmsuGE6zVHtaev5aVQFssQibRGAS2Kw1AoSNmdICRbNbCcN6JQ6ibL3upnBbQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

机器时钟不同步导致过期混乱

## 惰性删除

惰性删除很简单，就是当有客户端的请求查询该 `key` 的时候，检查下 `key` 是否过期，如果过期，则删除该 `key`。

比如当 Redis 收到客户端的`GET movie:小泽#玛……利亚.rmvb` 请求，就会先检查 `key = movie:小泽#玛……利亚.rmvb` 是否已经过期，如果过期那就删除。

**删除过期数据的主动权交给了每次访问请求。**

该实现通过 `expireIfNeeded`函数实现，源码路径：`src/db.c`。

```
int expireIfNeeded(redisDb *db, robj *key, int force_delete_expired) {   // key 没有过期，return 0    if (!keyIsExpired(db,key)) return 0;    if (server.masterhost != NULL) {        if (server.current_client == server.master) return 0;        if (!force_delete_expired) return 1;    }    if (checkClientPauseTimeoutAndReturnIfPaused()) return 1;    /* Delete the key */    deleteExpiredKeyAndPropagate(db,key);    return 1;}
```

## 定期删除

仅仅靠客户端访问来判断 key 是否过期才执行删除肯定不够，因为有的 key 过期了，但未来再也没人访问，这些数据要怎么删除呢？

不能让这些数据「占着茅坑不拉屎」。

所谓定期删除，也就是 Redis 默认每 1 秒运行 10 次（每 100 ms 执行一次），每次随机抽取一些设置了过期时间的 key，检查是否过期，如果发现过期了就直接删除。

**注意：并不是一次运行就检查所有的库，所有的键，而是随机检查一定数量的键。**

具体步骤如下：

定时删除

1.  从所有设置了过期时间的 key 集合中随机**选择 20 个 key**；
    
2.  删除「步骤 1」发现的所有过期 key 数据；
    
3.  「步骤 2 」结束，过期的 key 超过 25%，则继续执行「步骤 1」。
    

删除的源码 `expire.c 的 activeExpireCycle 函数实现`。

这也就意味着在任何时候，过期 key 的最大数量等于每秒最大写入操作量除以 4。

> ❝
> 
> 为啥不检查所有设置过期时间的 key？

你想呀，假设 Redis 里存放了 100 w 个 key，都设置了过期时间，每隔 100 毫秒就检查 100 w 个 key，CPU 全浪费在检查过期 key 上了，Redis 也就废了。

注意了：**不管是定时删除，还是惰性删除。当数据删除后**，`master` **会生成删除的指令记录到** `AOF` 和 `slave` **节点**。

> ❝
> 
> 码哥，如果过期的数据太多，定时删除无法删除完全（每次删除完过期的 key 还是超过 25%），同时这些 key 也再也不会被客户端请求，也就是无法走惰性删除，会怎样？
> 
> 会不会导致 Redis 内存耗尽，怎么破？

这个问题问得好，答案是走**内存淘汰机制**。

今天就到这里，说太多的话，大家容易在知识的海量里呛死，保命要紧，至于内存淘汰机制详情，请看下回分解。

点击**下方**名片关注，回复**资料**获取「码哥 Redis 高手心法.pdf」  

## 送书福利

送 **4 本**《Spring Boot 企业级项目开发实战》给一直关注并支持码哥的读者。

**中奖规则：在本文「留言」、「点赞」、「在看」并将此文章****转发到朋友圈****，我从留言中随机抽取** **4** **位幸运读者，凭借朋友圈分享记录截图领取。****免费包邮到手！**

截止：2022 年 4 月 13 20:00

随着互联网的发展，越来越多的企业采用Spring Boot来完成Web项目的开发。本书专门为Spring Boot企业级项目开发者量身定制，内容涉及Spring Boot的理论基础、源码解析和各种项目开发技巧。

好文推荐  

[16 张图解 ｜ 淘宝 10年架构演进](http://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247499244&idx=1&sn=19d603ded8ee744bef63c270c2910e72&chksm=c27fbfdaf50836cc32b87ed3453c04e2c37e09f22c97a18e215cff3568427db66a30b5de733e&scene=21#wechat_redirect)  

[Redis 高手心法.pdf 发布](http://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247499199&idx=1&sn=13ed8543c424e02f7a846ed7baf7f6fe&chksm=c27fbf89f508369fbc7e429b5bb623d7623ca09e8a5e5524412f785f2e58a88fbd20296837b9&scene=21#wechat_redirect)  

[硬核 | Redis 布隆（Bloom Filter）过滤器原理与实战](http://mp.weixin.qq.com/s?__biz=MzkzMDI1NjcyOQ==&mid=2247498630&idx=1&sn=e71158946452791f7e9825e535145c71&chksm=c27fb9b0f50830a6fa6dfe31a62a2bd0a8786a5814faf57077fae8db8b3fb8622a8c7830a234&scene=21#wechat_redirect)