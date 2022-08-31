## 目录

1.  [Pod Security Policies](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#pod-security-policies)
    1.  [启用 Pod Security Policies](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%90%AF%E7%94%A8-pod-security-policies)
        1.  [创建安全策略资源](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%88%9B%E5%BB%BA%E5%AE%89%E5%85%A8%E7%AD%96%E7%95%A5%E8%B5%84%E6%BA%90)
        2.  [RBAC 身份认证](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#rbac-%E8%BA%AB%E4%BB%BD%E8%AE%A4%E8%AF%81)
        3.  [启用 admission controller 插件](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%90%AF%E7%94%A8-admission-controller-%E6%8F%92%E4%BB%B6)
    2.  [验证 psp 的安全限制](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E9%AA%8C%E8%AF%81-psp-%E7%9A%84%E5%AE%89%E5%85%A8%E9%99%90%E5%88%B6)
2.  [Pod Security Admission](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#pod-security-admission)
    1.  [为什么要替换 psp](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9B%BF%E6%8D%A2-psp)
    2.  [工作方式](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F)
    3.  [在旧版本集群中启用 psa](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%9C%A8%E6%97%A7%E7%89%88%E6%9C%AC%E9%9B%86%E7%BE%A4%E4%B8%AD%E5%90%AF%E7%94%A8-psa)
    4.  [验证 psa 的安全限制](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E9%AA%8C%E8%AF%81-psa-%E7%9A%84%E5%AE%89%E5%85%A8%E9%99%90%E5%88%B6)
3.  [参考链接](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

最近有客户反馈在开启了安全策略的集群中部署产品失败，因此研究了一下 Kubernetes 提供的 pod 安全策略。

文中的演示和示例均在 v1.18.17 集群中通过验证。

## [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#pod-security-policies)[Pod Security Policies](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:pod-security-policies)

**Pod Security Policies** （下文简称 psp 或 pod 安全策略）是一种集群级别的全局资源，能够对 pod 的创建和更新进行细粒度的授权控制。具体来说，一个 psp 对象定义了一组安全性条件，一个 pod 的 spec 字段必须满足这些条件以及适用相关字段的默认值，其创建或更新请求才会被 apiserver 所接受。

具体的 pod 字段和安全条件可见文档 [what-is-a-pod-security-policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy) 。

### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%90%AF%E7%94%A8-pod-security-policies)[启用 Pod Security Policies](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E5%90%AF%E7%94%A8-pod-security-policies)

Kubernetes 默认不开启 pod 安全策略功能，在集群中启用 pod 安全策略的步骤大体上分为三步：

1.  在集群中创建指定的安全策略资源。
2.  通过 RBAC 机制授予**创建 pod 的 user** 或者**被创建 pod 的 service account** 使用安全策略资源的权限，通常会将使用权限授予一组 users 或 service accounts。
3.  启用 apiserver 的 admission-controller 插件。

注意步骤 1、2 可以单独执行，因为它们不会对集群产生实际影响，但需要确保步骤 3 在前两步之后执行。

因为一旦启用 admission-controller 插件，apiserver 会对所有的 pod 创建/更新请求强制执行安全策略检查，如果集群中没有可用的 pod 安全策略资源或者未对安全策略资源预先授权，所有的 pod 创建/更新请求都会被拒绝。包括 kube-system 命名空间下的系统管理组件如 apiserver 本身（由于 apiserver 是受 kubelet 管理的静态 pod，实际上容器依然会运行）。

启用的整体流程如下示意图：

![podsecuritypolicy](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/2022-04-11-podsecuritypolicy-diagrams.png)

#### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%88%9B%E5%BB%BA%E5%AE%89%E5%85%A8%E7%AD%96%E7%95%A5%E8%B5%84%E6%BA%90)[创建安全策略资源](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E5%88%9B%E5%BB%BA%E5%AE%89%E5%85%A8%E7%AD%96%E7%95%A5%E8%B5%84%E6%BA%90)

1.  在集群中创建一个宽松限制的 PodSecurityPolicy 资源，命名为 `privileged`。
    
    <table><tbody><tr><td><pre tabindex="0"><code><span> 1
    </span><span> 2
    </span><span> 3
    </span><span> 4
    </span><span> 5
    </span><span> 6
    </span><span> 7
    </span><span> 8
    </span><span> 9
    </span><span>10
    </span><span>11
    </span><span>12
    </span><span>13
    </span><span>14
    </span><span>15
    </span><span>16
    </span><span>17
    </span><span>18
    </span><span>19
    </span><span>20
    </span><span>21
    </span><span>22
    </span><span>23
    </span><span>24
    </span><span>25
    </span><span>26
    </span><span>27
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>policy/v1beta1</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>PodSecurityPolicy</span><span>
    </span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>privileged</span><span>
    </span></span></span><span><span><span>  </span><span>annotations</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>seccomp.security.alpha.kubernetes.io/allowedProfileNames</span><span>:</span><span> </span><span>'*'</span><span>
    </span></span></span><span><span><span></span><span>spec</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>privileged</span><span>:</span><span> </span><span>true</span><span>
    </span></span></span><span><span><span>  </span><span>allowPrivilegeEscalation</span><span>:</span><span> </span><span>true</span><span>
    </span></span></span><span><span><span>  </span><span>allowedCapabilities</span><span>:</span><span>
    </span></span></span><span><span><span>  </span>- <span>'*'</span><span>
    </span></span></span><span><span><span>  </span><span>volumes</span><span>:</span><span>
    </span></span></span><span><span><span>  </span>- <span>'*'</span><span>
    </span></span></span><span><span><span>  </span><span>hostNetwork</span><span>:</span><span> </span><span>true</span><span>
    </span></span></span><span><span><span>  </span><span>hostPorts</span><span>:</span><span>
    </span></span></span><span><span><span>  </span>- <span>min</span><span>:</span><span> </span><span>0</span><span>
    </span></span></span><span><span><span>    </span><span>max</span><span>:</span><span> </span><span>65535</span><span>
    </span></span></span><span><span><span>  </span><span>hostIPC</span><span>:</span><span> </span><span>true</span><span>
    </span></span></span><span><span><span>  </span><span>hostPID</span><span>:</span><span> </span><span>true</span><span>
    </span></span></span><span><span><span>  </span><span>runAsUser</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'RunAsAny'</span><span>
    </span></span></span><span><span><span>  </span><span>seLinux</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'RunAsAny'</span><span>
    </span></span></span><span><span><span>  </span><span>supplementalGroups</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'RunAsAny'</span><span>
    </span></span></span><span><span><span>  </span><span>fsGroup</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'RunAsAny'</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    
2.  在集群中创建一个严格限制的 PodSecurityPolicy 资源，命名为 `restricted`。
    
    <table><tbody><tr><td><pre tabindex="0"><code><span> 1
    </span><span> 2
    </span><span> 3
    </span><span> 4
    </span><span> 5
    </span><span> 6
    </span><span> 7
    </span><span> 8
    </span><span> 9
    </span><span>10
    </span><span>11
    </span><span>12
    </span><span>13
    </span><span>14
    </span><span>15
    </span><span>16
    </span><span>17
    </span><span>18
    </span><span>19
    </span><span>20
    </span><span>21
    </span><span>22
    </span><span>23
    </span><span>24
    </span><span>25
    </span><span>26
    </span><span>27
    </span><span>28
    </span><span>29
    </span><span>30
    </span><span>31
    </span><span>32
    </span><span>33
    </span><span>34
    </span><span>35
    </span><span>36
    </span><span>37
    </span><span>38
    </span><span>39
    </span><span>40
    </span><span>41
    </span><span>42
    </span><span>43
    </span><span>44
    </span><span>45
    </span><span>46
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>policy/v1beta1</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>PodSecurityPolicy</span><span>
    </span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>restricted</span><span>
    </span></span></span><span><span><span>  </span><span>annotations</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>seccomp.security.alpha.kubernetes.io/allowedProfileNames</span><span>:</span><span> </span><span>'docker/default,runtime/default'</span><span>
    </span></span></span><span><span><span>    </span><span>apparmor.security.beta.kubernetes.io/allowedProfileNames</span><span>:</span><span> </span><span>'runtime/default'</span><span>
    </span></span></span><span><span><span>    </span><span>apparmor.security.beta.kubernetes.io/defaultProfileName</span><span>:</span><span>  </span><span>'runtime/default'</span><span>
    </span></span></span><span><span><span></span><span>spec</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>privileged</span><span>:</span><span> </span><span>false</span><span>
    </span></span></span><span><span><span>  </span><span># Required to prevent escalations to root.</span><span>
    </span></span></span><span><span><span>  </span><span>allowPrivilegeEscalation</span><span>:</span><span> </span><span>false</span><span>
    </span></span></span><span><span><span>  </span><span>requiredDropCapabilities</span><span>:</span><span>
    </span></span></span><span><span><span>    </span>- <span>ALL</span><span>
    </span></span></span><span><span><span>  </span><span># Allow core volume types.</span><span>
    </span></span></span><span><span><span>  </span><span>volumes</span><span>:</span><span>
    </span></span></span><span><span><span>    </span>- <span>'configMap'</span><span>
    </span></span></span><span><span><span>    </span>- <span>'emptyDir'</span><span>
    </span></span></span><span><span><span>    </span>- <span>'projected'</span><span>
    </span></span></span><span><span><span>    </span>- <span>'secret'</span><span>
    </span></span></span><span><span><span>    </span>- <span>'downwardAPI'</span><span>
    </span></span></span><span><span><span>    </span><span># Assume that ephemeral CSI drivers &amp; persistentVolumes set up by the cluster admin are safe to use.</span><span>
    </span></span></span><span><span><span>    </span>- <span>'csi'</span><span>
    </span></span></span><span><span><span>    </span>- <span>'persistentVolumeClaim'</span><span>
    </span></span></span><span><span><span>  </span><span>hostNetwork</span><span>:</span><span> </span><span>false</span><span>
    </span></span></span><span><span><span>  </span><span>hostIPC</span><span>:</span><span> </span><span>false</span><span>
    </span></span></span><span><span><span>  </span><span>hostPID</span><span>:</span><span> </span><span>false</span><span>
    </span></span></span><span><span><span>  </span><span>runAsUser</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span># Require the container to run without root privileges.</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'MustRunAsNonRoot'</span><span>
    </span></span></span><span><span><span>  </span><span>seLinux</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span># This policy assumes the nodes are using AppArmor rather than SELinux.</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'RunAsAny'</span><span>
    </span></span></span><span><span><span>  </span><span>supplementalGroups</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'MustRunAs'</span><span>
    </span></span></span><span><span><span>    </span><span>ranges</span><span>:</span><span>
    </span></span></span><span><span><span>      </span><span># Forbid adding the root group.</span><span>
    </span></span></span><span><span><span>      </span>- <span>min</span><span>:</span><span> </span><span>1</span><span>
    </span></span></span><span><span><span>        </span><span>max</span><span>:</span><span> </span><span>65535</span><span>
    </span></span></span><span><span><span>  </span><span>fsGroup</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>rule</span><span>:</span><span> </span><span>'MustRunAs'</span><span>
    </span></span></span><span><span><span>    </span><span>ranges</span><span>:</span><span>
    </span></span></span><span><span><span>      </span><span># Forbid adding the root group.</span><span>
    </span></span></span><span><span><span>      </span>- <span>min</span><span>:</span><span> </span><span>1</span><span>
    </span></span></span><span><span><span>        </span><span>max</span><span>:</span><span> </span><span>65535</span><span>
    </span></span></span><span><span><span>  </span><span>readOnlyRootFilesystem</span><span>:</span><span> </span><span>false</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    

#### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#rbac-%E8%BA%AB%E4%BB%BD%E8%AE%A4%E8%AF%81)[RBAC 身份认证](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:rbac-%E8%BA%AB%E4%BB%BD%E8%AE%A4%E8%AF%81)

1.  分别创建可访问两种安全策略资源的 ClusterRole：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span> 1
    </span><span> 2
    </span><span> 3
    </span><span> 4
    </span><span> 5
    </span><span> 6
    </span><span> 7
    </span><span> 8
    </span><span> 9
    </span><span>10
    </span><span>11
    </span><span>12
    </span><span>13
    </span><span>14
    </span><span>15
    </span><span>16
    </span><span>17
    </span><span>18
    </span><span>19
    </span><span>20
    </span><span>21
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>rbac.authorization.k8s.io/v1</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>ClusterRole</span><span>
    </span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>privileged-psp</span><span>
    </span></span></span><span><span><span></span><span>rules</span><span>:</span><span>
    </span></span></span><span><span><span></span>- <span>apiGroups</span><span>:</span><span> </span><span>[</span><span>'policy'</span><span>]</span><span>
    </span></span></span><span><span><span>  </span><span>resources</span><span>:</span><span> </span><span>[</span><span>'podsecuritypolicies'</span><span>]</span><span>
    </span></span></span><span><span><span>  </span><span>verbs</span><span>:</span><span>     </span><span>[</span><span>'use'</span><span>]</span><span>
    </span></span></span><span><span><span>  </span><span>resourceNames</span><span>:</span><span>
    </span></span></span><span><span><span>  </span>- <span>privileged</span><span>
    </span></span></span><span><span><span></span><span>---</span><span>
    </span></span></span><span><span><span></span><span>apiVersion</span><span>:</span><span> </span><span>rbac.authorization.k8s.io/v1</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>ClusterRole</span><span>
    </span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>restricted-psp</span><span>
    </span></span></span><span><span><span></span><span>rules</span><span>:</span><span>
    </span></span></span><span><span><span></span>- <span>apiGroups</span><span>:</span><span> </span><span>[</span><span>'policy'</span><span>]</span><span>
    </span></span></span><span><span><span>  </span><span>resources</span><span>:</span><span> </span><span>[</span><span>'podsecuritypolicies'</span><span>]</span><span>
    </span></span></span><span><span><span>  </span><span>verbs</span><span>:</span><span>     </span><span>[</span><span>'use'</span><span>]</span><span>
    </span></span></span><span><span><span>  </span><span>resourceNames</span><span>:</span><span>
    </span></span></span><span><span><span>  </span>- <span>restricted</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    
2.  通过 ClusterRoleBinding （或者 RoleBinding）将创建的 ClusterRole 绑定到指定命名空间下的所有 service account（也可以授权给指定的 user）。
    
    在 Kubernetes 中大多数 pod 并不是直接使用 user 创建的，而是通常作为 Deployment、ReplicaSet 或其他模板 controller 的子资源，由 controller 间接创建。授予 controller 用户对安全策略的使用权等同于为该 controller 创建的所有 pod 授予使用权，因此授权的推荐做法是授权给目标 pod 的 service account。
    
    为了进行后续的测试，我们将 `privileged-psp` 授权给 kubelet 所使用的 `system:nodes` 用户和 `privileged-ns` 命名空间下的所有 service account，将 `restricted-psp` 授权给 `restricted-ns` 命名空间下的所有 service account：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span> 1
    </span><span> 2
    </span><span> 3
    </span><span> 4
    </span><span> 5
    </span><span> 6
    </span><span> 7
    </span><span> 8
    </span><span> 9
    </span><span>10
    </span><span>11
    </span><span>12
    </span><span>13
    </span><span>14
    </span><span>15
    </span><span>16
    </span><span>17
    </span><span>18
    </span><span>19
    </span><span>20
    </span><span>21
    </span><span>22
    </span><span>23
    </span><span>24
    </span><span>25
    </span><span>26
    </span><span>27
    </span><span>28
    </span><span>29
    </span><span>30
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>rbac.authorization.k8s.io/v1</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>ClusterRoleBinding</span><span>
    </span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>privileged-psp-bind</span><span>
    </span></span></span><span><span><span></span><span>roleRef</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>kind</span><span>:</span><span> </span><span>ClusterRole</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>privileged-psp</span><span>
    </span></span></span><span><span><span>  </span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span></span><span>subjects</span><span>:</span><span>
    </span></span></span><span><span><span></span><span># 授权给指定命名空间下的所有 service account（推荐做法）:</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>Group</span><span>
    </span></span></span><span><span><span>  </span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>system:serviceaccounts:privileged-ns</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>Group</span><span>
    </span></span></span><span><span><span>  </span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>system:nodes</span><span>
    </span></span></span><span><span><span>  </span><span>namespace</span><span>:</span><span> </span><span>kube-system</span><span>
    </span></span></span><span><span><span></span><span>---</span><span>
    </span></span></span><span><span><span></span><span>apiVersion</span><span>:</span><span> </span><span>rbac.authorization.k8s.io/v1</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>ClusterRoleBinding</span><span>
    </span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>restricted-psp-bind</span><span>
    </span></span></span><span><span><span></span><span>roleRef</span><span>:</span><span>
    </span></span></span><span><span><span>  </span><span>kind</span><span>:</span><span> </span><span>ClusterRole</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>restricted-psp</span><span>
    </span></span></span><span><span><span>  </span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span></span><span>subjects</span><span>:</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>Group</span><span>
    </span></span></span><span><span><span>  </span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>system:serviceaccounts:restricted-ns</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    
    在 `subjects` 字段下添加更多记录还可以授权给所有的 service account 或者所有已授权的 user：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span> 1
    </span><span> 2
    </span><span> 3
    </span><span> 4
    </span><span> 5
    </span><span> 6
    </span><span> 7
    </span><span> 8
    </span><span> 9
    </span><span>10
    </span><span>11
    </span><span>12
    </span><span>13
    </span><span>14
    </span><span>15
    </span><span>16
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>subjects</span><span>:</span><span>
    </span></span></span><span><span><span></span><span># 授权给指定的 service account 或者用户（不推荐）:</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>ServiceAccount</span><span>
    </span></span></span><span><span><span></span><span>name</span><span>:</span><span> </span><span>&lt;authorized service account name&gt;</span><span>
    </span></span></span><span><span><span></span><span>namespace</span><span>:</span><span> </span><span>&lt;authorized pod namespace&gt;</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>User</span><span>
    </span></span></span><span><span><span></span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span></span><span>name</span><span>:</span><span> </span><span>&lt;authorized user name&gt;</span><span>
    </span></span></span><span><span><span></span><span># 授权给所有的 service accounts:</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>Group</span><span>
    </span></span></span><span><span><span></span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span></span><span>name</span><span>:</span><span> </span><span>system:serviceaccounts</span><span>
    </span></span></span><span><span><span></span><span># 授权给所有已认证的用户:</span><span>
    </span></span></span><span><span><span></span>- <span>kind</span><span>:</span><span> </span><span>Group</span><span>
    </span></span></span><span><span><span></span><span>apiGroup</span><span>:</span><span> </span><span>rbac.authorization.k8s.io</span><span>
    </span></span></span><span><span><span></span><span>name</span><span>:</span><span> </span><span>system:authenticated</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    

#### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%90%AF%E7%94%A8-admission-controller-%E6%8F%92%E4%BB%B6)[启用 admission controller 插件](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E5%90%AF%E7%94%A8-admission-controller-%E6%8F%92%E4%BB%B6)

在 apiserver 启用 admission controller 的 psp 插件有两种方式：

1.  在已存在的集群中通过修改 apiserver 的静态 manifest 文件，为 apiserver 增加启动参数 `enable-admission-plugins=PodSecurityPolicy`。kubelet 会自动检测到变更并重启 apiserver。下面的示例使用 sed 对原有参数进行了替换：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>sed -i <span>'s/enable-admission-plugins=NodeRestriction/enable-admission-plugins=NodeRestriction,PodSecurityPolicy/'</span> /etc/kubernetes/manifests/kube-apiserver.yaml
    </span></span></code></pre></td></tr></tbody></table>
    
2.  或者在初始化集群时，在 kubeadm 配置文件中添加额外参数（不推荐，默认会拒绝所有 pod 的创建）。
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span><span>2
    </span><span>3
    </span><span>4
    </span><span>5
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>kubeadm.k8s.io/v1beta2</span><span>
    </span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>ClusterConfiguration</span><span>
    </span></span></span><span><span><span></span><span>apiServer</span><span>:</span><span>
    </span></span></span><span><span><span></span><span>extraArgs</span><span>:</span><span>
    </span></span></span><span><span><span>    </span><span>enable-admission-plugins</span><span>:</span><span> </span><span>"PodSecurityPolicy"</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    

### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E9%AA%8C%E8%AF%81-psp-%E7%9A%84%E5%AE%89%E5%85%A8%E9%99%90%E5%88%B6)[验证 psp 的安全限制](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E9%AA%8C%E8%AF%81-psp-%E7%9A%84%E5%AE%89%E5%85%A8%E9%99%90%E5%88%B6)

接下分别在上文授权过的 `privileged-ns` 和 `restricted-ns` 命名空间进行测试，验证 psp 对 pod 请求的限制。

首先尝试在 `restricted-ns` 命名空间通过 deployment 创建一个需要使用 `hostNetwork` 的 pod：

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span><span>14
</span><span>15
</span><span>16
</span><span>17
</span><span>18
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>apps/v1</span><span>
</span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>Deployment</span><span>
</span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
</span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>nginx-hostnetwork</span><span>
</span></span></span><span><span><span></span><span>spec</span><span>:</span><span>
</span></span></span><span><span><span>  </span><span>selector</span><span>:</span><span>
</span></span></span><span><span><span>    </span><span>matchLabels</span><span>:</span><span>
</span></span></span><span><span><span>      </span><span>run</span><span>:</span><span> </span><span>nginx</span><span>
</span></span></span><span><span><span>  </span><span>template</span><span>:</span><span>
</span></span></span><span><span><span>    </span><span>metadata</span><span>:</span><span>
</span></span></span><span><span><span>      </span><span>labels</span><span>:</span><span>
</span></span></span><span><span><span>        </span><span>run</span><span>:</span><span> </span><span>nginx</span><span>
</span></span></span><span><span><span>    </span><span>spec</span><span>:</span><span>
</span></span></span><span><span><span>      </span><span>hostNetwork</span><span>:</span><span> </span><span>true</span><span>
</span></span></span><span><span><span>      </span><span>containers</span><span>:</span><span>
</span></span></span><span><span><span>      </span>- <span>image</span><span>:</span><span> </span><span>nginx</span><span>
</span></span></span><span><span><span>        </span><span>imagePullPolicy</span><span>:</span><span> </span><span>Always</span><span>
</span></span></span><span><span><span>        </span><span>name</span><span>:</span><span> </span><span>nginx-privileged</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

创建并查看结果：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ kubectl create -f hostnetwork-pod.yaml -n restricted-ns
</span></span><span><span>deployment.apps/nginx-hostnetwork created
</span></span><span><span>$ kubectl get deploy -n restricted-ns nginx-hostnetwork
</span></span><span><span>NAME                READY   UP-TO-DATE   AVAILABLE   AGE
</span></span><span><span>nginx-hostnetwork   0/1     <span>1</span>            <span>0</span>           21s
</span></span><span><span>$ kubectl -n restricted-ns get event <span>|</span> grep <span>"pod security policy"</span>
</span></span><span><span>103s        Warning   FailedCreate             deployment/nginx-hostnetwork                                Error creating: pods <span>"nginx-hostnetwork-"</span> is forbidden: unable to validate against any pod security policy: <span>[</span>spec.securityContext.hostNetwork: Invalid value: true: Host network is not allowed to be used<span>]</span>
</span></span></code></pre></td></tr></tbody></table>

由于授权给该命名空间 service account 的安全策略资源禁止 pod 使用 hostNetwork，因此该 deployment 创建 pod 的请求被拒绝。

接着在 `privileged-ns` 命名空间执行相同的操作：

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ kubectl create -f deploy.yaml -n privileged-ns
</span></span><span><span>deployment.apps/nginx-hostnetwork created
</span></span><span><span>$ kubectl get deploy -n privileged-ns nginx-hostnetwork
</span></span><span><span>NAME                READY   UP-TO-DATE   AVAILABLE   AGE
</span></span><span><span>nginx-hostnetwork   1/1     <span>1</span>            <span>1</span>           34s
</span></span><span><span>$ kubectl get po -n privileged-ns
</span></span><span><span>NAME                                 READY   STATUS   RESTARTS   AGE
</span></span><span><span>nginx-hostnetwork-644cdd6598-twds9   0/1     Error    <span>3</span>          77s
</span></span><span><span>$ kubectl get pod nginx-hostnetwork-644cdd6598-twds9 -o <span>jsonpath</span><span>=</span><span>'{.metadata.annotations}'</span> -n privileged-ns
</span></span><span><span>map<span>[</span>kubernetes.io/psp:privileged<span>]</span>
</span></span></code></pre></td></tr></tbody></table>

授权给该命名空间 service account 的安全策略资源允许 pod 使用 hostNetwork，因此 pod 成功被创建。我们可以通过 pod 的 `metadata.annotations` 字段检查其适用的安全策略资源。

## [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#pod-security-admission)[Pod Security Admission](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:pod-security-admission)

从 Kubernetes v1.21开始，Pod Security Policy 将被弃用，并将在 v1.25 中删除，Kubernetes 在 1.22 版本引入了 Pod Security Admission 作为其替代者。

### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9B%BF%E6%8D%A2-psp)[为什么要替换 psp](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9B%BF%E6%8D%A2-psp)

[KEP-2579](https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/2579-psp-replacement) 详细阐述了引入 Pod Security Admission 替代 Pod Security Policy 的三点主要理由：

1.  将策略与用户或 service account 绑定的模型削弱了安全性。
2.  功能无法流畅切换，在没有安全策略资源的情况下无法关闭检查。
3.  API 不一致且缺乏灵活性。

新的 Pod Security Admission 机制在易用性和灵活性上都有了很大提升，从使用角度有以下四点显著不同：

1.  可以在集群中默认开启，只要不设置约束条件就不会触发对 pod 的校验。
2.  只在命名空间级别生效，可以为不同命名空间通过添加标签的方式设置不同的安全限制。
3.  可以为特定的用户、命名空间或者运行时设置豁免规则。
4.  根据实践预设了三种安全等级，不需要由用户单独去设置每一项安全条件。

### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F)[工作方式](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F)

Pod Security Admission 将原来 Pod Security Policy 的安全条件划分成三种预设的安全等级：

-   `privileged`: 不受限，向 pod 提供所有可用的权限。
-   `baseline`：最低限度的限制策略，防止已知的特权升级。
-   `restricted`：严格限制策略，遵循当前 Pod 加固的最佳实践。

三种等级从宽松到严格递增，各自包含了[不同限度的安全条件](https://kubernetes.io/docs/concepts/security/pod-security-standards/#profile-details)，适用于不同的 pod 工作场景。此外还可以将安全等级设置为固定的 Kubernetes 版本，这样即使集群升级到了新的版本且新版本的安全等级定义发生变化，依然可以按旧版本的安全条件对 pod 进行检验。

当 pod 与安全等级冲突时，我们可通过三种模式来选择不同的处理方式：

-   `enforce`：只允许符合安全等级要求的 pod，拒绝与安全等级冲突的 pod。
-   `audit`：只将安全等级冲突记录在集群 event 中，不会拒绝 pod。
-   `warn`：与安全等级冲突时会向用户返回一个警告信息，但不会拒绝 pod。

audit 和 warn 模式是独立的，如果同时需要两者的功能必须分别设置两种模式。

应用安全策略不再需要创建单独的集群资源，只需在启用 Pod Security Admission 后为命名空间设置如下控制标签：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>pod-security.kubernetes.io/&lt;mode&gt;</span><span>:</span><span> </span><span>&lt;level&gt;</span><span>
</span></span></span><span><span><span></span><span>pod-security.kubernetes.io/&lt;mode&gt;-version</span><span>:</span><span> </span><span>&lt;version&gt;</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%9C%A8%E6%97%A7%E7%89%88%E6%9C%AC%E9%9B%86%E7%BE%A4%E4%B8%AD%E5%90%AF%E7%94%A8-psa)[在旧版本集群中启用 psa](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E5%9C%A8%E6%97%A7%E7%89%88%E6%9C%AC%E9%9B%86%E7%BE%A4%E4%B8%AD%E5%90%AF%E7%94%A8-psa)

虽然 **Pod Security Admission** 是一个在 Kubernetes v1.22 引入的功能，但旧版本可以通过安装 PodSecurity admission webhook 来启用该功能，具体步骤如下：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>git clone https://github.com/kubernetes/pod-security-admission.git</span><span>
</span></span></span><span><span><span></span><span>cd pod-security-admission/webhook</span><span>
</span></span></span><span><span><span></span><span>make certs</span><span>
</span></span></span><span><span><span></span><span>kubectl apply -k .</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

以上来自官方文档的步骤在 v1.18.17 集群中执行时会有两个兼容性问题，具体问题和解决方案如下：

1.  kubectl 内置的 kustomize 版本不支持 "replacements" 字段:
    
    ```
    $ kubectl apply -k .
    error: json: unknown field "replacements"
    ```
    
    解决方案：安装最新版本的 kusomize 然后在同一目录执行
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>$ kustomize build . | kubectl apply -f -</span><span>
    </span></span></span></code></pre></td></tr></tbody></table>
    
2.  `manifest/50-deployment.yaml` 文件中定义的 `Deployment.spec.template.spec.containers[0].securityContext` 字段在 v1.19 版本才开始引入，因此 v1.18 需要将该字段修改为对应的 annotation 版本，详见 [Seccomp](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#seccomp)：
    
    ```
    error: error validating "STDIN": error validating data: ValidationError(Deployment.spec.template.spec.containers[0].securityContext): unknown field "seccompProfile" in io.k8s.api.core.v1.SecurityContext; if you choose to ignore these errors, turn validation off with --validate=false
    ```
    

### [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E9%AA%8C%E8%AF%81-psa-%E7%9A%84%E5%AE%89%E5%85%A8%E9%99%90%E5%88%B6)[验证 psa 的安全限制](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E9%AA%8C%E8%AF%81-psa-%E7%9A%84%E5%AE%89%E5%85%A8%E9%99%90%E5%88%B6)

首先创建一个新的命名空间 `psa-test` 用于测试，并将其定义强制应用 `baseline` 安全等级，并对 `restricted` 等级进行警告和审计：

<table><tbody><tr><td><pre tabindex="0"><code><span> 1
</span><span> 2
</span><span> 3
</span><span> 4
</span><span> 5
</span><span> 6
</span><span> 7
</span><span> 8
</span><span> 9
</span><span>10
</span><span>11
</span><span>12
</span><span>13
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>apiVersion</span><span>:</span><span> </span><span>v1</span><span>
</span></span></span><span><span><span></span><span>kind</span><span>:</span><span> </span><span>Namespace</span><span>
</span></span></span><span><span><span></span><span>metadata</span><span>:</span><span>
</span></span></span><span><span><span>  </span><span>name</span><span>:</span><span> </span><span>psa-test</span><span>
</span></span></span><span><span><span>  </span><span>labels</span><span>:</span><span>
</span></span></span><span><span><span>    </span><span>pod-security.kubernetes.io/enforce</span><span>:</span><span> </span><span>baseline</span><span>
</span></span></span><span><span><span>    </span><span>pod-security.kubernetes.io/enforce-version</span><span>:</span><span> </span><span>v1.18</span><span>
</span></span></span><span><span><span>
</span></span></span><span><span><span>    </span><span># We are setting these to our _desired_ `enforce` level.</span><span>
</span></span></span><span><span><span>    </span><span>pod-security.kubernetes.io/audit</span><span>:</span><span> </span><span>restricted</span><span>
</span></span></span><span><span><span>    </span><span>pod-security.kubernetes.io/audit-version</span><span>:</span><span> </span><span>v1.18</span><span>
</span></span></span><span><span><span>    </span><span>pod-security.kubernetes.io/warn</span><span>:</span><span> </span><span>restricted</span><span>
</span></span></span><span><span><span>    </span><span>pod-security.kubernetes.io/warn-version</span><span>:</span><span> </span><span>v1.18</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

接着在该命名空间中创建上文示例中用过的 deployment：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="yaml"><span><span><span>$ kubectl create -f hostnetwork-pod.yaml -n psa-test</span><span>
</span></span></span><span><span><span></span><span>deployment.apps/nginx-hostnetwork created</span><span>
</span></span></span><span><span><span></span><span>$ kubectl get deploy -n psa-test nginx-hostnetwork</span><span>
</span></span></span><span><span><span></span><span>NAME                READY   UP-TO-DATE   AVAILABLE   AGE</span><span>
</span></span></span><span><span><span></span><span>nginx-hostnetwork   0/1     0            0           17s</span><span>
</span></span></span><span><span><span></span><span>$ kubectl -n psa-test get event | grep PodSecurity</span><span>
</span></span></span><span><span><span></span><span>104s        Warning   FailedCreate        replicaset/nginx-hostnetwork-644cdd6598   Error creating</span><span>:</span><span> </span><span>admission webhook "pod-security-webhook.kubernetes.io" denied the request: pods "nginx-hostnetwork-644cdd6598-7rb5m" is forbidden: violates PodSecurity "baseline:v1.23": host namespaces (hostNetwork=true)</span><span>
</span></span></span></code></pre></td></tr></tbody></table>

与 psp 的示例相比，psa 实现了基本一致的安全检查结果，但易用程度有了很大提升。

## [](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/enable-pod-security-policy-for-cluster/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [KEP-2579: Pod Security Admission Control](https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/2579-psp-replacement)
-   [Pod Security Admission | Kubernetes](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
-   [Pod Security Standards | Kubernetes](https://kubernetes.io/docs/concepts/security/pod-security-standards/#profile-details)
-   [Kubernetes: Assigning Pod Security Policies with RBAC](https://medium.com/coryodaniel/kubernetes-assigning-pod-security-policies-with-rbac-2ad2e847c754)
-   [An illustrated deepdive into Pod Security Policies · Banzai Cloud](https://banzaicloud.com/blog/pod-security-policy/)

updatedupdated2022-07-172022-07-17

[安全](https://waynerv.com/tags/%E5%AE%89%E5%85%A8/) [[[云原生]]](https://waynerv.com/tags/%E4%BA%91%E5%8E%9F%E7%94%9F/)