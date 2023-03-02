## 概述[](https://ewhisper.cn/posts/33147/#%E6%A6%82%E8%BF%B0)

有时候我们操作不规范，或者删除的先后顺序有问题，或者某项关键服务没有启动，导致 Kubernetes 经常会出现无法删除 NameSpace 的情况。这种情况下我们应该怎么办？

## 规范删除流程[](https://ewhisper.cn/posts/33147/#%E8%A7%84%E8%8C%83%E5%88%A0%E9%99%A4%E6%B5%81%E7%A8%8B)

其实，很多时候出现这种情况，主要是因为我们的删除操作不规范，典型的有下面几种情况：

-   删除的先后顺序有问题，如：
    -   先删除了 Traefik 的关键组件，再尝试删除包含 Traefik Ingress 或 EdgeIngress 的 CRD
-   某项关键服务没有启动，如：
    -   对于安装了 Prometheus Operator + custom adapter 的 Kubernetes 集群，在 Prometheus 的一些关键组件 scale down 的情况下，删除包含这些监控 CRD 或 HPA custom metric 的 NameSpace

…

综上，根源上，大部分情况下 NameSpace 无法删除，都是我们操作有错在先。

为了避免此类错误再犯，推荐搭建删除按照如下流程：

1.  保证所有基础服务组件都是正常运行的状态（如前面提到的，ingress 组件，监控组件，servicemesh 组件。…)
2.  检查要删除的 NameSpace 下的所有资源，**特别是 CRD**, 这里推荐使用 [Krew - Kubernetes 的 CLI 插件管理器](https://ewhisper.cn/posts/60907/) 安装 [`get-all`](https://github.com/corneliusweig/ketall) 来 **真正** 地获取该 NameSpace 下的所有资源，如后面的代码块所示：
3.  针对其中的一些 CRD 或特殊资源，最好先明确指定删除并确保可以成功删除掉
4.  最后，再删除该 NameSpace

第 2 步的代码块：（有如此多的 CRD)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br></pre></td><td><pre><code>❯ kubectl get-all -n cert-manager<br>NAME                                                                              NAMESPACE     AGE<br>configmap/cert-manager-webhook                                                    cert-manager  277d<br>configmap/kube-root-ca.crt                                                        cert-manager  277d<br>endpoints/cert-manager                                                            cert-manager  277d<br>endpoints/cert-manager-webhook                                                    cert-manager  277d<br>endpoints/cert-manager-webhook-dnspod                                             cert-manager  277d<br>pod/cert-manager-6d6bb4f487-hkwpn                                                 cert-manager  85d<br>pod/cert-manager-6d6bb4f487-wgtd8                                                 cert-manager  85d<br>pod/cert-manager-cainjector-7d55bf8f78-5797c                                      cert-manager  277d<br>pod/cert-manager-webhook-577f77586f-txlcx                                         cert-manager  85d<br>pod/cert-manager-webhook-577f77586f-xh4st                                         cert-manager  85d<br>pod/cert-manager-webhook-dnspod-5d5566c7bc-5cj4s                                  cert-manager  211d<br>secret/cert-manager-cainjector-token-h8cqq                                        cert-manager  277d<br>secret/cert-manager-token-28knj                                                   cert-manager  277d<br>secret/cert-manager-webhook-ca                                                    cert-manager  277d<br>secret/cert-manager-webhook-dnspod-ca                                             cert-manager  277d<br>secret/cert-manager-webhook-dnspod-letsencrypt                                    cert-manager  277d<br>secret/cert-manager-webhook-dnspod-secret                                         cert-manager  277d<br>secret/cert-manager-webhook-dnspod-token-jsjrn                                    cert-manager  277d<br>secret/cert-manager-webhook-dnspod-webhook-tls                                    cert-manager  277d<br>secret/cert-manager-webhook-token-qxq44                                           cert-manager  277d<br>secret/default-token-mkpmt                                                        cert-manager  277d<br>secret/ewhisper-crt-secret                                                        cert-manager  277d<br>secret/sh.helm.release.v1.cert-manager-webhook-dnspod.v1                          cert-manager  277d<br>secret/sh.helm.release.v1.cert-manager.v1                                         cert-manager  277d<br>serviceaccount/cert-manager                                                       cert-manager  277d<br>serviceaccount/cert-manager-cainjector                                            cert-manager  277d<br>serviceaccount/cert-manager-webhook                                               cert-manager  277d<br>serviceaccount/cert-manager-webhook-dnspod                                        cert-manager  277d<br>serviceaccount/default                                                            cert-manager  277d<br>service/cert-manager                                                              cert-manager  277d<br>service/cert-manager-webhook                                                      cert-manager  277d<br>service/cert-manager-webhook-dnspod                                               cert-manager  277d<br>order.acme.cert-manager.io/ewhisper-crt-6v6s4-2449993249                          cert-manager  209d<br>order.acme.cert-manager.io/ewhisper-crt-89n7g-2449993249                          cert-manager  23d<br>order.acme.cert-manager.io/ewhisper-crt-8g496-2449993249                          cert-manager  277d<br>order.acme.cert-manager.io/ewhisper-crt-jj24l-2449993249                          cert-manager  83d<br>order.acme.cert-manager.io/ewhisper-crt-q8pvw-2449993249                          cert-manager  149d<br>deployment.apps/cert-manager                                                      cert-manager  277d<br>deployment.apps/cert-manager-cainjector                                           cert-manager  277d<br>deployment.apps/cert-manager-webhook                                              cert-manager  277d<br>deployment.apps/cert-manager-webhook-dnspod                                       cert-manager  277d<br>replicaset.apps/cert-manager-6d6bb4f487                                           cert-manager  277d<br>replicaset.apps/cert-manager-cainjector-7d55bf8f78                                cert-manager  277d<br>replicaset.apps/cert-manager-webhook-577f77586f                                   cert-manager  277d<br>replicaset.apps/cert-manager-webhook-dnspod-5d5566c7bc                            cert-manager  211d<br>replicaset.apps/cert-manager-webhook-dnspod-5d78f9bfcb                            cert-manager  217d<br>replicaset.apps/cert-manager-webhook-dnspod-7c5cd575fc                            cert-manager  277d<br>app.catalog.cattle.io/cert-manager                                                cert-manager  270d<br>app.catalog.cattle.io/cert-manager-webhook-dnspod                                 cert-manager  270d<br>certificaterequest.cert-manager.io/cert-manager-webhook-dnspod-ca-l57hl           cert-manager  277d<br>certificaterequest.cert-manager.io/cert-manager-webhook-dnspod-webhook-tls-7zwdh  cert-manager  277d<br>certificaterequest.cert-manager.io/cert-manager-webhook-dnspod-webhook-tls-gs57f  cert-manager  34d<br>certificaterequest.cert-manager.io/ewhisper-crt-6v6s4                             cert-manager  209d<br>certificaterequest.cert-manager.io/ewhisper-crt-89n7g                             cert-manager  23d<br>certificaterequest.cert-manager.io/ewhisper-crt-8g496                             cert-manager  277d<br>certificaterequest.cert-manager.io/ewhisper-crt-jj24l                             cert-manager  83d<br>certificaterequest.cert-manager.io/ewhisper-crt-q8pvw                             cert-manager  149d<br>certificate.cert-manager.io/cert-manager-webhook-dnspod-ca                        cert-manager  277d<br>certificate.cert-manager.io/cert-manager-webhook-dnspod-webhook-tls               cert-manager  277d<br>certificate.cert-manager.io/ewhisper-crt                                          cert-manager  277d<br>issuer.cert-manager.io/cert-manager-webhook-dnspod-ca                             cert-manager  277d<br>issuer.cert-manager.io/cert-manager-webhook-dnspod-selfsign                       cert-manager  277d<br>endpointslice.discovery.k8s.io/cert-manager-9lm6j                                 cert-manager  277d<br>endpointslice.discovery.k8s.io/cert-manager-webhook-dnspod-q7f8n                  cert-manager  277d<br>endpointslice.discovery.k8s.io/cert-manager-webhook-z6qdd                         cert-manager  277d<br>rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving        cert-manager  277d<br>role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving               cert-manager  277d<br>ingressroute.traefik.containo.us/alertmanager                                     cert-manager  244d<br>ingressroute.traefik.containo.us/grafana                                          cert-manager  255d<br>ingressroute.traefik.containo.us/grafana-rancher                                  cert-manager  238d<br>ingressroute.traefik.containo.us/prometheus                                       cert-manager  244d<br>ingressroute.traefik.containo.us/rsshub                                           cert-manager  268d<br>ingressroute.traefik.containo.us/ttrss                                            cert-manager  257d<br>tlsstore.traefik.containo.us/default                                              cert-manager  268d<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 试试强制删除[](https://ewhisper.cn/posts/33147/#%E8%AF%95%E8%AF%95%E5%BC%BA%E5%88%B6%E5%88%A0%E9%99%A4)

如果 NameSpace 已经处于 `terminating` 的状态，且久久无法删除，可以试试加上这 2 个参数强制删除：

-   `--force`
-   `--grace-period=0`

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl delete ns <span>${NAMESPACE}</span> --force --grace-period=0<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 强制删除失败？再来试试这种办法[](https://ewhisper.cn/posts/33147/#%E5%BC%BA%E5%88%B6%E5%88%A0%E9%99%A4%E5%A4%B1%E8%B4%A5%EF%BC%9F%E5%86%8D%E6%9D%A5%E8%AF%95%E8%AF%95%E8%BF%99%E7%A7%8D%E5%8A%9E%E6%B3%95)

强制删除失败？再来试试这种办法：**调用 Kubernetes API 删除**

### Hard Way 步骤[](https://ewhisper.cn/posts/33147/#Hard-Way-%20%E6%AD%A5%E9%AA%A4)

首先，获取要删除 NameSpace 的 JSON 文件：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>NAMESPACE=cert-manager<br>kubectl get ns <span>${NAMESPACE}</span> -o json &gt; namespace.json<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

然后，编辑 `namespace.json`, 从 `finalizers` 字段中删除 `kubernetes` 的值并保存，示例如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br></pre></td><td><pre><code><span>{</span><br>    <span>"apiVersion"</span><span>:</span> <span>"v1"</span><span>,</span><br>    <span>"kind"</span><span>:</span> <span>"Namespace"</span><span>,</span><br>    <span>"metadata"</span><span>:</span> <span>{</span><br>        ...<span>:</span> ...<br>    <span>}</span><span>,</span><br>    <span>"spec"</span><span>:</span> <span>{</span><br>        <span>"finalizers"</span><span>:</span> <span>[</span><span>]</span><br>    <span>}</span><span>,</span><br>    <span>"status"</span><span>:</span> <span>{</span><br>        <span>"phase"</span><span>:</span> <span>"Terminating"</span><br>    <span>}</span><br><span>}</span><br></code><p><i></i>JSON</p></pre></td></tr></tbody></table>

之后，可以通过 `kubectl proxy` 设置 APIServer 的临时 IP 和端口

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl proxy --port=6880 &amp;<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

最后，进行 API 调用来强制删除：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>curl -k -H <span>"Content-Type: application/json"</span> -X  PUT --data-binary @namespace.json http://127.0.0.1:6880/api/v1/namespaces/<span>${NAMESPACE}</span>/finalize<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

验证是否已经成功删除：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>kubectl get ns <span>${NAMESPACE}</span><br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### 编排成脚本[](https://ewhisper.cn/posts/33147/#%E7%BC%96%E6%8E%92%E6%88%90%E8%84%9A%E6%9C%AC)

> 📝**Notes:**
> 
> 依赖组件：
> 
> -   kubectl
> -   jq
> -   curl

`force-delete-ns.sh`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><br><span>set</span> -ex<br>PATH=<span>$PATH</span>:.<br>NAMESPACE=<span>$1</span>    <span># 读取命令行第一个参数</span><br><span>kill</span> -9  $(ps -ef|grep proxy|grep -v grep |awk <span>'{print $2}'</span>)<br>kubectl proxy --port=6880 &amp;<br>kubectl get namespace <span>${NAMESPACE}</span> -o json |jq <span>'.spec = {"finalizers":[]}'</span> &gt; namespace.json<br>curl -k -H  <span>"Content-Type: application/json"</span>  -X  PUT --data-binary @namespace.json 127.0.0.1:6880/api/v1/namespaces/<span>${NAMESPACE}</span>/finalize<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

使用方式示例：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>bash force-delete-ns.sh cert-manager<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

🎉🎉🎉

## 总结[](https://ewhisper.cn/posts/33147/#%E6%80%BB%E7%BB%93)

经常会碰到 Kubernetes 的 NameSpace 无法删除的情况，这时候应该如何解决？这里提供了 3 种方案：

1.  尽量不要出现上面这种情况 (😑额。… 废话）
2.  加上 `--force` flag 强制删除
3.  调用 namespace 的 finalize API 强制删除

但是，真到了需要强制删除的阶段，2/3 部是无法保证 100% 成功的。  
所以第一步才是正道 …（呆，但是有用）

EOF