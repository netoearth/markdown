## 前言[](https://ewhisper.cn/posts/14449/#%E5%89%8D%E8%A8%80)

本文的目的是在 K8s 集群内搭建 Tailscale 的 DERP 服务器。

## 背景知识[](https://ewhisper.cn/posts/14449/#%E8%83%8C%E6%99%AF%E7%9F%A5%E8%AF%86)

### Tailscale[](https://ewhisper.cn/posts/14449/#Tailscale)

Tailscale 允许您轻松管理对私有资源的访问（本质上是个 VPN 工具），快速 SSH 进入网络上的设备，并且可以在世界上的任何地方安全地工作。

在您的设备、虚拟机和服务器之间创建一个安全的 WireGuard 网状网络 – 即使它们被防火墙或子网隔开。

### DERP[](https://ewhisper.cn/posts/14449/#DERP)

Tailscale 运行 DERP 中继服务器来帮助连接您的节点。除了使用 tailscale 提供的 DERP 服务器之外，您还可以运行自己的服务器。

Tailscale 运行分布在世界各地的 DERP 中继服务器，将您的 Tailscale 节点点对点作为 NAT 遍历期间的一个边通道，并作为 NAT 遍历失败和无法建立直接连接的备用。

Tailscale 在许多地方运行 DERP 服务器。截至 2022 年 9 月，这份名单包括：

-   Australia (Sydney)
-   Brazil (São Paulo)
-   Canada (Toronto)
-   Dubai (Dubai)
-   France (Paris)
-   Germany (Frankfurt)
-   Hong Kong (Hong Kong)
-   India (Bangalore)
-   Japan (Tokyo)
-   Netherlands (Amsterdam)
-   Poland (Warsaw)
-   Singapore (Singapore)
-   South Africa (Johannesburg)
-   Spain (Madrid)
-   United Kingdom (London)
-   United States (Chicago, Dallas, Denver, Honolulu, Los Angeles, Miami, New York City, San Francisco, and Seattle)

> 📝**Notes**:
> 
> 不包括中国。

Tailscale 客户端自动选择最近的低延迟中继。为了提供低延迟连接，Tailscale 正在根据需要不断扩展和增加更多的 DERP 服务器。

为了实现 **低延迟** 和**稳定性**, 因此需要搭建 DERP 服务器。

## 步骤[](https://ewhisper.cn/posts/14449/#%E6%AD%A5%E9%AA%A4)

