文章声明：此文基于木子实操撰写，本教程仅做为技术分享，请支持正版  
生产环境：Proxmox 6.2-11, macOS Big Sur  
问题关键字：macOS,macOS Big Sur,macOS Big Sur 虚拟机安装

## **前提要求**

macOS对于苹果APP开发的同学来说是非常重要的，对互联网企业的持续集成与持续发布来说更是不可或缺，此次仅分享如何在Proxmox上安装macOS Big Sur，至于显卡直通、USB直通，不在本次的分享范围。本教程使用OpenCore安装macOS Big Sur至Proxmox 6.2虚拟化平台。首先确保您已经安装了Proxmox 6.x，另外您还需要一台真正的Mac电脑才能获取OSK密钥。另外您的Proxmox主机的CPU必须支持SSE 4.2，因此对于Intel，您的CPU必须至少需要Intel Nehalem微架构及以上版本，Intel Nehalem是第一代带有Core i5/i7品牌标识的CPU。较旧的CPU会导致`finder`在安装完成后反复崩溃（图形代码中出现非法指令异常）。现在AMD CPU也支持SSE 4.2，所以同样可以采用此操作指南。

## **制作BigSur镜像**

如果您有Mac电脑，可以直接制作macOS Big Sur完整镜像，这样在安装的时候不需要从互联网下载镜像，安装速度相对于采用网络恢复模式来安装macOS Big Sur来说会快很多，但完整镜像制作只支持在Mac电脑上操作，不支持Linux操作系统，Linux操作系统只支持制作恢复镜像。

**macOS上操作**

```
xcode-select --install
git clone https://github.com/thenickdude/OSX-KVM.git
cd OSX-KVM/scripts/bigsur
make BigSur-full.img
```

**Linux上操作**

```
apt install qemu-utils make
git clone https://github.com/thenickdude/OSX-KVM.git
cd OSX-KVM/scripts/bigsur
make BigSur-recovery.img
```

镜像制作完成以后，将`BigSur-full.img`或`BigSur-recovery.img`文件上传到您的Proxmox的ISO存储目录（通常为`/var/lib/vz/template/iso`）。

## **准备OpenCore镜像**

