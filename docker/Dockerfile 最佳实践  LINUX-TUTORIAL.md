-   [一、Dockerfile 指令](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E4%B8%80dockerfile-%E6%8C%87%E4%BB%A4)
    -   [FROM(指定基础镜像)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#from%E6%8C%87%E5%AE%9A%E5%9F%BA%E7%A1%80%E9%95%9C%E5%83%8F)
    -   [RUN(执行命令)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#run%E6%89%A7%E8%A1%8C%E5%91%BD%E4%BB%A4)
    -   [COPY(复制文件)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#copy%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6)
    -   [ADD(更高级的复制文件)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#add%E6%9B%B4%E9%AB%98%E7%BA%A7%E7%9A%84%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6)
    -   [CMD(容器启动命令)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#cmd%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E5%91%BD%E4%BB%A4)
    -   [ENTRYPOINT(入口点)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#entrypoint%E5%85%A5%E5%8F%A3%E7%82%B9)
    -   [ENV(设置环境变量)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#env%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
    -   [ARG(构建参数)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#arg%E6%9E%84%E5%BB%BA%E5%8F%82%E6%95%B0)
    -   [VOLUME(定义匿名卷)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#volume%E5%AE%9A%E4%B9%89%E5%8C%BF%E5%90%8D%E5%8D%B7)
    -   [EXPOSE(暴露端口)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#expose%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3)
    -   [WORKDIR(指定工作目录)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#workdir%E6%8C%87%E5%AE%9A%E5%B7%A5%E4%BD%9C%E7%9B%AE%E5%BD%95)
    -   [USER(指定当前用户)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#user%E6%8C%87%E5%AE%9A%E5%BD%93%E5%89%8D%E7%94%A8%E6%88%B7)
    -   [HEALTHCHECK(健康检查)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#healthcheck%E5%81%A5%E5%BA%B7%E6%A3%80%E6%9F%A5)
    -   [ONBUILD(为他人作嫁衣裳)](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#onbuild%E4%B8%BA%E4%BB%96%E4%BA%BA%E4%BD%9C%E5%AB%81%E8%A1%A3%E8%A3%B3)
-   [参考资料](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E4%B8%80%E3%80%81dockerfile-%E7%AE%80%E4%BB%8B) 一、Dockerfile 简介

Docker 镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E4%BD%BF%E7%94%A8-dockerfile-%E6%9E%84%E5%BB%BA%E9%95%9C%E5%83%8F) 使用 Dockerfile 构建镜像

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E4%BA%8C%E3%80%81dockerfile-%E6%8C%87%E4%BB%A4%E8%AF%A6%E8%A7%A3) 二、Dockerfile 指令详解

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#from-%E6%8C%87%E5%AE%9A%E5%9F%BA%E7%A1%80%E9%95%9C%E5%83%8F) FROM(指定基础镜像)

> 作用：**`FROM` 指令用于指定基础镜像**。

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 `nginx` 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 `FROM` 就是指定**基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

在 [Docker Store (opens new window)](https://store.docker.com/) 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如 [`nginx` (opens new window)](https://store.docker.com/images/nginx/)、[`redis` (opens new window)](https://store.docker.com/images/redis/)、[`mongo` (opens new window)](https://store.docker.com/images/mongo/)、[`mysql` (opens new window)](https://store.docker.com/images/mysql/)、[`httpd` (opens new window)](https://store.docker.com/images/httpd/)、[`php` (opens new window)](https://store.docker.com/images/php/)、[`tomcat` (opens new window)](https://store.docker.com/images/tomcat/) 等；也有一些方便开发、构建、运行各种语言应用的镜像，如 [`node` (opens new window)](https://store.docker.com/images/node)、[`openjdk` (opens new window)](https://store.docker.com/images/openjdk/)、[`python` (opens new window)](https://store.docker.com/images/python/)、[`ruby` (opens new window)](https://store.docker.com/images/ruby/)、[`golang` (opens new window)](https://store.docker.com/images/golang/) 等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如 [`ubuntu` (opens new window)](https://store.docker.com/images/ubuntu/)、[`debian` (opens new window)](https://store.docker.com/images/debian/)、[`centos` (opens new window)](https://store.docker.com/images/centos/)、[`fedora` (opens new window)](https://store.docker.com/images/fedora/)、[`alpine` (opens new window)](https://store.docker.com/images/alpine/) 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

如果你以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 [`swarm` (opens new window)](https://hub.docker.com/_/swarm/)、[`coreos/etcd` (opens new window)](https://quay.io/repository/coreos/etcd)。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 `FROM scratch` 会让镜像体积更加小巧。使用 [Go 语言 (opens new window)](https://golang.org/) 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#run-%E6%89%A7%E8%A1%8C%E5%91%BD%E4%BB%A4) RUN(执行命令)

> **`RUN` 指令是用来执行命令行命令的**。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种：
> 
> -   _shell_ 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 `RUN` 指令就是这种格式。
> 
> -   _exec_ 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

既然 `RUN` 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

之前说过，Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

_Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。_

上面的 `Dockerfile` 正确的写法应该是这样：

首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 `RUN` 对一一对应不同的命令，而是仅仅使用一个 `RUN` 指令，并使用 `&&` 将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 `\` 的命令换行方式，以及行首 `#` 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 `apt` 缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#copy-%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6) COPY(复制文件)

> **`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。**

格式：

-   `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
-   `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

示例：

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 [`filepath.Match` (opens new window)](https://golang.org/pkg/path/filepath/#Match) 规则，如：

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 `COPY` 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#add-%E6%9B%B4%E9%AB%98%E7%BA%A7%E7%9A%84%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6) ADD(更高级的复制文件)

> `ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。
> 
> 比如 `<源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>`去。下载后的文件权限自动设置为 `600`，如果这并不是想要的权限，那么还需要增加额外的一层 `RUN` 进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 `RUN` 指令进行解压缩。所以不如直接使用 `RUN` 指令，然后使用 `wget` 或者 `curl` 工具下载，处理权限、解压缩、然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。
> 
> 如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 `ubuntu` 中：

但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 `ADD` 命令了。

在 Docker 官方的 [Dockerfile 最佳实践文档 (opens new window)](https://yeasy.gitbooks.io/docker_practice/content/appendix/best_practices.html) 中要求，尽可能的使用 `COPY`，因为 `COPY` 的语义很明确，就是复制文件而已，而 `ADD` 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用 `ADD` 的场合，就是所提及的需要自动解压缩的场合。

另外需要注意的是，`ADD` 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。

因此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#cmd-%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E5%91%BD%E4%BB%A4) CMD(容器启动命令)

> 之前介绍容器的时候曾经说过，Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。`CMD` 指令就是用于指定默认的容器主进程的启动命令的。

`CMD` 指令的格式和 `RUN` 相似，也是两种格式：

-   `shell` 格式：`CMD <命令>`
-   `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
-   参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，`ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 `bash`。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 `exec` 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 `"`，而不要使用单引号。

如果使用 `shell` 格式的话，实际的命令会被包装为 `sh -c` 的参数的形式进行执行。比如：

在实际执行中，会将其变更为：

这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。

提到 `CMD` 就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出现的一个混淆。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

一些初学者将 `CMD` 写为：

然后发现容器执行后就立即退出了。甚至在容器内去使用 `systemctl` 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

而使用 `service nginx start` 命令，则是希望 upstart 来以后台守护进程形式启动 `nginx` 服务。而刚才说了 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 `sh`。那么当 `service nginx start` 命令结束后，`sh` 也就结束了，`sh` 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 `nginx` 可执行文件，并且要求以前台形式运行。比如：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#entrypoint-%E5%85%A5%E5%8F%A3%E7%82%B9) ENTRYPOINT(入口点)

`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式。

`ENTRYPOINT` 的目的和 `CMD` 一样，都是在指定容器启动程序及参数。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令，换句话说实际执行时，将变为：

那么有了 `CMD` 后，为什么还要有 `ENTRYPOINT` 呢？这种 `<ENTRYPOINT> "<CMD>"` 有什么好处么？让我们来看几个场景。

#### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E5%9C%BA%E6%99%AF%E4%B8%80-%E8%AE%A9%E9%95%9C%E5%83%8F%E5%8F%98%E6%88%90%E5%83%8F%E5%91%BD%E4%BB%A4%E4%B8%80%E6%A0%B7%E4%BD%BF%E7%94%A8) 场景一：让镜像变成像命令一样使用

假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 `CMD` 来实现：

假如我们使用 `docker build -t myip .` 来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：

嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的 `CMD` 中可以看到实质的命令是 `curl`，那么如果我们希望显示 HTTP 头信息，就需要加上 `-i` 参数。那么我们可以直接加 `-i` 参数给 `docker run myip` 么？

我们可以看到可执行文件找不到的报错，`executable file not found`。之前我们说过，跟在镜像名后面的是 `command`，运行时会替换 `CMD` 的默认值。因此这里的 `-i` 替换了原来的 `CMD`，而不是添加在原来的 `curl -s https://ip.cn` 后面。而 `-i` 根本不是命令，所以自然找不到。

那么如果我们希望加入 `-i` 这参数，我们就必须重新完整的输入这个命令：

这显然不是很好的解决方案，而使用 `ENTRYPOINT` 就可以解决这个问题。现在我们重新用 `ENTRYPOINT` 来实现这个镜像：

这次我们再来尝试直接使用 `docker run myip -i`：

可以看到，这次成功了。这是因为当存在 `ENTRYPOINT` 后，`CMD` 的内容将会作为参数传给 `ENTRYPOINT`，而这里 `-i` 就是新的 `CMD`，因此会作为参数传给 `curl`，从而达到了我们预期的效果。

#### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E5%9C%BA%E6%99%AF%E4%BA%8C-%E5%BA%94%E7%94%A8%E8%BF%90%E8%A1%8C%E5%89%8D%E7%9A%84%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C) 场景二：应用运行前的准备工作

启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。

比如 `mysql` 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。

此外，可能希望避免使用 `root` 用户去启动服务，从而提高安全性，而在启动服务前还需要以 `root` 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 `root` 身份执行，方便调试等。

这些准备工作是和容器 `CMD` 无关的，无论 `CMD` 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 `ENTRYPOINT` 中去执行，而这个脚本会将接到的参数（也就是 `<CMD>`）作为命令，在脚本最后执行。比如官方镜像 `redis` 中就是这么做的：

可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 `ENTRYPOINT` 为 `docker-entrypoint.sh` 脚本。

该脚本的内容就是根据 `CMD` 的内容来判断，如果是 `redis-server` 的话，则切换到 `redis` 用户身份启动服务器，否则依旧使用 `root` 身份执行。比如：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#env-%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F) ENV(设置环境变量)

> `ENV` 指令用于设置环境变量。无论是后面的其它指令，如 `RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量。

格式：

-   `ENV <key> <value>`
-   `ENV <key1>=<value1> <key2>=<value2>...`

示例 1：

这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。

示例 2：

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 `node` 镜像 `Dockerfile` 中，就有类似这样的代码：

在这里先定义了环境变量 `NODE_VERSION`，其后的 `RUN` 这层里，多次使用 `$NODE_VERSION` 来进行操作定制。可以看到，将来升级镜像构建版本的时候，只需要更新 `7.2.0` 即可，`Dockerfile` 构建维护变得更轻松了。

下列指令可以支持环境变量展开： `ADD`、`COPY`、`ENV`、`EXPOSE`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`。

可以从这个指令列表里感觉到，环境变量可以使用的地方很多，很强大。通过环境变量，我们可以让一份 `Dockerfile` 制作更多的镜像，只需使用不同的环境变量即可。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#arg-%E6%9E%84%E5%BB%BA%E5%8F%82%E6%95%B0) ARG(构建参数)

> `Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。
> 
> 构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。

格式：`ARG <参数名>[=<默认值>]`

在 1.13 之前的版本，要求 `--build-arg` 中的参数名，必须在 `Dockerfile` 中用 `ARG` 定义过了，换句话说，就是 `--build-arg` 指定的参数，必须在 `Dockerfile` 中使用了。如果对应参数没有被使用，则会报错退出构建。从 1.13 开始，这种严格的限制被放开，不再报错退出，而是显示警告信息，并继续构建。这对于使用 CI 系统，用同样的构建流程构建不同的 `Dockerfile` 的时候比较有帮助，避免构建命令必须根据每个 Dockerfile 的内容修改。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#volume-%E5%AE%9A%E4%B9%89%E5%8C%BF%E5%90%8D%E5%8D%B7) VOLUME(定义匿名卷)

格式：

-   `VOLUME ["<路径1>", "<路径2>"...]`
-   `VOLUME <路径>`

之前我们说过，容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，后面的章节我们会进一步介绍 Docker 卷的概念。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

这里的 `/data` 目录就会在运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。比如：

在这行命令中，就使用了 `mydata` 这个命名卷挂载到了 `/data` 这个位置，替代了 `Dockerfile` 中定义的匿名卷的挂载配置。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#expose-%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3) EXPOSE(暴露端口)

> `EXPOSE` 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。
> 
> 要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

格式：`EXPOSE <端口1> [<端口2>...]`。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#workdir-%E6%8C%87%E5%AE%9A%E5%B7%A5%E4%BD%9C%E7%9B%AE%E5%BD%95) WORKDIR(指定工作目录)

> 使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

格式：`WORKDIR <工作目录路径>`。

示例 1：

之前提到一些初学者常犯的错误是把 `Dockerfile` 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 `hello`。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 `Dockerfile` 中，这两行 `RUN` 命令的执行环境根本不同，是两个完全不同的容器。这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。

之前说过每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#label) LABEL

`LABEL`用于为镜像添加元数据，元数以键值对的形式指定：

使用`LABEL`指定元数据时，一条`LABEL`指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条`LABEL`指令指定，以免生成过多的中间镜像。

如，通过`LABEL`指定一些元数据：

指定后可以通过`docker inspect`查看：

_注意；_`Dockerfile`中还有个`MAINTAINER`命令，该命令用于指定镜像作者。但`MAINTAINER`并不推荐使用，更推荐使用`LABEL`来指定镜像作者。如：

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#user-%E6%8C%87%E5%AE%9A%E5%BD%93%E5%89%8D%E7%94%A8%E6%88%B7) USER(指定当前用户)

> `USER` 指令和 `WORKDIR` 相似，都是改变环境状态并影响以后的层。`WORKDIR` 是改变工作目录，`USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。
> 
> 当然，和 `WORKDIR` 一样，`USER` 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

格式：`USER <用户名>[:<用户组>]`

示例 1：

如果以 `root` 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 `su`或者 `sudo`，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 [`gosu` (opens new window)](https://github.com/tianon/gosu)。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#healthcheck-%E5%81%A5%E5%BA%B7%E6%A3%80%E6%9F%A5) HEALTHCHECK(健康检查)

格式：

-   `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
-   `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12 引入的新指令。

在没有 `HEALTHCHECK` 指令前，Docker 引擎只可以通过容器内主进程是否退出来判断容器是否状态异常。很多情况下这没问题，但是如果程序进入死锁状态，或者死循环状态，应用进程并不退出，但是该容器已经无法提供服务了。在 1.12 以前，Docker 不会检测到容器的这种状态，从而不会重新调度，导致可能会有部分容器已经无法提供服务了却还在接受用户请求。

而自 1.12 之后，Docker 提供了 `HEALTHCHECK` 指令，通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

当在一个镜像指定了 `HEALTHCHECK` 指令后，用其启动容器，初始状态会为 `starting`，在 `HEALTHCHECK` 指令检查成功后变为 `healthy`，如果连续一定次数失败，则会变为 `unhealthy`。

`HEALTHCHECK` 支持下列选项：

-   `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
-   `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
-   `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 `unhealthy`，默认 3 次。

和 `CMD`, `ENTRYPOINT` 一样，`HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。

在 `HEALTHCHECK [选项] CMD` 后面的命令，格式和 `ENTRYPOINT` 一样，分为 `shell` 格式，和 `exec` 格式。命令的返回值决定了该次健康检查的成功与否：`0`：成功；`1`：失败；`2`：保留，不要使用这个值。

假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用 `curl` 来帮助判断，其 `Dockerfile` 的 `HEALTHCHECK` 可以这么写：

这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。

使用 `docker build` 来构建这个镜像：

构建好了后，我们启动一个容器：

当运行该镜像后，可以通过 `docker container ls` 看到最初的状态为 `(health: starting)`：

在等待几秒钟后，再次 `docker container ls`，就会看到健康状态变化为了 `(healthy)`：

如果健康检查连续失败超过了重试次数，状态就会变为 `(unhealthy)`。

为了帮助排障，健康检查命令的输出（包括 `stdout` 以及 `stderr`）都会被存储于健康状态里，可以用 `docker inspect` 来查看。

### [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#onbuild-%E4%B8%BA%E4%BB%96%E4%BA%BA%E4%BD%9C%E5%AB%81%E8%A1%A3%E8%A3%B3) ONBUILD(为他人作嫁衣裳)

格式：`ONBUILD <其它指令>`。

`ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

`Dockerfile` 中的其它指令都是为了定制当前镜像而准备的，唯有 `ONBUILD` 是为了帮助别人定制自己而准备的。

假设我们要制作 Node.js 所写的应用的镜像。我们都知道 Node.js 使用 `npm` 进行包管理，所有依赖、配置、启动信息等会放到 `package.json` 文件里。在拿到程序代码后，需要先进行 `npm install` 才可以获得所有需要的依赖。然后就可以通过 `npm start`来启动应用。因此，一般来说会这样写 `Dockerfile`：

把这个 `Dockerfile` 放到 Node.js 项目的根目录，构建好镜像后，就可以直接拿来启动容器运行。但是如果我们还有第二个 Node.js 项目也差不多呢？好吧，那就再把这个 `Dockerfile` 复制到第二个项目里。那如果有第三个项目呢？再复制么？文件的副本越多，版本控制就越困难，让我们继续看这样的场景维护的问题。

如果第一个 Node.js 项目在开发过程中，发现这个 `Dockerfile` 里存在问题，比如敲错字了、或者需要安装额外的包，然后开发人员修复了这个 `Dockerfile`，再次构建，问题解决。第一个项目没问题了，但是第二个项目呢？虽然最初 `Dockerfile` 是复制、粘贴自第一个项目的，但是并不会因为第一个项目修复了他们的 `Dockerfile`，而第二个项目的 `Dockerfile` 就会被自动修复。

那么我们可不可以做一个基础镜像，然后各个项目使用这个基础镜像呢？这样基础镜像更新，各个项目不用同步 `Dockerfile`的变化，重新构建后就继承了基础镜像的更新？好吧，可以，让我们看看这样的结果。那么上面的这个 `Dockerfile` 就会变为：

这里我们把项目相关的构建指令拿出来，放到子项目里去。假设这个基础镜像的名字为 `my-node` 的话，各个项目内的自己的 `Dockerfile` 就变为：

基础镜像变化后，各个项目都用这个 `Dockerfile` 重新构建镜像，会继承基础镜像的更新。

那么，问题解决了么？没有。准确说，只解决了一半。如果这个 `Dockerfile` 里面有些东西需要调整呢？比如 `npm install` 都需要加一些参数，那怎么办？这一行 `RUN` 是不可能放入基础镜像的，因为涉及到了当前项目的 `./package.json`，难道又要一个个修改么？所以说，这样制作基础镜像，只解决了原来的 `Dockerfile` 的前 4 条指令的变化问题，而后面三条指令的变化则完全没办法处理。

`ONBUILD` 可以解决这个问题。让我们用 `ONBUILD` 重新写一下基础镜像的 `Dockerfile`:

这次我们回到原始的 `Dockerfile`，但是这次将项目相关的指令加上 `ONBUILD`，这样在构建基础镜像的时候，这三行并不会被执行。然后各个项目的 `Dockerfile` 就变成了简单地：

是的，只有这么一行。当在各个项目目录中，用这个只有一行的 `Dockerfile` 构建镜像时，之前基础镜像的那三行 `ONBUILD` 就会开始执行，成功的将当前项目的代码复制进镜像、并且针对本项目执行 `npm install`，生成应用镜像。

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E4%BA%8C%E3%80%81%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5) 二、最佳实践

有任何的问题或建议，欢迎给我留言 😆

## [#](https://dunwu.github.io/linux-tutorial/docker/docker-dockerfile.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99) 参考资料

-   [Dockerfie 官方文档 (opens new window)](https://docs.docker.com/engine/reference/builder/)
-   [Best practices for writing Dockerfiles (opens new window)](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
-   [Docker 官方镜像 Dockerfile (opens new window)](https://github.com/docker-library/docs)
-   [Dockerfile 指令详解 (opens new window)](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/)