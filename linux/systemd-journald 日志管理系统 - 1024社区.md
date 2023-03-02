-   4 小时前
    
    发布 #1 2023年2月10日星期五 18点32分
    
-   #1

上次说到的，Linux系统警告：Journal has been rotated since unit was started  
讲得不是很详细，也就是参照下面的教程并不能有效解决以上问题：  
[https://1024.day/d/1633](https://1024.day/d/1633)

这次来说说 journald.conf 配置文件和 journald 日志存储之间的关系。

通过修改 `/etc/systemd/journald.conf` 文件控制存储类型值，在 Journal 字符串下面可以修改存储类型。

```
[Journal]
#Storage=auto
```

Storage 支持的值为 volatile，persistent，auto 和 none，默认是 auto，所有值的含义如下：

-   如果为 volatile，则日志数据将仅存储在内存中，即在 /run/log/journal 目录下。
-   如果是 persistent，则数据将会存储在磁盘上，即 /var/log/journal 目录下，并且在早期引导阶段磁盘不可写的时候把数据保存到 /run/log/journal 目录下。
-   auto值意味着把日志数据存储在 /var/log/journal/ 目录中。但是该目录必须已经存在并且设置了适当的权限。如果不存在，则日记数据将存储在易失性 /run/log/journal/ 目录中，并且在系统关闭时会删除该数据。
-   none关闭所有存储，所有接收到的日志数据将被丢弃。

以上可以看出，仅仅修改日志最大占用量是不够的，因为默认 auto 模式下，系统不会自动创建 `/var/log/journal` 目录，日志都是写入内存 `/run/log/journal/` 目录下。才会导致上面种种问题。

解决办法很简单，如日志存储在硬盘，则修改成下面配置：

```
[Journal]
Storage=persistent
SystemMaxUse=100M
```

以上，系统会自动创建 /var/log/journal 文件目录，并生成日志。限最大 100MB

如打算日志存储在内存中，改成配置如下：

```
[Journal]
Storage=volatile
RuntimeMaxUse=100M
```

最后，不要忘了重启一下 journald 使配置生效：

`systemctl status systemd-journald`