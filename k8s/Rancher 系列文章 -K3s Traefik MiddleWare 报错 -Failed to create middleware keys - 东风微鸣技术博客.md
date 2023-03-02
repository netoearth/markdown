## 概述[](https://ewhisper.cn/posts/7162/#%E6%A6%82%E8%BF%B0%20-2)

书接上回：《[Rancher 系列文章 -K3S 集群升级](https://ewhisper.cn/posts/13493/)》, 我们提到：通过一键脚本升级 K3S 集群有报错。

接下来开始进行 Traefik 报错的分析和修复, 问题是:

-   所有 Traefik 的 `IngressRoute` 访问报错 404

### 问题描述[](https://ewhisper.cn/posts/7162/#%E9%97%AE%E9%A2%98%E6%8F%8F%E8%BF%B0)

报错如下：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code><span>time</span>=<span>"2022-05-05T09:51:21Z"</span> <span>level</span>=error <span>msg</span>=<span>"Failed to create middleware keys: middleware kube-system/hsts-header is not in the IngressRoute namespace cert-manager"</span> <span>namespace</span>=cert-manager <span>providerName</span>=kubernetescrd <span>ingress</span>=grafana<br></code><p><i></i>ROUTEROS</p></pre></td></tr></tbody></table>

即无法跨 NameSpace 调用 Traefik MiddleWare.

### 解决过程[](https://ewhisper.cn/posts/7162/#%E8%A7%A3%E5%86%B3%E8%BF%87%E7%A8%8B)

首先根据官方文档说明：[Kubernetes IngressRoute & Traefik CRD - Traefik](https://doc.traefik.io/traefik/providers/kubernetes-crd/#allowcrossnamespace)

可以配置 `allowCrossNamespace` 参数，该参数默认为 `false`, 如果该参数设置为`true`, IngressRoutes 可以引用其他 NameSpace 中的资源。

基本上断定根因就是这个了。查看 K3s v1.22.5+k3s2 的 Traefik 配置，确实没有这个参数，如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br></pre></td><td><pre><code><span>...</span><br><span>containers:</span><br>  <span>-</span> <span>name:</span> <span>traefik</span><br>    <span>image:</span> <span>rancher/mirrored-library-traefik:2.5.0</span><br>    <span>args:</span><br>      <span>-</span> <span>'--entryPoints.metrics.address=:9100/tcp'</span><br>      <span>-</span> <span>'--entryPoints.traefik.address=:9000/tcp'</span><br>      <span>-</span> <span>'--entryPoints.web.address=:8000/tcp'</span><br>      <span>-</span> <span>'--entryPoints.websecure.address=:8443/tcp'</span><br>      <span>-</span> <span>'--api.dashboard=true'</span><br>      <span>-</span> <span>'--ping=true'</span><br>      <span>-</span> <span>'--metrics.prometheus=true'</span><br>      <span>-</span> <span>'--metrics.prometheus.entrypoint=metrics'</span><br>      <span>-</span> <span>'--providers.kubernetescrd'</span><br>      <span>-</span> <span>'--providers.kubernetesingress'</span><br>      <span>-</span> <span>&gt;-</span><br><span>        --providers.kubernetesingress.ingressendpoint.publishedservice=kube-system/traefik</span><br><span></span>      <span>-</span> <span>'--entrypoints.websecure.http.tls=true'</span><br><span>...</span>      <br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

所以，刚开始就计划通过编辑 Helm 的文件把这个参数加上。

#### 编辑 K3s 的 Manifests Helm 文件[](https://ewhisper.cn/posts/7162/#%E7%BC%96%E8%BE%91%20-K3s-%20%E7%9A%84%20-Manifests-Helm-%20%E6%96%87%E4%BB%B6)

> 📚️ **Reference**:
> 
> -   [自动部署 manifests 和 Helm charts](https://docs.rancher.cn/docs/k3s/helm/_index/#%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2-manifests-%E5%92%8C-helm-charts)  
>     在 `/var/lib/rancher/k3s/server/manifests` 中找到的任何 Kubernetes 清单将以类似 `kubectl apply` 的方式自动部署到 K3s。以这种方式部署的 manifests 是作为 AddOn 自定义资源来管理的，可以通过运行 `kubectl get addon -A` 来查看。你会发现打包组件的 AddOns，如 CoreDNS、Local-Storage、Traefik 等。AddOns 是由部署控制器自动创建的，并根据它们在 manifests 目录下的文件名命名。

该文件位于：`/var/lib/rancher/k3s/server/manifests/traefik.yaml`, 内容如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br></pre></td><td><pre><code><span>---</span><br><span>apiVersion:</span> <span>helm.cattle.io/v1</span><br><span>kind:</span> <span>HelmChart</span><br><span>metadata:</span><br>  <span>name:</span> <span>traefik-crd</span><br>  <span>namespace:</span> <span>kube-system</span><br><span>spec:</span><br>  <span>chart:</span> <span>https://%{KUBERNETES_API}%/static/charts/traefik-crd-10.3.001.tgz</span><br><span>---</span><br><span>apiVersion:</span> <span>helm.cattle.io/v1</span><br><span>kind:</span> <span>HelmChart</span><br><span>metadata:</span><br>  <span>name:</span> <span>traefik</span><br>  <span>namespace:</span> <span>kube-system</span><br><span>spec:</span><br>  <span>chart:</span> <span>https://%{KUBERNETES_API}%/static/charts/traefik-10.3.001.tgz</span><br>  <span>set:</span><br>    <span>global.systemDefaultRegistry:</span> <span>""</span><br>  <span>valuesContent:</span> <span>|-</span><br><span>    rbac:</span><br><span>      enabled: true</span><br><span>    ports:</span><br><span>      websecure:</span><br><span>        tls:</span><br><span>          enabled: true</span><br><span>    podAnnotations:</span><br><span>      prometheus.io/port: "8082"</span><br><span>      prometheus.io/scrape: "true"</span><br><span>    providers:</span><br><span>      kubernetesIngress:</span><br><span>        publishedService:</span><br><span>          enabled: true</span><br><span>    priorityClassName: "system-cluster-critical"</span><br><span>    image:</span><br><span>      name: "rancher/mirrored-library-traefik"</span><br><span>    tolerations:</span><br><span>    - key: "CriticalAddonsOnly"</span><br><span>      operator: "Exists"</span><br><span>    - key: "node-role.kubernetes.io/control-plane"</span><br><span>      operator: "Exists"</span><br><span>      effect: "NoSchedule"</span><br><span>    - key: "node-role.kubernetes.io/master"</span><br><span>      operator: "Exists"</span><br><span>      effect: "NoSchedule"</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

在上面的 yaml 中加入如下配置：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span>...</span><br>    <span>providers:</span><br>      <span>kubernetesCRD:</span><br>        <span>allowCrossNamespace:</span> <span>true</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

待其生效后, 确实可以恢复正常, 但是 K3s 会定期将 Manifests 重置为原有配置, 就会导致问题再次出现.

所以问题并没有最终解决.

#### 使用 HelmChartConfig 自定义打包的组件[](https://ewhisper.cn/posts/7162/#%E4%BD%BF%E7%94%A8%20-HelmChartConfig-%20%E8%87%AA%E5%AE%9A%E4%B9%89%E6%89%93%E5%8C%85%E7%9A%84%E7%BB%84%E4%BB%B6)

不过根据官方文档后续的内容, 我们可以通过 [使用 HelmChartConfig 自定义打包的组件](https://docs.rancher.cn/docs/k3s/helm/_index/#%E4%BD%BF%E7%94%A8-helmchartconfig-%E8%87%AA%E5%AE%9A%E4%B9%89%E6%89%93%E5%8C%85%E7%9A%84%E7%BB%84%E4%BB%B6) 的方式覆盖作为 HelmCharts（如 Traefik）部署的打包组件的值.

具体配置如下:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>helm.cattle.io/v1</span><br><span>kind:</span> <span>HelmChartConfig</span><br><span>metadata:</span><br>  <span>name:</span> <span>traefik</span><br>  <span>namespace:</span> <span>kube-system</span><br><span>spec:</span><br>  <span>valuesContent:</span> <span>|-</span><br><span>    globalArguments:</span><br><span>    - "--providers.kubernetescrd.allowcrossnamespace=true"</span><br><span></span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

生效后, 恢复正常, 并且没有发生回滚.

问题解决.

🎉🎉🎉

## 📚️ 参考文档[](https://ewhisper.cn/posts/7162/#%F0%9F%93%9A%EF%B8%8F-%20%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%20-4)

-   [自动部署 manifests 和 Helm charts](https://docs.rancher.cn/docs/k3s/helm/_index/#%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2-manifests-%E5%92%8C-helm-charts)
-   [使用 HelmChartConfig 自定义打包的组件](https://docs.rancher.cn/docs/k3s/helm/_index/#%E4%BD%BF%E7%94%A8-helmchartconfig-%E8%87%AA%E5%AE%9A%E4%B9%89%E6%89%93%E5%8C%85%E7%9A%84%E7%BB%84%E4%BB%B6)