## 1\. 硬盘格式化

-   查看新磁盘

通常，第二块硬盘的名字会是 `/dev/sdb` 。

-   磁盘分区

会有提示输入参数：

command (m for help):n  
Partition number(1-4):1  
First cylinder (1-22800,default 1):Enter  
command (m for help):w

-   格式化磁盘为 ext4

-   将磁盘挂载到指定目录

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>mkdir /data
</span></span><span><span>mount -t ext4 /dev/sdb /data
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   开机自动挂载目录

先找到设备的 UUID。

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>blkid <span>|</span>grep /dev/sdb
</span></span><span><span>
</span></span><span><span>/dev/sdb: <span>UUID</span><span>=</span><span>"328a9d32-abb6-492a-aabe-b6a63583674d"</span> <span>TYPE</span><span>=</span><span>"ext4"</span>
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

编辑 `/etc/fstab` 新增挂载项。

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>vim /etc/fstab 
</span></span><span><span>
</span></span><span><span><span>UUID</span><span>=</span>328a9d32-abb6-492a-aabe-b6a63583674d /dev/sdb ext4 defaults <span>0</span> <span>0</span>
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

## 2\. 迁移 Docker 存储

-   暂停 Docker

-   移动 Docker 存储数据

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>mv /var/lib/docker /data/
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   创建新的链接

<table><tbody><tr><td></td><td><pre tabindex="0"><code data-lang="bash"><span><span>ln -s /data/docker /var/lib/docker
</span></span></code></pre><span title="Copy to clipboard"></span></td></tr></tbody></table>

-   重启 Docker