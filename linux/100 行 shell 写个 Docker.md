作者：vivo 互联网运维团队- Hou Dengfeng

本文主要介绍使用shell实现一个简易的Docker。

一、目的

在初接触Docker的时候，我们必须要了解的几个概念就是Cgroup、Namespace、RootFs，如果本身对虚拟化的发展没有深入的了解，那么很难对这几个概念有深入的理解，本文的目的就是通过在操作系统中以交互式的方式去理解，Cgroup/Namespace/Rootfs到底实现了什么，能做到哪些事情，然后通过shell这种直观的命令行方式把我们的理解组合起来，去模仿Docker实现一个缩减的版本。

二、技术拆解

2.1 Namespace

**2.1.1 简介**

Linux Namespace是Linux提供的一种内核级别环境隔离的方法。学习过Linux的同学应该对chroot命令比较熟悉（通过修改根目录把用户限制在一个特定目录下），chroot提供了一种简单的隔离模式：chroot内部的文件系统无法访问外部的内容。Linux Namespace在此基础上，提供了对UTS、IPC、mount、PID、network、User等的隔离机制。Namespace是对全局系统资源的一种封装隔离，使得处于不同namespace的进程拥有独立的全局系统资源，改变一个namespace中的系统资源只会影响当前namespace里的进程，对其他namespace中的进程没有影响。

Linux Namespace有如下种类：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt4GpmhdsA6Px0Y1yiayw4URVOfT5ZVbfH1pdQ1ejb0so5ShiaqwHnrSianjKicvCJjwBCqBONmbXhIscw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**2.1.2 Namespace相关系统调用**

amespace相关的系统调用有3个，分别是clone(),setns(),unshare()。

-   **clone:** 创建一个新的进程并把这个新进程放到新的namespace中
    
-   **setns:** 将当前进程加入到已有的namespace中
    
-   **unshare:** 使当前进程退出指定类型的namespace，并加入到新创建的namespace中
    

**2.1.3 查看进程所属Namespace**

上面的概念都比较抽象，我们来看看在Linux系统中怎么样去get namespace。

系统中的每个进程都有/proc/\[pid\]/ns/这样一个目录，里面包含了这个进程所属namespace的信息，里面每个文件的描述符都可以用来作为setns函数(2.1.2)的fd参数。

```
#查看当前bash进程关联的Namespace
```

**2.1.4 相关命令及操作示例**

本节会用UTS/IPC/NET 3个Namespace作为示例演示如何在linux系统中创建Namespace，并介绍相关命令。

2.1.4.1 IPC Namespace

IPC namespace用来隔离System V IPC objects和POSIX message queues。其中System V IPC objects包含消息列表Message queues、信号量Semaphore sets和共享内存Shared memory segments。为了展现区分IPC Namespace我们这里会使用到ipc相关命令：

```
#    nsenter: 加入指定进程的指定类型的namespace中，然后执行参数中指定的命令。
```

下面将以消息队列为例，演示一下隔离效果，为了使演示更直观，我们在创建新的ipc namespace的时候，同时也创建新的uts namespace，然后为新的uts namespace设置新hostname，这样就能通过shell提示符一眼看出这是属于新的namespace的bash。示例中我们用两个shell来展示：

shell A

```
#查看当前shell的uts / ipc namespace number
```

shell B

```
#确认当前shell和shell A属于相同Namespace
```

2.1.4.2 Net Namespace

Network namespace用来隔离网络设备, IP地址, 端口等. 每个namespace将会有自己独立的网络栈，路由表，防火墙规则，socket等。每个新的network namespace默认有一个本地环回接口，除了lo接口外，所有的其他网络设备（物理/虚拟网络接口，网桥等）只能属于一个network namespace。每个socket也只能属于一个network namespace。当新的network namespace被创建时，lo接口默认是关闭的，需要自己手动启动起。标记为"local devices"的设备不能从一个namespace移动到另一个namespace，比如loopback, bridge, ppp等，我们可以通过ethtool -k命令来查看设备的netns-local属性。

我们使用以下命令来创建net namespace。

```
相关命令：
```

下面使用ip netns来演示创建net Namespace。  

shell A  

```
#创建一对网卡，分别命名为veth0_11/veth1_11
```

shell B

```
#在shell B中我们同样切换到netns r2中进行配置
```

示意图

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

2.2 Cgroup

**2.2.1 简介**

Cgroup和namespace类似，也是将进程进行分组，但它的目的和namespace不一样，namespace是为了隔离进程组之间的资源，而cgroup是为了对一组进程进行统一的资源监控和限制。

**Cgroup作用：**

1.  **资源限制(Resource limiting):** Cgroups可以对进程组使用的资源总额进行限制。如对特定的进程进行内存使用上限限制，当超出上限时，会触发OOM。
    
2.  **优先级分配(Prioritization):** 通过分配的CPU时间片数量及硬盘IO带宽大小，实际上就相当于控制了进程运行的优先级。
    
