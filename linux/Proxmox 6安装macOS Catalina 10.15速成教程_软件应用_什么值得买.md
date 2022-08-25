2020-03-12 14:48:55 10点赞 107收藏 37评论

**创作立场声明：**本文只讨论Proxmox 6安装macOS Catalina 10.15，其他的功能（硬件直通，改三码等）不在本文的讨论范围内。本文重在速成，所以每个步骤几乎没有说明，如果需要步骤的说明可以看文末的原教程链接。

**追加修改(2020-04-14 10:16:27):**  
因为虚拟机没有显卡的原因，启动时可能会花屏（就是开机时那个苹果logo不在中间），这时只需要把虚拟机重置即可（苹果logo在屏幕中间就是启动成功，大概每5次有两次启动成功）。

在[我上一篇文章](https://post.smzdm.com/p/ar08e8l7/)中，我提到了unRAID安装黑[苹果](https://pinpai.smzdm.com/1687/)

比Proxmox要简单。也正好最近闲得发慌，就在网上搜了一下Proxmox安装黑苹果的文章，结果要么就是旧版本Proxmox安装黑苹果的文章，要么就没有Proxmox安装macOS 10.15的文章。我这个人有一点小小的强迫症，最近也闲得发慌，于是就有了今天这篇文章。

本文所需文件：[OneDrive](https://jxjjxy-my.sharepoint.com/:f:/g/personal/u965j9ktu_t_odmail_cn/EpzFuEm_CNRNmdD0kdAvxkYBrf7aQkC0_1B2o8x6bIqB9g?e=qZjijm)（访问速度可能会非常慢，请耐心等待） [百度网盘](https://pan.baidu.com/s/1irs8PsEdWAACZdYwxl1-dQ) [](https://www.misonsky.cn/343.html)[WinSCP](https://winscp.net/eng/docs/lang:chs)或[MobaXterm](https://mobaxterm.mobatek.net/)

[顺](https://pinpai.smzdm.com/83694/)

便吐槽一下百度网盘，下载慢就算了上传还这么慢![Proxmox 6安装macOS Catalina 10.15速成教程](https://res.smzdm.com/images/emotions/23.png) 。

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68af62355672407.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_2/)

## 前期准备

下载完成OneDrive或百度网盘的文件后，把所有压缩包解压（那个 .zip 的压缩包是原教程的离线网页）。

上传这两个ISO到Proxmox上。

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e69a342f16264333.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_3/)

没错，前期准备就这么简单![Proxmox 6安装macOS Catalina 10.15速成教程](https://res.smzdm.com/images/emotions/111.png) 。

## 创建虚拟机

回到Proxmox控制台，创建一个macOS虚拟机。

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bca7526e36981.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_4/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bcb4c80033316.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_5/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bcc3734b04234.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_6/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bcd4c98ff4343.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_7/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/11/5e68bce2cd7ac6246.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_8/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bcf25cd739547.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_9/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bd05d4b354136.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_10/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bd6057f085519.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_11/)

创建完成后，**不要启动！不要启动！不要启动！**再给macOS虚拟机添加一个CD驱动器。

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/11/5e68bc7a029c59197.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_12/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68be623ba8c7041.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_13/)

然后点击“选项”，确保“使用平板指针”为“是”。  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bf745d5834898.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_14/)

如果是“否”，请按下图步骤操作：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68bfefda36b4.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_15/)

然后打开[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)，CMD，PowerSell或MobaXterm来连接Proxmox。账号为root，密码为你安装Proxmox时设置的密码。我这里使用PowerSell来连接：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68c437be48f6944.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_16/)

输入下面这条命令后回车：

> nano /etc/pve/qemu-server/你的macOS虚拟机的VM ID.conf

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68c709cb74b2764.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_17/)

把这条添加到文件里：  

> args: -device isa-applesmc,osk="ourhardworkbythesewordsguardedpleasedontsteal(c)[Apple](https://pinpai.smzdm.com/1687/)
> 
> ComputerInc" -smbios type=2 -cpu Penryn,kvm=on,vendor=GenuineIntel,+kvm\_pv\_unhalt,+kvm\_pv\_eoi,+invtsc,vmware-cpuid-freq=on,+pcid,+ssse3,+sse4.2,+popcnt,+avx,+aes,+xsave,+xsaveopt,check -device usb-kbd,bus=ehci.0,port=2

