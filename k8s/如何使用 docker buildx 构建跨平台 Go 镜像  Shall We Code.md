 2021.9.2  [Golang](https://waynerv.com/categories/golang/)  4325  9 分钟  6

## 目录

1.  [前提](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%89%8D%E6%8F%90)
2.  [docker buildx](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#docker-buildx)
    1.  [启用 Buildx](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%90%AF%E7%94%A8-buildx)
    2.  [builder 实例](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#builder-%E5%AE%9E%E4%BE%8B)
    3.  [构建驱动](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%9E%84%E5%BB%BA%E9%A9%B1%E5%8A%A8)
3.  [buildx 的跨平台构建策略](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#buildx-%E7%9A%84%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA%E7%AD%96%E7%95%A5)
4.  [一次构建多个架构 Go 镜像实践](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E4%B8%80%E6%AC%A1%E6%9E%84%E5%BB%BA%E5%A4%9A%E4%B8%AA%E6%9E%B6%E6%9E%84-go-%E9%95%9C%E5%83%8F%E5%AE%9E%E8%B7%B5)
    1.  [源代码和 Dockerfile](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%BA%90%E4%BB%A3%E7%A0%81%E5%92%8C-dockerfile)
    2.  [执行跨平台构建](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%89%A7%E8%A1%8C%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA)
    3.  [验证构建结果](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E9%AA%8C%E8%AF%81%E6%9E%84%E5%BB%BA%E7%BB%93%E6%9E%9C)
5.  [如何交叉编译 Golang 的 CGO 项目](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%A6%82%E4%BD%95%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-golang-%E7%9A%84-cgo-%E9%A1%B9%E7%9B%AE)
    1.  [准备交叉编译环境和依赖](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%87%86%E5%A4%87%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83%E5%92%8C%E4%BE%9D%E8%B5%96)
    2.  [交叉编译 CGO 示例](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-cgo-%E7%A4%BA%E4%BE%8B)
6.  [总结](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%80%BB%E7%BB%93)
7.  [参考链接](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

在不同操作系统和处理器架构上运行应用是很普遍的场景，因此为不同平台单独构建发布版本是一种常见做法。当我们用来开发应用的平台与部署的目标平台不同时，实现这一目标并不容易。例如在 x86 架构上开发一个应用程序并将其部署到 ARM 平台的机器上，通常需要准备 ARM 平台的基础设施用于开发和编译。

一次构建多处部署的镜像分发大幅提高了应用的交付效率，对于需要跨平台部署应用的场景，利用 docker buildx 构建跨平台的镜像也是一种快捷高效的解决方案。

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%89%8D%E6%8F%90)[前提](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E5%89%8D%E6%8F%90)

大部分镜像托管平台支持多平台镜像，这意味着镜像仓库中单个标签可以包含不同平台的多个镜像，以 `docker hub` 的 `python` 镜像仓库为例，`3.9.6` 这个标签就包含了 10 个不同系统和架构的镜像（平台 = 系统 + 架构）：

![Untitled](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/2020-09-02-python-multi-arch-images.png)

通过 `docker pull` 或 `docker run` 拉取一个支持跨平台的镜像时，`docker` 会自动选择与当前运行平台相匹配的镜像。由于该特性的存在，在进行镜像的跨平台分发时，我们不需要对镜像的消费做任何处理，只需要关心镜像的生产，即如何构建跨平台的镜像。

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#docker-buildx)[docker buildx](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:docker-buildx)

默认的 `docker build` 命令无法完成跨平台构建任务，我们需要为 `docker` 命令行安装 `buildx` 插件扩展其功能。`buildx` 能够使用由 [Moby BuildKit](https://github.com/moby/buildkit) 提供的构建镜像额外特性，它能够创建多个 builder 实例，在多个节点并行地执行构建任务，以及跨平台构建。

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%90%AF%E7%94%A8-buildx)[启用 Buildx](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E5%90%AF%E7%94%A8-buildx)

macOS 或 Windows 系统的 Docker Desktop，以及 Linux 发行版通过 `deb` 或者 `rpm` 包所安装的 `docker` 内置了 `buildx`，不需要另行安装。

如果你的 `docker` 没有 `buildx` 命令，可以下载二进制包进行安装：

1.  首先从 [Docker buildx](https://github.com/docker/buildx/releases/latest) 项目的 release 页面找到适合自己平台的二进制文件。
2.  下载二进制文件到本地并重命名为 `docker-buildx`，移动到 docker 的插件目录 `~/.docker/cli-plugins`。
3.  向二进制文件授予可执行权限。

如果本地的 `docker` 版本高于 19.03，可以通过以下命令直接在本地构建并安装，这种方式更为方便：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ <span>export</span> <span>DOCKER_BUILDKIT</span><span>=</span><span>1</span>
</span></span><span><span>$ docker build --platform<span>=</span><span>local</span> -o . git://github.com/docker/buildx
</span></span><span><span>$ mkdir -p ~/.docker/cli-plugins
</span></span><span><span>$ mv buildx ~/.docker/cli-plugins/docker-buildx
</span></span></code></pre></td></tr></tbody></table>

使用 `buildx` 进行构建的方法如下：

`buildx` 和 `docker build` 命令的使用体验基本一致，还支持 `build` 常用的选项如 `-t`、`-f`等。

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#builder-%E5%AE%9E%E4%BE%8B)[builder 实例](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:builder-%E5%AE%9E%E4%BE%8B)

`docker buildx` 通过 builder 实例对象来管理构建配置和节点，命令行将构建任务发送至 builder 实例，再由 builder 指派给符合条件的节点执行。我们可以基于同一个 `docker` 服务程序创建多个 builder 实例，提供给不同的项目使用以隔离各个项目的配置，也可以为一组远程 `docker` 节点创建一个 builder 实例组成构建阵列，并在不同阵列之间快速切换。

使用 `docker buildx create` 命令可以创建 builder 实例，这将以当前使用的 docker 服务为节点创建一个新的 builder 实例。要使用一个远程节点，可以在创建示例时通过 `DOCKER_HOST` 环境变量指定远程端口或提前切换到远程节点的 `docker context`。下面以远程节点创建一个新的 builder 实例并通过命令行选项指定其驱动、目标平台和实例名称：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ <span>export</span> <span>DOCKER_HOST</span><span>=</span>tcp://10.10.150.66:2375
</span></span><span><span>$ docker buildx create --driver docker-container --platform linux/amd64,linux/arm64 --name remote-builder
</span></span><span><span>remote-builder
</span></span></code></pre></td></tr></tbody></table>

`docker buildx ls` 将列出所有可用的 builder 实例和实例中的节点：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker buildx ls
</span></span><span><span>NAME/NODE         DRIVER/ENDPOINT         STATUS   PLATFORMS
</span></span><span><span>remote-builder    docker-container                 
</span></span><span><span>  remote-builder0 tcp://10.10.150.66:2375 inactive linux/amd64*, linux/arm64*
</span></span><span><span>default *         docker                           
</span></span><span><span>  default         default                 running  linux/amd64, linux/386
</span></span></code></pre></td></tr></tbody></table>

实例创建之后可以继续向它添加新的节点，通过 `docker buildx create` 命令的 `--append <node>` 选项可将节点加入到 `--name <builder>` 选项指定的 builder 实例：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span> $ docker buildx create --name default --append remote-builder0
</span></span></code></pre></td></tr></tbody></table>

`docker buildx inspect`、`docker buildx stop` 和 `docker buildx rm` 命令用于管理一个实例的生命周期。

`docker buildx use <builder>` 将切换到所指定的 builder 实例。

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%9E%84%E5%BB%BA%E9%A9%B1%E5%8A%A8)[构建驱动](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E6%9E%84%E5%BB%BA%E9%A9%B1%E5%8A%A8)

buildx 实例通过两种方式来执行构建任务，两种执行方式被称为使用不同的「驱动」：

-   `docker` 驱动：使用 Docker 服务程序中集成的 BuildKit 库执行构建。
-   `docker-container` 驱动：启动一个包含 BuildKit 的容器并在容器中执行构建。

`docker` 驱动无法使用一小部分 `buildx` 的特性（如在一次运行中同时构建多个平台镜像），此外在镜像的默认输出格式上也有所区别：`docker` 驱动默认将构建结果以 Docker 镜像格式直接输出到 `docker` 的镜像目录（通常是 `/var/lib/overlay2`），之后执行 `docker images` 命令可以列出所输出的镜像；而 `docker container` 则需要通过 `--output` 选项指定输出格式为镜像或其他格式。

为了一次性构建多个平台的镜像，下文将使用 `docker container` 驱动的 builder 实例。

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#buildx-%E7%9A%84%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA%E7%AD%96%E7%95%A5)[buildx 的跨平台构建策略](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:buildx-%E7%9A%84%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA%E7%AD%96%E7%95%A5)

根据构建节点和目标程序语言不同，`buildx` 支持以下三种跨平台构建策略：

1.  通过 QEMU 的用户态模式创建轻量级的虚拟机，在虚拟机系统中构建镜像。
2.  在一个 builder 实例中加入多个不同目标平台的节点，通过原生节点构建对应平台镜像。
3.  分阶段构建并且交叉编译到不同的目标架构。

QEMU 通常用于模拟完整的操作系统，它还可以通过用户态模式运行：以 `binfmt_misc` 在宿主机系统中注册一个二进制转换处理程序，并在程序运行时动态翻译二进制文件，根据需要将系统调用从目标 CPU 架构转换为当前系统的 CPU 架构。最终的效果就像在一个虚拟机中运行目标 CPU 架构的二进制文件。Docker Desktop 内置了 QEMU 支持，其他满足运行要求的平台可通过以下方式安装：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker run --privileged --rm tonistiigi/binfmt --install all
</span></span></code></pre></td></tr></tbody></table>

这种方式不需要对已有的 Dockerfile 做任何修改，实现的成本很低，但显而易见效率并不高。

将不同系统架构的原生节点添加到 builder 实例中可以为跨平台编译带来更好的支持，而且效率更高，但需要有足够的基础设施支持。

如果构建项目所使用的程序语言支持交叉编译（如 C 和 Go），可以利用 Dockerfile 提供的分阶段构建特性：首先在和构建节点相同的架构中编译出目标架构的二进制文件，再将这些二进制文件复制到目标架构的另一镜像中。下文会使用 Go 实现一个具体的示例。这种方式不需要额外的硬件，也能得到较好的性能，但只有特定编程语言能够实现。

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E4%B8%80%E6%AC%A1%E6%9E%84%E5%BB%BA%E5%A4%9A%E4%B8%AA%E6%9E%B6%E6%9E%84-go-%E9%95%9C%E5%83%8F%E5%AE%9E%E8%B7%B5)[一次构建多个架构 Go 镜像实践](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E4%B8%80%E6%AC%A1%E6%9E%84%E5%BB%BA%E5%A4%9A%E4%B8%AA%E6%9E%B6%E6%9E%84-go-%E9%95%9C%E5%83%8F%E5%AE%9E%E8%B7%B5)

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%BA%90%E4%BB%A3%E7%A0%81%E5%92%8C-dockerfile)[源代码和 Dockerfile](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E6%BA%90%E4%BB%A3%E7%A0%81%E5%92%8C-dockerfile)

下面将以一个简单的 Go 项目作为示例，假设示例程序文件 `main.go` 内容如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="go"><span><span><span>package</span> <span>main</span>
</span></span><span><span>
</span></span><span><span><span>import</span> <span>(</span>
</span></span><span><span><span>"fmt"</span>
</span></span><span><span><span>"runtime"</span>
</span></span><span><span><span>)</span>
</span></span><span><span>
</span></span><span><span><span>func</span> <span>main</span><span>()</span> <span>{</span>
</span></span><span><span><span>fmt</span><span>.</span><span>Println</span><span>(</span><span>"Hello world!"</span><span>)</span>
</span></span><span><span><span>fmt</span><span>.</span><span>Printf</span><span>(</span><span>"Running in [%s] architecture.\n"</span><span>,</span> <span>runtime</span><span>.</span><span>GOARCH</span><span>)</span>
</span></span><span><span><span>}</span>
</span></span></code></pre></td></tr></tbody></table>

定义构建过程的 Dockerfile 如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="dockerfile"><span><span><span>FROM</span><span> --platform=$BUILDPLATFORM golang:1.14 as builder</span><span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>ARG</span> TARGETARCH<span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>WORKDIR</span><span> /app</span><span>
</span></span></span><span><span><span></span><span>COPY</span> main.go /app/main.go<span>
</span></span></span><span><span><span></span><span>RUN</span> <span>GOOS</span><span>=</span>linux <span>GOARCH</span><span>=</span><span>$TARGETARCH</span> go build -a -o output/main main.go<span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>FROM</span><span> alpine:latest</span><span>
</span></span></span><span><span><span></span><span>WORKDIR</span><span> /root</span><span>
</span></span></span><span><span><span></span><span>COPY</span> --from<span>=</span>builder /app/output/main .<span>
</span></span></span><span><span><span></span><span>CMD</span> /root/main<span>
</span></span></span></code></pre></td></tr></tbody></table>

构建过程分为两个阶段：

-   在一阶段中，我们将拉取一个和当前构建节点相同平台的 `golang` 镜像，并使用 Go 的交叉编译特性将其编译为目标架构的二进制文件。
-   然后拉取目标平台的 `alpine` 镜像，并将上一阶段的编译结果拷贝到镜像中。

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%89%A7%E8%A1%8C%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA)[执行跨平台构建](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E6%89%A7%E8%A1%8C%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA)

执行构建命令时，除了指定镜像名称，另外两个重要的选项是指定目标平台和输出格式。

`docker buildx build` 通过 `--platform` 选项指定构建的目标平台。Dockerfile 中的 FROM 指令如果没有设置 `--platform` 标志，就会以目标平台拉取基础镜像，最终生成的镜像也将属于目标平台。此外 Dockerfile 中可通过 `BUILDPLATFORM`、`TARGETPLATFORM`、`BUILDARCH` 和 `TARGETARCH` 等参数使用该选项的值。当使用 `docker-container` 驱动时，这个选项可以接受用逗号分隔的多个值作为输入以同时指定多个目标平台，所有平台的构建结果将合并为一个整体的镜像列表作为输出，因此无法直接输出为本地的 `docker images` 镜像。

`docker buildx build` 支持丰富的输出行为，通过`--output=[PATH,-,type=TYPE[,KEY=VALUE]` 选项可以指定构建结果的输出类型和路径等，常用的输出类型有以下几种：

-   local：构建结果将以文件系统格式写入 `dest` 指定的本地路径， 如 `--output type=local,dest=./output`。
-   tar：构建结果将在打包后写入 `dest` 指定的本地路径。
-   oci：构建结果以 OCI 标准镜像格式写入 `dest` 指定的本地路径。
-   docker：构建结果以 Docker 标准镜像格式写入 `dest` 指定的本地路径或加载到 `docker` 的镜像库中。同时指定多个目标平台时无法使用该选项。
-   image：以镜像或者镜像列表输出，并支持 `push=true` 选项直接推送到远程仓库，同时指定多个目标平台时可使用该选项。
-   registry：`type=image,push=true` 的精简表示。

对本示例我们执行如下 `docker buildx build` 命令：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker buildx build --platform linux/amd64,linux/arm64,linux/arm -t registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo -o <span>type</span><span>=</span>registry .
</span></span></code></pre></td></tr></tbody></table>

该命令将在当前目录同时构建 `linux/amd64`、 `linux/arm64` 和 `linux/arm` 三种平台的镜像，并将输出结果直接推送到远程的阿里云镜像仓库中。

构建过程可拆解如下：

1.  `docker` 将构建上下文传输给 builder 实例。
2.  builder 为命令行 `--platform` 选项指定的每一个目标平台构建镜像，包括拉取基础镜像和执行构建步骤。
3.  导出构建结果，镜像文件层被推送到远程仓库。
4.  生成一个清单 JSON 文件，并将其作为镜像标签推送给远程仓库。

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E9%AA%8C%E8%AF%81%E6%9E%84%E5%BB%BA%E7%BB%93%E6%9E%9C)[验证构建结果](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E9%AA%8C%E8%AF%81%E6%9E%84%E5%BB%BA%E7%BB%93%E6%9E%9C)

