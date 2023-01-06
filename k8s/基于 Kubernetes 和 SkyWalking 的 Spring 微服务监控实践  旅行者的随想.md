 2022.11.17  2022.11.17  [Articles](https://blog.besscroft.com/articles/)  4270  9 分钟

## 目录

1.  [前言](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%89%8D%E8%A8%80)
2.  [让我们开始](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E8%AE%A9%E6%88%91%E4%BB%AC%E5%BC%80%E5%A7%8B)
    1.  [为什么需要 Skywalking？](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81-skywalking)
    2.  [Skywalking 是什么？](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#skywalking-%E6%98%AF%E4%BB%80%E4%B9%88)
    3.  [如何收集 Java 程序的数据？](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%A6%82%E4%BD%95%E6%94%B6%E9%9B%86-java-%E7%A8%8B%E5%BA%8F%E7%9A%84%E6%95%B0%E6%8D%AE)
3.  [如何使用](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8)
    1.  [在项目中使用 skywalking agent](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-skywalking-agent)
    2.  [Skywalking 服务配置](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#skywalking-%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE)
    3.  [idea 配置](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#idea-%E9%85%8D%E7%BD%AE)
4.  [图表信息](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%9B%BE%E8%A1%A8%E4%BF%A1%E6%81%AF)
    1.  [调用链路信息](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E8%B0%83%E7%94%A8%E9%93%BE%E8%B7%AF%E4%BF%A1%E6%81%AF)
    2.  [服务业务数据图表](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E6%9C%8D%E5%8A%A1%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%9B%BE%E8%A1%A8)
    3.  [JVM 数据图表](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#jvm-%E6%95%B0%E6%8D%AE%E5%9B%BE%E8%A1%A8)
5.  [开发中的注意点](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%BC%80%E5%8F%91%E4%B8%AD%E7%9A%84%E6%B3%A8%E6%84%8F%E7%82%B9)
    1.  [跨线程追踪](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E8%B7%A8%E7%BA%BF%E7%A8%8B%E8%BF%BD%E8%B8%AA)
    2.  [全局异常处理](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%85%A8%E5%B1%80%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86)
    3.  [traceId 生成规则](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#traceid-%E7%94%9F%E6%88%90%E8%A7%84%E5%88%99)
6.  [最后](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E6%9C%80%E5%90%8E)

## [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%89%8D%E8%A8%80)[前言](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%89%8D%E8%A8%80)

最近在哔哩哔哩做了一期直播技术分享，算是人生中第一次吧🙂内容是基于 Kubernetes 和 Skywalking 的 Spring 微服务监控实践，实现 Spring 云原生可观测性。自己排练的时候，准备了一个多小时的内容，结果直播时太紧张，半个多小时就讲完了😂

本来打算用 PPT 整理一些概念和注意点来讲，然后结合项目讲解项目怎么配置、怎么部署，以及请求进来了，怎么通过 Skywalking“观测”请求走过的调用栈，以及查看日志等。由于时间问题，部署这一块就省略掉了，只展示了最终的效果，毕竟部署特别费时间。后面也打算从 0 开始，搭建 K8S，然后部署 nacos、redis、数据库之类的，再把项目和 skywalking 给部署上去。直播录屏正好可以拿来做项目的视频部署教程，也锻炼锻炼自己临场实践能力，以及出问题了怎么查资料去解决。

> 开播前后有很多朋友的鼓励和支持，这里谢谢大家！本文的内容算是与视频内容的互补。

## [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E8%AE%A9%E6%88%91%E4%BB%AC%E5%BC%80%E5%A7%8B)[让我们开始](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E8%AE%A9%E6%88%91%E4%BB%AC%E5%BC%80%E5%A7%8B)

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81-skywalking)[为什么需要 Skywalking？](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81-skywalking)

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-001.png)

在 Spring 微服务场景中开发过的小伙伴应该都了解，我们在排查问题的过程中，往往会因为调用栈太长、跨过的服务太多、请求量太大、无法准确追踪异步线程等，浪费大量的时间。这种情况并不像单元测试对功能性方法测试一样，很方便的就能看到报错信息以及 Debug，我们需要有一种方式，帮我们追踪每一个”请求“线程，来实现”观测“。

拿图中的架构来举例子，请求打到外部网关入口，然后被转发到 Spring-Gateway 所在的 [Service](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/) 的 [NodePort](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)，对应的流量会负载均衡到每一个这个 Service 内的 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)。然后 Spring Gateway 会根据路由，将流量转发到对应的业务服务。业务服务可能会操作数据库、redis 以及消息队列之类的中间件，最终处理完之后，将响应回对应的请求。而这个过程产生的日志和监控数据，将会被上报给 Skywalking OAP，我们访问 Skywalking UI 的 Service 就能看到界面了，通过各个图表获取我们需要的信息。

程序在出现异常时，会将 traceId 一并响应给请求方，这样就可以通过 traceId 去 Skywalking 查询相应的数据了。在上线前，它有一个典型的案例场景：测试自己复现了 Bug 之后，拿到 traceId 给开发，开发去 UI 查看日志和数据。极大地降低了测试和开发人员间的友好互动频率、维护了团队的良好氛围形象、提升了 bug 响应修复的速度，同时避免了在测试电脑上有 bug 但在开发电脑上世界和平的现象（笑死

> 注意：这里每个负载间都是通过 Service 的虚拟 IP 来通信的，不能够使用 Pod IP，因为重新部署后 IP 会发生变化，同时也将无法做到负载均衡了。

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#skywalking-%E6%98%AF%E4%BB%80%E4%B9%88)[Skywalking 是什么？](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:skywalking-%E6%98%AF%E4%BB%80%E4%B9%88)

[SkyWalking](https://skywalking.apache.org/) 是一个开源可观测平台，用于收集、分析、聚合和可视化来自服务和云原生基础设施的数据。SkyWalking 提供了一种简单的方法来保持分布式系统的清晰视图，甚至跨云。它是一种现代 APM，专为云原生、基于容器的分布式系统而设计。

SkyWalking 为服务(Service)、服务实例(Service Instance)、端点(Endpoint)、进程(Process)提供可观察性能力。

服务可以映射成我们的 Spring 的每一个服务，在 Kubernetes 中也就是 Service，而服务实例对应着 Service 内的 Pod（举个例子：咱们的 Spring Gateway 网关”服务“，可以启动多个实例对吧？）。而在开发中我们最常关注的是端点，端点用于传入请求的服务中的路径，例如 HTTP URI 路径或 gRPC 服务类 + 方法签名。

> 举个例子，前端请求后端的登录接口，`http://127.0.0.1:8000/pisces-admin/user/login`，就可以是一个 endpoint，这个 endpoint 的内容是 `POST:/user/login`。

SkyWalking 在逻辑上分为四个部分：Probes、Platform backend、Storage 和 UI。作为开发，我们最需要关注 的是 probes 和 ui，probes 可以理解为程序中的“探针”，而 ui 是我们“观测”数据指标的界面。

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%A6%82%E4%BD%95%E6%94%B6%E9%9B%86-java-%E7%A8%8B%E5%BA%8F%E7%9A%84%E6%95%B0%E6%8D%AE)[如何收集 Java 程序的数据？](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%A6%82%E4%BD%95%E6%94%B6%E9%9B%86-java-%E7%A8%8B%E5%BA%8F%E7%9A%84%E6%95%B0%E6%8D%AE)

介绍完 Skywalking 的一些概念之后，我们最关心的肯定就是如何从 Java 程序中收集数据。刚才我们说到了”探针“，我们就是要靠它来实现。

在 SkyWalking 中，探针是指集成到目标系统中的代理或 SDK 库，负责收集遥测数据，包括跟踪和指标。

SkyWalking Probes 可以分为四种不同的类别：

-   Language based native agent，也就是基于语言的原生代理
-   Service Mesh probes
-   3rd-party instrument library
-   eBPF agent

显而易见，我们要用的就是针对于 Java 的基于语言的原生代理，也就是 [Skywalking Java Agent](https://skywalking.apache.org/docs/skywalking-java/v8.13.0/en/setup/service-agent/java-agent/readme/)，它为 Java 项目提供本机跟踪/指标/日志记录/事件功能。

## [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8)[如何使用](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8)

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-skywalking-agent)[在项目中使用 skywalking agent](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8-skywalking-agent)

**本文以 logback 日志实现为例**，首先我们要将下面的包引入进我们的项目：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span><span>9
</span></code></pre></td><td><pre tabindex="0"><code data-lang="xml"><span>&lt;dependency&gt;</span>
<span>&lt;groupId&gt;</span>org.springframework.cloud<span>&lt;/groupId&gt;</span>
<span>&lt;artifactId&gt;</span>spring-cloud-starter-sleuth<span>&lt;/artifactId&gt;</span>
<span>&lt;/dependency&gt;</span>
<span>&lt;dependency&gt;</span>
    <span>&lt;groupId&gt;</span>org.apache.skywalking<span>&lt;/groupId&gt;</span>
    <span>&lt;artifactId&gt;</span>apm-toolkit-logback-1.x<span>&lt;/artifactId&gt;</span>
    <span>&lt;version&gt;</span>${skywalking.version}<span>&lt;/version&gt;</span>
<span>&lt;/dependency&gt;</span>
</code></pre></td></tr></tbody></table>

> 注意：只要你的微服务项目配置了全局的 spring-cloud 版本，那么这里的 sleuth 就不需要单独指定版本，它会默认继承你当前 `spring-cloud-dependencies` 指定的版本。

-   在 `logback-spring.xml` 中添加 gRPC reporter.

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="xml"><span>&lt;appender</span> <span>name=</span><span>"grpc-log"</span> <span>class=</span><span>"org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender"</span><span>&gt;</span>
    <span>&lt;encoder</span> <span>class=</span><span>"ch.qos.logback.core.encoder.LayoutWrappingEncoder"</span><span>&gt;</span>
        <span>&lt;layout</span> <span>class=</span><span>"org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout"</span><span>&gt;</span>
            <span>&lt;Pattern&gt;</span>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n<span>&lt;/Pattern&gt;</span>
        <span>&lt;/layout&gt;</span>
    <span>&lt;/encoder&gt;</span>
<span>&lt;/appender&gt;</span>
</code></pre></td></tr></tbody></table>

注意，`name` 参数可以自定义，但是记得加上 `appender`：

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span></code></pre></td><td><pre tabindex="0"><code data-lang="xml"><span>&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span>&lt;configuration</span> <span>scan=</span><span>"true"</span> <span>scanPeriod=</span><span>"10 seconds"</span><span>&gt;</span>
    
<span>&lt;!-- gRPC reporter 在这一层级 --&gt;</span>
    
<span>&lt;root</span> <span>level=</span><span>"info"</span><span>&gt;</span>
<span>&lt;!-- 就是下面这一行 --&gt;</span>
<span>&lt;appender-ref</span> <span>ref=</span><span>"grpc-log"</span><span>/&gt;</span>
<span>&lt;/root&gt;</span>
<span>&lt;/configuration&gt;</span>
</code></pre></td></tr></tbody></table>

-   下面这行插件的配置，如果不添加的话，会使用默认值：

```
log.max_message_size=${SW_GRPC_LOG_MAX_MESSAGE_SIZE:10485760}
```

到这里，我们已经配置好基本的内容，这下你的服务已经可以上报数据了。

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#skywalking-%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE)[Skywalking 服务配置](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:skywalking-%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE)

程序中配置好了，那么我们把数据上报到哪里，以及如何查看呢？这里以开发环境为例子，毕竟如果在本地开发的话，还是需要调试的。

-   安装 skywalking

我们先去官网下载 [Skywalking APM](https://www.apache.org/dyn/closer.cgi/skywalking/9.2.0/apache-skywalking-apm-9.2.0.tar.gz)，下载完之后解压，本地我们不做特殊配置，直接双击 `apache-skywalking-apm-bin\bin\startup.bat` 就可以启动了。启动之后，访问 http://localhost:8080，就可以看到页面啦！

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-002.png)

> 启动完之后进去是没有数据的，等程序上报数据就好了。

-   安装 skywalking java agent

我们需要在程序启动时，让 agent 以插件的形式加载进来，当然也需要下载这些包了。去官网下载 [Skywalking Java Agent](https://www.apache.org/dyn/closer.cgi/skywalking/java-agent/8.13.0/apache-skywalking-java-agent-8.13.0.tgz)，然后解压之后得到 `skywalking-agent` 文件夹。点进去之后我们可以看到一个 jar 包 `skywalking-agent.jar`，记住这个包在你硬盘上的绝对路径。

由于 Spring Gateway 是基于 WebFlux 的，所以我们要添加 2 个插件进来。进入 `skywalking-agent\optional-plugins` 文件夹，找到如下 2 个 jar 包：

```
apm-spring-cloud-gateway-3.x-plugin-8.13.0.jar
apm-spring-webflux-5.x-plugin-8.13.0.jar
```

然后复制到 `skywalking-agent\plugins` 文件夹下就行了。

> 注意，具体的版本，取决于你当前所用的版本！

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#idea-%E9%85%8D%E7%BD%AE)[idea 配置](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:idea-%E9%85%8D%E7%BD%AE)

接下来我们去 idea 中配置，以便让 idea 中的 Java 服务启动时能顺利加载插件。

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-003.png)

我们先打开 `Run/Debug Configurations`，然后找到我们对应的服务（没有的就添加），选择 `Modify options`，勾选 `Environment variables` 和 `Add VM options` 选项。

然后添加对应的启动参数：

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-004.png)

第一个红框框的内容，是 `JVM Options` 参数：

```
-javaagent:D:\env\skywalking\skywalking-agent\skywalking-agent.jar
```

在 `-javaagent:` 后面的是你开发环境的 `skywalking-agent.jar` 包的绝对路径位置。

第二个红框框，是配置 `SkyWalking Agent` 的环境变量：

```
SW_AGENT_NAME=pisces-gateway;SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800
```

`SW_AGENT_NAME` 的值填写你要指定的 `Service Names`，如果你需要同时配置 `Service Groups`，你可以这样写：

```
SW_AGENT_NAME=Pisces-Cloud::pisces-gateway;
```

`::` 符号前面的就是 `Service Groups`，后面的就是 `Service Names`。

`SW_AGENT_COLLECTOR_BACKEND_SERVICES` 的值填写你的 `SkyWalking OAP` 的 `Agent Backend Service` 的地址。

## [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%9B%BE%E8%A1%A8%E4%BF%A1%E6%81%AF)[图表信息](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%9B%BE%E8%A1%A8%E4%BF%A1%E6%81%AF)

直播时基本上都介绍过了，这里挑几个典型大致讲讲吧。

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-005.png)

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-006.png)

