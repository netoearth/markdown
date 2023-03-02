![图片](https://mmbiz.qpic.cn/mmbiz_gif/Pn4Sm0RsAuiaPn8q2TmsZwA4RUjpMdPTZ3RKT9f7INA6jnnN4rn5QLB05fLOgYqaqjpmeBZo2FsxahGl5yiaP2ZA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

摘要：如果你喜欢性能工程，以及剥离层层抽象深入探究底层子系统，那么你一定很喜欢本文。

链接：https://about.gitlab.com/blog/2022/11/28/how-we-diagnosed-and-resolved-redis-latency-spikes/?continueFlag=942986d1d503b78fd935ad0b88d007cb

声明：本文为 CSDN 翻译，未经允许禁止转载。

作者 | Matt Smiley       

译者 | 弯月   责编 | 郑丽媛  

出品 | CSDN（ID：CSDNnews）

本文的背景是一个 Redis 慢性延迟的问题，在本文中，我们将使用 BPF 和分析工具，结合标准指标来揭示系统幕后的一些鲜为人知的行为。

除了工具和技术之外，我们还将使用迭代假设检验方法来构建系统动力行为模型，可以通过这个模型了解哪些因素会影响问题的严重性和触发条件。

最终，我们找到了根本原因，相应的补救措施虽然有效，但没什么新意。我们发现了一个包含三个阶段的环路，它有两个不同的饱和点，还找到了一个简单的修复方法来打破这个环路。在此过程中，我们使用了一系列技术来检查系统行为的各个方面，包括栈采样剖析、热图和火焰图、实验性的微调、源代码分析和二进制分析、指令级 BPF 检测，以及在特定进入和退出条件下的定向延迟注入。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Pn4Sm0RsAujkaH8wQRdgOQSiaDia6vLe2LOyFKFHgKNMJib5d85FSzIZjyuI5UmUKqG0M4OS6OxLoiaD0c0zJAsqgA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qkyxEibG4Uy3ibtP06LgbDfGUObnDFMEz9icU556dwCZyNgtwXH29EwH6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **问题介绍：慢性延迟**

GitLab 使用了大量 Redis，我们甚至为特定功能建立了单独的 Redis 集群。本文介绍的 Redis 实例的用途是 LRU 缓冲。

这个缓存有慢性延迟的问题，两年多前开始间歇性地发生，最近几个月越来越糟，每隔几分钟，就会出现突发性的超高延迟，相应的吞吐量也会下降，导致 SLO（Service Level Objective，服务水平目标）恶化。这些延迟峰值影响了面向用户的响应时间，并耗费了大量相关功能的错误预算，所以我们要设法解决这个问题。

图：Redis 请求的速率峰值（仅包含响应速度超过1秒的请求），每个峰值对应一次驱逐突发

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NARU4VOVKaibUcpQPPtGVkByrGlfDcHCzoOZCLLHUqic8V3ucyDJZK515Cg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在之前的工作中，我们已经完成了多项优化。之后，情况有所好转，并持续了一段时间，但后来延迟增长再次浮出水面，成为了一个重要的长期扩展问题。我们还排除了外部触发的可能性，例如请求泛滥、连接速率峰值、主机资源竞争等。这些延迟峰值是由于内存使用量达到驱逐阈值（maxmemory）造成的，与客户端流量的变化模式或其他与 Redis 竞争 CPU 时间、内存带宽或网络 I/O 的进程无关。  

最初，我们以为 Redis 6.2 新推出的驱逐节流机制可以降低我们的驱逐突发开销。结果却发现没有任何帮助，不过该机制解决了另一个问题：防止由 performEvictions 调用运行时间过长导致的停顿。相比之下，在分析过程中，我们发现我们的问题（无论是 Redis 升级之前还是之后）与大量调用导致 Redis 吞吐量降低有关，而不是因为一些调用过慢导致 Redis 完全停止。  

为了找出瓶颈以及潜在的解决方案，我们需要调查 Redis 工作负载驱逐爆发期间的行为。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qOfecjttGpW3J3tvO19FFTIAqx9ibBwTIZv6fzIvOlLJUicfaicjQJpZ1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **Redis 驱逐的一些背景知识**

当时，缓存的订阅数量超过了预期，导致试图保存的缓存键数量超过了 maxmemory 设置的阈值，因此发生 LRU 缓存驱逐并不意外，但这种驱逐额外开销的密集程度还是令人不安。

Redis 本质上是单线程的。除了少数例外，“主”线程会连续执行所有任务，包括处理客户端请求和驱逐等。在任务 X 上花费的时间过多，则意味着执行任务 Y 的时间就会减少，类似于队列的行为。

每当 Redis 达到其 maxmemory 阈值时，就会通过驱逐一些键来释放内存，直到恢复至 maxmemory 以下。然而，与预期相反，内存使用率与逐驱率的指标（如下所示）表明，驱逐率并不是连续或稳定的，而是会突然爆发，并释放比预期更多的内存。每次驱逐爆发后，直到内存使用率再次攀升至 maxmemory 阈值，才会再开始驱逐。

图：Redis 内存使用量在每次驱逐突发期间下降 300～500 MB

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NARBYZbIob5dvFqLXQfqqrvaypkevqvwwTpZldq9JyiaichhaAL1UxMp5zA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图：键的驱逐峰值与上面显示的内存使用下降的时间和大小相一致

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NARjibnSpxTq4mGxwk2CqbStbo0K2ZPIxbIwAbaV26j7ibkykHa6Twmeuaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

为什么会发生这种过量的驱逐？这一直是核心的谜团。最初，我们以为找出这个问题的原因，就能知道怎样才能平滑驱逐率、分散开销并避免延迟峰值。结果，我们发现这些爆发是需要避免的交互作用，我们后面会详细介绍。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qgxOukGzL6akS7ibtpEgg5xkibkgpJxVwylfcDicBABiaXlgBLsd1Afds1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**驱逐突发导致 CPU 饱和**  

如上所示，我们发现这些延迟峰值完全是由缓存驱逐率的峰值引发的，但我们仍然不明白为什么驱逐会集中在几秒内持续发生，而且每隔几分钟发生一次。

作为第一步，我们需要验证驱逐突发与延迟峰值之间的因果关系。

为了对此进行测试，我们使用 perf 在 Redis 主线程上运行 CPU 采样剖析。然后对剖析结果进行过滤，找出调用 performEvictions 函数时的样本。我们使用 flamescope 将剖析的 CPU 使用情况绘制成了亚秒级的偏移热图，其中 X 轴上每个柱体表示一秒，分布在 Y 轴上的格子中，每个格子表示 20 毫秒。这种可视化风格可以突出显示亚秒级的活动模式。比较这两个热图可以发现，在驱逐突发期间，CPU 几乎完全被 performEvictions 占据，主线程上的其他代码路径几乎没有占用任何 CPU 资源。

图：Redis 主线程的 CPU 占用时间，不包括 performEvictions 的调用

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NARsTUpfFv9btq1QluwFlHNPyVAOkFiceegty7Ldwibzk1ZTsQxNiajd3XAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图：同一份剖析的其余部分，仅显示 performEvictions 的调用

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NARzuV8jCia4SabSImDTxdtxO99z9QaFnO1pM9pVIXqPaGgJfOSRiczGVqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这些结果证实，驱逐突发导致主线程上的其他任务抢占不到 CPU 资源，这成为了吞吐量瓶颈，并导致 Redis 的响应时间延迟增加。这些 CPU 利用率爆发通常会持续几秒钟，由于持续时间太短，不会触发警报，但仍然会影响用户。

作为参考，下面的火焰图显示了 performEvictions 消耗 CPU 时间的详情。注意：

-   performEvictions 的调用与 processCommand（处理所有客户端请求）同步进行。
    
-   performEvictions 会开始自己执行删除。虽然从名称来看，函数 dbAsyncDelete 是异步删除，但它仅在特定条件下将删除委托给辅助线程，而这种情况对于此工作负载来说很少见。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NARgAQflOPx2v9hCicroa3xv5L0OVrCI4pibO8FzS2r2frFbDGJJhX1GBZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qA3wIF4M1rgJPLkc2t7tiatN4Txs9MPGEdYzyfH069BSHiatSpOjibBKlg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**performEvictions 的单次调用速度**  

Redis 的每个传入请求都通过调用 processCommand 来处理，并且结束时总是会调用 performEvictions 函数。performEvictions 的调用通常是空操作，在检查未超过 maxmemory 阈值后立即返回。但是，如果超过阈值，它就会持续驱逐缓存键，直到达到 mem\_tofree 目标值或超过每次调用的时间限制。

前面显示的 CPU 热图证明， performEvictions 调用消耗了大部分 CPU 时间，最多长达几秒钟。

作为补充，我们还测量了单词调用的时钟时间。

我们使用 funclatency 命令行工具（BPF 工具 BCC 套件的一部分），通过检测 performEvictions 函数的进入和退出来测量调用持续时间，并将这些测量值以 1 秒为间隔聚合到直方图中。在没有发生驱逐时，调用的延迟很低（每次调用 4～7 毫秒）。这是上面介绍的空操作的情况（包括每次调用2.5毫秒的检测开销）。但在驱逐爆发期间，结果转变为双峰分布，包括空操作调用（速度非常快）与主动执行驱逐（非常慢）的调用：

```
$ sudo funclatency-bpfcc --microseconds --timestamp --interval 1 --duration 600 --pid $( pgrep -o redis-server ) '/opt/gitlab/embedded/bin/redis-server:performEvictions'
```

此次测量还确认并量化了每秒处理的 Redis 请求的吞吐量下降：performEvictions（以及 processCommand）的调用率从驱逐开始前下降到其正常值的 20%，从每秒 2.5 万次调用下降到5 千次调用。  

这对客户端产生了巨大影响：新请求的到达速度是完成速度的 5 倍。最重要的是，我们很快就会看到这种不对称是导致驱逐爆发的原因。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qZDicaoukpArCbp8E1Z8kCR4PoiazstRK6YZq6iaon8P2yaG3tqgjBNt9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **实验：调优能否缓解驱逐导致的 CPU 饱和？**

到目前为止的分析表明，驱逐操作消耗了大量 Redis 主线程的 CPU 时间。但我们还有一些重要的问题没有得到解决，但这些信息足够我们开展一些实验来测试潜在的缓解措施：

-   我们能否分散驱逐开销，使其花费更长的时间到达目标值，并缩减占用的主线程时间？
    
-   lazyfree 机制计划了许多键的异步删除操作，这是否会导致释放的内存超过预期？Lazyfree 是一项可选功能，允许 Redis 主线程将开销较大的任务委托给异步辅助线程，比如删除超过 64 个元素的键。这些异步驱逐操作不会被立即计入驱逐循环的内存目标，因此如果有很多键符合 lazyfree 的条件，就有可能在驱逐循环内进行过多次迭代。
    

然而，这两个方法都行不通：

-   将 maxmemory-eviction-tenacity 降低到最小设定仍然没能将 performEvictions 的开销降到足以避免请求累积。它确实提高了响应率，但新请求的到达率仍然远远超过了响应率，因此这不是一种有效的缓解措施。
    
-   禁用 lazyfree-lazy-eviction 并不能阻止驱逐突发时释放的内存量远远超过 maxmemory。这些 lazyfrees 只包含一小部分内存回收。这排除了导致内存释放过多的可能原因之一。
    

在排除了两种潜在的缓解措施以及一项假设之后，我们回到了核心问题：为什么在每次驱逐爆发结束时都会额外释放数百兆字节的内存？

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qVYsK9rW3PnGicfctGFTuTNMT7ZAnZriaDjnwZnQbs6fPlWEkhHFoOsfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **为什么会出现驱逐突发并释放过多内存？**

每轮驱逐的目的是释放勉强够用的内存，恢复到 maxmemory 阈值以下。

随着内存分配的需求稳定，驱逐率同样应该趋于稳定。写入缓存的速率看起来确实很稳定。那么，为什么驱逐会密集爆发，而不是平滑地发生？为什么内存使用量突然减少了数百兆字节，而不是数百字节？

我们需要探索一些可能性：

-   驱逐是否仅在大型键被逐出时结束，自发地释放足够的内存，然后停止驱逐一段时间？不，内存使用量下降远大于数据集中最大的键。
    
-   延迟的 lazyfree 驱逐是否会导致驱逐循环超出其目标，释放比预期更多的内存？不，根据上述实验，这个假设不成立。
    
-   是否有什么原因导致驱逐循环有时计算出的 mem\_tofree 目标是一个超大值？这一点，我们接下来继续探索。答案是否定的，但这给我们带来了新的见解。
    
-   是否是因为反馈回路导致驱逐以某种方式自我放大？如果真是这样，这种状态发生和停止的条件呢？事实证明这是正确的。
    

这些都是合理且可检验的假设，每个假设都指向解决延迟问题的不同方案。我们已经排除了前两个假设。

为了测试后两个，我们构建了自定义 BPF 工具，在每次调用 performEvictions 开始时检查 mem\_tofree 的计算。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1q4xOcqPNS5Z6kCXdn3fesF3hsl5Zxgntcw5ImO64d58B91WguHpZ1yg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **使用 bpftrace 观察 mem\_tofree 的计算**

我个人最喜欢此次调查的这一部分，这项实验让我们对问题的性质有了新的认识。

如上所述，我们剩下的两个假设是：

-   mem\_tofree 的目标是一个超大值
    
-   自我放大反馈回路
    

为了甄别二者，我们使用 bpftrace 来检测 mem\_tofree 的计算，检查其输入变量和结果。

这组测量检查的是以下内容：

-   每次调用 performEvictions 是否真的是为了释放少量内存，大约为每个缓存条目的平均大小？如果 mem\_tofree 接近数百兆字节，那将证实第一个假设成立，而且还可以揭示哪部分的计算产生了这么大的一个值。否则，就会排除第一个假设，说明反馈回路的假设更有可能发生。
    
-   复制缓冲区的大小是否会对反馈机制的 mem\_tofree 产生很大影响？每次驱逐都会添加到这个缓冲区中，就像正常写入一样。如果这个缓冲区变大（可能部分是由于驱逐），然后突然缩小，就会导致内存使用量自动大幅下降，同时驱逐结束并造成内存使用量即刻减少。这是驱逐推动反馈循环的一种潜在方式。
    

为了检查 mem\_tofree 的计算（脚本），我们需要单独取出 performEvictions 调用函数 getMaxmemoryState 的代码，并进行逆向工程，找到正确的指令，并检查每个源代码级的变量。根据这些数据，我们生成了以下变量的直方图：

```
mem_reported = zmalloc_used_memory() // All used memory tracked by jemalloc
```

注意：我们自定义的 BPF 检测只能用于 redis-server 的这个构建，因为检测会附加到特定的虚拟地址上，而不同的Redis构建中这些地址并不一定相同。但这个方法能够通用化：利用 BPF 在函数调用过程中检查源代码变量，而无需重构二进制文件。因为我们查看的是函数的中间状态，并且因为编译器内联了这个函数调用，所以我们需要通过二进制分析找到正确的检测点。通常，查看函数的参数或返回值更容易且更易于移植，但在这种情况下这样做并不能满足我们的要求。  

结果：

-   排除第一个假设：每次调用 performEvictions 都会产生一个小目标值 (mem\_tofree < 2 MB)。这意味着，每次调用 performEvictions 的开销都很小。Redis 内存使用率快速下降的秘密不可能是由异常大的 mem\_tofree 目标值一次性驱逐一大批键造成的。相反，肯定有许多调用共同导致内存使用量降低。
    
-   复制输出缓冲区始终很小，排除了潜在的反馈循环机制之一。
    
-   令人惊讶的是，mem\_tofree 的大小通常为 16 KB～64 KB，大于常见的缓存条目。这种大小差异表明，缓存键并不是内存压力的主要来源，一旦开始驱逐爆发就会永久存在。
    

上述所有结果都符合反馈回路的假设。

除了回答最初的问题之外，我们还得到了一个额外的结果，同时测量 mem\_tofree 和 mem\_used 时我们还发现了一个重要情况：内存回收是一个完全不同于驱逐爆发的阶段。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAugJkibeIkBwI7feN33tcTm1qqAMiagSzJ8icbpo8M4jok0ficoITFTCLluKBDbnFaZgXbKibpSicjbfJxRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **三阶段回路**

上述结果表明，驱逐和内存回收之间存在完全独立，现在我们可以简单地绘制出由驱逐引发的延迟峰值循环的三个阶段。

图：比较每个阶段内存和 CPU 的使用率与请求率和响应率

![图片](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuhxxymJERzzcJiatQZ0Y1NAR1X3ibOB53OF38skvaOESQ7ugO7WricdmggxeTL2rHFI8T5p8baTianIaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

第 1 阶段：不饱和（7～15 分钟）  

-   内存使用量低于maxmemory。此阶段不会发生驱逐。
    
-   内存使用量有机增长，直到达到 maxmemory，进入下一阶段。
    

阶段 2：内存和 CPU 饱和（6～8 秒）

-   内存使用量达到 maxmemory，驱逐开始。
    
-   驱逐只发生在这个阶段，而且是间歇性地、频繁发生。
    
-   内存的需求常常超过可用容量，反复将内存使用推到 maxmemory 以上。在这个阶段，内存使用量在 maxmemory 阈值线上来回振荡，一次驱逐少量内存，刚好回到 maxmemory 阈值以下。
    

阶段 3：快速回收内存（30～60 秒）

-   此阶段不会发生驱逐。
    
-   在这个阶段，一直持有大量内存的进程开始快速稳定地释放内存。
    
-   没有运行驱逐的开销，CPU 时间再次回到请求处理上（第2个阶段积压的请求）。
    
-   内存使用量快速稳定地下降。到此阶段结束时，已释放数百兆字节。接下来，循环回到阶段1，重新开始。
    

从第2个阶段向第3个阶段过渡时，驱逐突然结束，因为内存使用量保持在 maxmemory 阈值以下。

在过渡的某个时间点，内存压力突然降低，这表明，第2个阶段消耗内存的某个因素开始释放内存，且释放速度超过了消耗速度，从而降低了前一阶段占用的内存空间。

这个神秘的内存消费者在第2个阶段时的需求不断膨胀，到了第3个阶段却开始释放内存，它究竟是谁？

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## **谜底揭晓**

阶段转换建模为我们提供了一些假设必须满足的约束。这个神秘的内存消费者必须满足以下条件：

-   在驱逐爆发触发的条件下，在不到10秒的时间内（第2个阶段的持续时间），内存的使用量迅速膨胀到数百兆字节。
    
-   在驱逐爆发触发的条件下，在短短几十秒的时间内（第3个阶段的持续时间），快速释放内存。
    

答案：客户端输入/输出缓冲区满足这些约束，它就是这个神秘的内存消费者。

以下是我们假设的整个经过：

-   在第1个阶段，Redis 主线程的 CPU 使用率已很高。进入第2个阶段，驱逐开始，驱逐开销导致主线程的 CPU 容量饱和，响应速度迅速下降，且低于请求的传入速度。
    
-   请求的到达速度与响应之间的吞吐量不匹配本身就是导致驱逐突发的放大器。随着二者的差距扩大，驱逐占用的时间比例也会增加。
    
-   请求积压造成内存需求增长，而积压的请求会越来越多，直到客户端停止，请求的到达率下降至与响应率匹配。随着客户端停止，请求的到达率下降，内存压力、驱逐率和 CPU 开销也随之下降。
    
-   当请求的到达率下降匹配响应率的平衡点，内存需求得到满足，并停止驱逐，第2个阶段结束。没有了驱逐的开销，更多CPU时间用于处理积压的请求，因此响应率会不断增加，直到超过请求的到达率。这个恢复阶段可以稳步消耗积压的请求，并逐渐释放内存（第3个阶段）。
    
-   在请求积压的问题得到解决后，请求的到达率和响应率会再次匹配。CPU的使用率恢复到第1个阶段的标准，内存使用量暂时下降，下降速度取决于第2个阶段积压的请求最大值。
    

我们通过延迟注入实验证实了这一假设，而且这个结果支持结论：额外的内存需求源于响应率低于请求的到达率。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## **补救措施：如何避免进入驱逐突发循环**

通过以上调查，我们搞清楚了问题的症结，下面我们来探索解决方案。

当满足以下所有条件时，Redis 的驱逐就会自我放大：

-   内存饱和：内存使用量达到 maxmemory 限制，导致驱逐开始。
    
-   CPU 饱和：Redis 主线程的正常工作负载消耗的 CPU 接近一个完整的核心，而驱逐开销将其推向饱和。这将导致响应速率降至请求的到达率以下，请求缓冲的增加导致内存需求增加，出现自我放大的效果。
    
-   许多活跃的客户端：只要请求的到达率超过响应率，饱和就会持续。客户端停止后，请求的到达率不会再增加，但如果 Redis 有许多活动客户端仍在发送请求，则饱和会持续更长时间并且影响更大。
    

可行的补救措施包括：

-   通过以下方式避免内存饱和，使内存使用量峰值低于 maxmemory 限制：
    

-   缩短缓存的存活时间（TTL）；
    
-   增加 maxmemory（并根据需要增加主机的内存，但请注意具有多个 NUMA 节点的主机上的 numa\_balancing CPU 开销）；
    
-   调整客户端行为，避免写入不必要的缓存条目；
    
-   将缓存拆分到多个实例上（分片或功能分区，有助于避免内存和 CPU 饱和）。
    

-   通过以下方式避免 CPU 饱和，使工作负载的 CPU 使用率峰值加上驱逐开销小于 1 个 CPU 内核：
    

-   使用处理单线程指令的速度最快的处理器；
    
-   将 redis-server 进程（特别是它的主线程）与任何其他竞争的 CPU 密集型进程（专用主机、任务集、cpuset）隔离开来；
    
-   调整客户端的行为，避免不必要的缓存查找或写入；
    
-   将缓存拆分到多个实例上（分片或功能分区，有助于避免内存和 CPU 饱和）；
    
-   减少 Redis 主线程的工作（io-threads、lazyfree）；
    
-   降低驱逐韧性（在我们的实验中带来的收益甚微）。
    

还有一些潜在的补救措施，比如使用 Redis 的新功能。一个思路是，不要将客户端缓冲区等临时分配的内存计算在 maxmemory 的限制之内，而是只让 maxmemory 限制键的存储。还有一种方式，我们可以限制驱逐占用主线程时间的最高比例，这样主线程的大部分时间仍然可用于处理请求，而不是用于驱逐开销。

不幸的是，这些方法在解决一个故障的同时有可能引发另一个故障，比如降低由于驱逐导致 CPU 饱和的风险，同时有可能导致进程消耗的内存增加，从而导致主机或 cgroup 饱和，并引发内存不足。两相权衡下来，孰优孰劣也未可知。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## **我们的解决方案**

我们已经优化了 CPU 的使用效率，接下来我们的注意力主要放在避免内存饱和上。

为了提高缓存的内存使用效率，我们评估了哪些类型的缓存键使用的空间最多，以及自最后一次访问以来它们累积了多少 IDLETIME。根据内存使用剖析，我们找到了一些很少使用的缓存条目（浪费空间），首先我们来调整处于空闲状态较多的键，并突出显示一些切入点对缓存进行功能分区。

我们决定同时改进多个缓存的使用效率。我们的目标是避免慢性内存饱和，主要措施包括：

-   逐步将缓存的默认存活时间从 2 周减少到 8 小时（帮助很大！）；
    
-   将某些缓存键换到客户端缓存（有效地避免非共享缓存条目占用共享缓存空间）；
    
-   将一组缓存键分区到一个单独的 Redis 实例上。
    

缩减存活时间是最简单的解决方案，但结果证明帮助很大。对于缩减存活时间，我们最担心的是缓存未命中增加，从而导致基础设施其他部分的工作量增加。有些缓存未命中的开销特别高，并且我们的指标不够精细，无法量化每种类型的缓存条目未命中时的成本。因此，我们采用迭代的方式，逐步调整存活时间，并严格监控 SLO 不达标的情况。幸运的是，我们的推断是正确的：缩减存活时间并没有显著降低缓存命中率，缓存未命中的增加也没有对下游子系统造成明显影响。

事实证明，缩减存活时间足以将内存使用量持续降低到其饱和点以下。

刚开始的时候，增加 maxmemory 并没有任何作用，因为我们预计最初的内存需求峰值（在提升效率之前）会超过我们为 Redis 投入的虚拟内存。但是，当内存需求降低到饱和以下之后，我们就可以为将来的增长情况预留空间，并重新启用饱和警报。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## **结果**

下图显示了 Redis 的内存使用摆脱了长期的饱和状态：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

观察我们调整存活时间的这段时间，可以看到驱逐引发的延迟峰值随着内存使用量降到饱和点以下而消失了：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

 ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这些驱逐引发的延迟峰值是导致 Redis 缓存异常慢的最大原因。

解决了这个缓慢的根源，用户体验显著改善。下图中1年的回顾只显示了改进的长尾部分，未能展示全部收益。每个工作日大约有 200 万个 Redis 请求的处理时间超过1秒，但在 8 月中旬我们修复这个问题后全面下降：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## **总结**

我们通过不懈的努力，终于解决了一个长期存在的延迟问题，我们在此过程中积累了很多经验。

总的来说，我们提升了多方面的效率，打破了由缓存驱逐引发的一系列周期性的问题。如今，内存需求远低于饱和点，并消除了导致开发团队预算超标以及用户经历间歇性响应延迟峰值。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## **要点总结**

下面，总结一下我们学习到的有关 Redis 驱逐行为的知识：

-   键存储和客户端连接缓冲区共享同一个内存预算（maxmemory）。客户端连接的缓冲需求激增会计算在maxmemory之内制，方式与插入键或键大小激增的计算方式相同。
    
-   Redis 的驱逐在其主线程的前台执行。performEvictions 占用的是处理客户端请求的时间，因此，在驱逐突发期间，Redis 的吞吐量上限较低。
    
-   如果驱逐开销导致主线程的 CPU 饱和，则响应率会低于请求的到达率，从而导致请求积压（这会消耗内存），而客户端也会体验到请求响应减慢。
    
-   用于保存待处理请求的内存需求增加，导致驱逐爆发，直到大量客户端停止，请求的到达率回落到响应率以下。当到达平衡点时，驱逐停止，驱逐开销消失，Redis 快速处理积压的请求，积压的请求占用的内存被释放。
    
-   触发此循环需要满足以下所有条件：
    

-   Redis 配置了 maxmemory 限制，且内存需求超过了这个限制。内存饱和导致驱逐开始。
    
-   在正常工作负载下，Redis 主线程的 CPU 使用率非常高，驱逐操作使其达到 CPU 饱和。这会导致响应率降低到请求率以下，从而导致请求积压和高延迟。
    
-   许多活动客户端连接。驱逐突发的持续时间和客户端连接缓冲区占用的内存大小与活动客户端的数量成比例增加。
    

-   避免内存或 CPU 饱和可以防止触发这种循环。在我们的这个例子中，我们通过缩减存活时间的方式，轻松地避免了内存饱和的问题。
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

```
☞Log4j 漏洞已被曝一年，曾支配 Java 界熬夜加班，如今仍四处潜伏......☞在抖音全程看世界杯，超高清直播背后的硬实力☞10 年磨一剑，阿里开源自研搜索引擎 Havenask！
```