运行结束后可以通过 `docker buildx imagetools` 探查已推送到远程仓库的镜像：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker buildx imagetools inspect registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest
</span></span><span><span>Name:      registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest
</span></span><span><span>MediaType: application/vnd.docker.distribution.manifest.list.v2+json
</span></span><span><span>Digest:    sha256:e2c3c5b330c19ac9d09f8aaccc40224f8673e12b88ff59cb68971c36b76e95ca
</span></span><span><span>           
</span></span><span><span>Manifests: 
</span></span><span><span>  Name:      registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest@sha256:cb6a7614ee3db03c8858e3680b1585f32a6fe3de9b371e37e25cf42a83f6e0ba
</span></span><span><span>  MediaType: application/vnd.docker.distribution.manifest.v2+json
</span></span><span><span>  Platform:  linux/amd64
</span></span><span><span>             
</span></span><span><span>  Name:      registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest@sha256:034aa0077a452a6c2585f8b4969c7c85d5d2bf65f801fcc803a00d0879ce900e
</span></span><span><span>  MediaType: application/vnd.docker.distribution.manifest.v2+json
</span></span><span><span>  Platform:  linux/arm64
</span></span><span><span>             
</span></span><span><span>  Name:      registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest@sha256:db0ee3a876fb789d2e733471385eef0a056f64ee12d9e7ef94e411469d054eb5
</span></span><span><span>  MediaType: application/vnd.docker.distribution.manifest.v2+json
</span></span><span><span>  Platform:  linux/arm/v7
</span></span></code></pre></td></tr></tbody></table>

