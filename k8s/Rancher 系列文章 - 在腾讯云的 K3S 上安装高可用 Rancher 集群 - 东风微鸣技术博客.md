## 开篇[](https://ewhisper.cn/posts/12857/#%E5%BC%80%E7%AF%87%20-9)

📜 **引言** ：

-   三人行必有我师焉
-   知识共享，天下为公

-   《[K3s 系列文章](https://ewhisper.cn/tags/K3S/)》
-   《[Rancher 系列文章](https://ewhisper.cn/tags/Rancher/)》

## 方案[](https://ewhisper.cn/posts/12857/#%E6%96%B9%E6%A1%88%20-2)

**在腾讯云的 K3S 上安装 Rancher**

### 方案目标[](https://ewhisper.cn/posts/12857/#%E6%96%B9%E6%A1%88%E7%9B%AE%E6%A0%87%20-2)

1.  高可用
    1.  3 台 master 的 k3s 集群
    2.  高可用模式的 rancher
2.  数据备份
    1.  rancher 数据备份到 腾讯云对象存储 cos
3.  安全加密
    1.  不能存在 http，全部是 https
4.  面向客户
    1.  公网可访问；
    2.  域名可访问；
    3.  正式证书
5.  尽量复用公有云的能力
    1.  ~Tencent Cloud Controller Manager~ (❌ 因为腾讯云已经放弃维护相关源码，所以无法复用）
    2.  ~SVC LoadBalancer 调用 CLB~ (❌ 因为腾讯云已经放弃维护相关源码，所以无法复用）
    3.  CLB - 使用 4 层 CLB
    4.  备份 - 使用腾讯云 COS

## 前提条件[](https://ewhisper.cn/posts/12857/#%E5%89%8D%E6%8F%90%E6%9D%A1%E4%BB%B6%20-2)

1.  有腾讯云账户，账户至少拥有如下权限：[auto k3s 安装 - 设置 CAM](https://docs.rancher.cn/docs/k3s/autok3s/tencent/_index#%E8%AE%BE%E7%BD%AE-cam) 以及这些权限：
    
    1.  `QcloudTAGFullAccess`
2.  该腾讯云账号有对应的 API 密钥，地址：[访问密钥 - 控制台 (tencent.com)](https://console.cloud.tencent.com/cam/capi) ，或者拥有相关权限：`cam:QueryCollApiKey`和 `cam:CreateCollApiKey`
    
3.  一个对象存储通 cos，用于备份
    
4.  Rancher 的域名
    
5.  Rancher 的域名对应的证书（如果没有，会尝试通过 cert-manager 和 let’s encrypt 自动生成免费的证书）
    

## 注意事项[](https://ewhisper.cn/posts/12857/#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

### Rancher 安装注意事项[](https://ewhisper.cn/posts/12857/#Rancher-%20%E5%AE%89%E8%A3%85%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

1.  [通过 Helm Chart 进行高可用安装](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/chart-options/_index)
    
2.  安装前需要调整：
    
    1.  安全组
3.  安装后需要配置：
    
    1.  LB
    2.  Backup
4.  ⚠️付费模式，COS 按需调整付费模式。
    

## 安装步骤[](https://ewhisper.cn/posts/12857/#%E5%AE%89%E8%A3%85%E6%AD%A5%E9%AA%A4%20-2)

### Rancher[](https://ewhisper.cn/posts/12857/#Rancher)

> 🚩 **Important:**
> 
> **通过 Helm Chart 安装**

#### Rancher 端口要求[](https://ewhisper.cn/posts/12857/#Rancher-%20%E7%AB%AF%E5%8F%A3%E8%A6%81%E6%B1%82)

> 📚️ **Quote:**
> 
> [K3s 上 Rancher Server 节点的端口](https://docs.rancher.cn/docs/rancher2.5/installation/requirements/ports/_index#k3s-%E4%B8%8A-rancher-server-%E8%8A%82%E7%82%B9%E7%9A%84%E7%AB%AF%E5%8F%A3)

Rancher Server 节点的入站规则

| 协议 | 端口 | 来源 | 描述 |
| --- | --- | --- | --- |
| TCP | 80 | Load balancer/proxy，做外部 SSL 终端 | 使用外部 SSL 终止时的 Rancher UI/API |
| TCP | 443 | server 节点 agent 节点托管 / 注册的 Kubernetes 任何需要能够使用 Rancher UI 或 API 的源 | Rancher agent, Rancher UI/API, kubectl |
| TCP | 6443 | K3s server 节点 | Kubernetes API |

最后具体的安全组配置如下：（应该可以进一步收紧）

[![](https://pic-cdn.ewhisper.cn/img/2022/02/15/3f4575ad71b533a47ff88439be4bd6b0-20220215215922.png)](https://pic-cdn.ewhisper.cn/img/2022/02/15/3f4575ad71b533a47ff88439be4bd6b0-20220215215922.png)

#### Rancher 高可用安装[](https://ewhisper.cn/posts/12857/#Rancher-%20%E9%AB%98%E5%8F%AF%E7%94%A8%E5%AE%89%E8%A3%85)

先安装 helm chart 并创建 ns:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>helm repo add rancher-stable http://rancher-mirror.oss-cn-beijing.aliyuncs.com/server-charts/stable<p>kubectl create namespace cattle-system</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

SSL 选项为：**已有的证书**，通过 Helm 安装 Rancher:

> 📚️ **Quote:**
> 
> [根据你选择的 SSL 选项，通过 Helm 安装 Rancher](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/_index/#5-%E6%A0%B9%E6%8D%AE%E4%BD%A0%E9%80%89%E6%8B%A9%E7%9A%84-ssl-%E9%80%89%E9%A1%B9%EF%BC%8C%E9%80%9A%E8%BF%87-helm-%E5%AE%89%E8%A3%85-rancher)

先添加证书到 k8s secret:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>kubectl -n cattle-system create secret tls tls-rancher-ingress \<br>  --cert=tls.crt \<br>  --key=tls.key<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>helm install rancher rancher-stable/rancher \<br> --namespace cattle-system \<br> --<span>set</span> hostname=&lt;your-rancher-domain&gt; \<br> --<span>set</span> replicas=3 \<br> --<span>set</span> ingress.tls.source=secret \<br> --<span>set</span> systemDefaultRegistry=registry.cn-hangzhou.aliyuncs.com \<br> --<span>set</span> auditLog.level=1 \<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

运行后输出如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br></pre></td><td><pre><code>NAME: rancher<br>LAST DEPLOYED: Sat Feb <span>12</span> <span>20</span>:<span>10</span>:<span>14</span> <span>2022</span><br>NAMESPACE: cattle-<span>system</span><br>STATUS: deployed<br>REVISION: <span>1</span><br>TEST SUITE: None<br>NOTES:<br>Rancher Server <span>has</span> been installed.<p>NOTE: Rancher may take several minutes <span>to</span> fully initialize. Please standby <span>while</span> Certificates are being issued, Containers are started <span>and</span> the Ingress rule comes <span>up</span>.</p><p>Check out our docs at http<span>s:</span>//rancher.<span>com</span>/docs/</p><p>If you provided your own bootstrap password during installation, <span>browse</span> <span>to</span> http<span>s:</span>//<span>&lt;your-rancher-domain&gt;</span> <span>to</span> <span>get</span> started.</p><p>If this <span>is</span> the <span>first</span> time you installed Rancher, <span>get</span> started by running this <span>command</span> <span>and</span> clicking the URL it generate<span>s:</span></p><p><span>echo</span> http<span>s:</span>//<span>&lt;your-rancher-domain&gt;</span>/dashboard/?setup=$(kubectl <span>get</span> secret --namespace cattle-<span>system</span> bootstrap-secret -<span>o</span> <span>go</span>-template=<span>'{{.data.bootstrapPassword|base64decode}}'</span>)</p><p>To <span>get</span> just the bootstrap password <span>on</span> its own, run:</p><p>kubectl <span>get</span> secret --namespace cattle-<span>system</span> bootstrap-secret -<span>o</span> <span>go</span>-template=<span>'{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'</span></p><p>Happy Containering!</p></code><p><i></i>VIM</p></pre></td></tr></tbody></table>

> 🔥 **Notice:**
> 
> 注意 Rancher 域名的 443 权限要开通。

> ℹ️ **Info:**
> 
> -   要安装一个特定的 Rancher 版本，使用`--version` 标志，例如：`--version 2.3.6`。

之后访问 UI 进行初始密码设置等工作。

🎉 至此，Rancher 高可用集群安装完成。

#### Rancher 中国区优化配置[](https://ewhisper.cn/posts/12857/#Rancher-%20%E4%B8%AD%E5%9B%BD%E5%8C%BA%E4%BC%98%E5%8C%96%E9%85%8D%E7%BD%AE)

参考这里：

-   [Rancher 中国区优化配置](https://ewhisper.cn/posts/36541/#2-1-4-Rancher-%20%E4%B8%AD%E5%9B%BD%E5%8C%BA%E4%BC%98%E5%8C%96%E9%85%8D%E7%BD%AE)

## 收尾工作[](https://ewhisper.cn/posts/12857/#%E6%94%B6%E5%B0%BE%E5%B7%A5%E4%BD%9C%20-2)

### 调整安全组[](https://ewhisper.cn/posts/12857/#%E8%B0%83%E6%95%B4%E5%AE%89%E5%85%A8%E7%BB%84%20-2)

入站规则：

1.  TCP:22(SSH) 端口权限收紧
2.  TCP:6443(K8S API) 端口权限收紧
3.  UDP:8472(K3s vxlan) 只开放给内网
4.  TCP:10250(kube api-server) 只开放给内网

最终效果如下：（应该可以进一步收紧）

[![](https://pic-cdn.ewhisper.cn/img/2022/02/15/3f4575ad71b533a47ff88439be4bd6b0-20220215215922.png)](https://pic-cdn.ewhisper.cn/img/2022/02/15/3f4575ad71b533a47ff88439be4bd6b0-20220215215922.png)

### 配置 LB[](https://ewhisper.cn/posts/12857/#%E9%85%8D%E7%BD%AE%20-LB)

> 📚️ **Quote:**
> 
> [外部 TLS Termination](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/chart-options/_index#%E5%A4%96%E9%83%A8-tls-termination):
> 
> 我们建议将负载均衡器配置为 4 层均衡器，将普通 80/tcp 和 443/tcp 转发到 Rancher 管理集群节点。集群上的 Ingress Controller 会将端口 80 上的 http 通信重定向到端口 443 上的 https。

如上面所述，所以通过 4 层 LB, 将 443/tcp 转到后端。如下图：

[![](https://pic-cdn.ewhisper.cn/img/2022/02/20/73e5cc262c347e88850a7c0157753fe2-20220220200157.png)](https://pic-cdn.ewhisper.cn/img/2022/02/20/73e5cc262c347e88850a7c0157753fe2-20220220200157.png)

### 配置 Rancher Backup[](https://ewhisper.cn/posts/12857/#%E9%85%8D%E7%BD%AE%20-Rancher-Backup)

> 📚️ **Quote:**
> 
> [Rancher v2.5 中的备份和恢复 | Rancher 文档](https://docs.rancher.cn/docs/rancher2.5/backups/_index#%E5%AE%89%E8%A3%85-rancher-backup-operator)
> 
> [备份 Rancher | Rancher 文档](https://docs.rancher.cn/docs/rancher2.5/backups/back-up-rancher/_index/)
> 
> [Rancher Backup Examples](https://rancher.com/docs/rancher/v2.5/en/backups/examples/#backup)

通过 UI 安装：

先创建 cos 存储的认证信息 Secret:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>stringData:</span><br>  <span>accessKey:</span> <span>&lt;your-ak&gt;</span><br>  <span>secretKey:</span> <span>&lt;your-sk&gt;</span><br><span>kind:</span> <span>Secret</span><br><span>metadata:</span><br>  <span>name:</span> <span>cos-creds</span><br>  <span>namespace:</span> <span>cattle-resources-system</span><br><span>type:</span> <span>Opaque</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

然后在 _应用市场_ 选择 _Rancher Backup_ 安装：

[![image-20220212234820849](https://pic-cdn.ewhisper.cn/img/2022/02/12/dbb3ce3aaee4da0fcf5869988412cb70-image-20220212234820849.png)](https://pic-cdn.ewhisper.cn/img/2022/02/12/dbb3ce3aaee4da0fcf5869988412cb70-image-20220212234820849.png "image-20220212234820849")

配置 对象存储：

[![](https://pic-cdn.ewhisper.cn/img/2022/02/20/09330fe280a5f8f84571a57553f731ac-20220220200303.png)](https://pic-cdn.ewhisper.cn/img/2022/02/20/09330fe280a5f8f84571a57553f731ac-20220220200303.png)

安装成功日志如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code><br>helm<span> upgrade </span><span>--install</span>=<span>true</span> <span>--namespace</span>=cattle-resources-system <span>--timeout</span>=10m0s <span>--values</span>=/home/shell/helm/values-rancher-backup-crd-2.1.0.yaml <span>--version</span>=2.1.0 <span>--wait</span>=<span>true</span> rancher-backup-crd /home/shell/helm/rancher-backup-crd-2.1.0.tgz<br><span>..</span>.<br>---------------------------------------------------------------------<br>SUCCESS: helm<span> upgrade </span><span>--install</span>=<span>true</span> <span>--namespace</span>=cattle-resources-system <span>--timeout</span>=10m0s <span>--values</span>=/home/shell/helm/values-rancher-backup-crd-2.1.0.yaml <span>--version</span>=2.1.0 <span>--wait</span>=<span>true</span> rancher-backup-crd /home/shell/helm/rancher-backup-crd-2.1.0.tgz<br>---------------------------------------------------------------------<br>helm<span> upgrade </span><span>--install</span>=<span>true</span> <span>--namespace</span>=cattle-resources-system <span>--timeout</span>=10m0s <span>--values</span>=/home/shell/helm/values-rancher-backup-2.1.0.yaml <span>--version</span>=2.1.0 <span>--wait</span>=<span>true</span> rancher-backup /home/shell/helm/rancher-backup-2.1.0.tgz<br><span>..</span>.<br>---------------------------------------------------------------------<br>SUCCESS: helm<span> upgrade </span><span>--install</span>=<span>true</span> <span>--namespace</span>=cattle-resources-system <span>--timeout</span>=10m0s <span>--values</span>=/home/shell/helm/values-rancher-backup-2.1.0.yaml <span>--version</span>=2.1.0 <span>--wait</span>=<span>true</span> rancher-backup /home/shell/helm/rancher-backup-2.1.0.tgz<br>---------------------------------------------------------------------<br></code><p><i></i>ROUTEROS</p></pre></td></tr></tbody></table>

配置 _Backup_, 如下：

[![image-20220213000206732](https://pic-cdn.ewhisper.cn/img/2022/02/13/d8426a13b5b0f37c8098cfbb9a0cec30-image-20220213000206732.png)](https://pic-cdn.ewhisper.cn/img/2022/02/13/d8426a13b5b0f37c8098cfbb9a0cec30-image-20220213000206732.png "image-20220213000206732")

🎉 登录 COS 发现已经成功备份。

## 总结[](https://ewhisper.cn/posts/12857/#%E6%80%BB%E7%BB%93%20-9)

🎉🎉🎉 至此，完成腾讯云上 K3S 高可用集群 及 Rancher 高可用集群的搭建，并配置备份。

以下是安装的相关信息：

### K3s[](https://ewhisper.cn/posts/12857/#K3s-2)

1.  3 个 Master 和 Server 地址
    
2.  K3S API Server 地址：`https://<3 个 master IP 地址任一个 >:6443` (6443 端口目前没有配置 CLB)
    
3.  K3S kubeconfig 配置：位于 k3s 的`/etc/rancher/k3s/k3s.yaml` 以及操作机的 `/root/.autok3s/.kube/config`
    

### Rancher[](https://ewhisper.cn/posts/12857/#Rancher-2)

1.  地址：
    1.  公网访问：`https://<your-rancher-domain>:<port>/`
    2.  内网访问：`https://<your-rancher-domain>:443` （需要编辑自己电脑的 `hosts` , 将 3 个 master 任一内网 IP 映射到该域名）
2.  账号：`Admin`
3.  密码

### 安全组[](https://ewhisper.cn/posts/12857/#%E5%AE%89%E5%85%A8%E7%BB%84)

使用的安全组，最终配置如下：（应该可以进一步收紧）

[![](https://pic-cdn.ewhisper.cn/img/2022/02/15/3f4575ad71b533a47ff88439be4bd6b0-20220215215922.png)](https://pic-cdn.ewhisper.cn/img/2022/02/15/3f4575ad71b533a47ff88439be4bd6b0-20220215215922.png)

### CLB[](https://ewhisper.cn/posts/12857/#CLB)

使用的 CLB

监听器为：`rancher(TCP:<port>)` 转到 3 台 master 的 443 端口。

### 备份 COS[](https://ewhisper.cn/posts/12857/#%E5%A4%87%E4%BB%BD%20-COS)

K3S 和 Rancher 都配置了备份，备份到对象存储 cos 中。具体的地址为：

1.  桶：rancher-backup-
2.  域名：`https://rancher-backup-<cos-id>.cos.ap-shanghai.myqcloud.com`
3.  S3 Endpoint: `cos.ap-shanghai.myqcloud.com`
4.  文件夹为：
    1.  k3s 为：`/rancher-1/rancher/rancher`（备份策略：每天 0 点备份，保留 5 份）
    2.  rancher 为：`/rancher-1/rancher/k3s` （备份策略，每天 0 点备份）
5.  COS 生命周期为：自动清理 1 个月前的文件。（配置 [自动清理规则](https://console.cloud.tencent.com/cos/bucket?bucket=rancher-backup-1258988025&region=ap-shanghai&type=basicconfig&anchorType=lifeCycle)）