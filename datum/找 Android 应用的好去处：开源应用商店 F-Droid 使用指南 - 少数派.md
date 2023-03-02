找 Android 应用的好去处：开源应用商店 F-Droid 使用指南

我的上一篇文章中提到了 [Fennec F-Droid](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Forg.mozilla.fennec_fdroid%2F) 和 [Mull](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fus.spotco.fennec_dos%2F) 两个 Firefox 的第三方构建版本，这两个应用都是上架在 F-Droid 上面的。不过 F-Droid 在中文互联网环境中知名度并不高，也很少有相关的介绍文章，我在少数派站内只找到了一篇写于两年前的[文章](https://sspai.com/post/63647)，这篇文章中的一些内容已经过时。

所以我写下这篇文章，旨在更加全面地介绍 F-Droid，并给出一些更新的使用指南。

F-Droid 是一个 Android 应用程序的软件资源库；其只包含 FOSS（Free and Open Source Software，自由开源软件）的应用程序。2010 年，F-Droid 由 Ciaran Gultnieks 创建。客户端是复刻 [Aptoide](https://sspai.com/link?target=https%3A%2F%2Fen.aptoide.com%2F) 的源代码。该项目现由英格兰的非盈利机构 F-Droid Limited 运营。

我们也可以简单地把它理解为应用商店，但不同于其他常见的应用商店（如 Google Play）大多都要求注册账号、分发的应用以专有软件为主，F-Droid 不要求用户注册账号，官方存储库中的应用几乎全都是自由软件，甚至客户端都不是强制性的，如果你愿意，完全可以只从他们的官网下载安装应用。

## 官方客户端的下载与使用

虽然你可以直接从 F-Droid 的[官网](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2F)搜索并下载应用，但这样未免有些太过麻烦，所以还是推荐使用客户端来管理应用，由于 Google Play 开发者分发协议中的反竞争条款，F-Droid 官方客户端不能通过 Google Play 商店安装，不过可以通过官网下载到[安装包](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2FF-Droid.apk)并安装，你也可以在[这里](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2FF-Droid.apk.asc)获取安装包的 PGP 签名来验证安装包文件是否完整或被篡改，安装后也可以通过客户端来管理自身的更新。

第一次打开客户端，需要等待其更新存储库，后续需要检查更新只需在主界面下拉刷新即可更新存储库。如果你没有互联网连接，也可以通过下方的 `附近` 通过其他人的网络共享安装应用，也可在设置中打开 `扫描可移动存储器` 来加载外接存储设备中的存储库，不过这两个功能估计几乎没有人用吧，这里只是简单提一下。

![](https://cdn.sspai.com/2023/01/04/9c974e992115666ea6c87193bbfd44a8.png)

官方客户端的界面其实很简洁明了， 最下方分为了五个常用分区，右下角的搜索按钮可以搜索应用，虽然客户端界面支持中文，但对中文搜索的支持并不是很好，建议搜索的时候还是尽量使用英文。

![](https://cdn.sspai.com/2023/01/04/0e25f3086a4058fb48fe2fc8cbd7e009.png)

F-Droid 中的应用若包含广告、应用分析器、追踪器或依赖非自由软件等等，会被标记为存在「负特征（antifeatures）」，若你不希望看到某些包含负特征的应用，可以在设置中找到 `包含带有负特征的应用`，来禁用特定的负特征，若你并不是隐私狂人，并不十分不在乎应用是否存在负特征，则还是推荐勾选上所有的负特征。

![](https://cdn.sspai.com/2023/01/04/d0de4a47775e79305a8e820a9adf38c5.png)

### 存储库

F-Droid 分发应用的策略，有点类似于大部分 Linux 发行版包管理器的软件源，应用是以存储库（Repositories）的形式存储的，除了官方维护的存储库，用户也可以自行添加第三方维护的存储库来扩充软件列表。在设置中找到 `存储库` 选项，可以看到官方客户端除了内置了官方存储库，还有一个名为 Guardian Project 的第三方存储库，以及其各自旧版本的备份。

![](https://cdn.sspai.com/2023/01/04/8175c6a6ef0a6ad4dc9c686922a05a1e.png)

#### 官方存储库

和 Linux 软件源类似，F-Droid 官方存储库中的软件，几乎全部都是 F-Droid 的维护者从各个软件的源代码编译打包并签名的，你可以下载安装 F-Droid [Build Status](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fde.storchp.fdroidbuildstatus%2F) 来监控应用的构建状态。也正因为如此，同一个应用在其他渠道下载到的安装包与在 F-Droid 下载到的安装包签名往往是不同的，因而往往不能相互覆盖安装，版本更新也会慢一些，并且有些应用的 F-Droid 版本与其他渠道的版本在功能上还会有所不同，如开源的消息推送服务 [ntfy](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fio.heckel.ntfy%2F)，相比与其在 Google Play 上架的版本，其 F-Droid 版本移除了一些专有组件，如 Google Play Service 和 FCM 推送功能等；还有 RSS 阅读器 [Read You](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fme.ash.reader%2F)，最新版本添加了 Fever API 的支持，但是其 F-Droid 版本的 Fever API 当前不可用，这是一个[已知问题](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FAshinch%2FReadYou%2Fissues%2F299)，会在下个版本修复。在使用 F-Droid 安装应用时，这点需要格外注意。

本文完成时，官方库中共收录了 4010 个应用，相比于其他应用商店，少了很多，不过我看到一些一两年前的文章提到当时的官方库大约只有一千多应用，可以看出 F-Droid 收录的应用数量还是在稳步增加的。

同样，类似于 Linux 软件源，F-Droid 官方存储库也存在许多镜像源，在 `设置`\-`存储库`\-F-Droid 中，可以看到其已经内置了很多官方镜像，我们也可以手动添加镜像，我搜索到的国内 F-Droid 的镜像有清华大学开源软件镜像站的[镜像](https://sspai.com/link?target=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fhelp%2Ffdroid%2F)和中国互联网络信息中心的[镜像](https://sspai.com/link?target=https%3A%2F%2Fmirrors.cnnic.cn%2Fhelp%2Ffdroid%2F)，从网站界面也可以看出，中国互联网络信息中心的镜像站也是使用清华大学镜像站的源码创建的。以清华大学的镜像为例，复制[此链接](https://sspai.com/link?target=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Ffdroid%2Frepo%2F%3Ffingerprint%3D43238D512C1E5EB2D6569F4A3AFBF5523418B82E0A3ED1552770ABB9A9C9CCAB)，打开 F-Droid，找到 `设置`\-`存储库`，点击右上角的加号，软件会自动识别剪贴板中的链接，点击添加镜像，会自动添加为 F-Droid 的用户镜像，再次点击 F-Droid，就会发现，在官方镜像下方，多出了我们手动添加的用户镜像，取消勾选所有的官方镜像，这样以后存储库更新就会通过我们添加的国内镜像更新，速度会快上不少。

![](https://cdn.sspai.com/2023/01/04/db056fe6d2390c008ae63bca52283041.png)

#### 第三方存储库

因为 F-Droid 对其收录的应用要求十分严格，很多应用会因为种种原因并没有被收录进 F-Droid 的官方存储库，所以很多开发者会选择自建 F-Droid 存储库，或者收录进其他的第三方存储库，官方论坛中的这个[帖子](https://sspai.com/link?target=https%3A%2F%2Fforum.f-droid.org%2Ft%2Fknown-repositories%2F721)列出了所有已知的第三方存储库，这些存储库大部分都是一些未收录进官方库的应用的开发者自建的存储库，一般每个存储库只收录了少数几个应用，如 [Bitwarden 客户端](https://sspai.com/link?target=https%3A%2F%2Fmobileapp.bitwarden.com%2Ffdroid%2Frepo%2F%3Ffingerprint%3DBC54EA6FD1CD5175BCCCC47C561C5726E1C3ED7E686B6DB4B18BAC843A3EFE6C)，所以除非是对某些应用有需求，一般不需要添加太多的第三方存储库，还有少数的存储库收录的应用较多，我这里只介绍两个第三方存储库。

一个是 [Guardian Project](https://sspai.com/link?target=https%3A%2F%2Fguardianproject.info%2Ffdroid%2Frepo%3Ffingerprint%3DB7C2EEFD8DAC7806AF67DFCD92EB18126BC08312A7F2D6F3862E46013C7A6135)，这个存储库被内置进了官方客户端，但默认没有启用，只收录了 14 个应用，基本都是隐私保护相关的应用，如 [PixelKot](https://sspai.com/link?target=https%3A%2F%2Fguardianproject.info%2Fapps%2Finfo.guardianproject.pixelknot%2F)，使用 [f5-steganography](https://sspai.com/link?target=https%3A%2F%2Fcode.google.com%2Farchive%2Fp%2Ff5-steganography%2F) 算法将文字加密进图片里。如果有需要，可以启用这个存储库。

另一个是 [IzzyOnDroid](https://sspai.com/link?target=https%3A%2F%2Fandroid.izzysoft.de%2Frepo%2Finfo)，共收录了约一千多个应用，应该是除官方库以外收录应用最多的存储库了，其中不乏一些国内开发者的应用，如 adb 授权工具 [Shizuku](https://sspai.com/link?target=https%3A%2F%2Fapt.izzysoft.de%2Ffdroid%2Findex%2Fapk%2Fmoe.shizuku.privileged.api)，开源阅读软件 [Legado](https://sspai.com/link?target=https%3A%2F%2Fapt.izzysoft.de%2Ffdroid%2Findex%2Fapk%2Fio.legado.app.release)，系统管理工具 [Thanox](https://sspai.com/link?target=https%3A%2F%2Fapt.izzysoft.de%2Ffdroid%2Findex%2Fapk%2Fgithub.tornaco.android.thanos)，应用备份软件 [DataBackup](https://sspai.com/link?target=https%3A%2F%2Fapt.izzysoft.de%2Ffdroid%2Findex%2Fapk%2Fcom.xayah.databackup) 等等，推荐添加此存储库。

两年前少数派的[这篇文章](https://sspai.com/post/63647)还推荐了 [Rakshazi](https://sspai.com/link?target=https%3A%2F%2Fgitlab.com%2Frakshazi%2Ffdroid) F-Droid，但这个存储库目前已经停更，故不做推荐。

据我所知，目前的已知的的三方存储库都没有国内镜像源，故更新存储库时速度可能会慢很多。

**注意，F-Droid 官方并不为第三方存储库中的应用质量负责，而是完全取决于存储库的维护者，可能存在安全问题，甚至可能存在后门软件，添加时请注意鉴别**

## 第三方客户端

F-Droid 的官方客户端可能是考虑到旧设备的兼容性问题，软件的架构较老，界面也并不是十分的现代，我自己在使用的过程中还遇见过它有异常占用很多存储空间的问题。得益于自由软件开放的特性，F-Droid 存在许多第三方客户端，如果你的设备没有那么老旧，且并不关心一些不常用功能，如局域网共享，还是推荐使用一些第三方客户端，它们有着更现代化更美观的界面，往往还提供更多功能。

过去被推荐的比较多的两个第三方客户端，[Foxy Droid](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fnya.kitsunyan.foxydroid%2F) 和 [Aurora Droid](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fcom.aurora.adroid%2F) 都已经很久没更新了，故不再推荐，我这里推荐另外两个还在积极维护更新的客户端。

[Droid-ify](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fcom.looker.droidify%2F)，从 Foxy Droid 修改而来，相比于官方客户端更新存储库更快；采用了最新的 Material You 设计语言，在 Android 12 及以上系统甚至还支持系统的动态取色，笔者目前的主力机系统还是 Android 11，可惜目前无法演示这个功能；支持 Shizuku 或 root 授权静默安装；内置了很多第三方存储库，可以按需启用。不过这个客户端对中文支持比官方客户端还差，应用详情没有中文，也不支持中文搜索，英文不好的人用起来可能有些费劲。

![](https://cdn.sspai.com/2023/01/04/b093746ef3b180e025953064ede2c9f7.png)

[Neo Store](https://sspai.com/link?target=https%3A%2F%2Ff-droid.org%2Fzh_Hans%2Fpackages%2Fcom.machiav3lli.fdroid%2F) 是另一个修改自 Foxy Droid 的第三方客户端，由 [NeoApplications](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FNeoApplications) 开发，功能和 Droid-ify 基本相同，界面也大同小异，同样的中文支持也很差，不过更新频率更快，比如就在最近的更新中在应用详情界面添加了跟踪器和权限请求信息，不过也正因为更新频率快，界面更改比较频繁，优化较差，有时比较卡顿。此外 NeoApplications 还开发了 [Neo Backup](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FNeoApplications%2FNeo-Backup)、[Neo Launcher](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FNeoApplications%2FNeo-Launcher) 和 [Neo Feed](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FNeoApplications%2FNeo-Feed) 等其他应用，有些已经被 F-Droid 官方库收录，有些可以通过添加 IzzyOnDroid 存储库下载，感兴趣的话可以下载尝试。

![](https://cdn.sspai.com/2023/01/04/d69e26291879c93588e7913b9eab5e86.png)

最后，目前维护较多的 F-Droid 客户端如 Droid-ify 和 Neo Store，都已经支持 Android 12 引入的[自动更新应用特性](https://sspai.com/link?target=https%3A%2F%2Fdeveloper.android.com%2Fabout%2Fversions%2F12%2Ffeatures%3Fhl%3Dzh-cn%23automatic-app-updates)，通过这些商店进行安装的应用，在无需 root 和 ADB 调试操作的前提下，后续升级过程中也能像 Play 商店一样进行无感更新，无需经过弹窗提醒反复点击确认。

## 如何发现 F-Droid 优质应用

就我个人而言，我喜欢在客户端中分类进行浏览，说不定什么时候就翻到了一个我感兴趣的应用了，另外在 GitHub 浏览时，遇到感兴趣的应用，如果应用已经被 F-Droid 官方库收录，也一般会在 README 中提供 F-Droid 的下载链接。另外[这篇文章](https://sspai.com/post/63647)中也提到了两篇帖子，我也放在这里，供大家参考：

-   [GitHub Android FOSS](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Foffa%2Fandroid-foss)，并不全是 F-Droid 应用；
-   [小众软件 F-Droid app 推荐](https://sspai.com/link?target=https%3A%2F%2Fmeta.appinn.net%2Ft%2Ftopic%2F15095%2F1)

## 参考链接

-   [F-Droid 使用指南 少数派](https://sspai.com/post/63647)
-   [F-droid 维基百科](https://sspai.com/link?target=https%3A%2F%2Fzh.m.wikipedia.org%2Fzh-hans%2FF-Droid)
-   [清华大学开源软件镜像站](https://sspai.com/link?target=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fhelp%2Ffdroid%2F)
-   [Known Repositories 官方论坛](https://sspai.com/link?target=https%3A%2F%2Fforum.f-droid.org%2Ft%2Fknown-repositories%2F721)

\> 下载 [少数派 2.0 客户端](https://sspai.com/page/client)、关注 [少数派公众号](https://sspai.com/s/J71e)，解锁全新阅读体验 📰

\> 实用、好用的 [正版软件](https://sspai.com/mall)，少数派为你呈现 🚀

以下内容于 01 月 09 日更新

自动更新特性的相关细节补充

经热心读者 Smp Kong 提醒，补充了文中关于 Droid-ify 和 Neo Store 等客户端通过支持 Android 12 自动更新应用特性实现无感更新的说明。

© 本文著作权归作者所有，并授权少数派独家使用，未经少数派许可，不得转载使用。

[![so1ar](https://cdn.sspai.com/2022/10/24/06bbaefbc3d2833d1ae59696ad742b11.jpg?imageMogr2/auto-orient/quality/95/thumbnail/!84x84r/gravity/Center/crop/84x84/interlace/1)](https://sspai.com/u/6x1rhiai/updates)