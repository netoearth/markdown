作者：vivo 互联网服务器团队 - Wu Qinghua

本文从目前业界实现 Jenkins 的高可用的实现方案，分析各方案的优缺点，引入 vivo 目前使用的 Jenkins 高可用方案，以及目前 Jenkins 资源的调度方案的设计实践和目前的落地运行效果。

一、前言

现在的企业很多都在用 Jenkins 做持续集成，各个业务端都依靠 Jenkins，vivo Devops 也是使用 Jenkins 来进行持续构建，部署 Jenkins 服务时如何保障服务的高可用变得尤为重要。

下面是目前 Jenkins 存在的**一些问题**。

1.  Jenkins 本身是单体的，即只能有一个 Jenkins Master。虽然你也可以在多台机器上部署多个 Jenkins Master，但这些 Master 之间没有联系，都是各自把任务交给手下的 slaver 去执行，没有任何交集。  
    也许某个 master 下的 slaver 很忙，而另一个 master 下的 slaver 却很闲，资源得不到充分利用。
    
2.   当其中一个 slave 宕机之后，该 slave 上的运行的 job 任务没有版本重新进行分配，需要用户重新执行。并且 slave 节点离线之后没有通知管理员。
    
3.   当系统业务量比较大的时候业务请求集中在 Jenkins Master 上，会对 Jenkins 造成压力，甚至的造成 Jenkins 服务不可用。
    
4.   当有 job 任务在 jenkins Master 上队列排队的时候，Jenkins Master 宕机后，队列任务不可持久化。
    
5.  Jenkins Workspace 没有自动清理功能，会导致磁盘空间不足，任务执行不了的情况。
    

基于以上情况，vivo Devops 对 Jenkins 的部署架构进行优化搭建，并且配套了一套 Jenkins 资源调度系统用于管理 Jenkins 资源。

二、业界实现

目前业界也包含一些 Jenkins 高可用的设计方式，但是并不能完全的满足解决上述问题，比如：

2.1 方案一  Gearman + Jenkins

这是 OpenStack 团队使用的方案。这个方案使用了 gearman， gearman 是个任务分发框架。

需要在每个 Master 上安装好 gearman 的插件，并配置好能连接到 gearman server，同时在每个 Master 必须建立相同的 job。

之后运行任务的流程如下：

1.  gearman worker 运行在各个 Jenkins Master 中等待 gearman server 分发任务；
    
2.  gearman client 向 gearman server 发出运行 job 的请求；
    
3.  gearman server 通知各个 gearman worker 有任务拉，第一个闲着的 worker 会接受任务，如果所有的 worker 都忙，则放入 gearman 的任务队列，得 worker 空闲时再分配；
    
4.  gearman worker 闲下来后会从任务队列里取 job 来执行，执行完之后，将结果发回给 gearman server;
    
5.  gearman server 将结果返回给 gearman client。
    

优点：

这样各个 salver 资源可以得到充分利用，某个 master 挂掉另外的 master 可以继续服务。

弊端：

每个 master 的 slave 必须配置一致，否则会造成 job 调度错误，同时会造成一些资源的浪费。当一个 master 出现问题，该 master 的任务不会进行自动重新分配。

2.2 方案二 改造 Jenkins 的文件存储方式

目前 Jenkins 的配置文件都是直接在硬盘上以文件形式存储的，你在 JENKINS\_HOME 的个文件夹下能看到各种.xml 文件。有些公司在 Jenkins 上进行二次开发，将 Jenkins 的数据存储方式改为数据库存储，这样前端可以起多个 Jenkins 服务，后端连相同的数据库即可。数据库也有比较成熟的高可用方案。

优点:   

可以达到 Jenkins 的高可用也就是某个 master 挂掉另外的 master 可以继续服务。

弊端：

需要对 Jenkins 进行二次开发，使用数据库会降低读取资源效率下降。

2.3 方案三 最简单的 Jenkins 一主一备模式

平时让 Jenkins A 机器提供服务，并使用 SCM Sync configuration plugin 保存数据，JenkinsA 机器修改配置后触发 Jenkins B 更新配置，一旦 Jenkins A 出现问题挂掉后，切换到备机 Jenkins B 上。

优点:   

可以达到 Jenkins 的高可用，当 master 宕机后会进行切换到备机上。

弊端:   

会有一批 Jenkins 备机存在资源浪费，切换 master 时间过长，会导致有段时间 Jenkins 服务不可用。

三、vivo Jenkins Scheduler 系统目标

由于目前业界的一些实现还不能完全的满足我们目前的需求，所以我们进行了 vivo jenkins scheduler 系统的设计与实现。该系统需要达到如下的目的：

1.  **提升整个构建服务可靠性时长。**
    
    保证 jenkins 集群的高可用，解决目前 master-slave 的单点问题，保证整个构建服务的可靠性时长。  
    
