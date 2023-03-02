## 开篇[](https://ewhisper.cn/posts/60907/#%E5%BC%80%E7%AF%87%20-8)

📜 **引言** ：

-   磨刀不误砍柴工
-   工欲善其事必先利其器

-   第一篇：《[K8S 实用工具之一 - 如何合并多个 kubeconfig？](https://ewhisper.cn/posts/46789/)》
-   第二篇：《[K8S 实用工具之二 - 终端 UI K9S](https://ewhisper.cn/posts/48546/)》
-   第三篇：《[K8S 实用工具之三 - 图形化 UI Lens](https://ewhisper.cn/posts/33163)》

在《[K8S 实用工具之一 - 如何合并多个 kubeconfig？](https://ewhisper.cn/posts/46789/)》一文中，我们介绍了 kubectl 的插件管理工具 krew。接下来就顺势介绍几个实用的 kubectl 插件。

### [access-matrix](https://github.com/corneliusweig/rakkess)[](https://ewhisper.cn/posts/60907/#access-matrix)

显示服务器资源的 RBAC 访问矩阵。

您是否曾经想过您对所提供的 kubernetes 集群拥有哪些访问权限? 对于单个资源，您可以使用`kubectl auth can-i` 列表部署，但也许您正在寻找一个完整的概述? 这就是它的作用。它列出当前用户和所有服务器资源的访问权限，类似于`kubectl auth can-i --list`。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-3)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install access-matrix<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-2)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br></pre></td><td><pre><code>Review access to cluster-scoped resources<br> $ kubectl access-matrix<p>Review access to namespaced resources <span>in</span> <span>'default'</span><br> $ kubectl access-matrix --namespace default</p><p>Review access as a different user<br> $ kubectl access-matrix --as other-user</p><p>Review access as a service-account<br> $ kubectl access-matrix --sa kube-system:namespace-controller</p><p>Review access <span>for</span> different verbs<br> $ kubectl access-matrix --verbs get,watch,patch</p><p>Review access rights diff with another service account<br> $ kubectl access-matrix --diff-with sa=kube-system:namespace-controller</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

显示效果如下：

[![access-matrix](https://pic-cdn.ewhisper.cn/img/2021/11/27/2d3f56bfe8361237d318ca2e9f8209af-image-20211127132741553.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/2d3f56bfe8361237d318ca2e9f8209af-image-20211127132741553.png "access-matrix")

### [ca-cert](https://github.com/ahmetb/kubectl-extras)[](https://ewhisper.cn/posts/60907/#ca-cert)

打印当前集群的 PEM CA 证书

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-4)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install ca-cert<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-3)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl ca-cert<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

[![kubectl ca-cert](https://pic-cdn.ewhisper.cn/img/2021/11/27/4f0c4ec68a56fa17ccab634ec301b4ce-image-20211127133039866.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/4f0c4ec68a56fa17ccab634ec301b4ce-image-20211127133039866.png "kubectl ca-cert")

### [cert-manager](https://github.com/jetstack/cert-manager)[](https://ewhisper.cn/posts/60907/#cert-manager)

这个不用多介绍了吧？大名鼎鼎的 cert-manager，用来管理集群内的证书资源。

[![cert-manager](https://pic-cdn.ewhisper.cn/img/2021/11/27/99258d3eb9835e91919c8eba4b84060d-cert-manager-logo.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/99258d3eb9835e91919c8eba4b84060d-cert-manager-logo.png "cert-manager")

需要配合在 K8S 集群中安装 cert-manager 来使用。后面有时间再详细介绍

[![kubectl cert-manager help](https://pic-cdn.ewhisper.cn/img/2021/11/27/ce3abd31e68bb3506c4e0c252f817511-image-20211127133722939.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/ce3abd31e68bb3506c4e0c252f817511-image-20211127133722939.png "kubectl cert-manager help")

### [cost](https://github.com/kubecost/kubectl-cost)[](https://ewhisper.cn/posts/60907/#cost)

查看集群成本信息。

`kubectl-cost` 是一个 kubectl 插件，通过 kubeccost api 提供简单的 CLI 访问 Kubernetes 成本分配指标。它允许开发人员、devops 和其他人快速确定 Kubernetes 工作负载的成本和效率。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-5)

1.  安装 Kubecost （Helm 的 options 可以看这里：[cost-analyzer-helm-chart](https://github.com/kubecost/cost-analyzer-helm-chart/blob/master/README.md#config-options)）
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>helm repo add kubecost https://kubecost.github.io/cost-analyzer/<br>helm upgrade -i --create-namespace kubecost kubecost/cost-analyzer --namespace kubecost --<span>set</span> kubecostToken=<span>"a3ViZWN0bEBrdWJlY29zdC5jb20=xm343yadf98"</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
    部署完成显示如下：
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code>NAME: kubecost<br>LAST DEPLOYED: Sat Nov <span>27</span> <span>13</span>:<span>44</span>:<span>30</span> <span>2021</span><br>NAMESPACE: kubecost<br>STATUS: deployed<br>REVISION: <span>1</span><br>TEST SUITE: None<br>NOTES:<br><span>--------------------------------------------------Kubecost has been successfully installed. When pods are Ready, you can enable port-forwarding with the following command:</span><p>    kubectl port-forward <span>--namespace kubecost deployment/kubecost-cost-analyzer 9090</span></p><p>Next, navigate <span>to</span> <span>http</span>://localhost:<span>9090</span> <span>in</span> <span>a</span> web browser.</p><p>Having installation issues? View our Troubleshooting Guide <span>at</span> <span>http</span>://docs.kubecost.com/troubleshoot-install</p></code><p><i></i>LIVECODESERVER</p></pre></td></tr></tbody></table>
    
2.  安装 kubectl cost
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install cost<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-4)

使用可以直接通过浏览器来看：

[![kubecost UI](https://pic-cdn.ewhisper.cn/img/2021/11/27/3c30a9b1d66a8cccc8fe770a322b3bad-image-20211127155816450.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/3c30a9b1d66a8cccc8fe770a322b3bad-image-20211127155816450.png "kubecost UI")

### [ctx](https://github.com/ahmetb/kubectx)[](https://ewhisper.cn/posts/60907/#ctx)

在 kubeconfig 中切换上下文

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-6)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install ctx<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-5)

使用也很简单，执行 `kubectl ctx` 然后选择要切换到哪个 context 即可。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>$ kubectl ctx<br>Switched to context <span>"multicloud-k3s"</span>.<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [deprecations](https://github.com/rikatz/kubepug)[](https://ewhisper.cn/posts/60907/#deprecations)

检查集群中已经弃用的对象。一般用在升级 K8S 之前做检查。又叫 **KubePug**

[![KubePug](https://pic-cdn.ewhisper.cn/img/2021/11/27/ebadf76885b4b605517aefb85e75a3a2-kubepug.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/ebadf76885b4b605517aefb85e75a3a2-kubepug.png "KubePug")

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-7)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install deprecations<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-6)

使用也很简单，执行 `kubectl deprecations` 即可，然后如下面所示，它会告诉你哪些 API 已经弃用了，方便规划 K8S 升级规划。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br></pre></td><td><pre><code>$ kubectl deprecations<br>W1127 16:04:58.641429   28561 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated <span>in</span> v1.21+, unavailable <span>in</span> v1.25+<br>W1127 16:04:58.664058   28561 warnings.go:70] v1 ComponentStatus is deprecated <span>in</span> v1.19+<br>W1127 16:04:59.622247   28561 warnings.go:70] apiregistration.k8s.io/v1beta1 APIService is deprecated <span>in</span> v1.19+, unavailable <span>in</span> v1.22+; use apiregistration.k8s.io/v1 APIService<br>W1127 16:05:00.777598   28561 warnings.go:70] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated <span>in</span> v1.16+, unavailable <span>in</span> v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition<br>W1127 16:05:00.808486   28561 warnings.go:70] extensions/v1beta1 Ingress is deprecated <span>in</span> v1.14+, unavailable <span>in</span> v1.22+; use networking.k8s.io/v1 Ingress<br>RESULTS:<br>Deprecated APIs:<p>PodSecurityPolicy found <span>in</span> policy/v1beta1<br>         ├─ PodSecurityPolicy governs the ability to make requests that affect the Security Context that will be applied to a pod and container. Deprecated <span>in</span> 1.21.<br>                -&gt; GLOBAL: kube-prometheus-stack-admission<br>                -&gt; GLOBAL: loki-grafana-test<br>                -&gt; GLOBAL: loki-promtail<br>                -&gt; GLOBAL: loki<br>                -&gt; GLOBAL: loki-grafana<br>                -&gt; GLOBAL: prometheus-operator-grafana-test<br>                -&gt; GLOBAL: prometheus-operator-alertmanager<br>                -&gt; GLOBAL: prometheus-operator-grafana<br>                -&gt; GLOBAL: prometheus-operator-prometheus<br>                -&gt; GLOBAL: prometheus-operator-prometheus-node-exporter<br>                -&gt; GLOBAL: prometheus-operator-kube-state-metrics<br>                -&gt; GLOBAL: prometheus-operator-operator<br>                -&gt; GLOBAL: kubecost-grafana<br>                -&gt; GLOBAL: kubecost-cost-analyzer-psp</p><p>ComponentStatus found <span>in</span> /v1<br>         ├─ ComponentStatus (and ComponentStatusList) holds the cluster validation info. Deprecated: This API is deprecated <span>in</span> v1.19+<br>                -&gt; GLOBAL: controller-manager<br>                -&gt; GLOBAL: scheduler</p><p>Deleted APIs:</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

还可以和 CI 流程结合起来使用：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>$ kubectl deprecations --input-file=./deployment/ --error-on-deleted --error-on-deprecated<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [df-pv](https://github.com/yashbhutwala/kubectl-df-pv)[](https://ewhisper.cn/posts/60907/#df-pv)

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-8)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install df-pv<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

使用

执行 `kubectl df-pv`:

[![kubectl df-pv](https://pic-cdn.ewhisper.cn/img/2021/11/27/46e6fd7d4fdd515e018f4eaed0758708-20211127161338.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/46e6fd7d4fdd515e018f4eaed0758708-20211127161338.png "kubectl df-pv")

### [get-all](https://github.com/corneliusweig/ketall)[](https://ewhisper.cn/posts/60907/#get-all)

真正能 get 到 Kubernetes 的所有资源。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-9)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install get-all<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-7)

直接执行 `kubectl get-all`, 示例效果如下：

[![kubectl get-all](https://pic-cdn.ewhisper.cn/img/2021/11/27/b47699a168f25a214c311c78294d4f83-image-20211127162444588.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/b47699a168f25a214c311c78294d4f83-image-20211127162444588.png "kubectl get-all")

### [images](https://github.com/chenjiandongx/kubectl-images)[](https://ewhisper.cn/posts/60907/#images)

显示集群中使用的容器镜像。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-10)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install images<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-8)

执行 `kubectl images -A` ，结果如下：

[![kubectl images](https://pic-cdn.ewhisper.cn/img/2021/11/27/0806c8047d17f1c6db941a3b0c7f7cd6-image-20211127162716161.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/0806c8047d17f1c6db941a3b0c7f7cd6-image-20211127162716161.png "kubectl images")

### [kubesec-scan](https://github.com/controlplaneio/kubectl-kubesec)[](https://ewhisper.cn/posts/60907/#kubesec-scan)

使用 [kubesec.io](http://kubesec.io/) 扫描 Kubernetes 资源。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-11)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install kubesec-scan<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-9)

示例如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code>$ kubectl kubesec-scan statefulset loki -n loki-stack<br>scanning statefulset loki <span>in</span> namespace loki-stack<br>kubesec.io score: 4<br>-----------------<br>Advise1. .spec .volumeClaimTemplates[] .spec .accessModes | index(<span>"ReadWriteOnce"</span>)<br>2. containers[] .securityContext .runAsNonRoot == <span>true</span><br>Force the running image to run as a non-root user to ensure least privilege<br>3. containers[] .securityContext .capabilities .drop<br>Reducing kernel capabilities available to a container limits its attack surface<br>4. containers[] .securityContext .runAsUser &gt; 10000<br>Run as a high-UID user to avoid conflicts with the host<span>'s user table</span><br><span>5. containers[] .securityContext .capabilities .drop | index("ALL")</span><br><span>Drop all capabilities and add only those required to reduce syscall attack surface</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [neat](https://github.com/itaysk/kubectl-neat)[](https://ewhisper.cn/posts/60907/#neat)

从 Kubernetes 显示中删除杂乱以使其更具可读性。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-12)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install neat<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-10)

示例如下：

我们不关注的一些信息如：`creationTimeStamp` 、 `managedFields` 等被移除了。很清爽

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br></pre></td><td><pre><code>$ kubectl neat get -- pod loki-0 -oyaml -n loki-stack<br>apiVersion: v1<br>kind: Pod<br>metadata:<br>  annotations:<br>    checksum/config: b9ab988df734dccd44833416670e70085a2a31cfc108e68605f22d3a758f50b5<br>    prometheus.io/port: http-metrics<br>    prometheus.io/scrape: <span>"true"</span><br>  labels:<br>    app: loki<br>    controller-revision-hash: loki-79684c849<br>    name: loki<br>    release: loki<br>    statefulset.kubernetes.io/pod-name: loki-0<br>  name: loki-0<br>  namespace: loki-stack<br>spec:<br>  containers:<br>  - args:<br>    - -config.file=/etc/loki/loki.yaml<br>    image: grafana/loki:2.3.0<br>    livenessProbe:<br>      httpGet:<br>        path: /ready<br>        port: http-metrics<br>      initialDelaySeconds: 45<br>    name: loki<br>    ports:<br>    - containerPort: 3100<br>      name: http-metrics<br>    readinessProbe:<br>      httpGet:<br>        path: /ready<br>        port: http-metrics<br>      initialDelaySeconds: 45<br>    securityContext:<br>      readOnlyRootFilesystem: <span>true</span><br>    volumeMounts:<br>    - mountPath: /etc/loki<br>      name: config<br>    - mountPath: /data<br>      name: storage<br>    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount<br>      name: kube-api-access-jhsvm<br>      readOnly: <span>true</span><br>  hostname: loki-0<br>  preemptionPolicy: PreemptLowerPriority<br>  priority: 0<br>  securityContext:<br>    fsGroup: 10001<br>    runAsGroup: 10001<br>    runAsNonRoot: <span>true</span><br>    runAsUser: 10001<br>  serviceAccountName: loki<br>  subdomain: loki-headless<br>  terminationGracePeriodSeconds: 4800<br>  tolerations:<br>  - effect: NoExecute<br>    key: node.kubernetes.io/not-ready<br>    operator: Exists<br>    tolerationSeconds: 300<br>  - effect: NoExecute<br>    key: node.kubernetes.io/unreachable<br>    operator: Exists<br>    tolerationSeconds: 300<br>  volumes:<br>  - name: config<br>    secret:<br>      secretName: loki<br>  - name: storage<br>  - name: kube-api-access-jhsvm<br>    projected:<br>      sources:<br>      - serviceAccountToken:<br>          expirationSeconds: 3607<br>          path: token<br>      - configMap:<br>          items:<br>          - key: ca.crt<br>            path: ca.crt<br>          name: kube-root-ca.crt<br>      - downwardAPI:<br>          items:<br>          - fieldRef:<br>              fieldPath: metadata.namespace<br>            path: namespace<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [node-shell](https://github.com/kvaps/kubectl-node-shell)[](https://ewhisper.cn/posts/60907/#node-shell)

通过 kubectl 在一个 node 上生成一个 root shell

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-13)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install node-shell<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-11)

示例如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br></pre></td><td><pre><code>$ kubectl node-shell instance-ykx0ofns<br>spawning <span>"nsenter-fr393w"</span> on <span>"instance-ykx0ofns"</span><br>If you don<span>'t see a command prompt, try pressing enter.</span><br><span>root@instance-ykx0ofns:/# hostname</span><br><span>instance-ykx0ofns</span><br><span>root@instance-ykx0ofns:/# ifconfig</span><br><span>...</span><br><span>eth0: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1500</span><br><span>        inet 192.168.64.4  netmask 255.255.240.0  broadcast 192.168.79.255</span><br><span>        inet6 fe80::f820:20ff:fe16:3084  prefixlen 64  scopeid 0x20&lt;link&gt;</span><br><span>        ether fa:20:20:16:30:84  txqueuelen 1000  (Ethernet)</span><br><span>        RX packets 24386113  bytes 26390915146 (26.3 GB)</span><br><span>        RX errors 0  dropped 0  overruns 0  frame 0</span><br><span>        TX packets 18840452  bytes 3264860766 (3.2 GB)</span><br><span>        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0</span><br><span>...</span><br><span>root@instance-ykx0ofns:/# exit</span><br><span>logout</span><br><span>pod default/nsenter-fr393w terminated (Error)</span><br><span>pod "nsenter-fr393w" deleted</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [ns](https://github.com/ahmetb/kubectx)[](https://ewhisper.cn/posts/60907/#ns)

切换 Kubernetes 的 ns。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-14)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install ns<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-12)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code>$ kubectl ns loki-stack<br>Context <span>"multicloud-k3s"</span> modified.<br>Active namespace is <span>"loki-stack"</span>.<p>$ kubectl get pod<br>NAME                           READY   STATUS    RESTARTS   AGE<br>loki-promtail-fbbjj            1/1     Running   0          12d<br>loki-promtail-sx5gj            1/1     Running   0          12d<br>loki-0                         1/1     Running   0          12d<br>loki-grafana-8bffbb679-szdpj   1/1     Running   0          12d<br>loki-promtail-hmc26            1/1     Running   0          12d<br>loki-promtail-xvnbc            1/1     Running   0          12d<br>loki-promtail-5d5h8            1/1     Running   0          12d</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [outdated](https://github.com/replicatedhq/outdated)[](https://ewhisper.cn/posts/60907/#outdated)

查找集群中运行的过时容器镜像。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-15)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install outdated<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-13)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br></pre></td><td><pre><code>$ kubectl outdated<br>Image                                                  Current             Latest             Behind<br>index.docker.io/rancher/klipper-helm                   v0.6.6-build202110220.6.8-build202111232<br>docker.io/rancher/klipper-helm                         v0.6.4-build202108130.6.8-build202111234<br>docker.io/alekcander/k3s-flannel-fixer                 0.0.2               0.0.2              0<br>docker.io/rancher/metrics-server                       v0.3.6              0.4.1              1<br>docker.io/rancher/coredns-coredns                      1.8.3               1.8.3              0<br>docker.io/rancher/library-traefik                      2.4.8               2.4.9              1<br>docker.io/rancher/local-path-provisioner               v0.0.19             0.0.20             1<br>docker.io/grafana/promtail                             2.1.0               2.4.1              5<br>docker.io/grafana/loki                                 2.3.0               2.4.1              2<br>quay.io/kiwigrid/k8s-sidecar                           1.12.3              1.14.2             5<br>docker.io/grafana/grafana                              8.1.6               8.3.0-beta1        8<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v0.18.1             1.3.0              9<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v0.20.0             0.23.0             5<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v0.0.1              0.0.1              0<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v1.9.4              2.0.0-beta         5<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v2.15.2             2.31.1             38<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v0.35.0             0.42.1             11<br>docker.io/kiwigrid/k8s-sidecar                         0.1.20              1.14.2             46<br>docker.io/grafana/grafana                              6.5.2               8.3.0-beta1        75<br>registry.cn-hangzhou.aliyuncs.com/kubeapps/quay...     v0.35.0             0.42.1             12<br>docker.io/squareup/ghostunnel                          v1.5.2              1.5.2              0<br>docker.io/grafana/grafana                              8.1.2               8.3.0-beta1        12<br>docker.io/kiwigrid/k8s-sidecar                         1.12.3              1.14.2             5<br>docker.io/prom/prometheus                              v2.22.2             2.31.1             21<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [popeye](https://popeyecli.io/)（大力水手）[](https://ewhisper.cn/posts/60907/#popeye%EF%BC%88%E5%A4%A7%E5%8A%9B%E6%B0%B4%E6%89%8B%EF%BC%89)

扫描集群以发现潜在的资源问题。就是 K9S 也在使用的 popeye。

Popeye 是一个实用程序，它扫描实时的 Kubernetes 集群，并报告部署的资源和配置的潜在问题。它根据已部署的内容而不是磁盘上的内容来清理集群。通过扫描集群，它可以检测到错误配置，并帮助您确保最佳实践已经到位，从而避免未来的麻烦。它旨在减少人们在野外操作 Kubernetes 集群时所面临的认知过载。此外，如果您的集群使用度量服务器，它会报告分配的资源超过或低于分配的资源，并试图在集群耗尽容量时警告您。

Popeye 是一个只读的工具，它不会改变任何你的 Kubernetes 资源在任何方式!

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-16)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install popeye<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-14)

