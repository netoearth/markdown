我们今天为了方便大家学习理解SD-WAN，特意将它和现实生活中的某一实物进行一一对应。想来想去没有什么十分恰当的比喻，但最终还是选择以公路作为一个喻体。对于通信行业中从事技术的攻城狮来说，这可能毫无意义，但是对于非技术人员那简直就是福利。如何来讲呢？我们往下看。

[![](https://img1.sdnlab.com//wp-content/uploads/2019/06/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190610174804.jpg)](https://img1.sdnlab.com//wp-content/uploads/2019/06/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190610174804.jpg)

首先，大家应该都很清楚的一个事实：我们身边的交通情况。目前全国实现了道路的互通，这形成了一张非常庞大的交通网，可以让任意两地的人通过修好的交通网相互拜访。由于交通的便利，拉近了我们每个人的距离。所以以后担心的不只是隔壁老王，还有可能担心千里之外的老李。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-1.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-1.png)

全国高速公路网

与公路对比，我们将连接所有终端设备的连接形成的网络公路称为互联网，互联网负责将所有的电脑、手机、服务器等进行互联，实现人们的互相通信。如此看来交通网和互联网的意义几乎相同，都是实现互联互通。与交通网最大的不同，在互联网中，承载并交互信息的是耗电的，而不是烧油的（电动汽车车主请自动屏蔽此文章）。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-2.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-2.png)

全国互联网示意图

那么究竟什么才是互联网？应该说即使是通信专业的学生也未必能解释的清楚。那好，既然要普及，我们就先来聊聊互联网的起源。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-3.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-3.png)

其实互联网的定义真的很难讲，要说互联网的身世，还真是奇特，它没有妈妈，却有一堆爸爸。也就是人们所称的互联网之父，蒂姆·伯纳斯·李（Tim Berners-Lee）、温顿·瑟夫(Vinton Gray “Vint” Cerf ）、罗伯特卡恩（Robert Elliot Kahn）等， 所以“互联网之父”是一个群体,而不是一个人。

我们以覆盖区域从小到大来划分互联网，最终可以分成两大类：  
1、一类是LAN（Local Area Network局域网：例如1个家庭、1个办公室内部的通信网络）；

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-4.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-4.png)

图：典型的LAN（局域网）示意图

如上图所示：1个LAN内部（同1个办公室）任意2台电脑之间可通过交换机进行通信。

2、另一类是WAN（Wide Area Network广域网：世界上任意2个以上位置的通信互连网络）；

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-5.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-5.png)

典型的WAN（广域网）示意图

如上图所示：1个WAN内部（上海、纽约）的任意2台电脑之间可以通过路由器实现通信。  
好了，我们初步了解了LAN和WAN的区别，因为今天我们重点了解SD-WAN，所以我们需要先回顾下WAN，那么WAN常用的技术协议有哪些呢？请看下面这张图：

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-6.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-6.png)

通过上图可以看出，目前主流的WAN的VPN协议主要是包括两种：

-   基于Internet的IPSEC-VPN；
-   基于运营商专用网络MPLS-VPN；

好，这里谈到了1个概念:VPN（虚拟专用网络），那么什么叫VPN呢？我们还是以上面的公路交通做比喻进行解释。

如开篇介绍，我们将WAN理解为道路，道路上行驶的是各种车辆，那么有个问题：上下班高峰期时间，各种车辆挤在一起，极易交通堵塞，导致公交车、救护车等专用车辆无法迅速通过。为了解决这个问题，交管部门设置了公交专用道，极大程度解决了百姓上下班拥堵的问题。如下图所示：

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-7.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-7.png)

公交专用车道设置前后的对比

WAN网络中亦是如此，我们以企业客户来举例，在WAN中传递的应用和业务就如同道路上行驶的车辆一样各种各样，有视频会议、Skype语音、OA、ERP、e-Mail、金蝶财务软件等等，如果这些数据都跑在同1条拥堵的道路上，那么势必会导致某些重要的业务和应用（如视频会议）因为数据的延时、丢包、抖动而出现图像卡顿、马赛克甚至中断的故障。那么怎么去解决这个问题呢？就需要用到VPN这个技术手段了，如下图所示：

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-8.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-8.png)

