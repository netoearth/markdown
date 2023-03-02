## 概述[](https://ewhisper.cn/posts/20749/#%E6%A6%82%E8%BF%B0%20-2)

假设这样一个场景：

生产环境中，Node 都需要通过堡垒机登录，但是 kubectl 是可以直接在个人电脑上登录的。

这种场景下，我想要通过 kubectl 登录到 K8S 集群里的 Node，可以实现吗？

可以的！  
本质上是利用容器（runC）的弱隔离（共享内核，Cgruop 等实现进程隔离）实现的权限逃逸。

如果贵司使用的一些商业容器平台（如：openshift，rancher）等，可能默认安装时就会通过 PSP scc 或 policy 等预先屏蔽掉这层隐患。  
但是如果是原生的 Kubernetes， 往往下面的办法是可行的。

## 原理概述[](https://ewhisper.cn/posts/20749/#%E5%8E%9F%E7%90%86%E6%A6%82%E8%BF%B0)

先说本质，本质上就是：

**容器（runC）是弱隔离**

-   对于虚拟机来说，虚拟机是通过内核（kernel）级别的隔离，不同的虚拟机有不同的内核，所以安全性要高很多，从虚拟机逃逸到其所在的物理机上是非常困难的。
-   但是，容器（runC）是弱隔离，一台机器上的所有容器都共享同一个内核，他们之所以默认互相看不见，是通过 Cgroup、net namespace 等实现的进程级别的隔离。

那么，加入你没有对容器的权限做进一步的限制，我是可以通过运行一个特权容器，直接进入到其所在的 node 上的。

## 具体步骤[](https://ewhisper.cn/posts/20749/#%E5%85%B7%E4%BD%93%E6%AD%A5%E9%AA%A4)

> 适用于 K8S 1.25 之前的版本。

步骤很简单，就是创建上文说的这么一个 **特权** 容器，通过 `nsenter` command 进入 node shell。示例 yaml 如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br></pre></td><td><pre><code><span>apiVersion:</span> <span>v1</span><br><span>kind:</span> <span>Pod</span><br><span>metadata:</span><br>  <span>labels:</span><br>    <span>run:</span> <span>nsenter-v0l86q</span><br>  <span>name:</span> <span>nsenter-v0l86q</span><br>  <span>namespace:</span> <span>default</span><br><span>spec:</span><br>  <span>containers:</span><br>  <span>-</span> <span>command:</span><br>    <span>-</span> <span>nsenter</span><br>    <span>-</span> <span>--target</span><br>    <span>-</span> <span>"1"</span><br>    <span>-</span> <span>--mount</span><br>    <span>-</span> <span>--uts</span><br>    <span>-</span> <span>--ipc</span><br>    <span>-</span> <span>--net</span><br>    <span>-</span> <span>--pid</span><br>    <span>-</span> <span>--</span><br>    <span>-</span> <span>bash</span><br>    <span>-</span> <span>-l</span><br>    <span>image:</span> <span>docker.io/library/alpine</span><br>    <span>imagePullPolicy:</span> <span>Always</span><br>    <span>name:</span> <span>nsenter</span><br>    <span>securityContext:</span><br>      <span>privileged:</span> <span>true</span><br>    <span>stdin:</span> <span>true</span><br>    <span>stdinOnce:</span> <span>true</span><br>    <span>tty:</span> <span>true</span><br>  <span>hostNetwork:</span> <span>true</span><br>  <span>hostPID:</span> <span>true</span><br>  <span>restartPolicy:</span> <span>Never</span><br>  <span>tolerations:</span><br>  <span>-</span> <span>key:</span> <span>CriticalAddonsOnly</span><br>    <span>operator:</span> <span>Exists</span><br>  <span>-</span> <span>effect:</span> <span>NoExecute</span><br>    <span>operator:</span> <span>Exists</span><br></code><p><i></i>YAML</p></pre></td></tr></tbody></table>

直接 `kubectl apply -f node-shell.yaml` 即可进入 node shell。

