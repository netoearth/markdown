> 我想到用 QEMU 虚拟机来体验一下新发布的 Windows 11 尝尝鲜。

-   作者：Jrglinux

_（本文字数：4700，阅读时长大约：6 分钟）_

2021 年 06 月 24 日微软正式发布了 Windows 11 操作系统，这是微软时隔 6 年来的再次发布操作系统。根据其官方介绍，Windows 11 新增了许多特性，考虑到安装 Windows 11 到 PC 会耽误现在的正常工作（毕竟用 Windows 11 替换 PC 中的 Windows 10 后需要重新安装各种工程软件，这是件很耗时的事情），因此我想到用 QEMU 虚拟机来体验一下新发布的 Windows 11 尝尝鲜。

## 一、准备工作

我们需要准备 QEMU 环境、Windows 11 镜像文件、virtio-win 的镜像文件，这里罗列一下：

-   QEMU（本文是在 CentOS 环境下安装的 QEMU 工具）
-   Windows 11 镜像（下载地址：[win11.iso](https://link.zhihu.com/?target=https%3A//www.mutaz.net/free-programs/en/download/%3F2131))，需要空间 4.5G
-   virtio-win 镜像（下载地址：[virtio-win-0.1.190.iso](https://link.zhihu.com/?target=https%3A//fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.190-1/))，需要空间 479M

注意：下载完成后，为了简化，我这里将镜像都重新简化命名如下：

|  |  |
| --- | --- |
| virtio-win-xxx.iso | virtio-win.iso |
| xxx-windows11-xxx.iso | windows11.iso |

**1、安装 qemu 命令**

如果是 Ubuntu 系统，可以直接用`sudo apt-get install qemu`或者`sudo apt-get install qemu-system-i386`命令来安装 QEMU。

本文中是使用的 qemu-5.1.0（下载地址：[qemu](https://link.zhihu.com/?target=https%3A//www.qemu.org/)）来编译生成的 `qemu-system-x86_64`、`qemu-img` 等命令。

操作步骤：

```
# 在根目录下解压 qemu-5.1.0.tar.xz
cp qemue-5.1.10.tar.xz /
cd /
tar xvf qemu-5.1.0.tar.xz

# 进入 qemu-5.1.0 源码目录
cd /qemu-5.1.0

# 编译
mkdir build
cd build
../configure
make
```

编译完之后，`qemu-img` 以及 `qemu-system-x86_64` 两个命令所在的位置分别是：

|  |  |
| --- | --- |
| qemu-img | /qemu-5.1.0/build/qemu-img |
| qemu-system-x86\_64 | /qemu-5.1.0/build/x86\_64-softmmu/qemu-system-x86\_64 |

**2、制作 Windows 11 安装磁盘**

我们重新创建一个目录，用来存放 `windows11.iso`、`virtio-win.iso` 镜像文件以及马上就要生成的 `windows11.qcow2` 文件。

```
# 为了方便，依旧在根目录下操作（其实这是很不好的习惯）
cd /
mkdir win11
cd win11

# 将下载好的 windows 11 镜像以及 virtio-win 镜像拷贝进来
cp xxx/windows11.iso /win11/
cp xxx/virtio-win.iso /win11/

# 然后利用 qemu-img 命令制作系统安装磁盘，分配磁盘大小 120G 空间
/qemu-5.1.0/build/qemu-img create -f qcow2 ./windows11.qcow2 120G
```

此时，`/win11/`目录下应该是这样几个文件：

![](https://pic1.zhimg.com/v2-c97163f01b15fe09417bd333b952e418_b.jpg)

**3、编写启动 Windows 11 脚本**

为了方便后面启动 Windows 11 系统，将 qemu 启动命令写入一个脚本中。

编辑 start.sh 脚本：

编写脚本命令内容：

```
/qemu-5.1.0/build/x86_64-softmmu/qemu-system-x86_64 \
  -enable-kvm \
  -smp 4 \
  -m 4G \
  -machine usb=on \
  -device usb-tablet \
  -display default \
  -vga virtio \
  -device e1000,netdev=net0 \
  -netdev user,id=net0,net=192.168.20.0/24,dhcpstart=192.168.20.20 \
  -drive file=/win11/windows11.qcow2,if=virtio \
  -drive file=/win11/virtio-win.iso,index=1,media=cdrom \
  -drive file=/win11/windows11.iso,index=2,media=cdrom \
  -spice port=8891,addr=172.17.81.26,disable-ticketing
```

脚本中的第 9 行与第 10 行指定了 Windows 11 中网卡驱动类型为 `e1000`，并指定其采用 `dhcp` 方式获取 IP 地址。第 14 行则指定了 spice 协议连接地址，其中`172.17.81.26:8891`是指的我的宿主机的 IP 地址和端口号（**此处 IP 地址以及端口号需要根据个人的宿主机环境进行修改**），待执行 `start.sh` 脚本后可以通过 spice 协议连接 Windows 11 系统界面。

赋予 `start.sh` 可执行权限：

**4、安装 spice 客户端**

推荐使用`virt-vierer`工具客户端，用来通过spice协议连接即将安装的windows11系统桌面。

下载地址：[virt-viewer](https://link.zhihu.com/?target=https%3A//releases.pagure.org/virt-viewer/)，推荐下载`virt-vierer-x64-2.0.msi`版本。

**5、完成准备工作**

至此，准备工作都完成了，此时`/win11/`目录下应该有 4 个文件，如下图所示。

![](https://pic2.zhimg.com/v2-81ff7988c89b24379147ebed8def7029_b.jpg)

**1、启动 start.sh 脚本**

执行 `start.sh` 脚本，然后用 spice 协议连接`172.17.81.26:8891`端口：

![](https://pic3.zhimg.com/v2-d84e9b0808bedbc77541e3499e7e40e2_b.jpg)

**2、安装 Windows 11 系统**

连接上远程界面后，首先看到的是经典的 Windows 界面：

![](https://pic4.zhimg.com/v2-e25bddce12be0ddf5c2e26b0111ac8af_b.jpg)

然后进入语言、时区、键盘等选项，此处不用修改，一直选择默认的即可。

![](https://pic2.zhimg.com/v2-7705a83d35743028017df985a6485ea1_b.jpg)

接着需要输入密钥，但我们并没有，此处选择“I don't have a product key”即可。

![](https://pic4.zhimg.com/v2-10e5dbbeaf0cb069abeda02f1bc3dfdb_b.jpg)

然后进入了磁盘选择页面，会发现没有磁盘可选，此时单击“Load driver”选项就进行扫描，然后就可以发现磁盘（此处便是 `virtio-win.iso` 在起作用了）。此处发现**并没有 Windows 11 的选项，直接选择 “w10” 那一项即可**（我猜测是因为 `virtio-win.iso` 还并未支持 Windows 11 选项，相信马上就会更新了）。

![](https://pic3.zhimg.com/v2-494349356f1c021061198cb6f5250872_b.jpg)

然后找到安装磁盘，并选择，然后下一步。

![](https://pic4.zhimg.com/v2-240146ea45e798e7feb33da7eeb844f3_b.jpg)

然后进入安装过程，稍作等待 3-5 分钟。

![](https://pic2.zhimg.com/v2-4112cdda354a8cc8dc71e9ddf157fe19_b.jpg)

安装完成后，进入准备桌面过程，是不是很熟悉？

![](https://pic4.zhimg.com/v2-b47c24d96f222308a5a9cd603ab60a77_b.jpg)

接着就到了“just a moment”界面了，马上就可以进入桌面了，是不是很激动了，哈哈哈哈。

![](https://pic2.zhimg.com/v2-4cdc68d8712c2b9e16e4714267d4112d_b.jpg)

进入了桌面，此处其实是个动态的过程，因为截屏所以看不出效果。这个 Windows 界面重新设计过 UI了，个人觉得更好看了。

![](https://pic3.zhimg.com/v2-4213ef757baa7252942724cf80aa193a_b.jpg)

**3\. 进入桌面前的准备**

至此，Windows 11 安装即将完成，只差最后的初次设置步骤了。

首先是选择国家地区，此处也即默认即可，无需更改。

![](https://pic3.zhimg.com/v2-394b3034822cdd64da91c81d539580f6_b.jpg)

然后是检查更新，此处根据网速快慢，等待的时间不定，需要耐心等待。

![](https://pic3.zhimg.com/v2-91118fe128bdc9da44a3265b6a610d9a_b.jpg)

Windows 11 这里非要让用户填写 “Microsoft 账户”，无法跳过，很是郁闷，只能填写账户，然后下一步了。没有账户的可能得先申请一个微软账户了（估计正式版本会增加跳过选项吧）。

![](https://pic1.zhimg.com/v2-f6ee5655c37685de8abcae424b50bc00_b.jpg)

然后设置开机登录密码。

![](https://pic2.zhimg.com/v2-ce50c9255b3da9da37b94847b6b19c35_b.jpg)

最后，最后，最后，激动的时刻来了，进入桌面了。初次见面，什么感觉？

乍一看，怎么那么像 Mac 的风格和 UI 界面。

![](https://pic3.zhimg.com/v2-0866e37cff3a292674e5e84318d9f00a_b.jpg)

为了显示更舒适一点，推荐设置以下屏幕分辨率（根据个人电脑屏幕大小自行设定）。这里我选择的是`1920*1080`。

![](https://pic2.zhimg.com/v2-4c76002c42047ce6249191d05421ea59_b.jpg)

## 三、体验 Windows 11 系统

根据微软官方的介绍，Windows 11 增加了很多新的功能。这里挑几个新功能体验一下。

**1、新的 UI 外观以及菜单**

![](https://pic3.zhimg.com/v2-cae9e4c89ff6a873e031ab05e08d545a_b.jpg)

确实，这个 UI 风格和 Windows 10 还是有较大区别的，和 Windows 7 相比，特别时尚了。我感觉这个 UI 风格是为了适配平板、Surface 等便携式设备而优化的。

**2、“Snap Layout” 布局功能**

这个功能说实用也实用，说没啥用我觉得也没啥多大用（可能是我还没体会到多任务同时处理的便捷性吧）。Windows 旧版本中也有桌面并排处理等功能，但和这个布局功能比，还是逊色了点。

**在窗口的最大化按钮上，鼠标悬停，即可出现 “Snap Layout” 布局窗口**，然后可以选择一种布局，将该任务放置到某个位置中。这样做的目的是为了方便多任务同时处理。

如下图所示，选择了四个桌面的布局，将两个任务放在了上面两个布局框中。

![](https://pic1.zhimg.com/v2-0403b46dff84666cf9665281521d03ec_b.jpg)

**3、新的小工具窗口**

这是由 Microsoft Edge 和 AI 提供的全新 Widgets功能，包含日历、天气、待办事项、照片等功能。

![](https://pic2.zhimg.com/v2-d83374384bd06e15b371e951b9bd4c19_b.jpg)

**4、不同场景设置不同桌面**

这个功能我觉得还蛮实用的，可以根据使用的场景不同，设置不同的桌面（甚至包含常用软件的设置）。比如设置“home”、 “game”、“work”三种不同的桌面环境，方便场景快速切换。

![](https://pic1.zhimg.com/v2-ece53af1e35452836b467adea805a350_b.jpg)

**5、全新的应用商店**

这个应该是比较重大的新功能了。微软官方介绍，Windows 11 正式版可以安装安卓 APP，极大地方便了用户的使用。

微软商店界面：

![](https://pic2.zhimg.com/v2-eaf5e7eb71712664bd9c188df13810f5_b.jpg)

我们来安装个 tiktok 试试，看看效果如何。首先在商店中搜索“tiktok”。然后会发现，tiktok 有 PC 版、Moblie device 版本。

![](https://pic1.zhimg.com/v2-50645c88004746090905b49549bb4c00_b.jpg)

根据 tiktok 界面看，我觉得我这里安装的应该是 Mobile device 版本的 tiktok。这个界面是不是很类似手机和平板的 tiktok 界面风格呢？这应该是 Windows 11 的一个较为新颖的功能。

![](https://pic1.zhimg.com/v2-8615a71416e6c6f2cba419faeb3c0ad4_b.jpg)

## 总结

本文简略的在 QEMU 的帮助体验了 Windows 11 操作系统。由于是在虚拟机中体验的，并不能真实的体验到 Windows 11 的触摸便捷性、游戏画面优化、声音优化、以及其他的一些新功能特性。期待在将来能在真实设备上体验 Windows 11 操作系统。