## 开篇[](https://ewhisper.cn/posts/48546/#%E5%BC%80%E7%AF%87%20-5)

📜 **引言** ：

-   磨刀不误砍柴工
-   工欲善其事必先利其器

-   第一篇：《[K8S 实用工具之一 - 如何合并多个 kubeconfig？](https://ewhisper.cn/posts/46789/)》

像我这种，`kubectl` 用的不是非常溜，经常会碰到以下情况：

-   忘记命令，先敲 `--help`，再敲命令，效率低
-   忘记加 `-n` 指定 namespace
-   太长的命令经常记错或敲错，比如 `kubectl exec -it...`
-   无法快速将日志、yaml 复制出来
-   对于 CRD 类资源，记不住 CRD type，查不到相关信息
-   无法掌握集群的健康及监控状态
-   …

如果你的工作机（前置机、跳板机、操作机、堡垒机…）只是 Linux Shell，而没有桌面环境。那么我强烈推荐你使用这个 K8S 实用工具：终端 UI **K9S**。

K9S：K9s 是一个 **基于终端的 UI**，用于与 Kubernetes 集群进行交互。这个项目的目的是使导航、观察和管理已部署的应用程序变得更容易。K9s 持续监视 Kubernetes 的变化，并提供后续命令与观察到的资源进行交互。

[![k9s](https://pic-cdn.ewhisper.cn/img/2021/11/11/2c229ab93dd22d3791a06d482a56b89b-k9s.png)](https://pic-cdn.ewhisper.cn/img/2021/11/11/2c229ab93dd22d3791a06d482a56b89b-k9s.png "k9s")

## 🖌️ K9S 功能[](https://ewhisper.cn/posts/48546/#%F0%9F%96%8C%EF%B8%8F-K9S-%20%E5%8A%9F%E8%83%BD)

-   信息触手可及！
    -   跟踪 Kubernetes 集群中运行的资源的实时活动。
-   标准资源或 CRD？
    -   处理 Kubernetes 标准资源和自定义资源定义（即：CRD）。
-   集群指标
    -   跟踪与 pod、容器和节点（node）等资源相关的实时指标。
-   受到高级用户欢迎！
    -   提供标准的集群管理命令，如日志、伸缩、端口转发、重启等
    -   **定义您自己的命令快捷键**，通过命令别名和热键快速导航。
    -   k9s 支持插件扩展，以创建您自己的集群命令。
    -   强大的过滤模式，允许用户深入和查看与工作负载相关的资源。
-   错误钻取
    -   直接钻取群集资源的错误。
-   皮肤和可定制性
    -   通过 K9s 皮肤定义您自己的外观和感觉。
    -   自定义 / 排列要在每个资源基础上显示的列。
-   窄或宽?
    -   提供查看最小或完整资源定义的切换
-   多资源视图
    -   通过 **Pulses** 和 **XRay** 视图提供集群资源的概述。
-   我们拿到你的 RBAC 了！
    -   支持查看 RBAC 规则，如集群 / 角色及其关联绑定。
    -   反向查找断言用户 / 组或 ServiceAccount 在集群上可以做什么。
-   内置基准测试（Benchmarking）
    -   您可以直接从 K9s 对 HTTP 服务 /pod 进行基准测试，以查看应用程序的运行情况，并相应地调整资源请求 / 限制。
-   资源图遍历
    -   K9s 提供了 Kubernetes 资源及其关联资源的简单遍历。

## 🛠️ 安装[](https://ewhisper.cn/posts/48546/#%F0%9F%9B%A0%EF%B8%8F-%20%E5%AE%89%E8%A3%85)

直接从 [release](https://github.com/derailed/k9s/releases) 页面下载对应版本解压到 `/usr/local/bin` 即可。

## ⌨️ 命令[](https://ewhisper.cn/posts/48546/#%E2%8C%A8%EF%B8%8F-%20%E5%91%BD%E4%BB%A4)

安装后直接运行 `k9s`，就会进入 UI 界面，如下图：

[![k9s 首页](https://pic-cdn.ewhisper.cn/img/2021/11/11/04b5572dfe255bcc3c66e4fb2bc4e3fb-image-20211111232452195.png)](https://pic-cdn.ewhisper.cn/img/2021/11/11/04b5572dfe255bcc3c66e4fb2bc4e3fb-image-20211111232452195.png "k9s 首页")

### 👽️ 快捷键[](https://ewhisper.cn/posts/48546/#%F0%9F%91%BD%EF%B8%8F-%20%E5%BF%AB%E6%8D%B7%E9%94%AE)

| 操作 | 命令 | 备注 |
| --- | --- | --- |
| 显示活跃的键盘助记符和帮助 | `?` |  |
| 显示集群上所有可用的别名和资源 | `ctrl-a` or `:alias` |  |
| 退出 K9s | `:q`, `ctrl-c` |  |
| 使用单数 / 复数或短名称查看 Kubernetes 资源 | `:`po⏎ | 接受单数，复数，短名或别名如 `pod` 或 `pods` |
| 查看给定名称空间中的 Kubernetes 资源 | `:`alias namespace⏎ |  |
| 过滤出给定过滤器的资源视图 | `/`filter⏎ | 支持 Regex2，如 \` fred |
| 反向正则表达式过滤器 | `/`! filter⏎ | 保留所有 _不匹配_ 的东西。日志未实现。 |
| 按标签过滤资源视图 | `/`\-l label-selector⏎ |  |
| 模糊查找给定的资源 | `/`\-f filter⏎ |  |
| 退出视图 / 命令 / 过滤模式 | `<esc>` |  |
| 键映射来描述(describe)，查看(view)，编辑(edit)，查看日志(logs)，… | `d`,`v`, `e`, `l`,… |  |
| 查看并切换到另一个 Kubernetes 上下文 | `:`ctx⏎ |  |
| 查看并切换到另一个 Kubernetes 上下文 | `:`ctx context-name⏎ |  |
| 查看并切换到另一个 Kubernetes 名称空间 | `:`ns⏎ |  |
| 查看所有已保存的资源 | `:`screendump or sd⏎ |  |
| 要删除资源 (按`TAB` 键并输入`Enter`) | `ctrl-d` |  |
| 杀死一个资源(没有确认对话框!) | `ctrl-k` |  |
| 切换宽列 | `ctrl-w` | 等同于 `kubectl ... -o wide` |
| 切换错误状态 | `ctrl-z` | 查看有错误的资源 |
| 运行 pulses（脉冲）视图 | `:`pulses or pu⏎ |  |
| 运行 XRay（X 光）视图 | `:`xray RESOURCE \[NAMESPACE\]⏎ | 资源可以是以下之一：po, svc, dp, rs, sts, ds, NAMESPACE 参数可选 |
| 运行 Popeye（评估跑分） 视图 | `:`popeye or pop⏎ | 参阅 [https://popeyecli.io](https://popeyecli.io/) |

## 深度使用[](https://ewhisper.cn/posts/48546/#%E6%B7%B1%E5%BA%A6%E4%BD%BF%E7%94%A8)

### 快捷键[](https://ewhisper.cn/posts/48546/#%E5%BF%AB%E6%8D%B7%E9%94%AE)

花个 10 - 30 分钟熟悉快捷键，然后 —— 超级爽，各种快速进入、查找、切换、看 yaml、看日志、滚动日志、进 shell、编辑、复制 …

强烈推荐花时间熟悉，你会感觉效率飞升。🤓🤓🤓

### 过滤[](https://ewhisper.cn/posts/48546/#%E8%BF%87%E6%BB%A4)

它的过滤功能非常强大，使得你可以非常快速的定位资源，比如我想要看 traefik 的所有 CRD，操作如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>ctrl-<span>a</span><br>/traefik<br></code><p><i></i>CSS</p></pre></td></tr></tbody></table>

[![traefik crd](https://pic-cdn.ewhisper.cn/img/2021/11/11/c2a939670efa22bb82ef301f29a2203a-image-20211111234925565.png)](https://pic-cdn.ewhisper.cn/img/2021/11/11/c2a939670efa22bb82ef301f29a2203a-image-20211111234925565.png "traefik crd")

更多过滤功能，可以自己试一试，进一步研究。

### Pulses - 监控 Dashboard[](https://ewhisper.cn/posts/48546/#Pulses-%20%E7%9B%91%E6%8E%A7%20-Dashboard)

`:pulse` 就进入这个模式，这个就是一个监控 Dashboard，如下图：

[![Pulses](https://pic-cdn.ewhisper.cn/img/2021/11/11/d708e3e07cd9ecd59b01ac8641b4ab7e-image-20211111235352404.png)](https://pic-cdn.ewhisper.cn/img/2021/11/11/d708e3e07cd9ecd59b01ac8641b4ab7e-image-20211111235352404.png "Pulses")

可以非常直观看到集群现在的健康情况 —— 明显现在我的 Events 有异常，直接按 `5⏎`, 再按 `ctrl-z`查看异常事件：

[![Events](https://pic-cdn.ewhisper.cn/img/2021/11/12/65f360f8fef97840ebc6f521ce2ba8c6-image-20211111235655808.png)](https://pic-cdn.ewhisper.cn/img/2021/11/12/65f360f8fef97840ebc6f521ce2ba8c6-image-20211111235655808.png "Events")

### XRay[](https://ewhisper.cn/posts/48546/#XRay)

XRay 会提供以某个 Kubernetes 资源为维度的关联关系，像 X 光一样，透射到资源的内部。如下图：

[![XRay](https://pic-cdn.ewhisper.cn/img/2021/11/11/4537c60b970a849e76e6c355b18c2160-image-20211111235938637.png)](https://pic-cdn.ewhisper.cn/img/2021/11/11/4537c60b970a849e76e6c355b18c2160-image-20211111235938637.png "XRay")

以 traefik deployment 为例，位于 kube-system ns，启动了一个 `traefik-97b44b794-7qvzk` pod，pod 只有一个 `traefik` container，并通过 ServiceAccount `traefik` 挂载了 secret `traefik-token-r7vd2`。

### Popeye[](https://ewhisper.cn/posts/48546/#Popeye)

Popeye（大力水手）就是为集群、以及集群内的每隔资源打分，分数从 0 - 100，最后根据得分评出你的集群的情况：得分是 A 还是 C，并给出具体原因。

如下：

[![popeye](https://pic-cdn.ewhisper.cn/img/2021/11/12/39c6adab49035e12abd2c30d3d76216a-image-20211112000535421.png)](https://pic-cdn.ewhisper.cn/img/2021/11/12/39c6adab49035e12abd2c30d3d76216a-image-20211112000535421.png "popeye")

DaemonSet 得 0 分原因是都没指定 requests 和 limits：

[![DaemonSet 0 分原因](https://pic-cdn.ewhisper.cn/img/2021/11/12/9e81b367caa99f48782f68b7fc18b6ed-image-20211112000721855.png)](https://pic-cdn.ewhisper.cn/img/2021/11/12/9e81b367caa99f48782f68b7fc18b6ed-image-20211112000721855.png "DaemonSet 0 分原因")

Service 得 20 分原因就多了，甚至还贴心考虑到了开销贵不贵的问题：

[![Service 20 分 原因](https://pic-cdn.ewhisper.cn/img/2021/11/12/f4da4efb7c16ecb867152fa3ea63de53-image-20211112000919341.png)](https://pic-cdn.ewhisper.cn/img/2021/11/12/f4da4efb7c16ecb867152fa3ea63de53-image-20211112000919341.png "Service 20 分 原因")

### 🔐 直观的 RBAC[](https://ewhisper.cn/posts/48546/#%F0%9F%94%90-%20%E7%9B%B4%E8%A7%82%E7%9A%84%20-RBAC)

RBAC 的 yaml 看起来很不方便的，如果对权限比较要求比较多，那 K9S 绝对好用直观，如下，traefik role 有哪些权限一目了然：who、what、how。

[![RBAC 视图](https://pic-cdn.ewhisper.cn/img/2021/11/12/fb2524792e53b2327079a208c35524d7-image-20211112001218010.png)](https://pic-cdn.ewhisper.cn/img/2021/11/12/fb2524792e53b2327079a208c35524d7-image-20211112001218010.png "RBAC 视图")

## ✍ 总结[](https://ewhisper.cn/posts/48546/#%E2%9C%8D-%20%E6%80%BB%E7%BB%93%20-2)

K9S 是一个 **基于终端的 K8S UI**，在没有桌面、只有 终端的情况下使用它，可以大幅提升你的效率以及你对 K8S 的认知。

它有很多强大的功能，其中：快捷键、过滤、Pulses、XRay、Popeye、RBAC 这些功能一定要试一试，体验飞升！

一起使用吧~ 🤓🤓🤓