最后在不同的平台以 `latest` 标签拉取并运行镜像，验证构建结果是否正确。 使用 Docker Desktop 时，其本身集成的虚拟化功能可以运行不同平台的镜像，可以直接以 `sha256` 值拉取镜像：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker run --rm registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest@sha256:cb6a7614ee3db03c8858e3680b1585f32a6fe3de9b371e37e25cf42a83f6e0ba
</span></span><span><span>Hello world!
</span></span><span><span>Running in <span>[</span>amd64<span>]</span> architecture.
</span></span></code></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker run --rm registry.cn-hangzhou.aliyuncs.com/waynerv/arch-demo:latest@sha256:034aa0077a452a6c2585f8b4969c7c85d5d2bf65f801fcc803a00d0879ce900e
</span></span><span><span>Hello world!
</span></span><span><span>Running in <span>[</span>arm64<span>]</span> architecture.
</span></span></code></pre></td></tr></tbody></table>

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%A6%82%E4%BD%95%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-golang-%E7%9A%84-cgo-%E9%A1%B9%E7%9B%AE)[如何交叉编译 Golang 的 CGO 项目](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E5%A6%82%E4%BD%95%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-golang-%E7%9A%84-cgo-%E9%A1%B9%E7%9B%AE)

支持交叉编译到常见的操作系统和 CPU 架构是 Golang 的一大优势，但以上示例中的解决方案只适用于纯 Go 代码，如果项目中通过 `cgo` 调用了 C 代码，情况会变得更加复杂。

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%87%86%E5%A4%87%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83%E5%92%8C%E4%BE%9D%E8%B5%96)[准备交叉编译环境和依赖](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E5%87%86%E5%A4%87%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83%E5%92%8C%E4%BE%9D%E8%B5%96)

