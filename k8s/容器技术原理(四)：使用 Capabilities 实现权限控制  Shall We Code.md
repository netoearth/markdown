 2021.7.22  [Linux](https://waynerv.com/categories/linux/)  2352  5 分钟  2

## 目录

1.  [比超级用户更细粒度的权限控制](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E6%AF%94%E8%B6%85%E7%BA%A7%E7%94%A8%E6%88%B7%E6%9B%B4%E7%BB%86%E7%B2%92%E5%BA%A6%E7%9A%84%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6)
2.  [特权容器的安全风险](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E7%89%B9%E6%9D%83%E5%AE%B9%E5%99%A8%E7%9A%84%E5%AE%89%E5%85%A8%E9%A3%8E%E9%99%A9)
3.  [准备工作](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
4.  [创建容器时添加 capabilities](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%88%9B%E5%BB%BA%E5%AE%B9%E5%99%A8%E6%97%B6%E6%B7%BB%E5%8A%A0-capabilities)
    1.  [capabilities 的技术细节](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#capabilities-%E7%9A%84%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82)
5.  [容器运行时添加 capabilities](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E6%97%B6%E6%B7%BB%E5%8A%A0-capabilities)
6.  [查看进程具有的 capabilities](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E6%9F%A5%E7%9C%8B%E8%BF%9B%E7%A8%8B%E5%85%B7%E6%9C%89%E7%9A%84-capabilities)
    1.  [capsh](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#capsh)
    2.  [pscap](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#pscap)
7.  [参考链接](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

如果你使用 `runc` 运行一个容器并执行以下操作，会得到有趣的结果：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ whoami
</span></span><span><span>root
</span></span><span><span>$ id -u root
</span></span><span><span><span>0</span>
</span></span><span><span>$ hostname mybox
</span></span><span><span>hostname: sethostname: Operation not permitted
</span></span></code></pre></td></tr></tbody></table>

即使我们使用的是 `UID` 为 0 的 `root` 用户，也没有权限执行修改 hostname 的操作。

实际上 `root` 用户拥有最高特权早就成了过去式，Linux 内核在 2.2 版本就引入了一种新的权限检查机制 - capabilities。

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E6%AF%94%E8%B6%85%E7%BA%A7%E7%94%A8%E6%88%B7%E6%9B%B4%E7%BB%86%E7%B2%92%E5%BA%A6%E7%9A%84%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6)[比超级用户更细粒度的权限控制](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E6%AF%94%E8%B6%85%E7%BA%A7%E7%94%A8%E6%88%B7%E6%9B%B4%E7%BB%86%E7%B2%92%E5%BA%A6%E7%9A%84%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6)

传统的 Linux 权限检查模型较为简单，内核在进行权限检查时只会区分两类进程：

-   特权进程，其有效用户 ID为 0，该用户也就是我们常说的超级用户或 `root`。
-   非特权进程，有效用户 ID 不为 0。

特权进程将直接绕过内核的所有检查，非特权进程则需要基于进程的有效用户 ID 和有效用户组 ID 等凭证执行检查。

为了适应更复杂的权限需求，从 2.2 版本起 Linux 内核能够进一步将超级用户的权限分解为细颗粒度的单元，这些单元称为 capabilities。例如，capability `CAP_CHOWN` 允许用户对文件的 UID 和 GID 进行任意修改，即执行 `chown` 命令。几乎所有与超级用户相关的特权都被分解成了单独的 capability。

capabilities 的引入有以下好处：

-   从超级用户的权限中移除部分 capability 以削弱其权限，提高系统的安全性。
-   可以根据需求非常精准地向普通用户授予部分特殊权限。

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E7%89%B9%E6%9D%83%E5%AE%B9%E5%99%A8%E7%9A%84%E5%AE%89%E5%85%A8%E9%A3%8E%E9%99%A9)[特权容器的安全风险](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E7%89%B9%E6%9D%83%E5%AE%B9%E5%99%A8%E7%9A%84%E5%AE%89%E5%85%A8%E9%A3%8E%E9%99%A9)

容器通过 namespace 来隔离进程和资源，但并不是所有的资源都可以被 namespace 化，容器和宿主机并不是完全隔离的，比如容器和宿主机中的时间就是共享的。如果容器中的进程拥有一切特权，它可以运行直接访问硬件的（恶意）程序甚至直接修改宿主机的文件系统，因此有必要对容器中的操作进行一定的限制，否则会影响到宿主机的稳定性，甚至带来严重的安全风险。

出于以上考虑，默认情况下容器运行时使用白名单的方式在创建容器时加入一部分的 capabilities，在容器中即使你是超级用户也没有权限执行特定的操作。

接下来我们通过实例加深对容器中 capabilities 的认识。

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)[准备工作](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)

我们将在容器中使用额外的工具库 `libcap` 实现和 capabilities 的交互，为此需要将其安装到一个 `filesystem bundle` 中，这种方式在之前的文章中已有介绍，具体方式如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># 创建 bundle 的顶层目录</span>
</span></span><span><span>mkdir /mycontainer2
</span></span><span><span><span>cd</span> /mycontainer2
</span></span><span><span>
</span></span><span><span><span># 创建用于存放 root filesystem 的 rootfs 目录</span>
</span></span><span><span>mkdir rootfs
</span></span><span><span>
</span></span><span><span><span># 利用 Docker 导出已安装 libcap 容器的 root filesystem </span>
</span></span><span><span>docker <span>export</span> <span>$(</span>docker create cmd.cat/capsh<span>)</span> <span>|</span> tar -C rootfs -xvf -
</span></span><span><span>
</span></span><span><span><span># 创建一个 config.json 作为整个 bundle 的 spec</span>
</span></span><span><span>runc spec
</span></span></code></pre></td></tr></tbody></table>

然后就可以使用 `runc run` 从 `/mycontainer2` 目录运行一个已安装该库的基础容器了。

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%88%9B%E5%BB%BA%E5%AE%B9%E5%99%A8%E6%97%B6%E6%B7%BB%E5%8A%A0-capabilities)[创建容器时添加 capabilities](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E5%88%9B%E5%BB%BA%E5%AE%B9%E5%99%A8%E6%97%B6%E6%B7%BB%E5%8A%A0-capabilities)

在开头的示例中，我们无法在容器中以 `root` 用户设置 hostname，是因为缺少了 `CAP_SYS_ADMIN` 这一 capability，它并未包含在容器默认添加 capabilities 的白名单中。

在之前的一篇文章中，我们介绍过容器运行时会根据 `bundle` 中的 `config.json` ，为其创建的容器设置运行参数和执行环境，这一过程也包括了设置容器内进程的 capabilities。

通过修改 `config.json` ，向 JSON 中的 `process.capabilities` 对象的 `bounding`，`permitted` 和 `effective` 列表中加入 `"CAP_SYS_ADMIN"`，该 capability 将加入到容器 `init` 进程的对应 capabilities 集合中。

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>"capabilities"</span>: <span>{</span>
</span></span><span><span><span>"bounding"</span>: <span>[</span>
</span></span><span><span><span>"CAP_AUDIT_WRITE"</span>,
</span></span><span><span><span>"CAP_KILL"</span>,
</span></span><span><span><span>"CAP_NET_BIND_SERVICE"</span>,
</span></span><span><span><span>"CAP_SYS_ADMIN"</span>
</span></span><span><span><span>]</span>,
</span></span><span><span><span>"effective"</span>: <span>[</span>
</span></span><span><span><span>"CAP_AUDIT_WRITE"</span>,
</span></span><span><span><span>"CAP_KILL"</span>,
</span></span><span><span><span>"CAP_NET_BIND_SERVICE"</span>,
</span></span><span><span><span>"CAP_SYS_ADMIN"</span>
</span></span><span><span><span>]</span>,
</span></span><span><span><span>"inheritable"</span>: <span>[</span>
</span></span><span><span><span>"CAP_AUDIT_WRITE"</span>,
</span></span><span><span><span>"CAP_KILL"</span>,
</span></span><span><span><span>"CAP_NET_BIND_SERVICE"</span>
</span></span><span><span><span>]</span>,
</span></span><span><span><span>"permitted"</span>: <span>[</span>
</span></span><span><span><span>"CAP_AUDIT_WRITE"</span>,
</span></span><span><span><span>"CAP_KILL"</span>,
</span></span><span><span><span>"CAP_NET_BIND_SERVICE"</span>,
</span></span><span><span><span>"CAP_SYS_ADMIN"</span>
</span></span><span><span><span>]</span>,
</span></span><span><span><span>"ambient"</span>: <span>[</span>
</span></span><span><span><span>"CAP_AUDIT_WRITE"</span>,
</span></span><span><span><span>"CAP_KILL"</span>,
</span></span><span><span><span>"CAP_NET_BIND_SERVICE"</span>
</span></span><span><span><span>]</span>
</span></span><span><span><span>}</span>
</span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#capabilities-%E7%9A%84%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82)[capabilities 的技术细节](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:capabilities-%E7%9A%84%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82)

capabilities 可以应用于文件和进程（或线程，Linux 内核不区分进程和线程），文件的 capabilities 存储在文件的扩展属性中，扩展属性在构建镜像时会被清理掉，所以在容器中我们基本不需要考虑文件的 capabilities。

进程的 capabilities 通过每个进程单独维护的 5 个 capability 集合来控制，每个集合中都包含 0 个或多个 capabilities：

-   Permitted：进程所能够使用的 capabilities 的超集
-   Inheritable：进程在执行 `exec()` 系统调用时，能够被新的派生进程所继承的 capabilities
-   Effective：内核对进程执行权限检查时所使用的集合
-   Bounding：Inheritable 集合的超集，一个capability 必须在 Bounding 集合中才能添加到Inheritable
-   Ambient：非特权程序执行 `exec()` 系统调用时将保留的 capabilities

如上所示我们向 `init` 进程的 Permitted、Bounding 和 Effective 集合中加入了 `CAP_SYS_ADMIN`，因此 `init` 进程将通过内核对 `CAP_SYS_ADMIN` 的检查。

下面我们根据新的 `config.json` 运行一个新的容器，现在可以修改 hostname 了：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc run mybox2
</span></span><span><span>$ hostname super
</span></span><span><span>$ hostname
</span></span><span><span>super
</span></span></code></pre></td></tr></tbody></table>

执行以上操作时，我们位于作为容器 `init` 进程的 `sh` 进程中，如果在容器中继续创建新的进程，是否也会具有新加入的 capability？我们来试一下，在一个新的窗口中执行以下命令：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc <span>exec</span> -t mybox2 sh
</span></span><span><span>$ hostname
</span></span><span><span>super
</span></span><span><span>$ hostname hello
</span></span><span><span>$ hostname
</span></span><span><span>hello
</span></span></code></pre></td></tr></tbody></table>

修改 hostname 的操作执行成功，因为新创建的进程完全复制了 `init` 进程的 capabilities。

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E6%97%B6%E6%B7%BB%E5%8A%A0-capabilities)[容器运行时添加 capabilities](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E6%97%B6%E6%B7%BB%E5%8A%A0-capabilities)

除了修改 `config.json` 加入 capabilities，我们还能够在容器运行时阶段添加 capabilities。

首先将 `config.json` 还原，然后运行一个新的容器 `mybox3`，在新的 `sh` 进程中确认已经不再具有 `CAP_SYS_ADMIN`。

然后通过 `runc exec` 在该容器中创建一个新的进程，并通过 `--cap` 选项为该进程添加 `CAP_SYS_ADMIN` :

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>runc <span>exec</span> --cap CAP_SYS_ADMIN mybox3 /bin/hostname origin
</span></span></code></pre></td></tr></tbody></table>

该操作的原理是，既然 `runc` 能够根据 `config.json` 设置 `init` 进程的 capabilities 集合，它同样也能为容器内运行的其他进程设置。

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E6%9F%A5%E7%9C%8B%E8%BF%9B%E7%A8%8B%E5%85%B7%E6%9C%89%E7%9A%84-capabilities)[查看进程具有的 capabilities](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E6%9F%A5%E7%9C%8B%E8%BF%9B%E7%A8%8B%E5%85%B7%E6%9C%89%E7%9A%84-capabilities)

