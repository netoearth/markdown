## 前言[](https://ewhisper.cn/posts/52241/#%E5%89%8D%E8%A8%80%20-9)

随着容器、芯片技术的进一步发展，以及绿色、节能、信创等方面的要求，多 CPU 架构的场景越来越常见。典型的应用场景包括：

1.  信创：x86 服务器 + 鲲鹏 ARM 等信创服务器；
2.  个人电脑：苹果 Mac M1 + Windows 电脑（或旧的 Intel 芯片苹果电脑）；
3.  Edge：数据中心使用 x86 服务器，边缘 Edge 端使用低功耗的 arm 边缘设备（如树莓派等）。

容器云原生技术在这方面支持的是很好，但是实际使用中细节会有一些问题，举一个例子，就是：**如何保存 / 同步多架构容器 Docker 镜像**

本次先以将 Docker Hub 的镜像同步到本地镜像仓库为例说明。

## 词汇表[](https://ewhisper.cn/posts/52241/#%E8%AF%8D%E6%B1%87%E8%A1%A8%20-2)

| 英文 | 中文 | 说明 |
| --- | --- | --- |
| multi-arch image | 多架构镜像 |  |
| variant | 变体 | 不同变体指的如：redis 镜像的 `arm/v5` 和 `arm/v7` 两种变体 |
| manifest | 清单 |  |
| manifest-list | 清单（的）列表 |  |
| layer | （镜像）层 |  |
| image index | 镜像索引 | OCI 专有名词，含义和 manifest-list 相同 |
| manifest digest | 清单摘要 |  |

## 容器镜像如何支持多架构[](https://ewhisper.cn/posts/52241/#%E5%AE%B9%E5%99%A8%E9%95%9C%E5%83%8F%E5%A6%82%E4%BD%95%E6%94%AF%E6%8C%81%E5%A4%9A%E6%9E%B6%E6%9E%84)

一个多架构镜像（A multi-arch image）是一种容器镜像，它可以组合不同架构体系（如 amd64 和 arm）的变体（variants），有时还可以组合不同操作系统（如 windows 和 linux）的变体。运行支持多架构的镜像时，容器客户端会自动选择与你的 OS 和架构相匹配的镜像变体。

多架构镜像是基于镜像清单和清单列表实现的。

### 清单（Manifests）[](https://ewhisper.cn/posts/52241/#%E6%B8%85%E5%8D%95%EF%BC%88Manifests%EF%BC%89)

每个容器镜像都由一个“清单”表示。清单是一个 JSON 文件，用于唯一标识镜像，并引用其层（layer）及其相应的大小。

`hello-world` **Linux** 镜像的基本清单类似于以下内容：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br></pre></td><td><pre><code>{<br>  <span>"schemaVersion"</span>: <span>2</span>,<br>  <span>"mediaType"</span>: <span>"application/vnd.docker.distribution.manifest.v2+json"</span>,<br>  <span>"config"</span>: {<br>      <span>"mediaType"</span>: <span>"application/vnd.docker.container.image.v1+json"</span>,<br>      <span>"size"</span>: <span>1510</span>,<br>      <span>"digest"</span>: <span>"sha256:fbf289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e"</span><br>    },<br>  <span>"layers"</span>: [<br>      {<br>        <span>"mediaType"</span>: <span>"application/vnd.docker.image.rootfs.diff.tar.gzip"</span>,<br>        <span>"size"</span>: <span>977</span>,<br>        <span>"digest"</span>: <span>"sha256:2c930d010525941c1d56ec53b97bd057a67ae1865eebf042686d2a2d18271ced"</span><br>      }<br>    ]<br>}<br></code><p><i></i>JSON</p></pre></td></tr></tbody></table>

### 清单列表 (Manifest-lists)[](https://ewhisper.cn/posts/52241/#%E6%B8%85%E5%8D%95%E5%88%97%E8%A1%A8%20-Manifest-lists)

