![About Neovim](https://ubunlog.com/wp-content/uploads/2018/05/about-neovim-830x490.png)

在下一篇文章中，我们将看一下Neovim。 是关于 **Vim代码的一个分支**。 该程序通过配置的可能性为我们带来了Vim的优势，并带来了更好的用户体验。 如果有人不知道，必须说Vim是基于模式的文本编辑器。 它是对Vi（1976）的改进而诞生的。 它的界面不是图形的，而是基于文本的。 尽管有几种带有图形界面的实现，例如gVim。 手边的编辑是 **直接替代Vim**。 如果您是Vim用户，您会发现自己对Neovim感到满意。

在这个编辑器中 **一切都可以通过键盘通过命令进行控制**。 起初似乎很难记住所有这些，起初确实如此。 但是，以合理的方式组织它们，并最终独立出来，这也是事实。 该程序将使我们更轻松地编辑文本，从而使我们可以自动执行重复性任务。 仅需几个键就可以完成无聊的任务。

指数

-   [1 Neovim的一般特征](https://ubunlog.com/zh-CN/neovim%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%9A%84vim%E5%88%86%E6%94%AF%EF%BC%8C%E4%BB%A5%E6%8F%90%E4%BE%9B%E6%9B%B4%E5%A5%BD%E7%9A%84%E7%94%A8%E6%88%B7%E4%BD%93%E9%AA%8C/#Caracteristicas_generales_de_Neovim)
-   [2 Neovim在Ubuntu上的安装](https://ubunlog.com/zh-CN/neovim%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%9A%84vim%E5%88%86%E6%94%AF%EF%BC%8C%E4%BB%A5%E6%8F%90%E4%BE%9B%E6%9B%B4%E5%A5%BD%E7%9A%84%E7%94%A8%E6%88%B7%E4%BD%93%E9%AA%8C/#Instalacion_de_Neovim_en_Ubuntu)
-   [3 设置Neovim](https://ubunlog.com/zh-CN/neovim%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%9A%84vim%E5%88%86%E6%94%AF%EF%BC%8C%E4%BB%A5%E6%8F%90%E4%BE%9B%E6%9B%B4%E5%A5%BD%E7%9A%84%E7%94%A8%E6%88%B7%E4%BD%93%E9%AA%8C/#Configurando_Neovim)

![Código php de Neovim](https://ubunlog.com/wp-content/uploads/2018/05/codigo-php-neovim-830x490.png)

-   该 **默认设置** 使您可以立即使用它。
-   Un **终端模拟器**.
-   编辑器为我们提供了一个API，该API允许 **通过任何语言与Neovim进行交流** 安全且异步地进行编程。
-   **现代终端功能** 例如光标样式，焦点事件，括号内的粘贴等。
-   正如我已经写的，它是 **非常可配置**。 可以说，就好像您正在构建自己的编辑器一样。 完成设置后，您将拥有一个满足您特定需求的自定义编辑器。
-   他的行为是 **可通过插件扩展**。 如果您是Vim用户，则可以 [继续使用相同的插件](https://ubunlog.com/zh-CN/vundle%E7%AE%A1%E7%90%86vim%E6%8F%92%E4%BB%B6/ "Vudle，管理vim插件")，以及社区为Neovim开发的内容。 而且，如果您找不到适合自己的插件，就可以使用自己喜欢的语言来创建自己的插件。
-   此外，它将为我们提供 **与其他任何代码编辑器相同的功能**，例如：自动完成，拼写检查，标签，语法着色，用正则表达式搜索和替换等。

El **项目源代码** 我们可以在 [GitHub页面](https://github.com/neovim/neovim "Neovim GitHub页面") 从编辑器。

## Neovim在Ubuntu上的安装

![instalación neovim desde la opción de software de Ubuntu](https://ubunlog.com/wp-content/uploads/2018/05/instalacion-opcion-de-software-ubuntu-830x320.png)

我们可以通过多种方式安装此编辑器。 最简单的是执行 **从安装 [软件选件](apt://neovim "从Ubuntu的软件选项安装Neovim") 从Ubuntu**。 要拥有最新版本，您必须 **添加Neovim PPA**。 在终端中运行以下命令（Ctrl + Alt + T）：

<table><tbody><tr><td><p>1</p></td><td><div><p><code>sudo apt-add-repository ppa:neovim-ppa/stable</code></p></div></td></tr></tbody></table>

然后，您必须更新软件包并通过在同一终端上键入来安装Neovim：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p></td><td><div><p><code>sudo apt-get update</code></p><p><code>sudo apt-get install neovim</code></p></div></td></tr></tbody></table>

这两个选项将安装相同版本的程序。 如果我们不想在系统中安装任何东西，可以使用 **Neovim .Appimage文件**。 要获得它，您需要安装curl。 确保拥有此工具后，在终端（Ctrl + Alt + T）中输入：

![descarga naovim appimage](https://ubunlog.com/wp-content/uploads/2018/05/descarga-neovim-appimage-830x60.png)

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p></td><td><div><p><code>curl -LO https://github.com/neovim/neovim/releases/download/nightly/nvim.appimage</code></p><p><code>chmod u+x nvim.appimage</code></p></div></td></tr></tbody></table>

下载并获得必要的权限后，我们可以在同一终端上键入以下内容来启动编辑器：

经过以上任何一个选项之后，我们现在就可以使用基于vim的编辑器。 谁需要它可以找到 **户田拉 [有关可能安装的文档](https://github.com/neovim/neovim/wiki/Installing-Neovim "Neovim安装在不同的操作系统上")** 在项目的GitHub页面上。

必须说这个程序有 **许多配置可能性**，所以通过 [官方文件](https://neovim.io/doc/ "Neovim官方文档") 或按 [用户手册](https://neovim.io/doc/user/ "Neovim用户手册") 这将使我们的编辑器看起来比默认情况下更好和更友好。

## 设置Neovim

![Tutorial Neovim](https://ubunlog.com/wp-content/uploads/2018/05/Tutorial-neovim-830x491.png)

Neovim包括一个 **互动教学**，运行命令 _：导师_ 启动它。

如果关闭Neovim，则您在会话中的所有设置都将丢失。 为了维护它们， **init.vim文件**，每次Neovim启动时都会加载。 如果您使用Vim，则此文件 **与vim的.vimrc文件具有相同的功能**.

这个配置文件 **它位于〜/ .config / nvim / init.vim中。** 如果不存在，请创建它。 配置文件可能会变得非常大，因此请尝试记录您放置在其中的所有内容。 注释可以加上«。 我们将能够获得 **有关此配置文件的更多信息** 在 [百科](https://github.com/neovim/neovim/wiki/FAQ#where-should-i-put-my-config-vimrc "关于init.vim的维基") 该计划。

  

本文内容遵循我们的原则 [编辑伦理](https://ubunlog.com/zh-CN/%E7%BC%96%E8%BE%91%E4%BC%A6%E7%90%86/)。 要报告错误，请单击 [信息](https://ubunlog.com/zh-CN/%E8%81%94%E7%B3%BB%E4%BA%BA/).