2.  **降低灾难时服务恢复时长。**
    
    ①提供精准流控方式，在 jenkins 构建出现请求量过高的时候可以进行流控和持久化操作，减少对目前系统的冲击。
    
    ②当系统压力减少后，放开流控可以快速的对堆积的请求进行分配执行。
    
3.  **有效分配任务至各个子节点，**保证资源的有效利用。
    
4.  **能保证灾难时的及时切换任务至可用节点上**，同时能快速的通知管理员进行处理。
    
5.  **能进行数据的可视化分析**，能提供一系列帮助改善开发效率的视图，比如构建时长报表、构建量报表等。
    

四、 vivo Jenkins Scheduler 设计

该系统我们从两大部分进行了设计，首先，我们不采用原生的 Jenkins 部署方案，而是采用全 master 的方式。第二，设计并开发了一套用于管理 Jenkins 集群的调度系统。

五、底层 Jenkins 工具部署方案

不采用目前单 master 的搭建方案，采用多 master 的搭建方案，master 下不进行挂载 slave 机器，任务直接有 master 进行处理，master 之间的关系、任务分配、离线、插件安装等由调度系统进行管理。这样由于 vivo Jenkins Scheduler 系统为高可用的，解决了目前 Jenkins 的单点问题。

![](https://oscimg.oschina.net/oscnet/7511e502-2810-4ff0-a87e-0f4327cf6de4.png)

六、系统架构图

![](https://oscimg.oschina.net/oscnet/7bc1dc14-ffac-4d7b-84f5-8ddf0e3a84dd.png)

七、系统说明

7.1 API-Gateway

主要提供系统的外部请求，网关系统，功能包含：

-   **权限校验：**校验用户发送集群管理系统的请求的权限。
    
-   **智能路由：**接收外部一切请求，并转发到后端的外服上去。
    
-   **限流：**与监控线程配合（当构建请求达到某个阈值时），进行限流操作。
    
-   **API 日志统一收集：**类似于一个 aspect 切面，记录接口的进入和出去时的相关日志。
    
-   **数据处理：**对请求的参数进行数据的转换处理。
    

7.2 事件中心

是整个系统通信调用的主要模块，采用的是 Spring 的 Event 机制实现，主要核心事件如下：

1.  **Jenkins 注册事件**
    
    **（EVENT\_REGIST\_JENKINS）**：
    
    Jenkins 启动后，通过自定的插件会向系统发送注册请求时，系统接收到后会触发 Jenkins 管理模块将 Jenkins 的信息注册至调度系统中。
    
2.  **Jenkins 宕机事件** 
    
    **(EVENT\_DOWN\_JENKINS)**    : 
    
    监控管理轮询检查 Jenkins 状态，当发现有 Jenkins 宕机的情况会触发该事件，Jenkins 管理模块处理将 Jenkins 的信息状态设置为不可用状态，从而是任务不能分配至该台 jenkins。
    
3.  **任务从分配事件  (EVENT\_JOB\_REDO)** : 
    
    当 Jenkins 宕机后，如果该台 jenkins 上存在未执行完的任务时候，由 job 监控模块触发，job 管理莫管处理，会对该 Jenkins 上未执行的 job 进行重新分配。
    
4.  **任务接受事件  (EVENT\_JOB\_RECIVE)** :
    
    当 job 管理模块接受到创建请求，会触发该事件，由 job 管理模块放入 Redis 执行队列。
    
5.  **任务执行事件  (EVENT\_JOB\_EXECUTE)** : 
    
    job 管理模块中的执行线程 (10s 执行一次，会从 Redis 队列中弹出任务)，弹出任务后触发该事件，由调度中心选取合适的 jenkins 进行执行。
    

7.3 调度中心

是整个系统的核心模块，主要的功能是进行执行 job 时候能选取合适的 jenkins 进行处理任务，包含两个核心算法：

**7.3.1 Jenkins 分组算法**

每台 jenkins 都可以使用标签的方式，打上多个标签，比如 jenkins 可以构建 java 程序，使用的构建工具可以是 maven 和 gradle，这个时候我们就可以给其打上 java、maven、gradle 三个标签。

标签的维度主要有以下几个：

-   **标签配置：** 判断构建配置是否配置了标签，根据标签选择对应标签的 Jenkins，比如配置了（docker 等）。
    
-   **构建语言：** 根据构建配置的语言，比如 Java、C++、Python、Go 等。
    
-   **构建工具和版本：** 比如 Maven、gradle、Ant，Cmark、Blade 等。
    
-   **JDK 版本：**比如 JDK7、JDK8 等。
    
-   **Go 语言版本：**比如 1.15.x.、1.16.x 等。
    
-   **GCC 版本：**如 6.x、4.x 等。
    
-   **Python 版本：**2.x、3.x 等。
    
-   **是否存活：**判断 Jenkins 是否存活，如果宕机直接过滤。
    
-   **（可选策略）选择执行过该 job 的 Jenkins，减少下载代码的过程：**（第一次构建还是会比较慢，可以采用预执行的方式，在配置构建配置的时候，就预先执行一次，这样在用户执行的时候就使用该 job 执行过得 workspace，减少代码下载的时间）。
    
-   **（可选策略）根据 job 的构建的平均构建时长**，如果构建时长达到某个配置阈值时，优先选择构建器空闲多的 Jenkins 进行执行，并指出 Jenkins 的锁定功能。其他的 job 不允许分配上来。
    

如果我们给 Jenkins 打上标签，那么我们就可以使用标签为维度将 Jenkins 进行分组，并且存入至 Redis 中缓存，方便后续选取 Jenkins 用来执行任务：

![](https://oscimg.oschina.net/oscnet/203e00fb-8e2b-47bb-81c4-298ac8d89066.png)

**7.3.2 Jenkins 选取算法**

当 Jenkins 分组好了后，我们接受到执行的 job 的信息就可以使用 Jenkins 选取算法进行快速的选取合适的 Jenkins 进行处理 job，如下图所示。

**其中 label 子线程、语言子线程…… 就是我们上面的 Jenkins 分组的维度，有多少维度，那么这里就会有多少子线程处理。**

构建任务进入主线程，然后主线程会按照分组维度分组操作并进行过滤，然后获取到每个分组中合适的 Jenkins，再进行取交集（这个时候就获取到可以执行该构建任务的 Jenkins 了），在判断是否需要经过可选策略，最终得到 Jenkins。

![](https://oscimg.oschina.net/oscnet/670b5371-b415-4316-806a-7ba0d90a3d7d.png)

7.4 流控管理 & 队列管理

调度系统中的的任务接受采用的是队列的方式实现，当系统请求量达到阀后，系统将不会进入 Redis 队列，会将请求持久化至 MySQL。后续如果有请求过来，job 管理模块会检查数据库 MySQL 中是否有请求，如果有请求，会将请求放入 Redis 队列，如果没有请求就会将当前请求放入 Redis 队列，具体流程如下:

![](https://oscimg.oschina.net/oscnet/8a9dea12-9ca8-402b-a2e7-7ab5dd687d2c.png)

其中基于 Redis 实现的消息队列的时序图如下:

![](https://oscimg.oschina.net/oscnet/af4e1476-49de-4e32-9b7a-2a94a05cfc37.png)

7.5 回调中心

该模块主要是监控任务的状态，当任务开始执行、中断执行、执行成功、执行失败的时候进行通知业务并存储数据，用于保存构建记录，方便后续数据的统计，用来完成数据的可视化。

八、实施效果

目前该系统已经投入生产环境运行，Jenkins 任务已采用调度系统进行调度执行，运行稳定，运行效果。

![](https://oscimg.oschina.net/oscnet/6eef97e4-8bf4-4270-a906-23c4aaad8edc.png)

![](https://oscimg.oschina.net/oscnet/b5e753a2-50f4-428e-a7ee-bf12f290bad1.png)

九、后续展望

随着 vivo Jenkins 调度系统的功能慢慢完善，Jenkins 的机器也越来越多，目前还大多数运行在虚拟机上，从资源利用率和业务发布效率来看，未来的业务发布形态将会是以容器为主。目前公司也在大力发展 k8s 的容器生态建设，

所以我们希望将 Jenkins 工具后期进行容器化、池化，在提高资源利用率和发布效率的同时也可以为用户提供可靠的、简洁的、稳定调度执行。

END

猜你喜欢

-   [100 行 shell 写个 Docker](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI4NjY4MTU5Nw%3D%3D%26mid%3D2247495998%26idx%3D2%26sn%3D62815b2d66e21958b53db431c026a6ce%26chksm%3Debdb81acdcac08ba521dcb54dfcf11b75f4c0d33bbb077d04fb994a89b252a16fb8c542b29fc%26scene%3D21%23wechat_redirect)
    
-   [Dubbo 中 Zookeeper 注册中心原理分析](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI4NjY4MTU5Nw%3D%3D%26mid%3D2247495997%26idx%3D1%26sn%3Df165a2de458f21a6e5ce00e92cdd85df%26chksm%3Debdb81afdcac08b9ac790eb638f39f4f09622e3bc72a6a3ebf9174fc7d574afb3aeb243aac1d%26scene%3D21%23wechat_redirect)
    
-   [委派模式 —— 从 SLF4J 说起](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI4NjY4MTU5Nw%3D%3D%26mid%3D2247495986%26idx%3D2%26sn%3D9488faa16be8a970157c9257bbaf0bc6%26chksm%3Debdb81a0dcac08b6cc49eb1c7400017829cbd82c4459847e96d2c230c1fc6d73a167b68fe39d%26scene%3D21%23wechat_redirect)