多架构镜像的 _清单列表_ （通常称为 OCI 镜像 [的镜像索引](https://github.com/opencontainers/image-spec/blob/master/image-index.md)）是镜像的集合（索引），您可以通过指定一个或多个镜像名称来创建一个。它包括有关每个镜像的详细信息，例如支持的操作系统和体系架构、大小和清单摘要 (manifest digest)。清单列表的使用方式与 `docker pull` 和 `docker run` 命令 中的镜像名称相同。

[docker](https://docs.docker.com/engine/reference/commandline/manifest/) CLI 使用 `docker manifest`命令管理清单和清单列表。

> 🐾 **Warning**:
> 
> 目前，该命令 `docker manifest` 和子命令是实验性的。有关使用实验性命令的详细信息，请参阅 Docker 文档。
> 
> ✍️笔者注：可能是因为 **实验性** 的原因，使用过程中有几个多架构镜像碰到了诡异的问题。

您可以使用该命令 `docker manifest inspect` 查看清单列表。以下是多架构镜像`hello-world:latest` 的输出，它有三个清单：两个用于 Linux 操作系统体系架构，一个用于 Windows 体系架构。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br></pre></td><td><pre><code>{<br>  <span>"schemaVersion"</span>: <span>2</span>,<br>  <span>"mediaType"</span>: <span>"application/vnd.docker.distribution.manifest.list.v2+json"</span>,<br>  <span>"manifests"</span>: [<br>    {<br>      <span>"mediaType"</span>: <span>"application/vnd.docker.distribution.manifest.v2+json"</span>,<br>      <span>"size"</span>: <span>524</span>,<br>      <span>"digest"</span>: <span>"sha256:83c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a"</span>,<br>      <span>"platform"</span>: {<br>        <span>"architecture"</span>: <span>"amd64"</span>,<br>        <span>"os"</span>: <span>"linux"</span><br>      }<br>    },<br>    {<br>      <span>"mediaType"</span>: <span>"application/vnd.docker.distribution.manifest.v2+json"</span>,<br>      <span>"size"</span>: <span>525</span>,<br>      <span>"digest"</span>: <span>"sha256:873612c5503f3f1674f315c67089dee577d8cc6afc18565e0b4183ae355fb343"</span>,<br>      <span>"platform"</span>: {<br>        <span>"architecture"</span>: <span>"arm64"</span>,<br>        <span>"os"</span>: <span>"linux"</span><br>      }<br>    },<br>    {<br>      <span>"mediaType"</span>: <span>"application/vnd.docker.distribution.manifest.v2+json"</span>,<br>      <span>"size"</span>: <span>1124</span>,<br>      <span>"digest"</span>: <span>"sha256:b791ad98d505abb8c9618868fc43c74aa94d08f1d7afe37d19647c0030905cae"</span>,<br>      <span>"platform"</span>: {<br>        <span>"architecture"</span>: <span>"amd64"</span>,<br>        <span>"os"</span>: <span>"windows"</span>,<br>        <span>"os.version"</span>: <span>"10.0.17763.1697"</span><br>      }<br>    }<br>  ]<br>}<br></code><p><i></i>JSON</p></pre></td></tr></tbody></table>

## 使用 `docker manifest` 保存多架构镜像[](https://ewhisper.cn/posts/52241/#%E4%BD%BF%E7%94%A8%20-docker-manifest-%20%E4%BF%9D%E5%AD%98%E5%A4%9A%E6%9E%B6%E6%9E%84%E9%95%9C%E5%83%8F)

这里是将多架构的镜像推送到本地镜像仓库步骤：

1.  标记每个特定于体系结构的镜像并将其推送到容器注册表。以下示例假定有两个 Linux 体系结构：arm64 和 amd64。
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code>docker tag myimage:arm64 \<br>  192.168.2.23:5000/multi-arch-samples/myimage:arm64<p>docker push 192.168.2.23:5000/multi-arch-samples/myimage:arm64</p><p>docker tag myimage:amd64 \<br>  192.168.2.23:5000/multi-arch-samples/myimage:amd64</p><p>docker push 192.168.2.23:5000/multi-arch-samples/myimage:amd64</p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
2.  运行 `docker manifest create` 以创建清单列表，以将前面的镜像合并到多架构镜像中。
    
    <table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><code>docker manifest create 192.168.2.23:5000/multi-arch-samples/myimage:multi \<br> 192.168.2.23:5000/multi-arch-samples/myimage:arm64 \<br> 192.168.2.23:5000/multi-arch-samples/myimage:amd64<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
3.  使用以下命令 `docker manifest push` 将清单推送到镜像仓库：
    
    <table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>docker manifest push 192.168.2.23:5000/multi-arch-samples/myimage:multi<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>
    
4.  使用命令 `docker manifest inspect` 查看清单列表。上一节显示了命令输出的示例。
    

将多架构清单推送到镜像仓库后，使用多架构镜像的方式与处理单架构镜像的方式相同。例如，使用 `docker pull` 拉取镜像。

## 保存 / 同步多架构镜像实用脚本一 - 基于 `docker manifest`[](https://ewhisper.cn/posts/52241/#%E4%BF%9D%E5%AD%98%20-%20%E5%90%8C%E6%AD%A5%E5%A4%9A%E6%9E%B6%E6%9E%84%E9%95%9C%E5%83%8F%E5%AE%9E%E7%94%A8%E8%84%9A%E6%9C%AC%E4%B8%80%20-%20%E5%9F%BA%E4%BA%8E%20-docker-manifest)

### 场景一[](https://ewhisper.cn/posts/52241/#%E5%9C%BA%E6%99%AF%E4%B8%80)

**已有多架构压缩包 需要 load 压缩包并将多架构镜像上传到本地镜像仓库**

以 K3s 为例，官方在 release 时已经发布了多架构的离线镜像压缩包，分别为：

-   `k3s-airgap-images-amd64.tar.gz`
-   `k3s-airgap-images-arm.tar.gz`
-   `k3s-airgap-images-arm64.tar.gz`
-   …

这些包已经下载好，并传到客户 / 用户的离线环境机器上。现在需要 load 压缩包并将多架构镜像上传到本地镜像仓库

### 大致步骤[](https://ewhisper.cn/posts/52241/#%E5%A4%A7%E8%87%B4%E6%AD%A5%E9%AA%A4)

1.  `docker load` 压缩包
2.  其中的镜像逐个打 tag, 改为 `< 本地镜像仓库地址 >/.../...:<tag>-<arch>`
3.  push 镜像
4.  以上步骤重复 3 遍，将 tag 带有 `-amd64` `-arm` `-arm64` 的镜像都 push 到本地镜像仓库
5.  镜像逐个 `docker manifest create` 以创建清单列表
6.  使用以下命令 `docker manifest push` 将清单逐个推送到镜像仓库

完整脚本如下：

> 🐾 **Warning:**
> 
> 由于本人能力有限，在使用 k3s v1.21.7+k3s1 版本的 8\*3 个离线镜像做测试的时候，总是 5 个成功，另外 3 个出现 manifest list 的 arch 和 manifest 对不上的情况。  
> 不知道是我脚本问题还是 `docker manifest` 命令是实验性导致的。  
> 有经验的还请帮忙看看。谢谢~

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br><span>87</span><br><span>88</span><br><span>89</span><br><span>90</span><br><span>91</span><br><span>92</span><br><span>93</span><br><span>94</span><br><span>95</span><br><span>96</span><br><span>97</span><br><span>98</span><br><span>99</span><br><span>100</span><br><span>101</span><br><span>102</span><br><span>103</span><br><span>104</span><br><span>105</span><br><span>106</span><br><span>107</span><br><span>108</span><br><span>109</span><br><span>110</span><br><span>111</span><br><span>112</span><br><span>113</span><br><span>114</span><br><span>115</span><br><span>116</span><br><span>117</span><br><span>118</span><br><span>119</span><br><span>120</span><br><span>121</span><br><span>122</span><br><span>123</span><br><span>124</span><br><span>125</span><br><span>126</span><br><span>127</span><br><span>128</span><br><span>129</span><br><span>130</span><br><span>131</span><br><span>132</span><br><span>133</span><br><span>134</span><br><span>135</span><br><span>136</span><br><span>137</span><br><span>138</span><br><span>139</span><br><span>140</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><br>amd64_images=<span>"k3s-airgap-images-amd64.tar.gz"</span><br>arm64_images=<span>"k3s-airgap-images-arm64.tar.gz"</span><br>arm_images=<span>""</span><br>list=<span>"k3s-images.txt"</span><p><span><span>usage</span></span>() {<br>    <span>echo</span> <span>"USAGE: <span>$0</span> [--amd64-images k3s-airgap-images-amd64.tar.gz] [--arm64-images k3s-airgap-images-arm64.tar.gz] [---arm-images k3s-airgap-images-arm.tar.gz] --registry my.registry.com:5000"</span><br>    <span>echo</span> <span>"  [-l|--image-list path] text file with list of images; one image per line."</span><br>    <span>echo</span> <span>"  [-x|--amd64-images path] amd64 arch tar.gz generated by docker save."</span><br>    <span>echo</span> <span>"  [-a|--arm64-images path] arm64 arch tar.gz generated by docker save."</span><br>    <span>echo</span> <span>"  [---arm-images path] arm arch tar.gz generated by docker save."</span><br>    <span>echo</span> <span>"  [-r|--registry registry:port] target private registry:port."</span><br>    <span>echo</span> <span>"  [-h|--help] Usage message"</span><br>}</p><p><span><span>push_manifest</span></span>() {<br>    <span>export</span> DOCKER_CLI_EXPERIMENTAL=enabled<br>    manifest_list=()<br>    <span>for</span> i_arch <span>in</span> <span>"<span>${arch_list[@]}</span>"</span>; <span>do</span><br>        manifest_list+=(<span>"<span>$1</span>-<span>${i_arch}</span>"</span>)<br>    <span>done</span></p><p>    <span>echo</span> <span>"Preparing manifest <span>$1</span>, list[<span>${arch_list[@]}</span>]"</span><br>    docker manifest create <span>"<span>$1</span>"</span> <span>"<span>${manifest_list[@]}</span>"</span> --insecure<br>    docker manifest push <span>"<span>$1</span>"</span> --purge --insecure<br>}</p><p><span>while</span> [[<span>$#</span> -gt 0 ]]; <span>do</span><br>    key=<span>"<span>$1</span>"</span><br>    <span>case</span> <span>$key</span> <span>in</span><br>    -r | --registry)<br>        reg=<span>"<span>$2</span>"</span><br>        <span>shift</span> <span># past argument</span><br>        <span>shift</span> <span># past value</span><br>        ;;<br>    -l | --image-list)<br>        list=<span>"<span>$2</span>"</span><br>        <span>shift</span> <span># past argument</span><br>        <span>shift</span> <span># past value</span><br>        ;;<br>    -x | --amd64-images)<br>        amd64_images=<span>"<span>$2</span>"</span><br>        <span>shift</span> <span># past argument</span><br>        <span>shift</span> <span># past value</span><br>        ;;<br>    -a | --arm64-images)<br>        arm64_images=<span>"<span>$2</span>"</span><br>        <span>shift</span> <span># past argument</span><br>        <span>shift</span> <span># past value</span><br>        ;;<br>    --arm-images)<br>        arm_images=<span>"<span>$2</span>"</span><br>        <span>shift</span> <span># past argument</span><br>        <span>shift</span> <span># past value</span><br>        ;;<br>    -h | --<span>help</span>)<br>        <span>help</span>=<span>"true"</span><br>        <span>shift</span><br>        ;;<br>    *)<br>        usage<br>        <span>exit</span> 1<br>        ;;<br>    <span>esac</span><br><span>done</span></p><p><span>if</span> [[-z <span>$reg</span> ]]; <span>then</span><br>    usage<br>    <span>exit</span> 1<br><span>fi</span></p><p><span>if</span> [[<span>$help</span> ]]; <span>then</span><br>    usage<br>    <span>exit</span> 0<br><span>fi</span></p><p>arch_list=()<br><span>if</span> [[-n <span>"<span>${amd64_images}</span>"</span> ]]; <span>then</span><br>    arch_list+=(<span>"amd64"</span>)<br><span>fi</span><br><span>if</span> [[-n <span>"<span>${arm64_images}</span>"</span> ]]; <span>then</span><br>    arch_list+=(<span>"arm64"</span>)<br><span>fi</span><br><span>if</span> [[-n <span>"<span>${arm_images}</span>"</span> ]]; <span>then</span><br>    arch_list+=(<span>"arm"</span>)<br><span>fi</span></p><p>image_list=()<br><span>while</span> IFS= <span>read</span> -r i; <span>do</span><br>    [-z <span>"<span>${i}</span>"</span> ] &amp;&amp; <span>continue</span><br>    image_list+=(<span>"<span>${i}</span>"</span>)<br><span>done</span> &lt;<span>"<span>${list}</span>"</span></p><p><span>for</span> arch <span>in</span> <span>"<span>${arch_list[@]}</span>"</span>; <span>do</span><br>    [-z <span>"<span>${arch}</span>"</span> ] &amp;&amp; <span>continue</span></p><p>    <span>case</span> <span>$arch</span> <span>in</span><br>    amd64)<br>        docker load --input <span>${amd64_images}</span><br>        ;;<br>    arm64)<br>        docker load --input <span>${arm64_images}</span><br>        ;;<br>    arm)<br>        docker load --input <span>${arm_images}</span><br>        ;;<br>    <span>esac</span></p><p>    <span>for</span> i <span>in</span> <span>"<span>${image_list[@]}</span>"</span>; <span>do</span><br>        [-z <span>"<span>${i}</span>"</span> ] &amp;&amp; <span>continue</span></p><p>        <span>case</span> <span>$i</span> <span>in</span><br>        */*)<br>            image_name=<span>"<span>${reg}</span>/<span>${i}</span>"</span><br>            ;;<br>        *)<br>            image_name=<span>"<span>${reg}</span>/library/<span>${i}</span>"</span><br>            ;;<br>        <span>esac</span></p><p>        docker tag <span>"<span>${i}</span>"</span> <span>"<span>${image_name}</span>-<span>${arch}</span>"</span><br>        docker rmi -f <span>"<span>${i}</span>"</span><br>        docker push <span>"<span>${image_name}</span>-<span>${arch}</span>"</span><br>    <span>done</span><br><span>done</span></p><p><span>for</span> i <span>in</span> <span>"<span>${image_list[@]}</span>"</span>; <span>do</span><br>    [-z <span>"<span>${i}</span>"</span> ] &amp;&amp; <span>continue</span></p><p>    <span>case</span> <span>$i</span> <span>in</span><br>    */*)<br>        image_name=<span>"<span>${reg}</span>/<span>${i}</span>"</span><br>        ;;<br>    *)<br>        image_name=<span>"<span>${reg}</span>/library/<span>${i}</span>"</span><br>        ;;<br>    <span>esac</span><br>    push_manifest <span>"<span>${image_name}</span>"</span><br><span>done</span></p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

使用方法：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><code>./load-images-multi-arch.sh --registry 192.168.2.23:5000 --arm-images k3s-airgap-images-arm.tar.gz<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

日志输出如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br></pre></td><td><pre><code>$ ./load-images-multi-arch.sh --registry <span>192.168</span>.<span>2.23</span>:<span>5000</span> --arm_images k3s-airgap-images-arm.tar.gz<br># docker load 镜像第一轮，是 amd64 架构的<br><span>67</span>f770da229b: Loading layer [==================================================&gt;]   <span>1.45</span>MB/<span>1.45</span>MB<br>Loaded image: rancher/library-busybox:<span>1.32</span>.<span>1</span><br>...<p># 打 tag 并 <span>delete</span> 原 tag 镜像，并 <span>push</span><br>Untagged: rancher/coredns-coredns:<span>1.8</span>.<span>3</span><br>The <span>push</span> refers to repository [<span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>coredns-coredns]<br><span>85</span>c53e1bd74e: Pushed<br><span>225</span>df95e717c: Pushed<br><span>1.8</span>.<span>3</span>-amd64: digest: sha256:db4f1c57978d7372b50f416d1058beb60cebff9a0d5b8bee02bfe70302e1cb2f <span>size</span>: <span>739</span><br>...</p><p># docker load 镜像第二轮，是 arm64 架构的<br>...<br><span>32626</span>eb1fe89: Loading layer [==================================================&gt;]  <span>526.8</span>kB/<span>526.8</span>kB<br>Loaded image: rancher/pause:<span>3.1</span></p><p>...<br>Untagged: rancher/pause:<span>3.1</span><br>The <span>push</span> refers to repository [<span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>pause]<br><span>32626</span>eb1fe89: Pushed<br><span>3.1</span>-arm64: digest: sha256:<span>2</span>aac966ece8906a535395f92bb25f0e8e21dac737df75b381e8f9bdd3ed56528 <span>size</span>: <span>527</span></p><p># docker load 镜像第二轮，是 arm 架构的<br><span>8</span>e322dc9c333: Loading layer [==================================================&gt;]  <span>5.045</span>MB/<span>5.045</span>MB<br>efed3cfd1b26: Loading layer [==================================================&gt;]  <span>1.623</span>MB/<span>1.623</span>MB<br>a46153382f22: Loading layer [==================================================&gt;]  <span>3.584</span>kB/<span>3.584</span>kB<br>Loaded image: rancher/klipper-lb:v0.<span>3.4</span><br>...</p><p>Untagged: rancher/coredns-coredns:<span>1.8</span>.<span>3</span><br>The <span>push</span> refers to repository [<span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>coredns-coredns]<br><span>9</span>f4a0b0fd8b2: Pushed<br><span>225</span>df95e717c: Layer already exists<br><span>1.8</span>.<span>3</span>-arm: digest: sha256:dfc241eae22da74dd378535b69d7927f897acf48424cdcb90991b33f412cb7ae <span>size</span>: <span>739</span></p><p># docker manifest create<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>coredns-coredns:<span>1.8</span>.<span>3</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>coredns-coredns:<span>1.8</span>.<span>3</span><br>sha256:dc76fece93e42f05e7013e159097a0d426734fd268467f242d5b155dd49b0221<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>klipper-helm:v0.<span>6.6</span>-build20211022, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>klipper-helm:v0.<span>6.6</span>-build20211022<br>sha256:e1c6842554ea37e66443cfab9a2422231bf8390b4c69711a74eb4cccde9d3dba<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>klipper-lb:v0.<span>3.4</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>klipper-lb:v0.<span>3.4</span><br>sha256:<span>98842</span>bae8630a2aab1a94960185e152745ecf16ca69cf1eefdb53848cbc41063<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>library-busybox:<span>1.32</span>.<span>1</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>library-busybox:<span>1.32</span>.<span>1</span><br>sha256:<span>0</span>b93c11bfd89ee5c971deaf9f312d115b2e1d797f79a7f68a266baecfb09a99f<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>library-traefik:<span>2.4</span>.<span>8</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>library-traefik:<span>2.4</span>.<span>8</span><br>sha256:<span>58464</span>dda10504d271a17855541ed8d31a787ea25eb751ecce90e14256f23eb24<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>local-path-provisioner:v0.<span>0.19</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>local-path-provisioner:v0.<span>0.19</span><br>sha256:<span>0</span>c797ef85540a4934ea84a9471f4f5a10c93f749ee668d92527361c61bbe98c3<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/m</span>etrics-server:v0.<span>3.6</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/m</span>etrics-server:v0.<span>3.6</span><br>sha256:<span>742595</span>f61320bcaead987c5aafc3eb64b9a9151edb02b9e4d27f8abcae26d92e<br>Preparing manifest <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>pause:<span>3.1</span>, list[amd64 arm64 arm]<br>Created manifest list <span>192.168</span>.<span>2.23</span>:<span>5000</span><span>/rancher/</span>pause:<span>3.1</span><br>sha256:f3ef3cbaf2ea466a0c2a2cf3db0d9fbc30f4c24e57a79603aa0fa8999d4813b0</p></code><p><i></i>GRADLE</p></pre></td></tr></tbody></table>

## Skopeo 简介[](https://ewhisper.cn/posts/52241/#Skopeo-%20%E7%AE%80%E4%BB%8B%20-2)

[![](https://pic-cdn.ewhisper.cn/img/2021/11/07/267c7fdfcfa44357c7a2bd8f77ecb613-skopeo.svg)](https://pic-cdn.ewhisper.cn/img/2021/11/07/267c7fdfcfa44357c7a2bd8f77ecb613-skopeo.svg)

-   [Skopeo 简介 - K8S 1.20 弃用 Docker 评估之 Docker CLI 的替代产品 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/36509/#Skopeo-%20%E7%AE%80%E4%BB%8B)

最近 Skopeo 版本更新到了 [v1.8](https://quay.io/repository/containers/skopeo/manifest/sha256:79649f77bbbabb486444ee29ee4af5a55f51db1bf12c385c7bf55447cc5be42a), 最近的版本增加了一些与 **多架构** 有关的 flags, 使得通过 `skopeo` 进行多架构镜像的保存 / 同步更为方便。

> 📝 **Notes:**
> 
> 目前关于多架构，只有 3 个选项，3 个选项都没有选择源镜像多个架构的其中几个的能力，但正在开发中。  
> 具体见这个 Issue: [feature: Support list of archs for `sync` command · Issue #1694 · containers/skopeo (github.com)](https://github.com/containers/skopeo/issues/1694)

以下是一些相关 flags:

-   `skopeo`
    -   `--override-arch <arch>`: 使用 `arch` 代替机器的架构来选择镜像。
    -   `--override-os <os>`: 使用 `os` 代替机器的 OS 来选择镜像。
    -   `--override-variant <variant>`: 使用 `variant` 运行的架构的变体来选择镜像。（不同变体指的如：redis 镜像的 `arm/v5` 和 `arm/v7` 两种变体）
-   `skopeo copy`
    -   `--all, -a`: 如果 source-image 引用的是一个镜像列表，那么 **不要** 只复制与当前操作系统和体系架构匹配的镜像（取决于全局的 `--override-os`、`--override-arch` 和`--override-variant`选项的使用），而是尝试复制列表中的所有镜像，以及列表本身。
    -   `--multi-arch`: 如果源镜像引用多架构镜像，则控制要复制的内容。默认设置是`system`。
        -   `system`: 仅复制与系统架构匹配的镜像
        -   `all`: 复制完整的多架构镜像
        -   `index-only`: 仅复制镜像索引 (image index).(`index-only`选项通常会失败，除非目标中已经存在每个架构所引用的镜像，或者目标注册中心支持稀疏索引。)
-   `skopeo sync`
    -   `--all, -a`: 同上

> 📝 **Notes:**
> 
> 根据 `skopeo copy --multi-arch index-only` 的描述，[场景一](https://ewhisper.cn/posts/52241/#%E5%9C%BA%E6%99%AF%E4%B8%80) 还有一种实现就是：
> 
> 1.  `docker manifest` 之前的步骤，维持原状
> 2.  将 `docker manifest create` 和 `docker manifest push` 替换为 `skopeo copy --multi-arch index-only`

## 保存 / 同步多架构镜像实用脚本二 - 基于 `skopeo copy`[](https://ewhisper.cn/posts/52241/#%E4%BF%9D%E5%AD%98%20-%20%E5%90%8C%E6%AD%A5%E5%A4%9A%E6%9E%B6%E6%9E%84%E9%95%9C%E5%83%8F%E5%AE%9E%E7%94%A8%E8%84%9A%E6%9C%AC%E4%BA%8C%20-%20%E5%9F%BA%E4%BA%8E%20-skopeo-copy)

### 场景二[](https://ewhisper.cn/posts/52241/#%E5%9C%BA%E6%99%AF%E4%BA%8C)

**直接从 [docker.io](http://docker.io/) 同步镜像到本地镜像仓库**

以 K3s 某一版本为例，镜像列表为：

-   rancher/coredns-coredns:1.8.3
-   rancher/klipper-helm:v0.6.6-build20211022
-   rancher/klipper-lb:v0.3.4
-   rancher/library-busybox:1.32.1
-   rancher/library-traefik:2.4.8
-   rancher/local-path-provisioner:v0.0.19
-   rancher/metrics-server:v0.3.6
-   rancher/pause:3.1

这里直接基于 [镜像搬运工 skopeo](https://blog.k8s.li/skopeo.html) 提供的脚本做修改，修改后如下：

> 📝 **Notes:**
> 
> 因为较新版本的 `skopeo` 才有上面说的一系列 flags, 我的 Ubuntu apt 安装的 `skopeo` 还停留在 `v1.5` 版本，没有上述功能。所以直接通过 `docker run` 方式运行  
> 除此之外还添加了 `--multi-arch all` 选项。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br></pre></td><td><pre><code><span>#!/bin/bash</span><br>GREEN_COL=<span>"\\033[32;1m"</span><br>RED_COL=<span>"\\033[1;31m"</span><br>NORMAL_COL=<span>"\\033[0;39m"</span><br>SOURCE_REGISTRY=<span>$1</span><br>TARGET_REGISTRY=<span>$2</span><br>IMAGES_LIST_FILE=<span>$3</span><br>: <span>${IMAGES_LIST_FILE:="k3s-images.txt"}</span><br>: <span>${TARGET_REGISTRY:="192.168.2.23:5000"}</span><br>: <span>${SOURCE_REGISTRY:="docker.io"}</span><p><span>set</span> -eo pipefail</p><p>CURRENT_NUM=0<br>ALL_IMAGES=<span>"<span>$(sed -n '/#/d;s/:/:/p' ${IMAGES_LIST_FILE} | sort -u)</span>"</span><br>TOTAL_NUMS=$(<span>echo</span> <span>"<span>${ALL_IMAGES}</span>"</span> | wc -l)</p><p><span><span>skopeo_copy</span></span>() {<br>    <span>if</span> docker run -it quay.io/skopeo/stable:latest copy --insecure-policy --src-tls-verify=<span>false</span> --dest-tls-verify=<span>false</span> \<br>        --src-creds caseycui:xxxxxxxxxxxxxxxxxxxxx --multi-arch all --override-os linux -q docker://<span>$1</span> docker://<span>$2</span>; <span>then</span><br>        <span>echo</span> -e <span>"<span>$GREEN_COL</span> Progress: <span>${CURRENT_NUM}</span>/<span>${TOTAL_NUMS}</span> sync <span>$1</span> to <span>$2</span> successful <span>$NORMAL_COL</span>"</span><br>    <span>else</span><br>        <span>echo</span> -e <span>"<span>$RED_COL</span> Progress: <span>${CURRENT_NUM}</span>/<span>${TOTAL_NUMS}</span> sync <span>$1</span> to <span>$2</span> failed <span>$NORMAL_COL</span>"</span><br>        <span>exit</span> 2<br>    <span>fi</span><br>}</p><p><span>for</span> image <span>in</span> <span>${ALL_IMAGES}</span>; <span>do</span><br>    <span>let</span> CURRENT_N192.168.2.23:5000UM=<span>${CURRENT_NUM}</span>+1<br>    skopeo_copy <span>${SOURCE_REGISTRY}</span>/<span>${image}</span> <span>${TARGET_REGISTRY}</span>/<span>${image}</span><br><span>done</span></p></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

运行效果如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><code>$ bash sync.sh<br> Progress: <span>1</span><span>/8 sync docker.io/</span>rancher<span>/coredns-coredns:1.8.3 to 192.168.2.23:5000/</span>rancher/coredns-coredns:<span>1.8</span>.<span>3</span> successful<br> Progress: <span>2</span><span>/8 sync docker.io/</span>rancher<span>/klipper-helm:v0.6.6-build20211022 to 192.168.2.23:5000/</span>rancher/klipper-helm:v0.<span>6.6</span>-build20211022 successful<br> Progress: <span>3</span><span>/8 sync docker.io/</span>rancher<span>/klipper-lb:v0.3.4 to 192.168.2.23:5000/</span>rancher/klipper-lb:v0.<span>3.4</span> successful<br> Progress: <span>4</span><span>/8 sync docker.io/</span>rancher<span>/library-busybox:1.32.1 to 192.168.2.23:5000/</span>rancher/library-busybox:<span>1.32</span>.<span>1</span> successful<br> Progress: <span>5</span><span>/8 sync docker.io/</span>rancher<span>/library-traefik:2.4.8 to 192.168.2.23:5000/</span>rancher/library-traefik:<span>2.4</span>.<span>8</span> successful<br> Progress: <span>6</span><span>/8 sync docker.io/</span>rancher<span>/local-path-provisioner:v0.0.19 to 192.168.2.23:5000/</span>rancher/local-path-provisioner:v0.<span>0.19</span> successful<br> Progress: <span>7</span><span>/8 sync docker.io/</span>rancher<span>/metrics-server:v0.3.6 to 192.168.2.23:5000/</span>rancher/metrics-server:v0.<span>3.6</span> successful<br> Progress: <span>8</span><span>/8 sync docker.io/</span>rancher<span>/pause:3.1 to 192.168.2.23:5000/</span>rancher/pause:<span>3.1</span> successful<br></code><p><i></i>AWK</p></pre></td></tr></tbody></table>

### 最终效果[](https://ewhisper.cn/posts/52241/#%E6%9C%80%E7%BB%88%E6%95%88%E6%9E%9C%20-2)

最终本地的镜像效果如下：

[![Docker Registry 中的多架构镜像](https://pic-cdn.ewhisper.cn/img/2022/07/04/0ea79f652edff190c1ea8c91a1fdde9d-multi-arch-image-in-registry.jpg)](https://pic-cdn.ewhisper.cn/img/2022/07/04/0ea79f652edff190c1ea8c91a1fdde9d-multi-arch-image-in-registry.jpg "Docker Registry 中的多架构镜像")

🎉🎉🎉

## 📚️ **Reference**[](https://ewhisper.cn/posts/52241/#%F0%9F%93%9A%EF%B8%8F-Reference-2)

-   [K8S 1.20 弃用 Docker 评估之 Docker 和 OCI 镜像格式的差别 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/7740/)
-   [Skopeo 简介 - K8S 1.20 弃用 Docker 评估之 Docker CLI 的替代产品 - 东风微鸣技术博客 (ewhisper.cn)](https://ewhisper.cn/posts/36509/#Skopeo-%20%20%E7%AE%80%E4%BB%8B)
-   [docker manifest | Docker Documentation](https://docs.docker.com/engine/reference/commandline/manifest/)
-   [containers/skopeo: Work with remote images registries - retrieving information, images, signing content (github.com)](https://github.com/containers/skopeo)
-   [Multi-architecture images in your registry - Azure Container Registry | Microsoft Docs](https://docs.microsoft.com/en-us/azure/container-registry/push-multi-architecture-images)
-   [镜像搬运工 skopeo](https://blog.k8s.li/skopeo.html)