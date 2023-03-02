> 内容主要搬迁自：[Docker Cheat Sheet (opens new window)](https://github.com/wsargent/docker-cheat-sheet/tree/master/zh-cn)

-   [为何使用 Docker](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E4%B8%BA%E4%BD%95%E4%BD%BF%E7%94%A8-docker)
-   [运维](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E8%BF%90%E7%BB%B4)
-   [容器(Container)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%B9%E5%99%A8container)
-   [镜像(Images)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E9%95%9C%E5%83%8Fimages)
-   [网络(Networks)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%BD%91%E7%BB%9Cnetworks)
-   [仓管中心和仓库(Registry & Repository)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E4%BB%93%E7%AE%A1%E4%B8%AD%E5%BF%83%E5%92%8C%E4%BB%93%E5%BA%93registry--repository)
-   [Dockerfile](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#dockerfile)
-   [层(Layers)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%B1%82layers)
-   [链接(Links)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E9%93%BE%E6%8E%A5links)
-   [卷标(Volumes)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8D%B7%E6%A0%87volumes)
-   [暴露端口(Exposing ports)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3exposing-ports)
-   [最佳实践](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)
-   [安全(Security)](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%89%E5%85%A8security)
-   [小贴士](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%B0%8F%E8%B4%B4%E5%A3%AB)
-   [参考资料](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E4%B8%BA%E4%BD%95%E4%BD%BF%E7%94%A8-docker) 为何使用 Docker

「通过 Docker，开发者可以使用任何语言任何工具创建任何应用。“Dockerized” 的应用是完全可移植的，能在任何地方运行 - 不管是同事的 OS X 和 Windows 笔记本，或是在云端运行的 Ubuntu QA 服务，还是在虚拟机运行的 Red Hat 产品数据中心。

Docker Hub 上有 13000+ 的应用，开发者可以从中选取一个进行快速扩展开发。Docker 跟踪管理变更和依赖关系，让系统管理员能更容易理解开发人员是如何让应用运转起来的。而开发者可以通过 Docker Hub 的共有/私有仓库，构建他们的自动化编译，与其他合作者共享成果。

Docker 帮助开发者更快地构建和发布高质量的应用。」—— [什么是 Docker (opens new window)](https://www.docker.com/what-docker/#copy1)

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E8%BF%90%E7%BB%B4) 运维

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%89%E8%A3%85) 安装

Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。

Docker CE 的安装请参考官方文档。

-   [Mac (opens new window)](https://docs.docker.com/docker-for-mac/install/)
-   [Windows (opens new window)](https://docs.docker.com/docker-for-windows/install/)
-   [Ubuntu (opens new window)](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
-   [Debian (opens new window)](https://docs.docker.com/install/linux/docker-ce/debian/)
-   [CentOS (opens new window)](https://docs.docker.com/install/linux/docker-ce/centos/)
-   [Fedora (opens new window)](https://docs.docker.com/install/linux/docker-ce/fedora/)
-   [其他 Linux 发行版 (opens new window)](https://docs.docker.com/install/linux/docker-ce/binaries/)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%A3%80%E6%9F%A5%E7%89%88%E6%9C%AC) 检查版本

[`docker version` (opens new window)](https://docs.docker.com/engine/reference/commandline/version/) 查看你正在运行的 Docker 版本。

获取 Docker 服务版本：

你也可以输出原始的 JSON 数据：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#docker-%E5%8A%A0%E9%80%9F) Docker 加速

国内访问 Docker Hub 很慢，所以，推荐配置 Docker 镜像仓库来提速。

镜像仓库清单：

| 镜像仓库 | 镜像仓库地址 | 说明 |
| --- | --- | --- |
| [DaoCloud 镜像站 (opens new window)](https://daocloud.io/mirror) | `http://f1361db2.m.daocloud.io` | 开发者需要开通 DaoCloud 账户，然后可以得到专属加速器。 |
| [阿里云 (opens new window)](https://cr.console.aliyun.com/) | `https://yourcode.mirror.aliyuncs.com` | 开发者需要开通阿里开发者帐户，再使用阿里的加速服务。登录后阿里开发者帐户后，`https://cr.console.aliyun.com/undefined/instances/mirrors` 中查看你的您的专属加速器地址。 |
| [网易云 (opens new window)](https://c.163yun.com/hub) | `https://hub-mirror.c.163.com` | 直接配置即可，亲测较为稳定。 |

配置镜像仓库方法（以 CentOS 为例）：

> 下面的示例为在 CentOS 环境中，指定镜像仓库为 `https://hub-mirror.c.163.com`

（1）修改配置文件

修改 `/etc/docker/daemon.json` ，如果不存在则新建。执行以下 Shell：

重启 docker 以生效：

执行 `docker info` 命令，查看 `Registry Mirrors` 是否已被改为 `https://hub-mirror.c.163.com` ，如果是，则表示配置成功。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%B9%E5%99%A8-container) 容器(Container)

[关于 Docker 进程隔离的基础 (opens new window)](http://etherealmind.com/basics-docker-containers-hypervisors-coreos/)。容器 (Container) 之于虚拟机 (Virtual Machine) 就好比线程之于进程。或者你可以把他们想成是「吃了类固醇的 chroots」。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F) 生命周期

-   [`docker create` (opens new window)](https://docs.docker.com/engine/reference/commandline/create) 创建容器但不启动它。
-   [`docker rename` (opens new window)](https://docs.docker.com/engine/reference/commandline/rename/) 用于重命名容器。
-   [`docker run` (opens new window)](https://docs.docker.com/engine/reference/commandline/run) 一键创建并同时启动该容器。
-   [`docker rm` (opens new window)](https://docs.docker.com/engine/reference/commandline/rm) 删除容器。
    -   如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。
-   [`docker update` (opens new window)](https://docs.docker.com/engine/reference/commandline/update/) 调整容器的资源限制。
-   清理掉所有处于终止状态的容器。

通常情况下，不使用任何命令行选项启动一个容器，该容器将会立即启动并停止。若需保持其运行，你可以使用 `docker run -td container_id` 命令。选项 `-t` 表示分配一个 pseudo-TTY 会话，`-d` 表示自动将容器与终端分离（也就是说在后台运行容器，并输出容器 ID）。

如果你需要一个临时容器，可使用 `docker run --rm` 会在容器停止之后删除它。

如果你需要映射宿主机 (host) 的目录到 Docker 容器内，可使用 `docker run -v $HOSTDIR:$DOCKERDIR`。详见 [卷标(Volumes) (opens new window)](https://github.com/wsargent/docker-cheat-sheet/tree/master/zh-cn#%E5%8D%B7%E6%A0%87volumes) 一节。

如果你想同时删除与容器相关联的卷标，那么在删除容器的时候必须包含 `-v` 选项，像这样 `docker rm -v`。

从 Docker 1.10 起，其内置一套各容器独立的 [日志引擎 (opens new window)](https://docs.docker.com/engine/admin/logging/overview/)，每个容器可以独立使用。你可以使用 `docker run --log-driver=syslog` 来自定义日志引擎（例如以上的 `syslog`）。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%90%AF%E5%8A%A8%E5%92%8C%E5%81%9C%E6%AD%A2) 启动和停止

-   [`docker start` (opens new window)](https://docs.docker.com/engine/reference/commandline/start) 启动已存在的容器。
-   [`docker stop` (opens new window)](https://docs.docker.com/engine/reference/commandline/stop) 停止运行中的容器。
-   [`docker restart` (opens new window)](https://docs.docker.com/engine/reference/commandline/restart) 重启容器。
-   [`docker pause` (opens new window)](https://docs.docker.com/engine/reference/commandline/pause/) 暂停运行中的容器，将其「冻结」在当前状态。
-   [`docker unpause` (opens new window)](https://docs.docker.com/engine/reference/commandline/unpause/) 结束容器暂停状态。
-   [`docker wait` (opens new window)](https://docs.docker.com/engine/reference/commandline/wait) 阻塞地等待某个运行中的容器直到停止。
-   [`docker kill` (opens new window)](https://docs.docker.com/engine/reference/commandline/kill) 向运行中的容器发送 SIGKILL 指令。
-   [`docker attach` (opens new window)](https://docs.docker.com/engine/reference/commandline/attach) 连接到运行中的容器。

如果你想将容器的端口 (ports) 暴露至宿主机，请见 [暴露端口 (opens new window)](https://github.com/wsargent/docker-cheat-sheet/tree/master/zh-cn#%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3exposing-ports) 一节。

关于 Docker 实例崩溃后的重启策略，详见 [本文 (opens new window)](http://container42.com/2014/09/30/docker-restart-policies/)。

#### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#cpu-%E9%99%90%E5%88%B6) CPU 限制

你可以限制 CPU 资源占用，无论是指定百分比，或是特定核心数。

例如，你可以设置 [`cpu-shares` (opens new window)](https://docs.docker.com/engine/reference/run/#/cpu-share-constraint)。该配置看起来有点奇怪 -- 1024 表示 100% CPU，因此如果你希望容器使用所有 CPU 内核的 50%，应将其设置为 512：

更多信息请参阅 https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/#\_cpu。

通过 [`cpuset-cpus` (opens new window)](https://docs.docker.com/engine/reference/run/#/cpuset-constraint) 可使用特定 CPU 内核。

请参阅 https://agileek.github.io/docker/2014/08/06/docker-cpuset/ 获取更多细节以及一些不错的视频。

注意，Docker 在容器内仍然能够 **看到** 全部 CPU -- 它仅仅是不使用全部而已。请参阅 https://github.com/docker/docker/issues/20770 获取更多细节。

#### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6) 内存限制

同样，亦可给 Docker 设置 [内存限制 (opens new window)](https://docs.docker.com/engine/reference/run/#/user-memory-constraints)：

#### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E8%83%BD%E5%8A%9B-capabilities) 能力(Capabilities)

Linux 的 Capability 可以通过使用 `cap-add` 和 `cap-drop` 设置。请参阅 https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities 获取更多细节。这有助于提高安全性。

如需要挂载基于 FUSE 的文件系统，你需要结合 `--cap-add` 和 `--device` 使用：

授予对某个设备的访问权限：

授予对所有设备的访问权限：

有关容器特权的更多信息请参阅 [本文 (opens new window)](https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E4%BF%A1%E6%81%AF) 信息

-   [`docker ps` (opens new window)](https://docs.docker.com/engine/reference/commandline/ps) 查看运行中的所有容器。
-   [`docker logs` (opens new window)](https://docs.docker.com/engine/reference/commandline/logs) 从容器中读取日志。（你也可以使用自定义日志驱动，不过在 1.10 中，它只支持 `json-file` 和 `journald`）。
-   [`docker inspect` (opens new window)](https://docs.docker.com/engine/reference/commandline/inspect) 查看某个容器的所有信息（包括 IP 地址）。
-   [`docker events` (opens new window)](https://docs.docker.com/engine/reference/commandline/events) 从容器中获取事件 (events)。
-   [`docker port` (opens new window)](https://docs.docker.com/engine/reference/commandline/port) 查看容器的公开端口。
-   [`docker top` (opens new window)](https://docs.docker.com/engine/reference/commandline/top) 查看容器中活动进程。
-   [`docker stats` (opens new window)](https://docs.docker.com/engine/reference/commandline/stats) 查看容器的资源使用量统计信息。
-   [`docker diff` (opens new window)](https://docs.docker.com/engine/reference/commandline/diff) 查看容器文件系统中存在改动的文件。

`docker ps -a` 将显示所有容器，包括运行中和已停止的。

`docker stats --all` 同样将显示所有容器，默认仅显示运行中的容器。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AF%BC%E5%85%A5-%E5%AF%BC%E5%87%BA) 导入 / 导出

-   [`docker cp` (opens new window)](https://docs.docker.com/engine/reference/commandline/cp) 在容器和本地文件系统之间复制文件或目录。
-   [`docker export` (opens new window)](https://docs.docker.com/engine/reference/commandline/export) 将容器的文件系统打包为归档文件流 (tarball archive stream) 并输出至标准输出 (STDOUT)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%89%A7%E8%A1%8C%E5%91%BD%E4%BB%A4) 执行命令

-   [`docker exec` (opens new window)](https://docs.docker.com/engine/reference/commandline/exec) 在容器内执行命令。

例如，进入正在运行的 `foo` 容器，并连接 (attach) 到一个新的 Shell 进程：`docker exec -it foo /bin/bash`。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E9%95%9C%E5%83%8F-images) 镜像(Images)

镜像是 [Docker 容器的模板 (opens new window)](https://docs.docker.com/engine/understanding-docker/#how-does-a-docker-image-work)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F-2) 生命周期

-   [`docker images` (opens new window)](https://docs.docker.com/engine/reference/commandline/images) 查看所有镜像。
-   [`docker import` (opens new window)](https://docs.docker.com/engine/reference/commandline/import) 从归档文件创建镜像。
-   [`docker build` (opens new window)](https://docs.docker.com/engine/reference/commandline/build) 从 Dockerfile 创建镜像。
-   [`docker commit` (opens new window)](https://docs.docker.com/engine/reference/commandline/commit) 为容器创建镜像，如果容器正在运行则会临时暂停。
-   [`docker rmi` (opens new window)](https://docs.docker.com/engine/reference/commandline/rmi) 删除镜像。
-   [`docker load` (opens new window)](https://docs.docker.com/engine/reference/commandline/load) 从标准输入 (STDIN) 加载归档包 (tar archive) 作为镜像，包括镜像本身和标签 (tags, 0.7 起)。
-   [`docker save` (opens new window)](https://docs.docker.com/engine/reference/commandline/save) 将镜像打包为归档包，并输出至标准输出 (STDOUT)，包括所有的父层、标签和版本 (parent layers, tags, versions, 0.7 起)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%85%B6%E5%AE%83%E4%BF%A1%E6%81%AF) 其它信息

-   [`docker history` (opens new window)](https://docs.docker.com/engine/reference/commandline/history) 查看镜像的历史记录。
-   [`docker tag` (opens new window)](https://docs.docker.com/engine/reference/commandline/tag) 给镜像打标签命名（本地或者仓库均可）。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%B8%85%E7%90%86) 清理

虽然你可以用 `docker rmi` 命令来删除指定的镜像，不过有个名为 [docker-gc (opens new window)](https://github.com/spotify/docker-gc) 的工具，它可以以一种安全的方式，清理掉那些不再被任何容器使用的镜像。Docker 1.13 起，使用 `docker image prune` 亦可删除未使用的镜像。参见 [清理 (opens new window)](https://github.com/wsargent/docker-cheat-sheet/tree/master/zh-cn#%E6%B8%85%E7%90%86)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8A%A0%E8%BD%BD-%E4%BF%9D%E5%AD%98%E9%95%9C%E5%83%8F) 加载 / 保存镜像

从文件中加载镜像：

保存既有镜像：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AF%BC%E5%85%A5-%E5%AF%BC%E5%87%BA%E5%AE%B9%E5%99%A8) 导入 / 导出容器

从文件中导入容器镜像：

导出既有容器：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8A%A0%E8%BD%BD%E5%B7%B2%E4%BF%9D%E5%AD%98%E7%9A%84%E9%95%9C%E5%83%8F-%E4%B8%8E-%E5%AF%BC%E5%85%A5%E5%B7%B2%E5%AF%BC%E5%87%BA%E4%B8%BA%E9%95%9C%E5%83%8F%E7%9A%84%E5%AE%B9%E5%99%A8-%E7%9A%84%E4%B8%8D%E5%90%8C) 加载已保存的镜像 与 导入已导出为镜像的容器 的不同

通过 `load` 命令来加载镜像，会创建一个新的镜像，并继承原镜像的所有历史。 通过 `import` 将容器作为镜像导入，也会创建一个新的镜像，但并不包含原镜像的历史，因此会比使用 `load` 方式生成的镜像更小。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%BD%91%E7%BB%9C-networks) 网络(Networks)

Docker 具备 [网络 (opens new window)](https://docs.docker.com/engine/userguide/networking/) 功能。我并不是很了解它，所以这是一个扩展本文的好地方。文档 [使用网络 (opens new window)](https://docs.docker.com/engine/userguide/networking/work-with-networks/) 指出，这是一种无需暴露端口即可实现 Docker 容器间通信的好方法。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F-3) 生命周期

-   [`docker network create` (opens new window)](https://docs.docker.com/engine/reference/commandline/network_create/)
-   [`docker network rm` (opens new window)](https://docs.docker.com/engine/reference/commandline/network_rm/)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%85%B6%E5%AE%83%E4%BF%A1%E6%81%AF-2) 其它信息

-   [`docker network ls` (opens new window)](https://docs.docker.com/engine/reference/commandline/network_ls/)
-   [`docker network inspect` (opens new window)](https://docs.docker.com/engine/reference/commandline/network_inspect/)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%BB%BA%E7%AB%8B%E8%BF%9E%E6%8E%A5) 建立连接

-   [`docker network connect` (opens new window)](https://docs.docker.com/engine/reference/commandline/network_connect/)
-   [`docker network disconnect` (opens new window)](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

你可以 [为容器指定 IP 地址 (opens new window)](https://blog.jessfraz.com/post/ips-for-all-the-things/)：

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3-exposing-ports) 暴露端口(Exposing ports)

通过宿主容器暴露输入端口相当 [繁琐但有效的 (opens new window)](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)。

例如使用 `-p` 将容器端口映射到宿主端口上（只使用本地主机 (localhost) 接口）：

你可以使用 [EXPOSE (opens new window)](https://docs.docker.com/engine/reference/builder/#expose) 告知 Docker，该容器在运行时监听指定的端口：

但是注意 EXPOSE 并不会直接暴露端口，你需要用参数 `-p` 。比如说你要在 localhost 上暴露容器的端口:

如果你是在 Virtualbox 中运行 Docker，那么你需要配置端口转发 (forward the port)。使用 [forwarded\_port (opens new window)](https://docs.vagrantup.com/v2/networking/forwarded_ports.html) 在 Vagrantfile 上配置暴露的端口范围，这样你就可以动态地映射了：

如果你忘记了将什么端口映射到宿主机上的话，可使用 `docker port` 查看：

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E4%BB%93%E7%AE%A1%E4%B8%AD%E5%BF%83%E5%92%8C%E4%BB%93%E5%BA%93-registry-repository) 仓管中心和仓库(Registry & Repository)

仓库 (repository) 是 _被托管(hosted)_ 的已命名镜像 (tagged images) 的集合，这组镜像用于构建容器文件系统。

仓管中心 (registry) 则是 _托管服务(host)_ -- 用于存储仓库并提供 HTTP API，以便 [管理仓库的上传和下载 (opens new window)](https://docs.docker.com/engine/tutorials/dockerrepos/)。

Docker 官方托管着自己的 [仓管中心 (opens new window)](https://hub.docker.com/)，包含着数量众多的仓库。不过话虽如此，这个仓管中心 [并没有很好地验证镜像 (opens new window)](https://titanous.com/posts/docker-insecurity)，所以如果你担心安全问题的话，请尽量避免使用它。

-   [`docker login` (opens new window)](https://docs.docker.com/engine/reference/commandline/login) 登入仓管中心。
-   [`docker logout` (opens new window)](https://docs.docker.com/engine/reference/commandline/logout) 登出仓管中心。
-   [`docker search` (opens new window)](https://docs.docker.com/engine/reference/commandline/search) 从仓管中心检索镜像。
-   [`docker pull` (opens new window)](https://docs.docker.com/engine/reference/commandline/pull) 从仓管中心拉取镜像到本地。
-   [`docker push` (opens new window)](https://docs.docker.com/engine/reference/commandline/push) 从本地推送镜像到仓管中心。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9C%AC%E5%9C%B0%E4%BB%93%E7%AE%A1%E4%B8%AD%E5%BF%83) 本地仓管中心

你可以使用 [docker distribution (opens new window)](https://github.com/docker/distribution) 项目搭建本地的仓管中心，详情参阅 [本地发布 (local deploy) (opens new window)](https://github.com/docker/docker.github.io/blob/master/registry/deploying.md) 的介绍。

科学上网后，也可以看看 [Google+ Group (opens new window)](https://groups.google.com/a/dockerproject.org/forum/#!forum/distribution)。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#dockerfile) Dockerfile

当你执行 `docker build` 时，Docker 将会根据 [配置文件 (opens new window)](https://docs.docker.com/engine/reference/builder/) 启动 Docker 容器。远优于使用 `docker commit`。

以下是一些编写 Dockerfile 的常用编辑器，并链接到适配的语法高亮模块︰

-   如果你在使用 [jEdit (opens new window)](http://jedit.org/)，你可以使用我开发的 Dockerfile [语法高亮模块 (opens new window)](https://github.com/wsargent/jedit-docker-mode)。
-   \[Sublime Text 2\](https://packagecontrol.io/packages/Dockerfile Syntax Highlighting)
-   [Atom (opens new window)](https://atom.io/packages/language-docker)
-   [Vim (opens new window)](https://github.com/ekalinin/Dockerfile.vim)
-   [Emacs (opens new window)](https://github.com/spotify/dockerfile-mode)
-   [TextMate (opens new window)](https://github.com/docker/docker/tree/master/contrib/syntax/textmate)
-   更多信息请参阅 [Docker 遇上 IDE (opens new window)](https://domeide.github.io/)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%8C%87%E4%BB%A4) 指令

-   [.dockerignore (opens new window)](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
-   [FROM (opens new window)](https://docs.docker.com/engine/reference/builder/#from) 为其他指令设置基础镜像 (Base Image)。
-   [MAINTAINER (deprecated - use LABEL instead) (opens new window)](https://docs.docker.com/engine/reference/builder/#maintainer-deprecated) 为生成的镜像设置作者字段。
-   [RUN (opens new window)](https://docs.docker.com/engine/reference/builder/#run) 在当前镜像的基础上生成一个新层并执行命令。
-   [CMD (opens new window)](https://docs.docker.com/engine/reference/builder/#cmd) 设置容器默认执行命令。
-   [EXPOSE (opens new window)](https://docs.docker.com/engine/reference/builder/#expose) 告知 Docker 容器在运行时所要监听的网络端口。注意：并没有实际上将端口设置为可访问。
-   [ENV (opens new window)](https://docs.docker.com/engine/reference/builder/#env) 设置环境变量。
-   [ADD (opens new window)](https://docs.docker.com/engine/reference/builder/#add) 将文件、目录或远程文件复制到容器中。缓存无效。请尽量用 `COPY` 代替 `ADD`。
-   [COPY (opens new window)](https://docs.docker.com/engine/reference/builder/#copy) 将文件或文件夹复制到容器中。注意：将使用 ROOT 用户复制文件，故无论 USER / WORKDIR 指令如何配置，你都需要手动修改其所有者（`chown`），`ADD` 也是一样。
-   [ENTRYPOINT (opens new window)](https://docs.docker.com/engine/reference/builder/#entrypoint) 将容器设为可执行的。
-   [VOLUME (opens new window)](https://docs.docker.com/engine/reference/builder/#volume) 在容器内部创建挂载点 (mount point) 指向外部挂载的卷标或其他容器。
-   [USER (opens new window)](https://docs.docker.com/engine/reference/builder/#user) 设置随后执行 RUN / CMD / ENTRYPOINT 命令的用户名。
-   [WORKDIR (opens new window)](https://docs.docker.com/engine/reference/builder/#workdir) 设置工作目录 (working directory)。
-   [ARG (opens new window)](https://docs.docker.com/engine/reference/builder/#arg) 定义编译时 (build-time) 变量。
-   [ONBUILD (opens new window)](https://docs.docker.com/engine/reference/builder/#onbuild) 添加触发指令，当该镜像被作为其他镜像的基础镜像时该指令会被触发。
-   [STOPSIGNAL (opens new window)](https://docs.docker.com/engine/reference/builder/#stopsignal) 设置停止容器时，向容器内发送的系统调用信号 (system call signal)。
-   [LABEL (opens new window)](https://docs.docker.com/config/labels-custom-metadata/) 将键值对元数据 (key/value metadata) 应用到镜像、容器或是守护进程。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%95%99%E7%A8%8B) 教程

-   [Flux7's Dockerfile Tutorial (opens new window)](http://flux7.com/blogs/docker/docker-tutorial-series-part-3-automation-is-the-word-using-dockerfile/)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E4%BE%8B%E5%AD%90) 例子

-   [Examples (opens new window)](https://docs.docker.com/engine/reference/builder/#dockerfile-examples)
-   [Best practices for writing Dockerfiles (opens new window)](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
-   [Michael Crosby (opens new window)](http://crosbymichael.com/) 还有更多的 [Dockerfiles best practices (opens new window)](http://crosbymichael.com/dockerfile-best-practices.html) / [take 2 (opens new window)](http://crosbymichael.com/dockerfile-best-practices-take-2.html)
-   [Building Good Docker Images (opens new window)](http://jonathan.bergknoff.com/journal/building-good-docker-images) / [Building Better Docker Images (opens new window)](http://jonathan.bergknoff.com/journal/building-better-docker-images)
-   [Managing Container Configuration with Metadata (opens new window)](https://speakerdeck.com/garethr/managing-container-configuration-with-metadata)

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%B1%82-layers) 层(Layers)

Docker 的版本化文件系统是基于层的。就像 [Git 的提交或文件变更系统 (opens new window)](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/) 一样。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E9%93%BE%E6%8E%A5-links) 链接(Links)

链接 (links) [通过 TCP/IP 端口 (opens new window)](https://docs.docker.com/userguide/dockerlinks/) 实现 Docker 容器之间的通讯。[Atlassian (opens new window)](https://blogs.atlassian.com/2013/11/docker-all-the-things-at-atlassian-automation-and-wiring/) 展示了可用的例子。你还可以 [通过主机名 (hostname) 链接 (opens new window)](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/#/updating-the-etchosts-file)。

在某种意义上来说，该特性已经被 [自定义网络 (opens new window)](https://docs.docker.com/network/) 所替代。

注意: 如果你希望容器之间**只**通过链接进行通讯，在启动 Docker 守护进程时，请使用 `-icc=false` 来禁用内部进程通讯。

假设你有一个名为 CONTAINER 的容器（通过 `docker run --name CONTAINER` 指定）并且在 Dockerfile 中，暴露了一个端口:

然后，我们创建另外一个名为 LINKED 的容器:

然后 CONTAINER 暴露的端口和别名将会以如下的环境变量出现在 LINKED 中:

那么你便可以通过这种方式来连接它了。

使用 `docker rm --link` 即可删除链接。

通常，Docker 容器（亦可理解为「服务」）之间的链接，是「服务发现」的一个子集。如果你打算在生产中大规模使用 Docker，这将是一个很大的问题。请参阅[The Docker Ecosystem: Service Discovery and Distributed Configuration Stores (opens new window)](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-service-discovery-and-distributed-configuration-stores) 获取更多信息。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8D%B7%E6%A0%87-volumes-%E5%92%8C%E6%8C%82%E8%BD%BD) 卷标(Volumes)和挂载

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8D%B7%E6%A0%87) 卷标

Docker 的卷标 (volumes) 是 [独立的文件系统 (opens new window)](https://docs.docker.com/engine/tutorials/dockervolumes/)。它们并非必须连接到特定的容器上。

`数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

-   `数据卷` 可以在容器之间共享和重用
-   对 `数据卷` 的修改会立马生效
-   对 `数据卷` 的更新，不会影响镜像
-   `数据卷` 默认会一直存在，即使容器被删除

卷标相关命令：

-   [`docker volume create` (opens new window)](https://docs.docker.com/engine/reference/commandline/volume_create/) - 创建卷标
    
-   [`docker volume rm` (opens new window)](https://docs.docker.com/engine/reference/commandline/volume_rm/) - 删除卷标
    
-   [`docker volume ls` (opens new window)](https://docs.docker.com/engine/reference/commandline/volume_ls/) - 查看卷标
    
-   [`docker volume inspect` (opens new window)](https://docs.docker.com/engine/reference/commandline/volume_inspect/) - 查看数据卷的具体信息
    
-   [`docker volume prune` (opens new window)](https://docs.docker.com/engine/reference/commandline/volume_prune/) - 清理无主的数据卷
    

卷标在不能使用链接（只有 TCP/IP）的情况下非常有用。例如，如果你有两个 Docker 实例需要通讯并在文件系统上留下记录。

你可以一次性将其挂载到多个 docker 容器上，通过 `docker run --volumes-from`。

因为卷标是独立的文件系统，它们通常被用于存储各容器之间的瞬时状态。也就是说，你可以配置一个无状态临时容器，关掉之后，当你有第二个这种临时容器实例的时候，你可以从上一次保存的状态继续执行。

查看 [卷标进阶 (opens new window)](http://crosbymichael.com/advanced-docker-volumes.html) 来获取更多细节。[Container42 (opens new window)](http://container42.com/2014/11/03/docker-indepth-volumes/) 非常有用。

你可以 [将宿主 MacOS 的文件夹映射为 Docker 卷标 (opens new window)](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume)：

你也可以用远程 NFS 卷标，如果你觉得你 [有足够勇气 (opens new window)](https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-shared-storage-volume-as-a-data-volume)。

还可以考虑运行一个纯数据容器，像 [这里 (opens new window)](http://container42.com/2013/12/16/persistent-volumes-with-docker-container-as-volume-pattern/) 所说的那样，提供可移植数据。

记得，[文件也可以被挂载为卷标 (opens new window)](https://github.com/wsargent/docker-cheat-sheet/tree/master/zh-cn#%E5%B0%86%E6%96%87%E4%BB%B6%E6%8C%82%E8%BD%BD%E4%B8%BA%E5%8D%B7%E6%A0%87)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%8C%82%E8%BD%BD) 挂载

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

在用 `docker run` 命令的时候，使用 `--mount` 标记来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5) 最佳实践

这里有一些最佳实践，以及争论焦点：

-   [The Rabbit Hole of Using Docker in Automated Tests (opens new window)](http://gregoryszorc.com/blog/2014/10/16/the-rabbit-hole-of-using-docker-in-automated-tests/)
-   [Bridget Kromhout (opens new window)](https://twitter.com/bridgetkromhout) has a useful blog post on [running Docker in production (opens new window)](http://sysadvent.blogspot.co.uk/2014/12/day-1-docker-in-production-reality-not.html) at Dramafever.
-   There's also a best practices [blog post (opens new window)](http://developers.lyst.com/devops/2014/12/08/docker/) from Lyst.
-   [A Docker Dev Environment in 24 Hours! (opens new window)](https://engineering.salesforceiq.com/2013/11/05/a-docker-dev-environment-in-24-hours-part-2-of-2.html)
-   [Building a Development Environment With Docker (opens new window)](https://tersesystems.com/2013/11/20/building-a-development-environment-with-docker/)
-   [Discourse in a Docker Container (opens new window)](https://samsaffron.com/archive/2013/11/07/discourse-in-a-docker-container)

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%89%E5%85%A8-security) 安全(Security)

这节准备讨论一些关于 Docker 安全性的问题。Docker 官方文档 [安全 (opens new window)](https://docs.docker.com/articles/security/) 页面讲述了更多细节。

首先第一件事：Docker 是有 root 权限的。如果你在 `docker` 组，那么你就有 [root 权限 (opens new window)](https://web.archive.org/web/20161226211755/http://reventlov.com/advisories/using-the-docker-command-to-root-the-host)。如果你将 Docker 的 Unix Socket 暴露给容器，意味着你赋予了容器 [宿主机 root 权限 (opens new window)](https://www.lvh.io/posts/dont-expose-the-docker-socket-not-even-to-a-container.html)。

Docker 不应当作为唯一的防御措施。你应当使其更加安全可靠。

为了更好地理解容器暴露了什么，可参阅由 [Aaron Grattafiori (opens new window)](https://twitter.com/dyn___) 编写的 [Understanding and Hardening Linux Containers (opens new window)](https://www.nccgroup.trust/globalassets/our-research/us/whitepapers/2016/april/ncc_group_understanding_hardening_linux_containers-1-1.pdf)。这是一个完整全面且包含大量链接和脚注的容器问题指南，介绍了许多有用的内容。即使你已经加固过容器，以下的安全提示依然十分有帮助，但并不能代替理解的过程。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%89%E5%85%A8%E6%8F%90%E7%A4%BA) 安全提示

为了最大的安全性，你应当考虑在虚拟机上运行 Docker。这是直接从 Docker 安全团队拿来的资料 -- [slides (opens new window)](http://www.slideshare.net/jpetazzo/linux-containers-lxc-docker-and-security) / [notes (opens new window)](http://www.projectatomic.io/blog/2014/08/is-it-safe-a-look-at-docker-and-security-from-linuxcon/)。之后，可使用 AppArmor、seccomp、SELinux、grsec 等来 [限制容器的权限 (opens new window)](http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers/)。更多细节，请查阅 [Docker 1.10 security features (opens new window)](https://blog.docker.com/2016/02/docker-engine-1-10-security/)。

Docker 镜像 ID 属于 [敏感信息 (opens new window)](https://medium.com/@quayio/your-docker-image-ids-are-secrets-and-its-time-you-treated-them-that-way-f55e9f14c1a4) 所以它不应该向外界公开。请将它们当作密码来对待。

阅读由 [Thomas Sjögren (opens new window)](https://github.com/konstruktoid) 编写的 [Docker Security Cheat Sheet (opens new window)](https://github.com/konstruktoid/Docker/blob/master/Security/CheatSheet.adoc)：关于加固容器的不错的建议。

查看 [Docker 安全测试脚本 (opens new window)](https://github.com/docker/docker-bench-security)，下载 [最佳实践白皮书 (opens new window)](https://blog.docker.com/2015/05/understanding-docker-security-and-best-practices/)。

你应当远离使用非稳定版本 grsecurity / pax 的内核，比如 [Alpine Linux (opens new window)](https://en.wikipedia.org/wiki/Alpine_Linux)。如果在产品中用了 grsecurity，那么你应该考虑使用有 [商业支持 (opens new window)](https://grsecurity.net/business_support.php) 的 [稳定版本 (opens new window)](https://grsecurity.net/announce.php)，就像你对待 RedHat 那样。虽然要 $200 每月，但对于你的运维预算来说不值一提。

从 Docker 1.11 开始，你可以轻松的限制在容器中可用的进程数，以防止 fork 炸弹。 这要求 Linux 内核 >= 4.3，并且要在内核配置中打开 CGROUP\_PIDS=y。

同时，你也可以限制进程再获取新权限。该功能是 Linux 内核从 3.5 版本开始就拥有的。你可以从 [这篇博客 (opens new window)](http://www.projectatomic.io/blog/2016/03/no-new-privs-docker/) 中阅读到更多关于这方面的内容。

以下内容摘选自 [Container Solutions (opens new window)](http://container-solutions.com/is-docker-safe-for-production/) 的 [Docker Security Cheat Sheet (opens new window)](http://container-solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf)（PDF 版本，难以使用，故复制至此）：

关闭内部进程通讯：

设置容器为只读：

通过 hashsum 来验证卷标：

设置卷标为只读：

在 Dockerfile 中定义用户并以该用户运行，避免在容器中以 ROOT 身份操作：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%94%A8%E6%88%B7%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4-user-namespaces) 用户命名空间(User Namespaces)

还可以通过使用 [用户命名空间 (opens new window)](https://s3hh.wordpress.com/2013/07/19/creating-and-using-containers-without-privilege/) -- 自 1.10 版本起已内置，但默认并未启用。

要在 Ubuntu 15.10 中启用用户命名空间 (remap the userns)，请 [跟着这篇博客的例子 (opens new window)](https://raesene.github.io/blog/2016/02/04/Docker-User-Namespaces/) 来做。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%89%E5%85%A8%E7%9B%B8%E5%85%B3%E8%A7%86%E9%A2%91) 安全相关视频

-   [Using Docker Safely (opens new window)](https://youtu.be/04LOuMgNj9U)
-   [Securing your applications using Docker (opens new window)](https://youtu.be/KmxOXmPhZbk)
-   [Container security: Do containers actually contain? (opens new window)](https://youtu.be/a9lE9Urr6AQ)
-   [Linux Containers: Future or Fantasy? (opens new window)](https://www.youtube.com/watch?v=iN6QbszB1R8)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%AE%89%E5%85%A8%E8%B7%AF%E7%BA%BF%E5%9B%BE) 安全路线图

Docker 的路线图提到关于 [seccomp 的支持 (opens new window)](https://github.com/docker/docker/blob/master/ROADMAP.md#11-security)。 一个名为 [bane (opens new window)](https://github.com/jfrazelle/bane) 的 AppArmor 策略生成器正在实现 [安全配置文件 (opens new window)](https://github.com/docker/docker/issues/17142)。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%B0%8F%E8%B4%B4%E5%A3%AB) 小贴士

链接：

-   [15 Docker Tips in 5 minutes (opens new window)](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes)
-   [CodeFresh Everyday Hacks Docker (opens new window)](https://codefresh.io/blog/everyday-hacks-docker/)

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%B8%85%E7%90%86-2) 清理

最新的 [数据管理命令 (opens new window)](https://github.com/docker/docker/pull/26108) 已在 Docker 1.13 实现：

-   `docker system prune`
-   `docker volume prune`
-   `docker network prune`
-   `docker container prune`
-   `docker image prune`

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#df-%E5%91%BD%E4%BB%A4) df 命令

`docker system df` 将显示当前 Docker 各部分占用的磁盘空间。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#heredoc-%E5%A3%B0%E6%98%8E-docker-%E5%AE%B9%E5%99%A8) Heredoc 声明 Docker 容器

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9C%80%E8%BF%91%E4%B8%80%E6%AC%A1%E7%9A%84%E5%AE%B9%E5%99%A8-id) 最近一次的容器 ID

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%B8%A6%E5%91%BD%E4%BB%A4%E7%9A%84%E6%8F%90%E4%BA%A4-%E9%9C%80%E8%A6%81-dockerfile) 带命令的提交（需要 Dockerfile）

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E8%8E%B7%E5%8F%96-ip-%E5%9C%B0%E5%9D%80) 获取 IP 地址

或使用 [jq (opens new window)](https://stedolan.github.io/jq/):

或使用 [go 模板 (opens new window)](https://docs.docker.com/engine/reference/commandline/inspect)：

或在通过 Dockerfile 构建镜像时，通过构建参数 (build argument) 传入：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E8%8E%B7%E5%8F%96%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84) 获取端口映射

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E9%80%9A%E8%BF%87%E6%AD%A3%E5%88%99%E5%8C%B9%E9%85%8D%E5%AE%B9%E5%99%A8) 通过正则匹配容器

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E8%8E%B7%E5%8F%96%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E9%85%8D%E7%BD%AE) 获取环境变量配置

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%BC%BA%E8%A1%8C%E7%BB%88%E6%AD%A2%E8%BF%90%E8%A1%8C%E4%B8%AD%E7%9A%84%E5%AE%B9%E5%99%A8) 强行终止运行中的容器

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%88%A0%E9%99%A4%E6%89%80%E6%9C%89%E5%AE%B9%E5%99%A8-%E5%BC%BA%E8%A1%8C%E5%88%A0%E9%99%A4-%E6%97%A0%E8%AE%BA%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E6%88%96%E5%81%9C%E6%AD%A2) 删除所有容器（强行删除！无论容器运行或停止）

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%88%A0%E9%99%A4%E6%97%A7%E5%AE%B9%E5%99%A8) 删除旧容器

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%88%A0%E9%99%A4%E5%B7%B2%E5%81%9C%E6%AD%A2%E7%9A%84%E5%AE%B9%E5%99%A8) 删除已停止的容器

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%81%9C%E6%AD%A2%E5%B9%B6%E5%88%A0%E9%99%A4%E5%AE%B9%E5%99%A8) 停止并删除容器

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%88%A0%E9%99%A4%E6%97%A0%E7%94%A8-dangling-%E7%9A%84%E9%95%9C%E5%83%8F) 删除无用 (dangling) 的镜像

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%88%A0%E9%99%A4%E6%89%80%E6%9C%89%E9%95%9C%E5%83%8F) 删除所有镜像

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%88%A0%E9%99%A4%E6%97%A0%E7%94%A8-dangling-%E7%9A%84%E5%8D%B7%E6%A0%87) 删除无用 (dangling) 的卷标

Docker 1.9 版本起：

1.9.0 中，参数 `dangling=false` 居然 _没_ 用 - 它会被忽略然后列出所有的卷标。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E6%9F%A5%E7%9C%8B%E9%95%9C%E5%83%8F%E4%BE%9D%E8%B5%96) 查看镜像依赖

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#docker-%E5%AE%B9%E5%99%A8%E7%98%A6%E8%BA%AB) Docker 容器瘦身

-   在某层 (RUN layer) 清理 APT

这应当和其他 apt 命令在同一层中完成。 否则，前面的层将会保持原有信息，而你的镜像则依旧臃肿。

-   压缩镜像

-   备份

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E7%9B%91%E8%A7%86%E8%BF%90%E8%A1%8C%E4%B8%AD%E5%AE%B9%E5%99%A8%E7%9A%84%E7%B3%BB%E7%BB%9F%E8%B5%84%E6%BA%90%E5%88%A9%E7%94%A8%E7%8E%87) 监视运行中容器的系统资源利用率

检查某个容器的 CPU、内存以及网络 I/O 使用情况，你可以：

按 ID 列出所有容器：

按名称列出所有容器：

按指定镜像名称列出所有容器：

删除所有未标签命名 (untagged) 的容器：

通过正则匹配删除指定容器：

删除所有已退出 (exited) 的容器：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%B0%86%E6%96%87%E4%BB%B6%E6%8C%82%E8%BD%BD%E4%B8%BA%E5%8D%B7%E6%A0%87) 将文件挂载为卷标

文件也可以被挂载为卷标。例如你可以仅仅注入单个配置文件：

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-cheat-sheet.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99) 参考资料

-   [Docker Cheat Sheet (opens new window)](https://github.com/wsargent/docker-cheat-sheet/tree/master/zh-cn)