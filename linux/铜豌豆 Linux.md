

铜豌豆 Linux 是基于 Debian 的 Linux 中文 开源操作系统。

___

[Debian](https://www.debian.org/) 是一款非常优秀的 Linux 操作系统，但默认安装缺少中文桌面用户常用的软件。《**铜豌豆 Linux**》操作系统在 Debian 基础上，收集制作这些常用软件，一次性安装完成，节省大家定制 Debian 的时间，做到“**开箱即用**”。


[操作系统安装文件 iso 下载](https://www.atzlinux.com/download-iso.htm)


两个软件源

##### 软件使用十分重要，[铜豌豆软件生态](https://www.atzlinux.com/allpackages.htm)目前已经完成了许多流行的中文软件的收录工作：

-   QQ
-   微信
-   百度网盘
-   搜狗输入法
-   网易云音乐
-   有道词典
-   WPS
-   中文字体
-   星际译王

[中文软件完整列表](https://www.atzlinux.com/allpackages.htm)

Debian 自身软件包资源也很丰富，约有 59000 个软件包，《铜豌豆 Linux》100% 兼容这些软件包，并内置 Debian 国内官方镜像源。

[搜索 Debian 软件包](https://www.debian.org/distrib/packages#search_packages) [Debian 软件包分类展示](https://packages.debian.org/stable/)

理性、易用、清晰的桌面环境

##### 安装完成后，各个软件在同一个屏幕展示的截屏如下：

![一键安装软件截屏](https://cdn.atzlinux.com/atzlinux/img/yjaz-v11-jieping.png) ![系统安装软件截屏](https://cdn.atzlinux.com/atzlinux/img/jieping-v11.png)

___

#### CPU 架构支持

目前铜豌豆操作系统支持 amd64 和 arm64 两个 CPU 架构。

-   amd64
支持目前国内市场上常用的 amd64、X86 64 位 CPU， Intel 和 AMD 的 CPU 都支持。-   arm64
ARM 架构版本，支持 arm v8 指令集 64位 CPU。

#### 外设支持特性

除操作系统常用功能外，还 支持蓝牙网络热点接入，可以把手机 4G 网络通过蓝牙让电脑共享； 支持蓝牙音频源，可以把手机声音输出设置为电脑音箱。 支持 iPhone 手机通过数据线连接到电脑，拷贝手机上的照片到电脑。 支持部分型号摄像头，可用名为"茄子"的软件打开摄像头进行拍照和录视频。

## 进入铜豌豆 Linux 的世界

### 安装准备

操作系统的安装不同于一般软件的安装，有一定的危险性。使用台式机、笔记本电脑等物理机进行安装前，请做好如下准备工作：

-   如果目标机器已经有工作、学习、生活等等相关的重要数据，_请务必在安装前要**备份数据！**_
-   笔记本电脑请连接电源线
-   建议进入 BIOS 或 EFI，加载默认选项
-   笔记本电脑有硬件级无线网卡开关、蓝牙开关的，请打开
这类开关一般在键盘上面，也有位于电脑底部的。-   笔记本电脑有 Fn + 组合键 无线网卡开关、蓝牙开关的，请在原有操作系统上打开
从 F1 键到 F12 键，其中的部分按键，笔记本电脑厂家印有各种功能的图标，具体请查看厂家电脑说明书。-   无线 WiFi 网络、有线网络建议支持自动获取网络信息
-   如果无线 WiFi 网络需要连接密码，也请准备好

### 支持安装方式

-   物理机
支持多系统共存；支持多个硬盘,可以在机器上任一硬盘上的任一分区上安装。

### 制作安装 U 盘 #makeusb

建议使用 U 盘制作 iso 文件安装盘进行安装： 在任一 Linux 系统下，可以使用如下命令制作安装 U 盘：（假设 U 盘为 /dev/sdb，U 盘的设备名可以在 U 盘插入后，用 root 用户在命令行执行 dmesg 命令，看最后输出的几行中，就有 sd 字符串开头的 U 盘设备名）  
在命令行下，先 cd 进入下载的 iso 文件所在目录，再用 root 账号执行：

```
dd if=atzlinux-11.5.1-amd64-DVD-1.iso of=/dev/sdb bs=8M status=progress

```

注：请把 if 后面的 iso 文件名，替换为下载的 iso 文件名。

其它操作系统可以用各类光盘镜像制作软件制作。 如：UltraISO 软碟通（[下载](http://cn.ultraiso.net/xiazai.html) [制作 u 盘启动盘教程](https://jingyan.baidu.com/article/19020a0a7dabaa529d28428b.html)）、 [Win32DiskImager](https://www.onlinedown.net/soft/110173.htm) 、 [ventoy](https://www.ventoy.net/cn/index.html)(1.0.21 以上版本）

_注意：_数据从电脑内存完全同步到 U 盘，需要一定时间。为保证 iso 文件完整写入 U 盘，**请不要着急拔出 U 盘**。 在看到 U 盘子制作完成的提示后，请使用操作系统的“卸载卷”、弹出 U 盘功能，保证 U 盘完全同步数据。 部分有读写指示灯的 U 盘，请确认指示灯熄灭。

_注：_ 使用虚拟机安装时，可以不用制作启动 U 盘，直接把下载的 iso 文件挂载到光驱，再启动安装。

### 设置 BIOS EFI 从 U 盘启动

关闭机器，插入 U 盘，重启机器，按键进入 BIOS 设置，在 BIOS 或 EFI 里面设置为 U 盘启动。该镜像同时支持老的 BIOS 主板和新的 EFI 主板。

注：

不同机器开机进入 BIOS 设置的按键不一样-   一般在开机屏幕时有提示。有按 Delete 键进入，也有按 F2 键……
-   如果开机屏幕没有提示，建议查下主板说明书，或者上网查询、咨询机器厂家。

安装双系统的话，请主意要使用原有系统一样的启动模式来启动 U 盘安装。

-   原有系统是使用 BIOS 启动
请继续在 BIOS 里面设置成 U 盘启动安装，安装过程中，不要使用 EFI 分区。-   原有系统是使用 EFI 启动
请继续在 EFI 里面，设置成 U 盘启动安装，安装过程中，使用自动分区或者手工设置一个 EFI 分区。

_注：同一个机器上的不同 USB 口，有不兼容的情况_。当在安装过程中，有出现**无法读取文件**的情况时，请尝试重新拔插使用不同 USB 口重新启动安装，台式机请优先选择机器背面的 USB 口。

-   虚拟机
支持 vmware、VirtualBox、PVE 等多种虚拟机。内存设置为 2 G 即可正常运行。 在安装时，可以将 iso 文件，直接挂载为光驱启动安装。

### 磁盘空间

建议 25G 以上。

-   新电脑
建议使用整个硬盘空间。-   双系统

单硬盘：

建议先在原有系统上，把准备用来安装新系统的磁盘分区删除，使之成为**空闲磁盘空间**，然后再重启机器进行安装。

多个硬盘：

建议单独拿出一个硬盘的全部空间。

-   虚拟机
磁盘镜像文件建议 25 G 以上。

### 安装过程

本 iso 文件的安装过程，已经进行了大量自动化定制，节省安装过程的手工输入。 安装过程中，需要人工输入的信息如下：

-   如果是用 WiFi 连接网络，需要输入选择使用哪个 WiFi，并输入WiFi 密码。
如果是网线接入，这个步骤省略。-   选择安装版本
根据不同用途，ISO 文件集成了 5 个软件包组合子版本， 请按需选择其中的一个子版本进行安装： ![](https://cdn.atzlinux.com/pics/d-i/simple-cdd_profiles_0.png)  
非技术人员，请选择“大众用户版”。 [5 个子版本区别的详细说明](https://www.atzlinux.com/faq.htm#ver)

注：以上不同的软件包组合版本，只是安装的软件包搭配不同，不能够同时选择安装的。 如果想尽量多安装软件包，就请选择“三合一桌面环境 普通用户版”。  
**请只选择其中一个**安装，如果同时勾选多个，会导致**安装报错！**

4.  对磁盘进行分区
根分区**至少需要 10G** 空间才可以完成安装。 swap 分区一般按 1 倍内存，但不超过 2 G 为宜。 建议 home 分区使用单独分区。

-   自动分区
安装程序可以自动识别磁盘的空闲空间，在操作磁盘分区的步骤中，选择空闲分区进行安装即可。推荐新手使用。-   手工分区
手工分区，有搞错分区**丢失数据**的风险，请务必先在网络、移动硬盘或者其它机器备份数据！ 请务必小心操作！在重新调整分区前，需要确认原有分区和 Linux 分区设备名的对应关系，不建议新手手工分区来安装多系统。-   EFI 分区
如果电脑是使用 EFI 启动，在安装时，需要有一个单独的 EFI 分区。EFI 分区只需要很小的磁盘空间，35 MB 就够了。

7.  安装耗时
在分区操作完成后，接下来的安装过程，不再需要进行其它人为操作。 三合一版本安装软件包数量最多，耗时最长。 三合一版本，SSD 硬盘安装，预计 40 多分钟；机械硬盘，最长可达 2 小时。 期间系统需要联网，自动下载少量更新文件，请确保网络畅通。

### 用户登录 #login

安装完成后，机器会自动重启进入用户登录界面。

-   用户登录时，三合一版本的用户，请**先输入用户名**后，再在用户登录界面 _**右上角**_ 选择桌面环境，有 xfce、MATE、gnome 三类桌面环境供选择。
然后再输入密码登录。  
注：用户如果需要在登录进入系统后，使用另外的桌面环境，请先退出登录，重新在登录界面右上角选择。

___

### **默认用户名** wo root

系统默认创建两个用户， 一个是名字为 wo 的普通用户； 一个是具有最高系统权限的管理员用户 root 。

-   默认普通用户名为： **wo** ，**默认密码**为：
_debian168;_  
最后一个字符是英文的分号-   root 用户可以直接登录系统， _**默认密码**_为：
_debian-cn;168_  
中间为英文的连接符，英文的分号

___

-   请登录机器后，在终端命令行用 passwd 命令及时**修改密码， root 用户和默认的普通用户 wo，都要修改密码！！！**
    图形界面修改密码,不同桌面环境，修改密码的位置分别如下：-   xfce:点击桌面“口令”图标修改
    -   MATE:顶部菜单栏“系统”--》“控制中心”--》“个人”--》“关于我”--》“更改密码”
    -   gnome:点击顶部菜单右上角托盘图标区域，在弹出的菜单中点击设置图标（螺丝刀扳手）打开设置程序，再依次点击“用户”--》“密码”
-   如果不需要使用默认的普通用户 wo，请用 root 创建新的用户后，再删除 wo 用户即可。
具体方法，请参见： [如何创建自己的账号，删除默认的 wo 账号？](https://www.atzlinux.com/faq.htm#user)-   本系统没有安装任何网络远程登录服务端，**默认不开启网络远程登录**功能。
如果需要允许远程登录，请切记在修改密码后，才安装远程登录相关服务。同时建议关闭远程登录的密码功能，使用带密码的私钥。

### 网络

-   在安装过程中，会自动使用 DHCP 动态 IP 方式连接网络
-   默认主机名为： atzlinux
-   在安装完成后，可以用 _nmtui_ 命令修改网络连接方式、主机名等网络相关信息。
在“编辑连接”完成后，再进入“启用连接”菜单，对网络连接进行“停用”，再“激活”操作后，生效。-   有 GUI 界面的，也可以点击网络图标调整
断开，重新连接生效-   如果修改主机名，建议再手工修改下 /etc/hosts 文件
在 127.0.0.1 条目最后面，加上新修改的主机名。-   支持 ipv6

对网络信息的修改完成后，也可以重启机器，让新的网络配置信息生效。

### 已知问题：

-   安装阶段

1.  部分笔记本电脑触控板在图形安装时，无法使用；在操作系统安装完成后能正常工作。请使用非图形安装，或者用键盘操作图形安装。
2.  部分网卡在安装过程中无法使用，在操作系统安装完成后能正常工作。

-   设备驱动
少量 USB 无线网卡、高端显卡、HDMI 显卡集成声卡，默认没有驱动，需要在操作系统安装完成后再手工安装。

在安装过程中，如出现无法识别网卡，或者无法连接网络的情况，可能需要在安装后手工 [添加 Debian 官方软件源 和 铜豌豆软件源](https://www.atzlinux.com/faq.htm#addapt)。

#### NVIDIA 显卡驱动

铜豌豆默认安装NVIDIA 显卡开源驱动 nouveau，软件包名为：xserver-xorg-video-nouveau。

如需要安装 NVIDIA 闭源驱动，请参阅： [安装 NVIDIA 显卡厂家闭源驱动及相关软件](https://www.atzlinux.com/skills-tech.htm#nvidia)

### 安装后优化

-   安装 wine qq #wineqq

因腾讯官方发布的 linuxqq 版本,目前功能还比较少，本系统还支持安装 wine qq。  
这两个版本的 QQ 都可以同时在系统上使用。 请在命令行执行如下命令安装：

_apt -y install com.qq.im.deepin_

需要下载 120 多 M 文件，请耐心等待。安装完成后，请到应用程序菜单启动 QQ 即可使用，首次启动较慢。-   安装 wine 微信 #winewx
因部分微信账号被限制使用 [网页版微信](https://web.wechat.com/)， 本系统还支持安装 wine 微信。  
这两个版本的微信都可以同时在系统上使用。 请在命令行执行如下命令安装：

```
apt -y install com.qq.weixin.deepin

```

-   安装其它软件
请到铜豌豆软件源[软件包列表](https://www.atzlinux.com/allpackages-x86_64.htm)，按需安装其它软件。

注：请先确认操作系统上，Debian 官方软件源 和 铜豌豆软件源 都已经安装好后，再进行软件包安装操作。 如果在执行软件包安装命令时报错，请先手工添加如下软件源：

-   [添加 Debian 官方软件源](https://www.atzlinux.com/faq.htm#addapt)
-   [添加 铜豌豆软件源](https://www.atzlinux.com/faq.htm#apt-hand-iso)

## 更新升级

-   Debian 升级和安全更新，默认使用 Debian 国内官方镜像。
-   本操作系统软件源仓库也设置在国内服务器上。

一键更新所有软件包到最新版本：

_apt update  
apt upgrade_

## 一键安装脚本

对于已经安装好 Debian 系统的用户，可以使用一键安装脚本一次性安装中文软件， [详情请点击这里访问](https://www.atzlinux.com/yjaz-v11.htm)。

## 开发

遵循 GPL3.0 协议,项目源代码地址：

[https://gitee.com/atzlinux/debian-cn/tree/apt-install/](https://gitee.com/atzlinux/debian-cn/tree/apt-install/)

欢迎各位参与!

___

## 捐赠 《铜豌豆 Linux》项目

《铜豌豆 Linux》项目是开源项目，为维持项目持续发展运行，请积极 [捐赠](https://www.atzlinux.com/juanzeng.htm)。

![微信捐赠收款码](https://cdn.atzlinux.com/atzlinux/img/wechat-pay.jpg) ![支付宝捐赠收款码](https://cdn.atzlinux.com/atzlinux/img/ali-pay.jpg)

### [**捐赠记录列表**](https://www.atzlinux.com/juanzeng.htm#liebiao)

建议通过 iso 安装操作系统的用户，可以捐赠 10 元以上。

___

## 安全性声明

Linux/Debian 的安全性在业界享有盛誉，本系统默认继承

[Debian 的所有安全机制](https://www.debian.org/security/)。

-   Debian 软件包
已经设置好国内的 Debian 安全软件更新源，使用 _apt update;apt upgrade_ 命令即可安装升级修复 Debian 软件包的所有安全补丁。-   商业软件
对于使用的商业软件，都从商业公司官网下载，本系统上原则上不做修改；如商业软件的原有软件包有无法适配 Debian 的问题，也只做最小的适配修改，所有改动都开放源代码。-   开源软件
对于采用的开源项目软件包，都从开源项目主页下载；有 deb 包，能够用的直接使用，需要适配 Debian 进行格式转换和修改的软件包，改动过程也全部开放代码。-   自制软件
本系统自己制作的软件包，全部开放源代码。

[本项目所有源代码地址](https://gitee.com/atzlinux/projects)

大家有发现本系统的任何安全问题，麻烦及时向[我们反馈处理](mailto:xsw@atzlinux.com)，谢谢！ 欢迎使用加密邮件，gpg id: 0xC7909957

## 免责声明

本操作系统基于开源软件和各个厂家的商业软件集成，对使用此操作系统造成的任何问题和损失，均不承担任何赔偿责任。  
相关商业软件的知识产权归各商业公司拥有，相关知识产权风险请自行承担。  
本项目基于 Debian ，但不属于 Debian 官方。本项目的任何问题，均与 Debian 官方项目无关。

Debian 是 Software in the Public Interest, Inc. 的注册商标。  
[Debian is a registered trademark owned by Software in the Public Interest, Inc.](https://www.debian.org/trademark)  
本操作系统与 Debian 官方组织没有任何关系。

其它资源： [OS2ATC 开源操作系统年度技术会议](https://bss.csdn.net/m/topic/os2atc/index#a) 2019-12-14 [资料下载](https://www.atzlinux.com/atzlinux/doc/os2atc2019/)

[Made with Bluefish HTML editor.](http://bluefish.openoffice.nl/) 本文最后修改时间：2021-09-26 [粤ICP备19157078号-1](http://beian.miit.gov.cn/) [![公安备案图标](https://cdn.atzlinux.com/atzlinux/img/ghs.png) 公安备案号 44030502004897](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=44030502004897)

[https://gitee.com/atzlinux  
感谢码云 ![Gitee — 基于 Git 的代码托管和研发协作平台](https://gitee.com/static/images/logo-black.svg?t=158106664) 为本项目提供源代码托管平台！](https://gitee.com/atzlinux)

___

[首页](https://www.atzlinux.com/index.htm) [一键安装](https://www.atzlinux.com/yjaz-v11.htm) [软件](https://www.atzlinux.com/allpackages-x86_64.htm) [下载](https://www.atzlinux.com/download-iso.htm) [新闻](https://www.atzlinux.com/News/index.htm) [使用技巧](https://www.atzlinux.com/skills.htm) [常见问题](https://www.atzlinux.com/faq.htm) [参与开发](https://www.atzlinux.com/devel.htm) [开源贡献](https://www.atzlinux.com/contribute.htm) [友情链接](https://www.atzlinux.com/links.htm) [捐赠](https://www.atzlinux.com/juanzeng.htm) [反馈](https://gitee.com/atzlinux/debian-cn/issues) [联系方式](https://www.atzlinux.com/contact.htm)

《铜豌豆 Linux》官网二维码  
![《铜豌豆 Linux》官网二维码](https://cdn.atzlinux.com/pics/qr-atzlinux-com.svg)  
版权所有 © 《铜豌豆 Linux》 项目网站版权协议为(CC BY-NC-ND 4.0)