3.  **资源统计（Accounting):** Cgroups可以统计系统的资源使用量，如CPU使用时长、内存用量等等，这个功能非常适用于计费。
    
4.  **进程控制（Control）：**Cgroups可以对进程组执行挂起、恢复等操作。
    

**Cgroups的组成：**

1.  **task:** 在Cgroups中，task就是系统的一个进程。
    
2.  **cgroup:** Cgroups中的资源控制都以cgroup为单位实现的。cgroup表示按照某种资源控制标准划分而成的任务组，包含一个或多个子系统。一个任务可以加入某个cgroup，也可以从某个cgroup迁移到另外一个cgroup。
    
3.  **subsystem**:**** 一个subsystem就是一个内核模块，被关联到一颗cgroup树之后，就会在树的每个节点（进程组）上做具体的操作。subsystem经常被称作"resource controller"，因为它主要被用来调度或者限制每个进程组的资源，但是这个说法不完全准确，因为有时我们将进程分组只是为了做一些监控，观察一下他们的状态，比如perf\_event subsystem。到目前为止，Linux支持13种subsystem（Cgroup v1），比如限制CPU的使用时间，限制使用的内存，统计CPU的使用情况，冻结和恢复一组进程等。
    
4.  **hierarchy****:** 一个hierarchy可以理解为一棵cgroup树，树的每个节点就是一个进程组，每棵树都会与零到多个subsystem关联。在一颗树里面，会包含Linux系统中的所有进程，但每个进程只能属于一个节点（进程组）。系统中可以有很多颗cgroup树，每棵树都和不同的subsystem关联，一个进程可以属于多颗树，即一个进程可以属于多个进程组，只是这些进程组和不同的subsystem关联。如果不考虑不与任何subsystem关联的情况（systemd就属于这种情况），Linux里面最多可以建13颗cgroup树，每棵树关联一个subsystem，当然也可以只建一棵树，然后让这棵树关联所有的subsystem。当一颗cgroup树不和任何subsystem关联的时候，意味着这棵树只是将进程进行分组，至于要在分组的基础上做些什么，将由应用程序自己决定，systemd就是一个这样的例子。
    

**2.2.2 查看Cgroup信息**

查看当前系统支持的subsystem

```
#通过/proc/cgroups查看当前系统支持哪些subsystem
```

查看进程所属cgroup

```
#查看当前shell进程所属的cgroup
```

**2.2.3 相关命令**

使用cgroup

cgroup相关的所有操作都是基于内核中的cgroup virtual filesystem，使用cgroup很简单，挂载这个文件系统就可以了。一般情况下都是挂载到/sys/fs/cgroup目录下，当然挂载到其它任何目录都没关系。

查看下当前系统cgroup挂载情况。

```
#过滤系统挂载可以查看cgroup
```

除了mount命令之外我们还可以使用以下命令对cgroup进行创建、属性设置等操作，这也是我们后面脚本中用于创建和管理cgroup的命令。

```
# Centos操作系统可以通过yum install cgroup-tools 来安装以下命令
```

2.3 Rootfs

**2.3.1 简介**

Rootfs 是 Docker 容器在启动时内部进程可见的文件系统，即 Docker容器的根目录。rootfs 通常包含一个操作系统运行所需的文件系统，例如可能包含典型的类 Unix 操作系统中的目录系统，如 /dev、/proc、/bin、/etc、/lib、/usr、/tmp 及运行 Docker 容器所需的配置文件、工具等。

就像Linux启动会先用只读模式挂载rootfs，运行完完整性检查之后，再切换成读写模式一样。Docker deamon为container挂载rootfs时，也会先挂载为只读模式，但是与Linux做法不同的是，在挂载完只读的rootfs之后，Docker deamon会利用联合挂载技术（Union Mount）在已有的rootfs上再挂一个读写层。container在运行过程中文件系统发生的变化只会写到读写层，并通过whiteout技术隐藏只读层中的旧版本文件。

Docker支持不同的存储驱动，包括 aufs、devicemapper、overlay2、zfs 和 vfs 等，目前在 Docker 中，overlay2 取代了 aufs 成为了推荐的存储驱动。

**2.3.2 overlayfs**

overlayFS是联合挂载技术的一种实现。除了overlayFS以外还有aufs，VFS，Brtfs，device mapper等技术。虽然实现细节不同，但是他们做的事情都是相同的。Linux内核为Docker提供的overalyFS驱动有2种：overlay2和overlay，overlay2是相对于overlay的一种改进，在inode利用率方面比overlay更有效。

overlayfs通过三个目录来实现：lower目录、upper目录、以及work目录。三种目录合并出来的目录称为merged目录。

-   **lower：**可以是多个，是处于最底层的目录，作为只读层。
    
-   **upper：**只有一个，作为读写层。
    
-   **work：**为工作基础目录，挂载后内容会被清空，且在使用过程中其内容用户不可见。
    
