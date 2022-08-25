## 容器技术原理(五)：文件系统的隔离和共享

 2021.7.27  [Linux](https://waynerv.com/categories/linux/)  3561  8 分钟  5

## 目录

1.  [背景知识](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E8%83%8C%E6%99%AF%E7%9F%A5%E8%AF%86)
    1.  [目录 VS 文件系统](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E7%9B%AE%E5%BD%95-vs-%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
    2.  [什么是 `rootfs`](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E4%BB%80%E4%B9%88%E6%98%AF-rootfs)
2.  [mount namespace 的本质和工作方式](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#mount-namespace-%E7%9A%84%E6%9C%AC%E8%B4%A8%E5%92%8C%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F)
    1.  [在容器中创建新的 mount namespace](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84-mount-namespace)
3.  [使用 pivot\_root 或 chroot 切换根目录](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E4%BD%BF%E7%94%A8-pivot_root-%E6%88%96-chroot-%E5%88%87%E6%8D%A2%E6%A0%B9%E7%9B%AE%E5%BD%95)
    1.  [chroot 命令的使用示例](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#chroot-%E5%91%BD%E4%BB%A4%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)
4.  [bind mount ：在宿主机和容器间共享文件](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#bind-mount-%E5%9C%A8%E5%AE%BF%E4%B8%BB%E6%9C%BA%E5%92%8C%E5%AE%B9%E5%99%A8%E9%97%B4%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6)
    1.  [在容器中使用 bind mount](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E4%BD%BF%E7%94%A8-bind-mount)
    2.  [docker volumes 是什么](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#docker-volumes-%E6%98%AF%E4%BB%80%E4%B9%88)
5.  [总结](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E6%80%BB%E7%BB%93)
6.  [参考链接](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

## [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E8%83%8C%E6%99%AF%E7%9F%A5%E8%AF%86)[背景知识](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E8%83%8C%E6%99%AF%E7%9F%A5%E8%AF%86)

Unix 系统中所有可访问的文件都被组织在一个巨大的树状文件层次结构中，这颗文件树的根节点就是 `/` 目录。这些文件可以分散保存在不同的设备中，前提是我们使用 `mount` 系统调用将这些设备上的文件系统挂载到文件树中。

### [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E7%9B%AE%E5%BD%95-vs-%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)[目录 VS 文件系统](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E7%9B%AE%E5%BD%95-vs-%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)

了解文件系统和目录之间的区别是很重要的。文件系统是存储设备如硬盘的一个部分，它被分配来保存文件数据。在目录上挂载文件系统后，可以访问这部分存储的文件。文件系统被挂载后，对终端用户来说访问起来就像普通目录一样。

我们常常提到 `ext` 、`xfs` 和 `zfs` 是文件系统的类型，不同类型的文件系统存取和管理数据的方式不同，Linux 支持多种类型的文件系统。

### [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E4%BB%80%E4%B9%88%E6%98%AF-rootfs)[什么是 `rootfs`](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E4%BB%80%E4%B9%88%E6%98%AF-rootfs)

`rootfs` （Root Filesystem）是分层文件树的顶端。它包含对系统运行至关重要的文件和目录，包括设备目录和用于启动系统的程序。`rootfs` 还包含了许多挂载点，其他文件系统可以通过这些挂载点连接到 `rootfs` 的文件树中。`rootfs` 通常由 Linux 发行版提供，一个典型的 `rootfs` 内容如下：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ ls /
</span></span><span><span>boot  etc   home  lib64  mnt  proc  run   srv  tmp  var
</span></span><span><span>bin   dev   lib   media  opt  root  sbin  sys  usr
</span></span></code></pre></td></tr></tbody></table>

系统启动时，初始化进程会将 `rootfs` 挂载到 `/` 目录，之后再挂载其他的文件系统到其子目录中。这期间所有的 `mount` 系统调用都会被记录到初始化进程的 `mount table` 中，所有的进程都有一张独立的 `mount table`，记录于 `/proc/{PID}/mounts` 中。但一般情况下，系统中的所有进程都会直接使用初始化进程的 `mount table`。

## [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#mount-namespace-%E7%9A%84%E6%9C%AC%E8%B4%A8%E5%92%8C%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F)[mount namespace 的本质和工作方式](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:mount-namespace-%E7%9A%84%E6%9C%AC%E8%B4%A8%E5%92%8C%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F)

每个进程可以创建属于自己的 `mount table`，但前提是必须先复制父进程的 `mount table`，之后再调用 `mount` 发生的更改都只会影响当前进程的 `mount table`，这就是 mount namespace 的工作原理。如果多个进程在同一 mount namespace 内，其中一个进程对 `mount table` 的更改对其他进程来说也是可见的。

现在我们来实际查看系统中当前存在的所有 mount namespace，这需要借助之前的文章中介绍过的 `cinf` 工具：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cinf <span>|</span> grep mnt
</span></span><span><span>
</span></span><span><span> NAMESPACE   TYPE  NPROCS  USERS                      CMD
</span></span><span><span> 
</span></span><span><span> <span>4026531840</span>  mnt   <span>130</span>     0,32,70,81,89,998,999      /sbin/init splash
</span></span><span><span> <span>4026531860</span>  mnt   <span>1</span>       <span>0</span>
</span></span><span><span> <span>4026532320</span>  mnt   <span>1</span>       <span>997</span>                        /usr/sbin/chronyd
</span></span><span><span> <span>4026532389</span>  mnt   <span>1</span>       <span>0</span>                          sh
</span></span></code></pre></td></tr></tbody></table>

命令的输出经过一定处理以展示更清晰的信息，从进程数量（NPROCS）可以看到绝大部分进程都位于由初始化进程 `/sbin/init` 创建的 `4026531840` mount namspace 中，一般情况下新的进程并不会创建新的 mount namespace，如果你打印这些进程的 `mount table` 即 `/proc/{PID}/mounts` ，会发现输出结果都是相同的。

创建一个新的 mount namespace 时，会在新的 mount namspace 中创建一个来自父命名空间的挂载点副本。我们将通过 `unshare -m` 创建一个新的 mount namespace 来验证，还将要用到我们在前面的文章中通过 Docker 镜像创建的 `filesystem bundle`：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>cd</span> /mycontainer/rootfs
</span></span><span><span><span>PS1</span><span>=</span><span>'\u@new-mnt$ '</span> unshare -Umr
</span></span></code></pre></td></tr></tbody></table>

执行该命令后我们进入到了一个新的 mount namespace 中，但依然能看到宿主机中的所有挂载点：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ df -h
</span></span><span><span>文件系统        容量  已用  可用 已用% 挂载点
</span></span><span><span>/dev/sdb3       119G   28G   86G   25% /
</span></span><span><span>tmpfs           7.8G     <span>0</span>  7.8G    0% /dev/shm
</span></span><span><span>tmpfs           1.6G  1.9M  1.6G    1% /run
</span></span><span><span>tmpfs           5.0M  4.0K  5.0M    1% /run/lock
</span></span><span><span>tmpfs           1.6G   84K  1.6G    1% /run/user/121
</span></span><span><span>tmpfs           1.6G   68K  1.6G    1% /run/user/0
</span></span><span><span>tmpfs           4.0M     <span>0</span>  4.0M    0% /sys/fs/cgroup
</span></span><span><span>/dev/sdb2        94G  5.8G   83G    7% /home
</span></span><span><span>/dev/sda2        96M   30M   67M   31% /boot/efi
</span></span></code></pre></td></tr></tbody></table>

反过来当我们在新的 mount namespace 执行新的挂载，从宿主机看不到相应的记录：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># in new mount namspace</span>
</span></span><span><span>$ mount -t tmpfs tmpfs /mnt
</span></span><span><span>$ findmnt <span>|</span> grep mnt
</span></span><span><span>└─/mnt
</span></span><span><span>tmpfs                  tmpfs           rw,relatime,inode64
</span></span><span><span><span># in host</span>
</span></span><span><span>$ findmnt <span>|</span> grep mnt
</span></span></code></pre></td></tr></tbody></table>

这就是 mount namespace 的工作方式。

### [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84-mount-namespace)[在容器中创建新的 mount namespace](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84-mount-namespace)

现在创建一个新的容器来看看 mount namespace 的变化：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ <span>cd</span> /mycontainer
</span></span><span><span>$ runc run mybox
</span></span></code></pre></td></tr></tbody></table>

在一个新的窗口中，我们从宿主机环境使用 `cinf` 查询 namespace：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cinf <span>|</span> grep mnt
</span></span><span><span> <span>4026531840</span>  mnt   <span>134</span>     0,32,70,81,89,998,999      /usr/lib/systemd/systemd --swi
</span></span><span><span> <span>4026531860</span>  mnt   <span>1</span>       <span>0</span>
</span></span><span><span> <span>4026532320</span>  mnt   <span>1</span>       <span>997</span>                        /usr/sbin/chronyd
</span></span><span><span> <span>4026532325</span>  mnt   <span>1</span>       <span>0</span>                          sh
</span></span><span><span> <span>4026532389</span>  mnt   <span>1</span>       <span>0</span>                          sh
</span></span></code></pre></td></tr></tbody></table>

会发现增加了一个新的 mount namespace `4026532325`，创建该 mount namespace 的进程正是容器的 `init` 进程：

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
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ cinf --namespace <span>4026532325</span>
</span></span><span><span>
</span></span><span><span> PID   PPID  NAME  CMD  NTHREADS  CGROUPS                                                           STATE
</span></span><span><span>
</span></span><span><span> <span>7656</span>  <span>7646</span>  sh    sh   <span>1</span>         11:blkio:/mybox 10:devices:/mybox 9:pids:/mybox                   S <span>(</span>sleeping<span>)</span>
</span></span><span><span>                                  8:cpu,cpuacct:/mybox 7:net_cls,net_prio:/mybox
</span></span><span><span>                                  6:freezer:/mybox 5:memory:/mybox 4:cpuset:/mybox
</span></span><span><span>                                  3:hugetlb:/mybox 2:perf_event:/mybox
</span></span><span><span>                                  1:name<span>=</span>systemd:/user.slice/user-0.slice/session-1231.scope/mybox
</span></span><span><span>
</span></span><span><span>$ runc ps mybox
</span></span><span><span>UID        PID  PPID  C STIME TTY          TIME CMD
</span></span><span><span>root      <span>7656</span>  <span>7646</span>  <span>0</span> 14:39 pts/0    00:00:00 sh
</span></span></code></pre></td></tr></tbody></table>

需要注意的是，创建新的 mount namspace 并不会创建一个全新的 `mount table`，而是在父进程的 mount namspace 的副本上进行变更，因此在创建容器时，新的容器容器拥有宿主机的全部挂载点，这显然与我们的预期是不符的，我们希望使用 `filesystem bundle` 中的 `rootfs` 为容器建立一个隔离的文件系统，这要求我们在新的 mount namespace 中 `unmount` 当前 `rootfs`，并将 `filesystem bundle` 中的 `rootfs` `mount` 到 `/` 目录，我们可以分别在容器和宿主机中使用 `ls -id` 打印 `/` 目录的 `inode` 号码予以验证：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># 在容器中打印 / 目录</span>
</span></span><span><span>$ ls -id /
</span></span><span><span><span>7077890</span> /
</span></span><span><span><span># 在宿主机中打印 filesystem bundle 中 rootfs 所在目录</span>
</span></span><span><span>$ ls -id /mycontainer/rootfs
</span></span><span><span><span>7077890</span> /mycontainer/rootfs
</span></span></code></pre></td></tr></tbody></table>

通过 `mount` 无法实现这一点，因为我们无法使所有的进程停止使用当前的 `rootfs`，当前的 `rootfs` 一定处于使用状态而不能被 `unmount`。但可以采取另一种途径：使用 `pivot_root` 系统调用，它允许我们将 `rootfs` 重新挂载到一个非 `/` 的位置，同时在 `/` 目录上挂载一个新的目录，并将所有当前进程的根目录切换为新目录。之后我们可以顺利 `unmount` 原来的 `rootfs`。

## [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E4%BD%BF%E7%94%A8-pivot_root-%E6%88%96-chroot-%E5%88%87%E6%8D%A2%E6%A0%B9%E7%9B%AE%E5%BD%95)[使用 pivot\_root 或 chroot 切换根目录](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E4%BD%BF%E7%94%A8-pivot_root-%E6%88%96-chroot-%E5%88%87%E6%8D%A2%E6%A0%B9%E7%9B%AE%E5%BD%95)

`pivot_root` 是由 Linux 提供的一种系统调用，它能够将一个 mount namespace 中的所有进程的根目录和当前工作目录切换到一个新的目录。`pivot_root` 的主要用途是在系统启动时，先挂载一个临时的 `rootfs` 完成特定功能，然后再切换到真正的 `rootfs`。

创建容器的过程中，在创建新的 mount namespace 之后，我们可以通过 `pivot_root()` 将容器内进程的根目录切换到 `filesystem bundle` 中的 `rootfs` 所在目录。

### [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#chroot-%E5%91%BD%E4%BB%A4%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)[chroot 命令的使用示例](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:chroot-%E5%91%BD%E4%BB%A4%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)

除了 `pivot_root` ，Linux 还提供了 `chroot` 系统调用能够将当前进程的根目录更改为一个新的目录，新的根目录还将被当前进程的所有子进程所继承。

上文提到的 `pivot_root` 和 `chroot` 都是由 Linux 内核提供的系统调用，它们分别都有命令行程序实现了对系统调用的简单封装，接下来我们以 `chroot` 命令为例：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># 切换到有效的 filesystem bundle 目录</span>
</span></span><span><span>$ <span>cd</span> /mycontainer
</span></span><span><span>$ chroot rootfs /bin/sh
</span></span><span><span><span># 进入到一个新的shell中</span>
</span></span><span><span>$ ls /
</span></span><span><span>bin   dev   etc   home  old   proc  root  sys   tmp   usr   var
</span></span></code></pre></td></tr></tbody></table>

在新的 shell 程序中，执行 `ls` 返回的是 `/mycontainer/rootfs` 目录下的文件内容，而不是宿主机根目录下的内容。通过 `chroot` 命令，在不需要 `mount namespace` 的情况下我们也实现切换容器内进程根目录的效果。

相比 `chroot`，`pivot_root` 系统调用配合 mount namspace 更加安全，容器运行时会优先使用这种方式，但 `chroot` 也是一种可选的方式。在 `runC` 的实现中有以下（Golang）代码片段：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="go"><span><span><span>if</span> <span>config</span><span>.</span><span>NoPivotRoot</span> <span>{</span>
</span></span><span><span><span>err</span> <span>=</span> <span>msMoveRoot</span><span>(</span><span>config</span><span>.</span><span>Rootfs</span><span>)</span>
</span></span><span><span><span>}</span> <span>else</span> <span>if</span> <span>config</span><span>.</span><span>Namespaces</span><span>.</span><span>Contains</span><span>(</span><span>configs</span><span>.</span><span>NEWNS</span><span>)</span> <span>{</span>
</span></span><span><span><span>err</span> <span>=</span> <span>pivotRoot</span><span>(</span><span>config</span><span>.</span><span>Rootfs</span><span>)</span>
</span></span><span><span><span>}</span> <span>else</span> <span>{</span>
</span></span><span><span><span>err</span> <span>=</span> <span>chroot</span><span>()</span>
</span></span><span><span><span>}</span>
</span></span></code></pre></td></tr></tbody></table>

忽略第一个由用户指定的「不切换根目录」的选择分支，其余分支代表两种切换根目录的方式：

-   如果创建了新的 mount namespace ，将使用 `pivot_root` 系统调用。
-   如果没有创建新的 mount namespace，直接使用 `chroot` 。

## [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#bind-mount-%E5%9C%A8%E5%AE%BF%E4%B8%BB%E6%9C%BA%E5%92%8C%E5%AE%B9%E5%99%A8%E9%97%B4%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6)[bind mount ：在宿主机和容器间共享文件](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:bind-mount-%E5%9C%A8%E5%AE%BF%E4%B8%BB%E6%9C%BA%E5%92%8C%E5%AE%B9%E5%99%A8%E9%97%B4%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6)

建立隔离的文件系统之后，我们还需要一种机制从容器访问宿主机的部分文件系统，或者将容器运行过程产生的数据持久化到宿主机中。

bind mount （绑定挂载）是由 Linux 提供的一种挂载类型，它能够将一个文件或目录再次挂载到一个新的目标路径，挂载后从新旧两个路径都能访问到原来的数据，从两个路径对数据的修改也都会生效，目标路径的原有内容将会被隐藏。以如下目录结构为例：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ tree .
</span></span><span><span>.
</span></span><span><span>├── A
</span></span><span><span>│&nbsp;&nbsp; ├── a
</span></span><span><span>│&nbsp;&nbsp; └── a.conf
</span></span><span><span>└── B
</span></span><span><span> &nbsp;&nbsp; ├── b
</span></span><span><span> &nbsp;&nbsp; └── b.conf
</span></span></code></pre></td></tr></tbody></table>

通过如下命令将 A 绑定挂载到 B 目录：

之后从 A 和 B 目录都可以访问原 A 目录的文件内容，而 B 目录的原内容将被隐藏：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span><span>8
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ tree .
</span></span><span><span>.
</span></span><span><span>├── A
</span></span><span><span>│&nbsp;&nbsp; ├── a
</span></span><span><span>│&nbsp;&nbsp; └── a.conf
</span></span><span><span>└── B
</span></span><span><span> &nbsp;&nbsp; ├── a
</span></span><span><span> &nbsp;&nbsp; └── a.conf
</span></span></code></pre></td></tr></tbody></table>

我们可以使用 bind mount 在宿主机和容器之间共享文件，将宿主机中的目录甚至是块设备挂载到容器中。

### [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E4%BD%BF%E7%94%A8-bind-mount)[在容器中使用 bind mount](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E5%9C%A8%E5%AE%B9%E5%99%A8%E4%B8%AD%E4%BD%BF%E7%94%A8-bind-mount)

下面我们将在容器建立一个连接到宿主机的 bind mount。容器运行时的实现方式是修改 `filesystem bundle` 中的 `config.json`，在 JSON 对象的 `mounts` 列表中加入以下对象：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span></code></pre></td><td><pre tabindex="0"><code data-lang="json"><span><span><span>{</span>
</span></span><span><span>        <span>"destination"</span><span>:</span> <span>"/host_dir"</span><span>,</span>
</span></span><span><span>        <span>"type"</span><span>:</span> <span>"bind"</span><span>,</span>
</span></span><span><span>        <span>"source"</span><span>:</span> <span>"/mycontainer/host"</span><span>,</span>
</span></span><span><span>        <span>"options"</span><span>:</span> <span>[</span><span>"bind"</span><span>]</span>
</span></span><span><span><span>}</span>
</span></span></code></pre></td></tr></tbody></table>

这告诉容器运行时，将宿主机中的 `/mycontainer/host` 目录（如果使用相对路径则是相对于 `filesystem bundle` 的目录即 `host`），挂载到容器中 `rootfs` 的 `/host_dir` 目录。宿主机中的 `host` 目录必须提前存在，而容器中的 `host_dir` 不存在时将由容器运行时自动创建。现在我们在宿主机中创建 `host` 目录并填充一个文件：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span>$ <span>cd</span> /mycontainer
</span></span><span><span>$ mkdir host
</span></span><span><span>$ touch mkdir/hello
</span></span></code></pre></td></tr></tbody></table>

然后运行容器，我们将在容器的 `/host_dir` 目录中看到宿主机 `/mycontainer/host` 目录的内容，在容器内修改该目录内的文件也将在宿主机可见：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span><span>6
</span><span>7
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># in container</span>
</span></span><span><span>$ ls /host_dir
</span></span><span><span>hello
</span></span><span><span>$ touch /host_dir/world
</span></span><span><span><span># in host</span>
</span></span><span><span>$ ls /mycontainer/host
</span></span><span><span>hello  world
</span></span></code></pre></td></tr></tbody></table>

而且，由于绑定挂载的过程发生在容器的 mount namespace 中，宿主机并不知道该挂载点的存在：

<table><tbody><tr><td><pre tabindex="0"><code><span>1
</span><span>2
</span><span>3
</span><span>4
</span><span>5
</span></code></pre></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span># in container</span>
</span></span><span><span>$ cat /proc/self/mounts <span>|</span> grep host_dir
</span></span><span><span>/dev/sdb3 /host_dir ext4 rw,relatime,errors<span>=</span>remount-ro <span>0</span> <span>0</span>
</span></span><span><span><span># in host</span>
</span></span><span><span>$ cat /proc/self/mounts <span>|</span> grep host_dir
</span></span></code></pre></td></tr></tbody></table>

在系列的第一篇文章中，我们提到绑定挂载的挂载点位于容器的可写层中，虽然容器删除后整个可写层将被删除，但容器运行过程中的写入数据依然会保留在宿主机的挂载路径，因此可通过该途径持久化容器中的数据。

### [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#docker-volumes-%E6%98%AF%E4%BB%80%E4%B9%88)[docker volumes 是什么](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:docker-volumes-%E6%98%AF%E4%BB%80%E4%B9%88)

Dcoekr 提供了 Volumes（该术语并不存在于 OCI 规范中）来将容器中的数据持久化，底层的实现方式也是使用 bind mount，将宿主机中的路径绑定挂载到容器中，相比之下 Docker 提供了更友好的命令行接口，此外还提供了 `named volume`，（在 Linux 系统中）本质是不需要在宿主机中指定已存在的目录，而是由 Docker 来管理，在其宿主机的数据目录中建立一个单独的目录。

## [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E6%80%BB%E7%BB%93)[总结](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E6%80%BB%E7%BB%93)

这篇文章我们讨论了和容器文件系统有关的几个方面：

-   使用 mount namespace 在容器中建立单独的文件系统环境，但此时并未与宿主机完全隔离。
-   使用 `pivot_root` 将容器中的根目录切换到镜像提供的 `rootfs` 中，使其无法访问宿主机的其他路径而实现隔离。
-   使用 bind mount 在宿主机和容器间共享数据或将容器运行时生成的数据持久化。

## [](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)[参考链接](https://waynerv.com/posts/container-fundamentals-filesystem-isolation-and-sharing/#contents:%E5%8F%82%E8%80%83%E9%93%BE%E6%8E%A5)

-   [Understand Container 4: Mount and Jail](https://pierrchen.blogspot.com/2018/05/understand-container-4.html)
-   [mount(8) — Linux manual page](https://man7.org/linux/man-pages/man8/mount.8.html)
-   [Building a container by hand using namespaces: The mount namespace](https://www.redhat.com/sysadmin/mount-namespaces)
-   [Mounting the Root Filesystem](https://www.halolinux.us/kernel-reference/mounting-the-root-filesystem.html)
-   [File systems](https://www.ibm.com/docs/en/aix/7.1?topic=management-file-systems)
-   [pivot\_root(2) — Linux manual page](https://man7.org/linux/man-pages/man2/pivot_root.2.html)
-   [DIfference between chroot & pivot\_root](https://lists.linuxcontainers.org/pipermail/lxc-devel/2011-September/002065.html)
-   [What is the purpose of pivot\_root system call in Linux?](https://www.quora.com/What-is-the-purpose-of-pivot_root-system-call-in-Linux)
-   [What is a bind mount?](https://unix.stackexchange.com/questions/198590/what-is-a-bind-mount)

updatedupdated2022-07-172022-07-17

[容器](https://waynerv.com/tags/%E5%AE%B9%E5%99%A8/) [Namespace](https://waynerv.com/tags/namespace/) [文件系统](https://waynerv.com/tags/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F/)