再把箭头所指的两个地方都替换成：

> cache=unsafe

[![替换前](https://qnam.smzdm.com/202003/11/5e68c7817e9cb78.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_18/)替换前

[![替换后](https://qnam.smzdm.com/202003/11/5e68c7f4c4e738668.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_19/)替换后

点击[键盘](https://www.smzdm.com/fenlei/jianpan/)上的 Ctrl+X ，输入 Y ，然后按 回车 即可保存并退出：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68c8864209e3261.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_20/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/11/5e68c8cbb5d936408.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_21/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68c8e27e9c65476.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_22/)

回到Proxmox控制台，点击“选项”，把 ide2 设置为第一启动项（即装载Clover引导镜像的CD驱动器）：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d360539fc1177.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_23/)

## 配置Proxmox

输入以下命令并回车：

> echo "options kvm ignore\_msrs=Y" >> /etc/modprobe.d/kvm.conf && update-initramfs -k all -u

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68ced0dbf504071.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_24/)

然后把 pve-edk2-firmware\_2.20191127-1\_all.deb 这个文件上传到 /root 目录，再 cd 到 /root 目录，输入 ls ，即可看到刚才上传的文件：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d00bcdb919738.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_25/)

输入这条命令后回车：

> dpkg -i pve-edk2-firmware\_2.20191127-1\_all.deb

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d088183314617.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_26/)

最后输入这条命令并回车：

> apt-mark hold pve-edk2-firmware  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d284c2fe27576.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_27/)

## 安装macOS Catalina

（注：安装过程中需**全程联网**！）  

开启虚拟机，并快速回到 控制台 选项卡，点击 Esc 进入OVMF设置。如果没有进入到OVMF设置，请重启虚拟机并再试一次：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d4b8940898403.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_28/)

选择 Device Manager ，回车：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d53aceb703988.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_29/)

选择 OVMF Platform Configuration ，回车：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d56dcf5fe6205.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_30/)

在 Change Preferred 里选择 1920x1080 ，回车：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d5f1606858424.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_31/)

按键盘上的 F10 ，再按 Y 保存设置：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/11/5e68d65db06e4506.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_32/)

按两次 Esc ，选择 Rest ，回车：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d6b682c8f9911.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_33/)

然后就会进入Clover，回车以引导macOS Catalina的恢复镜像：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d749414a9486.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_34/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d751845683732.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_35/)

选择 简体中文 ，然后点 下一步 ：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d7bf99e8b418.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_36/)

选择 磁盘工具 ，然后点 继续 ：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d80d4574d385.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_37/)

这里要注意，磁盘要选容量最大的那一个：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d90783a4f3984.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_38/)

点击 完成 ，然后关闭 磁盘工具 ：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68d997862471269.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_39/)

点击 重新安装macOS ，这里忘了截图。再点击 继续 ：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68da2ec374e3431.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_40/)

点击 同意 ，再点击 同意 ：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/11/5e68daf3576746424.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_41/)

选择磁盘，然后点击 安装 ：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/11/5e68db4342f214163.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_42/)

此时，你可以选择打几把王者荣耀，打几把吃鸡，或者去睡一觉（这里我等了两个小时![Proxmox 6安装macOS Catalina 10.15速成教程](https://res.smzdm.com/images/emotions/97.png) ）。  

[![Apple定义的“20分钟”](https://qnam.smzdm.com/202003/11/5e68dbae12cf88972.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_43/)Apple定义的“20分钟”

如果你看到这样的画面，就把虚拟机重置，重置后在Clover里选 Boot macOS from macOS ，回车。或者在第二次重启时选 Boot macOS from macOS ，再回车：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e698b1b382e68365.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_44/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e698b2b2ce4f6388.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_45/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e698bf9028974625.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_46/)

启动完成后，选 中国大陆 ，然后一路 继续 ：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e698be0585a59101.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_47/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e698e2f044ba950.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_48/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e698e3d0b26a8525.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_49/)

注意：这里**千万不要登录Apple ID！千万不要登录Apple ID！千万不要登录Apple ID！**否则你的Apple ID可能会被Apple封禁：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e698e4d0bdf73264.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_50/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e698ee118ae26404.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_51/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e699031da7549426.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_52/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e6990d1640ce3373.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_53/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e6990dd3e13e3001.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_54/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e6990e7bbc9f4927.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_55/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e6991389fdfb1387.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_56/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e6992095a7f45532.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_57/)