图中我们看到的是服务概览，以及整个服务链路的拓扑图。当然，关于链路追踪重要的部分，还得是 Trace 和 Log，下面我以 `POST:/user/login` 这个登录的端点来举例子。

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E8%B0%83%E7%94%A8%E9%93%BE%E8%B7%AF%E4%BF%A1%E6%81%AF)[调用链路信息](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E8%B0%83%E7%94%A8%E9%93%BE%E8%B7%AF%E4%BF%A1%E6%81%AF)

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-007.png)

我们可以看到每一个上报过的端点，以及端点对应的信息。图中端点的请求走过的每一个不同的服务，都是用了不同的颜色区分开了。而竖着的直线，就是对应着当前的服务，中间的每一个跨度都能点开查看日志，包括执行的 SQL 语句，然后只要输出了日志信息，都可以关联到对应的 LOG，是不是特别方便？下面放一张全貌图。

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-008.png)

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E6%9C%8D%E5%8A%A1%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%9B%BE%E8%A1%A8)[服务业务数据图表](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E6%9C%8D%E5%8A%A1%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%9B%BE%E8%A1%A8)

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-009.png)

图中的注释已经说的很详细了，有一点需要注意的是，消息队列相关的那 2 个图表，需要你当前服务具有”消息消费者“的时候，才会显示数据的，毕竟是展示 `Consuming` 数据嘛。

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#jvm-%E6%95%B0%E6%8D%AE%E5%9B%BE%E8%A1%A8)[JVM 数据图表](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:jvm-%E6%95%B0%E6%8D%AE%E5%9B%BE%E8%A1%A8)

