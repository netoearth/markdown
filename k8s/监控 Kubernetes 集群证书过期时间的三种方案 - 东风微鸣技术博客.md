## 前言[](https://ewhisper.cn/posts/44110/#%E5%89%8D%E8%A8%80)

Kubernetes 中大量用到了证书, 比如 ca 证书、以及 kubelet、apiserver、proxy、etcd 等组件，还有 kubeconfig 文件。

如果证书过期，轻则无法登录 Kubernetes 集群，重则整个集群异常。

为了解决证书过期的问题，一般有以下几种方式：

1.  大幅延长证书有效期，短则 10 年，长则 100 年；
2.  证书快过期是自动轮换，如 Rancher 的 K3s，RKE2 就采用这种方式；
3.  增加证书过期的监控，便于提早发现证书过期问题并人工介入

本次主要介绍关于 Kubernetes 集群证书过期的监控，这里提供 3 种监控方案：

1.  使用 [Blackbox Exporter](https://ewhisper.cn/posts/26225/) 通过 Probe 监控 Kubernetes apiserver 证书过期时间；
2.  使用 [kube-prometheus-stack](https://ewhisper.cn/posts/3988/) 通过 apiserver 和 kubelet 组件监控获取相关证书过期时间;
3.  使用 [enix 的 x509-certificate-exporter](https://github.com/enix/x509-certificate-exporter/)监控集群所有 node 的 `/etc/kubernetes/pki` 和 `/var/lib/kubelet` 下的证书以及 kubeconfig 文件

## 方案一: Blackbox Exporter 监控 Kubernetes apiserver 证书过期时间[](https://ewhisper.cn/posts/44110/#%E6%96%B9%E6%A1%88%E4%B8%80%20-Blackbox-Exporter-%20%E7%9B%91%E6%8E%A7%20-Kubernetes-apiserver-%20%E8%AF%81%E4%B9%A6%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4)

Blackbox Exporter 用于探测 HTTPS、HTTP、TCP、DNS、ICMP 和 grpc 等 Endpoint。在你定义 Endpoint 后，Blackbox Exporter 会生成指标，可以使用 Grafana 等工具进行可视化。Blackbox Exporter 最重要的功能之一是测量 Endpoint 的可用性。

当然, Blackbox Exporter 探测 HTTPS 后就可以获取到证书的相关信息, 就是利用这种方式实现对 Kubernetes apiserver 证书过期时间的监控.

### 配置步骤[](https://ewhisper.cn/posts/44110/#%E9%85%8D%E7%BD%AE%E6%AD%A5%E9%AA%A4)

1.  调整 Blackbox Exporter 的配置, 增加 `insecure_tls_verify: true`, 如下:  
    [![调整 Blackbox Exporter 配置](https://pic-cdn.ewhisper.cn/img/2022/08/25/8efb2e4ec9b4185a7b9b3d514fad268b-clip_image002.jpg)](https://pic-cdn.ewhisper.cn/img/2022/08/25/8efb2e4ec9b4185a7b9b3d514fad268b-clip_image002.jpg "调整 Blackbox Exporter 配置")
    
2.  重启 blackbox exporter: `kubectl rollout restart deploy ...`
    
3.  增加对 Kubernetes APIServer 内部端点 [https://kubernetes.default.svc.cluster.local/readyz](https://kubernetes.default.svc.cluster.local/readyz) 的监控.
    
    1.  如果你没有使用 Prometheus Operator, 使用的是原生的 Prometheus, 则需要修改 Prometheus 配置文件的 configmap 或 secret, 添加 scrape config, 示例如下:
        
        [![Prometheus 增加 scrape config](https://pic-cdn.ewhisper.cn/img/2022/08/25/a809d3078ab45890b94e09e94b840f23-20220825165118.png)](https://pic-cdn.ewhisper.cn/img/2022/08/25/a809d3078ab45890b94e09e94b840f23-20220825165118.png "Prometheus 增加 scrape config")
        
    2.  如果在使用 Prometheus Operator, 则可以增加如下 Probe CRD, Prometheus Operator 会自动将其转换并 merge 到 Prometheus 中.
        

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>monitoring.coreos.com/v1</span><br><span>kind:</span> <span>Probe</span><br><span>metadata:</span><br>  <span>name:</span> <span>kubernetes-apiserver</span><br><span>spec:</span><br>  <span>interval:</span> <span>60s</span><br>  <span>module:</span> <span>http_2xx</span><br>  <span>prober:</span><br>    <span>path:</span> <span>/probe</span><br>    <span>url:</span> <span>monitor-prometheus-blackbox-exporter.default.svc.cluster.local:9115</span><br>  <span>targets:</span><br>    <span>staticConfig:</span><br>      <span>static:</span><br>      <span>-</span> <span>https://kubernetes.default.svc.cluster.local/readyz</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

最后, 可以增加 Prometheus 告警 Rule, 这里就直接用 Prometheus Operator 创建 PrometheusRule CRD 做示例了, 示例如下:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>monitoring.coreos.com/v1</span><br><span>kind:</span> <span>PrometheusRule</span><br><span>metadata:</span><br>  <span>name:</span> <span>prometheus-blackbox-exporter</span><br><span>spec:</span><br>  <span>groups:</span><br>  <span>-</span> <span>name:</span> <span>prometheus-blackbox-exporter</span><br>    <span>rules:</span><br>    <span>-</span> <span>alert:</span> <span>BlackboxSslCertificateWillExpireSoon</span><br>      <span>expr:</span> <span>probe_ssl_earliest_cert_expiry</span> <span>-</span> <span>time()</span> <span>&lt;</span> <span>86400</span> <span>*</span> <span>30</span><br>      <span>for:</span> <span>0m</span><br>      <span>labels:</span><br>        <span>severity:</span> <span>warning</span><br>    <span>-</span> <span>alert:</span> <span>BlackboxSslCertificateWillExpireSoon</span><br>      <span>expr:</span> <span>probe_ssl_earliest_cert_expiry</span> <span>-</span> <span>time()</span> <span>&lt;</span> <span>86400</span> <span>*</span> <span>14</span><br>      <span>for:</span> <span>0m</span><br>      <span>labels:</span><br>        <span>severity:</span> <span>critical</span><br>    <span>-</span> <span>alert:</span> <span>BlackboxSslCertificateExpired</span><br>      <span>annotations:</span><br>        <span>description:</span> <span>|-</span><br><span>          SSL certificate has expired already</span><br><span>            VALUE = {{$value}}</span><br><span>            LABELS = {{$labels}}</span><br><span></span>        <span>summary:</span> <span>SSL</span> <span>certificate</span> <span>expired</span> <span>(instance</span> {{<span>$labels.instance</span> }}<span>)</span><br>      <span>expr:</span> <span>probe_ssl_earliest_cert_expiry</span> <span>-</span> <span>time()</span> <span>&lt;=</span> <span>0</span><br>      <span>for:</span> <span>0m</span><br>      <span>labels:</span><br>        <span>severity:</span> <span>emergency</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

### 效果[](https://ewhisper.cn/posts/44110/#%E6%95%88%E6%9E%9C)

[![Probe 查询证书过期时间](https://pic-cdn.ewhisper.cn/img/2022/08/25/f4a89d9e40b2d02cd92b1a159aac2884-20220825165659.png)](https://pic-cdn.ewhisper.cn/img/2022/08/25/f4a89d9e40b2d02cd92b1a159aac2884-20220825165659.png "Probe 查询证书过期时间")

## 方案二: kube-prometheus-stack 通过 apiserver 和 kubelet 组件监控证书过期时间[](https://ewhisper.cn/posts/44110/#%E6%96%B9%E6%A1%88%E4%BA%8C%20-kube-prometheus-stack-%20%E9%80%9A%E8%BF%87%20-apiserver-%20%E5%92%8C%20-kubelet-%20%E7%BB%84%E4%BB%B6%E7%9B%91%E6%8E%A7%E8%AF%81%E4%B9%A6%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4)

这里可以参考我的文章:[Prometheus Operator 与 kube-prometheus 之二 - 如何监控 1.23+ kubeadm 集群](https://ewhisper.cn/posts/3988/), 安装完成后, 开箱即用.

开箱即用内容包括:

1.  抓取 apiserver 和 kubelet 指标;(即 serviceMonitor)
2.  配置证书过期时间的相关告警; (即 PrometheusRule)

这里用到的指标有:

1.  apiserver
    1.  `apiserver_client_certificate_expiration_seconds_count`
    2.  `apiserver_client_certificate_expiration_seconds_bucket`
2.  kubelet
    1.  `kubelet_certificate_manager_client_expiration_renew_errors`
    2.  `kubelet_server_expiration_renew_errors`
    3.  `kubelet_certificate_manager_client_ttl_seconds`
    4.  `kubelet_certificate_manager_server_ttl_seconds`

### 监控效果[](https://ewhisper.cn/posts/44110/#%E7%9B%91%E6%8E%A7%E6%95%88%E6%9E%9C)

对应的 Prometheus 告警规则如下:

[![证书过期时间相关 PrometheusRule](https://pic-cdn.ewhisper.cn/img/2022/08/25/614d1c5f5e8c34674ab14e58225c4875-20220825172831.png)](https://pic-cdn.ewhisper.cn/img/2022/08/25/614d1c5f5e8c34674ab14e58225c4875-20220825172831.png "证书过期时间相关 PrometheusRule")

## 方案三: 使用 enix 的 x509-certificate-exporter[](https://ewhisper.cn/posts/44110/#%E6%96%B9%E6%A1%88%E4%B8%89%20-%20%E4%BD%BF%E7%94%A8%20-enix-%20%E7%9A%84%20-x509-certificate-exporter)

### 监控手段[](https://ewhisper.cn/posts/44110/#%E7%9B%91%E6%8E%A7%E6%89%8B%E6%AE%B5)

该 Exporter 是通过监控集群所有 node 的指定目录或 path 下的证书文件以及 kubeconfig 文件来获取证书信息.

如果是使用 kubeadm 搭建的 Kubernetes 集群, 则可以监控如下包含证书的文件和 kubeconfig:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br></pre></td><td><pre><code><span>watchFiles:</span><br><span>-</span> <span>/var/lib/kubelet/pki/kubelet-client-current.pem</span><br><span>-</span> <span>/etc/kubernetes/pki/apiserver.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/apiserver-etcd-client.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/apiserver-kubelet-client.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/ca.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/front-proxy-ca.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/front-proxy-client.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/etcd/ca.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/etcd/healthcheck-client.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/etcd/peer.crt</span><br><span>-</span> <span>/etc/kubernetes/pki/etcd/server.crt</span><br><span>watchKubeconfFiles:</span><br><span>-</span> <span>/etc/kubernetes/admin.conf</span><br><span>-</span> <span>/etc/kubernetes/controller-manager.conf</span><br><span>-</span> <span>/etc/kubernetes/scheduler.conf</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

### 安装配置[](https://ewhisper.cn/posts/44110/#%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE)

编辑 values.yaml:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br><span>87</span><br><span>88</span><br><span>89</span><br><span>90</span><br><span>91</span><br><span>92</span><br><span>93</span><br><span>94</span><br><span>95</span><br><span>96</span><br><span>97</span><br><span>98</span><br><span>99</span><br><span>100</span><br><span>101</span><br><span>102</span><br><span>103</span><br><span>104</span><br><span>105</span><br><span>106</span><br><span>107</span><br><span>108</span><br><span>109</span><br><span>110</span><br><span>111</span><br><span>112</span><br><span>113</span><br><span>114</span><br><span>115</span><br><span>116</span><br><span>117</span><br><span>118</span><br><span>119</span><br><span>120</span><br><span>121</span><br><span>122</span><br><span>123</span><br><span>124</span><br><span>125</span><br><span>126</span><br><span>127</span><br><span>128</span><br><span>129</span><br><span>130</span><br><span>131</span><br><span>132</span><br><span>133</span><br><span>134</span><br><span>135</span><br><span>136</span><br><span>137</span><br><span>138</span><br><span>139</span><br><span>140</span><br><span>141</span><br><span>142</span><br><span>143</span><br><span>144</span><br><span>145</span><br><span>146</span><br><span>147</span><br><span>148</span><br><span>149</span><br><span>150</span><br><span>151</span><br><span>152</span><br><span>153</span><br></pre></td><td><pre><code><span>kubeVersion:</span> <span>''</span><br><span>extraLabels:</span> {}<br><span>nameOverride:</span> <span>''</span><br><span>fullnameOverride:</span> <span>''</span><br><span>imagePullSecrets:</span> []<br><span>image:</span><br>  <span>registry:</span> <span>docker.io</span><br>  <span>repository:</span> <span>enix/x509-certificate-exporter</span><br>  <span>tag:</span><br>  <span>pullPolicy:</span> <span>IfNotPresent</span><br><span>psp:</span><br>  <span>create:</span> <span>false</span><br><span>rbac:</span><br>  <span>create:</span> <span>true</span><br>  <span>secretsExporter:</span><br>    <span>serviceAccountName:</span><br>    <span>serviceAccountAnnotations:</span> {}<br>    <span>clusterRoleAnnotations:</span> {}<br>    <span>clusterRoleBindingAnnotations:</span> {}<br>  <span>hostPathsExporter:</span><br>    <span>serviceAccountName:</span><br>    <span>serviceAccountAnnotations:</span> {}<br>    <span>clusterRoleAnnotations:</span> {}<br>    <span>clusterRoleBindingAnnotations:</span> {}<br><span>podExtraLabels:</span> {}<br><span>podAnnotations:</span> {}<br><span>exposePerCertificateErrorMetrics:</span> <span>false</span><br><span>exposeRelativeMetrics:</span> <span>false</span><br><span>metricLabelsFilterList:</span> <span>null</span><br><span>secretsExporter:</span><br>  <span>enabled:</span> <span>true</span><br>  <span>debugMode:</span> <span>false</span><br>  <span>replicas:</span> <span>1</span><br>  <span>restartPolicy:</span> <span>Always</span><br>  <span>strategy:</span> {}<br>  <span>resources:</span><br>    <span>limits:</span><br>      <span>cpu:</span> <span>200m</span><br>      <span>memory:</span> <span>150Mi</span><br>    <span>requests:</span><br>      <span>cpu:</span> <span>20m</span><br>      <span>memory:</span> <span>20Mi</span><br>  <span>nodeSelector:</span> {}<br>  <span>tolerations:</span> []<br>  <span>affinity:</span> {}<br>  <span>podExtraLabels:</span> {}<br>  <span>podAnnotations:</span> {}<br>  <span>podSecurityContext:</span> {}<br>  <span>securityContext:</span><br>    <span>runAsUser:</span> <span>65534</span><br>    <span>runAsGroup:</span> <span>65534</span><br>    <span>readOnlyRootFilesystem:</span> <span>true</span><br>    <span>capabilities:</span><br>      <span>drop:</span><br>        <span>-</span> <span>ALL</span><br>  <span>secretTypes:</span><br>    <span>-</span> <span>type:</span> <span>kubernetes.io/tls</span><br>      <span>key:</span> <span>tls.crt</span><br>  <span>includeNamespaces:</span> []<br>  <span>excludeNamespaces:</span> []<br>  <span>includeLabels:</span> []<br>  <span>excludeLabels:</span> []<br>  <span>cache:</span><br>    <span>enabled:</span> <span>true</span><br>    <span>maxDuration:</span> <span>300</span><br><span>hostPathsExporter:</span><br>  <span>debugMode:</span> <span>false</span><br>  <span>restartPolicy:</span> <span>Always</span><br>  <span>updateStrategy:</span> {}<br>  <span>resources:</span><br>    <span>limits:</span><br>      <span>cpu:</span> <span>100m</span><br>      <span>memory:</span> <span>40Mi</span><br>    <span>requests:</span><br>      <span>cpu:</span> <span>10m</span><br>      <span>memory:</span> <span>20Mi</span><br>  <span>nodeSelector:</span> {}<br>  <span>tolerations:</span> []<br>  <span>affinity:</span> {}<br>  <span>podExtraLabels:</span> {}<br>  <span>podAnnotations:</span> {}<br>  <span>podSecurityContext:</span> {}<br>  <span>securityContext:</span><br>    <span>runAsUser:</span> <span>0</span><br>    <span>runAsGroup:</span> <span>0</span><br>    <span>readOnlyRootFilesystem:</span> <span>true</span><br>    <span>capabilities:</span><br>      <span>drop:</span><br>        <span>-</span> <span>ALL</span><br>  <span>watchDirectories:</span> []<br>  <span>watchFiles:</span> []<br>  <span>watchKubeconfFiles:</span> []<br>  <span>daemonSets:</span><br>    <span>cp:</span><br>      <span>nodeSelector:</span><br>        <span>node-role.kubernetes.io/master:</span> <span>''</span><br>      <span>tolerations:</span><br>        <span>-</span> <span>effect:</span> <span>NoSchedule</span><br>          <span>key:</span> <span>node-role.kubernetes.io/master</span><br>          <span>operator:</span> <span>Exists</span><br>      <span>watchFiles:</span><br>        <span>-</span> <span>/var/lib/kubelet/pki/kubelet-client-current.pem</span><br>        <span>-</span> <span>/etc/kubernetes/pki/apiserver.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/apiserver-etcd-client.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/apiserver-kubelet-client.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/ca.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/front-proxy-ca.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/front-proxy-client.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/etcd/ca.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/etcd/healthcheck-client.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/etcd/peer.crt</span><br>        <span>-</span> <span>/etc/kubernetes/pki/etcd/server.crt</span><br>      <span>watchKubeconfFiles:</span><br>        <span>-</span> <span>/etc/kubernetes/admin.conf</span><br>        <span>-</span> <span>/etc/kubernetes/controller-manager.conf</span><br>        <span>-</span> <span>/etc/kubernetes/scheduler.conf</span><br>    <span>nodes:</span><br>      <span>watchFiles:</span><br>        <span>-</span> <span>/var/lib/kubelet/pki/kubelet-client-current.pem</span><br>        <span>-</span> <span>/etc/kubernetes/pki/ca.crt</span><br><span>rbacProxy:</span><br>  <span>enabled:</span> <span>false</span><br><span>podListenPort:</span> <span>9793</span><br><span>hostNetwork:</span> <span>false</span><br><span>service:</span><br>  <span>create:</span> <span>true</span><br>  <span>port:</span> <span>9793</span><br>  <span>annotations:</span> {}<br>  <span>extraLabels:</span> {}<br><span>prometheusServiceMonitor:</span><br>  <span>create:</span> <span>true</span><br>  <span>scrapeInterval:</span> <span>60s</span><br>  <span>scrapeTimeout:</span> <span>30s</span><br>  <span>extraLabels:</span> {}<br>  <span>relabelings:</span> {}<br><span>prometheusPodMonitor:</span><br>  <span>create:</span> <span>false</span><br><span>prometheusRules:</span><br>  <span>create:</span> <span>true</span><br>  <span>alertOnReadErrors:</span> <span>true</span><br>  <span>readErrorsSeverity:</span> <span>warning</span><br>  <span>alertOnCertificateErrors:</span> <span>true</span><br>  <span>certificateErrorsSeverity:</span> <span>warning</span><br>  <span>certificateRenewalsSeverity:</span> <span>warning</span><br>  <span>certificateExpirationsSeverity:</span> <span>critical</span><br>  <span>warningDaysLeft:</span> <span>30</span><br>  <span>criticalDaysLeft:</span> <span>14</span><br>  <span>extraLabels:</span> {}<br>  <span>alertExtraLabels:</span> {}<br>  <span>rulePrefix:</span> <span>''</span><br>  <span>disableBuiltinAlertGroup:</span> <span>false</span><br>  <span>extraAlertGroups:</span> []<br><span>extraDeploy:</span> []<br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

通过 Helm Chart 安装:

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><code>helm repo add enix https://charts.enix.io<br>helm install x509-certificate-exporter enix/x509-certificate-exporter<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

通过这个 Helm Chart 也会自动安装:

-   ServiceMonitor
-   PrometheusRule

其监控指标为:

-   `x509_cert_not_after`

### 监控效果[](https://ewhisper.cn/posts/44110/#%E7%9B%91%E6%8E%A7%E6%95%88%E6%9E%9C%20-2)

该 Exporter 还提供了一个比较花哨的 Grafana Dashboard, 如下:

[![x509 Exporter Grafana Dashboard](https://pic-cdn.ewhisper.cn/img/2022/08/25/af4036b5606d35731af09460682b664d-clip_image002.jpg)](https://pic-cdn.ewhisper.cn/img/2022/08/25/af4036b5606d35731af09460682b664d-clip_image002.jpg "x509 Exporter Grafana Dashboard")

Alert Rules 如下:

[![x509 Exporter Prometheus Rule](https://pic-cdn.ewhisper.cn/img/2022/08/25/c243db7d40088c441e05111e3c46d705-20220825174532.png)](https://pic-cdn.ewhisper.cn/img/2022/08/25/c243db7d40088c441e05111e3c46d705-20220825174532.png "x509 Exporter Prometheus Rule")

## 总结[](https://ewhisper.cn/posts/44110/#%E6%80%BB%E7%BB%93)

为了监控 Kubernetes 集群的证书过期时间, 我们提供了 3 种方案, 各有优劣:

1.  使用 [Blackbox Exporter](https://ewhisper.cn/posts/26225/) 通过 Probe 监控 Kubernetes apiserver 证书过期时间；
    1.  优势: 实现简单;
    2.  劣势: 只能监控 https 的证书;
2.  使用 [kube-prometheus-stack](https://ewhisper.cn/posts/3988/) 通过 apiserver 和 kubelet 组件监控获取相关证书过期时间;
    1.  优势: 开箱即用, 安装 kube-prometheus-stack 后无需额外安装其他 exporter
    2.  劣势: 只能监控 apiserver 和 kubelet 的证书;
3.  使用 [enix 的 x509-certificate-exporter](https://github.com/enix/x509-certificate-exporter/)监控集群所有 node 的 `/etc/kubernetes/pki` 和 `/var/lib/kubelet` 下的证书以及 kubeconfig 文件
    1.  优势: 可以监控所有 node, 所有 kubeconfig 文件, 以及 所有 tls 格式的 secret 证书, 如果要监控 Kubernetes 集群以外的证书, 也可以如法炮制; 范围广而全;
    2.  需要额外安装: x509-certificate-exporter, 对应有 1 个 Deployment 和 多个 DaemonSet, 对 Kubernetes 集群的资源消耗不少.

可以根据您的实际情况灵活进行选择.

🎉🎉🎉

## 📚️参考文档[](https://ewhisper.cn/posts/44110/#%F0%9F%93%9A%EF%B8%8F%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

-   [如何使用 Blackbox Exporter 监控 URL? - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/26225/)
-   [Prometheus Operator 与 kube-prometheus 之二 - 如何监控 1.23+ kubeadm 集群 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/3988/)
-   [x509-certificate-exporter/deploy/charts/x509-certificate-exporter at master · enix/x509-certificate-exporter (github.com)](https://github.com/enix/x509-certificate-exporter/tree/master/deploy/charts/x509-certificate-exporter)