![](https://img.linux.net.cn/data/attachment/album/202210/30/124029bf0qjklphcpzpclh.jpg)

[GNOME 有了一个全新的文本编辑器](https://itsfoss.com/gnome-text-editor/)\[1\]，以取代旧的 Gedit 编辑器。

虽然 GNOME 42 已经可以使用了它，但 Ubuntu 22.04 还依赖于 Gedit。

这在 Ubuntu 22.10 中发生了变化。 GNOME 文本编辑器Text Editor 现在是默认程序，甚至没有安装 Gedit。

![搜索文本编辑器只出现 GNOME 文本编辑器](https://img.linux.net.cn/data/attachment/album/202210/30/124112hpt4czutjixloalc.png)

_搜索文本编辑器只出现 GNOME 文本编辑器_

虽然新编辑器足够好，但并不是每个人都喜欢它。如果你将 Gedit 与其他插件一起频繁使用，则尤其如此。

如果你属于这类人，让我向你展示如何在 Ubuntu 上安装 Gedit。我还将分享如何将其设为默认文本编辑器。

这实际上是不费吹灰之力的。虽然默认未安装 Gedit，但它仍然可以在 Ubuntu 仓库中找到。

所以，你所要做的就是使用 `apt` 命令来安装它：

```
sudo apt install gedit
```

Gedit 也可以在软件中心中找到，但它是 Snap 包。如果你愿意，你可以安装它。

![Gedit 也可以在 Ubuntu 的 Snap 商店中找到](https://img.linux.net.cn/data/attachment/album/202210/30/124112dnggkp0rrrpd202k.png)

_Gedit 也可以在 Ubuntu 的 Snap 商店中找到_

默认情况下，Gedit 为你提供访问一些插件的选项。你可以从 “汉堡菜单->偏好Preference\->插件Plugins” 启用或禁用插件。

![在 Gedit 中访问插件](https://img.linux.net.cn/data/attachment/album/202210/30/124112gc31tzflb6g9fz9q.png)

_在 Gedit 中访问插件_

你可以在这里看到可用的插件。检查已安装或正在使用的插件。

![查看 Gedit 中可用和已安装的插件](https://img.linux.net.cn/data/attachment/album/202210/30/124113s9j0k897jxj070zl.png)

_查看 Gedit 中可用和已安装的插件_

但是，你可以通过安装 gedit-plugins 元数据包将插件选择提升到一个新的水平。

```
sudo apt install gedit-plugins
```

这将使你可以访问其他插件，如书签、括号补全、Python 控制台等。

![其他 Gedit 插件](https://img.linux.net.cn/data/attachment/album/202210/30/124113mrhhh2zrp0hbdn0y.png)

_其他 Gedit 插件_

**提示**：如果你发现 Gedit 因缺少底角而显得有些格格不入，你可以安装一个名为 [Round Bottom Corner](https://extensions.gnome.org/extension/5237/rounded-window-corners/)\[2\] 的 GNOME 扩展。这将为包括 Gedit 在内的所有应用强制添加圆底角。

好了！你已经安装了 Gedit，但文本文件仍然在双击操作后使用 GNOME 文本编辑器打开。要使用 Gedit 打开文件，你需要右键单击，然后选择“打开方式open with”选项。

如果你希望一直使用 Gedit 打开文本文件，你可以将其设置为默认程序。

右键单击文本文件并选择“打开方式open with”选项。在此处选择 Gedit 并从底部启用“始终用于此文件类型Always use for this file type”选项。

![设置 Gedit 为默认文本编辑器](https://img.linux.net.cn/data/attachment/album/202210/30/124113aevt74728utvrv34.png)

_设置 Gedit 为默认文本编辑器_

觉得 Gedit 没达到预期么？这很少见，但我不会评判你。要从 Ubuntu 中删除 Gedit，请使用以下命令：

```
sudo apt remove gedit
```

你也可以尝试从软件中心卸载它。

GNOME 文本编辑器是下一代从头开始创建的编辑器，它与新的 GNOME 完美融合。

对于简单的文本编辑来说已经足够了。然而，Gedit 有一个插件生态系统，赋予它更多功能。

对于那些将它广泛用于编码和其他事情的人来说，安装 Gedit 仍然是 Ubuntu 中的一个选项。

那你呢？你会坚持使用默认的新文本编辑器还是回到旧的 Gedit？

___

via: [https://itsfoss.com/install-gedit-ubuntu/](https://itsfoss.com/install-gedit-ubuntu/)

作者：[Abhishek Prakash](https://itsfoss.com/)\[3\] 选题：[lkxed](https://github.com/lkxed)\[4\] 译者：[geekpi](https://github.com/geekpi)\[5\] 校对：[wxy](https://github.com/wxy)\[6\]

本文由 [LCTT](https://github.com/LCTT/TranslateProject)\[7\] 原创编译，[Linux 中国](https://linux.cn/)\[8\] 荣誉推出

___

\[1\]: https://itsfoss.com/gnome-text-editor/  
\[2\]: https://extensions.gnome.org/extension/5237/rounded-window-corners/  
\[3\]: https://itsfoss.com/  
\[4\]: https://github.com/lkxed  
\[5\]: https://github.com/geekpi  
\[6\]: https://github.com/wxy  
\[7\]: https://github.com/LCTT/TranslateProject  
\[8\]: https://linux.cn/