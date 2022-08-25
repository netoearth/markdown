1.  [陈少文的网站](https://www.chenshaowen.com/)
2.  [Posts](https://www.chenshaowen.com/post/)
3.  常用的清理 Kubernetes 集群资源命令

Please enable Javascript to view the contents

![](https://www.chenshaowen.com/blog/images/2021/12/k8s-cleaner.png)

> 长时间运行的集群，常会面临各种资源耗尽的问题，另外磁盘不足时 Kubelet 还会主动清理镜像增加不确定因素，本文提供了一些命令片段用于清理工作。

## 1\. Kubernetes 基础对象清理

-   清理 Evicted 状态的 Pod

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>kubectl get pods --all-namespaces -o wide <span>|</span> grep Evicted <span>|</span> awk <span>'{print $1,$2}'</span> <span>|</span> xargs -L1 kubectl delete pod -n
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理 Error 状态的 Pod

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>kubectl get pods --all-namespaces -o wide <span>|</span> grep Error <span>|</span> awk <span>'{print $1,$2}'</span> <span>|</span> xargs -L1 kubectl delete pod -n
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理 Completed 状态的 Pod

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>kubectl get pods --all-namespaces -o wide <span>|</span> grep Completed <span>|</span> awk <span>'{print $1,$2}'</span> <span>|</span> xargs -L1 kubectl delete pod -n
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理没有被使用的 PV

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>kubectl describe -A pvc <span>|</span> grep -E <span>"^Name:.*</span>$<span>|^Namespace:.*</span>$<span>|^Used By:.*</span>$<span>"</span> <span>|</span> grep -B <span>2</span> <span>"&lt;none&gt;"</span> <span>|</span> grep -E <span>"^Name:.*</span>$<span>|^Namespace:.*</span>$<span>"</span> <span>|</span> cut -f2 -d: <span>|</span> paste -d <span>" "</span> - - <span>|</span> xargs -n2 bash -c <span>'kubectl -n ${1} delete pvc ${0}'</span>
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理没有被绑定的 PVC

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>kubectl get pvc --all-namespaces <span>|</span> tail -n +2 <span>|</span> grep -v Bound <span>|</span> awk <span>'{print $1,$2}'</span> <span>|</span> xargs -L1 kubectl delete pvc -n
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理没有被绑定的 PV

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>kubectl get pv <span>|</span> tail -n +2 <span>|</span> grep -v Bound <span>|</span> awk <span>'{print $1}'</span> <span>|</span> xargs -L1 kubectl delete pv
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

## 2\. Linux 清理

-   查看磁盘全部空间

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>df -hl /
</span></span><span><span>
</span></span><span><span>Filesystem      Size  Used Avail Use% Mounted on
</span></span><span><span>/dev/sda2       100G   47G   54G  47% /
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   查看指定目录占用

-   删除指定前缀的文件夹

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span><span>cd</span> /nfsdata
</span></span><span><span>ls <span>|</span> grep archived- <span>|</span>xargs -L1 rm -r
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理僵尸进程

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>ps -A -ostat,ppid <span>|</span> grep -e <span>'^[Zz]'</span> <span>|</span> awk <span>'{print }'</span> <span>|</span> xargs <span>kill</span> -HUP &gt; /dev/null 2&gt;<span>&amp;</span><span>1</span>
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

## 3\. Docker 清理

-   查看磁盘使用情况

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>docker system df
</span></span><span><span>
</span></span><span><span>TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
</span></span><span><span>Images              <span>361</span>                 <span>23</span>                  178.5GB             173.8GB <span>(</span>97%<span>)</span>
</span></span><span><span>Containers          <span>29</span>                  <span>9</span>                   6.682GB             6.212GB <span>(</span>92%<span>)</span>
</span></span><span><span>Local Volumes       <span>4</span>                   <span>0</span>                   3.139MB             3.139MB <span>(</span>100%<span>)</span>
</span></span><span><span>Build Cache         <span>0</span>                   <span>0</span>                   0B                  0B
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理 none 镜像

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>docker images <span>|</span> grep none <span>|</span> awk <span>'{print $3}'</span> <span>|</span> xargs docker rmi
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   清理不再使用的数据卷

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>docker volume rm <span>$(</span>docker volume ls -q<span>)</span>
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

或者

-   清理缓存

-   全面清理

删除关闭的容器、无用的存储卷、无用的网络、dangling 镜像（无 tag 镜像）

-   清理正则匹配上的镜像

这里清理的是 `master-8bcf8d7-20211206-111155163` 格式的镜像。

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>docker images <span>|</span>grep -E <span>"([0-9a-z]*[-]){3,}[0-9]{9}"</span> <span>|</span>awk <span>'{print $3}'</span> <span>|</span> xargs docker rmi
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

## 4\. 设置定时

-   查看定时任务

-   设置定时任务

文本新增定时任务

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>*/35 */6 * * *  /usr/bin/docker images <span>|</span> grep none <span>|</span> awk <span>'{print $3}'</span> <span>|</span> xargs /usr/bin/docker rmi
</span></span><span><span><span>45</span> <span>1</span> * * * /usr/bin/docker system prune -f
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

这里第一个任务是每隔六个小时的第 35 分钟执行，第二个任务每天的 1 时 45 分执行。这里需要注意的是，命令需要使用绝对路径，否则可能会执行失败。

-   定时任务的格式

设置定时格式: `* * * * * shell`

第一个星号，minute，分钟，值为 0-59  
第二个星号，hour，小时，值从 0-23  
第三个星号，day，天，值为从 1-31  
第四个星号，month，月，值为从 1-12 月，或者简写的英文，比如 Nov、Feb 等  
第五个星号，week 周，值为从 0-6 或者简写的英文，Wen、Tur 等，代表周几，其中 0 代表周末

___

![微信公众号](https://www.chenshaowen.com/static/images/vinqi.jpg)

___