为了能够顺利编译 C 代码到目标平台，首先需要在编译环境中安装目标平台的 C 交叉编译器（通常基于 `gcc`），常用的 Linux 发行版会提供大部分平台的交叉编译器安装包，可以直接通过包管理器安装。

其次还需要安装目标平台的 C 标准库（通常标准库会作为交叉编译器的安装依赖，不需要单独安装），另外取决于你所调用的 C 代码的依赖关系，可能还需要安装一些额外的 C 依赖库（如 `libopus-dev` 之类）。

我们将使用 `amd64` 架构的 `golang:1.14` 官方镜像作为基础镜像执行编译，其使用的 Linux 发行版为 Debian。假设交叉编译的目标平台是 `linux/arm64`，则需要准备的交叉编译器为 `gcc-aarch64-linux-gnu`，C 标准库为 `libc6-dev-arm64-cross`，安装方式为：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ apt-get update
</span></span><span><span>$ apt-get install gcc-aarch64-linux-gnu
</span></span></code></pre></td></tr></tbody></table>

`libc6-dev-arm64-cross` 会同时被安装。

得益于 Debian 包管理器 `dpkg` 提供的多架构安装能力，假如我们的代码依赖 `libopus-dev` 等非标准库，可通过 `<library>:<architecture>` 的方式安装其 `arm64` 架构的安装包：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ dpkg --add-architecture arm64
</span></span><span><span>$ apt-get update
</span></span><span><span>$ apt-get install -y libopus-dev:arm64
</span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-cgo-%E7%A4%BA%E4%BE%8B)[交叉编译 CGO 示例](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91-cgo-%E7%A4%BA%E4%BE%8B)

