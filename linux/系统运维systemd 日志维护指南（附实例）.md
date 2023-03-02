![](https://img.linux.net.cn/data/attachment/album/202302/10/072955z20ipg8vlpvdt1jq.jpg)

> systemd 内置了很多管理系统日志的功能。在本指南中，我们将介绍如何管理系统日志，并对其采取轮换、归档和清除日志等操作。我们还解释了手动系统日志清理方法和使用配置文件的变化。

如果你的 Linux 发行版支持 [systemd](https://www.freedesktop.org/wiki/Software/systemd/)\[1\]，那么从启动时开始，它每秒钟都会从系统的所有进程和应用程序中收集日志。所有这些日志事件都由 systemd 的 `journald` 守护程序管理。journald 收集所有的日志（信息、警告、错误等），并将其作为二进制数据存储在磁盘文件中。

由于日志保留在磁盘中，而且每秒钟都在收集，所以它占用了巨大的磁盘空间；特别是对于旧的系统、服务器来说。例如，在我的一个运行了一年左右的测试系统中，日志文件的大小是 GB 级的。

如果你管理多个系统、服务器，建议一定要正确管理 journald 日志，以便高效运行。让我们来看看如何管理日志文件。

使用 systemd 的 `journalctl` 工具，你可以查询这些日志，对其进行各种操作。例如，查看不同启动时的日志文件，检查特定进程或应用程序的最后警告和错误。如果你对这些不了解，我建议你在学习本指南之前先快速浏览一下此教程：[使用 journalctl 查看和分析 systemd 日志（附实例）](https://www.debugpoint.com/2020/12/systemd-journalctl/)\[2\] 》。

systemd 的 journald 守护进程在每次启动时都会收集日志。这意味着，它根据启动情况对日志文件进行分类。

日志以二进制形式存储在路径 `/var/log/journal`，文件夹为机器 ID。

比如说：

![日志文件位置的截图-1](https://img.linux.net.cn/data/attachment/album/202302/10/073113v2ejbxxwxwuq0jg0.jpg)

_日志文件位置的截图-1_

![日志文件位置的截图-2](https://img.linux.net.cn/data/attachment/album/202302/10/073121c4dhtrs3tdndhmtr.jpg)

_日志文件位置的截图-2_

另外，请记住，根据系统配置，运行时日志文件被存储在 `/run/log/journal/`。而这些在每次启动时都会被删除。

你可以，但不要这样做。相反，请按照下面的说明，使用 `journalctl` 工具清除日志文件以释放磁盘空间。

打开一个终端，运行以下命令。

```
journalctl --disk-usage
```

这应该为你提供系统中的日志文件实际使用的数量。

![journalctl 磁盘使用命令](https://img.linux.net.cn/data/attachment/album/202302/10/073129pgc4a44m9pmg33mv.jpg)

_journalctl 磁盘使用命令_

如果你有一个图形化的桌面环境，你可以打开文件管理器，浏览路径 `/var/log/journal`，并检查属性。

清理日志文件的有效方法应该是通过 `journald.conf` 配置文件来完成。正常情况下，即使 `journalctl` 提供了删除日志文件的工具，你也不应该手动删除这些文件。

让我们来看看如何 [手动](https://www.debugpoint.com/#delete-using-journal-conf)\[3\] 删除它，然后我将解释 `journald.conf` 中的配置变化，这样你就不需要时不时地手动删除文件；相反，systemd 会根据你的配置自动处理它。

首先，你必须 `flush` 和 `rotate` 日志文件。轮换rotate是将当前活动的日志文件归档，并立即开始创建一个新的日志文件继续记录日志。冲洗flush 开关要求日志守护进程将存储在 `/run/log/journal/` 中的所有日志数据冲入 `/var/log/journal/`，如果持久性存储被启用的话。

然后，在 `flush` 和 `rotate` 之后，你需要用 `vacuum-size`、`vacuum-time` 和 `vacuum-files` 选项运行 `journalctl` 来强制 systemd 清除日志。

例 1：

```
sudo journalctl --flush --rotate
```

```
sudo journalctl --vacuum-time=1s
```

上面这组命令会删除所有存档的日志文件，直到最后一秒。这有效地清除了一切。因此，在运行该命令时要小心。

![日志清理-例子](https://img.linux.net.cn/data/attachment/album/202302/10/073140ddbaaqa1ovrdvvjq.jpg)

_日志清理-例子_

清理完毕后：

![清理后--日志的占用空间](https://img.linux.net.cn/data/attachment/album/202302/10/073148ugif7obej3zz7zi7.jpg)

_清理后--日志的占用空间_

你也可以根据你的需要在 `--vacuum-time` 的数字后面提供以下后缀：

-   `s`：秒
-   `m`：分钟
-   `h`：小时
-   `days`：天
-   `months`：月
-   `weeks`：周
-   `years`：年

例 2：

```
sudo journalctl --flush --rotate
```

```
sudo journalctl --vacuum-size=400M
```

这将清除所有存档的日志文件，并保留最后 400MB 的文件。记住这个开关只适用于存档的日志文件，不适用于活动的日志文件。你也可以使用后缀，如下所示。

-   `K`：KB
-   `M`：MB
-   `G`：GB

例 3：

```
sudo journalctl --flush --rotate
```

```
sudo journalctl --vacuum-files=2
```

`vacuum-files` 选项会清除所有低于指定数量的日志文件。因此，在上面的例子中，只有最后两个日志文件被保留，其他的都被删除。同样，这只对存档的文件有效。

如果你愿意，你可以把两种选项结合起来，但我建议不要这样做。然而，如果同时使用两个选项，请确保先用 `--rotate` 选项运行。

虽然上述方法很好，也很容易使用，但建议你使用 journald 配置文件来控制日志文件的清理过程，该文件存在于 `/etc/systemd/journald.conf`。

systemd 为你提供了许多参数来有效管理日志文件。通过组合这些参数，你可以有效地限制日志文件所占用的磁盘空间。让我们来看看。

| journald.conf 参数 | 描述 | 实例 |
| --- | --- | --- |
| `SystemMaxUse` | 指定日志在持久性存储中可使用的最大磁盘空间 | `SystemMaxUse=500M` |
| `SystemKeepFree` | 指定在将日志条目添加到持久性存储时，日志应留出的空间量。 | `SystemKeepFree=100M` |
| `SystemMaxFileSize` | 控制单个日志文件在被轮换之前在持久性存储中可以增长到多大。 | `SystemMaxFileSize=100M` |
| `RuntimeMaxUse` | 指定在易失性存储中可以使用的最大磁盘空间（在 `/run` 文件系统内）。 | `RuntimeMaxUse=100M` |
| `RuntimeKeepFree` | 指定将数据写入易失性存储（在 `/run` 文件系统内）时为其他用途预留的空间数量。 | `RuntimeMaxUse=100M` |
| `RuntimeMaxFileSize` | 指定单个日志文件在被轮换之前在易失性存储（在 `/run` 文件系统内）所能占用的空间量。 | `RuntimeMaxFileSize=200M` |

如果你在运行中的系统的 `/etc/systemd/journald.conf` 文件中添加这些值，那么在更新文件后，你必须重新启动 journald。要重新启动，请使用以下命令。

```
sudo systemctl restart systemd-journald
```

在你清理完文件后，检查日志文件的完整性是比较明智的。要做到这一点，请运行下面的命令。该命令显示了日志文件是否通过（`PASS`）、失败（`FAIL`）。

```
journalctl --verify
```

![验证日志文件](https://img.linux.net.cn/data/attachment/album/202302/10/073205wmvvtfwk2k90twjm.png)

_验证日志文件_

希望本指南能帮助你了解 systemd 日志管理流程的基本情况。通过这些，你可以通过限制空间、清除旧的日志文件来管理系统或服务器中的日志文件所使用的磁盘空间。这些只是指导性的命令，你可以通过多种方式组合这些命令来实现你的系统需求。

-   [journalctl 手册](https://www.freedesktop.org/software/systemd/man/journalctl.html)\[4\]
-   [journald.conf 手册](https://www.freedesktop.org/software/systemd/man/journald.conf.html)\[5\]

___

via: [https://www.debugpoint.com/systemd-journald-clean/](https://www.debugpoint.com/systemd-journald-clean/)

作者：[Arindam](https://www.debugpoint.com/author/admin1/)\[6\] 选题：[lkxed](https://github.com/lkxed)\[7\] 译者：[Chao-zhi](https://github.com/Chao-zhi)\[8\] 校对：[wxy](https://github.com/wxy)\[9\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[10\] 原创编译，[Linux中国](https://linux.cn/article-15526-1.html?utm_source=rss&utm_medium=rss)\[11\] 荣誉推出

___

\[1\]: https://www.freedesktop.org/wiki/Software/systemd/  
\[2\]: https://www.debugpoint.com/2020/12/systemd-journalctl/  
\[3\]: https://www.debugpoint.com#delete-using-journal-conf  
\[4\]: https://www.freedesktop.org/software/systemd/man/journalctl.html  
\[5\]: https://www.freedesktop.org/software/systemd/man/journald.conf.html  
\[6\]: https://www.debugpoint.com/author/admin1/  
\[7\]: https://github.com/lkxed  
\[8\]: https://github.com/Chao-zhi  
\[9\]: https://github.com/wxy  
\[10\]: https://github.com/LCTT/TranslateProject  
\[11\]: https://linux.cn/article-15526-1.html?utm\_source=rss&utm\_medium=rss