基于业务和应用在WAN通道中设置VPN

如上图所示：我们根据企业的业务和应用的重要性，用VPN技术为他们划分了不同的优先级，让高优先级的业务，如视频会议在WAN通道中优先通过，让低优先级应用慢慢排队，从而确保了视频会议、Skype等重要业务的高度稳定。当然，我们也可以手动修改各类业务调整VPN和优先级。

我们终于明白了VPN的重要性，并且有了 IPSEC-VPN和MPLS-VPN两大利器，运营商是否可以为企业搭建出完美的虚拟专用网络（VPN）？我们的答案是否定的。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-9.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-9.png)  
[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-9-1.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-9-1.png)

传统的WAN架构以及主要问题

传统的VPN有着非常明显的缺陷：  
1、业务开通成本比较高，开通时间长；  
开通成本高主要是因为需要人工上门进行终端的配置。另外，业务开通周期较长，因为电信运营商内部要走流程，而这些开通流程短则1周，长则1个月。具体流程如下图所示：

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-10.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-10.png)

传统WAN业务开通示意图

2、某些企业缺乏专业的IT技术人员；  
换句话说就是IT人员技术水平有限，当业务出现卡顿、中断时，无法快速定位故障，只能告知运营商，然后由运营商技术专家排查故障，从而导致维护时间长，企业生产受影响。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-11.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-11.png)

3、故障处理缺乏有效工具和流程：  
即使企业配备了IT技术人员，就一定能快速处理业务卡顿、中断的故障吗？答案是否定的，因为以下几点：  
（1）MPLS-VPN的网络维护权限不在企业自己手中，很多的网络参数是由运营商设置的。企业的IT人员只能排查企业侧设备，如防火墙，路由器，交换机等，排查手段无非是Ping,Tracert这些基于数据包的工具，当检查了这些设备配置以及Ping包检测发现延时正常后，业务仍然卡顿、中断时，他们往往束手无策，只能再次求助于运营商，可是却耽误了故障最佳的恢复时间。  
（2）作为运营商，当接到类似业务卡顿、中断的故障时，最为头疼，解决手段也很有限，在协助客户排查了内网问题后，往往只能免费增加VPN带宽，但有时候并不能完全根除，故障仍然存在，因为有些卡顿、中断的故障并不是带宽不够引起的。  
（3）某些企业IT技术人员虽然通过基于五元组的方式为各类应用设置了优先级，但是这种方式存在缺陷，原因是五元组（源IP地址、目的IP地址、源TCP/UDP端口号、目的TCP/UDP端口号、协议号）中的任意参数发生变化时，之前做的策略就会失效，业务就会卡顿甚至中断。

4、业务卡顿无法定责：  
某些企业客户，只要网络出了问题，就认为是运营商的问题，运营商一直扮演“背锅侠”，而现有的检测机制无法精准定位故障位置，运营商也无法输出有说服力的图形化报告给自己甩锅。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-12.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-12.png)

5、MPLS-VPN部署成本高：  
以某省电信的跨省10M的MPLS-VPN资费举例，每端的MPLS-VPN设备端口+带宽费用是10000元，开通1条端到端（2端）的MPLS-VPN则需要20000元，在目前的经济不景气的背景下，不是每个企业都能负担得起这个昂贵资费的。

并且运营商的国际VPN线路价格更加昂贵（某省电信的跨国10M的MPLS-VPN的价格为80000元/月，导致某些有海外业务的企业客户望而却步。而普通的海外线路质量又差，企业又不敢用，这就严重制约了企业的海外业务拓展。

6、缺乏备份保护机制：  
缺乏MPLS-VPN专线没有备份线路，存在单点故障：连1条都嫌贵，还弄2条来做主备？  
如此看来，传统VPN被替代是大势所趋，那么SD-WAN与WAN相比多了哪些东西呢？SD-WAN做了哪些优化呢？下面我们将重点阐述。

