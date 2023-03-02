![](https://img.linux.net.cn/data/attachment/album/202302/16/085250d5ngtogo2fjjn8o2.jpg)

> 本指南介绍了 [systemd](https://freedesktop.org/wiki/Software/systemd/)\[1\] 的 journalctl 工具及其各种命令的基础知识。你可以使用这些命令对 Linux 中的桌面和服务器日志进行故障诊断。以下是如何使用 journalctl 查看和分析 systemd 日志的不同例子。

很多人说 systemd 不好，它对系统的影响很大，这也是一个有争议的话题。但你不能否认的是，它提供了一套完善的工具来管理和排除系统故障。想象一下，当你遇到一个没有 GUI 的损坏系统时，你可能会把启动和 GRUB 弄得一团糟。在这种情况下，你可以从一个立付Live系统启动，挂上你的 Linux 分区，然后浏览 systemd 的日志，找出问题所在。

systemd 有三个基本组件，如下所示：

-   `systemd`：Linux 操作系统的系统和服务管理器。
-   `systemctl` ：该命令用于反观和控制 systemd 系统和服务管理器的状态。
-   `systemd-analyze`：该命令提供系统启动时的性能统计，并从系统和服务管理器中检索其他状态和跟踪信息。

除了这三个服务外，systemd 还提供其他服务，如 journald、logind、networkd 等。在本指南中，我们将讨论 systemd 的 journald 服务。

根据设计，systemd 提供了一个集中的方式来处理所有来自进程、应用程序等的操作系统日志。所有这些日志事件都由 systemd 的 journald 守护进程来处理。journald 守护进程收集所有来自 Linux 操作系统各处的日志，并将其作为二进制数据存储在文件中。

以二进制数据集中记录事件、系统问题的好处有很多。例如，由于系统日志是以二进制而不是文本形式存储的，你可以以文本、JSON 对象等多种方式进行转译，以满足各种需求。另外，由于日志是按顺序存储的，通过对日志的日期/时间操作，超级容易追踪到单个事件。

请记住，journald 收集的日志文件数以千行计，而且不断更新每次开机、每个事件。因此，如果你有一个长期运行的 Linux 操作系统，日志的大小应该以 GB 为单位。由于有着数以千计的日志，最好用基本命令进行过滤，以了解更多系统问题。

journald 的配置文件存在于以下路径中。它包含了关于如何进行日志记录的各种标志。你可以看一下这个文件，并进行必要的修改。但我建议不要修改这个文件，除非你知道自己在做什么。

```
/etc/systemd/journald.conf
```

journald 以二进制格式存储日志。它们被保存在这个路径下的一个目录中：

```
/var/log/journal
```

例如，在下面的路径中，有一个目录包含了迄今为止的所有系统日志。

![journalctl log file path](https://img.linux.net.cn/data/attachment/album/202302/16/085624f7tesrmwwteyteme.jpg)

_journalctl log file path_

不要使用 `cat` 命令，也不要使用 `nano` 或 `vi` 来打开这些文件。它们（是二进制的），无法正常显示。

查看 journald 日志的基本命令是：

```
journalctl
```

![journalctl](https://img.linux.net.cn/data/attachment/album/202302/16/085631r6caqam285i81an1.jpg)

_journalctl_

该命令提供了所有应用程序和进程的日志条目，包括错误、警告等。它显示的列表中，最旧的日志在顶部，当前的日志在底部。你需要不断按回车键来逐行滚动浏览。你也可以使用 `PAGE UP` 和 `PAGE DOWN` 键来滚动。按 `q` 键可以退出这个视图。

默认情况下，`journalctl` 以当前系统时区显示日志的时间。然而，你可以很容易地在命令中提供时区，将同一日志转换为不同的时区。例如，要以 UTC 查看日志，请使用以下命令：

```
journalctl --utc
```

![journalctl --utc](https://img.linux.net.cn/data/attachment/album/202302/16/085639zkkbf3c33hztsv3p.jpg)

_journalctl --utc_

系统产生的日志有不同的优先级。有些日志可能是可以忽略的警告，有些可能是重要的错误。你可能想只看错误，不看警告。这也可以用下面的命令来实现。

要查看紧急系统信息，请使用：

```
journalctl -p 0
```

![journalctl -p 0](https://img.linux.net.cn/data/attachment/album/202302/16/085650h3bkt6qc6bz16kh3.jpg)

_journalctl -p 0_

错误代码：

```
0: 紧急情况
1: 警报
2: 危急
3: 错误
4: 警告
5: 通知
6: 信息
7：调试
```

当你指定错误代码时，它显示该等级及更高的所有信息。例如，如果你指定下面的命令，它会显示所有优先级为 2、1 和 0 的信息：

```
journalctl -p 2
```

当你运行 `journalctl` 命令时，它会显示当前启动的信息，即你正在运行的会话中的信息。但也可以查看过去的启动信息。

在每次重启时，日志都会持续更新。journald 会记录不同启动时的日志。要查看不同启动时的日志，请使用以下命令。

```
journalctl --list-boots
```

![journalctl list-boots](https://img.linux.net.cn/data/attachment/album/202302/16/085702i5bqnzkqqk5c5hk5.jpg)

_journalctl list-boots_

-   第一个数字显示的是 journald 的唯一的启动跟踪号码，你可以在下一个命令中使用它来分析该特定的启动。
-   第二个数字是启动 ID，你也可以在命令中指定。
-   接下来的两个日期、时间组合是存储在相应文件中的日志的时间。如果你想找出某个特定日期、时间的日志或错误，这就非常方便了。

要查看一个特定的启动号码，你可以选择第一个启动跟踪号码或启动 ID，如下所示。

```
journalctl -b -45
```

```
journalctl -b 8bab42c7e82440f886a3f041a7c95b98
```

![journalctl -b 45](https://img.linux.net.cn/data/attachment/album/202302/16/085712tf7mekc9izym4cyh.jpg)

_journalctl -b 45_

你也可以使用 `-x` 选项，在显示屏上添加 systemd 错误信息的解释。在某些情况下，这是个救命稻草。

```
journalctl -xb -p 3
```

![journalctl -xb](https://img.linux.net.cn/data/attachment/album/202302/16/085721x1ep6410v011h403.jpg)

_journalctl -xb_

`journalctl` 功能强大，可以在命令中提供类似英语的参数，用于时间和日期操作。

你可以使用 `--since` 选项与 `yesterday`、`today`、`tomorrow` 或 `now` 组合。

下面是一些不同命令的例子。你可以根据你的需要修改它们。它们是不言自明的。以下命令中的日期、时间格式为 `"YYYY-MM-DD HH:MM:SS"`

```
journalctl --since "2020-12-04 06:00:00"
```

```
journalctl --since "2020-12-03" --until "2020-12-05 03:00:00"
```

```
journalctl --since yesterday
```

```
journalctl --since 09:00 --until "1 hour ago"
```

![journalctl --since 09:00 --until](https://img.linux.net.cn/data/attachment/album/202302/16/085733cjgfsfskzvknlf6f.jpg)

_journalctl --since 09:00 --until_

你也可以将上述内容与错误级别开关结合起来。

Linux 内核信息也可以从日志中提取出来。要查看当前启动时的内核信息，请使用以下命令：

```
journalctl -k
```

你可以从 journald 日志中过滤出某个 systemd 服务单元的特定日志。例如，如果要查看 NetworkManager 服务的日志，请使用下面的命令。

```
journalctl -u NetworkManager.service
```

![journalctl NetworkManager service](https://img.linux.net.cn/data/attachment/album/202302/16/085744qllx96oummlg3x0a.jpg)

_journalctl NetworkManager service_

如果你不知道服务名称，可以使用下面的命令来列出系统中的 systemd 服务。

```
systemctl list-units --type=service
```

如果你正在分析服务器日志，在多个用户登录的情况下，这个命令很有帮助。你可以先用下面的命令从用户名中找出用户的 ID。例如，要找出用户 `debugpoint` 的 ID：

```
id -u debugpoint
```

然后使用 `_UID` 选项指定该 ID 与来查看该用户产生的日志。

```
journalctl _UID=1000 --since today
```

![journalctl _UID](https://img.linux.net.cn/data/attachment/album/202302/16/085755rgnlnahueulijfcn.jpg)

_journalctl \_UID_

同样地，使用 `_GID` 选项也可以查到用户组的情况。

你也可以查看某个特定程序或可执行文件的日志。例如，如果你想找出 `gnome-shell` 的信息，你可以运行以下命令。

```
journalctl /usr/bin/gnome-shell --since today
```

![journalctl gnome-shell](https://img.linux.net.cn/data/attachment/album/202302/16/085803f9deadg66cx8dxhm.jpg)

_journalctl gnome-shell_

希望本指南能帮助你使用 `journalctl` 查看分析 Linux 桌面或服务器上的 systemd 日志，排除故障。如果你知道如何使用这些命令，systemd 日志管理的功能非常强大，它能让你在调试时的生活变得轻松一些。现在所有主流的 Linux 发行版都使用 systemd。Ubuntu、Debian、Fedora、Arch 它们都使用 systemd 作为其默认的操作系统组件。如果你想了解不使用 systemd 的 Linux发行版，你可能想看看 [MX-Linux](https://www.debugpoint.com/tag/mx-linux)\[2\]、Gentoo、Slackware、Void Linux。

___

via: [https://www.debugpoint.com/systemd-journalctl/](https://www.debugpoint.com/systemd-journalctl/)

作者：[Arindam](https://www.debugpoint.com/author/admin1/)\[3\] 选题：[lkxed](https://github.com/lkxed)\[4\] 译者：[Chao-zhi](https://github.com/Chao-zhi)\[5\] 校对：[wxy](https://github.com/wxy)\[6\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[7\] 原创编译，[Linux中国](https://linux.cn/article-15544-1.html?utm_source=rss&utm_medium=rss)\[8\] 荣誉推出

___

\[1\]: https://freedesktop.org/wiki/Software/systemd/  
\[2\]: https://www.debugpoint.com/tag/mx-linux  
\[3\]: https://www.debugpoint.com/author/admin1/  
\[4\]: https://github.com/lkxed  
\[5\]: https://github.com/Chao-zhi  
\[6\]: https://github.com/wxy  
\[7\]: https://github.com/LCTT/TranslateProject  
\[8\]: https://linux.cn/article-15544-1.html?utm\_source=rss&utm\_medium=rss