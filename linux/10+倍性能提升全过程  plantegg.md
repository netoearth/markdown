## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#10-%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B%E2%80%93%E4%BC%98%E9%85%B7%E8%B4%A6%E5%8F%B7%E7%BB%91%E5%AE%9A%E6%B7%98%E5%AE%9D%E8%B4%A6%E5%8F%B7%E7%9A%84TPS%E4%BB%8E500%E5%88%B05400%E7%9A%84%E4%BC%98%E5%8C%96%E5%8E%86%E7%A8%8B "10+倍性能提升全过程–优酷账号绑定淘宝账号的TPS从500到5400的优化历程")10+倍性能提升全过程–优酷账号绑定淘宝账号的TPS从500到5400的优化历程

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E8%83%8C%E6%99%AF%E8%AF%B4%E6%98%8E "背景说明")背景说明

> 2016年的双11在淘宝上买买买的时候，天猫和优酷土豆一起做了联合促销，在天猫双11当天购物满XXX元就赠送优酷会员，这个过程需要用户在优酷侧绑定淘宝账号(登录优酷、提供淘宝账号，优酷调用淘宝API实现两个账号绑定）和赠送会员并让会员权益生效(看收费影片、免广告等等）
> 
> 这里涉及到优酷的两个部门：Passport(在上海，负责登录、绑定账号，下文中的优化过程主要是Passport部分）；会员(在北京，负责赠送会员，保证权益生效）
> 
> 在双11活动之前，Passport的绑定账号功能一直在运行，只是没有碰到过大促销带来的挑战

___

整个过程分为两大块：

1.  整个系统级别，包括网络和依赖服务的性能等，多从整个系统视角分析问题；
2.  但服务器内部的优化过程，将CPU从si/sy围赶us，然后在us从代码级别一举全歼。

系统级别都是最容易被忽视但是成效最明显的，代码层面都是很细致的力气活。

整个过程都是在对业务和架构不是非常了解的情况下做出的。

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E4%BC%9A%E5%91%98%E9%83%A8%E5%88%86%E7%9A%84%E6%9E%B6%E6%9E%84%E6%94%B9%E9%80%A0 "会员部分的架构改造")会员部分的架构改造

-   接入中间件DRDS，让优酷的数据库支持拆分，分解MySQL压力
-   接入中间件vipserver来支持负载均衡
-   接入集团DRC来保障数据的高可用
-   对业务进行改造支持Amazon的全链路压测

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E4%B8%BB%E8%A6%81%E7%9A%84%E5%8E%8B%E6%B5%8B%E8%BF%87%E7%A8%8B "主要的压测过程")主要的压测过程

[![screenshot.png](https://plantegg.github.io/images/oss/6b24a854d91aba4dcdbd4f0155683d93.png)](https://plantegg.github.io/images/oss/6b24a854d91aba4dcdbd4f0155683d93.png)

**上图是压测过程中主要的阶段中问题和改进,主要的问题和优化过程如下：**

```
- docker bridge网络性能问题和网络中断si不均衡    (优化后：500->1000TPS)
- 短连接导致的local port不够                   (优化后：1000-3000TPS)
- 生产环境snat单核导致的网络延时增大             (优化后生产环境能达到测试环境的3000TPS)
- Spring MVC Path带来的过高的CPU消耗           (优化后：3000->4200TPS)
- 其他业务代码的优化(比如异常、agent等)          (优化后：4200->5400TPS)
```

**优化过程中碰到的比如淘宝api调用次数限流等一些业务原因就不列出来了**

___

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E6%A6%82%E8%BF%B0 "概述")概述

由于用户进来后先要登录并且绑定账号，实际压力先到Passport部分，在这个过程中最开始单机TPS只能到500，经过N轮优化后基本能达到5400 TPS，下面主要是阐述这个优化过程

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#Passport%E9%83%A8%E5%88%86%E7%9A%84%E5%8E%8B%E5%8A%9B "Passport部分的压力")Passport部分的压力

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#Passport-%E6%A0%B8%E5%BF%83%E6%9C%8D%E5%8A%A1%E5%88%86%E4%B8%A4%E4%B8%AA%EF%BC%9A "Passport 核心服务分两个：")Passport 核心服务分两个：

-   Login 主要处理登录请求
-   userservice 处理登录后的业务逻辑，比如将优酷账号和淘宝账号绑定

为了更好地利用资源每台物理加上部署三个docker 容器，跑在不同的端口上(8081、8082、8083），通过bridge网络来互相通讯

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#Passport%E6%9C%BA%E5%99%A8%E5%A4%A7%E8%87%B4%E7%BB%93%E6%9E%84 "Passport机器大致结构")Passport机器大致结构

[![screenshot.png](https://plantegg.github.io/images/oss/b509b30218dd22e03149985cf5e15f8e.png)](https://plantegg.github.io/images/oss/b509b30218dd22e03149985cf5e15f8e.png)

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#userservice%E6%9C%8D%E5%8A%A1%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3%E7%9A%84%E5%90%84%E7%A7%8D%E9%97%AE%E9%A2%98 "userservice服务网络相关的各种问题")userservice服务网络相关的各种问题

___

#### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E5%A4%AA%E5%A4%9ASocketConnect%E5%BC%82%E5%B8%B8-%E5%A6%82%E4%B8%8A%E5%9B%BE%EF%BC%89 "太多SocketConnect异常(如上图）")太多SocketConnect异常(如上图）

在userservice机器上通过netstat也能看到大量的SYN\_SENT状态，如下图：  
[![image.png](https://plantegg.github.io/images/oss/99bf952b880f17243953da790ff0e710.png)](https://plantegg.github.io/images/oss/99bf952b880f17243953da790ff0e710.png)

#### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E5%9B%A0%E4%B8%BAdocker-bridge%E9%80%9A%E8%BF%87nat%E6%9D%A5%E5%AE%9E%E7%8E%B0%EF%BC%8C%E5%B0%9D%E8%AF%95%E5%8E%BB%E6%8E%89docker%EF%BC%8C%E8%AE%A9tomcat%E7%9B%B4%E6%8E%A5%E8%B7%91%E5%9C%A8%E7%89%A9%E7%90%86%E6%9C%BA%E4%B8%8A "因为docker bridge通过nat来实现，尝试去掉docker，让tomcat直接跑在物理机上")因为docker bridge通过nat来实现，尝试去掉docker，让tomcat直接跑在物理机上

这时SocketConnect异常不再出现  
[![image.png](https://plantegg.github.io/images/oss/6ed62fd6b50ad2785e5b57687d95ad6e.png)](https://plantegg.github.io/images/oss/6ed62fd6b50ad2785e5b57687d95ad6e.png)

#### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E4%BB%8E%E6%96%B0%E6%A2%B3%E7%90%86%E4%B8%80%E4%B8%8B%E7%BD%91%E7%BB%9C%E6%B5%81%E7%A8%8B "从新梳理一下网络流程")从新梳理一下网络流程

docker(bridge)—-短连接—>访问淘宝API(淘宝open api只能短连接访问），性能差，cpu都花在si上；

如果 docker(bridge)—-长连接到宿主机的某个代理上(比如haproxy）—–短连接—>访问淘宝API， 性能就能好一点。问题可能是短连接放大了Docker bridge网络的性能损耗

#### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E5%BD%93%E6%97%B6%E7%9C%8B%E5%88%B0%E7%9A%84cpu-si%E9%9D%9E%E5%B8%B8%E9%AB%98%EF%BC%8C%E6%88%AA%E5%9B%BE%E5%A6%82%E4%B8%8B%EF%BC%9A "当时看到的cpu si非常高，截图如下：")当时看到的cpu si非常高，截图如下：

[![image.png](https://plantegg.github.io/images/oss/4c1eff0f925f59977e2557acff5cf03b.png)](https://plantegg.github.io/images/oss/4c1eff0f925f59977e2557acff5cf03b.png)

去掉Docker后，性能有所提升，继续通过perf top看到内核态寻找可用的Local Port消耗了比较多的CPU，gif动态截图如下(可以点击看高清大图）：

[![perf-top-netLocalPort-issue.gif](https://plantegg.github.io/images/oss/fff502ca73e3112e585560ffe4a4dbf1.gif)](https://plantegg.github.io/images/oss/fff502ca73e3112e585560ffe4a4dbf1.gif)

**注意图中ipv6\_rcv\_saddr\_equal和inet\_csk\_get\_port 总共占了30%的CPU** (系统态的CPU使用率高意味着共享资源有竞争或者I/O设备之间有大量的交互。)

**一般来说一台机器默认配置的可用 Local Port 3万多个，如果是短连接的话，一个连接释放后默认需要60秒回收，30000/60 =500 这是大概的理论TPS值【这里只考虑连同一个server IP:port 的时候】**

这500的tps算是一个老中医的经验。不过有些系统调整过Local Port取值范围，比如从1024到65534，那么这个tps上限就是1000附近。

同时观察这个时候CPU的主要花在sy上，最理想肯定是希望CPU主要用在us上，截图如下：  
[![image.png](https://plantegg.github.io/images/oss/05703c168e63e96821ea9f921d83712b.png)](https://plantegg.github.io/images/oss/05703c168e63e96821ea9f921d83712b.png)

**规则：性能优化要先把CPU从SI、SY上的消耗赶到US上去(通过架构、系统配置）；然后提升 US CPU的效率(代码级别的优化）**

sy占用了30-50%的CPU，这太不科学了，同时通过 netstat 分析连接状态，确实看到很多TIME\_WAIT：  
[![localportissue-time-wait.png](https://plantegg.github.io/images/oss/2ae2cb8b0cb324b68ca22c48c019e029.png)](https://plantegg.github.io/images/oss/2ae2cb8b0cb324b68ca22c48c019e029.png)

**cpu要花在us上，这部分才是我们代码吃掉的**

**_于是让PE修改了tcp相关参数：降低 tcp\_max\_tw\_buckets和开启tcp\_tw\_reuse，这个时候TPS能从1000提升到3000_**

鼓掌，赶紧休息，迎接双11啊

[![image.png](https://plantegg.github.io/images/oss/91353fb9c88116be3ff109e3528a4651.png)](https://plantegg.github.io/images/oss/91353fb9c88116be3ff109e3528a4651.png)

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83%E4%BC%98%E5%8C%96%E5%88%B03000-TPS%E5%90%8E%E4%B8%8A%E7%BA%BF%E7%BB%A7%E7%BB%AD%E5%8E%8B%E6%B5%8B "测试环境优化到3000 TPS后上线继续压测")测试环境优化到3000 TPS后上线继续压测

**居然性能又回到了500，太沮丧了**，其实最开始账号绑定慢，Passport这边就怀疑taobao api是不是在大压力下不稳定，一般都是认为自己没问题，有问题的一定是对方。我不觉得这有什么问题，要是知道自己有什么问题不早就优化掉了，但是这里缺乏证据支撑，也就是如果你觉得自己没有问题或者问题在对方，一定要拿出证据来(有证据那么大家可以就证据来讨论，而不是互相苍白地推诿）。

这个时候Passport更加理直气壮啊，好不容易在测试环境优化到3000，怎么一调taobao api就掉到500呢，这么点压力你们就扛不住啊。 但是taobao api那边给出调用数据都是1ms以内就返回了(alimonitor监控图表–拿证据说话）。

看到alimonitor给出的api响应时间图表后，我开始怀疑从优酷的机器到淘宝的机器中间链路上有瓶颈，但是需要设计方案来证明这个问题在链路上，要不各个环节都会认为自己没有问题的，问题就会卡死。但是当时Passport的开发也只能拿到Login和Userservice这两组机器的权限，中间的负载均衡、交换机都没有权限接触到。

在没有证据的情况下，肯定机房、PE配合你排查的欲望基本是没有的(被坑过很多回啊，你说我的问题，结果几天配合排查下来发现还是你程序的问题，凭什么我要每次都陪你玩？），所以我要给出证明问题出现在网络链路上，然后拿着这个证据跟网络的同学一起排查。

讲到这里我禁不住要插一句，在出现问题的时候，都认为自己没有问题这是正常反应，毕竟程序是看不见的，好多意料之外逻辑考虑不周全也是常见的，出现问题按照自己的逻辑自查的时候还是没有跳出之前的逻辑所以发现不了问题。但是好的程序员在问题的前面会尝试用各种手段去证明问题在哪里，而不是复读机一样我的逻辑是这样的，不可能出问题的。即使目的是证明问题在对方，只要能给出明确的证据都是负责任的，拿着证据才能理直气壮地说自己没有问题和干净地甩锅。

**在尝试过tcpdump抓包、ping等各种手段分析后，设计了场景证明问题在中间链路上。**

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E8%AE%BE%E8%AE%A1%E5%A6%82%E4%B8%8B%E4%B8%89%E4%B8%AA%E5%9C%BA%E6%99%AF%E8%AF%81%E6%98%8E%E9%97%AE%E9%A2%98%E5%9C%A8%E4%B8%AD%E9%97%B4%E9%93%BE%E8%B7%AF%E4%B8%8A%EF%BC%9A "设计如下三个场景证明问题在中间链路上：")设计如下三个场景证明问题在中间链路上：

1.  压测的时候在userservice ping 依赖服务的机器；
2.  将一台userservice机器从负载均衡上拿下来(没有压力），ping 依赖服务的机器；
3.  从公网上非我们机房的机器 ping 依赖服务的机器；

这个时候奇怪的事情发现了，压力一上来**场景1、2**的两台机器ping淘宝的rt都从30ms上升到100-150ms，**场景1** 的rt上升可以理解，但是**场景2**的rt上升不应该，同时**场景3**中ping淘宝在压力测试的情况下rt一直很稳定(说明压力下淘宝的机器没有问题），到此确认问题在优酷到淘宝机房的链路上有瓶颈，而且问题在优酷机房出口扛不住这么大的压力。于是从上海Passport的团队找到北京Passport的PE团队，确认在优酷调用taobao api的出口上使用了snat，PE到snat机器上看到snat只能使用单核，而且对应的核早就100%的CPU了，因为之前一直没有这么大的压力所以这个问题一直存在只是没有被发现。

**于是PE去掉snat，再压的话 TPS稳定在3000左右**

___

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E5%88%B0%E8%BF%99%E9%87%8C%E7%BB%93%E6%9D%9F%E4%BA%86%E5%90%97%EF%BC%9F-%E4%BB%8E3000%E5%88%B05400TPS "到这里结束了吗？ 从3000到5400TPS")到这里结束了吗？ 从3000到5400TPS

优化到3000TPS的整个过程没有修改业务代码，只是通过修改系统配置、结构非常有效地把TPS提升了6倍，对于优化来说这个过程是最轻松，性价比也是非常高的。实际到这个时候也临近双11封网了，最终通过计算(机器数量\*单机TPS）完全可以抗住双11的压力，所以最终双11运行的版本就是这样的。 但是有工匠精神的工程师是不会轻易放过这么好的优化场景和环境的(基线、机器、代码、工具都具备配套好了）

**优化完环境问题后，3000TPS能把CPU US跑上去，于是再对业务代码进行优化也是可行的了**。

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E8%BF%9B%E4%B8%80%E6%AD%A5%E6%8C%96%E6%8E%98%E4%BB%A3%E7%A0%81%E4%B8%AD%E7%9A%84%E4%BC%98%E5%8C%96%E7%A9%BA%E9%97%B4 "进一步挖掘代码中的优化空间")进一步挖掘代码中的优化空间

双11前的这段封网其实是比较无聊的，于是和Passport的开发同学们一起挖掘代码中的可以优化的部分。这个过程中使用到的主要工具是这三个：火焰图、perf、perf-map-java。相关链接：[http://www.brendangregg.com/perf.html](http://www.brendangregg.com/perf.html) ; [https://github.com/jrudolph/perf-map-agent](https://github.com/jrudolph/perf-map-agent)

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E9%80%9A%E8%BF%87Perf%E5%8F%91%E7%8E%B0%E7%9A%84%E4%B8%80%E4%B8%AASpringMVC-%E7%9A%84%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98 "通过Perf发现的一个SpringMVC 的性能问题")通过Perf发现的一个SpringMVC 的性能问题

这个问题具体参考我之前发表的优化文章。 主要是通过火焰图发现spring mapping path消耗了过多CPU的性能问题，CPU热点都在methodMapping相关部分，于是修改代码去掉spring中的methodMapping解析后性能提升了40%，TPS能从3000提升到4200.

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E8%91%97%E5%90%8D%E7%9A%84fillInStackTrace%E5%AF%BC%E8%87%B4%E7%9A%84%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98 "著名的fillInStackTrace导致的性能问题")著名的fillInStackTrace导致的性能问题

代码中的第二个问题是我们程序中很多异常(fillInStackTrace），实际业务上没有这么多错误，应该是一些不重要的异常，不会影响结果，但是异常频率很高，对这种我们可以找到触发的地方，catch住，然后不要抛出去(也就是别触发fillInStackTrace)，打印一行error日志就行，这块也能省出10%的CPU，对应到TPS也有几百的提升。

[![screenshot.png](https://plantegg.github.io/images/oss/36ef4b16c3c400abf6eb7e6b0fbb2f58.png)](https://plantegg.github.io/images/oss/36ef4b16c3c400abf6eb7e6b0fbb2f58.png)

部分触发fillInStackTrace的场景和具体代码行(点击看高清大图）：  
[![screenshot.png](https://plantegg.github.io/images/oss/7eb2cbb4afc2c7d7007c35304c95342a.png)](https://plantegg.github.io/images/oss/7eb2cbb4afc2c7d7007c35304c95342a.png)

对应的火焰图(点击看高清大图）：  
[![screenshot.png](https://plantegg.github.io/images/oss/894bd736dd03060e89e3fa49cc98ae5e.png)](https://plantegg.github.io/images/oss/894bd736dd03060e89e3fa49cc98ae5e.png)

[![screenshot.png](https://plantegg.github.io/images/oss/2bb7395a2cc6833c9c7587b38402a301.png)](https://plantegg.github.io/images/oss/2bb7395a2cc6833c9c7587b38402a301.png)

### [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E8%A7%A3%E6%9E%90useragent-%E4%BB%A3%E7%A0%81%E9%83%A8%E5%88%86%E7%9A%84%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98 "解析useragent 代码部分的性能问题")解析useragent 代码部分的性能问题

整个useragent调用堆栈和cpu占用情况，做了个汇总(useragent不启用TPS能从4700提升到5400）  
[![screenshot.png](https://plantegg.github.io/images/oss/8a4a97cb74724b8baa3b90072a1914e0.png)](https://plantegg.github.io/images/oss/8a4a97cb74724b8baa3b90072a1914e0.png)

实际火焰图中比较分散：  
[![screenshot.png](https://plantegg.github.io/images/oss/afacc681a9550cd087838c2383be54c8.png)](https://plantegg.github.io/images/oss/afacc681a9550cd087838c2383be54c8.png)

**最终通过对代码的优化勉勉强强将TPS从3000提升到了5400(太不容易了，改代码过程太辛苦，不如改配置来得快）**

优化代码后压测tps可以跑到5400，截图：

[![image.png](https://plantegg.github.io/images/oss/38bb043c85c7b50007609484c7bf5698.png)](https://plantegg.github.io/images/oss/38bb043c85c7b50007609484c7bf5698.png)

## [](https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/#%E6%9C%80%E5%90%8E%E5%86%8D%E6%AC%A1%E6%80%BB%E7%BB%93%E6%95%B4%E4%B8%AA%E5%8E%8B%E6%B5%8B%E8%BF%87%E7%A8%8B%E7%9A%84%E9%97%AE%E9%A2%98%E5%92%8C%E4%BC%98%E5%8C%96%E5%8E%86%E7%A8%8B "最后再次总结整个压测过程的问题和优化历程")最后再次总结整个压测过程的问题和优化历程

```
- docker bridge网络性能问题和网络中断si不均衡    (优化后：500->1000TPS)
- 短连接导致的local port不够                   (优化后：1000-3000TPS）
- 生产环境snat单核导致的网络延时增大             (优化后能达到测试环境的3000TPS）
- Spring MVC Path带来的过高的CPU消耗           (优化后：3000->4200TPS)
- 其他业务代码的优化(比如异常、agent等）         (优化后：4200->5400TPS)
```

[![image.png](https://plantegg.github.io/images/oss/2be2799d1eef982d77e5c0a5c896a0e9.png)](https://plantegg.github.io/images/oss/2be2799d1eef982d77e5c0a5c896a0e9.png)