-   **merged：**为最后联合挂载完成给用户呈现的统一视图，也就是说merged目录里面本身并没有任何实体文件，给我们展示的只是参与联合挂载的目录里面文件而已，真正的文件还是在lower和upper中。所以，在merged目录下编辑文件，或者直接编辑lower或upper目录里面的文件都会影响到merged里面的视图展示。
    

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2.3.3 文件规则**

merged层目录会显示离它最近层的文件。层级关系中upperdir比lowerdir更靠近merged层，而多个lowerdir的情况下，写的越靠前的目录离merged层目录越近。相同文件名的文件会依照层级规则进行“覆盖”。

**2.3.4 overlayFS如何工作**

-   **读：**  
    如果文件在容器层（upperdir），直接读取文件；  
    如果文件不在容器层（upperdir），则从镜像层（lowerdir）读取；
    
-   **写：**  
    **①首次写入：** 如果在upperdir中不存在，overlay执行cow操作，把文件从lowdir拷贝到upperdir，由于overlayfs是文件级别的（即使文件只有很少的一点修改，也会产生的cow的行为），后续对同一文件的在此写入操作将对已经复制到容器的文件的副本进行操作。值得注意的是，cow操作只发生在文件首次写入，以后都是只修改副本。  
    **②删除文件和目录：** 当文件在容器被删除时，在容器层（upperdir）创建whiteout文件，镜像层(lowerdir)的文件是不会被删除的，因为他们是只读的，但whiteout文件会阻止他们显示。
    

**2.3.5 在系统里创建overlayfs**

shell

```
# 创建所需的目录
```

三 、Bocker

3.1 功能演示

第二部分中我们对Namespace，cgroup，overlayfs有了一定的了解，接下来我们通过一个脚本来实现个建议的Docker。脚本源自于[https://github.com/p8952/bocker](https://github.com/p8952/bocker)，我做了image/pull/存储驱动的部分修改，下面先看下脚本完成后的示例：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

3.2 完整脚本

脚本一共用130行代码，完成了上面的功能，也算符合我们此次的标题了。为了大家可以更深入的理解脚本内容，这里就不再对脚本进行拆分讲解，以下是完整脚本。

```
#!/usr/bin/env bash
```

README

**Bocker**

使用100行bash实现一个docker，本脚本是依据bocker实现，更换了存储驱动，完善了pull等功能。

**前置条件**

为了脚本能够正常运行，机器上需要具备以下组件：

-   overlayfs
    
-   iproute2
    
-   iptables
    
-   libcgroup-tools
    
-   util-linux >= 2.25.2
    
-   coreutils >= 7.5
    

大部分功能在centos7上都是满足的，overlayfs可以通过modprobe overlay挂载。

另外你可能还要做以下设置：

-   创建bocker运行目录 /var/lib/bocker/overlay,/var/lib/bocker/containers
    
-   创建一个IP地址为 172.18.0.1/24 的桥接网卡 br1
    
-   确认开启IP转发 /proc/sys/net/ipv4/ip\_forward = 1
    
-   创建iptables规则将桥接网络流量转发至物理网卡，示例：iptables -t nat -A POSTROUTING -s 172.18.0.0/24 -o eth0 -j MASQUERADE
    

**实现的功能**

-   docker build +
    
-   docker pull
    
-   docker images
    
-   docker ps
    
-   docker **run**
    
-   docker exec
    
-   docker logs
    
-   docker commit
    
-   docker rm / docker rmi
    
-   Networking
    
-   Quota Support / CGroups
    

+bocker init 提供了有限的 bocker build 能力

四、总结

到此本文要介绍的内容就结束了，正如开篇我们提到的，写出最终的脚本实现这样一个小玩意并没有什么实用价值，真正的价值是我们通过100行左右的脚本，以交互式的方式去理解Docker的核心技术点。在工作中与容器打交道时能有更多的思路去排查、解决问题。

END

猜你喜欢

-   [Dubbo 中 Zookeeper 注册中心原理分析](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247495997&idx=1&sn=f165a2de458f21a6e5ce00e92cdd85df&chksm=ebdb81afdcac08b9ac790eb638f39f4f09622e3bc72a6a3ebf9174fc7d574afb3aeb243aac1d&scene=21#wechat_redirect)
    
-   [委派模式——从SLF4J说起](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247495986&idx=2&sn=9488faa16be8a970157c9257bbaf0bc6&chksm=ebdb81a0dcac08b6cc49eb1c7400017829cbd82c4459847e96d2c230c1fc6d73a167b68fe39d&scene=21#wechat_redirect)
    
-   [vivo 故障定位平台的探索与实践](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247495930&idx=2&sn=8830003fac4710e1daceb76d766849f1&chksm=ebdb8068dcac097ecb61a3873a478daf0a35436e7d791e22486e43f7309a42b42b1ee0d7cb85&scene=21#wechat_redirect)
    
-   [规则引擎Drools在贷后催收业务中的应用](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247495455&idx=2&sn=6bb29f3c4de7e8ec68011e7bb9fefa79&chksm=ebdb9f8ddcac169b978dd6a8e6450b65770a76aa5254be9dc1e085287558f650a2675b420d81&scene=21#wechat_redirect)