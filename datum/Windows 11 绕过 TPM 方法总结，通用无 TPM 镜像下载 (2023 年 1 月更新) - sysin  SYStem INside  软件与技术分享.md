更新日期：Tue Feb 14 2023 15:20:00 GMT+0800，阅读量: 25944

请访问原文链接：[Windows 11 绕过 TPM 方法总结，通用无 TPM 镜像下载 (2023 年 1 月更新)](https://sysin.org/blog/windows-11-no-tpm/ "Windows 11 绕过 TPM 方法总结，通用无 TPM 镜像下载 (2023 年 1 月更新)")，查看最新版。原创作品，转载请保留出处。

作者主页：[www.sysin.org](https://sysin.org/)

___

本文要解决的问题：

-   如何安装 Windows 11 虚拟机绕过 TPM 检测，Windows 11 ISO 虚机直装镜像下载。
-   如何在没有 TPM 或者没有 TPM 2.0 的 PC 安装 Windows 11，Windows 11 绕过 TPM 检测直装版下载。
-   如何在 Mac 上安装 Windows 11（完全没有必要，这里只是提供一种技术可行性），Windows 11 for Mac Boot Camp 直装版 ISO 下载。

![windows-11-22h2.webp](https://sysin.org/blog/windows-11-no-tpm/windows-11-22h2.webp)  
图：A3 8年磨一剑：2022 年 9 月的 22H2 版本，设置和控制面板仍然混乱，经典风格和麦德龙风格的分裂设计仍然没有改观。

![windows11-in-windows10-by-sysin](https://sysin.org/blog/windows-11-no-tpm/windows11-in-windows10-by-sysin.webp)

系统要求系统要求这些是在电脑上安装 Windows 11 的最低系统要求。如果您的设备不满足这些要求，您可能无法在设备上安装 Windows 11，建议您考虑购买 [一台新电脑](https://www.microsoft.com/zh-cn/windows/computers)。如果您不确定您的电脑是否满足这些要求，可以咨询您的原始设备制造商 (OEM)；如果您的设备已经在运行 Windows 10，您可以使用 [电脑健康状况检查应用](https://www.microsoft.com/zh-cn/windows/windows-11#pchealthcheck) 来评估兼容性。请注意，此应用不会检查显卡或显示器，因为大多数的兼容设备都能满足以下列出的要求 (sysin)。您的设备必须 [已安装 Windows 10](https://support.microsoft.com/zh-cn/windows/which-version-of-windows-operating-system-am-i-running-628bec99-476a-2c13-5296-9dd081cdd808) 的 2004 或更高版本，才能升级。可在‘设置 > 更新和安全’中的 Windows 更新功能中获取免费更新。

| **处理器** | **1 GHz** 或更快的 [支持 64 位的处理器](https://aka.ms/CPUlist)（双核或多核）或系统单芯片 **(SoC)**。 |
| --- | --- |
| **内存** | 4 GB。 |
| **存储** | 64 GB 或更大的存储设备，注：有关详细信息，请参见以下 “关于保持 Windows 11 最新所需存储空间的更多信息”。 |
| **系统固件** | 支持 UEFI 安全启动。请在 [此处](https://support.microsoft.com/topic/a8ff1202-c0d9-42f5-940f-843abef64fad) 查看关于如何启用电脑以满足这一要求的说明。 |
| **TPM** | [受信任的平台模块 (TPM)](https://docs.microsoft.com/zh-cn/windows/security/information-protection/tpm/trusted-platform-module-overview) 2.0 版本。请在 [此处](https://support.microsoft.com/windows/1fd5a332-360d-4f46-a1e7-ae6b0c90645c) 查看关于如何启用电脑以满足这一要求的说明。 |
| **显卡** | 支持 DirectX 12 或更高版本，支持 WDDM 2.0 驱动程序。 |
| **显示器** | 对角线长大于 9 英寸的高清 (720p) 显示屏，每个颜色通道为 8 位。 |
| **电脑健康检查互联网连接和 Microsoft 帐户** | Windows 11 家庭版要求具有互联网连接和 [Microsoft 帐户](https://account.microsoft.com/account)。 将设备切换出 Windows 11 家庭版 S 模式也需要有互联网连接。[在此处进一步了解 S 模式](https://support.microsoft.com/help/4020089/windows-10-in-s-mode-faq)。 所有的 Windows 11 版本都需要联网才能执行更新，以及下载和利用某些功能。有些功能需要使用 Microsoft 帐户。 |

某些 [功能需要特定硬件支持](https://www.microsoft.com/zh-cn/windows/windows-11-specifications#table2)。运行某些应用程序所需满足的系统要求可能高于 Windows 11 的最低设备规格要求。检查设备与您想要安装应用程序的兼容情况。所需的设备存储空间将根据实际的应用程序和更新而有所不同。更高端、更强大的电脑性能也较高。以后或更新时可能会有其它的要求。

> 以上为 Windows 11 的官方系统要求。

关键是这个 TPM 芯片，通常在虚拟机、MacBook，没有 TPM 或者没有 TPM 2.0 的旧 PC 无法正常安装 Windows 11（报错如下图）。

![windows-11-tpm-check](https://sysin.org/blog/windows-11-no-tpm/windows-11-tpm-check.webp)

## [](https://sysin.org/blog/windows-11-no-tpm/#2-%E7%BD%91%E4%B8%8A%E5%B8%B8%E8%A7%81%E7%9A%84%E6%96%B9%E6%B3%95)2\. 网上常见的方法

### [](https://sysin.org/blog/windows-11-no-tpm/#%EF%BC%881%EF%BC%89%E4%BF%AE%E6%94%B9%E6%B3%A8%E5%86%8C%E8%A1%A8%EF%BC%88%E4%B8%8D%E6%94%AF%E6%8C%81-Boot-Camp%EF%BC%89)（1）修改注册表（不支持 Boot Camp）

在 Windows 11 安装界面按 Shift + F10 打开命令行界面，执行如下命令：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>REG ADD HKLM\SYSTEM\Setup\LabConfig /v BypassTPMCheck /t REG_DWORD /d 1</span><br><span>REG ADD HKLM\SYSTEM\Setup\LabConfig /v BypassSecureBootCheck /t REG_DWORD /d 1</span><br></pre></td></tr></tbody></table>

或者使用图形界面的注册表编辑器添加：

输入 regedit 进入注册表编辑器，然后定位到如下位置 HKEY\_LOCAL\_MACHINE\\SYSTEM\\Setup，创建一个名为 “LabConfig” 的项，接着在 “LabConfig” 下创建两个 DWORD 值：

键名 “BypassTPMCheck”，赋值 “00000001”

键名 “BypassSecureBootCheck”，赋值 “00000001”

保存退出后，无法安装的提示就消失了。

> 优点：不用修改 ISO 文件，原版即可。
> 
> 缺点：操作稍微有点繁琐，也不容易记住。并且不支持 Mac 上 Boot Camp 安装方式。

### [](https://sysin.org/blog/windows-11-no-tpm/#%EF%BC%882%EF%BC%89%E6%9B%BF%E6%8D%A2%E6%96%87%E4%BB%B6%EF%BC%9Aappraiserres-dll%EF%BC%88%E6%97%A0%E6%95%88%EF%BC%89)（2）替换文件：appraiserres.dll（无效）

将 Windows 10 ISO 中的 appraiserres.dll（在 sources 文件夹下），替换 Windows 11 ISO 中的同名文件或者在 Windows 11 ISO 中直接删除该 dll 文件。

上述方法经过测试当前版本（2021 年 11 月）无效，可能需要特定的版本匹配。

有读者分享：新建一个空白文件命名为 appraiserres.dll 替换 Windows 11 ISO 中的同名文件。经测同样无效（2022 年 2 月更新）。

### [](https://sysin.org/blog/windows-11-no-tpm/#%EF%BC%883%EF%BC%89%E6%B7%BB%E5%8A%A0-vTPM-%EF%BC%88%E7%B9%81%E7%90%90%EF%BC%8C%E4%BB%85%E9%99%90%E8%99%9A%E6%8B%9F%E5%8C%96%EF%BC%89)（3）添加 vTPM （繁琐，仅限虚拟化）

请自行查看，不再赘述。

-   [Windows 11 on VMware ESXi - This PC can’t run Windows 11](https://www.virten.net/2021/10/windows-11-on-vmware-esxi-this-pc-cant-run-windows-11/)
-   [Workstation 16.2](https://blogs.vmware.com/workstation/2021/10/workstation-16-2-now-available.html)
-   [Fusion 12.2](https://blogs.vmware.com/teamfusion/2021/10/fusion-12-2-now-available.html)
-   [Parallels Desktop 17 中的 Windows 11 虚拟机](https://www.parallels.com/cn/blogs/windows-11-tpm/)

### [](https://sysin.org/blog/windows-11-no-tpm/#%EF%BC%884%EF%BC%89%E5%B0%86-WIM-%E9%95%9C%E5%83%8F%E5%B1%9E%E6%80%A7%E4%BF%AE%E6%94%B9%E4%B8%BA-Server)（4）将 WIM 镜像属性修改为 Server

第三方脚本和小工具有不少是采用这种方式，类似如下操作：

通过 [wimlib](https://wimlib.net/)（the open source Windows Imaging (WIM) library）将 Windows 11 ISO 中 sources 文件夹下的 install.wim 的 image-property 修改为 Server，将无需 TPM 检测。

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>wimlib-imagex.exe info install.wim --header</span><br><span># 显示 Image Count                 = <span>5</span></span><br><span>wimlib-imagex.exe info install.wim [<span>1</span>-<span>5</span>] --image-property WINDOWS/INSTALLATIONTYPE=Server</span><br></pre></td></tr></tbody></table>

示例脚本：[MediaCreationTool.bat](https://github.com/AveYo/MediaCreationTool.bat)

如果已经下载了 Windows 11 的 ISO 镜像，双击一下 Quick\_11\_iso\_esd\_wim\_TPM\_toggle.bat

然后右键点击 Windows 11 的 ISO 文件，菜单 “发送到 (N) > Quick\_11\_iso\_esd\_wim\_TPM\_toggle.bat” 即可免 TPM 补丁成功。

再次双击一下 Quick\_11\_iso\_esd\_wim\_TPM\_toggle.bat 右键菜单中的上述项移除。

该脚本实际也是将 WIM 镜像属性修改为 Server 来绕过 TPM 检测。

> 该镜像仍然不支持在 Mac 上 Boot Camp 安装方式。

某些第三方脚本也使用了该方法，经过测试虚机安装报错，未知。Boot Camp 应该也不支持，就不在验证了。

### [](https://sysin.org/blog/windows-11-no-tpm/#%EF%BC%885%EF%BC%89Rufus-%E5%88%9B%E5%BB%BA-USB-%E5%90%AF%E5%8A%A8%E7%9B%98)（5）Rufus 创建 USB 启动盘

[Rufus](https://rufus.ie/) v3.19+ 创建 USB 启动盘，增加了一个新功能可以绕过 TPM 检测，还是挺方便的。

![Rufus screenshot 4](https://rufus.ie/pics/screenshot4_en.png)

### [](https://sysin.org/blog/windows-11-no-tpm/#%EF%BC%886%EF%BC%89install-wim-%E6%9B%BF%E6%8D%A2)（6）install.wim 替换

使用 Windows 10 的安装介质，将其中的 install.wim 删除，替换为 Windows 11 安装介质中的 install.wim 并改名为 install.esd（改名仅仅是便于区分，原文件名替换也可以）。

该方法创建可分发的 ISO 文件，可能是相对最佳方案，一个 iso 文件就可以解决文首的三种问题，视为通用方法，下面具体说明一下操作步骤。

## [](https://sysin.org/blog/windows-11-no-tpm/#3-%E9%80%9A%E7%94%A8%E6%96%B9%E6%B3%95%EF%BC%9Ainstall-wim-%E6%9B%BF%E6%8D%A2)3\. 通用方法：install.wim 替换

使用以下方法创建的 Windows 11 iso 文件可以直接在以下情况安装：

-   虚拟机安装：包括不限于 VMware、KVM 和 Hyper-V 等 (sysin)
-   [在 Mac 上 Boot Camp 安装 Windows 11，如同安装 Windows 10 一样](https://support.apple.com/zh-cn/HT201468)，注意仅限 Intel 处理器的机型（不适用于 [搭载 Apple 芯片的 Mac 电脑](https://support.apple.com/zh-cn/HT211814)）
-   在老旧的 PC 上直接安装 Windows 11，只要原来可以安装 Windows 10，没有 TPM 芯片，或者只有 TPM 1.2 的版本。

分别下载 [Windows 10](https://sysin.org/blog/windows-10/) 和 [Windows 11](https://sysin.org/blog/windows-11/) 的原版 ISO 镜像。

打开 Windows 11 的 ISO 文件，提取 sources 文件夹下的 install.wim，并将 install.wim 改名为 instsall.esd 备用。**实际上，无需改名，直接使用 install.wim 替换即可，仅仅是网传要改名，好处在于改名后同原镜像容易区分。**

使用 ISO 编辑软件（如 UltraISO、WinISO、PowerISO）编辑 Windows 10 的 ISO 文件，删除 sources 文件夹下的 install.wim，然后将 Windows 11 的 install.esd 添加到该文件夹，保存（另存为）ISO 文件即可。

> 某些文章表示用以下命令将 wim 转换为 esd 格式，本例并没有使用该操作，镜像完全正常可用。
> 
> `dism /Export-Image /SourceImageFile:C:\Windows11\install.wim /SourceIndex:INDEX /DestinationImageFile:E:\Downloads\OSes\install.esd /Compress:recovery /CheckIntegrity`

直接在 Windows 中双击 setup 会提示如下，没错，Windows 10 安装程序安装 Windows 11 镜像。实际上用 ISO 或者 USB 引导安装只是启动 Logo 不同，如文首图片，Windows 11 的窗口是正方形的。

![windows10-setup-windows11](https://sysin.org/blog/windows-11-no-tpm/windows10-setup-windows11.webp)

## [](https://sysin.org/blog/windows-11-no-tpm/#4-CPU-%E9%99%90%E5%88%B6)4\. CPU 限制

在 Windows 11 系统要求中，通常内存、存储、显示和网络链接都容易满足要求。文中的方法和镜像主要是绕过 **TPM** 和 **UEFI 安全启动**的限制，但是 CPU 仍然需要满足硬性要求，详见下表中的链接。

**Windows 11 支持的处理器**：

如果进行升级安装 Windows 11，不满足系统要求，打开命令提示符，执行如下命令：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>REG ADD HKLM\SYSTEM\Setup\MoSetup /v AllowUpgradesWithUnsupportedTPMOrCPU /t REG_DWORD /d 1</span><br></pre></td></tr></tbody></table>

然后重启系统，再次进行升级安装，即可烧过 TPM 和 CPU 限制。

## [](https://sysin.org/blog/windows-11-no-tpm/#5-%E4%B8%8B%E8%BD%BD%E5%9C%B0%E5%9D%80)5\. 下载地址

下载没有 TPM 限制的 Windows 11：

Windows 11 21H2

-   Windows 11, version 21H2 (updated Jan 2022) 64-bit 简体中文 - 商业版（NoTPMCheck）  
    教育版、企业版、专业版、专业教育版、专业工作站版  
    百度网盘链接：[https://pan.baidu.com/s/1NgWFAiPcQlsxSyincypA2Q?pwd=umce](https://pan.baidu.com/s/1NgWFAiPcQlsxSyincypA2Q?pwd=umce)
    
-   Windows 11, version 21H2 (updated Feb 2022) 64-bit 简体中文 - 商业版（NoTPMCheck）  
    教育版、企业版、专业版、专业教育版、专业工作站版  
    百度网盘链接：[https://pan.baidu.com/s/1oyVXmxsDR989P5PdBGbcBg?pwd=ui9r](https://pan.baidu.com/s/1oyVXmxsDR989P5PdBGbcBg?pwd=ui9r)
    
-   Windows 11, version 21H2 (updated Mar 2022) 64-bit 简体中文 - 商业版（NoTPMCheck）  
    教育版、企业版、专业版、专业教育版、专业工作站版  
    百度网盘链接：[https://pan.baidu.com/s/1STS3qXM1WUucrAk8IF4N8g?pwd=mnhu](https://pan.baidu.com/s/1STS3qXM1WUucrAk8IF4N8g?pwd=mnhu)
    
-   Windows 11, version 21H2 (updated Aug 2022) 64-bit 简体中文 - 商业版（NoTPMCheck）  
    教育版、企业版、专业版、专业教育版、专业工作站版  
    SHA256SUM：815DF06DEF41AE24CD4758803F32D399C829B0DD2F620F3C5E63B307B4B412C8  
    百度网盘链接：[https://pan.baidu.com/s/14mk9RWE5FIkWu7NgjvyZaQ?pwd=jnvp](https://pan.baidu.com/s/14mk9RWE5FIkWu7NgjvyZaQ?pwd=jnvp)
    

Windows 11 22H2

-   Windows 11, version **22H2** (released Sep 2022) 64-bit 简体中文 - 商业版（NoTPMCheck）  
    教育版、企业版、专业版、专业教育版、专业工作站版  
    SHA256SUM：45275B20B7483F0B607DCDCCAFB886CA16B4F1433FE1C1A1D5D5943EFA97090E  
    百度网盘链接：[https://pan.baidu.com/s/1bEJ7\_C6capk6ycXnnRiLYg?pwd=huya](https://pan.baidu.com/s/1bEJ7_C6capk6ycXnnRiLYg?pwd=huya)
    
-   Windows 11, version **22H2** (**updated Jan 2023**) 64-bit 简体中文 - 商业版（NoTPMCheck）  
    教育版、企业版、专业版、专业教育版、专业工作站版  
    SHA256SUM：36C687D48D0866083C07AB89370C354D7FED5F87EE610E6CB2D179B9F0287C65  
    百度网盘链接：[https://pan.baidu.com/s/1nQBXM8RaC9ImJmGNPsGVxQ?pwd=ii30](https://pan.baidu.com/s/1nQBXM8RaC9ImJmGNPsGVxQ?pwd=ii30)
    

更多下载：

-   Windows 11 官方版本下载：[Windows 11 22H2 中文版、英文版 (x64、ARM64) 下载 (updated Feb 2023)](https://sysin.org/blog/windows-11/ "Windows 11 22H2 中文版、英文版 (x64、ARM64) 下载 (updated Feb 2023)")
-   Windows 10 官方版本下载：[Windows 10, version 22H2 (released Oct 2022) 简体中文版、英文版下载](https://sysin.org/blog/windows-10/ "Windows 10, version 22H2 (released Oct 2022) 简体中文版、英文版下载")
-   Windows 10 LTSC 官方版本下载：[Windows 10 Enterprise LTSC 2021 简体中文版、英文版下载](https://sysin.org/blog/windows-10-ltsc-2021/ "Windows 10 Enterprise LTSC 2021 简体中文版、英文版下载")

___

[![点击访问官方网站](https://sysin.org/img/banner-qcloud.webp?v20230203)](https://cloud.tencent.com/act/cps/redirect?redirect=2446&cps_key=78f957e545ecb2e5e2720d6ccf030209&from=console)

___

文章用于推荐和分享优秀的软件产品及其相关技术，所有软件默认提供官方原版（免费版或试用版），免费分享。对于部分产品笔者加入了自己的理解和分析，方便学习和测试使用。任何内容若侵犯了您的版权，请联系作者删除。如果您喜欢这篇文章或者觉得它对您有所帮助，或者发现有不当之处，欢迎您发表评论，也欢迎您分享这个网站，或者赞赏一下作者，谢谢！

赞 ![](https://sysin.org/img/alipay.webp)**支付宝赞赏** ![](https://sysin.org/img/wechatpay.webp)**微信赞赏**

赞赏一下

___

☑️ 评论恢复，欢迎留言❗️  
敬请注册！点击 “登录” - “用户注册”。社交媒体联合登录无法接收通知邮件。已知不支持 21.cn/189.cn 邮箱。

站长您好，请问SHA256SUM值是多少？我下载的文件SHA256SUM值是36c687d48d0866083c07ab89370c354d7fed5f87ee610e6cb2d179b9f0287c65

感谢站长分享，赞👍

[@ChinaCXO](https://sysin.org/blog/windows-11-no-tpm/#63de4ba0ea4a7600083faff4): 老总好，您的捐赠已经退还，谢谢。下次别再这样了，发一堆要求，实在无法达到您的要求。不过还是感谢您的支持。

我使用上面连接下载的WIN11 NOTPMCHECK的ISO，持在ESXI的WIN10虚拟机，打开虚拟光驱的SETUP，显示的是WIN10升级检测，没有WIN11的提示。(已在WIN10注册表添加BypassTPMCheck键)

[@wjsc](https://sysin.org/blog/windows-11-no-tpm/#634f82ce9d78c7177a1f7e57): 升级成功了，显示升级WIN10估计是替换了某个文件的误显。  
升级前，先断网，双击setup.exe，不用等界面出来，进C盘$WINDOWS.~BT\\Sources把appraiserres.dll删掉。  
就可以正常升级了。只是全程显示WIN10升级中![qq_grin](https://unpkg.com/@waline/emojis@1.0.1/qq/qq_grin.gif)

install.wim替换对于21390版本win10用户不适用（仅指想保全所有数据升级的用户）  
#个人猜测原因是：win10最新版 21H2 内部版本号19044比win10最后一个预览版内部版本号21390低  
如果用win10.0.21390.2025的安装ISO做嫁衣可能能解决该问题（我没试过）

appraiserres.dll替换或移出法（正确方法）：

1、打开ISO镜像里的升级程序

2、前往系统盘一般为C盘$WINDOWS.~BT\\Sources文件夹\*1，

打开Sources子文件夹，并找到其中的appraiserres.dll文件（先别删或替换或将其移出该文件夹）

【目录参考C:$WINDOWS.~BT\\Sources】，【注意：这一步要快！！！】

3、返回升级程序等它出现“已安装这些更新，但您需要重新启动Windows11安装程序才能使这些更新…”时赶紧返回系统盘的$WINDOWS.~BT\\Sources文件夹，

删除或用win10的appraiserres.dll替换或将其移出$WINDOWS.~BT\\Sources文件夹

【这一步要赶在升级程序重新启动完之前完成！！！】

4、等待升级程序自动重启并正常升级【期间可能会出现让你放弃保修的协议，如果你不想放弃保修请关闭该升级程序，如果你坚持升级win11请点接受，并选择继续】

\*1：$WINDOWS.~BT\\Sources文件夹为隐藏文件夹，让其显示方法：打开文件资源管理器，选“查看”（在菜单栏上），找到“隐藏的项目”复选框并勾选

[@FFF](https://sysin.org/blog/windows-11-no-tpm/#633b83315bfde245cb9df5c5): 网页可能无法正常正常显示我的评论，在这里我重新说一下  
【必须按我说的操作顺序来！】  
1、打开ISO里的升级程序  
2、打开系统盘$WINDOWS.~BT\\Sources

这个文件夹是隐藏文件夹，可以通过打开文件资源管理器  
选“查看”菜单，勾选隐藏的项目让其显示  
目录参考C:$WINDOWS.~BT\\Sources  
【这步要赶在升级程序重启前完成！】

3、返回升级程序，等到它出现“重启Windows 11安装程序”时快速返回系统盘$WINDOWS.~BT\\Sources  
并找到appraiserres.dll文件，将其用相同win10的appraiserres.dll文件替换或直接删除或移出  
【这步要赶在升级程序重新启动完成前完成！】  
4、按升级程序引导正常升级  
可能会出现让你放弃保修的协议，如果你想升级请点“接受”并选择“继续”不想升级直接关闭升级程序即可

[@FFF](https://sysin.org/blog/windows-11-no-tpm/#633b898e06e24022ea8b7527): 该方法对win11 22h2 版本有效  
满足win11 21H1 版本CPU最低要求但不满足win11 22H2 版本CPU最低要求的用户也可以升级

[@FFF](https://sysin.org/blog/windows-11-no-tpm/#633b898e06e24022ea8b7527): 如果安装过程中卡在31%，请参考 \`https://blog.csdn.net/qq37724861/article/details/127020965\`

请问安装后能正常刚更新吗？

[@kaven](https://sysin.org/blog/windows-11-no-tpm/#62b6d79a1054e678a050134d) , A3 系统强烈建议禁用自动更新，否则更新个半年，装数十数百个垃圾软件，几乎报废（重装）。

> 该镜像任然不支持在 Mac 上 Boot Camp 安装方式。

应该是「仍然」吧 ![doge](https://img.t.sinajs.cn/t4/appstyle/expression/ext/normal/a1/2018new_doge02_org.png)

修改过的IOS镜像直接用软碟通制作U盘启动 无法引导呢

[@ZERO](https://sysin.org/blog/windows-11-no-tpm/#623e8b183e94a2115acc8319) , 与修改无关，工具用错了，写U盘工具，a3官方可以使用Windows USB/DVD Download Tool，第三方使用rufus，要在Unix-Like系统下写入使用unetbootin，等等还有很多。

[@sysin](https://sysin.org/blog/windows-11-no-tpm/#623e92819649363f624fe114) , 没太明白 之前WIN10的一直都用软碟通写的 我试试你推荐的工具

[@sysin](https://sysin.org/blog/windows-11-no-tpm/#623e92819649363f624fe114) , 镜像选择你的 ESD还是WIM 有什么区别吗

[@ZERO](https://sysin.org/blog/windows-11-no-tpm/#623eaf433e94a2115acc9473) , 如果是 A3 系统都是用Windows USB/DVD Download Tool，官方出品。

[@ZERO](https://sysin.org/blog/windows-11-no-tpm/#623eb47c9649363f624ffc6b) , 其实没有区别，相同的文件改了不同的扩展名，为了验证网传谣言不实，文中有描述。

[@sysin](https://sysin.org/blog/windows-11-no-tpm/#623ec4919649363f6250012d) , 刚才用两个工具都写过 尝试用微软PRO4 启动 还是不能引导 提示没有发现可引导文件

[@ZERO](https://sysin.org/blog/windows-11-no-tpm/#623ec8659649363f62500491) , 早说啊，Surface 一般用专用镜像，某些机型可以用通用镜像+驱动。搜索了一下 Surface Pro 4 有 TPM 2.0 和 Secure Boot，但是 CPU 不受官方支持，本镜像意在绕过 TPM 和 安全启动，无法支持。

[@ZERO](https://sysin.org/blog/windows-11-no-tpm/#623ec8659649363f62500491) , 另外网传文章预览版可以支持这个机型，有可能在22H2的更新中提供官方支持。

___