假设有如下 `cgo` 的示例代码：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="go"><span><span><span>package</span> <span>main</span>
</span></span><span><span>
</span></span><span><span><span>/*
</span></span></span><span><span><span>#include &lt;stdlib.h&gt;
</span></span></span><span><span><span>*/</span>
</span></span><span><span><span>import</span> <span>"C"</span>
</span></span><span><span><span>import</span> <span>"fmt"</span>
</span></span><span><span>
</span></span><span><span><span>func</span> <span>Random</span><span>()</span> <span>int</span> <span>{</span>
</span></span><span><span>    <span>return</span> <span>int</span><span>(</span><span>C</span><span>.</span><span>random</span><span>())</span>
</span></span><span><span><span>}</span>
</span></span><span><span>
</span></span><span><span><span>func</span> <span>Seed</span><span>(</span><span>i</span> <span>int</span><span>)</span> <span>{</span>
</span></span><span><span>    <span>C</span><span>.</span><span>srandom</span><span>(</span><span>C</span><span>.</span><span>uint</span><span>(</span><span>i</span><span>))</span>
</span></span><span><span><span>}</span>
</span></span><span><span>
</span></span><span><span><span>func</span> <span>main</span><span>()</span>  <span>{</span>
</span></span><span><span>    <span>rand</span> <span>:=</span> <span>Random</span><span>()</span>
</span></span><span><span>    <span>fmt</span><span>.</span><span>Printf</span><span>(</span><span>"Hello %d\n"</span><span>,</span> <span>rand</span><span>)</span>
</span></span><span><span><span>}</span>
</span></span></code></pre></td></tr></tbody></table>

