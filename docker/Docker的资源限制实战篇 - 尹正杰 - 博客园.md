　　　　　　　　　　　　　**Docker的资源限制实战篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.资源限制概述**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　默认情况下,容器没有资源限制，可以使用系统所有资源。docker 通过 docker run 配置容器的内存，cpu, 磁盘io使用量。

　　其中许多功能都要求您的内核支持Linux功能。 要检查支持，可以使用docker info命令。 如果内核中禁用了某项功能，您可能会在输出结尾处看到警告，如"WARNING: No swap limit support"。

　　对于Linux 主机，如果没有足够的内容来执行重要的系统任务，将会抛出OOM或者Out of Memory Exception(内存溢出、内存泄漏、内存异常), 随后系统会开始杀死进程以释放内存。每个进程都有可能被kill，包括Dockerd和其它的应用程序。如果重要的系统进程被Kill,会导致整个系统宕机。
　　内存资源限制:
　　　　产生OOM异常时，Docker尝试通过调整Docker守护程序上的OOM优先级来减轻这些风险，以便它比系统上的其他进程更不可能被杀死。 容器上的OOM优先级未调整，这使得单个容器被杀死的可能性比Docker守护程序或其他系统进程被杀死的可能性更大，不推荐通过在守护程序或容器上手动设置--oom-score-adj为极端负数，或通过在容器上设置--oom-kill-disable来绕过这些安全措施。
　　　　Docker 可以强制执行硬性内存限制，即只允许容器使用给定的内存大小。 
　　　　Docker 也可以执行非硬性内存限制，即容器可以使用尽可能多的内存，除非内核检测到主机上的内存不够用了。
　　　　CPU资源限制:
　　　　一个宿主机，有几十个核心的CPU，CPU为可压缩资源，但是有成百上千的进程，那么这么多进程怎么执行的？
　　　　实时优先级：0-99
　　　　　　非实时优先级(nice)：-20-19，对应100-139的进程优先级Linux kernel进程的调度基于CFS(Completely Fair Scheduler)，完全公平调度
　　　　CPU密集型的场景：　　　　　　优先级越低越好，计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力
　　　　IO密集型的场景：　　　　　　优先级值调高点，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度），比如Web应用，高并发，数据量大的动态网站来说，数据库应该为IO密集型。

　　博主推荐阅读:
　　　　https://docs.docker.com/config/containers/resource_constraints/
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129063350816-1945035925.png)

**二.****限制容器对内存的访问实战案例**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
内存限制参数：
　　　　-m or --memory=:
　　　　　　容器可以使用的最大内存量，如果您设置此选项，则允许的最小值为4m （4兆字节）。
　　　　--memory-swap *:
　　　　　　容器可以使用的交换分区大小，要在设置物理内存限制的前提才能设置交换分区的限制。该参数咱们可以忽略掉，因为在K8S 1.8.3版本中(https://github.com/kubernetes/kubernetes/blob/release-1.8/CHANGELOG-1.8.md),如果你的服务器开启了swap分区就会报错,而且生产环境并不建议使用swap分区，它可能会导致你的服务响应会很慢，但你检查了所有配置均是正常的，这会让你查无可查,禁用swap分区从你我做起。
　　　　--memory-swappiness:
　　　　　　设置容器使用交换分区的倾向性，值越高表示越倾向于使用swap分区，范围为0-100，0为能不用就不用，100为能用就用
　　　　--kernel-memory：
　　　　　　容器可以使用的最大内核内存量，最小为4m，由于内核内存与用户空间内存隔离，因此无法与用户空间内存直接交换，因此内核内存不足的容器可能会阻塞宿主主机资源，这会对主机和其他容器产生副作用。生产环境中尽量不要使用这个参数,如果内存不足的情况下就直接向领导申请加内存就好。
　　　　--memory-reservation:
　　　　　　允许您指定小于--memory的软限制，当Docker检测到主机上的争用或内存不足时会激活该限制，如果使用--memory-reservation，则必须将其设置为低于--memory才能使其优先。 因为它是软限制，所以不能保证容器不超过限制。
　　　　--oom-kill-disable：
　　　　　　默认情况下，发生OOM时，kernel会杀死容器内进程，但是可以使用--oom-kill-disable参数，可以禁止oom发生在指定的容器上，即 仅在已设置-m /  -  memory选项的容器上禁用OOM，如果-m 参数未配置，产生OOM时，主机为了释放内存还会杀死系统进程

　　swap限制：
　　　　--memory-swap:
　　　　　　只有在设置了 --memory 后才会有意义。使用Swap,可以让容器将超出限制部分的内存置换到磁盘上。WARNING：经常将内存交换到磁盘的应用程序会降低性能
　　　　　　不同的设置会产生不同的效果：
　　　　　　　　--memory-swap：
　　　　　　　　　　值为正数， 那么--memory和--memory-swap都必须要设置，--memory-swap表示你能使用的内存和swap分区大小的总和，例如： --memory=300m, --memory-swap=1g, 那么该容器能够使用 300m 内存和 700m swap，即--memory是实际物理内存大小值不变，而实际的计算方式为(--memory-swap)-(--memory)=容器可用swap
　　　　　　　　--memory-swap：
　　　　　　　　　　如果设置为0，则忽略该设置，并将该值视为未设置，即未设置交换分区。
　　　　　　　　--memory-swap：
　　　　　　　　　　如果等于--memory的值，并且--memory设置为正整数，容器无权访问swap即也没有设置交换分区
　　　　　　　　--memory-swap：
　　　　　　　　　　如果设置为unset，如果宿主机开启了swap，则实际容器的swap值为2x( --memory)，即两倍于物理内存大小，但是并不准确。
　　　　　　　　--memory-swap：
　　　　　　　　　　如果设置为-1，如果宿主机开启了swap，则容器可以使用主机上swap的最大空间。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129064851011-1019742656.png)**