![](https://blog.besscroft.com/images/articles/2022/spring-microservice-monitoring-practice/spring-microservice-monitoring-practice-010.png)

每一个实例进去，都能够看到对应的 JVM 的信息，毕竟有时候需要诊断 JVM 数据来定位问题嘛。我们可以看到有内存/CPU占用情况，新生代、老年代的 gc 时间和次数，各种状态的线程（守护线程、阻塞线程、可运行线程、等待线程）次数啦等等。

毕竟对于初学者来说，学习使用各种工具去查看这些指标，是需要一定的精力的。尤其是碰到线上不给任何权限的那种，这时候程序能够主动上报数据，我们就能直观的看到了。

> 注意，线上环境的任何观测/上报数据的接口，都不建议对外公开访问，做好白名单/黑名单机制拦住它。

## [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%BC%80%E5%8F%91%E4%B8%AD%E7%9A%84%E6%B3%A8%E6%84%8F%E7%82%B9)[开发中的注意点](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%BC%80%E5%8F%91%E4%B8%AD%E7%9A%84%E6%B3%A8%E6%84%8F%E7%82%B9)

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E8%B7%A8%E7%BA%BF%E7%A8%8B%E8%BF%BD%E8%B8%AA)[跨线程追踪](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E8%B7%A8%E7%BA%BF%E7%A8%8B%E8%BF%BD%E8%B8%AA)