如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br></pre></td><td><pre><code>❯ kubectl popeye<p> ___     ___ _____   _____                                                      K          .-<span>'-.</span><br><span>| _ \___| _ \ __\ \ / / __|                                                      8     __|      `\</span><br><span>|  _/ _ \  _/ _| \ V /| _|                                                        s   `-,-`--._   `\</span><br><span>|_| \___/_| |___| |_| |___|                                                      []  .-&gt;'</span>  a     `|-<span>'</span><br><span>  Biffs`em and Buffs`em!                                                          `=/ (__/_       /</span><br><span>                                                                                    \_,    `    _)</span><br><span>                                                                                       `----;  |</span><br><span></span><br><span></span><br><span></span><br><span></span><br><span>DAEMONSETS (1 SCANNED)                                                         💥 0 😱 1 🔊 0 ✅ 0 0٪</span><br><span>┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅</span><br><span>  · loki-stack/loki-promtail.......................................................................😱</span><br><span>    🔊 [POP-404] Deprecation check failed. Unable to assert resource version.</span><br><span>    🐳 promtail</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span></span><br><span></span><br><span>DEPLOYMENTS (1 SCANNED)                                                        💥 0 😱 1 🔊 0 ✅ 0 0٪</span><br><span>┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅</span><br><span>  · loki-stack/loki-grafana........................................................................😱</span><br><span>    🔊 [POP-404] Deprecation check failed. Unable to assert resource version.</span><br><span>    🐳 grafana</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span>    🐳 grafana-sc-datasources</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span></span><br><span></span><br><span></span><br><span>PODS (7 SCANNED)                                                               💥 0 😱 7 🔊 0 ✅ 0 0٪</span><br><span>┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅</span><br><span>  · loki-stack/loki-0..............................................................................😱</span><br><span>    🔊 [POP-206] No PodDisruptionBudget defined.</span><br><span>    😱 [POP-301] Connects to API Server? ServiceAccount token is mounted.</span><br><span>    🐳 loki</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span>  · loki-stack/loki-grafana-8bffbb679-szdpj........................................................😱</span><br><span>    🔊 [POP-206] No PodDisruptionBudget defined.</span><br><span>    😱 [POP-301] Connects to API Server? ServiceAccount token is mounted.</span><br><span>    🐳 grafana</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span>      🔊 [POP-105] Liveness probe uses a port#, prefer a named port.</span><br><span>      🔊 [POP-105] Readiness probe uses a port#, prefer a named port.</span><br><span>    🐳 grafana-sc-datasources</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span>  · loki-stack/loki-promtail-5d5h8.................................................................😱</span><br><span>    🔊 [POP-206] No PodDisruptionBudget defined.</span><br><span>    😱 [POP-301] Connects to API Server? ServiceAccount token is mounted.</span><br><span>    😱 [POP-302] Pod could be running as root user. Check SecurityContext/image.</span><br><span>    🐳 promtail</span><br><span>      😱 [POP-106] No resources requests/limits defined.</span><br><span>      😱 [POP-103] No liveness probe.</span><br><span>      😱 [POP-306] Container could be running as root user. Check SecurityContext/Image.</span><br><span></span><br><span>SUMMARY</span><br><span>┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅</span><br><span>Your cluster score: 80 -- B</span><br><span>                                                                                o          .-'</span>-.<br>                                                                                 o     __| B    `\<br>                                                                                  o   `-,-`--._   `\<br>                                                                                 []  .-&gt;<span>'  a     `|-'</span><br>                                                                                  `=/ (__/_       /<br>                                                                                    \_,    `    _)<br>                                                                                       `----;  |</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [resource-capacity](https://github.com/robscott/kube-capacity)[](https://ewhisper.cn/posts/60907/#resource-capacity)

提供资源请求、限制和使用率的概览。

这是一个简单的 CLI，它提供 Kubernetes 集群中资源请求、限制和利用率的概览。它试图将来自 kubectl top 和 kubectl describe 的输出的最好部分组合成一个简单易用的 CLI，专注于集群资源。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-17)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install resource-capacity<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-15)