**1>.下载专门用于测试资源限制的镜像(https://hub.docker.com/r/lorel/docker-stress-ng)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy      v1.8.20             1858fe05d96f        5 days ago          606MB
registry            latest              708bc6af7e5e        5 days ago          25.8MB
tomcat-app01        v0.1                bf45c22f2d5b        5 days ago          983MB
tomcat-base         8.5.50              9ff79f369094        7 days ago          968MB
jdk-base            1.8.0_231           0f63a97ddc85        7 days ago          953MB
centos-base         7.6.1810            b4931fd9ace2        7 days ago          551MB
centos              centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image pull lorel/docker-stress-ng
Using default tag: latest
latest: Pulling from lorel/docker-stress-ng
Image docker.io/lorel/docker-stress-ng:latest uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
c52e3ed763ff: Pull complete 
a3ed95caeb02: Pull complete 
7f831269c70e: Pull complete 
Digest: sha256:c8776b750869e274b340f8e8eb9a7d8fb2472edd5b25ff5b7d55728bca681322
Status: Downloaded newer image for lorel/docker-stress-ng:latest
docker.io/lorel/docker-stress-ng:latest
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy           v1.8.20             1858fe05d96f        5 days ago          606MB
registry                 latest              708bc6af7e5e        5 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        5 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        7 days ago          968MB
jdk-base                 1.8.0_231           0f63a97ddc85        7 days ago          953MB
centos-base              7.6.1810            b4931fd9ace2        7 days ago          551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# docker image pull lorel/docker-stress-ng

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129072606211-743723499.png)

**2>.**未限制容器**内存******

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# lscpu 
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 158
Model name:            Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
Stepping:              10
CPU MHz:               2207.997
BogoMIPS:              4415.99
Virtualization:        VT-x
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              9216K
NUMA node0 CPU(s):     0,1
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr ss
e sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ssbd ibrs ibpb stibp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid rdseed adx smap xsaveopt arat spec_ctrl intel_stibp flush_l1d arch_capabilities[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-haproxy           v1.8.20             1858fe05d96f        5 days ago          606MB
registry                 latest              708bc6af7e5e        5 days ago          25.8MB
tomcat-app01             v0.1                bf45c22f2d5b        5 days ago          983MB
tomcat-base              8.5.50              9ff79f369094        7 days ago          968MB
jdk-base                 1.8.0_231           0f63a97ddc85        7 days ago          953MB
centos-base              7.6.1810            b4931fd9ace2        7 days ago          551MB
centos                   centos7.6.1810      f1cb7c7d58b7        10 months ago       202MB
lorel/docker-stress-ng   latest              1ae56ccafe55        3 years ago         8.1MB
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker container run -it --rm lorel/docker-stress-ng --vm 2 --vm-bytes 128M
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 2 vm
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129074451323-2062282574.png)