进入桌面后，会提示设置键盘，按提示来即可：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e699234050dc7035.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_58/)

[![不同的电脑，键盘布局可能不一样，但绝大对数是ANSI](https://qnam.smzdm.com/202003/12/5e69923ecfbd28922.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_59/)不同的电脑，键盘布局可能不一样，但绝大对数是ANSI

## 安装Clover到磁盘  

首先，打开终端。由于我们并没有安装[显卡](https://www.smzdm.com/fenlei/xianka/)和显卡驱动，所以画面会非常卡，这是正常现象：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e69964c384184248.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_60/)

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://am.zdmimg.com/202003/12/5e6996569c5605980.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_61/)

在终端输入以下命令以查看可用的磁盘：  

> diskutil list

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e6998829e7cd8004.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_62/)

接下来要注意，把Clover引导盘的EFI分区写入到macOS安装盘的EFI分区，**如果写入错误可能需要重新安装系统！**

输入以下命令以拷贝分区：

> sudo dd if=<Clover引导盘的EFI分区> of=<macOS安装盘的EFI分区>

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e699b3882f823432.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_63/)

如果终端上显示的和图片上的一样，你可以直接使用下面这条命令：

> sudo dd if=/dev/disk1s1 of=/dev/disk2s1

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e699d1dc67774085.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_64/)

然后关闭虚拟机，点击 选项 ，把 sata0 设置为第一启动项：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e69a04f4d3c7534.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_65/)

然后启动虚拟机，如果没有拷贝错磁盘分区的话将会进入Clover引导界面。选择 Boot macOS from Main 以启动：  

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e69a10fb52032245.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_66/)

可能时因为没有显卡的原因，我在这里翻车了![Proxmox 6安装macOS Catalina 10.15速成教程](https://res.smzdm.com/images/emotions/136.png) ：

[![Proxmox 6安装macOS Catalina 10.15速成教程](https://qnam.smzdm.com/202003/12/5e69a8740c5f9655.png_e1080.jpg)](https://post.smzdm.com/p/a4wm2mdk/pic_67/)

所以就没有下文了![Proxmox 6安装macOS Catalina 10.15速成教程](https://res.smzdm.com/images/emotions/115.png) 。

最后附上[原教程链接](https://www.nicksherlock.com/2019/10/installing-macos-catalina-10-15-on-proxmox-6/)，里面有每个步骤的说明和我没有讲到的东西。最后，祝你好运。

![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png)

-   相关商品推荐

![](https://y.zdmimg.com/202206/24/62b58a0f0a1cb4610.jpg_a200.jpg)

阳光心健3D心理电子沙盘系统电子心理沙盘管理软件3D沙具茶几外形设备

阳光心健3D心理电子沙盘系统电子心理沙盘管理软件3D沙具茶几外形设备

_75600元起_

![](https://y.zdmimg.com/202206/01/6296e7eeedeed3045.png_a200.jpg)

会员4年卡+28天

会员4年卡+28天

_288元起_

![](https://qny.smzdm.com/202206/11/62a3f0426c1926369.png_a200.jpg)

WPS 金山软件 WPS超级会员 2年卡

WPS 金山软件 WPS超级会员 2年卡

_279元起_

![](https://qny.smzdm.com/202206/10/62a2df13f0d803957.png_a200.jpg)

WPS 金山软件 超级会员5年卡+12个月

WPS 金山软件 超级会员5年卡+12个月

_649元起_

#### 相关文章推荐