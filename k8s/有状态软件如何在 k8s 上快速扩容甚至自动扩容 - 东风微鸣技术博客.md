## 概述[](https://ewhisper.cn/posts/4749/#%E6%A6%82%E8%BF%B0%20-12)

在传统的虚机 / 物理机环境里, 如果我们想要对一个有状态应用扩容, 我们需要做哪些步骤?

1.  申请虚机 / 物理机
2.  安装依赖
3.  下载安装包
4.  按规范配置主机名, hosts
5.  配置网络: 包括域名, DNS, 虚 ip, 防火墙…
6.  配置监控

今天虚机环境上出现了问题, 是因为 RabbitMQ 资源不足. 手动扩容的过程中花费了较长的时间.

但是在 K8S 上, 有状态应用的扩容就很简单, YAML 里改一下 `replicas` 副本数, 等不到 1min 就扩容完毕.

当然, 最基本的: 下镜像, 启动 pod(相当于上边的前 3 步), 就不必多提. 那么, 还有哪些因素, 让有状态应用可以在 k8s 上快速扩容甚至自动扩容呢?

**原因就是这两点:**

1.  peer discovery +peer discovery 的 相关实现(通过 hostname, dns, k8s api 或其他)
2.  可观察性 + 自动伸缩

我们今天选择几个典型的有状态应用, 一一梳理下:

1.  Eureka
2.  Nacos
3.  Redis
4.  RabbitMQ
5.  Kafka
6.  TiDB

在 Kubernetes 上, 有状态应用快速扩容甚至自动扩容很容易. 这得益于 Kubernetes 优秀的设计以及良好的生态. Kubernetes 就像是一个云原生时代的操作系统. 它自身就具有:

1.  自动化工具;
2.  内部服务发现 + 负载均衡
3.  内部 DNS
4.  和 Prometheus 整合
5.  统一的声明式 API
6.  标准, 开源的生态环境.

所以, 需要扩容, 一个 yaml 搞定全部. 包括上边提到的: 下载, 安装, 存储配置, 节点发现, 加入集群, 监控配置…

### Eureka 扩容[](https://ewhisper.cn/posts/4749/#Eureka-%20%E6%89%A9%E5%AE%B9)

