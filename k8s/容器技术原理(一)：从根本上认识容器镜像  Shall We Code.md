## 容器技术原理(一)：从根本上认识容器镜像

 2021.7.20  [Linux](https://waynerv.com/categories/linux/)  5145  11 分钟  5

## 目录

1.  [从 OCI 规范说起](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E4%BB%8E-oci-%E8%A7%84%E8%8C%83%E8%AF%B4%E8%B5%B7)
2.  [镜像规范](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E9%95%9C%E5%83%8F%E8%A7%84%E8%8C%83)
    1.  [镜像里面都有什么](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E9%95%9C%E5%83%8F%E9%87%8C%E9%9D%A2%E9%83%BD%E6%9C%89%E4%BB%80%E4%B9%88)
    2.  [如何在镜像层中删除一个文件](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%A6%82%E4%BD%95%E5%9C%A8%E9%95%9C%E5%83%8F%E5%B1%82%E4%B8%AD%E5%88%A0%E9%99%A4%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6)
    3.  [如何将多个镜像层合并成一个文件系统](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%A6%82%E4%BD%95%E5%B0%86%E5%A4%9A%E4%B8%AA%E9%95%9C%E5%83%8F%E5%B1%82%E5%90%88%E5%B9%B6%E6%88%90%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
        1.  [什么是联合文件系统](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E4%BB%80%E4%B9%88%E6%98%AF%E8%81%94%E5%90%88%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
    4.  [Docker 中的 OverlayFS 是如何工作的](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#docker-%E4%B8%AD%E7%9A%84-overlayfs-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84)
        1.  [为什么容器的读写效率不如原生文件系统](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AE%B9%E5%99%A8%E7%9A%84%E8%AF%BB%E5%86%99%E6%95%88%E7%8E%87%E4%B8%8D%E5%A6%82%E5%8E%9F%E7%94%9F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
3.  [运行时规范](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E8%BF%90%E8%A1%8C%E6%97%B6%E8%A7%84%E8%8C%83)
    1.  [容器的生命周期](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%AE%B9%E5%99%A8%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
    2.  [容器的本质是进程](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%AE%B9%E5%99%A8%E7%9A%84%E6%9C%AC%E8%B4%A8%E6%98%AF%E8%BF%9B%E7%A8%8B)
4.  [实现和生态](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%AE%9E%E7%8E%B0%E5%92%8C%E7%94%9F%E6%80%81)
5.  [参考链接](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

## [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E4%BB%8E-oci-%E8%A7%84%E8%8C%83%E8%AF%B4%E8%B5%B7)[从 OCI 规范说起](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E4%BB%8E-oci-%E8%A7%84%E8%8C%83%E8%AF%B4%E8%B5%B7)

OCI（Open Container Initiative）规范是事实上的[[容器]]标准，已经被大部分容器实现以及容器编排系统所采用，包括[[ Docker ]]和 [[Kubernetes]]。它的出现是一段关于开源商业化的有趣历史：它由 Dokcer 公司作为领头者在 2015 年推出，但如今 Docker 公司在容器行业中已经成了打工仔。

从 OCI 规范开始了解容器镜像，可以让我们对容器技术建立更全面清晰的认知，而不是囿于实现细节。OCI 规范分为 `Image spec` 和 `Runtime spec` 两部分，它们分别覆盖了容器生命周期的不同阶段：

![](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/2021-07-20-oci-spec.png)

## [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E9%95%9C%E5%83%8F%E8%A7%84%E8%8C%83)[镜像规范](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E9%95%9C%E5%83%8F%E8%A7%84%E8%8C%83)

镜像规范定义了如何创建一个符合 OCI 规范的镜像，它规定了镜像的构建系统需要输出的内容和格式，输出的容器镜像可以被解包成一个 `runtime bundle` ，`runtime bundle` 是由特定文件和目录结构组成的一个文件夹，从中可以根据运行时标准运行容器。

### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E9%95%9C%E5%83%8F%E9%87%8C%E9%9D%A2%E9%83%BD%E6%9C%89%E4%BB%80%E4%B9%88)[镜像里面都有什么](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E9%95%9C%E5%83%8F%E9%87%8C%E9%9D%A2%E9%83%BD%E6%9C%89%E4%BB%80%E4%B9%88)

规范要求镜像内容必须包括以下 3 部分：

-   **Image Manifest**：提供了镜像的配置和文件系统层定位信息，可以看作是镜像的目录，文件格式为 `json` 。
-   **Image Layer Filesystem Changeset**：序列化之后的文件系统和文件系统变更，它们可按顺序一层层应用为一个容器的 rootfs，因此通常也被称为一个 `layer`（与下文提到的镜像层同义），文件格式可以是 `tar` ，`gzip` 等存档或压缩格式。
-   **Image Configuration**：包含了镜像在运行时所使用的执行参数以及有序的 rootfs 变更信息，文件类型为 `json`。

_rootfs (root file system)即 `/` 根挂载点所挂载的文件系统，是一个操作系统所包含的文件、配置和目录，但并不包括操作系统内核，同一台机器上的所有容器都共享宿主机操作系统的内核。_

接下来我们以 `Docker` 和 `nginx` 为例探索一个镜像的实际内容。拉取一个最新版本的 `nginx` 镜像将其 `save` 为 tar 包后解压：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ docker pull nginx
</span></span><span><span>$ docker save nginx -o nginx-img.tar
</span></span><span><span>$ mkdir nginx-img
</span></span><span><span>$ tar -xf nginx-img.tar --directory<span>=</span>nginx-img
</span></span></code></pre></td></tr></tbody></table>

得到 `nginx-img` 目录中的内容如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>nginx-img
</span></span><span><span>├── 013a6edf61f54428da349193e7a2077a714697991d802a1c5298b07dbe0519c9
</span></span><span><span>│&nbsp;&nbsp; ├── json
</span></span><span><span>│&nbsp;&nbsp; ├── layer.tar
</span></span><span><span>│&nbsp;&nbsp; └── VERSION
</span></span><span><span>├── 2bf70c858e6c8243c4713064cf43dea840866afefe52089a3b339f06576b930e
</span></span><span><span>│&nbsp;&nbsp; ├── json
</span></span><span><span>│&nbsp;&nbsp; ├── layer.tar
</span></span><span><span>│&nbsp;&nbsp; └── VERSION
</span></span><span><span>├── 490a3e67a61048564048a15d501b8e075d951d0dbba8098d5788bb8453f2371f
</span></span><span><span>│&nbsp;&nbsp; ├── json
</span></span><span><span>│&nbsp;&nbsp; ├── layer.tar
</span></span><span><span>│&nbsp;&nbsp; └── VERSION
</span></span><span><span>├── 4cdc5dd7eaadff5080649e8d0014f2f8d36d4ddf2eff2fdf577dd13da85c5d2f.json
</span></span><span><span>├── 761c908ee54e7ccd769e815f38e3040f7b3ff51f1c04f55aac12b9ea3d544cfe
</span></span><span><span>│&nbsp;&nbsp; ├── json
</span></span><span><span>│&nbsp;&nbsp; ├── layer.tar
</span></span><span><span>│&nbsp;&nbsp; └── VERSION
</span></span><span><span>├── 96bfd5bf4ab4c2513fb43534d51e816c4876620767858377d14dcc5a7de5f1fd
</span></span><span><span>│&nbsp;&nbsp; ├── json
</span></span><span><span>│&nbsp;&nbsp; ├── layer.tar
</span></span><span><span>│&nbsp;&nbsp; └── VERSION
</span></span><span><span>├── d18832ef411b346c36b7ba42a6c2e3f77097026fb80651c2d870f19c6fd9ccef
</span></span><span><span>│&nbsp;&nbsp; ├── json
</span></span><span><span>│&nbsp;&nbsp; ├── layer.tar
</span></span><span><span>│&nbsp;&nbsp; └── VERSION
</span></span><span><span>├── manifest.json
</span></span><span><span>└── repositories
</span></span></code></pre></td></tr></tbody></table>

首先查看 `manifest.json` 文件的内容，即该镜像的 Image Manifest：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ python -m json.tool manifest.json
</span></span><span><span>
</span></span><span><span><span>[</span>
</span></span><span><span>    <span>{</span>
</span></span><span><span>        <span>"Config"</span>: <span>"4cdc5dd7eaadff5080649e8d0014f2f8d36d4ddf2eff2fdf577dd13da85c5d2f.json"</span>,
</span></span><span><span>        <span>"Layers"</span>: <span>[</span>
</span></span><span><span>            <span>"490a3e67a61048564048a15d501b8e075d951d0dbba8098d5788bb8453f2371f/layer.tar"</span>,
</span></span><span><span>            <span>"2bf70c858e6c8243c4713064cf43dea840866afefe52089a3b339f06576b930e/layer.tar"</span>,
</span></span><span><span>            <span>"013a6edf61f54428da349193e7a2077a714697991d802a1c5298b07dbe0519c9/layer.tar"</span>,
</span></span><span><span>            <span>"761c908ee54e7ccd769e815f38e3040f7b3ff51f1c04f55aac12b9ea3d544cfe/layer.tar"</span>,
</span></span><span><span>            <span>"d18832ef411b346c36b7ba42a6c2e3f77097026fb80651c2d870f19c6fd9ccef/layer.tar"</span>,
</span></span><span><span>            <span>"96bfd5bf4ab4c2513fb43534d51e816c4876620767858377d14dcc5a7de5f1fd/layer.tar"</span>
</span></span><span><span>        <span>]</span>,
</span></span><span><span>        <span>"RepoTags"</span>: <span>[</span>
</span></span><span><span>            <span>"nginx:latest"</span>
</span></span><span><span>        <span>]</span>
</span></span><span><span>    <span>}</span>
</span></span><span><span><span>]</span>
</span></span></code></pre></td></tr></tbody></table>

其中记载了 `Config` 和 `Layers` 的文件定位信息，也就是标准中所规定的 Image Layer Filesystem Changeset 和 Image Configuration。

`Config` 存放在另一个 json 文件中，内容较多我们不做展示，具体包含了以下信息：

-   镜像的配置，在镜像解压成 `runtime bundle` 后将写入运行时配置文件。
-   镜像的 `layers` 之间的 Diff ID。
-   镜像的构建历史等元信息。

`Layers` 列表中的 tar 包共同组成了生成容器的 rootfs，容器的镜像是分层构建的， `Layers` 中的元素顺序还代表了镜像层叠加的顺序，所有 `layer` 组成一个由下往上叠加的栈式的结构。首先看一下基础层即第一条记录中的内容：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ mkdir base
</span></span><span><span>$ tar -xf 490a3e67a61048564048a15d501b8e075d951d0dbba8098d5788bb8453f2371f/layer.tar --directory<span>=</span>base
</span></span></code></pre></td></tr></tbody></table>

`base` 目录中解压得到的文件内容如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>drwxr-xr-x  <span>2</span> root root <span>4096</span> 6月  <span>21</span> 08:00 bin
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 boot
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 dev
</span></span><span><span>drwxr-xr-x <span>28</span> root root <span>4096</span> 6月  <span>21</span> 08:00 etc
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 home
</span></span><span><span>drwxr-xr-x  <span>7</span> root root   <span>85</span> 6月  <span>21</span> 08:00 lib
</span></span><span><span>drwxr-xr-x  <span>2</span> root root   <span>34</span> 6月  <span>21</span> 08:00 lib64
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 media
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 mnt
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 opt
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 proc
</span></span><span><span>drwx------  <span>2</span> root root   <span>37</span> 6月  <span>21</span> 08:00 root
</span></span><span><span>drwxr-xr-x  <span>3</span> root root   <span>30</span> 6月  <span>21</span> 08:00 run
</span></span><span><span>drwxr-xr-x  <span>2</span> root root <span>4096</span> 6月  <span>21</span> 08:00 sbin
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 srv
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 sys
</span></span><span><span>drwxrwxrwt  <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 tmp
</span></span><span><span>drwxr-xr-x <span>10</span> root root  <span>105</span> 6月  <span>21</span> 08:00 usr
</span></span><span><span>drwxr-xr-x <span>11</span> root root  <span>139</span> 6月  <span>21</span> 08:00 var
</span></span></code></pre></td></tr></tbody></table>

这已经是一个完整的 rootfs，再观察最上面一层 `layer` 所得到的文件内容：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>96bfd5bf4ab4c2513fb43534d51e816c4876620767858377d14dcc5a7de5f1fd/
</span></span><span><span>└── docker-entrypoint.d
</span></span><span><span>    └── 30-tune-worker-processes.sh
</span></span></code></pre></td></tr></tbody></table>

其中只有一个 shell 脚本文件，这说明镜像的构建过程是增量的，每一层都只包含了和更低一层相比所变更的文件内容，这也是容器镜像得以保持较小体积的原因。

### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%A6%82%E4%BD%95%E5%9C%A8%E9%95%9C%E5%83%8F%E5%B1%82%E4%B8%AD%E5%88%A0%E9%99%A4%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6)[如何在镜像层中删除一个文件](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E5%A6%82%E4%BD%95%E5%9C%A8%E9%95%9C%E5%83%8F%E5%B1%82%E4%B8%AD%E5%88%A0%E9%99%A4%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6)

`Layers` 中的每一层都是文件系统的变更集（ChangeSet），变更集包含新增、修改和删除三种变更，新增或修改（替换）文件的情况较好处理，但如何在应用变更集时删除一个文件呢，答案是用 Whiteouts 表示要删除的文件或文件夹。

Whiteouts 文件是一个具有特殊文件名的空文件，文件名中通过在要删除的路径基本名称添加前缀 `.wh.` 标志一个（更低一层中的）路径应该被删除。假如在某个 `layer` 有以下文件：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>./etc/my-app.d/
</span></span><span><span>./etc/my-app.d/default.cfg
</span></span><span><span>./bin/my-app-tools
</span></span><span><span>./etc/my-app-config
</span></span></code></pre></td></tr></tbody></table>

如果在应用的更高层 `layer` 中含有 `./etc/.wh.my-app-config` ，应用该层变更时原有的 `./etc/my-app-config` 路径将被删除。

### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%A6%82%E4%BD%95%E5%B0%86%E5%A4%9A%E4%B8%AA%E9%95%9C%E5%83%8F%E5%B1%82%E5%90%88%E5%B9%B6%E6%88%90%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)[如何将多个镜像层合并成一个文件系统](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E5%A6%82%E4%BD%95%E5%B0%86%E5%A4%9A%E4%B8%AA%E9%95%9C%E5%83%8F%E5%B1%82%E5%90%88%E5%B9%B6%E6%88%90%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)

规范中对于如何将多个镜像层应用成一个文件系统只有原理性的描述，假如我们要在 `layer A` 的基础上应用 `Layer B` ：

-   首先将 `Layer A` 中的文件系统目录以保留文件属性的方式复制到另一个快照目录 `A.snapshot`
-   然后在快照目录中执行 `Layer B` 所包含的文件变更，所有的更改不会影响原有的变更集。

在实践中会采用联合文件系统等更为高效的实现。

#### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E4%BB%80%E4%B9%88%E6%98%AF%E8%81%94%E5%90%88%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)[什么是联合文件系统](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E4%BB%80%E4%B9%88%E6%98%AF%E8%81%94%E5%90%88%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)

联合文件系统（Union File System）也叫 UnionFS，主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下。

下面以 Ubuntu 发行版以及 `unionfs-fuse` 实现为例演示联合挂载的效果：

1.  首先使用包管理器安装 `unionfs-fuse`，这是 UnionFS 的一个实现：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ apt install unionfs-fuse
    </span></span></code></pre></td></tr></tbody></table>
    
2.  然后创建如下目录结构：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span><span>2
    </span><span>3
    </span><span>4
    </span><span>5
    </span><span>6
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>A
    </span></span><span><span>├── a
    </span></span><span><span>└── x
    </span></span><span><span>B
    </span></span><span><span>├── b
    </span></span><span><span>└── x
    </span></span></code></pre></td></tr></tbody></table>
    
3.  创建目录 C 并将 A、B 目录联合挂载到 C 下：
    
4.  挂载后 C 目录内容如下：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span><span>2
    </span><span>3
    </span><span>4
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>C
    </span></span><span><span>├── a
    </span></span><span><span>├── b
    </span></span><span><span>└── x
    </span></span></code></pre></td></tr></tbody></table>
    
5.  如果我们分别编辑 A、B 目录中的 x 文件，会发现访问目录 C 中 x 文件得到的是 B/x 的内容（因为 B 在挂载时位于更上层）。
    

### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#docker-%E4%B8%AD%E7%9A%84-overlayfs-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84)[Docker 中的 OverlayFS 是如何工作的](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:docker-%E4%B8%AD%E7%9A%84-overlayfs-%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84)

Docker 目前在大部分发行版本中使用的联合文件系统实现是 `overlay2` ，相比其他实现它更加的轻量和高效，下面以实例来简单了解其工作方式。

接着上面 `nginx` 镜像的例子，拉取镜像后相应的 `layer` 解压在 `/var/lib/docker/overlay2` 目录中：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ll /var/lib/docker/overlay2 <span>|</span> tee layers.a
</span></span><span><span>
</span></span><span><span>drwx-----x <span>4</span> root root     <span>72</span> 7月  <span>20</span> 17:20 335aaf02cbde069ddf7aa0077fecac172d4b2f0240975ab0ebecc3f94f1420cc
</span></span><span><span>drwx-----x <span>3</span> root root     <span>47</span> 7月  <span>19</span> 10:04 560df35d349e6a750f1139db22d4cb52cba2a1f106616dc1c0c68b3cf11e3df6
</span></span><span><span>drwx-----x <span>4</span> root root     <span>72</span> 7月  <span>20</span> 17:20 769a9f5d698522d6e55bd9882520647bd84375a751a67a8ccad1f7bb1ca066dd
</span></span><span><span>drwx-----x <span>4</span> root root     <span>72</span> 7月  <span>20</span> 17:20 97aaf293fef495f0f06922d422a6187a952ec6ab29c0aa94cd87024c40e1a7e8
</span></span><span><span>drwx-----x <span>4</span> root root     <span>72</span> 7月  <span>20</span> 17:20 a91fb6955249dadfb34a3f5f06d083c192f2774fbec5fbb1db42a04e918432c0
</span></span><span><span>brw------- <span>1</span> root root 253, <span>1</span> 7月  <span>19</span> 10:00 backingFsBlockDev
</span></span><span><span>drwx-----x <span>4</span> root root     <span>72</span> 7月  <span>20</span> 17:20 fa29ec8cfe5a6c0b2cd1486f27a20a02867126edf654faad7f3520a220f3705f
</span></span><span><span>drwx-----x <span>2</span> root root    <span>278</span> 7月  <span>20</span> 17:25 l
</span></span></code></pre></td></tr></tbody></table>

我们将输出结果保存到 `layers.a` 文件中供之后对比。其中 6 个名称特别长的目录中存放了镜像的 6 个 `layer` （目录名称和 `manifest.json` 中的名称并不对应）， `l` 目录中包含了指向 `layers` 文件夹的软链接，主要目的是在执行 `mount` 命令时缩短目录标识符的长度以避免超出页大小限制。

每个 `layer` 文件夹包含的内容如下：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ <span>cd</span> /var/lib/docker/overlay2/
</span></span><span><span>$ ll 335aaf02cbde069ddf7aa0077fecac172d4b2f0240975ab0ebecc3f94f1420cc
</span></span><span><span>
</span></span><span><span>-rw------- <span>1</span> root root  <span>0</span> 7月  <span>15</span> 17:00 committed
</span></span><span><span>drwxr-xr-x <span>3</span> root root <span>33</span> 7月  <span>15</span> 17:00 diff
</span></span><span><span>-rw-r--r-- <span>1</span> root root <span>26</span> 7月  <span>15</span> 17:00 link
</span></span><span><span>-rw-r--r-- <span>1</span> root root <span>86</span> 7月  <span>15</span> 17:00 lower
</span></span><span><span>drwx------ <span>2</span> root root  <span>6</span> 7月  <span>15</span> 17:00 work
</span></span></code></pre></td></tr></tbody></table>

`link` 记录了 `l` 目录中的短链接， `lower` 中记录该 `layer` 的更低一层（如果没有该文件说明当前 `layer` 已经是最底下一层即基础层）， `work` 目录被 `overlay2` 内部所使用， `diff` 目录中存放了该 `layer` 所包含的文件系统内容：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ll 335aaf02cbde069ddf7aa0077fecac172d4b2f0240975ab0ebecc3f94f1420cc/diff/
</span></span><span><span>drwxr-xr-x  <span>2</span> root root    <span>6</span> 7月   <span>7</span> 03:39 docker-entrypoint.d
</span></span><span><span>drwxr-xr-x <span>20</span> root root <span>4096</span> 7月   <span>7</span> 03:39 etc
</span></span><span><span>drwxr-xr-x  <span>5</span> root root   <span>56</span> 7月   <span>7</span> 03:39 lib
</span></span><span><span>drwxrwxrwt  <span>2</span> root root    <span>6</span> 7月   <span>7</span> 03:39 tmp
</span></span><span><span>drwxr-xr-x  <span>7</span> root root   <span>66</span> 6月  <span>21</span> 08:00 usr
</span></span><span><span>drwxr-xr-x  <span>5</span> root root   <span>41</span> 6月  <span>21</span> 08:00 var
</span></span></code></pre></td></tr></tbody></table>

现在我们尝试基于该镜像运行一个容器，看看在容器阶段的联合挂载效果：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ docker run -d --name nginx_container  nginx
</span></span></code></pre></td></tr></tbody></table>

执行 `mount` 命令可确认新增了一个可读写的 `overlay` 挂载点：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ mount <span>|</span> grep overlay
</span></span><span><span>
</span></span><span><span>overlay on /var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/merged <span>type</span> overlay <span>(</span>rw,relatime,lowerdir<span>=</span>/var/lib/docker/overlay2/l/6Y7DPCTGLB6JHUPVBGTOEK2QFN:/var/lib/docker/overlay2/l/QMEEOSTJM2QON4M7PJJBB4KDEF:/var/lib/docker/overlay2/l/XNN2MRN4KWITFTZYLFUSLBP322:/var/lib/docker/overlay2/l/6DC6VDOMBZMLBZBT3QSOWLCR37:/var/lib/docker/overlay2/l/NXYWG253WSMELQKF2E2NH2GWCG:/var/lib/docker/overlay2/l/M4SO5XMO4VXRIJIGUHDMTATWH3:/var/lib/docker/overlay2/l/QI3P6ONJSLQI26DVPFGWIZI2EW,upperdir<span>=</span>/var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/diff,workdir<span>=</span>/var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/work<span>)</span>
</span></span></code></pre></td></tr></tbody></table>

该挂载点中即包含了所有镜像层 `layer` 组合而成的一个 rootfs：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ll /var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/merged
</span></span><span><span>
</span></span><span><span>drwxr-xr-x <span>2</span> root root <span>4096</span> 6月  <span>21</span> 08:00 bin
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 boot
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>43</span> 7月  <span>16</span> 16:52 dev
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>41</span> 7月   <span>7</span> 03:39 docker-entrypoint.d
</span></span><span><span>-rwxrwxr-x <span>1</span> root root <span>1202</span> 7月   <span>7</span> 03:39 docker-entrypoint.sh
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>19</span> 7月  <span>16</span> 16:52 etc
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 home
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>56</span> 7月   <span>7</span> 03:39 lib
</span></span><span><span>drwxr-xr-x <span>2</span> root root   <span>34</span> 6月  <span>21</span> 08:00 lib64
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 media
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 mnt
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 opt
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 proc
</span></span><span><span>drwx------ <span>2</span> root root   <span>37</span> 6月  <span>21</span> 08:00 root
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>23</span> 7月  <span>16</span> 16:52 run
</span></span><span><span>drwxr-xr-x <span>2</span> root root <span>4096</span> 6月  <span>21</span> 08:00 sbin
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>21</span> 08:00 srv
</span></span><span><span>drwxr-xr-x <span>2</span> root root    <span>6</span> 6月  <span>13</span> 18:30 sys
</span></span><span><span>drwxrwxrwt <span>1</span> root root    <span>6</span> 7月   <span>7</span> 03:39 tmp
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>66</span> 6月  <span>21</span> 08:00 usr
</span></span><span><span>drwxr-xr-x <span>1</span> root root   <span>19</span> 6月  <span>21</span> 08:00 var
</span></span></code></pre></td></tr></tbody></table>

除了将原来的镜像层联合挂载到如上所示的 `merged` 目录，通过 `diff` 命令可以看到，容器运行成功后 `/var/lib/docker/overlay2` 还会新增两个 `layer` 目录，`merged` 也位于其中一个目录下:

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ll /var/lib/docker/overlay2 <span>|</span> tee layers.b
</span></span><span><span>$ diff layers.a layers.b
</span></span><span><span>
</span></span><span><span>&gt; drwx-----x <span>5</span> root root     <span>69</span> 7月  <span>19</span> 10:08 bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011
</span></span><span><span>&gt; drwx-----x <span>4</span> root root     <span>72</span> 7月  <span>19</span> 10:08 bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011-init
</span></span><span><span>&lt; drwx-----x <span>2</span> root root    <span>210</span> 7月  <span>19</span> 10:08 l
</span></span><span><span>---
</span></span><span><span>&gt; drwx-----x <span>2</span> root root    <span>278</span> 7月  <span>19</span> 10:08 l
</span></span></code></pre></td></tr></tbody></table>

通过 `inspect` 命令探查已运行容器的 `GraphDriver`，可以更清晰地看到与镜像相比容器的 `layers` 所发生的变化 :

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ docker inspect nginx_container
</span></span><span><span>
</span></span><span><span>....
</span></span><span><span><span>"GraphDriver"</span>: <span>{</span>
</span></span><span><span>    <span>"Data"</span>: <span>{</span>
</span></span><span><span>        <span>"LowerDir"</span>: <span>"/var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011-init/diff:/var/lib/docker/overlay2/97aaf293fef495f0f06922d422a6187a952ec6ab29c0aa94cd87024c40e1a7e8/diff:/var/lib/docker/overlay2/fa29ec8cfe5a6c0b2cd1486f27a20a02867126edf654faad7f3520a220f3705f/diff:/var/lib/docker/overlay2/769a9f5d698522d6e55bd9882520647bd84375a751a67a8ccad1f7bb1ca066dd/diff:/var/lib/docker/overlay2/a91fb6955249dadfb34a3f5f06d083c192f2774fbec5fbb1db42a04e918432c0/diff:/var/lib/docker/overlay2/335aaf02cbde069ddf7aa0077fecac172d4b2f0240975ab0ebecc3f94f1420cc/diff:/var/lib/docker/overlay2/560df35d349e6a750f1139db22d4cb52cba2a1f106616dc1c0c68b3cf11e3df6/diff"</span>,
</span></span><span><span>        <span>"MergedDir"</span>: <span>"/var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/merged"</span>,
</span></span><span><span>        <span>"UpperDir"</span>: <span>"/var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/diff"</span>,
</span></span><span><span>        <span>"WorkDir"</span>: <span>"/var/lib/docker/overlay2/bab121ecb1d54b787b7b1834810baf212b035e28ca8d7875a09b1af837116011/work"</span>
</span></span><span><span>    <span>}</span>,
</span></span><span><span>    <span>"Name"</span>: <span>"overlay2"</span>
</span></span><span><span><span>}</span>
</span></span><span><span>....
</span></span></code></pre></td></tr></tbody></table>

`LowerDir` 中记录了原有的镜像层文件系统，另外在最上层还新增了一个 `init` 层，它们在容器运行阶段都是只读的；`MergedDir` 中记录了将 `LowerDir` 的所有目录进行联合挂载的挂载点；`UpperDir` 也是新增的一个 `layer` ，此时位于以上所有 `layers` 的最上层，与其他镜像层相比它是可读写的。容器阶段的 `layers` 示意图如下：

![](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/2021-07-20-image-layers.png)

创建容器时所新增的可写 `layer` 我们称为 `container layer`，容器运行阶段对文件系统的变更都只会写入到该 `layer` 中，包括对文件的新增、修改和删除，而不会改变更低层的原有镜像内容，这极大提升了镜像的分发效率。在镜像层和 `container layer` 之间的 `init` 层记录了容器在启动时写入的一些配置文件，这一过程发生在新增读写层之前，我们不希望把这些数据写入到原始镜像中。

这两个新增的层仅在容器运行阶段存在，容器删除后它们也会被删除。同一个镜像可以创建多个不同的容器，仅需要创建多个不同的可写层；运行时更改过的容器也可以重新打包为一个新的镜像，将可写层添加到新镜像的只读层中即可。

#### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AE%B9%E5%99%A8%E7%9A%84%E8%AF%BB%E5%86%99%E6%95%88%E7%8E%87%E4%B8%8D%E5%A6%82%E5%8E%9F%E7%94%9F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)[为什么容器的读写效率不如原生文件系统](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AE%B9%E5%99%A8%E7%9A%84%E8%AF%BB%E5%86%99%E6%95%88%E7%8E%87%E4%B8%8D%E5%A6%82%E5%8E%9F%E7%94%9F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)

为了最小化 I/O 以及缩减镜像体积，容器的联合文件系统在读写文件时会采取写时复制策略（copy-on-write），如果一个文件或目录存在于镜像中的较低层，而另一个层（包括可写层）需要对其进行读取访问时，会直接访问较低层的文件。当另一个层第一次需要写入该文件时（在构建镜像或运行容器时），该文件会被复制到该层并被修改。这一举措大大减少了容器的启动时间（启动时新建的可写层只有很少的文件写入），但容器运行后每次第一次修改某个文件都需要先将整个文件复制到 `container layer` 中。

以上原因导致容器运行时的读写效率不如原生文件系统（尤其是写入效率），在 `container layer` 中不适合进行大量的文件读写，通常建议将频繁写入的数据库、日志文件或目录等单独挂载出去，如使用 Docker 提供的 `Volume`，此时目录将通过绑定挂载（Bind Mount）直接挂载在可读写层中，绕过了写时复制带来的性能损耗。

## [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E8%BF%90%E8%A1%8C%E6%97%B6%E8%A7%84%E8%8C%83)[运行时规范](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E8%BF%90%E8%A1%8C%E6%97%B6%E8%A7%84%E8%8C%83)

运行时规范描述了容器的配置、执行环境和生命周期。它详细描述了不同容器运行时架构的配置文件 `config.json` 的字段格式，如何在执行环境中应用、注入这些配置，以确保容器内运行的程序在不同运行时之间环境一致，并通过容器的生命周期定义了一套统一的操作行为。

### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%AE%B9%E5%99%A8%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)[容器的生命周期](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E5%AE%B9%E5%99%A8%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)

规范所定义的[容器生命周期](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md#lifecycle)，描述了容器从创建到退出所发生的事件构成的时间线，该时间线中定义了 13 个不同的事件，下图描述了时间线中容器状态的变化：

![](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/2021-07-20-runtime-lifecycle.png)

规范中只定义了4种[容器状态](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md#state)，运行时实现可以在规范的基础上添加其他状态，同时标准还规定了运行时必须支持的[操作](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md#operations)：

-   Query State，查询容器的当前状态
-   Create，根据镜像及配置创建一个新的容器，但是不运行用户指定程序
-   Start，在一个已创建的容器中运行用户指定程序
-   Kill，发送特定信号终止容器进程
-   Delete，删除已停止容器所创建的资源

每个操作之前或之后还会触发不同的 hooks，符合规范的运行时必须执行这些 hooks。

### [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%AE%B9%E5%99%A8%E7%9A%84%E6%9C%AC%E8%B4%A8%E6%98%AF%E8%BF%9B%E7%A8%8B)[容器的本质是进程](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E5%AE%B9%E5%99%A8%E7%9A%84%E6%9C%AC%E8%B4%A8%E6%98%AF%E8%BF%9B%E7%A8%8B)

运行时规范中使用了 `container process` 这一概念，`container process` 等同于上文提到的用户指定程序和容器进程，有些场景也会称该进程为容器的 `init` 进程。运行一个容器必须在 `config.json` 中定义容器的 `container process`，可定义的字段包括命令参数、环境参数和执行路径等等。

容器状态的变化，实际上反映的是 `container process` 的变化。我们可以将容器的生命周期和状态变化划分为以下几个阶段：

1.  `container process` 执行之前。
    
    -   运行时执行 `create` 命令，根据 `config.json` 创建指定的资源。
        
    -   资源创建成功后，容器进入 `created` 状态。
        
2.  执行 `container process`。
    
    -   运行时执行 `start` 命令。
        
    -   运行时运行用户指定程序，即 `container process`。
        
    -   容器进入 `running` 状态。
        
3.  `container process` 进程结束，结束的原因可能是该程序执行结束、出错或崩溃，以及运行时通过 `kill` 命令向其发出终止信号。
    
    -   容器进入 `stoppped` 状态。
        
    -   运行时执行 `delete` 命令，所有通过 `create` 命令创建的资源被清除。
        

容器运行时的核心就是 `container process`：镜像文件系统满足运行进程所需的依赖，运行时所作的准备工作是为了正确的运行该进程，运行时持续的监测该进程的状态，一旦该进程结束即宣告容器（暂时）死亡，运行时进行收尾的清理工作。

当然容器中也可以运行其他进程，但这些进程只是共用 `container process` 的环境。

## [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%AE%9E%E7%8E%B0%E5%92%8C%E7%94%9F%E6%80%81)[实现和生态](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E5%AE%9E%E7%8E%B0%E5%92%8C%E7%94%9F%E6%80%81)

Docker 向 OCI 规范捐献了其容器运行时 `runC` 项目，作为该规范的标准实现。目前已有的大部分容器项目都直接将 `runC` 作为运行时实现。

下图可以概括容器生态内 Docker 相关的组织及项目之间的关系：

![](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/2021-07-20-runc.png)

Kubernetes 定义了 CRI（Container Runtime Interface）以实现可替换的容器运行时，目前有 `cri-containerd` 、`cri-o` 和 `docker` 等几种实现，但它们实际上也都基于 `runC`：

![](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/2021-07-20-k8s-cri.png)

## [](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/container-fundamentals-learn-container-with-oci-spec/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Understand Container: OCI Specification](https://pierrchen.blogspot.com/2018/04/oci-open-container-spec-that-rules-all.html)
    
-   [Open Container Initiative Image Format Specification](https://github.com/opencontainers/image-spec/blob/main/spec.md)
    
-   [Open Container Initiative Runtime Specification](https://github.com/opencontainers/runtime-spec/blob/master/spec.md)
    
-   [About storage drivers](https://docs.docker.com/storage/storagedriver/)
    
-   [Use the OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)
    
-   [Docker (容器) 的原理](https://www.kawabangga.com/posts/4224)
    
-   深入剖析Kubernetes - 张磊
    

updatedupdated2022-07-172022-07-17

[容器](https://waynerv.com/tags/%E5%AE%B9%E5%99%A8/) [Kubernetes](https://waynerv.com/tags/kubernetes/) [Docker](https://waynerv.com/tags/docker/)