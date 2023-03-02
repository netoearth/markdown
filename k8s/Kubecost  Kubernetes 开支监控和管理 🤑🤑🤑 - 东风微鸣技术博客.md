> 👉️**URL:** [https://www.kubecost.com/](https://www.kubecost.com/)
> 
> 📝**Description:**
> 
> Kubeccost 为使用 Kubernetes 的团队提供实时成本可视化和洞察，帮助您持续降低云成本。

昨天浏览 Kubectl 插件的时候发现了 Kubecost，一看惊为天人啊，这个功能对于运营团队和 PM 团队领导来说太重要了。直接把监控数据换算成钱，而且明确告诉你钱花在哪个 namespace、哪个应用、哪个标签、哪个 deployment 下，明确告诉你那些钱花得值、哪些钱浪费了，有哪些办法可以减少浪费… 真的都是实打实的「降本」功能。

下面详细介绍一下。

## 亮点：监控和降低云成本[](https://ewhisper.cn/posts/57817/#%E4%BA%AE%E7%82%B9%EF%BC%9A%E7%9B%91%E6%8E%A7%E5%92%8C%E9%99%8D%E4%BD%8E%E4%BA%91%E6%88%90%E6%9C%AC)

Kubeccost 为使用 Kubernetes 的团队提供实时成本可视化和洞察，帮助您持续降低云成本。

[![Kubecost 概览页](https://pic-cdn.ewhisper.cn/img/2021/11/29/97b53e1fa3876907eeaa1ae03be2e9cb-kubecost-landing.svg)](https://pic-cdn.ewhisper.cn/img/2021/11/29/97b53e1fa3876907eeaa1ae03be2e9cb-kubecost-landing.svg "Kubecost 概览页")

## 产品功能[](https://ewhisper.cn/posts/57817/#%E4%BA%A7%E5%93%81%E5%8A%9F%E8%83%BD)

### 💰️ 成本分摊[](https://ewhisper.cn/posts/57817/#%F0%9F%92%B0%EF%B8%8F-%20%E6%88%90%E6%9C%AC%E5%88%86%E6%91%8A)

[![成本分配功能示意图](https://pic-cdn.ewhisper.cn/img/2021/11/29/d89f37b6435bee20a4aa5c15324b803d-cost-allocation.svg)](https://pic-cdn.ewhisper.cn/img/2021/11/29/d89f37b6435bee20a4aa5c15324b803d-cost-allocation.svg "成本分配功能示意图")

按 Kubernetes 概念划分成本，包括部署（Deployment）、服务（Service）、命名空间（Namespace）、标签（Label）等等。开销视图可以跨越单个视图中的多个集群或通过单个 API 端点。

### 📺 统一成本监控[](https://ewhisper.cn/posts/57817/#%F0%9F%93%BA-%20%E7%BB%9F%E4%B8%80%E6%88%90%E6%9C%AC%E7%9B%91%E6%8E%A7)

[![统一成本监控](https://pic-cdn.ewhisper.cn/img/2021/11/29/7fc43f8042f5f9a302a6c815598b2abc-unified-cost.svg)](https://pic-cdn.ewhisper.cn/img/2021/11/29/7fc43f8042f5f9a302a6c815598b2abc-unified-cost.svg "统一成本监控")

将 Kubernetes 的成本与任何外部云服务或基础设施的支出结合起来，就可以获得一个完整的图景。可以分摊外部成本，然后归因于任何 Kubernetes 概念，以实现综合支出。

### ⚖️ 成本优化方案[](https://ewhisper.cn/posts/57817/#%E2%9A%96%EF%B8%8F-%20%E6%88%90%E6%9C%AC%E4%BC%98%E5%8C%96%E6%96%B9%E6%A1%88)

在 **不牺牲绩效** 的前提下接受 **动态** 的建议。优先考虑关键基础设施或应用程序更改，以提高资源效率和可靠性。

### 🔔 开销警报和治理[](https://ewhisper.cn/posts/57817/#%F0%9F%94%94-%20%E5%BC%80%E9%94%80%E8%AD%A6%E6%8A%A5%E5%92%8C%E6%B2%BB%E7%90%86)

[![开销警报](https://pic-cdn.ewhisper.cn/img/2021/11/29/40a140d3c5689a7fdc9fcf2ab1e87632-alerts.svg)](https://pic-cdn.ewhisper.cn/img/2021/11/29/40a140d3c5689a7fdc9fcf2ab1e87632-alerts.svg "开销警报")

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code><span>alerts:</span> <span># Kubecost 产生关于群集数据的警报</span><br>    <span># 在命名空间 `kubecost` 的每日预算警报</span><br>  <span>-</span> <span>type:</span> <span>budget</span> <span># 支持: 预算, recurringUpdate, 等.</span><br>    <span>threshold:</span> <span>50</span> <span># 预算警报所需</span><br>    <span>window:</span> <span>daily</span> <span># 或 1d</span><br>    <span>aggregation:</span> <span>namespace</span><br>    <span>filter:</span> <span>kubecost</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

在成本超支和基础设施中断风险成为实时通知问题之前，迅速捕捉它们。通过集成像 PagerDuty 和 Slack 这样的工具来保留工程工作流。

而且可以运行在以下环境：

-   Azure
-   AWS
-   Google Cloud
-   内网环境
-   Kubernetes
-   本地部署

## 安装[](https://ewhisper.cn/posts/57817/#%E5%AE%89%E8%A3%85%20-27)

可以使用 Helm Chart 进行安装。

!\[\[K8S 实用工具之四 - kubectl 实用插件 #cost https github com kubecost kubectl-cost\]\]

安装见这里：《[K8S 实用工具之四 - kubectl 实用插件：cost](https://ewhisper.cn/posts/60907/#cost)》

部署完成后，访问 kubecost-cost-analyzer 的 9090 端口即可查看 UI，Ingress 方式或者 port-forward 都可以。

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl port-forward --namespace kubecost deployment/kubecost-cost-analyzer 9090<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 升级[](https://ewhisper.cn/posts/57817/#%E5%8D%87%E7%BA%A7)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>helm repo update &amp;&amp; helm upgrade kubecost kubecost/cost-analyzer -n kubecost<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 卸载[](https://ewhisper.cn/posts/57817/#%E5%8D%B8%E8%BD%BD%20-2)

也是 Helm：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>helm uninstall kubecost -n kubecost<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 🤑 实际效果展示[](https://ewhisper.cn/posts/57817/#%F0%9F%A4%91-%20%E5%AE%9E%E9%99%85%E6%95%88%E6%9E%9C%E5%B1%95%E7%A4%BA)

以我的集群为例，这展示的不是 UI，这展示的是白花花银子、绿油油的美元 💵 啊！

Kubecost 有以下几大菜单项，各个都是省钱能手。

[![Kubecost 菜单](https://pic-cdn.ewhisper.cn/img/2021/11/29/ba7adb5cfdc19729d4ffe0abd858efa6-image-20211129230359782.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/ba7adb5cfdc19729d4ffe0abd858efa6-image-20211129230359782.png "Kubecost 菜单")

1.  🏠️ Overview（总览）
2.  📊 Cost Allocation（成本分摊）
3.  🧰 资产
4.  💲 节流
5.  🛑 健康状态
6.  📃 报告
7.  🔔 开销警报

### 总览[](https://ewhisper.cn/posts/57817/#%E6%80%BB%E8%A7%88)

通过 port-forward 方式，访问 `http://localhost:9090`，首先的页面平平无奇：

Kubecost 第一屏：Cluster 集群 #1, 5 个节点，每月开销 138.39 美元。

[![Kubecost 第一屏](https://pic-cdn.ewhisper.cn/img/2021/11/29/73e65e4e9aa50fbd79d8fe1993919f59-image-20211129222616753.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/73e65e4e9aa50fbd79d8fe1993919f59-image-20211129222616753.png "Kubecost 第一屏")

点进去后，真正的大杀器来了：

[![Kubecost Overview - 1](https://pic-cdn.ewhisper.cn/img/2021/11/29/e9b9ee4977e338083a2d2f2afb8413e0-image-20211129222820673.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/e9b9ee4977e338083a2d2f2afb8413e0-image-20211129222820673.png "Kubecost Overview - 1")

-   左上角：识别到 5 条省钱小妙招，每月可以帮我节省 $93.64。🤑
    
-   右上角：每月开销 $138.39，成本效益 2.7％，97.3% 都让云厂商白嫖啦 😱
    
-   左下角：每月集群开销，基于资源价格的每月运行费率费用走势图，这里还可以拆分到：
    
    -   计算：
        
        [![计算开销](https://pic-cdn.ewhisper.cn/img/2021/11/29/95c31c5138bd81b4851ac27063da5cd2-image-20211129223510771.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/95c31c5138bd81b4851ac27063da5cd2-image-20211129223510771.png "计算开销")
        
    -   内存
        
        [![内存开销](https://pic-cdn.ewhisper.cn/img/2021/11/29/ddcac556ecda9c8612c4e5ea152b2323-image-20211129223540024.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/ddcac556ecda9c8612c4e5ea152b2323-image-20211129223540024.png "内存开销")
        
    -   存储
        
        [![存储](https://pic-cdn.ewhisper.cn/img/2021/11/29/2f7bfaa6adb2ccdc1e02188a4fb08c74-image-20211129223607810.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/2f7bfaa6adb2ccdc1e02188a4fb08c74-image-20211129223607810.png "存储")
        
    -   此外！点击右侧「CLUSTER METRICS」还可以直接跳转到我们熟悉的 Grafana 仪表板，查看监控指标和 💵 的具体联系。（下一篇再补充）
        
-   右下角：资源浪费率（🙊资源利用率），基于当前已购的资源和过去 7 天的用量
    
    -   计算：每月空跑 $105.10 😱
    -   内存：每月空跑 $20.51 😱
    -   存储：每月空跑 $9.10 😱
    -   刨去空跑，我的应用主要的消耗在内存的 $1.34，另外存储方面 System 用了 $1.53
    
    [![资源利用率](https://pic-cdn.ewhisper.cn/img/2021/11/29/687e0db66677dee2675ef803a8e16443-image-20211129223835616.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/687e0db66677dee2675ef803a8e16443-image-20211129223835616.png "资源利用率")
    

Overview 继续下拉，还是震惊：

[![Kubecost Overview - 2](https://pic-cdn.ewhisper.cn/img/2021/11/29/2f6c1cc2b07d349a0a5cac84cfb447c2-image-20211129224449149.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/2f6c1cc2b07d349a0a5cac84cfb447c2-image-20211129224449149.png "Kubecost Overview - 2")

-   左上角：基于 Controller 维度的成本分摊，根据过去 2 天的资源消耗，算出 Controller 的每月分摊成本，比如我的：
    -   `kubecost/deployment:kubecost-cost-analyzer` 成本占比 $8.37 (17.1%)
    -   `monitoring/statefulset:prometheus-prometheus-operator-prometheus` 成本占比 $5.83(11.9%)
    -   `monitoring/statefulset:alertmanager-prometheus-operator-alertmanager`: 成本占比 $2.81(5.7%)
-   右上角：基于 Service 维度的成本分摊，根据过去 2 天的资源消耗，算出 Service 的每月分摊成本，比如我的：
    -   `kubecost/kubecost-cost-analyzer` 成本占比 $8.37 (17.1%)
    -   …
-   左下角：基于 NameSpace 维度的成本分摊，以及成本效益评分（1-100 分），比如我的：
    -   kubecost：每月开销 $10.74，效益 22 分，不及格！😱
    -   monitoring：每月开销 $9.63，效益 42 分，不及格！😱
    -   crossplane-system：每月开销 $5.70，效益 5 分，战五渣！ 😱
    -   kube-system：每月开销 $2.71, 效益 17 分，不及格！😱
    -   loki-stack：每月开销 $0.66，满分！（Loki YYDS ？）💯
-   右下角：基础架构健康度，94 分（集群运行状况评级是对基础设施可靠性和性能风险的评估，分数范围从 1-100），属于花钱保平安了这是。😂

> ℹ️ **提示**：
> 
> 成本效益定义为 CPU 和 RAM 的(使用量 / request)。如果使用了资源，但没有 request 资源，那么效率被认为是无限的。

### 成本分摊[](https://ewhisper.cn/posts/57817/#%E6%88%90%E6%9C%AC%E5%88%86%E6%91%8A)

进入第二个菜单，成本分摊，效果如下：

[![成本分摊图表](https://pic-cdn.ewhisper.cn/img/2021/11/29/3fcf4be82c87b64a61d1a9b64a9d086e-image-20211129230926512.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/3fcf4be82c87b64a61d1a9b64a9d086e-image-20211129230926512.png "成本分摊图表")

细粒度到：CPU、GPU、RAM、PV、Network、LB、Shared。

另外，成本效益可以根据非常多的维度去进行分析，我想应该可以满足领导的需求：

[![成本效益归并的维度](https://pic-cdn.ewhisper.cn/img/2021/11/29/e663161899bf2f0e89514af982f8e407-image-20211129231629047.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/e663161899bf2f0e89514af982f8e407-image-20211129231629047.png "成本效益归并的维度")

### 资产[](https://ewhisper.cn/posts/57817/#%E8%B5%84%E4%BA%A7)

> ℹ️ **提示**：
> 
> 资产和公有云的信息对接后，可以获得更丰富的信息，如：云账号、供应商类型等。
> 
> 目前版本可以对接：AWS 和 GCP，但是是付费功能。

下图为资产信息：

[![资产](https://pic-cdn.ewhisper.cn/img/2021/11/29/9cce60ee5ff393fe89e3087ffb9ceb3c-image-20211129231945086.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/9cce60ee5ff393fe89e3087ffb9ceb3c-image-20211129231945086.png "资产")

也可以从多个维度拆分：

[![资产拆分维度](https://pic-cdn.ewhisper.cn/img/2021/11/29/a4a1be9a341fb5c276ecab69be9cd167-image-20211129232150862.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/a4a1be9a341fb5c276ecab69be9cd167-image-20211129232150862.png "资产拆分维度")

悬停到信息按钮，会告诉你计费单位：（计费单价可调整的）。如下图：

-   每小时 Node 开销为：$0.03733。

[![计费单位](https://pic-cdn.ewhisper.cn/img/2021/11/29/bfcd5357bee63f521525578c7714d3aa-image-20211129232239802.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/bfcd5357bee63f521525578c7714d3aa-image-20211129232239802.png "计费单位")

### 节流[](https://ewhisper.cn/posts/57817/#%E8%8A%82%E6%B5%81)

如下图，可以评估每月大概能省多少钱，节省的比例。以及具体的节流措施：

-   管理低利用率节点
-   本地磁盘利用率低
-   Pod 配置的 Request 太多
-   识别到潜在的不用的 wordload

[![节流](https://pic-cdn.ewhisper.cn/img/2021/11/29/6926141151c05b68bdf1e36040f83905-image-20211129234236139.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/6926141151c05b68bdf1e36040f83905-image-20211129234236139.png "节流")

### 健康状态[](https://ewhisper.cn/posts/57817/#%E5%81%A5%E5%BA%B7%E7%8A%B6%E6%80%81)

这个功能比较一般，就是类似 K9S 的 popeye。效果如下图：

[![Cluster Health](https://pic-cdn.ewhisper.cn/img/2021/11/29/c5ce083bb007856edc421f9ec1631643-image-20211129232802167.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/c5ce083bb007856edc421f9ec1631643-image-20211129232802167.png "Cluster Health")

-   有 Pod pending
-   Worker node 没有分散在多个可用区（这个提示不错👍️）
-   集群没有 master 副本

点进去还会有详细的指南，如下图：

[![多可用区指南](https://pic-cdn.ewhisper.cn/img/2021/11/29/6d058974c53f5add8c627b7c57dc90b1-image-20211129233018265.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/6d058974c53f5add8c627b7c57dc90b1-image-20211129233018265.png "多可用区指南")

### 报告[](https://ewhisper.cn/posts/57817/#%E6%8A%A5%E5%91%8A)

报告就是基于 成本 和 资产两个维度，根据上面的仪表来定制定期的报告。

[![Reports](https://pic-cdn.ewhisper.cn/img/2021/11/29/6101e8f5241c99696b0e8851c0a4f18a-image-20211129233153282.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/6101e8f5241c99696b0e8851c0a4f18a-image-20211129233153282.png "Reports")

### 成本警报[](https://ewhisper.cn/posts/57817/#%E6%88%90%E6%9C%AC%E8%AD%A6%E6%8A%A5)

成本警报就是告警，主要的区别是基于💵的维度：

-   反复出现类
-   成本效益类
-   预算类
-   开销变化类（如上文产品功能中的告警就是开销突然上升了 50%）
-   健康类
-   诊断类

[![告警分类](https://pic-cdn.ewhisper.cn/img/2021/11/29/677361c6bcea013dfa845fc65da86c33-image-20211129233336834.png)](https://pic-cdn.ewhisper.cn/img/2021/11/29/677361c6bcea013dfa845fc65da86c33-image-20211129233336834.png "告警分类")

## 设置[](https://ewhisper.cn/posts/57817/#%E8%AE%BE%E7%BD%AE)

定制化还挺全面的，说一些我认为实用的配置吧：

1.  配置 Label，比如：租户对应的 Label 是 Tenant，部门对应的 Label 是 Apartment，产品对应的 Label 是 app…
2.  价格类设置，可以设置：
    1.  折扣
    2.  共享开销比例、对应的 NS、Label 等
    3.  单价
    4.  货币

## 总结[](https://ewhisper.cn/posts/57817/#%E6%80%BB%E7%BB%93%20-7)

完整看下来，如果让运营团队和 PM 团队领导看到，一定会爱不释手的。🤑🤑🤑