### 

前言

在我 2019 年写下 [How Kubernetes Wins](http://mp.weixin.qq.com/s?__biz=Mzg4MDEwOTcxMw==&mid=2247483659&idx=1&sn=c2daaa7f207d8f613d811751f1930e13&chksm=cf7b70f5f80cf9e37a1569958aa6d11cc0087329fa4a8109a2595550b459911512d89229fb73&scene=21#wechat_redirect) 的时候没想到这快它就统一了整个江湖，现在不管哪种团队都是以上 K8s 为终极目标，公有云上的托管型 K8s 也卖得越来越好。我换了工作后也在围绕 K8s 做生态建设，成为了一个 Yaml boy。最近和同事在聊 K8s 的设计和实现以及历史的一些八卦，某种意义来说我确实觉得从项目的角度 K8s is fucked。

### 

缘起

我之所以这么觉得是因为我听到了一个故事，这个故事发生在某个奇怪行业的公司，其运维团队还停留在执行 nohup java -jar xxx & 的阶段。某天这个公司空降了一个新的老板，他从成本的角度要求全盘上 K8s，并从外部凑了几个还算会 K8s 的工程师开始了「容器化」的进程。

在执行的过程中，这几个会 K8s 的首先就要打起来了。A 说 Helm 随便搞搞得了，B 说不行 Kubevela 才是未来，C 说我是原教旨 Kubectl，颇有宗教辩经即视感，你是逊尼派我是什叶派他是苏菲派。但由于运维线连容器化进程都没搞过，自然其 Leader 啥都不懂，没法判断到底走哪条路，只能让他们自己 align。好不容易 align 好后，在细节方面又开始扯皮，配置项放哪里，operator 要搞几个颗粒度怎样，监控数据 label 怎么打等等等。不过扯皮归扯皮，得益于 K8s 条条大路通罗马的设计，最终这个积木还是搭建了起来。就结果来说成本自然是省不了成本的，毕竟公有云上的托管型的 K8s 除了收服务器费用外还得收托管费。维护嘛对之前的运维心智负担太重了，他们也不愿意干，学是肯定学不会的，改个配置 value 都得 ABC 来。只要 ABC 还在那肯定是能维护的，ABC 一休假在服务器上放个乖乖又不是不能用。

好了到这里这项目也算凑合跑起来了，在外部来看那确实很时尚，业界前沿，公有云 K8s 全家桶，Operator 「全自动运维」等。但天下无不散宴席，这 ABC 终究还是离职了。新加入的人呢，倒也有会 K8s 的，不过人家就觉得你这 ABC 搞的什么鬼，让我这天降伟人给你重构一下吧。怎么重构的我不清楚，反正最后肯定是更复杂了，因为跟我讲故事的这个人就是 ABC 中的一个。他说到后来随着 K8s 的懂王们来来走走这项目在老板眼中已经处于失控边缘，成本扯不清，架构很复杂，很多组件还得找以前离职员工问一下才知道是做什么的。终于有一天老板开始召集下面的 Leader 讨论是否换回 VM，没想到大家都还挺支持，一对比确实也是，懂这个的更多，知识积累更丰富，成本还低一些。

最后换没换回去我不知道，但我知道的是从时间成本上来说，这段折腾的时间算是打水漂了。

**Why Fucked**

后来我跟故事亲历者喝酒的时候，我就说出了我常说的「XX 就是傻逼，不要给他们选择」。本来就是嘛，做工程选择越少鲁棒性自然也就越好。他问我为啥这么想的，我就跟他说 KISS 原则和 BDFL 没想到他竟然不知道。我只能说新时代的工程师啊，你们这个姿势水平还是低了点。

不可否认的是 K8s 确实赢了，我也不会打自己的脸撒，现代 K8s 的影响力远胜于当年的 Openstack，但项目管理上我觉得是真的不太行。当然这也不是 K8s 的问题，一个开源项目要成功要影响力大，必然大家都要凑份子使得项目包含各种「最佳实践」，这是相辅相成的。不过这种包罗万象的实现，一定违背 KISS 原则。这时候就需要 BDFL 来整体控制项目的实现和决定长期设计了。

因此在其他开源项目比如 Python 或者内核经常能看到 Guido 和 Torvalds 口吐芬芳，可惜的是在 K8s 这块就比较迷了。虽然说是 Google 的亲儿子，但似乎这个爹也没怎么强势的管一管。好处嘛，就是对于工程团队来说条条大路通罗马，人人都能完成 KPI，一个 Operator 不够分 Credit 搞 10 个总够分了。坏处嘛，你看搞个 VM 都能搞出两条路 Kubevirt 和 Virtlet ，nginx-ingress 和 ingress-nginx 是两个不同的 controller。人力足够人人都是懂王的时候，确实可以基于 K8s 把基建像搭积木一样玩出花，比如某宇宙条。一旦你的团队不那么行，那就尴尬了。这车不上把被人喷老套，这车上了吧管理成本时间成本心智负担都是头疼的事。虽然总说着小孩子才做选择，大人我全都要，但你也得看看你有没有全要的本钱没毛病吧。

我个人来说如果从 Leader 到下面执行层工程师对 K8s 都有深入理解的话，定死一个实现方向去落地 K8s 才会比较实际。Leader 不懂那就是百花齐放百家争鸣，轮子挤破头。运维再差一点就更惨，所有维护修改更新的职责落到几个懂王身上，这几个人一跑路项目就完犊子。

像上面这个故事中，老板定了个 O，下面执行层的 Leader 没办法做技术路径判断，来了一个会 Kubevela 就上 Kubevela，明天再来个会 Helm 的又觉得似乎 Helm 也行。得亏 K8s 条条大路通罗马，都能用都能跑，但这复杂度就上去了，带来了更多的问题，以至于现在讨论要不要回归 VM。如果你是老板，看到一地鸡毛，你会怎么想？

我接触的团队多了后，只能说像故事里面的那种团队才是大多数。别说 K8s，就连简单的容器化都没搞过，Dockerfile 不会写，进程管理靠 Kill -9。看着 CI/CD 自动打镜像 tag 和 SCM 联动，用 Harbor 存 Charts 觉得这他妈是什么魔法。好了你现在一空降点名就要 K8s 全套大保健，最后看了个猴王 72 变，你是不是觉得这叫 K8s 的项目是不是有毛病？

**尾声**

我在看[金山世游的HashiStack 实践](http://mp.weixin.qq.com/s?__biz=MzI3Mzg4NTAxMw==&mid=2247486735&idx=1&sn=18cba365ba7ccfee68f54c60d3dd96f2&chksm=eb1d3f35dc6ab623376f5d529318972a10c816fcd6a1e978beb8613b2452cb746a1ef4cd5e46&scene=21#wechat_redirect)一文时候就很认可他们说的「可以在不用将应用容器化的前提下就引入一个现代化的任务调度平台」，基于心智负担的出发点是一个很理性的选择，Nomad 本身就很简单，只有一条路通北京。另外多提一句 Nomad 确实是一个被低估的产品。K8s 好不好当然好，条条大路通罗马，这对团队实在是太有用了，人人都有饭恰。但项目本身没有 BDFL，比较发散。低情商的说法就是整个项目心智负担太高，高情商的说法就是站在巨人肩膀上不过就是这巨人有点多。  

所以单纯从项目管理角度来看，K8s is fucked。如果项目本身还是现在这样广纳百川，最后大概率还是一个历史的轮回吧，就像它没有血缘的前辈 Openstack，

又不是不能用。