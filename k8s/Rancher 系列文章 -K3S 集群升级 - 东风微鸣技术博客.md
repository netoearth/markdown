## 概述[](https://ewhisper.cn/posts/13493/#%E6%A6%82%E8%BF%B0)

书接上回：《[Rancher 系列文章 -Rancher 升级](https://ewhisper.cn/posts/35816/)》, 我们提到：将 Rancher 用 Helm 从 v2.6.3 升级到 [v2.6.4](https://ewhisper.cn/posts/13493/(https://www.suse.com/zh-cn/suse-rancher/support-matrix/all-supported-versions/rancher-v2-6-4/)).

接下来开始进行 K3S 集群的升级：将 K3S 集群从 v1.21.7+k3s1 升级到 v1.22.5+k3s2

## 相关信息[](https://ewhisper.cn/posts/13493/#%E7%9B%B8%E5%85%B3%E4%BF%A1%E6%81%AF)

本次升级的 K3S 集群的基本信息为：

1.  天翼云上用 4 台机器安装的一个 1 master（及 etcd) 3 node 的 K3S 集群
2.  其实… 这个 K3S 集群使用 [k3s-ansible](https://gitee.com/caseycui/k3s-ansible) 脚本批量安装的。…
3.  K3S v1.21.7+k3s1
4.  Rancher 刚升级到 v2.6.4, 验证没啥大问题
5.  K3S 集群有用到 Traefik 管理 Ingress
6.  K3S 集群使用嵌入式 etcd 数据存储

## 升级方式评估[](https://ewhisper.cn/posts/13493/#%E5%8D%87%E7%BA%A7%E6%96%B9%E5%BC%8F%E8%AF%84%E4%BC%B0)

[官方](https://docs.rancher.cn/docs/k3s/upgrades/_index/) 提供了以下几种升级方式：

-   基础升级
    -   使用安装脚本升级 K3s
    -   使用二进制文件手动升级 K3s
-   自动升级
    -   使用 Rancher 来升级 K3s 集群
    -   使用 system-upgrad-controller 来管理 K3s 集群升级

我大概都过了一下，先说 Pass 的原因：

### 使用 Rancher 来升级 K3s 集群 - 🙅♂️[](https://ewhisper.cn/posts/13493/#%E4%BD%BF%E7%94%A8%20-Rancher-%20%E6%9D%A5%E5%8D%87%E7%BA%A7%20-K3s-%20%E9%9B%86%E7%BE%A4%20-%F0%9F%99%85%E2%80%8D%E2%99%82%EF%B8%8F)

详细的文档在这里：[升级 Kubernetes 版本 | Rancher | Rancher 文档](https://docs.rancher.cn/docs/rancher2.5/cluster-admin/upgrading-kubernetes/_index/#%E5%8D%87%E7%BA%A7-kubernetes-%E7%89%88%E6%9C%AC)

原文如下：

> 📚️ **Quote:**
> 
> > **先决条件：**
> > 
> > -   以下选项仅适用于 [RKE 集群](https://docs.rancher.cn/docs/rancher2.5/cluster-provisioning/rke-clusters/_index) 和 [导入的 K3s Kubernetes 集群](https://docs.rancher.cn/docs/rancher2/cluster-provisioning/imported-clusters/_index)。
> > -   升级 Kubernetes 之前，请 [备份您的集群](https://docs.rancher.cn/docs/rancher2.5/backups/_index)。
> 
> 1.  从 **全局** 视图中，找到要升级 Kubernetes 版本的集群。选择 **省略号 > 编辑**。
> 2.  点开 **集群选项**。
> 3.  从 **Kubernetes 版本** 下拉菜单中，选择要用于集群的 Kubernetes 版本。
> 4.  单击 **保存**。
> 
> **结果：** 集群开始升级 Kubernetes 版本。

但是，但是！我在我的 Rancher v2.6.4 上始终没找到 `省略号 > 编辑` 在哪里，😂😂😂

我猜可能是因为我看的中文文档只有 Rancher v2.5 的，而 Rancher v2.6 UI 又经过了很大的调整，所以找不到了。

另外，这种 Rancher 的 local 集群，而且还是 **单 master** 节点，我个人评估是无法实现 **自动升级** 的。

PAAS

### 使用 system-upgrad-controller 来管理 K3s 集群升级 - 🙅♂️[](https://ewhisper.cn/posts/13493/#%E4%BD%BF%E7%94%A8%20-system-upgrad-controller-%20%E6%9D%A5%E7%AE%A1%E7%90%86%20-K3s-%20%E9%9B%86%E7%BE%A4%E5%8D%87%E7%BA%A7%20-%F0%9F%99%85%E2%80%8D%E2%99%82%EF%B8%8F)

详细文档见这里：[自动升级 | Rancher 文档](https://docs.rancher.cn/docs/k3s/upgrades/automated/_index)

我试了一下，结果就是在我创建了 `server-plan` 后，提示我 `server-plan` 的 POD 无法进行调度，因为所有节点都不满足调度的条件。

我大概看了一下，调度的条件是要求在 `master` 节点上，同时我只有 1 个 master, 其在升级前已经设置为了 `cordon: true`, 导致冲突，升级无法进行。

也正是因为这个，所以我判断：

-   **单 master** 节点，是无法实现 **自动升级** 的, 或者即使可以进行升级, 风险也较大

PAAS

### 使用二进制文件手动升级 K3s - 🙅♂️[](https://ewhisper.cn/posts/13493/#%E4%BD%BF%E7%94%A8%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6%E6%89%8B%E5%8A%A8%E5%8D%87%E7%BA%A7%20-K3s-%F0%9F%99%85%E2%80%8D%E2%99%82%EF%B8%8F)

这个还行，步骤也很清晰明了, 也正好可以在 [k3s-ansible](https://gitee.com/caseycui/k3s-ansible) 脚本增加 `upgrade.yml` playbook 来实现。

但是… 近期没时间，先记下这个事吧，后面有时间再增加这个功能。

### 使用安装脚本升级 K3s - ✔️[](https://ewhisper.cn/posts/13493/#%E4%BD%BF%E7%94%A8%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC%E5%8D%87%E7%BA%A7%20-K3s-%E2%9C%94%EF%B8%8F)

虽然我不是用安装脚本安装的 K3s, 但是 [k3s-ansible](https://gitee.com/caseycui/k3s-ansible) 脚本的逻辑基本上和官方的安装脚本是一样的，只是用的是 ansible 而已。个人评估后认为：只要确保 **使用相同的标志重新运行安装脚本** 即可从旧版本升级 K3s.

就决定是你啦 ✔️

## 升级步骤[](https://ewhisper.cn/posts/13493/#%E5%8D%87%E7%BA%A7%E6%AD%A5%E9%AA%A4)

### 〇、信息收集[](https://ewhisper.cn/posts/13493/#%E3%80%87%E3%80%81%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86)

#### `registries.yaml`[](https://ewhisper.cn/posts/13493/#registries-yaml)

有配置 registries.yaml, 如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span>mirrors:</span><br>  <span>docker.io:</span><br>    <span>endpoint:</span><br>      <span>-</span> <span>"https://registry.cn-hangzhou.aliyuncs.com"</span><br>      <span>-</span> <span>"https://docker.mirrors.ustc.edu.cn"</span><br><span>configs:</span><br>  <span>'docker.io':</span><br>    <span>auth:</span><br>      <span>username:</span> <span>caseycui</span><br>      <span>password:</span> <span>&lt;my-password&gt;</span><br>  <span>'quay.io':</span><br>    <span>auth:</span><br>      <span>username:</span> <span>east4ming</span><br>      <span>password:</span> <span>&lt;my-password&gt;</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

但是位置没有动，还是 `/etc/rancher/k3s/registries.yaml`. 所以不会因此导致有额外的升级步骤。

#### K3s Server 和 Agent 其他配置[](https://ewhisper.cn/posts/13493/#K3s-Server-%20%E5%92%8C%20-Agent-%20%E5%85%B6%E4%BB%96%E9%85%8D%E7%BD%AE)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code><span>---</span><br><span>k3s_version:</span> <span>v1.21.7+k3s1</span><br><span>ansible_user:</span> <span>caseycui</span><br><span>systemd_dir:</span> <span>/etc/systemd/system</span><br><span>master_ip:</span> <span>"<span>{{ hostvars[groups['master'][0]]['ansible_host'] | default(groups['master'][0]) }}</span>"</span><br><span>extra_server_args:</span> <span>'--write-kubeconfig-mode "644" --cluster-init --disable-cloud-controller --tls-san &lt;my-public-ip&gt; --kube-apiserver-arg "feature-gates=EphemeralContainers=true" --kube-scheduler-arg "feature-gates=EphemeralContainers=true" --kube-apiserver-arg=default-watch-cache-size=1000 --kube-apiserver-arg=delete-collection-workers=10 --kube-apiserver-arg=event-ttl=30m --kube-apiserver-arg=max-mutating-requests-inflight=800 --kube-apiserver-arg=max-requests-inflight=1600 --etcd-expose-metrics=true'</span><br><span>extra_agent_args:</span> <span>''</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

分析上面配置，就是多了一些 server 的安装配置参数而已，使用官方安装脚本时注意确保 **使用相同的标志重新运行安装脚本** 即可。

### 一、备份[](https://ewhisper.cn/posts/13493/#%E4%B8%80%E3%80%81%E5%A4%87%E4%BB%BD)

使用 `k3s etcd-snapshot` 进行备份，如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code><span>#</span><span> k3s etcd-snapshot</span><br>INFO[2022-05-05T17:10:01.884597095+08:00] Managed etcd cluster bootstrap already complete and initialized<br>W0505 17:10:02.477542 2431147 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition<br>W0505 17:10:02.923819 2431147 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition<br>W0505 17:10:03.398185 2431147 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition<br>INFO[2022-05-05T17:10:03.687171696+08:00] Saving etcd snapshot to /var/lib/rancher/k3s/server/db/snapshots/on-demand-4azlmvglqkx7migt-0002-1651741803<br>{"level":"info","msg":"created temporary db file","path":"/var/lib/rancher/k3s/server/db/snapshots/on-demand-4azlmvglqkx7migt-0002-1651741803.part"}<br>{"level":"info","ts":"2022-05-05T17:10:03.693+0800","caller":"clientv3/maintenance.go:200","msg":"opened snapshot stream; downloading"}<br>{"level":"info","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}<br>{"level":"info","ts":"2022-05-05T17:10:14.841+0800","caller":"clientv3/maintenance.go:208","msg":"completed snapshot read; closing"}<br>{"level":"info","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"327 MB","took":"11.182733612s"}<br>{"level":"info","msg":"saved","path":"/var/lib/rancher/k3s/server/db/snapshots/on-demand-4azlmvglqkx7migt-0002-1651741803"}<br>INFO[2022-05-05T17:10:14.879646814+08:00] Saving current etcd snapshot set to k3s-etcd-snapshots ConfigMap<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

> 📝**Notes**:
> 
> 也可以增加更多的参数以将数据备份到 s3 中。  
> 之所以这次没选择，是因为集群的互联网带宽太小，备份到 s3 频繁中断所以放弃。

备份结果位于：`/var/lib/rancher/k3s/server/db/snapshots/`, 如下图：

[![K3s 备份结果](https://pic-cdn.ewhisper.cn/img/2022/05/07/f6b28d70a7cbbb9d13bda2d0ab2bf3b6-20220507173732.png)](https://pic-cdn.ewhisper.cn/img/2022/05/07/f6b28d70a7cbbb9d13bda2d0ab2bf3b6-20220507173732.png "K3s 备份结果")

### 二、`k3s-killall.sh`[](https://ewhisper.cn/posts/13493/#%E4%BA%8C%E3%80%81k3s-killall-sh)

为了保证升级成功率，且当前 K3s 集群主要用于测试和 Demo, 完全可以停机，所以使用 `k3s-killall.sh` 停止对应 node 后，再进行升级。

升级对应 node 前先执行如下命令：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>/usr/<span>local</span>/bin/k3s-killall.sh<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 三、使用安装脚本升级 server[](https://ewhisper.cn/posts/13493/#%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC%E5%8D%87%E7%BA%A7%20-server)

> 🐾**Notes**:
> 
> 要从旧版本升级 K3s，你可以使用相同的标志重新运行安装脚本

执行如下命令进行升级：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_VERSION=v1.22.5+k3s2  INSTALL_K3S_MIRROR=cn K3S_KUBECONFIG_MODE=644 sh -s - --cluster-init --disable-cloud-controller --tls-san &lt;my-public-ip&gt; --kube-apiserver-arg <span>"feature-gates=EphemeralContainers=true"</span> --kube-scheduler-arg <span>"feature-gates=EphemeralContainers=true"</span>  --kube-apiserver-arg default-watch-cache-size=1000 --kube-apiserver-arg delete-collection-workers=10 --kube-apiserver-arg event-ttl=30m --kube-apiserver-arg max-mutating-requests-inflight=800 --kube-apiserver-arg max-requests-inflight=1600 --etcd-expose-metrics <span>true</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

说明如下：

-   `INSTALL_K3S_VERSION=v1.22.5+k3s2` 升级的目标版本
-   `K3S_KUBECONFIG_MODE=644 ... --etcd-expose-metrics true` 都是和之前的安装标志保持一致

升级成功，日志如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><code>[INFO]  Using v1.<span>22.5</span>+k3s2 as release<br>[INFO]  Downloading hash https:<span>//</span>rancher-mirror.rancher.cn<span>/k3s/</span>v1.<span>22.5</span>-k3s2/sha256sum-amd64.txt<br>[INFO]  Downloading binary https:<span>//</span>rancher-mirror.rancher.cn<span>/k3s/</span>v1.<span>22.5</span>-k3s2/k3s<br>[INFO]  Verifying binary download<br>[INFO]  Installing k3s to <span>/usr/</span>local<span>/bin/</span>k3s<br>[INFO]  Skipping <span>/usr/</span>local<span>/bin/</span>kubectl symlink to k3s, already exists<br>[INFO]  Skipping <span>/usr/</span>local<span>/bin/</span>crictl symlink to k3s, already exists<br>[INFO]  Skipping <span>/usr/</span>local<span>/bin/</span>ctr symlink to k3s, already exists<br>[INFO]  Creating killall script <span>/usr/</span>local<span>/bin/</span>k3s-killall.sh<br>[INFO]  Creating uninstall script <span>/usr/</span>local<span>/bin/</span>k3s-uninstall.sh<br>[INFO]  env: Creating environment file <span>/etc/</span>systemd<span>/system/</span>k3s.service.env<br>[INFO]  systemd: Creating service file <span>/etc/</span>systemd<span>/system/</span>k3s.service<br>[INFO]  systemd: Enabling k3s unit<br>Created symlink <span>/etc/</span>systemd<span>/system/mu</span>lti-user.target.wants<span>/k3s.service → /</span>etc<span>/systemd/</span>system/k3s.service.<br>[INFO]  systemd: Starting k3s<br></code><p><i></i>AWK</p></pre></td></tr></tbody></table>

### 四、使用安装脚本升级 Agent[](https://ewhisper.cn/posts/13493/#%E5%9B%9B%E3%80%81%E4%BD%BF%E7%94%A8%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC%E5%8D%87%E7%BA%A7%20-Agent)

> 🐾**Notes**:
> 
> 要从旧版本升级 K3s，你可以使用相同的标志重新运行安装脚本

执行如下命令进行升级：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_VERSION=v1.22.5+k3s2  INSTALL_K3S_MIRROR=cn K3S_URL=https://&lt;my-master-ip&gt;:6443  K3S_TOKEN=&lt;my-token&gt; sh -s -<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

说明如下：

-   其他和 server 升级类似，主要就是版本和相同的标志
-   `K3S_URL=https://<my-master-ip>:6443 K3S_TOKEN=<my-token>` 这是作为 agent 安装需要的参数
    -   `K3S_TOKEN` 位于：`/var/lib/rancher/k3s/server/node-token`, 该 token 升级前后没有发生变化

### 五、验证[](https://ewhisper.cn/posts/13493/#%E4%BA%94%E3%80%81%E9%AA%8C%E8%AF%81)

验证可以通过一些 kubectl 命令，或图形化界面 [Lens](https://ewhisper.cn/posts/33163/) 或 [K9S](https://ewhisper.cn/posts/48546/) 或 Rancher 进行验证。

粗略看这些地方：

-   Events: 有没有 Warning
-   Node 状态：有没有异常的
-   Pod 状态：有没有异常的
-   Jobs 状态：有没有失败的
-   Ingress 状态：有没有访问异常的
-   PVC 状态：有没有非 `Bound` 状态的
-   `kind: Addon` status 有没有异常的

🎉🎉🎉  
但是，验证过程中也发现几个问题，下面一一描述及解决:

-   [Rancher 系列文章 -K3s Traefik MiddleWare 报错 -Failed to create middleware keys](https://ewhisper.cn/posts/7162/)

## 📚️参考文档[](https://ewhisper.cn/posts/13493/#%F0%9F%93%9A%EF%B8%8F%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%20-2)

-   [Support matrix - Rancher v2.6.4](https://www.suse.com/zh-cn/suse-rancher/support-matrix/all-supported-versions/rancher-v2-6-4/)
-   [升级介绍 | Rancher 文档](https://docs.rancher.cn/docs/k3s/upgrades/_index/)
-   [自动升级 | Rancher 文档](https://docs.rancher.cn/docs/k3s/upgrades/automated/_index)
-   [备份和恢复 | Rancher 文档](https://docs.rancher.cn/docs/k3s/backup-restore/_index)