我们还是拿上面的公交专用道来比喻。

SD-WAN来说，意义就完全不一样了，首先要保证专用道上一定行驶的是公交车，不允许其他车辆抢占，然后优先保证公交专用道的完整，同时设置紧急预案，道路出现问题或是车出现问题，及时安排优先处理。很明显SD-WAN不仅仅是专用道。

**而是通过软件来定义出公交的专用车道。**

我们发现这里有一些微妙的变化：  
1、原有道路并没有拓宽，只是划出公交车道；  
2、公交车道只允许公交车行驶；  
3、公交车道的使用时间并不是持续的，而是有规定的时间；  
我们从某一方面可以理解为，这是一种将现有资源最大化合理利用的一种方案，SD-WAN在网络中正是扮演这个角色。它将网络现有的资源进行最大化的调度。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-13.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-13.png)

## [](https://www.sdnlab.com/23294.html#3%E3%80%81sd-wan%E8%83%8C%E6%99%AF)3、SD-WAN背景

为什么SD-WAN会出现，和上面说的公交车道出现的原因一样。我们目前并不是缺资源，而用户的需求也从以前没有路需要修路，变为如何将现有的道路合理化使用。

在云计算、移动应用、企业全球化成为大背景的环境下，越来越多的实时应用（异地办公、视频会议、远程桌面、支付交易系统、远程医疗）要在多个节点间传递，断线、访问慢等问题将会放大用户的不满，造成交易流失。而SD-WAN的出现不仅解决了互联网不稳定、专线造价昂贵的问题，最重要的是能够极大程度上满足这些应用即时性和实时性的要求。

换句话说，以前我们想让两地人民见面，必须新建道路，这样一来成本和时间都很大。现在不一样了，我们要实现两地的互联，只需要在原有的道路上画出一条专用线，然后规定好使用时间即可。

## [](https://www.sdnlab.com/23294.html#4%E3%80%81sd-wan%E4%BC%98%E7%82%B9)4、SD-WAN优点

### [](https://www.sdnlab.com/23294.html#4.1%E3%80%81sd-wan%E7%9A%84%E5%8A%A0%E9%80%9F%E8%83%BD%E5%8A%9B)4.1、SD-WAN的加速能力

如果SD-WAN不是用来加速，那么将毫无意义。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-14.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-14.png)

但是，SD-WAN可不仅仅只是加速这么简单。在整个SD-WAN的服务体系中，大体可分为四部分：

### [](https://www.sdnlab.com/23294.html#4.2%E3%80%81sd-wan%E7%9A%84%E6%9C%8D%E5%8A%A1%E8%83%BD%E5%8A%9B)4.2、SD-WAN的服务能力

-   开放性：兼容桥接、路由、NAT、私网互联和VPN等多种接入方式，主流协议对接各种SD-WAN的POP点
-   精确性：将流量准确分为不同服务等级应用，指向不同的WAN方向，包括SD-WAN方向；
-   简化部署：一键部署，不只是接入，更是融合的计费策略；
-   服务稳定性：链路探测，评估质量，WAN优化失去后服务回退保障；
-   合规性：不做触犯法律合规，只加速合法应用；
-   全局监控：云平台监控，集中资源下发，集中大数据分析

### [](https://www.sdnlab.com/23294.html#4.3%E3%80%81sd-wan%E7%9A%84%E7%AA%81%E5%87%BA%E4%BC%98%E5%8A%BF)4.3、SD-WAN的突出优势

**4.3.1、业务快速自动开通**  
客户侧设备易部署：通过邮件方式完成业务上线只需三步

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-18.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-18.png)

SD-WAN可以采用邮件开局方式，支持在总部端提前配置好分支基础网络、总部接入等信息， 将配置加密存储在 URL 链接中发送给分支管理员。 通过分支管理员手动点击该链接， 配置将自动完成， 实施的分支管理员即使没有任何 IT 经验， 也可在邮件内容的指导下轻松完成上架部署。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-19.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-19.png)

