上一篇博文《[Redhat 设置 NFS 挂载的简单步骤](https://cloud.tencent.com/developer/article/1072082?from=10680)》,其中摘录了一段 nfs 中 fuser 的使用，索性将其全部发出，以供参考。

___

对于 NFS 的安全问题，我们是不能掉以轻心的。那么我们如何确保它的安全呢？这里我们首先我们需要分析一下它的不安全性。看看在那些方面体现了它的不安全。NFS 服务安全性分析：不安全性主要体现于以下 4 个方面:

-   1、新手对 NFS 的访问控制机制难于做到得心应手,控制目标的精确性难以实现
-   2、NFS 没有真正的用户验证机制,而只有对 RPC/Mount 请求的过程验证机制
-   3、较早的 NFS 可以使未授权用户获得有效的文件句柄
-   4、在 RPC 远程调用中,一个 SUID 的程序就具有超级用户权限.

**加强 NFS 服务安全的方法:**

1、合理的设定/etc/exports\*\*享出去的目录,最好能使用 anonuid,anongid 以使 MOUNT 到 NFS SERVER 的 CLIENT 仅仅有最小的权限,最好不要使用 root\_squash.

2、使用 IPTABLE 防火墙限制能够连接到 NFS SERVER 的机器范围

```
iptables -A INPUT -i eth0 -p TCP -s 192.168.0.0/24 --dport 111 -j ACCEPT  
iptables -A INPUT -i eth0 -p UDP -s 192.168.0.0/24 --dport 111 -j ACCEPT  
iptables -A INPUT -i eth0 -p TCP -s 140.0.0.0/8 --dport 111 -j ACCEPT  
iptables -A INPUT -i eth0 -p UDP -s 140.0.0.0/8 --dport 111 -j ACCEPT
```

3、为了防止可能的 Dos 攻击,需要合理设定 NFSD 的 COPY 数目.

4、修改/etc/hosts.allow 和/etc /hosts.deny 达到限制 CLIENT 的目的

```
/etc/hosts.allow   
portmap: 192.168.0.0/255.255.255.0 : allow   
portmap: 140.116.44.125 : allow   
/etc/hosts.deny   
portmap: ALL : deny
```

5、改变默认的 NFS 端口

NFS 默认使用的是 111 端口,但同时你也可以使用 port 参数来改变这个端口,这样就可以在一定程度上增强安全性.

6、使用 Kerberos V5 作为登陆验证系统

修改/etc/hosts.allow 和/etc/hosts.deny 达到限制 CLIENT 的目的

```
/etc/hosts.allow   
portmap: 192.168.0.0/255.255.255.0 : allow   
portmap: 140.116.44.125 : allow
```

这个 NFS 服务安全得多注意!!

/tmp　\*(rw,no\_root\_squash)

no\_root\_squash:登入到 NFS 主机的用户如果是 ROOT 用户,他就拥有 ROOT 的权限,此参数很不安全,建议不要使用.

有时需要执行 umont 卸载 nfs 盘阵时,会遇见 device is busy 的情况,字面意思理解为设备忙,有其他进程正在使用此设备.

此时需要用到命令 fuser

其格式为: $ fuser -m -v  （nfs 挂载点） 回车执行后得到的结果依次是:用户 进程号 权限 命令

此命令可以查看到访问此设备的所有进程,停止进程后 umount.

如果添加参数 －k 则可以一次性将所有当前访问 nfs 共享盘阵的进程停止 也可以加－i 打开交互显示,以便用户确认

或者用 fuser 命令:

#fuser -v -m 挂载点

即可查处 用户 PID 等,KILL 掉该进程后再 umount.

或者

#umount -l 挂载点

选项 –l 并不是马上 umount,而是在该目录空闲后再 umount.还可以先用命令 ps aux 来查看占用设备的程序 PID,然后用命令 kill 来杀死占用设备的进程,这样就 umount 的 NFS 服务安全非常放心了.

本文分享自作者个人站点/博客：https://zhangge.net/复制

如有侵权，请联系

cloudcommunity@tencent.com

删除。