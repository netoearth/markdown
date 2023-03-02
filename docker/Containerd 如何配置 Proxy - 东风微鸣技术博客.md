## 前言[](https://ewhisper.cn/posts/36730/#%E5%89%8D%E8%A8%80%20-2)

在某些 air gap 场景中，往往需要离线或使用代理 (Proxy), 例如：

1.  需要通过 Proxy pull 容器镜像：
    1.  Docker Hub: `docker.io`
    2.  Quay: `quay.io`
    3.  GCR: `gcr.io`
    4.  GitHub 镜像库：`ghcr.io`
2.  在某些企业环境中，需要通过代理访问外部服务

Docker 如何配置代理想必大家都很清楚，但是自从 [Kubernetes 1.20 版本以后开始弃用 Docker](https://ewhisper.cn/posts/36509/), containerd 逐渐成为主流 CRI.  
所以我们下面介绍一下如何配置 contaienrd 的 Proxy.

> 📝**Notes:**
> 
> 还有一种场景需要 containerd 配置 proxy, 就是将 [Dragonfly 和 containerd 结合使用](https://d7y.io/docs/setup/runtime/containerd/proxy/) 的时候。

这里以通过 systemd 安装的 containerd 为例。

containerd 的配置一般位于 `/etc/containerd/config.toml` 下，service 文件位于：`/etc/systemd/system/containerd.service`  
配置 Proxy 可以通过 service 环境变量方式配置，具体如下：

创建或编辑文件：`/etc/systemd/system/containerd.service.d/http-proxy.conf`

内容如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>[Service]</span><br><span>Environment</span>=<span>"HTTP_PROXY=http://127.0.0.1:7890"</span><br><span>Environment</span>=<span>"HTTPS_PROXY=http://127.0.0.1:7890"</span><br><span>Environment</span>=<span>"NO_PROXY=localhost"</span><br></code><p><i></i>INI</p></pre></td></tr></tbody></table>

配置后保存重启即可：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>systemctl restart containerd.service<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 最佳实践：Proxy 中 `NO_PROXY` 的推荐配置[](https://ewhisper.cn/posts/36730/#%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%9AProxy-%20%E4%B8%AD%20-NO-PROXY-%20%E7%9A%84%E6%8E%A8%E8%8D%90%E9%85%8D%E7%BD%AE)

在配置 Proxy 时要特别注意，哪些要走 Proxy, 哪些不走 Proxy 要非常明确，避免出现网络访问异常甚至业务异常。

这里有个推荐 `NO_PROXY` 配置：

1.  本地地址和网段：`localhost` 和 `127.0.0.1` 或 `127.0.0.0/8`
2.  Kubernetes 的默认域名后缀：`.svc` 和 `.cluster.local`
3.  Kubernetes Node 的网段甚至所有应该不用 proxy 访问的 node 网段：`<nodeCIDR>`
4.  APIServer 的内部 URL: `<APIServerInternalURL>`
5.  Service Network: `<serviceNetworkCIDRs>`
6.  （如有）etcd 的 Discovery Domain: `<etcdDiscoveryDomain>`
7.  Cluster Network: `<clusterNetworkCIDRs>`
8.  其他特定平台相关网段（如 DevOps, Git/ 制品仓库。…): `<platformSpecific>`
9.  其他特定 `NO_PROXY` 网段：`<REST_OF_CUSTOM_EXCEPTIONS>`
10.  常用内网网段：
    1.  `10.0.0.0/8`
    2.  `172.16.0.0/12`
    3.  `192.168.0.0/16`

最终配置如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>[Service]</span><br><span>Environment</span>=<span>"HTTP_PROXY=http://127.0.0.1:7890"</span><br><span>Environment</span>=<span>"HTTPS_PROXY=http://127.0.0.1:7890"</span><br><span>Environment</span>=<span>"NO_PROXY=localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.svc,.cluster.local,.ewhisper.cn,&lt;nodeCIDR&gt;,&lt;APIServerInternalURL&gt;,&lt;serviceNetworkCIDRs&gt;,&lt;etcdDiscoveryDomain&gt;,&lt;clusterNetworkCIDRs&gt;,&lt;platformSpecific&gt;,&lt;REST_OF_CUSTOM_EXCEPTIONS&gt;"</span><br></code><p><i></i>INI</p></pre></td></tr></tbody></table>

🎉🎉🎉

## 总结[](https://ewhisper.cn/posts/36730/#%E6%80%BB%E7%BB%93%20-4)

Kubernetes 1.20 以上，企业 air gap 场景下可能会需要用到 containerd 配置 Proxy.  
本文介绍了其配置方法，以及配置过程中 `NO_PROXY` 的最佳实践。