**3>.硬限制内存**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# lscpu 
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 158
Model name:            Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
Stepping:              10
CPU MHz:               2207.997
BogoMIPS:              4415.99
Virtualization:        VT-x
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              9216K
NUMA node0 CPU(s):     0,1
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2
 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ssbd ibrs ibpb stibp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid rdseed adx smap xsaveopt arat spec_ctrl intel_stibp flush_l1d arch_capabilities[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker run -it -m 128M --rm lorel/docker-stress-ng --vm 2 --vm-bytes 128M
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 2 vm
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129080343465-347924539.png)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS               NAMES
934a595104be        lorel/docker-stress-ng   "/usr/bin/stress-ng …"   29 minutes ago      Up 29 minutes                           cool_boyd
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ls /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/
cgroup.clone_children  memory.kmem.limit_in_bytes          memory.kmem.tcp.usage_in_bytes  memory.memsw.max_usage_in_bytes  memory.soft_limit_in_bytes  tasks
cgroup.event_control   memory.kmem.max_usage_in_bytes      memory.kmem.usage_in_bytes      memory.memsw.usage_in_bytes      memory.stat
cgroup.procs           memory.kmem.slabinfo                memory.limit_in_bytes           memory.move_charge_at_immigrate  memory.swappiness
memory.failcnt         memory.kmem.tcp.failcnt             memory.max_usage_in_bytes       memory.numa_stat                 memory.usage_in_bytes
memory.force_empty     memory.kmem.tcp.limit_in_bytes      memory.memsw.failcnt            memory.oom_control               memory.use_hierarchy
memory.kmem.failcnt    memory.kmem.tcp.max_usage_in_bytes  memory.memsw.limit_in_bytes     memory.pressure_level            notify_on_release
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/memory.limit_in_bytes 
134217728
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo "134217728/1024/1024" | bc
128
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.使用cgroup动态扩容内存(温馨提示:资源限制只能改大不能改小哟~)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS            
934a595104be        lorel/docker-stress-ng   "/usr/bin/stress-ng …"   5 seconds ago       Up 5 seconds     
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ls /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/cgroup.clone_children           memory.kmem.tcp.max_usage_in_bytes  memory.oom_control
cgroup.event_control            memory.kmem.tcp.usage_in_bytes      memory.pressure_level
cgroup.procs                    memory.kmem.usage_in_bytes          memory.soft_limit_in_bytes
memory.failcnt                  memory.limit_in_bytes               memory.stat
memory.force_empty              memory.max_usage_in_bytes           memory.swappiness
memory.kmem.failcnt             memory.memsw.failcnt                memory.usage_in_bytes
memory.kmem.limit_in_bytes      memory.memsw.limit_in_bytes         memory.use_hierarchy
memory.kmem.max_usage_in_bytes  memory.memsw.max_usage_in_bytes     notify_on_release
memory.kmem.slabinfo            memory.memsw.usage_in_bytes         tasks
memory.kmem.tcp.failcnt         memory.move_charge_at_immigrate
memory.kmem.tcp.limit_in_bytes  memory.numa_stat
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/memory.limit_in_bytes 134217728
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/memory.limit_in_bytes 134217728
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo "256*1024*1024" | bc
268435456
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo 268435456 > /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/memory.limit_in_bytes [root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/fs/cgroup/memory/docker/934a595104beb2cb986bcf1601c0fc36d9463131cc167b4df8cc31ff74ed34ec/memory.limit_in_bytes 268435456
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129082658151-218974346.png)

**5>.软**限制内存****

```
[root@docker101.yinzhengjie.org.cn ~]# docker run -it -m 128M --memory-reservation 64m --rm lorel/docker-stress-ng --vm 2 --vm-bytes 128M
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 2 vm
```

****![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129084826548-4170223.png)****

****6>.关闭一个容器的oom机制(开启该功能宿主机无论占用多少内存都不会去强行kill该容器，此方式生产环境并不推荐使用，了解即可)****

```
[root@docker101.yinzhengjie.org.cn ~]# docker run -it -m 128M --oom-kill-disable --rm lorel/docker-stress-ng --vm 2 --vm-bytes 128M
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 2 vm
```

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129085339266-925852557.png)

