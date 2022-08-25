 2021.7.21  [Linux](https://waynerv.com/categories/linux/)  1670  4 分钟  2

## 目录

1.  [cgroup 在何时创建](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#cgroup-%E5%9C%A8%E4%BD%95%E6%97%B6%E5%88%9B%E5%BB%BA)
2.  [cgroup 如何控制容器的资源](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#cgroup-%E5%A6%82%E4%BD%95%E6%8E%A7%E5%88%B6%E5%AE%B9%E5%99%A8%E7%9A%84%E8%B5%84%E6%BA%90)
    1.  [init 进程](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#init-%E8%BF%9B%E7%A8%8B)
    2.  [容器中的其他进程](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E5%AE%B9%E5%99%A8%E4%B8%AD%E7%9A%84%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B)
3.  [如何配置 cgroup](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E5%A6%82%E4%BD%95%E9%85%8D%E7%BD%AE-cgroup)
    1.  [文件系统方式](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%96%B9%E5%BC%8F)
    2.  [高阶工具方式](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E9%AB%98%E9%98%B6%E5%B7%A5%E5%85%B7%E6%96%B9%E5%BC%8F)
4.  [参考链接](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

cgroups（control groups）是由 Linux 内核提供的一种特性，它能够限制、核算和隔离一组进程所使用的系统资源（如 CPU、内存、磁盘 I/O、网络等）。

在上一篇文章中我们已了解 Namespace 在容器技术中扮演的角色，如果说 Namespace 控制了容器中的进程能看到什么，那么 cgroups 则控制了容器中的进程能使用多少资源。Namespace 实现了进程的隔离，cgroups 则实现了资源的限制，后者同样是构建容器的基础。

本文将沿袭 Namespace 文章的行文思路，实际创建一个容器，观察宿主机中 cgroups 的变化，来实际展示 cgroups 如何工作，然后了解如何自行配置 cgroups。

## [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#cgroup-%E5%9C%A8%E4%BD%95%E6%97%B6%E5%88%9B%E5%BB%BA)[cgroup 在何时创建](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:cgroup-%E5%9C%A8%E4%BD%95%E6%97%B6%E5%88%9B%E5%BB%BA)

Linux 内核通过一个叫做 `cgroupfs` 的伪文件系统来提供管理 cgroup 的接口，我们可以通过 `lscgroup` 命令来列出系统中已有的 cgroup，该命令实际上遍历了 `/sys/fs/cgroup/` 目录中的文件：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ lscgroup <span>|</span> tee cgroup.a
</span></span></code></pre></td></tr></tbody></table>

_如果你使用的 Linux 发行版没有 `lscgroup`_ 命令，可通过 [command-not-found.com](https://command-not-found.com/lscgroup) 提供的指令下载安装。

我们将输出结果保存到 `cgroup.a` 文件中。接着在另一窗口中根据 Namespace 文章中的步骤启动一个容器：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ <span>cd</span> /mycontainer
</span></span><span><span>$ runc run mybox
</span></span></code></pre></td></tr></tbody></table>

回到原来的窗口再次执行 `lsgroup` 命令：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ lscgroup <span>|</span> tee group.b
</span></span></code></pre></td></tr></tbody></table>

现在对比两次 `lscgroup` 命令的输出结果：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ diff group.a group.b
</span></span><span><span>
</span></span><span><span>&gt; perf_event:/mybox
</span></span><span><span>&gt; freezer:/mybox
</span></span><span><span>&gt; net_cls,net_prio:/mybox
</span></span><span><span>&gt; cpu,cpuacct:/user.slice/mybox
</span></span><span><span>&gt; blkio:/user.slice/mybox
</span></span><span><span>&gt; cpuset:/mybox
</span></span><span><span>&gt; hugetlb:/mybox
</span></span><span><span>&gt; pids:/user.slice/user-0.slice/session-5.scope/mybox
</span></span><span><span>&gt; memory:/user.slice/user-0.slice/session-5.scope/mybox
</span></span><span><span>&gt; devices:/user.slice/mybox
</span></span></code></pre></td></tr></tbody></table>

从结果中可看到，`mybox` 容器创建后，系统中专门为其创建了所有类型的新的 cgroup。

## [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#cgroup-%E5%A6%82%E4%BD%95%E6%8E%A7%E5%88%B6%E5%AE%B9%E5%99%A8%E7%9A%84%E8%B5%84%E6%BA%90)[cgroup 如何控制容器的资源](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:cgroup-%E5%A6%82%E4%BD%95%E6%8E%A7%E5%88%B6%E5%AE%B9%E5%99%A8%E7%9A%84%E8%B5%84%E6%BA%90)

cgroup 所控制的对象是进程，它控制一个或一组进程所能使用多少内存/CPU/网络等等。一个 cgroup 的 `tasks` 列表中记录了其所控制进程的 PID，该 `tasks` 实际上也是 `cgroupfs` 中的一个文件。

### [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#init-%E8%BF%9B%E7%A8%8B)[init 进程](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:init-%E8%BF%9B%E7%A8%8B)

我们首先在宿主机中打印出容器中的进程信息，找到容器的 `init` 进程：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc ps mybox
</span></span><span><span>
</span></span><span><span>UID          PID    PPID  C STIME TTY          TIME CMD
</span></span><span><span>root        <span>2250</span>    <span>2240</span>  <span>0</span> 15:28 pts/0    00:00:00 sh
</span></span></code></pre></td></tr></tbody></table>

任意打印一些类型的 cgroup 的 `tasks` 列表：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cat /sys/fs/cgroup/memory/user.slice/user-0.slice/session-5.scope/mybox/tasks
</span></span><span><span><span>2250</span>
</span></span><span><span>$ cat /sys/fs/cgroup/blkio/user.slice/mybox/tasks
</span></span><span><span><span>2250</span>
</span></span></code></pre></td></tr></tbody></table>

这一过程简单明了：容器创建之后，容器的 `init` 进程会被加入到为该容器所创建的 cgroups 之中，我们可以通过 `/proc/$PID/cgroup` 得到更肯定的结果：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cat /proc/2250/cgroup
</span></span><span><span>12:devices:/user.slice/mybox
</span></span><span><span>11:memory:/user.slice/user-0.slice/session-5.scope/mybox
</span></span><span><span>10:pids:/user.slice/user-0.slice/session-5.scope/mybox
</span></span><span><span>9:hugetlb:/mybox
</span></span><span><span>8:cpuset:/mybox
</span></span><span><span>7:rdma:/
</span></span><span><span>6:blkio:/user.slice/mybox
</span></span><span><span>5:cpu,cpuacct:/user.slice/mybox
</span></span><span><span>4:net_cls,net_prio:/mybox
</span></span><span><span>3:freezer:/mybox
</span></span><span><span>2:perf_event:/mybox
</span></span><span><span>1:name<span>=</span>systemd:/user.slice/user-0.slice/session-5.scope/mybox
</span></span><span><span>0::/user.slice/user-0.slice/session-5.scope
</span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E5%AE%B9%E5%99%A8%E4%B8%AD%E7%9A%84%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B)[容器中的其他进程](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:%E5%AE%B9%E5%99%A8%E4%B8%AD%E7%9A%84%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B)

接下来我们在 `mybox` 容器中运行一个新的进程：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># 在 mybox 容器中运行</span>
</span></span><span><span>$ top -b
</span></span></code></pre></td></tr></tbody></table>

看看是否会创建新的 cgroup：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ lscgroup <span>|</span> tee group.c
</span></span><span><span>$ diff group.b group.c
</span></span></code></pre></td></tr></tbody></table>

没有输出任何结果，说明没有创建新的 cgroup。既然 cgroup 可以控制一组进程，我们猜测在已运行容器中新建的进程，也都会加入到 `init` 进程所属的 cgroups 中。

下面开始验证，首先找到新建进程的 PID：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc ps mybox
</span></span><span><span>UID          PID    PPID  C STIME TTY          TIME CMD
</span></span><span><span>root        <span>2250</span>    <span>2240</span>  <span>0</span> 15:28 pts/0    00:00:00 sh
</span></span><span><span>root        <span>2576</span>    <span>2250</span>  <span>0</span> 15:59 pts/0    00:00:00 top -b
</span></span></code></pre></td></tr></tbody></table>

新进程的 PID 是 2576，然后打印该进程的 cgroups 信息：

输出和 PID 2250 进程的输出完全一致，我们也可以打印其中一个 cgroup 的 `tasks` 列表：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>cat /sys/fs/cgroup/blkio/user.slice/mybox/tasks
</span></span><span><span><span>2250</span>
</span></span><span><span><span>2576</span>
</span></span></code></pre></td></tr></tbody></table>

完全符合预期。实际上向 `tasks` 文件直接写入进程的 PID 就实现了将进程加入到该 cgroup 中。当一个容器被创建时，将为每种类型的资源创建一个新的 cgroup，在容器中运行的所有进程都将加入到这些 cgroup 中。

通过控制容器中运行的所有进程，cgroups 实现了对容器的资源限制。

## [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E5%A6%82%E4%BD%95%E9%85%8D%E7%BD%AE-cgroup)[如何配置 cgroup](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:%E5%A6%82%E4%BD%95%E9%85%8D%E7%BD%AE-cgroup)

下面我们将以内存 cgroup 为例，了解如何配置 cgroup 以实现对 `mybox` 容器的内存限制。

配置 cgroup 有两种方式，一种是直接修改 `cgroupfs` 中的指定文件，另一种是通过 `runc` 或 `docker` 等高阶工具实现。

### [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%96%B9%E5%BC%8F)[文件系统方式](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%96%B9%E5%BC%8F)

通过 `cgroupfs` 的方式，查看/修改该 cgroup 目录下的特定文件即可查看/设置该 cgroup 的限额：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>cat /sys/fs/cgroup/memory/user.slice/user-0.slice/session-5.scope/mybox/memory.limit_in_bytes
</span></span><span><span><span>9223372036854771712</span>
</span></span></code></pre></td></tr></tbody></table>

修改 `memory.limit_in_bytes` 文件即可设置最大可用内存，现在我们并未对该容器设置任何限制，因此内存限制的当前值是一个无意义的特别大的值，现在我们向该文件直接写入新的值：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>echo</span> <span>"100000000"</span> &gt; /sys/fs/cgroup/memory/user.slice/user-0.slice/session-5.scope/mybox/memory.limit_in_bytes
</span></span></code></pre></td></tr></tbody></table>

这样就设置了新的内存限制。写入新的限制值后，容器中的所有进程不能使用总共超过 100M 的内存，超过后将根据 `memory.oom_control` 文件中设置的 `OOM` 策略 `kill` 或 `sleep` 容器中的进程。

### [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E9%AB%98%E9%98%B6%E5%B7%A5%E5%85%B7%E6%96%B9%E5%BC%8F)[高阶工具方式](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:%E9%AB%98%E9%98%B6%E5%B7%A5%E5%85%B7%E6%96%B9%E5%BC%8F)

通过高阶工具提供的途径来配置 cgroup 是一种更友好的方式，虽然这些工具背后的实现也是如上所述更改 `cgroupfs`。

对于 `runc` 来说，需要修改 `filesystem bundle` 中的 `config.json` 文件来配置 cgroup。设置内存限制需要如下修改 JSON 对象中的 `linux.resources` 字段：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="json"><span><span><span>"resources"</span><span>:</span> <span>{</span>
</span></span><span><span>    <span>"memory"</span><span>:</span> <span>{</span>
</span></span><span><span>    <span>"limit"</span><span>:</span> <span>100000</span><span>,</span>
</span></span><span><span>    <span>"reservation"</span><span>:</span> <span>200000</span>
</span></span><span><span>    <span>},</span>
</span></span><span><span>    <span>...</span>
</span></span><span><span><span>}</span>
</span></span></code></pre></td></tr></tbody></table>

对于 `docker` 来说更为简单，它本身就是一个面向用户的封装好的工具，执行 `docker run` 命令时通过 `--memory` 选项即可指定内存限制。实际上该参数会被写入到 `config.json` 由运行时实现 `runc` 使用，再由 `runc` 去更改 `cgroupfs`。

## [](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/container-fundamentals-resource-limitation-using-cgroups/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Understand Container 2: Linux cgroups](https://pierrchen.blogspot.com/2018/04/container-deep-dive-2-linux-cgroups.html)
-   [cgroups-wikipedia](https://en.wikipedia.org/wiki/Cgroups)
-   [Limit a container’s access to memory](https://docs.docker.com/config/containers/resource_constraints/#limit-a-containers-access-to-memory)
-   [Control groups - Memory](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md#memory)

updatedupdated2022-07-172022-07-17

[容器](https://waynerv.com/tags/%E5%AE%B9%E5%99%A8/) [Cgroups](https://waynerv.com/tags/cgroups/)