下面示例是看 node 的，也可以看 pod，通过 label 筛选， 并排序等功能。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code>$ kubectl resource-capacity<br>NODE                       CPU REQUESTS   CPU LIMITS   MEMORY REQUESTS   MEMORY LIMITS<br>*                          710m (14%)     300m (6%)    535Mi (6%)        257Mi (3%)<br>09b2brd7robnn5zi-1106883   0Mi (0%)       0Mi (0%)     0Mi (0%)          0Mi (0%)<br>hecs-348550                100m (10%)     100m (10%)   236Mi (11%)       27Mi (1%)<br>instance-wy7ksibk          310m (31%)     0Mi (0%)     174Mi (16%)       0Mi (0%)<br>instance-ykx0ofns          200m (20%)     200m (20%)   53Mi (5%)         53Mi (5%)<br>izuf656om146vu1n6pd6lpz    100m (10%)     0Mi (0%)     74Mi (3%)         179Mi (8%)<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [score](https://github.com/zegl/kube-score)[](https://ewhisper.cn/posts/60907/#score)

Kubernetes 静态代码分析。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-18)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install score<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-16)

也是可以和 CI 进行集成的。示例如下：

[![kubectl score](https://pic-cdn.ewhisper.cn/img/2021/11/27/28b607bc22de8edffdac10c2cb57efc7-20211127205516.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/28b607bc22de8edffdac10c2cb57efc7-20211127205516.png "kubectl score")

### [sniff](https://github.com/eldadru/ksniff)[](https://ewhisper.cn/posts/60907/#sniff)

强烈推荐，之前有次 POD 网络出现问题就是通过这个帮助来进行分析的。它会使用 tcpdump 和 wireshark 在 pod 上启动远程抓包

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-19)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install sniff<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-17)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><code>kubectl &lt; 1.12:<br>kubectl plugin sniff &lt;POD_NAME&gt; [-n &lt;NAMESPACE_NAME&gt;] [-c &lt;CONTAINER_NAME&gt;] [-i &lt;INTERFACE_NAME&gt;] [-f &lt;CAPTURE_FILTER&gt;] [-o OUTPUT_FILE] [-l LOCAL_TCPDUMP_FILE] [-r REMOTE_TCPDUMP_FILE]<p>kubectl &gt;= 1.12:<br>kubectl sniff &lt;POD_NAME&gt; [-n &lt;NAMESPACE_NAME&gt;] [-c &lt;CONTAINER_NAME&gt;] [-i &lt;INTERFACE_NAME&gt;] [-f &lt;CAPTURE_FILTER&gt;] [-o OUTPUT_FILE] [-l LOCAL_TCPDUMP_FILE] [-r REMOTE_TCPDUMP_FILE]</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

