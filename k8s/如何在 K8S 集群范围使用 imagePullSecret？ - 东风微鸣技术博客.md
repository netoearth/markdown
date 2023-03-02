在这篇文章中，我将向你展示如何在 Kubernetes 中使用 imagePullSecrets。

## imagePullSecrets 简介[](https://ewhisper.cn/posts/39547/#imagePullSecrets-%20%E7%AE%80%E4%BB%8B)

Kubernetes 在每个 Pod 或每个 Namespace 的基础上使用 imagePullSecrets 对私有容器注册表进行身份验证。要做到这一点，你需要创建一个秘密与凭据：

> ⚠️ **警告**：
> 
> 现在随着公共镜像仓库（如：[docker.io](http://docker.io/) 等）开始对匿名用户进行限流，配置公共仓库的身份认证也变得有必要。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br></pre></td><td><pre><code>kubectl create secret docker-registry image-pull-secret \<br>  -n &lt;your-namespace&gt; \<br>  --docker-server=&lt;your-registry-server&gt; \<br>  --docker-username=&lt;your-name&gt; \<br>  --docker-password=&lt;your-password&gt; \<br>  --docker-email=&lt;your-email&gt;<br>```  <p>例如配置 docker.io 的 pull secret：</p><p>```bash<br>kubectl create secret docker-registry image-pull-secret-src \<br>        -n imagepullsecret-patcher \<br>        --docker-server=docker.io \<br>        --docker-username=caseycui \<br>        --docker-password=c874d654-xxxx-40c6-xxxx-xxxxxxxx89c2 \<br>        --docker-email=cuikaidong@foxmail.com</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

> ℹ️ **信息**：
> 
> 如果 [docker.io](http://docker.io/) 启用了「2 阶段认证」，可能需要创建 Access Token（对应上面的 `docker-password`，创建链接在这里：[账号 -> 安全](https://hub.docker.com/settings/security)

现在我们可以在一个 pod 中使用这个 secret 来下载 docker 镜像：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>Pod</span><br><span>metadata:</span><br>  <span>name:</span> <span>busybox</span><br>  <span>namespace:</span> <span>private-registry-test</span><br><span>spec:</span><br>  <span>containers:</span><br>    <span>-</span> <span>name:</span> <span>my-app</span><br>      <span>image:</span> <span>my-private-registry.infra/busybox:v1</span><br>  <span>imagePullSecrets:</span><br>    <span>-</span> <span>name:</span> <span>image-pull-secret</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

另一种方法是将它添加到命名空间的默认 ServiceAccount 中：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>kubectl patch serviceaccount default \<br>  -p <span>"{\"imagePullSecrets\": [{\"name\": \"image-pull-secret\"}]}"</span> \<br>  -n &lt;your-namespace&gt;<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 在 K8S 集群范围使用 imagePullSecrets[](https://ewhisper.cn/posts/39547/#%E5%9C%A8%20-K8S-%20%E9%9B%86%E7%BE%A4%E8%8C%83%E5%9B%B4%E4%BD%BF%E7%94%A8%20-imagePullSecrets)

我找到了一个叫做 [`imagepullsecret-patch`](https://github.com/titansoft-pte-ltd/imagepullsecret-patcher) 的工具，它可以在你所有的命名空间上做这个：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code>wget https://raw.githubusercontent.com/titansoft-pte-ltd/imagepullsecret-patcher/185aec934bd01fa9b6ade2c44624e5f2023e2784/deploy-example/kubernetes-manifest/1_rbac.yaml<br>wget https://raw.githubusercontent.com/titansoft-pte-ltd/imagepullsecret-patcher/master/deploy-example/kubernetes-manifest/2_deployment.yaml<p>kubectl create ns imagepullsecret-patcher</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

编辑下载的文件，一般需要修改 `image-pull-secret-src` 的内容，这个 pull secret 就会应用到 K8S 集群范围。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code>nano 1_rbac.yaml<br>nano 2_deployment.yaml<br>kubectl apply -f 1_rbac.yaml<br>kubectl apply -f 2_deployment.yaml<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

这里背后创建的资源有：

1.  NameSpace
2.  RBAC 权限相关：
    1.  `imagepullsecret-patcher` ServiceAccount
    2.  `imagepullsecret-patcher` ClusterRole，具有对 service account 和 secret 的所有权限
    3.  `imagepullsecret-patcher` ClusterRoleBinding，为 `imagepullsecret-patcher` ServiceAccount 赋予 `imagepullsecret-patcher` ClusterRole 的权限。
3.  全局 pull secret `image-pull-secret-src`，里面是你的 K8S 全局包含的所有的镜像库地址和认证信息。
4.  Deployment `imagepullsecret-patcher`，指定 ServiceAccount 是 `imagepullsecret-patcher` 就有了操作 service account 和 secret 的所有权限，并将上面的 secret 挂载到 Deployment pod 内。

可以包含多个镜像库地址和认证信息，如：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span>{</span><br>    <span>"auths"</span><span>:</span> <span>{</span><br>        <span>"docker.io"</span><span>:</span> <span>{</span><br>            <span>"username"</span><span>:</span> <span>"caseycui"</span><span>,</span><br>            <span>"password"</span><span>:</span> <span>"c874xxxxxxxxxxxxxxxx1f89c2"</span><span>,</span><br>            <span>"email"</span><span>:</span> <span>"cuikaidong@foxmail.com"</span><span>,</span><br>            <span>"auth"</span><span>:</span> <span>"Y2FzxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxWMy"</span><br>        <span>}</span><span>,</span><br>        <span>"quay.io"</span><span>:</span> <span>{</span><br>            <span>"auth"</span><span>:</span> <span>"ZWFzdxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxlXWmpNPQ=="</span><span>,</span><br>            <span>"email"</span><span>:</span> <span>""</span><br>        <span>}</span><br>    <span>}</span><br><span>}</span><br></code><p><i></i>JSON</p></pre></td></tr></tbody></table>

base64 编码后写到 secret 的 `.dockerconfigjson` 字段即可：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>Secret</span><br><span>metadata:</span><br>  <span>name:</span> <span>image-pull-secret-src</span><br>  <span>namespace:</span> <span>imagepullsecret-patcher</span><br><span>data:</span><br>  <span>.dockerconfigjson:</span> <span>&gt;-</span><br><span>    eyJhdXRocyI6eyJkb2NrZXIuaW8iOnsidXNlcm5hbWUiOiJjYXNleWN1aSIsInB.............................................IiwiZW1haWwiOiIifX19</span><br><span></span><span>type:</span> <span>kubernetes.io/dockerconfigjson</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

启动后的 pod 会在所有 NameSpace 下创建 `image-pull-secret` secret（内容来自于`image-pull-secret-src`) 并把它 patch 到 `default` service account 及该 K8S 集群的所有 ServiceAccount 里，日志如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br></pre></td><td><pre><code>time="2022-01-12T16:07:30Z" level=info msg="Application started"<br>time="2022-01-12T16:07:30Z" level=info msg="[default] Created secret"<br>time="2022-01-12T16:07:30Z" level=info msg="[default] Patched imagePullSecrets to service account [default]"<br>time="2022-01-12T16:07:30Z" level=info msg="[kube-system] Created secret"<br>time="2022-01-12T16:07:31Z" level=info msg="[kube-system] Patched imagePullSecrets to service account [node-controller]"<br>...<br>time="2022-01-12T16:07:37Z" level=info msg="[kube-public] Created secret"<br>time="2022-01-12T16:07:37Z" level=info msg="[kube-public] Patched imagePullSecrets to service account [default]"<br>time="2022-01-12T16:07:38Z" level=info msg="[kube-node-lease] Created secret"<br>time="2022-01-12T16:07:38Z" level=info msg="[kube-node-lease] Patched imagePullSecrets to service account [default]"<br>time="2022-01-12T16:07:38Z" level=info msg="[prometheus] Created secret"<br>time="2022-01-12T16:07:39Z" level=info msg="[prometheus] Patched imagePullSecrets to service account [default]"<br>...<br>time="2022-01-12T16:07:41Z" level=info msg="[imagepullsecret-patcher] Created secret"<br>time="2022-01-12T16:07:41Z" level=info msg="[imagepullsecret-patcher] Patched imagePullSecrets to service account [default]"<br>time="2022-01-12T16:07:41Z" level=info msg="[imagepullsecret-patcher] Patched imagePullSecrets to service account [imagepullsecret-patcher]"<br></code><p><i></i>LOG</p></pre></td></tr></tbody></table>

今后我们只需要更新 `image-pull-secret-src` 这一个即可了。👍️👍️👍️

## Kyverno policy[](https://ewhisper.cn/posts/39547/#Kyverno-policy)

[Kyverno](https://kyverno.io/) policy 可以实现同样的效果:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>kyverno.io/v1</span><br><span>kind:</span> <span>ClusterPolicy</span><br><span>metadata:</span><br>  <span>name:</span> <span>sync-secret</span><br><span>spec:</span><br>  <span>background:</span> <span>false</span><br>  <span>rules:</span><br>  <span>-</span> <span>name:</span> <span>sync-image-pull-secret</span><br>    <span>match:</span><br>      <span>resources:</span><br>        <span>kinds:</span><br>        <span>-</span> <span>Namespace</span><br>    <span>generate:</span><br>      <span>kind:</span> <span>Secret</span><br>      <span>name:</span> <span>image-pull-secret</span><br>      <span>namespace:</span> <span>"<span>{{request.object.metadata.name}}</span>"</span><br>      <span>synchronize:</span> <span>true</span><br>      <span>clone:</span><br>        <span>namespace:</span> <span>default</span><br>        <span>name:</span> <span>image-pull-secret</span><br><span>---</span><br><span>apiVersion:</span> <span>kyverno.io/v1</span><br><span>kind:</span> <span>ClusterPolicy</span><br><span>metadata:</span><br>  <span>name:</span> <span>mutate-imagepullsecret</span><br><span>spec:</span><br>  <span>rules:</span><br>    <span>-</span> <span>name:</span> <span>mutate-imagepullsecret</span><br>      <span>match:</span><br>        <span>resources:</span><br>          <span>kinds:</span><br>          <span>-</span> <span>Pod</span><br>      <span>mutate:</span><br>        <span>patchStrategicMerge:</span><br>          <span>spec:</span><br>            <span>imagePullSecrets:</span><br>            <span>-</span> <span>name:</span> <span>image-pull-secret</span>  <span>## imagePullSecret that you created with docker hub pro account</span><br>            <span>(containers):</span><br>            <span>-</span> <span>(image):</span> <span>"*"</span> <span>## match all container images</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>