上面的 yaml，关键有这么几点：

进入 node shell 的命令：`nsenter --target 1 --mount --uts --ipc --net --pid -- bash -l`，在 Linux 系统里， nsenter 是一个命令行工具，用于进入到另一个 namespace 。 譬如， `nsenter -n -t 1 bash` 就是进入到 pid 为 1 的进程所在的网络 namespace 里。

以及进入 node shell 的权限：

-   `hostPID: true` 共享 host 的 pid
-   `hostNetwork: true` 共享 host 的网络
-   `privileged: true`: PSP 权限策略是 `privileged`, 即完全无限制。

进入 node shell 的 pod 后， 效果如下：

[![node shell- 可以切换 shell](https://pic-cdn.ewhisper.cn/img/2022/10/03/1571884c8e24e4392722b266c87d95f8-20221003194446.png)](https://pic-cdn.ewhisper.cn/img/2022/10/03/1571884c8e24e4392722b266c87d95f8-20221003194446.png "node shell- 可以切换 shell")

[![node shell- 可以查看所有的进程信息](https://pic-cdn.ewhisper.cn/img/2022/10/03/def61e43e48e2865a462d1c816f6a771-20221003194731.png)](https://pic-cdn.ewhisper.cn/img/2022/10/03/def61e43e48e2865a462d1c816f6a771-20221003194731.png "node shell- 可以查看所有的进程信息")

[![node shell- 可以执行 root 权限的 systemctl](https://pic-cdn.ewhisper.cn/img/2022/10/03/f5fbedf408ab6fb2aa154c37be980211-20221003194809.png)](https://pic-cdn.ewhisper.cn/img/2022/10/03/f5fbedf408ab6fb2aa154c37be980211-20221003194809.png "node shell- 可以执行 root 权限的 systemctl")

这里推荐 2 个工具，可以更方便地进入 node shell。

### krew node-shell[](https://ewhisper.cn/posts/20749/#krew-node-shell)

可以通过 kubectl 插件管理工具 [krew](https://ewhisper.cn/tags/krew/) 安装 node-shell.

如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><code><span># 安装工具</span><br>kubectl krew install node-shell<br><span># 进入 node shell</span><br>Kubectl node-shell &lt;node-name&gt;<br></code><p><i></i>BASH</p></pre></td></tr></tbody></table>

### Lens[](https://ewhisper.cn/posts/20749/#Lens)

Kubernetes 图形化管理工具 - [Lens](https://ewhisper.cn/posts/33163/) 也有相关功能。

具体使用方法如下：

[![Lens- 选择指定 node 进入 shell](https://pic-cdn.ewhisper.cn/img/2022/10/03/25614603d5326b13669acbbbc556f563-20221003195808.png)](https://pic-cdn.ewhisper.cn/img/2022/10/03/25614603d5326b13669acbbbc556f563-20221003195808.png "Lens- 选择指定 node 进入 shell")

[![Lens- 实际上也是启动个特权 pod，可以执行 root 命令](https://pic-cdn.ewhisper.cn/img/2022/10/03/f34c4af900420c19b79ac345a09521f2-20221003200007.png)](https://pic-cdn.ewhisper.cn/img/2022/10/03/f34c4af900420c19b79ac345a09521f2-20221003200007.png "Lens- 实际上也是启动个特权 pod，可以执行 root 命令")

## 总结[](https://ewhisper.cn/posts/20749/#%E6%80%BB%E7%BB%93)

上文介绍了通过 kubectl 命令以 root 权限进入 node shell 的方法，非常简单，实际上在大多数的原生 Kubernetes 上都生效。

这个命令实际上是一定程度上利用了安全上的未加固配置。

这里最后还是建议大家除了对 OS 进行安全加固，对 Kubernetes 也要按照安全最佳实践进行安全加固。（典型的就是起码 PSP 等 policy 不要设置为 `privileged`, 而是设置为 `Baseline` 或 `Restricted`）

注意安全！🚧🚧🚧

EOF