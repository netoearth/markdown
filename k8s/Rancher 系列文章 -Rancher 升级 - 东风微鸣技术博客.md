## 概述[](https://ewhisper.cn/posts/35816/#%E6%A6%82%E8%BF%B0%20-4)

之前在 天翼云上用 4 台机器安装了一个 1 master（及 etcd) 3 node 的 K3S 集群，并在其上使用 Helm 安装了 Rancher 2.6.3 版本。

前几天发现 Rancher 官方推荐的最新版为：[v2.6.4](https://www.suse.com/zh-cn/suse-rancher/support-matrix/all-supported-versions/rancher-v2-6-4/)

所以决定先后对 Rancher 和 K3S 集群进行升级。

根据官方推荐，计划：

1.  将 Rancher 从 v2.6.3 升级到 v2.6.4
2.  将 K3S 集群从 v1.21.7+k3s1 升级到 v1.22.5+k3s2

本文为 Rancher 的升级记录。

## 相关信息[](https://ewhisper.cn/posts/35816/#%E7%9B%B8%E5%85%B3%E4%BF%A1%E6%81%AF%20-2)

本次升级的 Rancher 的基本信息为：

1.  Rancher v2.6.3
2.  使用 Helm 3, 在线安装
3.  使用 cert-manager(v1.7.1) + let’s encrypt 管理证书

## 升级步骤[](https://ewhisper.cn/posts/35816/#%E5%8D%87%E7%BA%A7%E6%AD%A5%E9%AA%A4%20-2)

### 一、备份运行 Rancher Server 的 Kubernetes 集群[](https://ewhisper.cn/posts/35816/#%E4%B8%80%E3%80%81%E5%A4%87%E4%BB%BD%E8%BF%90%E8%A1%8C%20-Rancher-Server-%20%E7%9A%84%20-Kubernetes-%20%E9%9B%86%E7%BE%A4)

使用 [备份应用程序](https://docs.rancher.cn/docs/rancher2.5/backups/back-up-rancher/_index) 来备份 Rancher。

如果在升级过程中出现问题，你将使用备份作为恢复点。

备份结果如下图：

[![Rancher 界面备份结果](https://pic-cdn.ewhisper.cn/img/2022/05/07/5af5436c3940fa2d3038bd22d70b29c7-20220507150906.png)](https://pic-cdn.ewhisper.cn/img/2022/05/07/5af5436c3940fa2d3038bd22d70b29c7-20220507150906.png "Rancher 界面备份结果")

[![对象存储中的备份对象](https://pic-cdn.ewhisper.cn/img/2022/05/07/32a20aa8e7b0f5e869a625de3ec81cb5-20220507151045.png)](https://pic-cdn.ewhisper.cn/img/2022/05/07/32a20aa8e7b0f5e869a625de3ec81cb5-20220507151045.png "对象存储中的备份对象")

### 二、更新 Helm Chart repository[](https://ewhisper.cn/posts/35816/#%E4%BA%8C%E3%80%81%E6%9B%B4%E6%96%B0%20-Helm-Chart-repository)

1.  更新本地 helm 缓存。
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>helm repo update<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
2.  获取用来安装 Rancher 的存储库名称。
    
    关于存储库及其区别，请参见 [Helm Chart Repositories](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/chart-options/_index)。
    
    -   Latest：推荐用于尝试最新功能
    -   Stable：推荐用于生产环境 (📝 我用的是这个）
    -   Alpha：即将发布的版本的实验性预览
    
    请将命令中的 `<CHART_REPO>`，替换为 `latest` ，`stable` 或 `alpha`。
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span>$</span><span> helm repo list</span><p>NAME                    URL<br>bitnami                 https://charts.bitnami.com/bitnami<br>grafana                 https://grafana.github.io/helm-charts<br>aliyuncs                https://apphub.aliyuncs.com<br>rancher-stable          http://rancher-mirror.oss-cn-beijing.aliyuncs.com/server-charts/stable<br>prometheus-community    https://prometheus-community.github.io/helm-charts</p></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>
    
3.  从 Helm chart 库中获取最新的 chart 来安装 Rancher。
    
    该命令将提取最新的 chart，并将其作为 `.tgz` 文件保存在当前目录中。可以通过添加 `--version=` 标记来获取要升级到特定版本的 chart。如下：
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>helm fetch rancher-stable/rancher --version=v2.6.4<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    

### 三、升级 Rancher[](https://ewhisper.cn/posts/35816/#%E4%B8%89%E3%80%81%E5%8D%87%E7%BA%A7%20-Rancher)

使用 Helm 升级 Rancher 的普通（互联网连接）安装。

从当前安装的 Rancher Helm chart 中获取用 `--set` 传递的值。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span>$</span><span> helm get values rancher -n cattle-system</span><br>USER-SUPPLIED VALUES:<br>hostname: rancher.ewhisper.cn<br>ingress:<br>  tls:<br>    source: letsEncrypt<br>replicas: 1<br>systemDefaultRegistry: registry.cn-hangzhou.aliyuncs.com<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

> 🐾 **Notes:**
> 
> 因为我的集群是测试或 Demo 用途，所以 `replicas` 设置为 1

将上一步中的所有值用–set key=value 追加到命令中。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>helm upgrade rancher rancher-stable/rancher \<br>  --namespace cattle-system \<br>  --<span>set</span> hostname=rancher.ewhisper.cn \<br>  --<span>set</span> ingress.tls.source=letsEncrypt \<br>  --<span>set</span> replicas=1 \<br>  --<span>set</span> systemDefaultRegistry=registry.cn-hangzhou.aliyuncs.com \<br>  --version=2.6.4<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 四、验证升级是否成功[](https://ewhisper.cn/posts/35816/#%E5%9B%9B%E3%80%81%E9%AA%8C%E8%AF%81%E5%8D%87%E7%BA%A7%E6%98%AF%E5%90%A6%E6%88%90%E5%8A%9F)

登录 Rancher，确认升级成功。

[![Rancher 升级 v2.6.4 成功](https://pic-cdn.ewhisper.cn/img/2022/05/07/825b628687de25718e5bb6ff190a6923-20220507153128.png)](https://pic-cdn.ewhisper.cn/img/2022/05/07/825b628687de25718e5bb6ff190a6923-20220507153128.png "Rancher 升级 v2.6.4 成功")

🎉🎉🎉

但是，验证过程中也发现几个问题，下面一一描述及解决。

## 升级后出现的问题[](https://ewhisper.cn/posts/35816/#%E5%8D%87%E7%BA%A7%E5%90%8E%E5%87%BA%E7%8E%B0%E7%9A%84%E9%97%AE%E9%A2%98)

-   helm 升级失败，报错 `rendered manifests contain a resource that already exists`
-   受管集群 `home-k3s` 无法连接。

### Helm 升级 Rancher 失败[](https://ewhisper.cn/posts/35816/#Helm-%20%E5%8D%87%E7%BA%A7%20-Rancher-%20%E5%A4%B1%E8%B4%A5)

#### 问题[](https://ewhisper.cn/posts/35816/#%E9%97%AE%E9%A2%98)

报错如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><code><span>Error:</span> UPGRADE FAILED: rendered manifests contain a resource that already exists. <br>Unable <span>to</span> <span>continue</span> <span>with</span> update: Secret <span>"bootstrap-secret"</span> <span>in</span> <span>namespace</span> <span>"cattle-system"</span> exists <span>and</span> cannot be imported <span>into</span> the current release: invalid ownership metadata; <br>label validation <span>error</span>: missing <span>key</span> <span>"app.kubernetes.io/managed-by"</span>: must be <span>set</span> <span>to</span> <span>"Helm"</span>; <br>annotation validation <span>error</span>: missing <span>key</span> <span>"meta.helm.sh/release-name"</span>: must be <span>set</span> <span>to</span> <span>"rancher"</span>; <br>annotation validation <span>error</span>: missing <span>key</span> <span>"meta.helm.sh/release-namespace"</span>: must be <span>set</span> <span>to</span> <span>"cattle-system"</span><br></code><p><i></i>VBNET</p></pre></td></tr></tbody></table>

#### 解决办法[](https://ewhisper.cn/posts/35816/#%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95%20-2)

GitHub 搜索相关 Issue, 发现是 [v2.6.4 的 Bug](https://github.com/rancher/rancher/issues/37060#issuecomment-1102157986), Workaround 措施：

首先删除密钥，然后再次运行 helm 安装：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl delete secret -n cattle-system bootstrap-secret<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>helm upgrade rancher rancher-stable/rancher \<br>  --namespace cattle-system \<br>  --<span>set</span> hostname=rancher.ewhisper.cn \<br>  --<span>set</span> ingress.tls.source=letsEncrypt \<br>  --<span>set</span> replicas=1 \<br>  --<span>set</span> systemDefaultRegistry=registry.cn-hangzhou.aliyuncs.com \<br>  --version=2.6.4<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

问题解决。

### 受管集群 `home-k3s` 无法连接[](https://ewhisper.cn/posts/35816/#%E5%8F%97%E7%AE%A1%E9%9B%86%E7%BE%A4%20-home-k3s-%20%E6%97%A0%E6%B3%95%E8%BF%9E%E6%8E%A5)

#### 问题[](https://ewhisper.cn/posts/35816/#%E9%97%AE%E9%A2%98%20-2)

升级后发现：受管集群 `home-k3s` 无法连接，如下图：

[![受管集群无法连接](https://pic-cdn.ewhisper.cn/img/2022/05/07/5f9bd6d147b754d98d4db409180f345d-20220507153453.png)](https://pic-cdn.ewhisper.cn/img/2022/05/07/5f9bd6d147b754d98d4db409180f345d-20220507153453.png "受管集群无法连接")

登录受管集群，查看 `cattle-cluster-agent` 的日志，发现报错提示 镜像的格式不对，拉取的为 x86\_64 格式的镜像。

这是因为前面 Helm 安装的时候增加了 `systemDefaultRegistry=registry.cn-hangzhou.aliyuncs.com` 这个参数，而 `registry.cn-hangzhou.aliyuncs.com` 镜像库只有 x86\_64 格式的镜像，没有 arm64 格式的镜像，而我的 `home-k3s` 是安装在 树莓派 4 上面的。

#### 解决办法[](https://ewhisper.cn/posts/35816/#%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95%20-3)

移除 Helm 的 `systemDefaultRegistry=registry.cn-hangzhou.aliyuncs.com` 配置，执行 upgrade, 如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><code>helm upgrade rancher rancher-stable/rancher \<br>  --namespace cattle-system \<br>  --set hostname=rancher.ewhisper.cn \<br>  --set ingress.tls.source=letsEncrypt \<br>  --set replicas=1<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

执行成功后，发现 Helm 的配置已变更，但是 Rancher 的 `systemDefaultRegistry` 却仍是 `registry.cn-hangzhou.aliyuncs.com`.

这里发现 Rancher 界面显示如下 - `set by env value`:

[![Rancher 界面 systemDefaultRegistry 显示](https://pic-cdn.ewhisper.cn/img/2022/05/07/8e7e6f16620049132e0516b45a444104-20220507154222.png)](https://pic-cdn.ewhisper.cn/img/2022/05/07/8e7e6f16620049132e0516b45a444104-20220507154222.png "Rancher 界面 systemDefaultRegistry 显示")

最终发现是配置在这里：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>management.cattle.io/v3</span><br><span>kind:</span> <span>Setting</span><br><span>metadata:</span><br>  <span>name:</span> <span>system-default-registry</span><br><span>customized:</span> <span>false</span><br><span>default:</span> <span>''</span><br><span>source:</span> <span>''</span><br><span>value:</span> <span>'registry.cn-hangzhou.aliyuncs.com'</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

删除这个 yaml 或将 `value` 改为：`value: ''`, 并重启 Rancher, 重启后生效，发现 `'registry.cn-hangzhou.aliyuncs.com'` 以被移除。

问题解决。

## 📚️参考文档[](https://ewhisper.cn/posts/35816/#%F0%9F%93%9A%EF%B8%8F%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%20-4)

-   [Support matrix - Rancher v2.6.4](https://www.suse.com/zh-cn/suse-rancher/support-matrix/all-supported-versions/rancher-v2-6-4/)
-   [升级指南 | Rancher 文档](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/upgrades/_index)
-   [Secret “bootstrap-secret” in namespace “cattle-system” exists and cannot be imported seen when upgrading/re-installing Rancher when bootstrap-secret is not created by Helm · Issue #37060 · rancher/rancher (github.com)](https://github.com/rancher/rancher/issues/37060)
-   [v2.6.4 Milestone (github.com)](https://github.com/rancher/rancher/milestone/255)