### [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#capsh)[capsh](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:capsh)

在容器内执行 `capsh --print` 能够获取到更多关于 capabilities 的信息：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ capsh --print
</span></span><span><span>
</span></span><span><span>Current: <span>=</span> cap_kill,cap_net_bind_service,cap_audit_write+eip cap_sys_admin+ep
</span></span><span><span>Bounding <span>set</span> <span>=</span>cap_kill,cap_net_bind_service,cap_sys_admin,cap_audit_write
</span></span><span><span>Ambient <span>set</span> <span>=</span>cap_kill,cap_net_bind_service,cap_audit_write
</span></span><span><span>Securebits: 00/0x0/1<span>'</span>b0
</span></span><span><span> secure-noroot: no <span>(</span>unlocked<span>)</span>
</span></span><span><span> secure-no-suid-fixup: no <span>(</span>unlocked<span>)</span>
</span></span><span><span> secure-keep-caps: no <span>(</span>unlocked<span>)</span>
</span></span><span><span> secure-no-ambient-raise: no <span>(</span>unlocked<span>)</span>
</span></span><span><span><span>uid</span><span>=</span>0<span>(</span>root<span>)</span>
</span></span><span><span><span>gid</span><span>=</span>0<span>(</span>root<span>)</span>
</span></span><span><span><span>groups</span><span>=</span>
</span></span></code></pre></td></tr></tbody></table>