下载最新版本的[OpenCore.iso.gz](https://link.zhihu.com/?target=https%3A//github.com/thenickdude/KVM-Opencore/releases)文件，解压后，将对应ISO文件上传至Proxmox的ISO存储目录（通常为/var/lib/vz/template/iso）。

## **获取OSK身份验证密钥**

macOS检查它是否在真正的Mac硬件上运行，并拒绝在第三方硬件上启动。您可以通过从真实Mac硬件中读取身份验证密钥（OSK 密钥）来解决此问题。将本页的第一个[C代码块](https://link.zhihu.com/?target=https%3A//web.archive.org/web/20200603015401/http%3A//www.osxbook.com/book/bonus/chapter7/tpmdrmmyth/)保存为smc\_read.c，在命令提示符下，切换到与该文件相同的目录并运行：

```
# 因为需要使用gcc，所以需要安装xcode-select
xcode-select --install
# 编译
gcc -o smc_read smc_read.c -framework IOKit
# 获取OSK，它会为您打印出64个字符的OSK，记下它。每台Mac都使用相同的OSK。
./smc_read
xxxxxx(c)AppleComputerInc
```

## **创建虚拟机**

记住对应VM ID，后面需要修改对应配置文件。

![](https://pic1.zhimg.com/v2-a9b8fa0908d53af3a910f3262a2f167c_b.jpg)

选择您上传的OpenCore ISO，并将操作系统类型设置为：Other。

![](https://pic2.zhimg.com/v2-6ea72355bc67cd9cc93a0a72054f57f5_b.jpg)

将图形设置为：VMWare Compatible，将BIOS设置为OVMF（UEFI），将Machine设置为Q35，勾选QEMU Agent，勾选添加EFI Disk，并选择存储，木子这里为SSD。

![](https://pic4.zhimg.com/v2-e0cdb4913f0a368dc425f3b0a60da3ef_b.jpg)

设置硬盘大小为：100GB或更大。设置总线为：VirtIO Block，勾选“丢弃”，以支持TRIM。

![](https://pic4.zhimg.com/v2-79aafe1753d5335bd6f968a38ab3a777_b.jpg)

设置VM的核心数，使用2的幂（例如 1、2、4、8）。将CPU设置为：Penryn，勾选：启用NUMA。

![](https://pic1.zhimg.com/v2-31369a9fe9c2a55070f9958fa46714b8_b.jpg)

设置内存：16GB

![](https://pic1.zhimg.com/v2-da3e4eacac4cc469df9668ef4d158b68_b.jpg)

网络模型设置为：VMWare vmxnet3，其它信息根据自己的实际情况进行设置。

![](https://pic1.zhimg.com/v2-baf4f2b9ddd4d2a3ff9c25fe7a69adc8_b.jpg)

点击\[完成\]。

![](https://pic3.zhimg.com/v2-5e21a453921fb2ae25aa91740add24a2_b.jpg)

添加CD/DVD驱动器

![](https://pic1.zhimg.com/v2-66d715c9a32fa9a81193a9b54707e988_b.jpg)

选择对应macOS BigSur镜像，并将对应总线设置为：IDE0。

![](https://pic2.zhimg.com/v2-33b8def11ecbef7c8eadcafcd08260a5_b.jpg)

在VM的\[选项\]页面中，将\[使用平板指针\]设置为\[是\]。

![](https://pic2.zhimg.com/v2-3424cb34d99b40bac673a75d1af23a1d_b.jpg)

## **修改虚拟机配置文件参数**

暂时不要启动VM。首先，通过SSH连接到您的Proxmox服务器，以便我们可以对配置文件进行一些编辑。编辑/etc/pve/qemu-server/VM-ID.conf。添加这一行，确保将您之前提取的OSK替换到正确的位置：

```
args: -device isa-applesmc,osk="xxxxxx(c)AppleComputerInc" -smbios type=2 -device usb-kbd,bus=ehci.0,port=2
```

这里添加了一个USB键盘，因为macOS不支持QEMU的默认PS/2键盘。确保参数都在一行上！  
我们还需要添加一个`-cpu`参数。如果您的主机CPU是Intel，请将其添加到`args`行的末尾：

```
-cpu host,kvm=on,vendor=GenuineIntel,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc
```

如果您的主机CPU是AMD，或者上述参数对您不起作用，请使用此更兼容的替代方案：

```
-cpu Penryn,kvm=on,vendor=GenuineIntel,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc,+pcid,+ssse3,+sse4.2,+popcnt,+avx,+avx2,+aes,+fma,+fma4,+bmi1,+bmi2,+xsave,+xsaveopt,check
```

这会假装您的CPU是Penryn，即使您的主机CPU是AMD，它也会让macOS运行，并在上面添加了一堆更新的必需和可选的CPU功能。您的主机CPU不支持的功能将被忽略（使用启动时将向控制台打印警告qm start 1xx），但请注意如果没有SSE4.2支持，macOS将无法运行。  
现在找到定义两个`ISO`（`ide0` 和 `ide2`）的行，并从中删除`media=cdrom`部分，在其位置添加`cache=unsafe`，这会将其视为硬盘而不是DVD驱动器。设置引导磁盘`bootdisk`从`IDE2`启动（即OpenCore 映像），保存您的更改。  
您的最终VM配置文件应类似于：

```
root@S001:~# cat /etc/pve/qemu-server/120.conf
args: -device isa-applesmc,osk="xxxxxx(c)AppleComputerInc" -smbios type=2 -device usb-kbd,bus=ehci.0,port=2 -cpu host,kvm=on,vendor=GenuineIntel,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc
balloon: 0
bios: ovmf
boot: dcn
bootdisk: ide2
cores: 4
cpu: Penryn
efidisk0: SSD:vm-120-disk-1,size=1M
ide0: local:iso/BigSur-full.img,cache=unsafe,size=14G
ide2: local:iso/OpenCore-v13.iso,cache=unsafe
machine: q35
memory: 16384
name: macOSBugSur
net0: vmxnet3=5E:89:04:45:DC:A6,bridge=vmbr1,tag=3
numa: 1
ostype: other
scsihw: virtio-scsi-pci
smbios1: uuid=c9b7e2bf-6c52-4065-a588-380fa9a9a20a
sockets: 2
vga: vmware
virtio0: SSD:vm-120-disk-0,cache=unsafe,discard=on,size=100G
vmgenid: d795b429-82b1-4cb8-840b-26f5b94c0cb2
```

在Proxmox上运行以下命令，以避免在macOS启动期间出现引导循环。

```
# 临时生效
echo 1 > /sys/module/kvm/parameters/ignore_msrs
# 永久生效
echo "options kvm ignore_msrs=Y" >> /etc/modprobe.d/kvm.conf && update-initramfs -k all -u
```

## **安装macOS Big Sur**

现在启动您的虚拟机，它应该启动到OpenCore启动选择器，选择\[Install macOS Big Sur\]，按Enter启动。

![](https://pic2.zhimg.com/v2-f9cc9e0d26eaf4e62b85306665ebaa55_b.jpg)

在这里等待的时间会比较长，不要着急。

![](https://pic1.zhimg.com/v2-0101a141f949e62e16b0acb49052a33c_b.jpg)

初始化磁盘，选择\[Disk Utility\]，点击\[Continue\]。

![](https://pic1.zhimg.com/v2-bbd2fe854eb9eed06a38f50c3d4525a0_b.jpg)

点击\[Erase\]，在弹出的擦除框中进行以下设置：  
Name：macOS Big Sur(名字随便，默认为Untitled)  
Format：APFS  
Scheme：GUID Partition Map  
再点击\[Erase\]，完成磁盘格式化，退出磁盘工具。  

![](https://pic4.zhimg.com/v2-f690de5ba3fd14e88b6e6c1485a2ed1f_b.jpg)

点击\[Install macOS Big Sur\]，开始安装macOS Big Sur。

![](https://pic3.zhimg.com/v2-6ac3464e91d5951635d6c97c204b958e_b.jpg)

点击\[Continue\]

![](https://pic1.zhimg.com/v2-8f7319022dd64a92910f0b8f0c0f462c_b.jpg)

点击\[Agree\]

![](https://pic2.zhimg.com/v2-0b7e59f7b3f35c347c1cc7726f025ff1_b.jpg)

选择您刚刚擦除的磁盘，用于安装macOS Big Sur，开始安装。

![](https://pic1.zhimg.com/v2-68c685b489c05c13f6e4946084a79528_b.jpg)

安装第一阶段相对时间比较长，第一阶段以后，虚拟机会连续重启3到4次，每次都必须选择\[macOS Installer\]选项（这里是第二个，带有硬盘图标）才能继续安装。它不会自动为您选择：

![](https://pic4.zhimg.com/v2-4d6ec89c6fa78d9f48fc19f3aa414f1b_b.jpg)

安装完成以后，选择要启动的主磁盘的名称（木子的名为：Untitled）。

![](https://pic2.zhimg.com/v2-a85e8fe8f437fba64de727c56d11aa95_b.jpg)

安装完成，启动macOS Big Sur，开始初始化配置。

## **初始化macOS Big Sur**

初始化过程没有什么太多好说的，主要是国家选择、数据迁移等，这里略过。  
选择\[China mainland\]

![](https://pic1.zhimg.com/v2-98e0ffab50f80aacdf65c3fba23a75a8_b.jpg)

中间的步骤略过，包括设置账号密码、数据迁移等，初始化完成。

![](https://pic1.zhimg.com/v2-883e33e584a35a154abd6ddc2b80a4a0_b.jpg)

## **设置OpenCore永久引导**

我们目前正在从附加的OpenCore ISO中使用OpenCore进行引导。让我们将其安装到硬盘驱动器上。打开\[终端\]并运行`diskutil list`以查看我们有哪些驱动器可用。使用`sudo dd if=<source> of=<dest>`从OpenCore CD复制`EFI`分区并覆盖硬盘上的`EFI`分区。OpenCore CD是一个小磁盘（~150MB），上面只有一个`EFI`分区，主硬盘是一个大（>30GB）名称带`Apple_APFS Container`的硬盘。  
以木子的为例：OpenCore ISO的EFI分区为：disk0s1，主硬盘的EFI分区为：disk1s1，所以木子这里运行`sudo dd if=/dev/disk0s1 of=/dev/disk1s1`（注意不要覆盖错误，不然您得重新开始安装系统！）。  

![](https://pic2.zhimg.com/v2-f3bc98ed743ba587ce76eae3a499b49d_b.jpg)

现在关闭VM，并从硬件选项卡中删除OpenCore和Big Sur安装程序驱动器。

![](https://pic4.zhimg.com/v2-c3463c3c985d6e15dcbbea07fe4ca777_b.jpg)

在\[选项\]选项卡上，编辑引导顺序以将`virtio0`磁盘作为第一引导项。

![](https://pic4.zhimg.com/v2-f7fa5489c3b199eb67b8e39bf6f70b57_b.jpg)

开机，如果一切顺利，您应该会看到OpenCore启动菜单，选择\[Untitled\]磁盘来启动macOS Big Sur。

![](https://pic4.zhimg.com/v2-409167f0d1fb088c4fb726cd94d353d3_b.jpg)

## **睡眠管理**

木子发现无法使用鼠标或键盘将macOS Big Sur从睡眠中唤醒。如果遇到同样的问题，您可以在Big Sur的\[节能\]设置中禁用系统睡眠以避免该问题，或者您可以通过运行以下命令手动从Proxmox中唤醒虚拟机：

```
qm monitor VM-ID
system_wakeup
quit
```

![](https://pic3.zhimg.com/v2-fa6e1ba0a34b85eb35fd416041c95416_b.jpg)

**五平台同步更新：**  
**博客：** [欧巴云](https://link.zhihu.com/?target=https%3A//www.oubayun.com/)  
**知乎：** [欧巴云](https://www.zhihu.com/people/ou-ba-yun)  
**51CTO：** [欧巴云](https://link.zhihu.com/?target=https%3A//blog.51cto.com/oubayun)  
**云+社区：** [欧巴云](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/user/4202074)  
**微信公众号：** 欧巴云