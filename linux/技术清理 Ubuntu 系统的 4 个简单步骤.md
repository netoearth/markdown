![](https://img.linux.net.cn/data/attachment/album/202209/30/123825r2vri1kv0rc2f7xh.jpg)

> 现在，试试看这 4 个简单的步骤，来清理你的 Ubuntu 系统吧。

这份精简指南将告诉你如何清理 Ubuntu 系统以及如何释放一些磁盘空间。

如果你的 Ubuntu 系统已经运行了至少一年，尽管系统是最新的，你仍然可能会觉得你的 Ubuntu 系统运行缓慢且滞后。

在过去，因为你想试验某一应用程序，或是在看到了它的好评推荐之后，而安装了许多应用程序，但你并没有删除它们。下面这些方法可以帮助你找出一些可以释放的隐藏磁盘空间。

Apt 缓存是 Ubuntu 系统保存你下载过的所有文件的地方，以供你之后可以查看它。大多数用户不会去清理 Apt 缓存，而它却可能会占用数百兆字节。

打开终端，并运行以下命令，可以得到你的 Apt 缓存有多少：

```
du -sh /var/cache/apt/archives
```

![](https://img.linux.net.cn/data/attachment/album/202209/30/124344v4bel4mslbvzbde9.png)

如果你的 Ubuntu 系统已经安装了很久的话，你将看到这个数字非常大。运行以下命令来清理 Apt 缓存。

```
sudo apt-get clean
```

如果你已经运行 Ubuntu 系统超过了一年，那么你很可能安装多个内核。如果你的硬件是最新的，并且与 Linux 兼容而没有太多配置，你可以删除旧的内核，保留最新的内核。

在终端运行以下命令来删除旧的内核：

```
sudo apt-get autoremove --purge
```

![Autoremove Purge](https://img.linux.net.cn/data/attachment/album/202209/30/124355cj8ie9jzh778or8r.png)

_Autoremove Purge_

如果你是一个喜欢尝试 Linux 应用程序的人，那么你的系统中肯定有一些不再需要的没用的应用程序。

现在，你可能已经忘记了你安装过的应用程序名称。不过你可以在终端运行以下命令来查看你最近安装的内容：

你会得到通过 `apt` 命令安装的应用程序和软件包的列表：

```
history | grep "apt-get install"
```

![List of apt installed app – History](https://img.linux.net.cn/data/attachment/album/202209/30/124407z8nn4w60ri6ygwu5.png)

_List of apt installed app – History_

你将得到最近安装的应用程序列表：

```
grep " install " /var/log/dpkg.log.1
```

```
zgrep " install " /var/log/dpkg.log.2.gz
```

你可以运行以下命令来删除应用程序和软件包：

```
sudo apt remove app1 package1
```

有大量免费和原生的系统 [清理应用](https://www.debugpoint.com/2017/02/stacer-is-a-system-monitoring-and-clean-up-utility-for-ubuntu/)\[1\] 可以使用。但是，我认为 [BleachBit](https://www.bleachbit.org/)\[2\] 是清理系统最好的一个应用，因为它经久不衰。

使用以下命令安装 BleachBit 或通过应用商店安装。

```
sudo apt install bleachbit
```

安装后，打开 BleachBit，并运行扫描。它会向你显示浏览器占用的所有缓存文件、临时文件、垃圾等，你只需单击一个按钮即可清理它。

![](https://img.linux.net.cn/data/attachment/album/202209/30/124414akyk09hq9g0e4x88.png)

Flatpak 应用程序和运行时runtime会占用大量磁盘空间。因为在设计上，Flatpak 的可执行文件结合了运行时。尽管运行时可以在相关应用程序之间共享，但许多未使用的剩余运行时可能会占用你的磁盘空间。

删除一些未使用的 Flatpak 包最直接的方法是下面的命令。在终端运行这一命令。

```
flatpak uninstall --unused
```

可以参考 [这篇文章](https://www.debugpoint.com/clean-up-flatpak/)\[3\] 了解有关 Flatpak 包的更多信息。

如果你使用 Ubuntu 系统，那么你很有可能使用的是 Snap 软件包。随着时间的推移，Snap 会积累不相关的运行时和文件。你可以使用以下脚本来清理一些没用的 snap 运行时。

将下面的脚本复制到一个新文件中，并将其命名为 `clean_snap.sh`：

然后使用 `chmod +x clean_snap.sh` 命令来赋予它可执行权限，并通过 `./clean_snap.sh` 运行

```
#!/bin/bash  
#Removes old revisions of snaps  
#CLOSE ALL SNAPS BEFORE RUNNING THIS  
set -eu  
LANG=en_US.UTF-8 
snap list --all | awk '/disabled/{print $1, $3}' |
      while read snapname revision; do
          snap remove "$snapname" --revision="$revision"
      done
```

可以参考 [这篇文章](https://www.debugpoint.com/clean-up-snap/)\[4\] 了解有关清理 Snap 包的更多信息。

你还可以使用以下命令来手动搜索大文件。

```
find /home -type f -exec du -h {} + | sort -hr | head -20
```

例如，运行以下命令，你会得到根目录 `/` 中的前 20 个大文件。现在你可以查看大文件，并使用文件管理器手动删除它们。请注意删除文件时要非常小心。尽量不要涉及 `/home` 目录以外的任何内容。

![Find Large files in Linux](https://img.linux.net.cn/data/attachment/album/202209/30/124424r09untu4uumcx9k3.png)

_Find Large files in Linux_

这样就完成了。如果你按照上述步骤操作，你一定能够释放 Ubuntu 系统中的一些空间，现在你的系统有剩余空间了。你可以按照这些措施来清理 Ubuntu 系统。不要忘记使用最新的软件包，使你的系统保持到最新。

🗨️ 如果你认为使用上述的技巧可以释放一些磁盘空间，并使得你的 Ubuntu 更快了，请在下方评论区留言。你通常又是使用什么命令来清理你的 Ubuntu 系统？

快留言告诉我吧。

___

via: [https://www.debugpoint.com/4-simple-steps-clean-ubuntu-system-linux/](https://www.debugpoint.com/4-simple-steps-clean-ubuntu-system-linux/)

作者：[Arindam](https://www.debugpoint.com/author/admin1/)\[5\] 选题：[lkxed](https://github.com/lkxed)\[6\] 译者：[Donkey-Hao](https://github.com/Donkey-Hao)\[7\] 校对：[wxy](https://github.com/wxy)\[8\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[9\] 原创编译，[Linux中国](https://linux.cn/article-15089-1.html?utm_source=rss&utm_medium=rss)\[10\] 荣誉推出

___

\[1\]: https://www.debugpoint.com/2017/02/stacer-is-a-system-monitoring-and-clean-up-utility-for-ubuntu/  
\[2\]: https://www.bleachbit.org  
\[3\]: https://www.debugpoint.com/clean-up-flatpak/  
\[4\]: https://www.debugpoint.com/clean-up-snap/  
\[5\]: https://www.debugpoint.com/author/admin1/  
\[6\]: https://github.com/lkxed  
\[7\]: https://github.com/Donkey-Hao  
\[8\]: https://github.com/wxy  
\[9\]: https://github.com/LCTT/TranslateProject  
\[10\]: https://linux.cn/article-15089-1.html?utm\_source=rss&utm\_medium=rss