一个比较残忍的现实是：因为近几年 Google 越来越喜欢将平台特性打包进 Google 服务进行分发，在没有 Google 服务或没有完整 Google 服务的 Android 设备上，想要体验[照片选择器](https://sspai.com/link?target=https%3A%2F%2Fdeveloper.android.com%2Fabout%2Fversions%2F13%2Ffeatures%2Fphotopicker%3Fhl%3Dzh-cn)、[附近分享](https://sspai.com/link?target=https%3A%2F%2Fsupport.google.com%2Ffiles%2Fanswer%2F10514188%3Fhl%3Dzh-Hans)等特性，感受完整、纯粹的 Android 体验，通过诸如亚马逊海外淘这样的渠道获取一台 Google Pixel 设备是为数不多的选择之一。

**种草预警：**

-   [升级也要图个明白：8000 字详解 Android 13 正式版的 9 个新变化](https://sspai.com/post/75272)
-   [大船靠岸，激动暂缓：以 Pixel 为例谈二手/水货 Android 手机验机](https://sspai.com/s/3Rdl)
-   [挑台二手机，体验原生味：银河系 Google Pixel 漫游指南](https://sspai.com/post/75385)
-   [一派 | 你为什么选择 Google Pixel？](https://sspai.com/post/65767)

但当你最终入手一台 Google Pixel，就还得面对另一个比较残忍的现实：即便你拥有基础的网络条件，Google 有意或无意（大多数时候是前者）间也屏蔽了很多大陆地区用户可以访问的功能。因此想要解锁完整的 Pixel 使用体验一般还得适当折腾一番——此前我们已经介绍过为 Pixel 5、Pixel 6 等机型开启国内 5G 网络的方法，今天这篇文章我想借近期由韩国开发者 @[kyujin-cho](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fkyujin-cho) 发现的免 root 解锁 VoLTE 功能方案，分享一下 Pixel 7 系列机型的网络配置指南。

## 再聊解锁与 root

尽管都是为了感受最原汁原味的 Android，国内 Pixel 用户对使用体验的期望依然各有不同：看重 OTA 体验，希望尽可能减少玩机操作甚至 root 的影响，一般会选择仅解锁 Bootloader；喜欢完全掌控设备，享受 Magisk 模组带来的便利与实用性的朋友，解锁 Bootloader 之外则一定会选择 root。

无论你是哪种 Pixel 用户，我都建议在可能的情况下对 Bootloader 进行解锁。虽然理论上安全性会有所降低，但解锁状态的 Bootloader 另一方面也是各种翻车情况下进行恢复的急救通道（通过刷写工厂镜像）。

至于解锁流程就是老生常谈了，如果你对 Pixel 近几代机型不太熟悉，需要注意的地方是 Pixel 7 这样的新机型解锁 Bootloader 对应的命令是：

```
fastboot flashing unlock
```

而非此前的 `fastboot oem unlock`。解锁完成后，如果你想进一步安装 Magisk 并获取 root 权限，对 Pixel 7 机型而言操作也有一点特殊。

**下面的部分为 root 教程，没有需求的用户请选择性跳过。**

### 准备工作

下载最新的、适用于本机型的、完整的镜像文件，这三个条件请注意检查，任何一个错了都会导致「翻车」。更重要的是，请保证访问[这个页面](https://sspai.com/link?target=https%3A%2F%2Fdevelopers.google.com%2Fandroid%2Fimages)时页面语言为英文。中文页面存在更新不及时的问题。

**首先我们提取需要破解的镜像**。将下载下来的完整镜像压缩包解压，得到下图所示文件列表：

![boxcnYDjYmkQ2WmfbL4Weznt7tf](https://cdn.sspai.com/editor/u_/cfhp7utb34tejhhcqdog?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

然后打开如图所示红框中的压缩包，得到下图所示文件列表：

![boxcn1WF640zeVuUAVbYMtJguTb](https://cdn.sspai.com/editor/u_/cfhp7v5b34tejvlnkr00?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

这里就是 Pixel 7 和此前的 Pixel 机型最大的不同了：

**Pixel 7 开始我们使用的是 init\_boot.img 进行破解，而非此前的 boot.img！**

**Pixel 7 开始我们使用的是 init\_boot.img 进行破解，而非此前的 boot.img！**

**Pixel 7 开始我们使用的是 init\_boot.img 进行破解，而非此前的 boot.img！**

因此我们将 **init\_boot.img** 提取出来，扔进手机里，放到哪里不重要，待会儿能找到就行。

### 破解 init\_boot.img

然后我们前往 Magisk 的 GitHub 页面，下载 [Magisk Canary](https://sspai.com/link?target=https%3A%2F%2Fraw.githubusercontent.com%2Ftopjohnwu%2Fmagisk-files%2Fcanary%2Fapp-release.apk)，下载完成后用你喜欢的方式安装到手机上（`adb install` 或者传输后手动安装均可）。安装完成后开始破解镜像文件：

1.  打开 Magisk Manager，选择最上方的「安装 > 选择并修补一个文件」，然后找到刚刚扔进手机的 **init\_boot.img** 执行破解
2.  破解成功后，Magisk 会在 Download 这个目录下生成一个破解好的镜像文件，找到它，然后把它扔回电脑

### 更新系统

下面我们需要将手机的系统更新至与所破解的 init\_boot.img 相同的版本（就是你在上面下载的这个版本），否则可能会无法开机。

将手机重启至 Bootloader，在 Mac 上，刷机用到的脚本是 flash-all.sh，而在 Windows 上是 flash-all.bat。

**如果你尚未配置手机，手机里没什么要保留的数据**，那么直接运行脚本刷入即可，刷完手机会自动开机，开机后你可以跳转至下一小节「进行 root」继续操作。

**如果你已经配置了手机，手机里有数据需要保留**（比如每个月的安全更新发布了，你重新下载了最新版系统的完整工厂镜像），那么在获取并解压工厂镜像后，我们需要打开脚本文件进行编辑。

具体而言：

1.  打开脚本后，你能在最后一行看到类似 `fastboot -w update image-xxxxxx.zip` 这样的语句
2.  将这句更改为 `fastboot --skip-reboot update image-xxxxxx.zip`

其实就是将 `-w` 去掉了，替换成了 `--skip-reboot`，这样做的目的是避免刷机脚本清除用户数据，同时防止刷机完成后手机自动开机（开机前我们要先 root 啦）。

额外提一点，之前网上传过保留 Magisk 进行 OTA 更新的方法，少数派也写过，但目前并不适用于 Pixel 7！

对了如果你选择了 `--skip-reboot`，刷机脚本跑完后手机应该会停留在 fastbootd 界面，注意识别菜单选项，用音量键选择重启到 bootloader 然后电源键确认，即可继续下面的操作。

### 进行 root

接下来我们就可以进行 root 了：

1.  手机重启至 bootloader 并连接至电脑
2.  命令行执行 `fastboot flash init_boot 破解后的镜像.img`
3.  完成后重启，完事~

### 善后工作

为了保证 Pixel 7 Pro 在 root 后也能正常下载、安装和更新 Netflix，我建议在 root 完成并开机后先别干的，**优先执行 SafetyNet 相关的环境配置工作**。

SafetyNet 的介绍有兴趣可以读这个（文章里的方法已经过时了）：[Android 玩机「神器」的矛盾与新生：Magisk Canary 更新详解](https://sspai.com/post/69945)

1.  下载 [SafetyNet Fix Mod](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FDisplax%2Fsafetynet-fix%2Freleases) 和 [Shamiko](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FLSPosed%2FLSPosed.github.io%2Freleases) 这俩模块，然后扔手机里备用
2.  打开 Magisk Manager，安装上面两个模块，然后重启（对了，这一步你可以把一些额外的模块也顺手装上再重启，减少重启次数）
3.  重启后，打开 Magisk Manager，点击右上角的齿轮图标进入设置，勾选上 Zygisk 开关
4.  然后点击「配置排除列表」，在排除列表右上角菜单里勾上「显示系统应用」，然后搜索下面这几个东西并勾选（中文系统下名称/英文系统下名称）
5.  Google Play 商店/Google Play Store
6.  Google 服务框架/Google Play Framework
7.  Google Play 服务/Google Play Services

___

模块推荐

既然提到了安装模块，这里推荐一个个人认为可以提升幸福感的模块：[NotoSansCJK (v21\_lite)](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fsimonsmh%2Fnotocjk%2Freleases%2Ftag%2F21_lite)，思源黑，动态字重，粗细分明，原生必备。

___

最后为了保险，我们可以再去 Google 这个页面注册一下设备对应的 GSF ID：

关于 GSF ID 是什么，参考：[应用商店里搜不到 Netflix？你可能遗漏了这一步 | 一日一技](https://sspai.com/post/55536)

1.  APKMirror 下载、安装一个 [Inware](https://sspai.com/link?target=https%3A%2F%2Fwww.apkmirror.com%2Fapk%2Fevowizz%2Finware%2Finware-6-3-34-release%2Finware-6-3-34-android-apk-download%2F) 到手机上
2.  打开 Inware，打开「设备」信息模块，翻到最下面，找到 **GSF ID**
3.  电脑端打开[这个页面](https://sspai.com/link?target=https%3A%2F%2Fwww.google.com%2Fandroid%2Funcertified%2F)，输入 GSF ID、完成人机验证，然后点击注册
4.  如果你的设备在这个操作之前已经连接过能够正常访问 Google 服务的网络，保险起见，建议在应用列表里清除 Google Play 商店和 Google Play 服务这俩应用的应用数据
5.  如果没有，那么重启

重启后，连接能够正常访问 Google 服务的网络并完成新用户配置流程，在 Play 商店的「设置 > 关于」信息中，你应该就能看到「设备已通过认证」的字段了。

![boxcnaTDBbWMENZ6AVhLDwaFC8d](https://cdn.sspai.com/editor/u_/cfhp7vdb34tej9i9bap0?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

## 免 root 解锁 VoLTE 功能

如果第一部分的内容与你无关，或者你已经被第一部分内容的篇幅吓到了，这一部分就是为你准备的——在没有 root 的情况下，为 Pixel 7 系列（其实也同样适用于 Pixel 6 系列）解锁国内运营商 VoLTE 相关功能。

VoLTE 的基础性和重要性本文不再赘述了，对 Pixel 6/7 这两代设备而言，**VoLTE 功能同时也是电信用户正常使用通话功能的前提**。因此这里推荐没有 root 的用户选择近期由韩国开发者 @[kyujin-cho](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fkyujin-cho) 分享的免 root VoLTE 功能解锁方案。

说起 Android 这边的「免 root 玩机」话题，Shizuku 这款工具自然是少不了的。所幸 undefined此前已经分享过非常相近的介绍和配置方法，因此本教程的第一部分「Shizuku 配置」，请移步至下面这篇文章了解。

**关联阅读：**[别被 root 挡在门外：Shizuku 让 Android 玩机更简单](https://sspai.com/post/73294)

确保 Shizuku 服务已经正常运行之后，前往 Pixel IMS 的[发行版](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fkyujin-cho%2Fpixel-volte-patch%2Freleases)页面下载安装该工具的最新版本。安装完成后启动 Pixel IMS，此时你应该能够看到一个 Shizuku 接口调用的权限请求，点击「总是允许」：

![boxcnFqxDAILxxtv04yltSxzHNb](https://cdn.sspai.com/editor/u_/cfhp7vdb34tejhhcqdp0?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

Shizuku 授权 | 图：kyujin-cho

随后进入 Pixel IMS 的主界面（希望开发者后续能适配个 Material 3 不知算不算过分），点击左下角的 ENABLE VOLTE 按钮进行激活。激活后，VoLTE Status 区域下的 VoLTE Enabled by Config 选项开关会自动变成启用状态。

![boxcnqERtnhFMMvZS5VGkA0vAdf](https://cdn.sspai.com/editor/u_/cfhp7vlb34tej9i9bapg?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

![boxcnyMWhBSi0vpciktrm1zXrSg](https://cdn.sspai.com/editor/u_/cfhp7vtb34tejvlnkr0g?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

启用过程其实很简单 | 图：kyujin-cho

重启设备后你应该就能看见对应 SIM 设置中的 VoLTE 选项开关了。至此，电信用户已经可以在 Pixel 6/7 系列机型上拥有完整的 4G 网络体验。

**参考链接：**[pixel-volte-patch | GitHub](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fkyujin-cho%2Fpixel-volte-patch)

## root 后解锁完整网络体验

而如果你选择了 root 和 Magisk 这条路，root 权限则能在网络体验这件事情上为你带来更加完整的便利。本部分将基于此前的解锁流程简单介绍一下 **Pixel 7 系列机型的 5G 网络、Vo5G 通话开启方法**。

**关联阅读：**[一日一技 | 为 Google Pixel 6 解锁国内 5G 网络支持](https://sspai.com/post/73886)

还是从 VoLTE 说起，你可以参考去年的教程使用 [Magisk 模块](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FCHN-MuXin%2FMagiskModuleEnableChinaForVoTELtoPIxel)来解锁 VoLTE，不过我在后续的使用和测试过程中发现，对 root 用户而言即便不用模块也能打开 VoLTE 的相关设置。具体流程如下：

1.  前往 Google Play 商店下载 [MiX 文件管理器](https://sspai.com/link?target=https%3A%2F%2Fplay.google.com%2Fstore%2Fapps%2Fdetails%3Fid%3Dcom.mixplorer.silver)
2.  打开 MiX 文件管理器，然后导航至 /data/user\_de/0/com.android.phone/files 路径，过程中如果弹出 root 权限请求请选择授予
3.  在上述路径中找到 carrierconfig-com.google.android.carrier-xxxxxxx-xxx.xml 文件并使用 MiX 文件管理器内置的代码编辑器打开
4.  定位至编辑器末尾，在 `</init-array>` 和 `</bundle>` 中间，粘贴以下字段（#后为注释记得去掉）：

```
<boolean name="enhanced_4g_lte_on_by_default_bool" value="true" /> #默认开启 LTE
<boolean name="carrier_volte_available_bool" value="true" /> #开启 VoLTE 功能
<boolean name="vendor_hide_volte_settng_ui" value="false" /> #显示 VoLTE 设置
<boolean name="hide_lte_plus_data_icon_bool" value="false" /> #隐藏 LTE+ 图标
<boolean name="vonr_enabled_bool" value="true" /> #开启 Vo5G 功能
<boolean name="vonr_setting_visibility_bool" value="true" /> #显示 Vo5G 设置
<boolean name="show_4g_for_lte_data_icon_bool" value="true" /> #将 LTE 网络标识显示为「4G」，可选
```

这里需要注意的是，如果你是 eSIM 双卡双待用户，那么可能会像下图这样在 com.android.phone/files 路径中看到不止一份配置文件。如果你难以分辨哪一个对应想要开启的运营商，可以选择都加上。毕竟没什么害处。

搞定 VoLTE 和相关设置可见性之后别急着重启，接下来我们还是像去年一样，通过[网络信号大师](https://sspai.com/link?target=https%3A%2F%2Fplay.google.com%2Fstore%2Fapps%2Fdetails%3Fid%3Dcom.qtrun.QuickTest)这款应用来解锁 5G 网络。

具体配置流程还是参考去年的教程，唯一需要注意的是，Pixel 7 系列用户的配置流程与 Pixel 6 略有不同，需要在「NR MODE SETTING」里选择 SA 或者 NSA & SA（其实相对更简单了）。下方的 NR Band Combo 可随意（FR2 对应 mmWave 毫米波通讯功能，美版会有、国内没用）。

![boxcnmu7Idt5AJd1DLt2NVznJcf](https://cdn.sspai.com/editor/u_/cfhp805b34tejhhcqdpg?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

![boxcnr1T0PBUxG56l5rxCRPH0gc](https://cdn.sspai.com/editor/u_/cfhp80db34tej9i9baq0?imageView2/2/w/1120/q/40/interlace/1/ignore-error/1)

完成所有操作后重启手机，然后你应该就能在设置中看到 5G 网络、VoLTE、5G 通话以及 2G 网络开关等相关设定了。

## 小结

简而言之，如果你是没有 root 的用户，通过本文第二部分教程，简单几步便能在国内解锁电信网络和三大运营商的 VoLTE 功能；如果你是 root 用户，通过本文第三部分教程则可以拥有完整的 5G 网络体验。

值得一提的是，目前正在测试的 Android 13.1 QPR Beta 版本在最近的更新中为印度地区用户解锁了 5G 网络支持，同样作为非发售地区的中国大陆后续是否有希望少些折腾、多些便利，就看 Google 会不会良心发现了。

最后感谢为 Pixel 玩机社区不断贡献、为大家带来便利的所有开发者。如果本文对你有所帮助，请考虑为相关 GitHub 页面加上星标或帮助教程原始作者进行传播分享。

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