将使用的 Dockerfile 如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="dockerfile"><span><span><span>FROM</span><span> --platform=$BUILDPLATFORM golang:1.14 as builder</span><span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>ARG</span> TARGETARCH<span>
</span></span></span><span><span><span></span><span>RUN</span> apt-get update <span>&amp;&amp;</span> apt-get install -y gcc-aarch64-linux-gnu<span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>WORKDIR</span><span> /app</span><span>
</span></span></span><span><span><span></span><span>COPY</span> . /app/<span>
</span></span></span><span><span><span>
</span></span></span><span><span><span></span><span>RUN</span> <span>if</span> <span>[</span> <span>"</span><span>$TARGETARCH</span><span>"</span> <span>=</span> <span>"arm64"</span> <span>]</span><span>;</span> <span>then</span> <span>CC</span><span>=</span>aarch64-linux-gnu-gcc <span>&amp;&amp;</span> <span>CC_FOR_TARGET</span><span>=</span>gcc-aarch64-linux-gnu<span>;</span> <span>fi</span> <span>&amp;&amp;</span> <span>\
</span></span></span><span><span><span></span>  <span>CGO_ENABLED</span><span>=</span><span>1</span> <span>GOOS</span><span>=</span>linux <span>GOARCH</span><span>=</span><span>$TARGETARCH</span> <span>CC</span><span>=</span><span>$CC</span> <span>CC_FOR_TARGET</span><span>=</span><span>$CC_FOR_TARGET</span> go build -a -ldflags <span>'-extldflags "-static"'</span> -o /main main.go<span>
</span></span></span></code></pre></td></tr></tbody></table>