**三.**限制容器对CPU的访问实战案例****

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/block/sda/queue/scheduler 　　　　　　　　　　　　　　#查看磁盘调度算法
noop [deadline] cfq 
[root@docker101.yinzhengjie.org.cn ~]# 
```

\[root@docker101.yinzhengjie.org.cn ~\]# cat /sys/block/sda/queue/scheduler 　　　　　　　　　　　　　　　　　　#查看磁盘调度算法

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
　　默认情况下，每个容器对主机CPU周期的访问权限是不受限制的，但是我们可以设置各种约束来限制给定容器访问主机的CPU周期，大多数用户使用的是默认的CFS调度方式，在Docker 1.13及更高版本中，还可以配置实时优先级。

　　CPU限制参数:
　　　　--cpus=:　　　　　　指定容器可以使用多少可用CPU资源。例如，如果主机有两个CPU，并且您设置了--cpus =“1.5”，那么该容器将保证最多可以访问一个半的CPU。这相当于设置--cpu-period =“100000”和--cpu-quota =“150000”。在Docker 1.13和更高版本中可用。
　　　　--cpu-period:　　　　　　设置CPU CFS调度程序周期，它与--cpu-quota一起使用，，默认为100微妙，范围从 100ms~1s，即[1000, 1000000]
　　　　--cpu-quota:　　　　　　在容器上添加CPU CFS配额，也就是cpu-quota / cpu-period的值，通常使用--cpus设置此值
　　　　--cpuset-cpus:　　　　　　主要用于指定容器运行的CPU编号，也就是我们所谓的绑核。
　　　　--cpuset-mem:　　　　　　设置使用哪个cpu的内存，仅对非统一内存访问(NUMA)架构有效，而我们一般使用的是X86_64架构的CPU，因此一般情况下我们不会做这个限制。
　　　　--cpu-shares:　　　　　　主要用于cfs中调度的相对权重,cpushare值越高的，将会分得更多的时间片，默认的时间片1024，最大262144,一般情况下我们在生产环境中不会去设置该参数。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129091545598-1338047857.png)**

**1>.未限制CPU核心数**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# lscpu | grep "^CPU(s)"
CPU(s):                8
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker run -it -m 128M --rm lorel/docker-stress-ng --cpu 4 --vm 4
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 4 cpu, 4 vm
stress-ng: error: [14] stress-ng-vm: fork failed: errno=12: (Out of memory)
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129093530128-200746283.png)**

**2>.限制容器CPU**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# lscpu | grep "^CPU(s)"
CPU(s):                8
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker run -it -m 128M --cpus 2 --rm lorel/docker-stress-ng --cpu 4 --vm 4
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 4 cpu, 4 vm
stress-ng: error: [14] stress-ng-vm: fork failed: errno=12: (Out of memory)
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129093854597-225843586.png)**

**3>.将容器运行在指定的CPU核心上**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129095436274-461450014.png)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# lscpu | grep "^CPU(s)"　　　　　　　　　　　　#可用查看当前服务器可用的CPU核心数
CPU(s):                8
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# lscpu | grep  "On-line"　　　　　　　　　　　　#可用查看当前服务器可用的CPU编号
On-line CPU(s) list:   0-7
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker run -it -m 128M --cpus 2 --cpuset-cpus 1,4 --rm lorel/docker-stress-ng --cpu 4 --vm 4
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 4 cpu, 4 vm
stress-ng: error: [11] stress-ng-vm: fork failed: errno=12: (Out of memory)
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129095904534-536356621.png)

**4>.动态修改**容器运行在指定的CPU核心上****

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ls /sys/fs/cgroup/cpuset/docker/818e686e5932bbac2dd525c39c655bdd55135bfc3127297ef0ca813f05e7f77a/
cgroup.clone_children  cgroup.procs          cpuset.cpus            cpuset.effective_mems  cpuset.mem_hardwall    cpuset.memory_pressure     cpuset.memory_spread_slab  cpuset.sched_load_balance        notify_on_release
cgroup.event_control   cpuset.cpu_exclusive  cpuset.effective_cpus  cpuset.mem_exclusive   cpuset.memory_migrate  cpuset.memory_spread_page  cpuset.mems                cpuset.sched_relax_domain_level  tasks
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ls /sys/fs/cgroup/cpuset/docker/818e686e5932bbac2dd525c39c655bdd55135bfc3127297ef0ca813f05e7f77a/cpuset.cpus 
/sys/fs/cgroup/cpuset/docker/818e686e5932bbac2dd525c39c655bdd55135bfc3127297ef0ca813f05e7f77a/cpuset.cpus
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/fs/cgroup/cpuset/docker/818e686e5932bbac2dd525c39c655bdd55135bfc3127297ef0ca813f05e7f77a/cpuset.cpus 
1,4
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# echo "5,7" > /sys/fs/cgroup/cpuset/docker/818e686e5932bbac2dd525c39c655bdd55135bfc3127297ef0ca813f05e7f77a/cpuset.cpus 
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cat /sys/fs/cgroup/cpuset/docker/818e686e5932bbac2dd525c39c655bdd55135bfc3127297ef0ca813f05e7f77a/cpuset.cpus 
5,7
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129100849209-1239489313.png)**

**5>.基于cpu-shares对CPU进行切分使用(启动两个容器，一个“--cpu-shares”的值为500，另一个为1000,观察效果，如下图所示)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# docker run -it --cpu-shares 500 --name test01 --rm lorel/docker-stress-ng --cpu 4 --vm 4
 stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 4 cpu, 4 vm



[root@docker101.yinzhengjie.org.cn ~]# docker run -it --cpu-shares 1000 --name test02 --rm lorel/docker-stress-ng --cpu 4 --vm 4
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 4 cpu, 4 vm
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200129102326910-590727682.png)**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186