程序中教程会碰到，需要用异步线程处理的情况，那么这种时候如果需要对异步线程进行追踪的话，可以采用[跨线程追踪](https://skywalking.apache.org/docs/skywalking-java/v8.13.0/en/setup/service-agent/java-agent/application-toolkit-trace-cross-thread/)方案。

-   首先我们先引入包：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="xml"><span>&lt;dependency&gt;</span>
    <span>&lt;groupId&gt;</span>org.apache.skywalking<span>&lt;/groupId&gt;</span>
    <span>&lt;artifactId&gt;</span>apm-toolkit-trace<span>&lt;/artifactId&gt;</span>
    <span>&lt;version&gt;</span>${skywalking.version}<span>&lt;/version&gt;</span>
<span>&lt;/dependency&gt;</span>
</code></pre></td></tr></tbody></table>

-   拿下面这个场景来说，我需要发送一条消息：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="java"><span>CompletableFuture</span><span>.</span><span>runAsync</span><span>(()</span> <span>-&gt;</span> <span>{</span>
<span>messageSender</span><span>.</span><span>sendBark</span><span>(</span><span>String</span><span>.</span><span>format</span><span>(</span><span>"时间：%s，用户：%s 登录系统！"</span><span>,</span> <span>LocalDateTime</span><span>.</span><span>now</span><span>(),</span> <span>account</span><span>));</span>
<span>});</span>
</code></pre></td></tr></tbody></table>

这种写法是无法追踪到的，在引入包之后，我们可以这么写：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="java"><span>CompletableFuture</span><span>.</span><span>runAsync</span><span>(</span><span>RunnableWrapper</span><span>.</span><span>of</span><span>(()</span> <span>-&gt;</span> <span>{</span>
<span>messageSender</span><span>.</span><span>sendBark</span><span>(</span><span>String</span><span>.</span><span>format</span><span>(</span><span>"时间：%s，用户：%s 登录系统！"</span><span>,</span> <span>LocalDateTime</span><span>.</span><span>now</span><span>(),</span> <span>account</span><span>));</span>
<span>}));</span>
</code></pre></td></tr></tbody></table>

值得注意的是，`CompletableFuture.runAsync()` 是没有返回值的，所以用的是 `RunnableWrapper.of()`，如果我们用的是 `CompletableFuture.supplyAsync()` 的话，就需要换成支持返回值的 `SupplierWrapper.of()` 方法。

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E5%85%A8%E5%B1%80%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86)[全局异常处理](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E5%85%A8%E5%B1%80%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86)