[![eureka](https://pic-cdn.ewhisper.cn/img/2021/10/13/8825f9190e93dd6f6c2c94d079cde6d4-eureka.png)](https://pic-cdn.ewhisper.cn/img/2021/10/13/8825f9190e93dd6f6c2c94d079cde6d4-eureka.png "eureka")

> 🔖 备注:
> 
> 有状态扩容第一层:
> 
> StatefulSet + Headless Service

eureka 的扩容在 K8S 有状态应用中是最简单的, 就是:

**headless service + statefulset**

Eureka 要扩容, 只要 eureka 实例彼此能相互发现就可以. `headless service` 在这种情况下就派上用场了, 就是让彼此发现.

Eureka 的一个完整集群 yaml, 如下: 详细说明如下:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>Service</span><br><span>metadata:</span><br>  <span>name:</span> <span>eureka</span><br>  <span>namespace:</span> <span>ms</span><br><span>spec:</span><br>  <span>clusterIP:</span> <span>None</span><br>  <span>ports:</span><br>    <span>-</span> <span>name:</span> <span>eureka</span><br>      <span>port:</span> <span>8888</span><br>  <span>selector:</span><br>    <span>project:</span> <span>ms</span><br>    <span>app:</span> <span>eureka</span><br><span>---</span><br><span>apiVersion:</span> <span>apps/v1</span><br><span>kind:</span> <span>StatefulSet</span><br><span>metadata:</span><br>  <span>name:</span> <span>eureka</span><br>  <span>namespace:</span> <span>ms</span><br><span>spec:</span><br>  <span>serviceName:</span> <span>eureka</span><br>  <span>replicas:</span> <span>3</span><br>  <span>selector:</span><br>    <span>matchLabels:</span><br>      <span>project:</span> <span>ms</span><br>      <span>app:</span> <span>eureka</span><br>  <span>template:</span><br>    <span>metadata:</span><br>      <span>labels:</span><br>        <span>project:</span> <span>ms</span><br>        <span>app:</span> <span>eureka</span><br>    <span>spec:</span><br>      <span>terminationGracePeriodSeconds:</span> <span>10</span>   <br>      <span>imagePullSecrets:</span><br>      <span>-</span> <span>name:</span> <span>registry-pull-secret</span><br>      <span>containers:</span><br>        <span>-</span> <span>name:</span> <span>eureka</span><br>          <span>image:</span> <span>registry.example.com/kubernetes/eureka:latest</span><br>          <span>ports:</span><br>            <span>-</span> <span>protocol:</span> <span>TCP</span><br>              <span>containerPort:</span> <span>8888</span><br>          <span>env:</span><br>            <span>-</span> <span>name:</span> <span>APP_NAME</span><br>              <span>value:</span> <span>"eureka"</span><br>            <span>-</span> <span>name:</span> <span>POD_NAME</span><br>              <span>valueFrom:</span><br>                <span>fieldRef:</span><br>                  <span>fieldPath:</span> <span>metadata.name</span><br>            <span>-</span> <span>name:</span> <span>APP_OPTS</span><br>              <span>value:</span> <span>"</span><br><span>                     --eureka.instance.hostname=${POD_NAME}.${APP_NAME}</span><br><span>                     --registerWithEureka=true</span><br><span>                     --fetchRegistry=true</span><br><span>                     --eureka.instance.preferIpAddress=false</span><br><span>                     --eureka.client.serviceUrl.defaultZone=http://eureka-0.${APP_NAME}:8888/eureka/,http://eureka-1.${APP_NAME}:8888/eureka/,http://eureka-2.${APP_NAME}:8888/eureka/</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

1.  配置名为 `eureka` 的 Service
2.  在名为 `eureka` 的 statefulset 配置下, 共 3 个 eureka 副本, 每个 eureka 的 HOSTNAME 为: `${POD_NAME}.${SERVICE_NAME}`. 如: `eureka-0.eureka`
3.  彼此通过 `--registerWithEureka=true --fetchRegistry=true --eureka.instance.preferIpAddress=false --eureka.client.serviceUrl.defaultZone=http://eureka-0.${APP_NAME}:8888/eureka/,http://eureka-1.${APP_NAME}:8888/eureka/,http://eureka-2.${APP_NAME}:8888/eureka/` 通过 HOSTNAME 相互注册, 完成了集群的创建.

那么, 如果要快速扩容到 5 个:

1.  调整 StatefulSet: `replicas: 5`
2.  在环境变量 `APP_OPTS` 中加入新增的 2 个副本 hostname: `http://eureka-3.${APP_NAME}:8888/eureka/,http://eureka-4.${APP_NAME}:8888/eureka/`

即可完成.

#### Headless Service[](https://ewhisper.cn/posts/4749/#Headless-Service)

有时不需要或不想要负载均衡，以及单独的 Service IP。 遇到这种情况，可以通过指定 Cluster IP（`spec.clusterIP`）的值为 `None` 来创建 Headless Service。

您可以使用无头 Service 与其他服务发现机制进行接口，而不必与 Kubernetes 的实现捆绑在一起。

对这无头 Service 并不会分配 Cluster IP，kube-proxy 不会处理它们， 而且平台也不会为它们进行负载均衡和路由。 DNS 如何实现自动配置，依赖于 Service 是否定义了选择算符。

### Nacos[](https://ewhisper.cn/posts/4749/#Nacos)

[![nacos](https://pic-cdn.ewhisper.cn/img/2021/10/13/3e4b4d13c21e98e63296cc77937f3796-nacos.png)](https://pic-cdn.ewhisper.cn/img/2021/10/13/3e4b4d13c21e98e63296cc77937f3796-nacos.png "nacos")

> 🔖 备注:
> 
> 有状态扩容第二层:
> 
> StatefulSet + Headless Service + Init Container(自动化发现) + PVC

相比 Eureka, nacos 通过一个`init container`,(这个 init container, 就是一个自动化的 **peer discovery** 脚本.) , 实现了一行命令快速扩容:

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl scale sts nacos --replicas=3<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

脚本链接为: **[https://github.com/nacos-group/nacos-k8s/tree/master/plugin/peer](https://github.com/nacos-group/nacos-k8s/tree/master/plugin/peer)**

扩容的相关自动化操作为:

1.  从 Headless Service 自动发现所有的 replicas 的 HOSTNAME;
2.  并将 HOSTNAME 写入到: `${CLUSTER_CONF}` 这个文件下.
3.  `${CLUSTER_CONF}`这个文件就是 nacos 集群的所有 member 信息. 将新写入 HOSTNAME 的实例加入到 nacos 集群中.

在这里, 通过 Headless Service 和 PV/PVC(存储 nacos 插件或其他数据)，实现了对 Pod 的拓扑状态和存储状态的维护，从而让用户可以在 Kubernetes 上运行有状态的应用。

然而 Statefullset 只能提供受限的管理，通过 StatefulSet 我们还是需要编写复杂的脚本 (如 nacos 的`peer-finder` 相关脚本), 通过判断节点编号来区别节点的关系和拓扑，需要关心具体的部署工作。

### RabbitMQ[](https://ewhisper.cn/posts/4749/#RabbitMQ)

[![rabbitmq](https://pic-cdn.ewhisper.cn/img/2021/10/13/5b14848a3654b0099a325fad076e9657-RabbitMQ.sh-600x600.png)](https://pic-cdn.ewhisper.cn/img/2021/10/13/5b14848a3654b0099a325fad076e9657-RabbitMQ.sh-600x600.png "rabbitmq")

> 🔖 备注:
> 
> 有状态扩容第三层:
> 
> StatefulSet + Headless Service + 插件(自动化发现和监控) + PVC

RabbitMQ 的集群可以参考这边官方文档: **[Cluster Formation and Peer Discovery](https://www.rabbitmq.com/cluster-formation.html)**

这里提到的, 动态的发现机制需要依赖外部的服务, 如: DNS, API(AWS 或 K8S).

对于 Kubernetes, 使用的动态发现机制是基于 \*\*[rabbitmq-peer-discovery-k8s 插件](https://github.com/rabbitmq/rabbitmq-peer-discovery-k8s)\*\* 实现的.

通过这种机制，节点可以使用一组配置的值从 Kubernetes API 端点获取其对等方的列表：URI 模式，主机，端口以及令牌和证书路径。

另外, rabbitmq 镜像也默认集成了监控的插件 - **[rabbitmq\_prometheus](https://github.com/rabbitmq/rabbitmq-prometheus)**.

当然, 通过 `Helm Chart` 也能一键部署和扩容.

#### Helm Chart[](https://ewhisper.cn/posts/4749/#Helm-Chart)

一句话概括, Helm 之于 Kubernetes, 相当于 yum 之于 centos. 解决了依赖的问题. 将部署 rabbitmq 这么复杂的软件所需要的一大堆 yaml, 通过参数化抽象出必要的参数 (并且提供默认参数) 来快速部署.

### Redis[](https://ewhisper.cn/posts/4749/#Redis)

[![redis](https://pic-cdn.ewhisper.cn/img/2021/10/13/c7b4cfa90a668cb74f1271d720a2abfb-redis-logo.png)](https://pic-cdn.ewhisper.cn/img/2021/10/13/c7b4cfa90a668cb74f1271d720a2abfb-redis-logo.png "redis")

> 🔖 备注:
> 
> 有状态扩容第四层:
> 
> 通过 Operator 统一编排和管理:
> 
> Deployment(哨兵) + StatefulSet + Headless Service + Sidecar Container(监控) + PVC

这里以 UCloud 开源的: **[redis-operator](https://github.com/ucloud/redis-operator)** 为例. 它是基于 **哨兵模式** 的 redis 集群.

于之前的 StatefulSet + Headless 不同, 这里用到了一项新的 K8S 技术: `operator`.

#### Operator 原理[](https://ewhisper.cn/posts/4749/#Operator-%20%E5%8E%9F%E7%90%86)

> 📖 说明:
> 
> 解释 Operator 不得不提 Kubernetes 中两个最具价值的理念：“声明式 API” 和 “控制器模式”。“声明式 API”的核心原理就是当用户向 Kubernetes 提交了一个 API 对象的描述之后，Kubernetes 会负责为你保证整个集群里各项资源的状态，都与你的 API 对象描述的需求相一致。Kubernetes 通过启动一种叫做“控制器模式”的无限循环，WATCH 这些 API 对象的变化，不断检查，然后调谐，最后确保整个集群的状态与这个 API 对象的描述一致。
> 
> 比如 Kubernetes 自带的控制器：Deployment，如果我们想在 Kubernetes 中部署双副本的 Nginx 服务，那么我们就定义一个 repicas 为 2 的 Deployment 对象，Deployment 控制器 WATCH 到我们的对象后，通过控制循环，最终会帮我们在 Kubernetes 启动两个 Pod。
> 
> Operator 是同样的道理，以我们的 Redis Operator 为例，为了实现 Operator，我们首先需要将自定义对象的说明注册到 Kubernetes 中，这个对象的说明就叫 CustomResourceDefinition（CRD），它用于描述我们 Operator 控制的应用：redis 集群，这一步是为了让 Kubernetes 能够认识我们应用。然后需要实现自定义控制器去 WATCH 用户提交的 redis 集群实例，这样当用户告诉 Kubernetes 我想要一个 redis 集群实例后，Redis Operator 就能够通过控制循环执行调谐逻辑达到用户定义状态。

简单说, operator 可以翻译为: **运维人（操作员）** . 就是将高级原厂运维专家多年的经验, 浓缩为一个: `operator`. 那么, 我们所有的 **运维打工人** 就不需要再苦哈哈的 " 从零开始搭建 xxx 集群 ", 而是通过这个可扩展、可重复、标准化、甚至全生命周期运维管理的`operator`。 来完成复杂软件的安装，扩容，监控, 备份甚至故障恢复。

#### Redis Operator[](https://ewhisper.cn/posts/4749/#Redis-Operator)

使用 Redis Operator 我们可以很方便的起一个哨兵模式的集群，集群只有一个 Master 节点，多个 Slave 节点，假如指定 Redis 集群的 size 为 3，那么 Redis Operator 就会帮我们启动一个 Master 节点，两个 Salve 节点，同时启动三个 Sentinel 节点来管理 Redis 集群：

[![redis operator 哨兵架构](https://pic-cdn.ewhisper.cn/img/2021/10/13/4f9d1fbd57e21fead9131e74e3611b48-5830f84fc92db92f1cbf68ecface1767.png)](https://pic-cdn.ewhisper.cn/img/2021/10/13/4f9d1fbd57e21fead9131e74e3611b48-5830f84fc92db92f1cbf68ecface1767.png "redis operator 哨兵架构")

Redis Operator 通过 Statefulset 管理 Redis 节点，通过 Deployment 来管理 Sentinel 节点，这比管理裸 Pod 要容易，节省实现成本。同时创建一个 Service 指向所有的哨兵节点，通过 Service 对客户端提供查询 Master、Slave 节点的服务。最终，Redis Operator 控制循环会调谐集群的状态，设置集群的拓扑，让所有的 Sentinel 监控同一个 Master 节点，监控相同的 Salve 节点，Redis Operator 除了会 WATCH 实例的创建、更新、删除事件，还会定时检测已有的集群的健康状态，实时把集群的状态记录到 `spec.status.conditions` 中.

同时, 还提供了快速持久化, 监控, 自动化 redis 集群配置的能力. 只需一个 yaml 即可实现:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>redis.kun/v1beta1</span><br><span>kind:</span> <span>RedisCluster</span><br><span>metadata:</span><br>  <span>name:</span> <span>redis</span><br><span>spec:</span><br>  <span>config:</span>  <span># redis 集群配置</span><br>    <span>maxmemory:</span> <span>1gb</span><br>    <span>maxmemory-policy:</span> <span>allkeys-lru</span><br>  <span>password:</span> <span>sfdfghc56s</span>  <span># redis 密码配置</span><br>  <span>resources:</span>  <span># redis 资源配置</span><br>    <span>limits:</span><br>      <span>cpu:</span> <span>'1'</span><br>      <span>memory:</span> <span>1536Mi</span><br>    <span>requests:</span><br>      <span>cpu:</span> <span>250m</span><br>      <span>memory:</span> <span>1Gi</span><br>  <span>size:</span> <span>3</span>  <span># redis 副本数配置</span><br>  <span>storage:</span>  <span># 持久化存储配置</span><br>    <span>keepAfterDeletion:</span> <span>true</span><br>    <span>persistentVolumeClaim:</span><br>      <span>metadata:</span><br>        <span>name:</span> <span>redis</span><br>      <span>spec:</span><br>        <span>accessModes:</span><br>          <span>-</span> <span>ReadWriteOnce</span><br>        <span>resources:</span><br>          <span>requests:</span><br>            <span>storage:</span> <span>5Gi</span><br>        <span>storageClassName:</span> <span>nfs</span><br>        <span>volumeMode:</span> <span>Filesystem</span><br>  <span>sentinel:</span>   <span># 哨兵配置</span><br>    <span>image:</span> <span>'redis:5.0.4-alpine'</span>     <br>  <span>exporter:</span>  <span># 启用监控</span><br>    <span>enabled:</span> <span>true</span> <br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

要扩容也很简单, 将上边的 `size: 3` 按需调整即可. 调整后, 自动申请资源, 扩容, 加存储, 改 redis 配置, 加入 redis 集群, 并且自动添加监控.

### Kafka[](https://ewhisper.cn/posts/4749/#Kafka-2)

[![strimzi](https://pic-cdn.ewhisper.cn/img/2021/10/13/8495ccc539a0ad13cd8a0cf0023e096e-strimzi.jpg)](https://pic-cdn.ewhisper.cn/img/2021/10/13/8495ccc539a0ad13cd8a0cf0023e096e-strimzi.jpg "strimzi")

> 🔖 备注:
> 
> 有状态扩容第五层:
> 
> 通过 Operator 统一编排和管理多个有状态组件的:
> 
> StatefulSet + Headless Service + … + 监控

这里以 Strimzi 为例 - **[Strimzi Overview guide (0.20.0)](https://strimzi.io/docs/operators/latest/overview.html)**. 这是一个 Kafka 的 Operator.

提供了 Apache Kafka 组件以通过 Strimzi 发行版部署到 Kubernetes。 Kafka 组件通常以集群的形式运行以提高可用性。

包含 Kafka 组件的典型部署可能包括：

-   **Kafka** 代理节点集群集群
-   **ZooKeeper** - ZooKeeper 实例的集群
-   **Kafka Connect** 集群用于外部数据连接
-   **Kafka MirrorMaker** 集群可在第二个集群中镜像 Kafka 集群
-   **Kafka Exporter** 提取其他 Kafka 指标数据以进行监控
-   **Kafka Bridge** 向 Kafka 集群发出基于 HTTP 的请求

Kafka 的组件架构比较复杂, 具体如下:

[![img](https://pic-cdn.ewhisper.cn/img/2021/10/13/bbbc5e525fdfc04f6e1d7d1469927af8-37adb8a9f47039a38455b241f1b14bf9.png)](https://pic-cdn.ewhisper.cn/img/2021/10/13/bbbc5e525fdfc04f6e1d7d1469927af8-37adb8a9f47039a38455b241f1b14bf9.png "img")

通过 Operator, 一个 YAML 即可完成一套复杂的部署:

-   资源请求（CPU / 内存）
-   用于最大和最小内存分配的 JVM 选项
-   Listeners （和身份验证）
-   认证
-   存储
-   Rack awareness
-   监控指标

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br><span>87</span><br><span>88</span><br><span>89</span><br><span>90</span><br><span>91</span><br><span>92</span><br><span>93</span><br><span>94</span><br><span>95</span><br><span>96</span><br><span>97</span><br><span>98</span><br><span>99</span><br><span>100</span><br><span>101</span><br><span>102</span><br><span>103</span><br><span>104</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>kafka.strimzi.io/v1beta1</span><br><span>kind:</span> <span>Kafka</span><br><span>metadata:</span><br>  <span>name:</span> <span>my-cluster</span><br><span>spec:</span><br>  <span>kafka:</span><br>    <span>replicas:</span> <span>3</span><br>    <span>version:</span> <span>0.20</span><span>.0</span><br>    <span>resources:</span><br>      <span>requests:</span><br>        <span>memory:</span> <span>64Gi</span><br>        <span>cpu:</span> <span>"8"</span><br>      <span>limits:</span><br>        <span>memory:</span> <span>64Gi</span><br>        <span>cpu:</span> <span>"12"</span><br>    <span>jvmOptions:</span><br>      <span>-Xms:</span> <span>8192m</span><br>      <span>-Xmx:</span> <span>8192m</span><br>    <span>listeners:</span><br>      <span>-</span> <span>name:</span> <span>plain</span><br>        <span>port:</span> <span>9092</span><br>        <span>type:</span> <span>internal</span><br>        <span>tls:</span> <span>false</span><br>        <span>useServiceDnsDomain:</span> <span>true</span><br>      <span>-</span> <span>name:</span> <span>tls</span><br>        <span>port:</span> <span>9093</span><br>        <span>type:</span> <span>internal</span><br>        <span>tls:</span> <span>true</span><br>        <span>authentication:</span><br>          <span>type:</span> <span>tls</span><br>      <span>-</span> <span>name:</span> <span>external</span><br>        <span>port:</span> <span>9094</span><br>        <span>type:</span> <span>route</span><br>        <span>tls:</span> <span>true</span><br>        <span>configuration:</span><br>          <span>brokerCertChainAndKey:</span><br>            <span>secretName:</span> <span>my-secret</span><br>            <span>certificate:</span> <span>my-certificate.crt</span><br>            <span>key:</span> <span>my-key.key</span><br>    <span>authorization:</span><br>      <span>type:</span> <span>simple</span><br>    <span>config:</span><br>      <span>auto.create.topics.enable:</span> <span>"false"</span><br>      <span>offsets.topic.replication.factor:</span> <span>3</span><br>      <span>transaction.state.log.replication.factor:</span> <span>3</span><br>      <span>transaction.state.log.min.isr:</span> <span>2</span><br>      <span>ssl.cipher.suites:</span> <span>"TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"</span> <span>(17)</span><br>      <span>ssl.enabled.protocols:</span> <span>"TLSv1.2"</span><br>      <span>ssl.protocol:</span> <span>"TLSv1.2"</span><br>    <span>storage:</span> <br>      <span>type:</span> <span>persistent-claim</span><br>      <span>size:</span> <span>10000Gi</span><br>    <span>rack:</span><br>      <span>topologyKey:</span> <span>topology.kubernetes.io/zone</span><br>    <span>metrics:</span><br>      <span>lowercaseOutputName:</span> <span>true</span><br>      <span>rules:</span><br>      <span># Special cases and very specific rules</span><br>      <span>-</span> <span>pattern :</span> <span>kafka.server&lt;type=(.+),</span> <span>name=(.+),</span> <span>clientId=(.+),</span> <span>topic=(.+),</span> <span>partition=(.*)&gt;&lt;&gt;Value</span><br>        <span>name:</span> <span>kafka_server_$1_$2</span><br>        <span>type:</span> <span>GAUGE</span><br>        <span>labels:</span><br>          <span>clientId:</span> <span>"$3"</span><br>          <span>topic:</span> <span>"$4"</span><br>          <span>partition:</span> <span>"$5"</span><br>        <span># ...</span><br>  <span>zookeeper:</span><br>    <span>replicas:</span> <span>3</span><br>    <span>resources:</span><br>      <span>requests:</span><br>        <span>memory:</span> <span>8Gi</span><br>        <span>cpu:</span> <span>"2"</span><br>      <span>limits:</span><br>        <span>memory:</span> <span>8Gi</span><br>        <span>cpu:</span> <span>"2"</span><br>    <span>jvmOptions:</span><br>      <span>-Xms:</span> <span>4096m</span><br>      <span>-Xmx:</span> <span>4096m</span><br>    <span>storage:</span><br>      <span>type:</span> <span>persistent-claim</span><br>      <span>size:</span> <span>1000Gi</span><br>    <span>metrics:</span><br>      <span># ...</span><br>  <span>entityOperator:</span><br>    <span>topicOperator:</span><br>      <span>resources:</span><br>        <span>requests:</span><br>          <span>memory:</span> <span>512Mi</span><br>          <span>cpu:</span> <span>"1"</span><br>        <span>limits:</span><br>          <span>memory:</span> <span>512Mi</span><br>          <span>cpu:</span> <span>"1"</span><br>    <span>userOperator:</span><br>      <span>resources:</span><br>        <span>requests:</span><br>          <span>memory:</span> <span>512Mi</span><br>          <span>cpu:</span> <span>"1"</span><br>        <span>limits:</span><br>          <span>memory:</span> <span>512Mi</span><br>          <span>cpu:</span> <span>"1"</span><br>  <span>kafkaExporter:</span><br>    <span># ...</span><br>  <span>cruiseControl:</span><br>    <span># ...</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

当然, 由于 Kafka 的特殊性, 如果要将新增的 brokers 添加到现有集群, 还需要重新分区, 这里边涉及的更多操作详见: **[Scaling Clusters - Using Strimzi](https://strimzi.io/docs/operators/latest/full/using.html#scaling-clusters-deployment-configuration-kafka)**

### TiDB[](https://ewhisper.cn/posts/4749/#TiDB)

[![tidb](https://pic-cdn.ewhisper.cn/img/2021/10/13/6337722607a15dd9dc4cfeea050755bf-tidb.jpg)](https://pic-cdn.ewhisper.cn/img/2021/10/13/6337722607a15dd9dc4cfeea050755bf-tidb.jpg "tidb")

> 🔖 备注:
> 
> 有状态扩容第六层:
> 
> 通过 Operator 统一编排和管理多个有状态组件的:
> 
> StatefulSet + Headless Service + … + 监控 + TidbClusterAutoScaler(类似 HPA 的实现)
> 
> 甚至能做到备份和灾难恢复.

TiDB 更进一步, 可以实现 **有状态应用自动扩容**.

具体见这里: **[Enable TidbCluster Auto-scaling | PingCAP Docs](https://docs.pingcap.com/tidb-in-kubernetes/stable/enable-tidb-cluster-auto-scaling)**

Kubernetes 提供了`Horizontal Pod Autoscaler` ，这是一种基于 CPU 利用率的原生 API。 TiDB 4.0 基于 Kubernetes，实现了弹性调度机制。

只需要启用此功能即可使用:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code><span>features:</span><br>  <span>-</span> <span>AutoScaling=true</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

TiDB 实现了一个`TidbClusterAutoScaler CR` 对象用于控制 TiDB 集群中自动缩放的行为。 如果您使用过`Horizontal Pod Autoscaler` ，大概是您熟悉 TidbClusterAutoScaler 概念。 以下是 TiKV 中的自动缩放示例。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>pingcap.com/v1alpha1</span><br><span>kind:</span> <span>TidbClusterAutoScaler</span><br><span>metadata:</span><br>  <span>name:</span> <span>auto-scaling-demo</span><br><span>spec:</span><br>  <span>cluster:</span><br>    <span>name:</span> <span>auto-scaling-demo</span><br>    <span>namespace:</span> <span>default</span><br>  <span>monitor:</span><br>    <span>name:</span> <span>auto-scaling-demo</span><br>    <span>namespace:</span> <span>default</span><br>  <span>tikv:</span><br>    <span>minReplicas:</span> <span>3</span><br>    <span>maxReplicas:</span> <span>4</span><br>    <span>metrics:</span><br>      <span>-</span> <span>type:</span> <span>"Resource"</span><br>        <span>resource:</span><br>          <span>name:</span> <span>"cpu"</span><br>          <span>target:</span><br>            <span>type:</span> <span>"Utilization"</span><br>            <span>averageUtilization:</span> <span>80</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

需要指出的是: 需要向 `TidbClusterAutoScaler` 提供指标收集和查询(监控) 服务，因为它通过指标收集组件捕获资源使用情况。 monitor 属性引用 `TidbMonitor` 对象(其实就是自动化地配置 TiDB 的 prometheus 监控和展示等)。 有关更多信息，请参见 \*\* [使用 TidbMonitor 监视 TiDB 群集](https://docs.pingcap.com/tidb-in-kubernetes/stable/monitor-using-tidbmonitor)\*\*。

## 总结[](https://ewhisper.cn/posts/4749/#%E6%80%BB%E7%BB%93%20-22)

通过 6 个有状态软件, 我们见识到了层层递进的 K8S 上有状态应用的快速扩容甚至是自动扩容:

1.  最简单实现: StatefulSet + Headless Service – **Eureka**
2.  脚本 /Init Container 自动化实现: StatefulSet + Headless Service + Init Container(自动化发现) + PVC – **Nacos**
3.  通过插件实现扩容和监控:StatefulSet + Headless Service + 插件(自动化发现和监控) + PVC – **RabbitMQ**
4.  通过 Operator 统一编排和管理: – **Redis**
5.  对于复杂有状态, 是需要通过 Operator 统一编排和管理多个有状态组件的: – **Kafka**
6.  通过 Operator 统一编排和管理多个有状态组件的: – **TiDB**

😂😂😂 解放 **开发和运维打工人**, 是时候在 K8S 上部署有状态软件了! 💪💪💪