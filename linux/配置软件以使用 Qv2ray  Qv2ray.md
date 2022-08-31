恭喜！只剩下一个步骤就可以访问真正的互联网了！

## [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E4%B8%80%E8%88%AC%E6%96%B9%E6%B3%95) 一般方法

### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E4%BD%BF%E7%94%A8%E7%B3%BB%E7%BB%9F%E4%BB%A3%E7%90%86) 使用系统代理

对于 **Windows** 和 **macOS** 用户，几乎所有应用程序都遵循系统代理设置。对于 **Linux** 用户，一部分应用程序如 Firefox 和 Chromium 将会读取并使用 GNOME/KDE 设置的代理配置，但另一部分应用程序并不会这样。

目前，Qv2ray 的自动设置系统代理功能支持 **Windows**、**macOS** 和 **Linux**（GNOME/KDE）用户。您可以在以下位置找到 Qv2ray 的设置系统代理选项：

-   **Qv2ray 托盘菜单**。
    1.  右键点击托盘图标。
    2.  在的弹出菜单中，依次选择 **系统代理** -> **启用/禁用系统代理**。
-   **Qv2ray 首选项**。
    1.  点击 Qv2ray 主窗口中的 **首选项** 按钮。
    2.  在 **首选项** 中，选择 **[入站设置](qv2ray://open/preference/inbound)** 标签。
    3.  勾选 **设置系统代理** 选项。
    4.  点击 **确定** 保存您的设置。

Linux用户：KDE/GNOME 代理设置

如果您使用 GNOME 作为您的主要桌面环境，您可能会发现 GNOME 的系统代理设置非常有效。这是因为 GNOME 的系统代理设置得到了普遍的适配。

然而，KDE 用户可能会遭遇困境，因为 KDE 的系统代理设置更像是一个玩具。甚至 KDE 系列应用程序本身也不会读取和使用那个配置。在这种情况下，您可能需要寻求替代方案来配置您的应用程序。

### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E6%89%8B%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F) 手动配置应用程序

#### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#telegram) Telegram

您可以在 Telegram 应用中设置其使用代理。前往 **设置/Settings** -> **高级/Advanced** -> **网络和代理/Network and proxy** 然后点击 **连接类型/Connection**，**代理设置/Proxy Settings** 对话框将被打开。（译者注：不同的 Telegram 客户端和不同的翻译会导致选项略有不同。）

在 **代理设置/Proxy Settings** 中点击底部的 **添加代理/Add Proxy** 按钮。根据您自己的口味选择 SOCKS5/HTTP ，然后用 Qv2ray 入站设置中的信息填写剩下的选项。

最后，单击您刚刚配置的代理。Telegram 就设置好了。

#### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E7%BD%91%E7%BB%9C%E6%B5%8F%E8%A7%88%E5%99%A8) 网络浏览器

几乎所有的网络浏览器都支持手动配置代理服务器。将 Firefox 为例子，您可以在 **首选项 -> 常规-> 网络 -> 手动代理配置** 中找到代理设置。用 Qv2ray 入站设置中的信息填写这些字段以使用 Qv2ray 代理。

使用代理插件。

为了避免在代理配置之间重新切换，您可能想要使用第三方插件（例如，SwitchyOmega） 来增强您的浏览器。这些插件可以帮助实现更复杂的配置，包括多个配置文件和更多的流量转移。

#### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#java-%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F) Java 应用程序

对于 Java 应用程序，您可以使用 JVM 参数配置代理程序。

以下是一些例子：

-   使用 SOCKS5：
-   使用 HTTP(S)：

有问题的 Minecraft

Minecraft 的较新版本（`>=1.5.2`）不会使用 JVM 代理设置。这不是 Qv2ray 的问题。如果你真的想要通过代理玩 Minecraft，请考虑为该服务器设置一个 Dokodemo-door 入站，并直接连接到 `localhost`。

## [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E4%BE%9D%E8%B5%96%E5%B9%B3%E5%8F%B0%E7%9A%84%E6%96%B9%E6%B3%95) 依赖平台的方法

### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E4%BD%BF%E7%94%A8%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F) 使用环境变量

许多 CLI 程序（例如，`curl` 和 `wget`）将使用 `<PROTOCOL><PROTOCOL>_PROXY` 环境变量。

下面是一个配置示例：

如果在 Qv2ray 中启用了身份验证，请使用以下设置：

请注意，如果您的用户名或密码有特殊字符，您需要对其进行编码。下面是一个快速方法：

| `!` | `#` | `$` | `&` | `'` | `(` | `)` | `*` | `+` | `,` | `/` | `:` | `;` | `=` | `?` | `@` | `[` | `]` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `%21` | `%23` | `%24` | `%26` | `%27` | `%28` | `%29` | `%2A` | `%2B` | `%2C` | `%2F` | `%3A` | `%3B` | `%3D` | `%3F` | `%40` | `%5B` | `%5D` |

或者输入您想要编码的文本：

对于在 `sudo` 中运行的程序，如果您不在 shell 中运行 `sudo` 则需要配置 `sudo` 来保存这些变量。用 root 调用 `visudo` 并添加以下行：

但还有一些程序正在使用自己的变量。例如，`rsync` 使用 `RSYNC_PROXY` 用于 HTTP 代理：

强烈建议阅读您想要配置代理的程序手册。

### [#](https://qv2ray.net/lang/zh/getting-started/step4.html#%E4%BD%BF%E7%94%A8-proxychains) 使用 `proxychains`

如果上述任何方法都不起作用，您可以尝试使用 `proxychains`，它劫持了程序的功能/库来将网络连接重定向到您的代理服务器。

首先，您应该安装 `proxychains-ng`。每个操作系统的安装方法各不相同。

-   [Linux/macOS (opens new window)](https://github.com/rofl0r/proxychains-ng)
-   [Windows (opens new window)](https://github.com/shunf4/proxychains-windows)

编辑 `/etc/proxychains.conf`(用于全局代理链) 或 `$HOME/.proxychains/proxychains.conf`(用户)，编辑 `[ProxyList]` 部分，并在 Qv2ray 中将代理更改为 SOCKS5 代理：

配置 `proxychains` 之后，您可以在终端中使用 `proxychains <program>` 来让 `proxychains` 劫持程序的网络流量到指定的代理程序。如果你觉得控制台输出很多很烦，你可以在 `proxychains` 之后追加 `-q` 选项。

要注意的一件事是，`proxychains` 不能与静态链接的程序兼容，例如 Golang 程序。

[编辑本页](https://github.com/Qv2ray/qv2ray.github.io/edit/source/docs/lang/zh/getting-started/step4.md) (opens new window)

上次更新: 2021/8/13 上午7:56:47