如下：

[![kubectl sniff](https://pic-cdn.ewhisper.cn/img/2021/11/27/3ce36baded0d9ed48e5fd39f155d3e6f-sniff-demo.gif)](https://pic-cdn.ewhisper.cn/img/2021/11/27/3ce36baded0d9ed48e5fd39f155d3e6f-sniff-demo.gif "kubectl sniff")

### [starboard](https://github.com/aquasecurity/starboard)[](https://ewhisper.cn/posts/60907/#starboard)

也是一个安全扫描工具。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-20)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install starboard<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-18)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl starboard report deployment/nginx &gt; nginx.deploy.html<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

就可以生成一份安全报告：

[![starboard](https://pic-cdn.ewhisper.cn/img/2021/11/27/2e6c42a3753777da3660b104525bbe45-html-report.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/2e6c42a3753777da3660b104525bbe45-html-report.png "starboard")

### `tail` - kubernetes tail[](https://ewhisper.cn/posts/60907/#tail-kubernetes-tail)

Kubernetes tail。将所有匹配 pod 的所有容器的日志流。按 service、replicaset、deployment 等匹配 pod。调整到变化的集群——当 pod 落入或退出选择时，将从日志中添加或删除它们。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-21)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew <span>install</span> tail<br></code><p><i></i>CMAKE</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-19)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span># 匹配所有 pod</span><br>$ kubectl tail<p><span># 配置 staging ns 的所有 pod</span><br>$ kubectl tail --ns staging</p><p><span># 匹配所有 ns 的 rs name 为 worker 的 pod</span><br>$ kubectl tail --rs workers</p><p><span># 匹配 staging ns 的 rs name 为 worker 的 pod</span><br>$ kubectl tail --rs staging/workers</p><p><span># 匹配 deploy 属于 webapp，且 svc 属于 frontend 的 pod</span><br>$ kubectl tail --svc frontend --deploy webapp</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

