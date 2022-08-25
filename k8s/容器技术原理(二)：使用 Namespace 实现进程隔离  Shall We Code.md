 2021.7.21  [Linux](https://waynerv.com/categories/linux/)  2643  6 分钟  1

## 目录

1.  [准备工作](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
    1.  [filesystem bundle](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#filesystem-bundle)
    2.  [系统监测工具](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E7%B3%BB%E7%BB%9F%E7%9B%91%E6%B5%8B%E5%B7%A5%E5%85%B7)
2.  [使用 runc 运行容器](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E4%BD%BF%E7%94%A8-runc-%E8%BF%90%E8%A1%8C%E5%AE%B9%E5%99%A8)
    1.  [找到进程所属的 namespace](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E6%89%BE%E5%88%B0%E8%BF%9B%E7%A8%8B%E6%89%80%E5%B1%9E%E7%9A%84-namespace)
    2.  [观察 namespace 中的进程](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E8%A7%82%E5%AF%9F-namespace-%E4%B8%AD%E7%9A%84%E8%BF%9B%E7%A8%8B)
3.  [在容器中创建新的进程](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84%E8%BF%9B%E7%A8%8B)
4.  [总结](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E6%80%BB%E7%BB%93)
5.  [参考链接](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

Namespace 是由 Linux 内核提供的一种特性，它能够将一些系统资源包装到一个抽象的空间中，并使得该空间中的进程以为这些资源是系统中仅有的资源。Namespace 是构建容器技术的基石，它使得[[容器]]内的进程只能看到容器内的进程和资源，实现与宿主系统以及其他容器的进程和资源隔离。

Namespace 按操作的系统资源不同有很多种类，比如 cgroup namespace，mount namespace 等等，接下来我们仅以 pid namespace 为例，以 `runC` 作为容器运行时实现，来演示当我们执行对容器的操作时，namespace 是如何工作的。

在上一篇文章中我们已经介绍过，绝大部分容器系统都使用 `runC` 作为底层的运行时实现，如果你是在 Linux 发行版系统中使用 `docker` ，甚至不需要专门安装就能使用 `runc` 命令。

## [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)[准备工作](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)

### [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#filesystem-bundle)[filesystem bundle](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:filesystem-bundle)

`runC` 只能从 `filesystem bundle` 中执行容器（`filesystem bundle` 顾名思义就是一个满足特定结构的文件夹），但是我们可以使用 `docker` 来准备一个可用的 `bundle` :

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
</span></span><span><span>$ mkdir /mycontainer
</span></span><span><span>$ <span>cd</span> /mycontainer
</span></span><span><span>
</span></span><span><span><span># 创建用于存放 root filesystem 的 rootfs 目录</span>
</span></span><span><span>$ mkdir rootfs
</span></span><span><span>
</span></span><span><span><span># 利用 Docker 导出 busybox 容器的 root filesystem </span>
</span></span><span><span>$ docker <span>export</span> <span>$(</span>docker create busybox<span>)</span> <span>|</span> tar -C rootfs -xvf -
</span></span><span><span>
</span></span><span><span><span># 创建一个 config.json 作为整个 bundle 的 spec</span>
</span></span><span><span>$ runc spec
</span></span></code></pre></td></tr></tbody></table>

此时整个 `bundle` 的目录结构如下：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ tree -L <span>2</span> /mycontainer
</span></span><span><span>
</span></span><span><span>/mycontainer
</span></span><span><span>├── config.json
</span></span><span><span>└── rootfs
</span></span><span><span>    ├── bin
</span></span><span><span>    ├── dev
</span></span><span><span>    ├── etc
</span></span><span><span>    ├── home
</span></span><span><span>    ├── proc
</span></span><span><span>    ├── root
</span></span><span><span>    ├── sys
</span></span><span><span>    ├── tmp
</span></span><span><span>    ├── usr
</span></span><span><span>    └── var
</span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E7%B3%BB%E7%BB%9F%E7%9B%91%E6%B5%8B%E5%B7%A5%E5%85%B7)[系统监测工具](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E7%B3%BB%E7%BB%9F%E7%9B%91%E6%B5%8B%E5%B7%A5%E5%85%B7)

为了完成演示，我们需要一些第三方的系统监测工具作为辅助：

1.  监测进程的启动以获得容器中运行进程的 PID，如 ubuntu 中的 `forkstat` ，它可以实时地监测 `fork()`, `exec()` 和 `exit()` 等系统调用，安装方式如下：
    
2.  查看 namespace 信息，如 `[cinf](https://github.com/mhausenblas/cinf)` ，它是一个能够方便地列出系统中所有 namespace 或查看某个 namespce 详细信息的命令行工具，安装方式如下：
    
    <table><tbody><tr><td><pre tabindex="0"><code><span>1
    </span><span>2
    </span><span>3
    </span><span>4
    </span><span>5
    </span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ curl -s -L https://github.com/mhausenblas/cinf/releases/latest/download/cinf_linux_amd64.tar.gz <span>\
    </span></span></span><span><span><span></span>     -o cinf.tar.gz <span>&amp;&amp;</span> <span>\
    </span></span></span><span><span><span></span>     tar xvzf cinf.tar.gz cinf <span>&amp;&amp;</span> <span>\
    </span></span></span><span><span><span></span>     mv cinf /usr/local/bin <span>&amp;&amp;</span> <span>\
    </span></span></span><span><span><span></span>     rm cinf*
    </span></span></code></pre></td></tr></tbody></table>
    

## [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E4%BD%BF%E7%94%A8-runc-%E8%BF%90%E8%A1%8C%E5%AE%B9%E5%99%A8)[使用 runc 运行容器](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E4%BD%BF%E7%94%A8-runc-%E8%BF%90%E8%A1%8C%E5%AE%B9%E5%99%A8)

首先我们需要在一个窗口中运行 `forkstat` ：

接着另外新建一个终端窗口，切换到 `/mycontainer` 目录，使用 `runC` 运行容器：

执行后会直接进入到新创建的容器中，运行 `ps` 命令：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>PID   USER     TIME  COMMAND
</span></span><span><span>    <span>1</span> root      0:00 sh
</span></span><span><span>    <span>7</span> root      0:00 ps
</span></span></code></pre></td></tr></tbody></table>

`forkstat` 窗口将会有以下输出：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>Time     Event     PID Info   Duration Process
</span></span><span><span>12:35:22 <span>exec</span>    <span>33040</span>                 runc run mybox
</span></span><span><span>12:35:22 <span>exec</span>    <span>33047</span>                 runc init
</span></span><span><span>12:35:22 <span>exec</span>    <span>33049</span>                 dumpe2fs -h /dev/sdb3
</span></span><span><span>12:35:22 <span>exec</span>    <span>33050</span>                 dumpe2fs -h /dev/sdb3
</span></span><span><span>12:35:22 <span>exec</span>    <span>33047</span>                 runc init
</span></span><span><span>12:35:22 <span>exec</span>    <span>33052</span>                 sh
</span></span><span><span>12:35:37 <span>exec</span>    <span>33062</span>                 ps
</span></span></code></pre></td></tr></tbody></table>

从同步打印的结果可以判断， `ps` 和 `forkstat` 所分别输出的 `sh` 或 `ps` 实际上是同一个进程，但由于容器中的进程位于一个单独的 pid namespace 中，它们在容器中拥有另外的 PID，而且它们以为自己是容器中唯一存在的进程，因此 PID 会从 1 开始。

### [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E6%89%BE%E5%88%B0%E8%BF%9B%E7%A8%8B%E6%89%80%E5%B1%9E%E7%9A%84-namespace)[找到进程所属的 namespace](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E6%89%BE%E5%88%B0%E8%BF%9B%E7%A8%8B%E6%89%80%E5%B1%9E%E7%9A%84-namespace)

现在来找出容器所使用的 pid namespace，为此需要调整一下 `ps` 命令的输出格式：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ps -p <span>33052</span> -o pid,pidns
</span></span><span><span>PID      PIDNS  
</span></span><span><span><span>33052</span> <span>4026532395</span>
</span></span></code></pre></td></tr></tbody></table>

PIDNS 即 pid namespace，以上命令可得到 PID 为 33052 的 `sh` 进程属于 4026532395 这个 pid namespace。既然已经有了容器中进程的 PID，实际上我们可以通过宿主机的 `/proc` 文件系统获得该进程所属的所有 namespace：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ll /proc/33052/ns
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:37 cgroup -&gt; <span>'cgroup:[4026531835]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:36 ipc -&gt; <span>'ipc:[4026532394]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:36 mnt -&gt; <span>'mnt:[4026532383]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:36 net -&gt; <span>'net:[4026532397]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:36 pid -&gt; <span>'pid:[4026532395]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:37 pid_for_children -&gt; <span>'pid:[4026532395]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:37 <span>time</span> -&gt; <span>'time:[4026531834]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:37 time_for_children -&gt; <span>'time:[4026531834]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:36 user -&gt; <span>'user:[4026531837]'</span>
</span></span><span><span>lrwxrwxrwx <span>1</span> root root <span>0</span>  7月 <span>21</span> 12:36 uts -&gt; <span>'uts:[4026532393]'</span>
</span></span></code></pre></td></tr></tbody></table>

打印结果展示了一个进程所属的 namespace：

1.  每个 namespace 都是一个软链接，软链接的名称指示了 namespace 的类型，如 cgroup 表示 cgroup namespace， pid 表示 pid namespace。
2.  每个软链接指向该进程所属的真正 namespace 对象，该对象用 `inode` 号码表示，每个 `inode` 号码在宿主系统中都是唯一的。
3.  如果有两个进程的同一类型 namespace 软链接都指向同一个 `inode` ，说明他们属于同一个 namespace。

_实际上所有的进程都会属于至少一个 namespace，Linux 系统在启动时就会为所有类型创建一个默认的 namespace 供进程使用。_

我们也可以尝试在容器内获得 `sh` 所属的 namespace，此时需要在容器内使用 1 这个 PID：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ls -l /proc/1/ns
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 cgroup -&gt; cgroup:<span>[</span>4026531835<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 ipc -&gt; ipc:<span>[</span>4026532394<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 mnt -&gt; mnt:<span>[</span>4026532383<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 net -&gt; net:<span>[</span>4026532397<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 pid -&gt; pid:<span>[</span>4026532395<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 pid_for_children -&gt; pid:<span>[</span>4026532395<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 <span>time</span> -&gt; time:<span>[</span>4026531834<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 time_for_children -&gt; time:<span>[</span>4026531834<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 user -&gt; user:<span>[</span>4026531837<span>]</span>
</span></span><span><span>lrwxrwxrwx    <span>1</span> root     root             <span>0</span> Jul <span>21</span> 04:37 uts -&gt; uts:<span>[</span>4026532393<span>]</span>
</span></span></code></pre></td></tr></tbody></table>

### [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E8%A7%82%E5%AF%9F-namespace-%E4%B8%AD%E7%9A%84%E8%BF%9B%E7%A8%8B)[观察 namespace 中的进程](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E8%A7%82%E5%AF%9F-namespace-%E4%B8%AD%E7%9A%84%E8%BF%9B%E7%A8%8B)

下面我们将从 namespace 的角度，来观测 pid namespace 中的所有进程。Linux 系统并未提供类似的功能，因此需要借助上文安装的 `cinf` 工具来实现。

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cinf -namespace <span>4026532395</span>
</span></span><span><span>
</span></span><span><span> PID    PPID   NAME  CMD  NTHREADS  CGROUPS                                                          STATE
</span></span><span><span>
</span></span><span><span> <span>33052</span>  <span>33052</span>  sh    sh   <span>1</span>         12:devices:/user.slice/mybox                                     S <span>(</span>sleeping<span>)</span>
</span></span><span><span>                                    11:blkio:/user.slice/mybox 10:rdma:/
</span></span><span><span>                                    9:memory:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                    8:net_cls,net_prio:/mybox 7:freezer:/mybox
</span></span><span><span>                                    6:pids:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                    5:cpu,cpuacct:/user.slice/mybox 4:cpuset:/mybox
</span></span><span><span>                                    3:perf_event:/mybox 2:hugetlb:/mybox
</span></span><span><span>                                    1:name<span>=</span>systemd:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                    0::/user.slice/user-0.slice/session-590.scope
</span></span></code></pre></td></tr></tbody></table>

目前这个 namespace 中只有一个进程，这个进程也是我们所创建容器的 `init` 进程。当一个新的容器被创建时，系统将创建一些新的 namespace，容器的 `init` 进程将被加入到这些 namespace。

对于 pid namespace 来说，容器中运行的所有进程只能看到位于同一 pid namespace 即 `pid:[4026532395]` 中的其他进程。`sh` 进程在容器中被认为是系统运行的第一个进程，PID 为 1，但在宿主机中只是一个 PID 为 33052 的普通进程，同一个进程在不同 namespace 中拥有不同的 PID，这就是 pid namespace 的作用。某种程度上，容器就意味着一个新的 namespace 集合。

## [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84%E8%BF%9B%E7%A8%8B)[在容器中创建新的进程](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84%E8%BF%9B%E7%A8%8B)

创建一个新的终端窗口，在已运行的容器中运行一个新的进程：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc <span>exec</span> mybox /bin/top -b
</span></span></code></pre></td></tr></tbody></table>

从 `forkstat` 窗口中，我们可以看到新创建进程的 PID：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>Time     Event     PID Info   Duration Process
</span></span><span><span>12:40:23 <span>exec</span>    <span>33132</span>                 runc <span>exec</span> mybox /bin/top -b
</span></span><span><span>12:40:23 <span>exec</span>    <span>33140</span>                 runc init
</span></span><span><span>12:40:23 <span>exec</span>    <span>33140</span>                 runc init
</span></span><span><span>12:40:23 <span>exec</span>    <span>33142</span>                 /bin/top -b
</span></span></code></pre></td></tr></tbody></table>

实际上还有更直接的方式从宿主机中查看容器中运行的进程，我们可以使用 `runC` 提供的 `ps` 子命令：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ runc ps mybox
</span></span><span><span>UID          PID    PPID  C STIME TTY          TIME CMD
</span></span><span><span>root       <span>33052</span>   <span>33040</span>  <span>0</span> 12:35 pts/0    00:00:00 sh
</span></span><span><span>root       <span>33142</span>   <span>33132</span>  <span>0</span> 12:40 pts/1    00:00:00 /bin/top -b
</span></span></code></pre></td></tr></tbody></table>

接下来依然使用 `cinf` 来找出新创建进程所属的 namespace：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cinf --pid <span>33142</span>
</span></span><span><span>
</span></span><span><span> NAMESPACE   TYPE
</span></span><span><span>
</span></span><span><span> <span>4026532383</span>  mnt
</span></span><span><span> <span>4026532393</span>  uts
</span></span><span><span> <span>4026532394</span>  ipc
</span></span><span><span> <span>4026532395</span>  pid
</span></span><span><span> <span>4026532397</span>  net
</span></span><span><span> <span>4026531837</span>  user
</span></span></code></pre></td></tr></tbody></table>

从结果来看，并没有新的命名空间被创建，32608 进程的 namespace 和 mybox 容器的 `init` 进程- `sh` 所属的 namespace 是完全相同的。也就是说，在容器中创建一个新的进程，只是将这个进程加入到了容器 `init` 进程所属的 namespace。

下面来列出 4026532395 namespace 所拥有的所有进程：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cinf --namespace <span>4026532395</span>
</span></span><span><span>
</span></span><span><span> PID    PPID   NAME  CMD     NTHREADS  CGROUPS                                                          STATE
</span></span><span><span>
</span></span><span><span> <span>33052</span>  <span>33040</span>  sh    sh      <span>1</span>         12:devices:/user.slice/mybox                                     S <span>(</span>sleeping<span>)</span>
</span></span><span><span>                                       11:blkio:/user.slice/mybox 10:rdma:/
</span></span><span><span>                                       9:memory:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                       8:net_cls,net_prio:/mybox 7:freezer:/mybox
</span></span><span><span>                                       6:pids:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                       5:cpu,cpuacct:/user.slice/mybox 4:cpuset:/mybox
</span></span><span><span>                                       3:perf_event:/mybox 2:hugetlb:/mybox
</span></span><span><span>                                       1:name<span>=</span>systemd:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                       0::/user.slice/user-0.slice/session-590.scope
</span></span><span><span> <span>33142</span>  <span>33132</span>  top   top -b  <span>1</span>         12:devices:/user.slice/mybox                                     S <span>(</span>sleeping<span>)</span>
</span></span><span><span>                                       11:blkio:/user.slice/mybox 10:rdma:/
</span></span><span><span>                                       9:memory:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                       8:net_cls,net_prio:/mybox 7:freezer:/mybox
</span></span><span><span>                                       6:pids:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                       5:cpu,cpuacct:/user.slice/mybox 4:cpuset:/mybox
</span></span><span><span>                                       3:perf_event:/mybox 2:hugetlb:/mybox
</span></span><span><span>                                       1:name<span>=</span>systemd:/user.slice/user-0.slice/session-590.scope/mybox
</span></span><span><span>                                       0::/user.slice/user-0.slice/session-590.scope
</span></span></code></pre></td></tr></tbody></table>

如果在容器内运行 `ps -ef` ，我们也能看到这些进程，由于 pid namespace 的原因它们的 PID 将会有所不同：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>PID   USER     TIME  COMMAND
</span></span><span><span>    <span>1</span> root      0:00 sh
</span></span><span><span>   <span>19</span> root      0:00 top -b
</span></span><span><span>   <span>20</span> root      0:00 ps -ef
</span></span></code></pre></td></tr></tbody></table>

现在我们知道，`docker/runc exec` 实际上就是在已创建容器的 namespace 中运行一个新的进程。

## [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E6%80%BB%E7%BB%93)[总结](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E6%80%BB%E7%BB%93)

运行一个容器时，将创建一些新的 namespace， `init` 进程将被加入到这些 namespace；在一个容器中运行一个新进程时，新进程将加入创建容器时所创建的 namespace。

实际上创建容器时新建 namespace 这种行为是可以改变的，我们可以指定新建的容器使用已有的 namespace。

## [](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/container-fundamentals-process-isolation-using-namespace/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Understand Container 1: Linux Namespaces](https://pierrchen.blogspot.com/2018/04/linux-namespaces-in-action-using-runc.html)
-   [Creating an OCI Bundle](https://github.com/opencontainers/runc/blob/master/README.md#creating-an-oci-bundle)
-   [Monitoring new process creation](https://jarekprzygodzki.dev/post/monitor-new-process-creation/)
-   [Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)

updatedupdated2022-07-172022-07-17

[容器](https://waynerv.com/tags/%E5%AE%B9%E5%99%A8/) [Namespace](https://waynerv.com/tags/namespace/)