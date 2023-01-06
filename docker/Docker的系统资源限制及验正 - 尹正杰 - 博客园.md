　　　　　　　　　　　**Docker的系统资源限制及验正**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.容器资源限制概述**

**1>.什么是"Limit a container's resources"**

```
　　默认情况下，容器没有资源约束，并且可以使用主机内核调度程序所允许的给定资源;

　　Docker提供控制容器可以使用多少内存、CPU或块IO的方法，设置Docker run命令的运行时配置标志;

　　这些特性中的许多要求内核支持linux功能:要检查支持，可以使用docker info命令
```

**2>.资源限制（Eight-sided containers）**

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191024013106418-589925600.png)

**3>.Out Of Memory Exception(简称"OOME")**

```
　　在linux主机上，如果内核检测到没有足够的内存来执行重要的系统功能，它会抛出oome或内存不足异常，并开始终止进程以释放内存;　　一旦发生OOME，任何进程都有可能被杀死，包括docker daemon在内;　　为此，Docker特地调整了docker daemon的OOM优先级，以免它被内核"正法",但容器的优先级并未被调整。
```

**4>.限制容器对内存的访问相关参数**

```
　　博主推荐阅读：https://docs.docker.com/config/containers/resource_constraints/。
```

**二.案例演示**

**1>.容器内存资源限制验证**

```
[root@node102.yinzhengjie.org.cn ~]# docker run --name stress -it -m 256m lorel/docker-stress-ng:latest stress --vm 2　　　　#启动2个进程对容器进行压测
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 2 vm
```

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# docker stats　　　　　　　　#查看容器的状态

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
32cc396b4bc1        stress              0.04%               188.6MiB / 256MiB   73.65%              0B / 0B             0B / 0B             5

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
32cc396b4bc1        stress              0.04%               188.6MiB / 256MiB   73.65%              0B / 0B             0B / 0B             5

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
32cc396b4bc1        stress              95.88%              97.91MiB / 256MiB   38.25%              0B / 0B             0B / 0B             5

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
32cc396b4bc1        stress              95.88%              97.91MiB / 256MiB   38.25%              0B / 0B             0B / 0B             5

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
32cc396b4bc1        stress              96.82%              238MiB / 256MiB     92.98%              0B / 0B             0B / 0B             5

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
32cc396b4bc1        stress              96.82%              238MiB / 256MiB     92.98%              0B / 0B             0B / 0B             5
^C
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.容器CPU资源限制验证**

```
[root@node102.yinzhengjie.org.cn ~]# docker run --name stress2 -it --cpus 1 --rm lorel/docker-stress-ng:latest stress --cpu 8　　　　#定义核心数为1，因此CPU不会超过100%
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 8 cpu
```

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node102.yinzhengjie.org.cn ~]# docker stats　　　　　　　　　　#查看容器的状态

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             0.00%               15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             0.00%               15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.43%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.43%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.31%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.31%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             98.36%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             98.36%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             98.27%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             98.27%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.23%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.23%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             97.81%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             97.81%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.23%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.23%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.16%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             99.16%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             98.08%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PID
S7b818dd0554d        stress2             98.08%              15.81MiB / 3.701GiB   0.42%               0B / 0B             0B / 0B             9
^C
[root@node102.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186