该命令打印了当前进程所具有的 capabilities。

在 `Current` 和 `Bounding set` 中包含了我们通过 `config.json` 加入的 `cap_sys_admin`。capability 末尾的 `+eip` 代表了该 capability 同时存在于 Effective，Inheritable 和 Permitted 集合中。

### [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#pscap)[pscap](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:pscap)

首先在宿主机获取容器中已运行进程的 PID：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc ps mybox2
</span></span><span><span>UID          PID    PPID  C STIME TTY          TIME CMD
</span></span><span><span>root        <span>9592</span>    <span>9580</span>  <span>0</span> 14:39 pts/0    00:00:00 sh
</span></span><span><span>root        <span>9776</span>    <span>9765</span>  <span>0</span> 14:46 pts/1    00:00:00 sh
</span></span></code></pre></td></tr></tbody></table>

在宿主机中安装 `pscap` 程序：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ apt-get install libcap-ng-utils
</span></span></code></pre></td></tr></tbody></table>

根据获得的 PID，查看容器中进程所具有的 capabilities：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>pscap <span>|</span> grep <span>"9592\|9776"</span>
</span></span><span><span><span>9580</span>  <span>9592</span>  root        sh                kill, net_bind_service, sys_admin, audit_write
</span></span><span><span><span>9765</span>  <span>9776</span>  root        sh                kill, net_bind_service, sys_admin, audit_write
</span></span></code></pre></td></tr></tbody></table>

## [](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/container-fundamentals-permission-control-using-capabilities/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Understand Container 3: Linux Capabilities](https://pierrchen.blogspot.com/2018/05/container-deep-dive-3-linux-capabilities.html)
-   [capabilities(7) — Linux manual page](https://man7.org/linux/man-pages/man7/capabilities.7.html)
-   [dockerlabs-Capabilities](https://dockerlabs.collabnix.com/advanced/security/capabilities/)
-   [Linux Capabilities 入门教程：概念篇](https://fuckcloudnative.io/posts/linux-capabilities-why-they-exist-and-how-they-work/)
-   [Why A Privileged Container in Docker Is a Bad Idea](https://www.trendmicro.com/en_us/research/19/l/why-running-a-privileged-container-in-docker-is-a-bad-idea.html)

updatedupdated2022-07-172022-07-17

[容器](https://waynerv.com/tags/%E5%AE%B9%E5%99%A8/) [Capabilities](https://waynerv.com/tags/capabilities/)