全局异常处理这基本上业务开发都会用到，但是我们抛给前端的信息中，需要带上 traceId，这样不管是用户还是测试，都可以根据报错信息中的 traceId 反馈问题，排查也就减轻了一些负担。

-   首先引入相关的包：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="xml"><span>&lt;dependency&gt;</span>
    <span>&lt;groupId&gt;</span>org.apache.skywalking<span>&lt;/groupId&gt;</span>
    <span>&lt;artifactId&gt;</span>apm-toolkit-trace<span>&lt;/artifactId&gt;</span>
    <span>&lt;version&gt;</span>${skywalking.version}<span>&lt;/version&gt;</span>
<span>&lt;/dependency&gt;</span>
</code></pre></td></tr></tbody></table>

-   通过 Skywalking 手动 API，我们可以获取到 traceId：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="java"><span>String</span> <span>traceId</span> <span>=</span> <span>TraceContext</span><span>.</span><span>traceId</span><span>();</span>
</code></pre></td></tr></tbody></table>

-   然后加到异常处理中响应给前端：

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span></code></pre></td><td><pre tabindex="0"><code data-lang="xml">@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {
/**
     * 全局异常拦截 handleException
     */
    @ResponseBody
    @ExceptionHandler(Exception.class)
    public CommonResult<span>&lt;</span>?&gt; handleException(Exception ex) {
        log.error("全局异常信息.[异常原因={}]", ex.getMessage(), ex);
        return CommonResult.failed(HttpStatus.ERROR, "系统异常，请联系管理员！", null,"traceId=" + TraceContext.traceId());
    }
}
</code></pre></td></tr></tbody></table>

