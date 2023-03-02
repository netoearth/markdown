2021 年微软正式推出了 Windows 11 桌面操作系统，相比于旧版本，Windows 11 除了在系统界面上有着相当大的变化之外，更多的变化在于其加入了对 Android 子系统的支持，从而将移动端最大的生态系统融入其中。但微软对可以升级到 Windows 11 的设备硬件做出限制，这导致市面上绝大多数的可以支持 Windows 10 的设备无法升级到 Windows 11，也就无法体验到 Windows 11 带来的诸多新功能特性。

但经过一年时间的迭代，即便属于支持末期的 Windows 10，在这几次年度功能更新后也获得了 Windows 11 才有的新特性，比如通过 WSLg 已经可以让 Windows 10 可以运行图形化的 Linux 应用，作为 Windows 11 独占的 WSA 在 Windows 10 上运行自然也变得不再遥远。

相比直接使用虚拟机或者模拟器在 Windows 10 上运行 Android 应用，通过 WSA 运行 Android 应用显得效率更高：依赖 Windows 自身的虚拟化引擎，无需先启动虚拟机从而资源占用更低，对于 Android 应用的系统架构无要求，最重要的是可以完全窗口化运行从而可以和当前的 Windows 生态充分融合。而目前就有两种方法让 Windows 10 运行 WSA，适用于不同的系统场景，如果你有在 Windows 10 上运行 Android 应用需求不妨「按需索取」。

## 将 Windows 11 的 WSA 移植到 Windows 10：WSAPatch

-   优点：基于 WSA 原生打造，兼容性最好。
-   缺点：仅支持 Windows 10 22H2 最新版，对系统版本有要求

[WSAPatch](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fcinit%2FWSAPatch) 顾名思义就是让 Windows 10 也能运行 WSA 的补丁，通过一系列该项目中提供了两个补丁文件，我们可以让 Windows 10 也可以安装上 WSA。

首先我们需要将 Windows 10 更新至最新版本（Windows 10 22H2），对应的版本至少 Windows 10 10.0.19045.2311，如果不确定可以在终端或者 PowerShell 中输入`winver` 来查看 Windows 版本。

