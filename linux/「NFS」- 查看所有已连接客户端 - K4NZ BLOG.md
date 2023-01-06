## 问题描述

为了进行某些操作，我们需要查看所有挂载NFS服务的客户端。

## 解决办法

不建议使用 showmount -a 命令，因为结果并不可靠。

使用 netstat 命令，查看已建立的连接，能够得到准确结果：

```
netstat -an | grep 2049

```

## 参考文献

[How to get the list of clients connected to an NFS server within a local network?](https://stackoverflow.com/questions/34919597/how-to-get-the-list-of-clients-connected-to-an-nfs-server-within-a-local-network "How to get the list of clients connected to an NFS server within a local network?")