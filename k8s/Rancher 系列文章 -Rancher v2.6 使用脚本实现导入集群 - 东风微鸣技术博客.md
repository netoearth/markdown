## 概述[](https://ewhisper.cn/posts/20406/#%E6%A6%82%E8%BF%B0%20-3)

最近在玩 Rancher, 先从最基本的功能玩起, 目前有几个已经搭建好的 K8S 集群, 需要批量导入, 发现 [官网已经有批量导入的文档](https://docs.rancher.cn/docs/rancher2.5/api/api-import-cluster/_index) 了. 根据 Rancher v2.6 进行验证微调后总结经验.

## 1\. Rancher UI 获取创建集群参数[](https://ewhisper.cn/posts/20406/#1-Rancher-UI-%20%E8%8E%B7%E5%8F%96%E5%88%9B%E5%BB%BA%E9%9B%86%E7%BE%A4%E5%8F%82%E6%95%B0)

1.  访问`Rancher_URL/v3/clusters/`，单击右上角“Create”，创建导入集群：
    
    [![Rancher API 创建导入集群](https://pic-cdn.ewhisper.cn/img/2022/05/13/c2e3a1b38ba5b2f981f2b8b245f0f8fb-image-20191213210405727-242ecb8ec52a4a587cfbd10c375e4c66.png)](https://pic-cdn.ewhisper.cn/img/2022/05/13/c2e3a1b38ba5b2f981f2b8b245f0f8fb-image-20191213210405727-242ecb8ec52a4a587cfbd10c375e4c66.png "Rancher API 创建导入集群")
    
2.  在参数填写页面中，修改以下参数:
    
    -   `dockerRootDir` 默认为`/var/lib/docker`, 如果 dockerroot 路径有修改，需要修改此配置路径；
    -   `enableClusterAlerting`(可选) 根据需要选择是否默认开启集群告警；
    -   `enableClusterMonitoring`(可选) 根据需要选择是否默认开启集群监控；
    -   `name`(必填) 设置集群名称，名称具有唯一性，不能与现有集群名称相同；
3.  配置好参数后单击`Show Request`；
    
4.  在弹出的窗口中，复制 `API Request` 中`HTTP Request:`的 `{}` 中的内容，此内容即为创建的集群的 API 参数；
    

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><p>api_url=<span>'https://rancher-demo.example.com'</span><br>api_token=<span>'token-dbkgj:7pqf5rrjmlxxxxxxxxxxxxxxxxxxxxxxxtrnfljwtxh'</span><br>cluster_name=<span>$1</span></p><p><span><span>create_cluster_data</span></span>()<br>{<br>  cat &lt;&lt;<span>EOF</span><br><span>{</span><br><span> "agentEnvVars": [],</span><br><span> "aksConfig": null,</span><br><span> "aliyunEngineConfig": null,</span><br><span> "amazonElasticContainerServiceConfig": null,</span><br><span> "answers": null,</span><br><span> "azureKubernetesServiceConfig": null,</span><br><span> "clusterTemplateRevisionId": "",</span><br><span> "defaultClusterRoleForProjectMembers": "",</span><br><span> "defaultPodSecurityPolicyTemplateId": "",</span><br><span> "dockerRootDir": "/var/lib/docker",</span><br><span> "eksConfig": null,</span><br><span> "enableClusterAlerting": false,</span><br><span> "enableClusterMonitoring": false,</span><br><span> "gkeConfig": null,</span><br><span> "googleKubernetesEngineConfig": null,</span><br><span> "huaweiEngineConfig": null,</span><br><span> "k3sConfig": null,</span><br><span> "localClusterAuthEndpoint": null,</span><br><span> "name": "$cluster_name",</span><br><span> "rancherKubernetesEngineConfig": null,</span><br><span> "rke2Config": null,</span><br><span> "scheduledClusterScan": null,</span><br><span> "windowsPreferedCluster": false</span><br><span>}</span><br><span>EOF</span><br>}</p><p>curl -k -X POST \<br>    -H <span>"Authorization: Bearer <span>${api_token}</span>"</span> \<br>    -H <span>"Content-Type: application/json"</span> \<br>    -d <span>"<span>$(create_cluster_data)</span>"</span> <span>$api_url</span>/v3/clusters</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

## 2\. 创建集群[](https://ewhisper.cn/posts/20406/#2-%20%E5%88%9B%E5%BB%BA%E9%9B%86%E7%BE%A4)

1.  保存以上代码为脚本文件，最后执行脚本。
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>./rancher_import_cluster.sh &lt;your-cluster-name&gt;<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
2.  脚本执行完成后，集群状态如下所示，其状态为`Provisioning;`
    
    [![导入后状态](https://pic-cdn.ewhisper.cn/img/2022/05/13/e6a0a6a2cb72fe88af04011602557d92-image-20191213212549253-a531848dec2211a8315c2ba370b66f27.png)](https://pic-cdn.ewhisper.cn/img/2022/05/13/e6a0a6a2cb72fe88af04011602557d92-image-20191213212549253-a531848dec2211a8315c2ba370b66f27.png "导入后状态")
    

## ~3\. 创建注册命令~[](https://ewhisper.cn/posts/20406/#3-%20%E5%88%9B%E5%BB%BA%E6%B3%A8%E5%86%8C%E5%91%BD%E4%BB%A4)

> 这一步可能不需要, 创建集群时就会自动生成 clusterregistrationtokens
> 
> 这里又生成了一遍, 会导致有多条 clusterregistrationtokens

## 4\. 获取主机注册命令[](https://ewhisper.cn/posts/20406/#4-%20%E8%8E%B7%E5%8F%96%E4%B8%BB%E6%9C%BA%E6%B3%A8%E5%86%8C%E5%91%BD%E4%BB%A4)

复制并保存以下内容为脚本文件，修改前三行`api_url`、`token`、`cluster_name`，然后执行脚本。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><p>api_url=<span>'https://rancher-demo.example.com'</span><br>api_token=<span>'token-dbkgj:7pqf5rrjmlbgtssssssssssssssssssssssssssssnfljwtxh'</span><br>cluster_name=<span>$1</span></p><p>cluster_ID=$(curl -s -k -H <span>"Authorization: Bearer <span>${api_token}</span>"</span> <span>$api_url</span>/v3/clusters | jq -r <span>".data[] | select(.name == \"<span>$cluster_name</span>\") | .id"</span> )</p><p><span># nodeCommand</span><br><span>#curl -s -k -H "Authorization: Bearer ${api_token}" $api_url/v3/clusters/${cluster_ID}/clusterregistrationtokens | jq -r .data[].nodeCommand</span></p><p><span># command</span><br><span>#curl -s -k -H "Authorization: Bearer ${api_token}" $api_url/v3/clusters/${cluster_ID}/clusterregistrationtokens | jq -r .data[].command</span></p><p><span># insecureCommand</span><br>curl -s -k -H <span>"Authorization: Bearer <span>${api_token}</span>"</span> <span>$api_url</span>/v3/clusters/<span>${cluster_ID}</span>/clusterregistrationtokens | jq -r .data[].insecureCommand</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

> 📝**Notes:**
> 
> 这里看需要, 有 3 种命令:
> 
> 1.  `nodeCommand`: 直接通过 docker 来执行的;
> 2.  `command`: 通过`kubectl` 来执行的;
> 3.  `insecureCommand`: 私有 CA 证书, 通过 `curl` 结合 `kubectl` 来执行的.
> 
> 这里我使用了第三种

## AllInOne[](https://ewhisper.cn/posts/20406/#AllInOne)

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><p>api_url=<span>'https://rancher-demo.example.com'</span><br>api_token=<span>'token-dbkgj:7pqf5rrjxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxljwtxh'</span><br>cluster_name=<span>$1</span></p><p><span><span>create_cluster_data</span></span>()<br>{<br>  cat &lt;&lt;<span>EOF</span><br><span>{</span><br><span> "agentEnvVars": [],</span><br><span> "aksConfig": null,</span><br><span> "aliyunEngineConfig": null,</span><br><span> "amazonElasticContainerServiceConfig": null,</span><br><span> "answers": null,</span><br><span> "azureKubernetesServiceConfig": null,</span><br><span> "clusterTemplateRevisionId": "",</span><br><span> "defaultClusterRoleForProjectMembers": "",</span><br><span> "defaultPodSecurityPolicyTemplateId": "",</span><br><span> "dockerRootDir": "/var/lib/docker",</span><br><span> "eksConfig": null,</span><br><span> "enableClusterAlerting": false,</span><br><span> "enableClusterMonitoring": false,</span><br><span> "gkeConfig": null,</span><br><span> "googleKubernetesEngineConfig": null,</span><br><span> "huaweiEngineConfig": null,</span><br><span> "k3sConfig": null,</span><br><span> "localClusterAuthEndpoint": null,</span><br><span> "name": "$cluster_name",</span><br><span> "rancherKubernetesEngineConfig": null,</span><br><span> "rke2Config": null,</span><br><span> "scheduledClusterScan": null,</span><br><span> "windowsPreferedCluster": false</span><br><span>}</span><br><span>EOF</span><br>}</p><p>curl -k -X POST \<br>    -H <span>"Authorization: Bearer <span>${api_token}</span>"</span> \<br>    -H <span>"Content-Type: application/json"</span> \<br>    -d <span>"<span>$(create_cluster_data)</span>"</span> <span>$api_url</span>/v3/clusters &gt;/dev/null</p><p><span>if</span> [$? -eq 0]; <span>then</span><br>    cluster_ID=$(curl -s -k -H <span>"Authorization: Bearer <span>${api_token}</span>"</span> <span>$api_url</span>/v3/clusters | jq -r <span>".data[] | select(.name == \"<span>$cluster_name</span>\") | .id"</span> )<br>    <span># insecureCommand</span><br>    curl -s -k -H <span>"Authorization: Bearer <span>${api_token}</span>"</span> <span>$api_url</span>/v3/clusters/<span>${cluster_ID}</span>/clusterregistrationtokens | jq -r .data[].insecureCommand<br>    <span>echo</span> <span>"Please execute the above command in the imported cluster to complete the process."</span><br><span>else</span><br>    <span>echo</span> <span>"Import cluster in rancher failed"</span><br><span>fi</span></p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>./rancher_import_cluster.sh &lt;your-cluster-name&gt;<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

执行后会输出一条命令, 在被导入集群上执行如下命令:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br></pre></td><td><pre><code><span>#</span><span> curl --insecure -sfL https://rancher-demo.example.com/v3/import/lzxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxqm6v4lp576c6mg_c-vwv5l.yaml | kubectl apply -f -</span><br>clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver created<br>clusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master created<br>namespace/cattle-system created<br>serviceaccount/cattle created<br>clusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding created<br>secret/cattle-credentials-ec53bfa created<br>clusterrole.rbac.authorization.k8s.io/cattle-admin created<br>deployment.apps/cattle-cluster-agent created<br>service/cattle-cluster-agent created<br></code><p><i></i>SHELL</p></pre></td></tr></tbody></table>

即可导入成功.

🎉🎉🎉

> 📝**TODO:**
> 
> 后面再把登录到对应集群的 master 机器, 并执行命令纳入脚本.

## 系列文章[](https://ewhisper.cn/posts/20406/#%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%20-4)

-   [Rancher 系列文章](https://ewhisper.cn/tags/Rancher/)

## 📚️参考文档[](https://ewhisper.cn/posts/20406/#%F0%9F%93%9A%EF%B8%8F%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%20-3)

-   [使用脚本创建导入集群 | Rancher 文档](https://docs.rancher.cn/docs/rancher2.5/api/api-import-cluster/_index)