SD-WAN部署邮件链接

**4.3.2、2条WAN线路主备倒换，智能调度**  
根据线路侧的链路质量，按照业务优先级进行智能调度。

-   合理精细化的带宽管理策略，保证高等级的应用可以优于其它低等级应用的服务质量；  
    [![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-20.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-20.png)
-   基于应用的调度可以解决传统三层网络调度所无法避免的难题；

**4.3.3、WAN网络自管自控，智能运维**  
对于SD-WAN来说，多少还是遗传了SDN的基因。所以针对于开通的专线进行管控是十分重要的，也是必备的。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-21.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-21.png)

**4.3.4、应用级QOS**

应用识别、根据不同应用随时随地分配带宽，调整业务优先级；

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-22.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-22.png)

应用级QOS

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-23.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-23.png)

优化效果对比

**4.3.5、价格低廉**  
SD-WAN的开通成本与使用成本相比原有的MPLS VPN、MSTP低很多。

-   SD-WAN优化最后一里互联网连接，应用性能瓶颈的远程连接部分（中间一里）由SD-WAN厂商运营的高质量私网承载，大幅减少企业IT预算。
-   集中管理系统统一管理企业总部、数据中心、及所有分支机构的网络接入，降低企业分支机构的IT人员开支。

因此，对于那些在乎成本，用不起MSTP、国际CN2电路、但是注重业务体验，同时需要划分业务优先级，有海外线路需求的民营、外资企业来说，这就是一种福利；

此外，对于某些企业当前业务质量很差需要改善的情况，可以通过SD-WAN进行改善，例如双盾环保的金蝶软件访问卡顿、晶晟科技的SKYPE海外会议无法使用等等均可以通过SD-WAN进行解决。

总体来看，同比例带宽情况下，SD-WAN相较MPLS，每年至少可节省30%的成本投资。

## [](https://www.sdnlab.com/23294.html#5%E3%80%81sd-wan%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)5、SD-WAN应用场景

我们先来看一个组对话：  
客户：哎，你们公司有SD-WAN吗？  
销售：有呀，您具体的需求是什么？  
客户：多分支互联、WAN网络自管自控、混合云、海外加速…好多需求啊，你们能满足吗？  
销售：放心，我们的SD-WAN都满足

在SD-WAN这个领域，我觉得各个公司的市场销售会遇到类似的情况，用户会越来越多地问：你有SD-WAN吗？虽然用户也搞不清楚什么是SD-WAN，但是当他说这句话的时候你基本上就能猜个八九不离十，你会回答他你什么都有，SD-WAN的功能组成是无所不包的。这样你会发现一个问题，各个公司的产品组合，除了公司名称和LOGO不一样，其他的好像没什么区别。

多分支互联、混合云接入、海外业务加速、IDC互联、应用加速等等。原来在我们眼里SD-WAN无所不能。

由于SD-WAN部署使得添加新的分支站点变得更加容易，因此具有分布式分支站点的垂直行业是最先使用SD-WAN的一些行业，包括零售，制造业，医疗保健，餐馆和金融服务。而服务形式多分支互联、混合云接入、海外业务加速、IDC互联、应用加速等。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-24.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-24.png)

SD-WAN标准组网

**典型场景一：快速实现混合云组网，一键直达云业务**  
SD-WAN 提供一套简便且高品质的解决方案，快速实现数据中心（自建或托管 IDC ）与云之间的高速直连，同时还可利旧原有专线及互联网链路资源，通过云终端实现一键快速接入。

**典型场景二：构建企业专属广域网，轻松实现分支互联**  
通过SD-WAN，可依托骨干网资源快速构建专属的网络连接，分支机构也可通过云终端快速接入，组网效率显著提升，成本大幅降低。

**典型场景三：轻松构建异地灾备网络，带宽按需动态调整**  
通过SD-WAN实现全国动态多线 BGP 网络构建与异地数据灾备中心的网络连接，访问一跳直达、无拥塞与延时，并且可以按需灵活调整网络带宽，不仅节省带宽开支，更保证网络的高可用性。