![boxcnfpkgLN3GP9EKBNSAbIC0cc](https://cdn.sspai.com/editor/u_/cepdl8tb34tdunrsig4g?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

同时，Windows 10 版本至少为专业版，原因是后面我们需要安装 Hyper-V 虚拟化平台，而这也是 WSL 以及 WSA 的基础。如果你 Windows 10 使用的是家庭版，那么可能就无法使用这个办法来体验 WSA 了。

以上的准备工作就绪之后，首先依旧是在 BIOS 中开启虚拟化支持，并打开Windows 10 中的相关功能：

打开「控制面板」-「程序和功能」-「启用或关闭 Windows 功能」，在其中找到并开启 Hyper-V 、「虚拟机平台」、「Windows 虚拟机监控程序平台」以及「适用于Linux 的 Windows 子系统」。勾选安装后重启设备，最基础的工作已经准备完毕了。

![boxcnAN78kzu1LSFU9qBBSrTpvf](https://cdn.sspai.com/editor/u_/cepdl95b34tduvja0iv0?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

紧接着，我们需要在 Windows 10 中开启相关开发者权限，在「设置」-「更新和安全」中，找到「开发者选项」，打开「开发人员模式」，同时在 Powershell 项目中勾选「更改执行策略，以允许本地 PowerShell 脚本在未签名的情况下运行。远程脚本需要签名」并点击「应用」。

![boxcnBRrIoguUN6PgWuEXy2Mreh](https://cdn.sspai.com/editor/u_/cepdl95b34tduhil7jog?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

之后我们需要下载 WSA，为了方便后面的使用，我们可以自己定制 WSA，比如通过 MagiskOnWSA 项目将 Google 服务以及 Magisk 植入到 WSA。而生成对应的安装包则可以有下面两种方式：第一种方式即不依赖 GitHub Actions 服务的 [MagiskOnWSALocal](https://sspai.com/link?target=https://github.com/LSPosed/MagiskOnWSALocal)，这需要在本地安装 WSL 并安装 Ubuntu 来运行，具体可以参见：

-   [一日一技 | WSA 定制安装，找回你需要的 Google 服务和 Magisk](https://sspai.com/post/75351)

另一个办法是相对比较懒人的做法，依旧是通过依赖 GitHub Actions 服务的 [Magisk on WSA 的 fork 项目](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FAdityaAgarwal539%2FMagiskOnWSA-1)，将其 fork 到自己 Github 账户下，在 fork 项目中点击 Action 后，点击 Build WSA -- Run workflow，在配置菜单中选择想要的版本，这里 Build arch 选择`X64`，WSA release Type 选择`insider slow`，Magisk version 这里选择 `stable`。

![boxcnZONYnmea7fdMutiwLB4Dsz](https://cdn.sspai.com/editor/u_/cepdl9lb34tdunrsig50?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

如果需要集成 Google 服务，那么下面在 Variants of Gapps 选择`pico`，如果需要 root，那么 Root solution 中选择 `magisk`，其他默认点击`Run workflow`，等待一会儿在 `Artifacts` 中下载生成的 WSA 包即可。

![boxcn6oOTeN44KDMjaC9IitBXdb](https://cdn.sspai.com/editor/u_/cepdl9tb34tdt9qfne00?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

接下来我们解压缩生成的 WSA 包。紧接着下载需要的两个 dll 文件，这里我们可以直接从 WSAPatch 项目中的 [Releases](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fcinit%2FWSAPatch%2Freleases) 中下载，将下载的压缩包解压后，拷贝到刚刚解压的 WSA 目录下的 WsaClient 文件夹中。

![boxcn6RHUW6tZcrMhzeqadx80ze](https://cdn.sspai.com/editor/u_/cepdla5b34tdt9qfne0g?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

然后，我们需要使用代码编辑器修改 WSA 目录下的 `AppxManifest.xml` 文件。首先使用编辑器搜索 `AppxManifest.xml` 中的`TargetDeviceFamily` 这个关键字，然后定位到下面这段代码：

```
<TargetDeviceFamily Name="Windows.Desktop" MinVersion="10.0.22000.120" MaxVersionTested="10.0.22000.120"/>
```

将其中的 `MinVersion`中的 `10.0.22000.120`修改为`10.0.19045.2311`。

![boxcnQvx8kDRQ9OKEq2qsntFWtg](https://cdn.sspai.com/editor/u_/cepdladb34tdunrsig5g?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

然后再搜索`customInstall` ，注释掉下面两段内容：

```
<rescap:Capability Name="customInstallActions"/>
```

以及：

```
<desktop6:Extension Category="windows.customInstall">
    <desktop6:CustomInstall Folder="CustomInstall" desktop8:RunAsUser="true">
        <desktop6:RepairActions>
            <desktop6:RepairAction File="WsaSetup.exe" Name="Repair" Arguments="repair"/>
        </desktop6:RepairActions>
        <desktop6:UninstallActions>
            <desktop6:UninstallAction File="WsaSetup.exe" Name="Uninstall" Arguments="uninstall"/>
        </desktop6:UninstallActions>
    </desktop6:CustomInstall>
</desktop6:Extension>
```

![boxcn7PJ9tJ3z6xgyKKv9jdtvHf](https://cdn.sspai.com/editor/u_/cepdladb34tdt9qfne10?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

完成之后保存，接下来我们使用管理员权限打开 Powershell，定位到 WSA 目录后，执行脚本`.\Install.ps1` 来安装 WSA。

![boxcnXIu0nOVQvZJbh1Vzt1QiVb](https://cdn.sspai.com/editor/u_/cepdlalb34tdt9qfne1g?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

如果一切操作顺利，则可以看到 WSA 以及 Magisk 都已经陆续启动，你可以像在 Windows 11 上使用 WSA 那样通过 ADB 命令来安装应用，或者通过商店来安装。并且可以通过多窗口的形式来多个 Android 应用，相比运行虚拟机或者模拟器要更为方便，最重要的是，他同样支持支持显卡加速，在运行一些对图形化有要求的应用也更为稳定。不过如果你的设备较旧，那么可能依旧会存在部分应用显示不全等问题。

![boxcnMw9Eb6NkCqwXOXkg3HRNQc](https://cdn.sspai.com/editor/u_/cepdlatb34tdt9qfne20?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

## 傻瓜化的一键安装 WSA：兆懿移动应用运行平台

将 Windows 11 的 WSA 移植到 Windows 10 的做法对于不少一般用户来说还是过于复杂，一方面需要生成 MagiskOnWSA，同时还需要修改相关的配置参数。那么有没有更为简单办法来实现类似的功能呢？

兆懿移动应用运行平台就是这样一款「类 WSA」的实现方案，相比在 WSA 上直接打补丁来实现 Windows 10 上运行 Android 应用，兆懿移动应用运行平台对于系统要求更低（并不需要最新版本的 Windows 10），同时对于不具备 WSL2 支持的系统也可以得到很好的兼容，因此更适合运行较老硬件的 Windows 10 硬件。

-   优点：对于系统版本，硬件要求低
-   缺点：Android 运行时版本过低（Android 7 和 Android 9），应用启动较慢。

和通过 WSAPatch 安装类似，使用兆懿移动应用运行平台来运行 Android 应用之前，我们同样需要在「控制面板」-「程序和功能」，找到「启用或关闭 Windows 功能」，在其中找到并开启 Hyper -V 来启动虚拟化平台。

![boxcnqDryXmGAK9uhhKfv7U8WdU](https://cdn.sspai.com/editor/u_/cepdlb5b34tduhil7jpg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

重启后在兆懿移动应用运行平台下载最新的 [兆懿 2.0 Beta 1](https://sspai.com/link?target=https%3A%2F%2Fwww.wndroid.com%2FdownLoad) 安装包，然后根据步骤双击安装包进行安装，在安装过程中安装程序会完成环境配置等一系列操作，我们只需要等待完成即可。

![boxcne4aTbN70PSFSu27fx6fGrd](https://cdn.sspai.com/editor/u_/cepdlbdb34tdunrsig60?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

安装完成之后我们可以双击打开桌面上的「兆懿应用商城」，点击左上方的设置页面进行环境设置，在功能上类似于 WSA 的设置页面，我们可以设置 Android 是否常驻后台，性能上是否采用增强模式，分配给 Android 的处理器核心数以及内存数，安装的 Android 应用是否生成桌面快捷方式等等。

同样在这里还可以设置机型以及对应的快捷键等等，还可以在高级设置中调整 DPI 以及是否启用 ADB 日志打印等等。

![boxcnTbflj5UDckvmFS7Nt3KTp6](https://cdn.sspai.com/editor/u_/cepdlblb34tduhil7jq0?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

设置完成后，我们就可以通过兆懿应用商城来安装 Android 应用，下载和安装应用时会同步加载应用运行库来运行，默认兆懿采用的 Android 7.0 的运行库，对于较新的应用则可以在「我的应用」中，在右上角切换到 Android 9.0 的运行库来运行。

![boxcnKJ9zhxPb8ZDjDZIuvaXVwf](https://cdn.sspai.com/editor/u_/cepdlc5b34tdt9qfne2g?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

兆懿应用平台上支持窗口化运行多个 Android 应用。只不过其窗口 UI 并非系统原生而是经过了重新设计。同时也支持直接使用 Windows 上输入法在 Android 进行文本输入。只不过或许是显卡硬件加速支持还不够完善问题，在兆懿上运行 Android 应用会明显比 WSA 要卡顿不少，很多应用也只是勉强能够运行，但帧数确实十分「感人」。

![boxcnguf6uEeo3EwN8lWhkPY0tf](https://cdn.sspai.com/editor/u_/cepdlcdb34tduvja0ivg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

除了在商店中下载应用运行之外，兆懿也支持本地安装已经下载好的 APK 软件包运行，这一点上要比 WSA 要灵活不少（WSA 需要借助 ADB 或者辅助工具实现）。并且由于平台的限制，也无法通过类似 MagiskOnWSA 的方案，在兆懿上实现安装 Google 服务框架以及安装 Magisk 模块，所以你只是想要省心地在 Windows 10 运行 Android 应用，兆懿的这个「类WSA」方案倒是值得一试。

## 结语

在经过近一年的尝试之后，现在我们终于可以实现在 Windows 10 上通过 WSA 来运行 Android 应用，如果你恰好有这方面的需求，不妨试试以上的两种方法来在 Windows 上运行 Android 应用。

**关联阅读：**

-   [WSA patch for Windows 10](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fcinit%2FWSAPatch)
-   [一日一技 | WSA 定制安装，找回你需要的 Google 服务和 Magisk](https://sspai.com/post/75351)

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