-   响应的异常信息如下：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="json"><span>{</span>
    <span>code:</span> <span>500,</span>
    <span>message:</span> <span>"系统异常，请联系管理员！"</span><span>,</span>
    <span>data:</span> <span>null,</span>
    <span>traceId:</span> <span>"traceId=7d2705aebedb40a1985af4ed20f569b0.510.16686682751090531"</span>
<span>}</span>
</code></pre></td></tr></tbody></table>

### [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#traceid-%E7%94%9F%E6%88%90%E8%A7%84%E5%88%99)[traceId 生成规则](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:traceid-%E7%94%9F%E6%88%90%E8%A7%84%E5%88%99)

生成的 traceId 一大串，怎么读懂它？（虽然只是个符号而已，但是我们也可以了解下）

先看源码：

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span></code></pre></td><td><pre tabindex="0"><code data-lang="java"><span>private</span> <span>static</span> <span>final</span> <span>String</span> <span>PROCESS_ID</span> <span>=</span> <span>UUID</span><span>.</span><span>randomUUID</span><span>().</span><span>toString</span><span>().</span><span>replaceAll</span><span>(</span><span>"-"</span><span>,</span> <span>""</span><span>);</span>
<span>private</span> <span>static</span> <span>final</span> <span>ThreadLocal</span><span>&lt;</span><span>IDContext</span><span>&gt;</span> <span>THREAD_ID_SEQUENCE</span> <span>=</span> <span>ThreadLocal</span><span>.</span><span>withInitial</span><span>(</span>
    <span>()</span> <span>-&gt;</span> <span>new</span> <span>IDContext</span><span>(</span><span>System</span><span>.</span><span>currentTimeMillis</span><span>(),</span> <span>(</span><span>short</span><span>)</span> <span>0</span><span>));</span>

<span>public</span> <span>static</span> <span>String</span> <span>generate</span><span>()</span> <span>{</span>
    <span>return</span> <span>StringUtil</span><span>.</span><span>join</span><span>(</span>
        <span>'.'</span><span>,</span>
        <span>PROCESS_ID</span><span>,</span>
        <span>String</span><span>.</span><span>valueOf</span><span>(</span><span>Thread</span><span>.</span><span>currentThread</span><span>().</span><span>getId</span><span>()),</span>
        <span>String</span><span>.</span><span>valueOf</span><span>(</span><span>THREAD_ID_SEQUENCE</span><span>.</span><span>get</span><span>().</span><span>nextSeq</span><span>())</span>
    <span>);</span>
<span>}</span>
</code></pre></td></tr></tbody></table>

我们知道，traceId 是由 2 个点分隔开的一串长字符串。第一个点前面是生成的 UUID，并替换掉了 `-` 符号；中间的三个数字，是当前线程的线程 ID；最后的内容是根据时间戳 \* 10000 + 当前线程中的 seq 生成的。UUID 生成后就不会变了，线程 ID 也是一样。

> 注意，这里的线程 ID，并不是请求线程的线程 ID，说它是当前实例的生成器的线程 ID 更合适。而且 UUID 和线程 ID，每一个不同的实例生成的也是不同的，只是每一个实例下相同。这点其实也很好验证，每个 Service 启动多个实例，然后来一波多线程压测就显而易见了。

## [](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#%E6%9C%80%E5%90%8E)[最后](https://blog.besscroft.com/articles/2022/spring-microservice-monitoring-practice/#contents:%E6%9C%80%E5%90%8E)

这是第一次做直播技术分享，也非常感谢大家的支持！虽然第一次看的人不多，但是也留下了录屏，而且依旧可以写博客发出来，算是一次不错的经历，以后也打算继续做直播技术了，偶尔直播技术分享，同时也欢迎大家多多跟我交流技术呀！

updatedupdated2022-11-172022-11-17

[Spring Cloud](https://blog.besscroft.com/tags/spring-cloud/) [Kubernetes](https://blog.besscroft.com/tags/kubernetes/) [Spring 微服务](https://blog.besscroft.com/tags/spring-%E5%BE%AE%E6%9C%8D%E5%8A%A1/) [SkyWalking](https://blog.besscroft.com/tags/skywalking/) [Spring 微服务监控](https://blog.besscroft.com/tags/spring-%E5%BE%AE%E6%9C%8D%E5%8A%A1%E7%9B%91%E6%8E%A7/)