**典型场景四：基于桌面云组网，企业总部与分支协同办公**  
异地桌面云的访问，对于网络传输速率和安全性都有很高的要求。通过SD-WAN 服务，企业分支可灵活接入企业核心网络访问桌面云，满足日常办公、移动办公的需求。

## [](https://www.sdnlab.com/23294.html#6%E3%80%81panabit-sd-wan%E4%BC%98%E5%8A%BF)6、Panabit SD-WAN优势

在SD-WAN世界中，控制了网络边缘就等于控制了全场。  
日本著名中锋赤木刚宪曾表示：控制篮板球的人，就控制了全场……

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-25.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-25.png)

没错，在派网看来，控制了网络边缘，就能控制整个SD-WAN。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-26.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-26.png)

### [](https://www.sdnlab.com/23294.html#6.1%E3%80%81%E7%B2%BE%E5%87%86%E7%9A%84%E5%BA%94%E7%94%A8%E8%AF%86%E5%88%AB)6.1、精准的应用识别

Panabit通过14年的积累，在应用识别能力方面无人出其右。通过对DPI、DFI、节点跟踪、主动探测等方面的技术积累和研究。建立了在应用识别方面的绝对实力。也许Panabit应用识别只比别人强5%，但影响用户体验的往往就是那5%，占用带宽的往往就是那5%，出现安全问题也往往就是那5%。

不仅如此，Panabit在精确应用识别的前提下，还可以做到 1：1的会话级日志留存。对于设备的性能有十分苛刻的要求，这点得益与Panabit自身PanaOS操作系统对于X86内核的优化和经验积累。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-27.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-27.png)

### [](https://www.sdnlab.com/23294.html#6.2%E3%80%81%E7%A8%B3%E5%AE%9A%E7%9A%84sd-wan%E9%9A%A7%E9%81%93)6.2、稳定的SD-WAN隧道

派网具有自主研发的隧道协议iWAN。  
iWAN诞生的作用很简单：

-   多台Panabit设备间进行VPN组网；
-   为用户提供快速稳定的隧道；
-   不过iWAN优势却很明显：  
    [![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-28.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-28.png)
-   重连速度很快：比L2TP的要快一个数量级，L2TP要重连，需要有几十次交互，而iWAN只需要一次即可。
-   客户端不受底层承载线路IP变化影响：当底层承载线路（比如PPPOE拨号线路）的IP地址发生变化时，不会影响iWAN隧道，iWAN隧道不会中断，保证通信正常进行；因为很多用户是通过PPPOE拨号线路出去的，PPPOE拨号线路重拨时一般会改变IP地址，如果用L2TP的话，那么这个L2TP会话就要重建；而用iWAN的话，现有的会话可以照常使用，不需要做任何改变；
-   传输效率高：iWAN的包头很小，只有8个字节，而且在后续版本里，我们会压缩IP报文头，这样可以继续减少额外报文头的大小，所以能大幅度提升传输效率；
-   抗干扰：不像L2TP，中间人可以直接发包TERMINATE，iWAN控制命令有完整性检查，可以避免中间人攻击。

### [](https://www.sdnlab.com/23294.html#6.3%E3%80%81sd-wan%E9%9A%A7%E9%81%93%E8%B4%A8%E9%87%8F%E7%9B%91%E6%8E%A7)6.3、SD-WAN隧道质量监控

Panabit的网络业务性能感知系统对比传统网络性能感知系统，在感知的深度、广度上都有很大提升。

首先，在深度上对比网络层性能检测系统，Panabit检测的目标是会话级的，面向的是七层的应用。看到的更接近网络中实际发生的真相。除了可以识别出应用协议数据结构，还能看到应用协议的交互流程。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-29.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-29.png)

其次，在广度上可以实现单个协议时延、抖动等业务性能指标在时间维度、地区维度上的查询。可以实现多个协议和用户的交叉查询，监测异常情况，上报预警或告警。

Panabit可以区分SD-WAN中每条业务的“客户侧时延”、“服务响应时延”和“应用时延”。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-30.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-30.png)