使用效果如下，最前面会加上日志对应的 pod：

[![tail 效果](https://pic-cdn.ewhisper.cn/img/2021/11/27/c674b1c42b8b8e484d9a8766c142a3d1-image-20211127131559251.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/c674b1c42b8b8e484d9a8766c142a3d1-image-20211127131559251.png "tail 效果")

### [trace](https://github.com/iovisor/kubectl-trace)[](https://ewhisper.cn/posts/60907/#trace)

使用系统工具跟踪 Kubernetes pod 和 node。

kubectl trace 是一个 kubectl 插件，它允许你在 Kubernetes 集群中调度 bpftrace 程序的执行。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-22)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install trace<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-20)

这块不太了解，就不多做评论了。

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl trace run ip-180-12-0-152.ec2.internal -f read.bt<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### `tree`[](https://ewhisper.cn/posts/60907/#tree)

[![](https://pic-cdn.ewhisper.cn/img/2021/11/27/2b78802b13b8b22567dc897c4de9a9a5-kubetree-logo.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/2b78802b13b8b22567dc897c4de9a9a5-kubetree-logo.png)

一个 `kubectl` 插件，通过对 Kubernetes 对象的 `ownersReferences` 来探索它们之间的所有权关系。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-23)

使用 [krew](https://krew.sigs.k8s.io/) 插件管理器安装：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>kubectl krew install tree<br>kubectl tree --<span>help</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-21)

DaemonSet 示例：

[![DaemonSet  Tree](https://pic-cdn.ewhisper.cn/img/2021/11/27/f1da2a686f138cd5c9f0b5a240566075-image-20211127125645923.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/f1da2a686f138cd5c9f0b5a240566075-image-20211127125645923.png "DaemonSet  Tree")

Knative Service 示例：

[![Knative Service](https://pic-cdn.ewhisper.cn/img/2021/11/27/b7587d334065ea9f5994b124ab4930cd-20211127125757.png)](https://pic-cdn.ewhisper.cn/img/2021/11/27/b7587d334065ea9f5994b124ab4930cd-20211127125757.png "Knative Service")

### [tunnel](https://github.com/omrikiei/ktunnel)[](https://ewhisper.cn/posts/60907/#tunnel)

集群和你自己机器之间的反向隧道.

它允许您将计算机作为集群中的服务公开，或者将其公开给特定的部署。这个项目的目的是为这个特定的问题提供一个整体的解决方案(从 kubernetes pod 访问本地机器)。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-24)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install tunnel<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-22)

以下命令将允许集群中的 pod 通过 http 访问您的本地 web 应用程序(监听端口 8000)(即 kubernetes 应用程序可以发送请求到 myapp:8000)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>ktunnel expose myapp 80:8000<br>ktunnel expose myapp 80:8000 -r <span>#deployment &amp; service will be reused if exists or they will be created</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [warp](https://github.com/ernoaapa/kubectl-warp)[](https://ewhisper.cn/posts/60907/#warp)

在 Pod 中同步和执行本地文件

kubectl (Kubernetes CLI)插件，就像 kubectl 运行与 rsync。

它创建临时 Pod，并将本地文件同步到所需的容器，并执行任何命令。

例如，这可以用于在 Kubernetes 中构建和运行您的本地项目，其中有更多的资源、所需的架构等，同时在本地使用您的首选编辑器。

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-25)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install warp<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-23)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code><span># 在 ubuntu 镜像中启动 bash。并将当前目录中的文件同步到容器中</span><br>kubectl warp -i -t --image ubuntu testing -- /bin/bash<p><span># 在 node 容器中启动 nodejs 项目</span><br><span>cd</span> examples/nodejs<br>kubectl warp -i -t --image node testing-node -- npm run watch</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### [who-can](https://github.com/aquasecurity/kubectl-who-can)[](https://ewhisper.cn/posts/60907/#who-can)

显示谁具有访问 Kubernetes 资源的 RBAC 权限

#### 安装[](https://ewhisper.cn/posts/60907/#%E5%AE%89%E8%A3%85%20-26)

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl krew install who-can<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

#### 使用[](https://ewhisper.cn/posts/60907/#%E4%BD%BF%E7%94%A8%20-24)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><code>$ kubectl who-can create ns all-namespaces<br>No subjects found with permissions to create ns assigned through RoleBindings<p>CLUSTERROLEBINDING            SUBJECT           TYPE            SA-NAMESPACE<br>cluster-admin                 system:masters    Group<br>helm-kube-system-traefik-crd  helm-traefik-crd  ServiceAccount  kube-system<br>helm-kube-system-traefik      helm-traefik      ServiceAccount  kube-system</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

EOF