Dockerfile 中通过 `apt-get` 安装了 `gcc-aarch64-linux-gnu` 作为交叉编译器，示例程序较为简单因此不需要额外的依赖库。在执行 `go build` 进行编译时，需要通过 `CC` 和 `CC_FOR_TARGET` 环境变量指定所使用的交叉编译器。

为了基于同一份 Dockerfile 执行多个目标平台的编译（假设目标架构只有 `amd64`/`arm64`），最下方的 `RUN` 指令使用了一个小技巧，通过 Bash 的条件判断语法来执行不同的编译命令：

-   假如构建任务的目标平台是 `arm64`，则指定 `CC` 和 `CC_FOR_TARGET` 环境变量为已安装的交叉编译器（注意它们的值有所不同）。
-   假如构建任务的目标平台是 `amd64`，则不指定交叉编译器相关的变量，此时将使用默认的 `gcc` 作为编译器。

最后使用 buildx 执行构建的命令如下：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="shell"><span><span>$ docker buildx build --platform linux/amd64,linux/arm64 -t registry.cn-hangzhou.aliyuncs.com/waynerv/cgo-demo -o <span>type</span><span>=</span>registry .
</span></span></code></pre></td></tr></tbody></table>

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E6%80%BB%E7%BB%93)[总结](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E6%80%BB%E7%BB%93)

有了 `Buildx` 插件的帮助，在缺少基础设施的情况下，我们也能使用 `docker` 方便地构建跨平台的应用镜像。

但默认通过 QEMU 虚拟化目标平台指令的方式有明显地性能瓶颈，如果编写应用的语言支持交叉编译，我们可以通过结合 `buildx` 和交叉编译获得更高的效率。

本文最后介绍了一种进阶场景的解决方案：如何对使用了 CGO 的 Golang 项目进行交叉编译，并给出了编译到 `linux/arm64` 平台的示例。

## [](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Leverage multi-CPU architecture support](https://docs.docker.com/desktop/multi-arch/)
-   [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/#build-with-buildx)
-   [docker buildx build](https://docs.docker.com/engine/reference/commandline/buildx_build/#output)
-   [C? Go? Cgo! - go.dev](https://go.dev/blog/cgo)
-   [Cross-Compiling Golang (CGO) Projects](https://dh1tw.de/2019/12/cross-compiling-golang-cgo-projects/)

updatedupdated2022-07-172022-07-17

[[[云原生]]](https://waynerv.com/tags/%E4%BA%91%E5%8E%9F%E7%94%9F/) [Docker](https://waynerv.com/tags/docker/)