**客户侧时延：**指客户内网到Panabit的时延；如果此时延大，基本可以将问题定位在内网中毒或者内网网络设备故障等问题。  
**服务器响应时延：**指业务在服务器上停留和响应的时延；如果此时延过大，则说明提供业务的服务器负载或者故障等问题。  
**应用时延：**指业务经过Panabit，从服务器上返回的时延；如果此时延过大，而服务时延很小，则说明是运营商网络侧出现问题。  
区分这些的好处是，当出现网络和业务问题时，可以依据此功能快速定位问题的出处，让问题定位更有针对性，更有效率。

### [](https://www.sdnlab.com/23294.html#6.4%E3%80%81%E9%A6%96%E4%BE%8Bsd-wan%E6%9C%8D%E5%8A%A1%E5%B9%B3%E5%8F%B0-raas)6.4、首例SD-WAN服务平台-RAAS

利用发布的RAAS（Radius As A Service）平台，Panabit成为第一个向所有SD-WAN厂商开放接入网能力的厂商。RAAS是SD-WAN的电子市场，我们不做SD-WAN，而是架设了一个让所有Panabit用户与SD-WAN厂家的之间沟通的桥梁。

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-31.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-31.png)

Panabit为什么有胆量做RAAS？是因为Panabit有8000万在线IP，超过一亿在线终端设备。我们只是为自己的海量用户向SD-WAN世界敞开了大门，并没有去做空中楼阁。

所有SD-WAN厂商可以自由的在RAAS上发布流量或宽带服务，并且发布的这些服务可以被使用Panabit的用户看到。

对于那些缺乏云服务资源和能力的厂商来说，占领网络边缘是最优的选择，但所有人又都清楚，部署CPE节点的成本是巨大的，这就让一大部分人望而止步。不过Panabit的优势在于，现网的10万台Panabit智能应用网关，均可作为SD-WAN用户的入口。不仅如此，Panabit在提供用户与SD-WAN厂商接口的同时，还可以给用户提供精准的应用识别和保护应用路由、NPM等技术，会作为运维技术支撑出现在服务链条上。

服务场景：  
一、Panabit使用者本地资源发布

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-32.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-32.png)

针对现有的Panabit使用者来说，本地资源如果可以更合理的利用起来是求之不得的。使用者可以在RAAS服务平台上发布服务，为需要的人提供便利，同时将自己本地的资源合理提供给需要者使用和计费。  
二、SD-WAN厂家或公有云厂家资源发布（POP点可以部署Panabit）

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-33.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-33.png)

在公有云或者SD-WAN厂家POP点允许部署Panabit的情况下，使用上图部署场景。

Panabit在用户侧针对用户需求访问云平台资源的业务进行识别和分流，通过Panabit之间创建的iWAN通道，与SD-WAN或公有云直接对接。为用户提高使用体验，也为SD-WAN、公有云厂商提供更多的用户资源。

三、SD-WAN厂家或公有云厂家资源发布（POP点不可以部署Panabit）

[![](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-34.png)](https://img1.sdnlab.com/wp-content/uploads/2019/06/panabit-SD-WAN-34.png)

在已有CPE接入用户，并且POP点不具备部署Panabit能力的场景下，我们在用户和CPE之间部署Panabit（换句话说，是将我们现有的PA用户与CPE连接），而整个Panabit用户体系都将有能力访问这些部署的CPE设备。我们来为有云服务需求的客户讲流量识别后分流到CPE上，提升用户感知，同时为公有云厂商提供更多的用户资源。

参破这其中奥秘的人会发现，现有用户的数量是进入SD-WAN市场的网关供应商的最大价值。我们在网的10万台设备，将可成为SD-WAN厂商接近用户的触角，并且Panabit与生俱来的应用路由、大数据分析和NPM等能力正是用户和SD-WAN厂商对接时所需要的要素。

Panabit致力于为所有需要SD-WAN的客户提供优质服务，愿与所有SD-WAN厂商和公有云厂商共同合作、共同发展。