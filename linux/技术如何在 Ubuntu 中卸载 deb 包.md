![](https://img.linux.net.cn/data/attachment/album/202208/01/180906afaqifcsqqsfsxyq.jpg)

[从 .deb 文件安装应用](https://itsfoss.com/install-deb-files-ubuntu/)\[1\] 非常简单。双击它，它会在软件中心中打开，然后从那里安装它。

但是如何在 Ubuntu 或 Debian 中卸载 deb 包呢？如何删除一段时间前安装的软件包呢。

虽然这有几个如果和但是，但删除 .deb 文件的最简单和最可靠的方法是使用 `apt remove` 命令。

```
sudo apt remove program_name
```

如你所见，**你需要在这里知道确切的包名称**。这可能并不总是显而易见的。例如，如果你在 Ubuntu 上安装 Google Chrome，则该程序在命令行中称为 “google-chrome-stable”。你已经知道了吗？我猜你不知道。

在本教程中，我将详细介绍如何找到确切的包名称，然后使用它来删除应用。我还将讨论使用图形方法删除 deb 包。

在我向你展示如何从命令行删除 deb 包之前，让我们在软件中心应用中快速查看它。

Ubuntu 有软件中心 GUI 应用，允许搜索、安装和删除应用。

搜索时，软件中心可能不会显示已安装的应用。

![Searching for installed applications may not show any results in Ubuntu Software Center](https://img.linux.net.cn/data/attachment/album/202208/01/180927dejleni4nlmuxiui.png)

_Searching for installed applications may not show any results in Ubuntu Software Center_

但是，如果向下滚动，你仍可能在“已安装”部分下找到它。外部应用通常不带徽标显示。

![Some installed applications can be found in the ‘installed’ tab of the Software Center](https://img.linux.net.cn/data/attachment/album/202208/01/181108cbnzu7my5fl4imqx.png)

_Some installed applications can be found in the ‘installed’ tab of the Software Center_

如果找到它，你可以通过单击“垃圾桶”图标或“删除”按钮来删除该应用。

![Removing applications from the Ubuntu software center](https://img.linux.net.cn/data/attachment/album/202208/01/181641dum4hesbfhe5e55o.jpg)

_Removing applications from the Ubuntu software center_

**一句话：检查是否可以从软件中心删除应用。**

我假设你不知道该应用命令的确切名称。你可能不知道 Google Chrome 安装为 google-chrome-stable 而 Edge 安装为 microsoft-edge-stable，这很正常。

如果你知道前几个字母，那么 tab 补全可能会有所帮助。否则，你可以 [使用 apt 命令列出已安装的应用](https://itsfoss.com/list-installed-packages-ubuntu/)\[2\] 并使用 `grep` 搜索应用程序名称：

```
apt list --installed | grep -i possible_package_name
```

例如，你可以智能地猜测 Google Chrome 包的名称中应该包含 chrome。你可以这样搜索：

```
apt list --installed | grep -i chrome
```

在某些情况下，你可能会得到多个结果。

![check if google chrome installed in ubuntu](https://img.linux.net.cn/data/attachment/album/202208/01/180928wxkzzarxss2xjtsz.png)

_check if google chrome installed in ubuntu_

如果你不确定这些软件包的作用，你可以随时通过以下方式获取它们的详细信息：

```
apt info exact_package_name
```

获得确切的软件包名称后，你可以使用 `apt remove` 命令将其删除。

```
sudo apt remove exact_package_name
```

你还可以使用 `apt-get remove` 或 `dpkg uninstall` 命令来删除。

![Removing applications installed via .deb files using the apt command](https://img.linux.net.cn/data/attachment/album/202208/01/180929veyrxe2eyxtfvprh.png)

_Removing applications installed via .deb files using the apt command_

另一种方法是使用 [Synaptic 包管理器](https://itsfoss.com/synaptic-package-manager/)\[3\]。在 GNOME 以“软件中心”的形式创建其图形包管理器之前，Synaptic 是 Ubuntu 和许多其他发行版中的默认 GUI 包管理器。

它仍然是 [Xfce 桌面环境](https://www.xfce.org/)\[4\] 上的推荐工具。

首先安装它：

```
sudo apt install synaptic
```

打开 Synaptic 并搜索包名称。查找标记为绿色的已安装软件包。右键单击它们，然后单击“标记为删除”。之后点击应用。

![Removing Deb packages using Synaptic package manager](https://img.linux.net.cn/data/attachment/album/202208/01/181657f2uoah8nigewun3e.jpg)

_Removing Deb packages using Synaptic package manager_

我非常乐意使用 `apt` 命令删除从 .deb 文件中安装的软件包。但我可以理解，并不是每个人都喜欢使用命令行。

在删除从外部 .deb 文件安装的应用时，我发现软件中心中找不到它。软件中心还可以做的更好一些。

我希望你现在对删除 deb 包有更好的了解。如果你有任何问题，请告诉我。

___

via: [https://itsfoss.com/uninstall-deb-ubuntu/](https://itsfoss.com/uninstall-deb-ubuntu/)

作者：[Abhishek Prakash](https://itsfoss.com/)\[5\] 选题：[lkxed](https://github.com/lkxed)\[6\] 译者：[geekpi](https://github.com/geekpi)\[7\] 校对：[wxy](https://github.com/wxy)\[8\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[9\] 原创编译，[Linux中国](https://linux.cn/article-14885-1.html?utm_source=rss&utm_medium=rss)\[10\] 荣誉推出

___

\[1\]: https://itsfoss.com/install-deb-files-ubuntu/  
\[2\]: https://itsfoss.com/list-installed-packages-ubuntu/  
\[3\]: https://itsfoss.com/synaptic-package-manager/  
\[4\]: https://www.xfce.org/  
\[5\]: https://itsfoss.com/  
\[6\]: https://github.com/lkxed  
\[7\]: https://github.com/geekpi  
\[8\]: https://github.com/wxy  
\[9\]: https://github.com/LCTT/TranslateProject  
\[10\]: https://linux.cn/article-14885-1.html?utm\_source=rss&utm\_medium=rss