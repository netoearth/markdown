译者点评：

最近听了很多资深的人士关于开源，以及商业化的分析。开源与商业化，听起来就是一对矛盾的所在，似乎大家都在尝试做其二者的平衡。是先有开源，还是先有商业化？俗话说“谈钱不伤感情”，近几年背靠开源的创业公司如雨后春笋般涌现，即使是开发人员也是需要生活的。

容器神话 Docker 曾经无比风光，盛极一时。即使这样一个备受瞩目，大获风投的热捧的独角兽也未能免俗，并付出了不小的代价。

今天这篇文章讲述了 Docker 这家公司从诞生到巅峰到没落，这一路上所做的抉择，并最终做了开源与商业的分离，再一次从开源踏上找寻商业化之路。这些都是值得我们参考和思考的，不管是已经开源或者准备从事开源的。

## 这篇文章翻译自[How Docker broke in half](https://www.infoworld.com/article/3632142/how-docker-broke-in-half.html)

> 这家改变游戏规则的容器公司是其昔日的外衣。作为云时代最热门的企业技术业务之一的它到底发生了什么？

[Docker 并没有发明容器](https://www.infoworld.com/article/3204171/what-is-docker-the-spark-for-the-container-revolution.html)——将计算机代码打包成紧凑单元的方法，可以轻松地从笔记本电脑移植到服务器——但它确实通过创建一套通用的开源工具和可重用的镜像使其成为主流，这使所有开发人员只需构建一次软件即可在任何地方运行。

Docker 使开发人员能够轻松地将他们的代码“容器化”并将其从一个系统移动到另一个系统，迅速将其确立为行业标准，颠覆了在虚拟机 (VM) 上部署应用程序的主要方式，并使 Docker 成为新一代最快被采用的企业技术之一。

今天，Docker 仍然活着，但它只是它可能成为的公司的一小部分，从未成功地将这种技术创新转化为可持续的商业模式，最终导致其企业业务于 [2019 年 11 月出售给 Mirantis](https://twitter.com/QuinnyPig/status/1194687851587198977?s=20)。InfoWorld 采访了十几位前任和现任 Docker 员工、开源贡献者、客户和行业分析师，了解 Docker 如何分崩离析的故事。

## Docker 诞生了

2008 年由 Solomon Hykes 在巴黎创立的 DotCloud，这个后来成为 Docker 的公司最初被设计为供开发人员轻松构建和发布他们的应用程序的[平台即服务 (PaaS)](https://www.infoworld.com/article/3223434/what-is-paas-a-simpler-way-to-build-software-applications.html)。

在 2010 年夏天一起搬到硅谷[参加著名的 Y Combinator 计划](https://blog.ycombinator.com/solomon-hykes-docker-dotcloud-interview/)之前，Hykes 很快就加入了他的朋友兼程序员同事 Sebastien Pahl。已经被拒绝一次的 Hykes 和 Pahl 重新申请，Pahl 的父亲在他们面试前几周把去旧金山的机票钱放在他们面前。唉，这对夫妇再次被拒绝，直到 YC 校友 James Lindenbaum，一家名为 [Heroku 的竞争公司的创始人](https://www.infoworld.com/article/3614210/the-decline-of-heroku.html)，出面为他们担保。

我们所知道的 Docker 于 [2013 年 3 月在 PyCon 上由 Hykes 首次演示](https://www.youtube.com/watch?v=362sHaO5eGU)，他解释说开发人员一直要求访问支持 DotCloud 平台的底层技术。他在那次谈话中说：“我们一直认为能够说“是”会很酷，这是我们的底层部分，现在你可以和我们一起做 Linux 容器，做任何你想做的事，去构建你的平台，这就是我们正在做的，”。

Docker 首席执行官 Ben Golub 2013 年至 2017 年之间，告诉 InfoWorld，“这听起来很老套，但 Solomon 和我谈论的是预发布，我们可以看到所有集装箱船进入奥克兰港，我们正在谈论集装箱在航运界的价值。事实上，从世界的一侧运送汽车比将应用程序从一个服务器带到另一个服务器更容易，这似乎是一个需要解决的问题。”

Docker 开源项目迅速崛起，吸引了成千上万的用户，与[微软](https://www.docker.com/blog/docker-microsoft-partnership/)、[AWS](https://aws.amazon.com/blogs/apn/docker-trusted-registry-aws-marketplace/) 和 [IBM](https://www.eweek.com/cloud/ibm-partners-with-docker-launches-containers-service/) 等公司建立了备受瞩目的合作伙伴关系，并获得了[满满一车的风险投资](https://www.infoworld.com/article/3622960/another-day-another-multi-billion-ipo-for-open-source.html?nsdr=true)，包括 Benchmark 的 Peter Fenton 和 Trinity Ventures 的 Dan Scholnick 的早期投资。调整后的公司更名为 Docker，并从 Benchmark、Coatue Management、Goldman Sachs 和 Greylock Partners 等公司筹集了近 3 亿美元。然而，与许多基于开源软件的公司一样，它很难找到一种可盈利的商业模式，而这些投资者从未得到他们的大笔回报。。

RedMonk 分析师 James Governor 说，“Solomon 建立了过去 20 年来最引人注目的技术之一，并且在将观点打包并使其对大量开发人员非常有价值的业务中。Docker 是否做出了错误的决定？显然是的，但风投都疯了，他们向他们投入的钱意味着他们一定觉得他们可以做任何事情，这是有问题的。”

快进到 2021 年，这个故事的简版是，备受欢迎的开源容器编排工具 [Kubernetes](https://www.infoworld.com/article/3268073/what-is-kubernetes-your-next-application-platform.html) 通过取代其主要利润来源：一个名为 Docker Swarm 的企业版容器编排工具，吃掉了 Docker（业务）的午餐. 然而，真实的故事要复杂得多。

## 开源商业化很难

巨额的风险投资、快速增长的竞争格局以及云行业巨头都想分一杯羹的阴影，将这家年轻的公司带入了一个犹如压力锅的运营环境。

“有一种说法是’大象打架，草被践踏'，我们很清楚这不仅是针对 Docker，还有云供应商的相互竞争。他们都想把我们拉向不同的方向。既要保持我们的价值观和根基，又要建立一个企业，这个根本就是个困局”Golub 说。

这位前 CEO 指出，随着 Docker 的发展，所有这些因素都造成了“自然的紧张关系”。Golub 说：“我们希望建立伟大的社区并通过开发者产品获利，同时还希望建立一个伟大的运营商产品，让客户能够大规模构建和部署容器。这就是我们的愿景，很快我们意识到我们必须迅速扩大规模，而且没有太多时间来平衡社区和成为一家商业企业……在一家初创公司，你每天要做出 100 个决定，你希望 80 个是对的。”

2014 年左右，Docker 开始认真考虑将其在容器领域的领先地位货币化的商业战略，当时该公司将部分风险投资资金用于 [2014 年 Koality 的收购](https://techcrunch.com/2014/10/07/docker-acquires-koality-in-engineering-talent-grab/%23:~:text=Docker%2520decided%2520to%2520use%2520some,today%2520for%2520an%2520undisclosed%2520price.)和 [2015 年的 Tutum 的收购](https://www.docker.com/blog/docker-acquires-tutum/)，同时还推出了自己的企业支持计划的第一次迭代。

这些投资催生了像 Docker Hub 这样的产品——你可以认为它有点像 Docker 镜像的 GitHub（[现在也存在](https://www.infoworld.com/article/3623291/github-container-registry-available-for-production-use.html)）—— 以及最终的 Docker 企业版。但这些产品都没有真正受到企业客户的欢迎，他们通常乐于与更成熟的合作伙伴合作，或者构建而不是购买解决方案，尽管 Docker 努力生产客户真正想要的一系列产品。

今年夏天在法国度假时，Hykes 告诉 InfoWorld：“我们从未发布过出色的商业产品，原因是我们没有集中注意力。我们试图做每件事的一点点。维持开发者社区的增长并构建一个伟大的商业产品已经够难的了，更不用说三四个了，而且基本不可能同时做到，但这就是我们试图做的，我们花了大量的钱来做这些事。 ”

DockerDocker 业务发展和技术联盟的前副总裁、最早的员工之一 Nick Stinemates 说：“在开源之外出现了零技术交付，根本无法交付商业软件。”

事后看来，Hykes 认为 Docker 应该花更少的时间来运送产品，而应该花更多的时间倾听客户的意见。Hykes 说：“我本来不会急于扩大商业产品的规模，而是投入更多资金从我们的社区收集见解，并建立一个致力于了解他们的商业需求的团队。我们在 2014 年有一个转折点，当时我们觉得我们等不及了，但我认为我们等待的时间比我们意识到的要多。”

其他人认为 Docker 过早地免费提供了太多东西。今年早些时候，[谷歌的 Kelsey Hightower 告诉 Increment 杂志](https://increment.com/containers/docker-ce-and-ee/)： “他们免费推出了一些东西，这就是本垒打。他们解决了整个问题并达到了这个问题的天花板：创建一个映像、构建它、将它存储在某个地方、然后运行它。还需要做什么？”

Hykes 不同意这种说法。他说：“我认为这是错误的，一般来说，核心开源产品创造了巨大的增长，这首先创造了商业化的机会。许多公司成功地将 Docker 商业化，但 Docker 没有。有很多东西可以商业化，只是 Docker 未能将其商业化。”

例如，[红帽](https://www.openshift.com/blog/red-hat-puts-docker-kubernetes-at-the-center-of-its-openshift-3-paas)和 [Pivotal](https://tanzu.vmware.com/content/blog/pivotal-cloud-foundry-has-supported-docker-for-a-long-time-now-pivotal-web-services-does-too)（现在是 VMware 的一部分）都是 Docker 的早期合作伙伴，将 Docker 容器集成到他们的商业 [PaaS](https://www.infoworld.com/article/3223434/what-is-paas-a-simpler-way-to-build-software-applications.html) 产品（分别是 OpenShift 和 Cloud Foundry）中，并为开源项目做出了贡献。

Stinemates 说：“如果我是慷慨的，红帽公司早期的贡献让 Solomon 有点失控了。[Solomon 烧掉了很多桥梁，Hacker News 上有关于他与反对者争吵的帖子](https://news.ycombinator.com/threads?id=shykes)。企业合作伙伴不可能与 Solomon 一起拥有这些。”

今天，Hykes 说他犯了混淆“社区与生态系统”的错误。红帽特别“不是社区的一部分，他们从来没有为 Docker 的成功而生根，”他说。“我们的错误是非常希望他们成为社区的一部分。回想起来，我们永远不会从这种伙伴关系中受益。”

因此，旅游科技公司 Amadeus 等早期客户在 2015 年转向红帽，以填补他们认为 Docker 留下的企业级空白。Amadeus 的云平台负责人 Edouard Hubin 通过电子邮件告诉 InfoWorld：“我们直接从使用 \[Docker 的\] 开源版本的先驱模式转变为与红帽建立强大的合作伙伴关系，他们为我们提供容器技术的支持。容器化是远离虚拟化的技术变革的第一步。真正的游戏规则变革者是容器编排解决方案。显然，Docker 输给了 Kubernetes，这对他们来说是一个非常困难的局面。”

## Kubernetes 的决定

Docker 会后悔之前做的一系列决定，因为它拒绝真正接受 Kubernetes 作为首选的新兴容器编排工具 —— Kubernetes 允许客户大规模、一致地运行容器队列——而不是短视地推进自己专有的 Docker Swarm 编排器（[RIP](https://boxboat.com/2019/12/10/migrate-docker-swarm-to-kubernetes/)）。

Docker 最早也是服务时间最长的员工之一 Jérôme Petazzoni 说：“最大的错误是错过了 Kubernetes。我们处于集体思想泡沫中，我们在内部认为 Kubernetes 太复杂了，而 Swarm 会成功得多。没有意识到这一点是我们的集体失败。”

事实是，Docker 在 2014 年有机会与谷歌的 Kubernetes 团队密切合作，并在此过程中可能拥有整个容器生态系统。Stinemates 说：“我们本可以让 Kubernetes 成为 GitHub 上 Docker 旗帜下的一流 Docker 项目。事后看来，Swarm 上市太晚是一个重大错误。”

据在场的多名人士称，谷歌旧金山办公室的早期讨论是技术性的和紧张的，因为双方对如何进行容器编排持不同的意见。

Kubernetes 联合创始人、现任 VMware 副总裁 Craig McLuckie 表示，他提出将 Kubernetes 捐赠给 Docker，但双方未能达成协议。他告诉 InfoWorld：“那里有一个相互傲慢的因素，从他们那里我们不了解开发人员的经验，但相互的感觉是这些年轻的新贵真的不了解分布式系统管理。”其他人则表示讨论更为非正式，并且侧重于容器技术的联合开发。无论哪种方式，团队从来没有意见一致并最终分道扬镳，[谷歌在 2014 年夏天推出了 Kubernetes](https://kubernetes.io/blog/2018/07/20/the-history-of-kubernetes-the-community-behind-it/)。

Hykes 对 Google 向 Docker 提供 Kubernetes 项目的所有权提出异议，称他们“有机会像其他人一样成为生态系统的一部分”。

Hykes 承认当时 Docker 和 Google 团队之间处于紧张关系。Hykes 说：“有那么一刻，自负占了上风。谷歌的很多聪明和有经验的人都被 Docker 的完全局外人蒙蔽了双眼。我们没有在谷歌工作，我们没有去斯坦福，我们没有计算机科学博士学位。有些人觉得这是他们的事，所以有一场自我之战。结果并不是 Docker 和 Kubernetes 团队之间的良好合作，而此时合作确实有意义。”

Stinemates 说：“一方面是基本的自我，另一方面是与 \[Kubernetes 联合创始人\] Joe Beda、Brendan Burns 和 Craig McLuckie 的紧张关系——他们对服务级别 API 的需求有强烈的看法，而从简单的角度来看 Docker 在技术上对单个 API 有自己的看法，这意味着我们无法达成一致。”

Hykes 承认，当时 Docker 面临着为想要扩展容器使用规模的客户寻找编排解决方案的压力，但当时Kubernetes 将成为该解决方案并不明确。Hykes 说：“Kubernetes 太早了，而且是几十个中的一个，我们并没有神奇地猜测它会占据主导地位，甚至不清楚谷歌对它的承诺。我问我们的工程师和架构师该怎么做，他们建议我们继续使用 Swarm。”

甚至 McLuckie 也承认他“不知道 Kubernetes 会变成 Kubernetes。回顾历史很容易认为这是一个糟糕的选择。”

不管它失败了，Kubernetes 最终赢得了容器编排之战，其余的成为软件行业的匆匆过客。

451 Research 的分析师 Jay Lyman 说：“Kubernetes 来了，并抢走了所有的风头。它代表了谷歌在开发和开源方面对容器的使用，这在很多方面都超过了对 Docker 的关注。\[Docker\] 将 Docker Swarm 视为他们通过软件获利的方式。如果他们可以回去，他们可能会从一开始就与 Kubernetes 更紧密地集成。他们过于专注于独自行动。”

McLuckie 说：“我最深切的遗憾之一是我们没有找到谈判的方法。Docker 提供了一些非凡的体验，而 Kubernetes 提供的东西，从体验的角度来看，并不那么引人注目。” 或者，正如 Docker 联合创始人 Sebastien Pahl 指出的那样：“简单并没有获胜。我喜欢 Kubernetes，但它[不适合普通人](https://www.infoworld.com/article/3614850/no-one-wants-to-manage-kubernetes-anymore.html)。”

## 高层的紧张气氛

在 2015 年以 10 亿美元的“独角兽”估值完成 9500 万美元的大型 D 轮融资之后，Docker 最终达到了炒作周期的巅峰。

Steinmates 说：“这设定了非常高的期望，并暴露了我们作为一家公司将面临的一些基本问题。我认为 Ben \[Golub，首席执行官\] 对公司的想法与 Solomon 不同，两人没有意见一致应该不是什么秘密。董事会大量掺和努力让创始人开心，并给 CEO 足够的回旋余地，使公司取得成功。如果由 Solomon 决定，我们会坚持以社区为导向的路线来创造病毒式传播。如果由 Ben 决定，我们会更早地转向业务方面。这种紧张局势导致我们对两者都采取了半途而废的方式。”

[这种方法有效地催生了两个Docker](https://increment.com/containers/docker-ce-and-ee/)：Docker 社区版（面向开发人员的广受欢迎的命令行工具和开源项目）和 Docker 企业版（面向希望大规模采用容器的企业客户的商业工具套件）。不幸的是，公司的动作太慢了，无法正式进行拆分并相应地分配资源。

Golub 承认他们“应该比实际更早地拆分业务”，而 Hykes 同意 Docker “从未找到连接公司这两部分的方法”。

到了 2018 年，裂缝开始显现，因为该公司努力在[日益不满的开源社区](https://www.techrepublic.com/article/why-doesnt-anyone-weep-for-docker/)、强大的合作伙伴和要求在生产中运行容器的企业客户之间找到可行的道路。

不久之后，Hykes 于 2018 年 3 月离开了他在公司的日常角色，他在一篇博客文章中指出，“作为创始人，我当然有复杂的情绪。当你创建一家公司时，你的工作是确保它有一天可以在没有你的情况下取得成功。然后最终有一天会到来，庆祝活动可能是苦乐参半。对于创始人来说，放弃一生的工作绝非易事。”

今天回想起来，Hykes 更简单。“我意识到我不属于这家公司。留下对我来说没有任何意义，所以我离开了……我多半是个应该继续担任首席执行官或离开的不快乐创始人。”

## Docker 一分为二

面对[日益严重的资金问题](https://www.cnbc.com/2019/09/27/docker-is-trying-to-raise-money-following-arrival-of-ceo-rob-bearden.html)，Docker 轮换了新任 CEO，Golub 于 2017 年 5 月让位给前 SAP 执行官 Steve Singh，然后 Singh 于 2019 年 6 月让位给前 Hortonworks 首席执行官 Rob Bearden。

最终要接受批评的是 Bearden。上任后不久，[Docker 于 2019 年 11 月将其企业部分业务出售给 Mirantis](https://twitter.com/QuinnyPig/status/1194687851587198977?s=20)，Docker Enterprise 并入 Mirantis Kubernetes Engine。

Bearden 当时在一份新闻稿中说：“在与管理团队和董事会进行彻底分析后，我们确定 Docker 有两个截然不同的业务：一个是活跃的开发者业务，另一个是成长中的企业业务。我们还发现产品和财务模型大不相同。”

## Docker 如今在哪里？

有了原始投资者 Insight Venture Partners 和 Benchmark Capital 的 3500 万美元现金注入，剩下的 Docker 由原始 Docker Engine 容器运行时、Docker Hub 镜像存储库和 Docker 桌面应用程序支撑，在 7 年公司资深人士 Scott Johnston 的领导下活了下来。

Johnston 告诉 InfoWorld，他正试图通过“像激光一样重新关注开发人员的需求”，让公司回归本源。“我们认为该公司比以往任何时候都更强大，因为三点：以客户为中心、统一的市场定位和生态系统友好的商业模式。”

上周 Docker 宣布更改 Docker 软件的许可条款。很快，为大公司工作的 Docker Desktop 专业用户将不得不注册付费订阅才能继续使用该应用程序。

Johnston 决心不重蹈覆辙，专注于为公司的核心软件开发人员受众提供价值。他说：“我们的野心更大，因为开发人员生态系统要面向的是世界上的每个开发人员，而不仅仅是那些与我们的运行时一致的开发人员。”

Johnston 认为“Docker 2.0”的增长机会在于为安全、已验证的镜像构建新的开发人员工具和[可信内容](https://www.docker.com/press-release/docker-expands-trusted-content-offerings)，以及新兴计算模型（如[无服务器](https://www.infoworld.com/article/3406501/what-is-serverless-serverless-computing-explained.html)、[机器学习](https://www.infoworld.com/article/3214424/what-is-machine-learning-intelligence-derived-from-data.html)和[物联网](https://www.networkworld.com/article/3207535/what-is-iot-the-internet-of-things-explained.html) 以容器技术为基础的工作负载）背后的持续动力。

同时，Docker 仍然是行业标准的容器运行时，[如今 Docker Desktop 安装在 330 万台机器上](https://www.docker.com/blog/docker-index-shows-continued-massive-developer-adoption-and-activity-to-build-and-share-apps-with-docker/)。此外，在 [Stack Overflow 的 2021 年开发者调查](https://insights.stackoverflow.com/survey/2021)中，49% 的受访者表示他们经常使用该工具。

尽管如此，人们仍然对可能发生的事情深感失望。Stinemates 说：“如果我想要轻率，我会问今天 Docker 是否存在。从职业角度来看，这很可悲。我仍在寻找一家像 Docker 一样令人兴奋和充满活力的公司，并能创造出火花。”

Hykes 说：“可以公平地说，Docker 未能实现其商业化的潜力……到目前为止。我很高兴 Docker 在这么多年之后有再次有实现商业化的机会。这是对基础项目和品牌的证明。”