根据最后参考文档中的任选一份最简的 docker-compose 配置，转换为 K8s 的配置（可以使用工具：[`kompose`](https://ewhisper.cn/posts/35291/) 转换）, 转换后的配置如下：

> 📝**Notes:**
> 
> 为了方便以 Env 方式配置域名，这里使用了 StatefulSets.

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br><span>87</span><br><span>88</span><br><span>89</span><br></pre></td><td><pre><code><span>---</span><br><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>Service</span><br><span>metadata:</span><br>  <span>name:</span> <span>derper-tok</span><br>  <span>labels:</span><br>    <span>io.kompose.service:</span> <span>derper-tok</span><br><span>spec:</span><br>  <span>ports:</span><br>  <span>-</span> <span>port:</span> <span>443</span><br>    <span>name:</span> <span>https</span><br>  <span>-</span> <span>port:</span> <span>80</span><br>    <span>name:</span> <span>http</span><br>  <span>-</span> <span>protocol:</span> <span>UDP</span><br>    <span>port:</span> <span>3478</span><br>    <span>name:</span> <span>stun</span><br>  <span>clusterIP:</span> <span>None</span><br>  <span>selector:</span><br>    <span>io.kompose.service:</span> <span>derper-tok</span><br><span>---</span><br><span>apiVersion:</span> <span>apps/v1</span><br><span>kind:</span> <span>StatefulSet</span><br><span>metadata:</span><br>  <span>annotations:</span><br>    <span>kompose.cmd:</span> <span>kompose</span> <span>convert</span> <span>-f</span> <span>docker-compose.yml</span><br>    <span>kompose.version:</span> <span>1.26</span><span>.1</span> <span>(a9d05d509)</span><br>  <span>labels:</span><br>    <span>io.kompose.service:</span> <span>derper-tok</span><br>  <span>name:</span> <span>derper-tok</span><br>  <span>namespace:</span> <span>tailscale</span><br><span>spec:</span><br>  <span>replicas:</span> <span>3</span><br>  <span>selector:</span><br>    <span>matchLabels:</span><br>      <span>io.kompose.service:</span> <span>derper-tok</span><br>  <span>serviceName:</span> <span>derper-tok</span><br>  <span>template:</span><br>    <span>metadata:</span><br>      <span>annotations:</span><br>        <span>kompose.cmd:</span> <span>kompose</span> <span>convert</span> <span>-f</span> <span>docker-compose.yml</span><br>        <span>kompose.version:</span> <span>1.26</span><span>.1</span> <span>(a9d05d509)</span><br>      <span>labels:</span><br>        <span>io.kompose.service:</span> <span>derper-tok</span><br>    <span>spec:</span><br>      <span>containers:</span><br>      <span>-</span> <span>env:</span><br>        <span>-</span> <span>name:</span> <span>MY_POD_NAME</span><br>          <span>valueFrom:</span><br>            <span>fieldRef:</span><br>              <span>apiVersion:</span> <span>v1</span><br>              <span>fieldPath:</span> <span>metadata.name</span><br>        <span>-</span> <span>name:</span> <span>DOMAIN</span><br>          <span>value:</span> <span>example.com</span><br>        <span>-</span> <span>name:</span> <span>DERP_ADDR</span><br>          <span>value:</span> <span>:443</span><br>        <span>-</span> <span>name:</span> <span>DERP_CERT_DIR</span><br>          <span>value:</span> <span>/app/certs</span><br>        <span>-</span> <span>name:</span> <span>DERP_CERT_MODE</span><br>          <span>value:</span> <span>letsencrypt</span><br>        <span>-</span> <span>name:</span> <span>DERP_DOMAIN</span><br>          <span>value:</span> <span>$(MY_POD_NAME).$(DOMAIN)</span><br>        <span>-</span> <span>name:</span> <span>DERP_HTTP_PORT</span><br>          <span>value:</span> <span>"80"</span><br>        <span>-</span> <span>name:</span> <span>DERP_STUN</span><br>          <span>value:</span> <span>"true"</span><br>        <span>-</span> <span>name:</span> <span>DERP_VERIFY_CLIENTS</span><br>          <span>value:</span> <span>"true"</span><br>        <span>image:</span> <span>fredliang/derper:latest</span><br>        <span>imagePullPolicy:</span> <span>Always</span><br>        <span>name:</span> <span>derper-tok</span><br>        <span>securityContext:</span><br>          <span>capabilities:</span><br>            <span>add:</span><br>            <span>-</span> <span>NET_ADMIN</span><br>          <span>privileged:</span> <span>true</span><br>        <span>volumeMounts:</span><br>        <span>-</span> <span>mountPath:</span> <span>/var/run/tailscale/tailscaled.sock</span><br>          <span>name:</span> <span>tailscale-socket</span><br>      <span>hostNetwork:</span> <span>true</span><br>      <span>volumes:</span><br>      <span>-</span> <span>hostPath:</span><br>          <span>path:</span> <span>/run/tailscale/tailscaled.sock</span><br>          <span>type:</span> <span>Socket</span><br>        <span>name:</span> <span>tailscale-socket</span><br>  <span>updateStrategy:</span><br>    <span>rollingUpdate:</span><br>      <span>partition:</span> <span>0</span><br>    <span>type:</span> <span>RollingUpdate</span></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

具体说明如下：

-   为什么用 StatefulSets, 而不是 Deployment 或 DaemonSet, 主要是我自己的期望如下：
    -   希望可以使用 `derper-tok-{1..3}.example.com` 这样的域名，这样的话，使用 StatefulSets 的话，`derper-tok-1` 就是 POD Name, 方便配置。
    -   如果用 Deployment 或 DaemonSet, Pod name 随机，域名就需要想办法一个一个配置了。
-   这里 K8s 的 Service 纯粹是因为创建 StatefulSets 需要一个 service 而已，实际上并没用到
-   通过 `MY_POD_NAME` `DOMAIN` `DERP_DOMAIN` 就将域名根据 POD name 组合起来了
-   `DERP_CERT_MODE` 现在新版的 DERP 支持 let’s encrypt 自动申请证书，比之前方便很多
-   `DERP_VERIFY_CLIENTS: true` 保证只有自己能使用自己的 DERP 服务器，**需要配合 tailscale 使用**
-   `fredliang/derper:latest` 镜像直接是使用的该镜像
-   `securityContext` 需要确保有 `NET_ADMIN` 的能力，`privileged: true` 也最好加上保证更大的权限。
-   `hostNetwork: true` 直接使用主机网络，也就是：443, 3478 端口直接监听 K8s Node 的端口，简单粗暴。如果端口有冲突需要调整端口，或者不要使用这种模式。
-   `volumeMounts` 和 `volumes`: 这里我安装的 tailscale 在该 K8s Node 上的 socket 为：`/run/tailscale/tailscaled.sock`, 将其挂载到 DERP 容器的 `/var/run/tailscale/tailscaled.sock`, 配合 `DERP_VERIFY_CLIENTS: true`, DERP 服务器就会自动验证客户端，保证安全。

就这样，`kubectl apply` 即可。

🎉🎉🎉

## 总结[](https://ewhisper.cn/posts/14449/#%E6%80%BB%E7%BB%93)

本文比较纯粹，就是说明了一个场景：在 K8s 中安装 DERP 服务器。相关的上下文介绍不多，感兴趣的可以自行了解。

后面有时间可能会出一篇 K8s 中安装 tailscale 的文章。

安装完成后，在 tailscale 控制台上配置 ACL, 加入本次新建的几个 DERP 域名到 `derpMap` 即可。  
最后可以通过：`tailscale netcheck` 进行验证。

## 参考文档[](https://ewhisper.cn/posts/14449/#%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

-   [Custom DERP Servers · Tailscale](https://tailscale.com/kb/1118/custom-derp-servers/)
-   [fredliang/derper - Docker Image | Docker Hub](https://hub.docker.com/r/fredliang/derper)
-   [Tailscale 基础教程：部署私有 DERP 中继服务器 – 云原生实验室](https://icloudnative.io/posts/custom-derp-servers/)
-   [headscale 保底设施之 DERP 中继服务器自建 | 俊瑶先森 (junyao.tech)](http://junyao.tech/posts/18297f50.html)
-   [我的服务器系列：tailscale 使用自定义 derper 服务器（docker 部署） - 霖的个人开发笔记 (linshenkx.cn)](https://www.linshenkx.cn/archives/tailscale-derper-docker)
-   [Tailscale on Kubernetes · Tailscale](https://tailscale.com/kb/1185/kubernetes/)