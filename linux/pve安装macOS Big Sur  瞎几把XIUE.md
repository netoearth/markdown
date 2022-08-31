## [](https://blog.d2x3.cn/pve%E5%AE%89%E8%A3%85macOS-Big-Sur.html#%E5%88%B6%E4%BD%9Ciso "制作iso")制作iso

[fetch-macOS.py](https://raw.githubusercontent.com/kholia/OSX-KVM/master/fetch-macOS.py)

生成一个`BigSurInstaller.dmg`

```
python3 fetch-macOS.pyhdiutil create -o BigSurInstaller.dmg -size 14g -layout GPTSPUD -fs HFS+Jsudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/install_build --nointeractionhdiutil detach -force "/Volumes/Install macOS Big Sur"mv BigSurInstaller.dmg BigSurInstaller.iso
```

下载一个`OpenCore-v10.iso`

[KVM-Opencore](https://github.com/thenickdude/KVM-Opencore)

将`OpenCore-v10.iso`和`BigSurInstaller.iso`上传到pve

新建虚拟机,配置如下

编辑 `vim.tiny /etc/pve/qemu-server/101.conf`

```
args: -device isa-applesmc,osk="ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc" -smbios type=2 -device usb-kbd,bus=ehci.0,port=2 -cpu host,kvm=on,vendor=GenuineIntel,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc
```

终端输入

```
echo "options kvm ignore_msrs=Y" >> /etc/modprobe.d/kvm.conf && update-initramfs -k all -uecho 1 > /sys/module/kvm/parameters/ignore_msrs
```

将opencore放进efi

## [](https://blog.d2x3.cn/pve%E5%AE%89%E8%A3%85macOS-Big-Sur.html#disk2s1-%E6%98%AFopencorev10-iso-disk0s1%E6%98%AF%E5%AE%89%E8%A3%85BigSur%E7%9A%84%E7%A1%AC%E7%9B%98EFI%E5%88%86%E5%8C%BA "disk2s1 是opencorev10.iso disk0s1是安装BigSur的硬盘EFI分区")disk2s1 是opencorev10.iso disk0s1是安装BigSur的硬盘EFI分区

sudo dd if=/dev/disk2s1 of=/dev/disk0s1

## [](https://blog.d2x3.cn/pve%E5%AE%89%E8%A3%85macOS-Big-Sur.html#%E7%9B%B4%E9%80%9A%E8%AE%BE%E7%BD%AE "直通设置")直通设置

硬盘直通

qm set 592 -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166\_Z1F41BLC

显卡直通

加载vfio模块

编辑 `/etc/modules`

```
vfiovfio_iommu_type1vfio_pcivfio_virqfd
```

屏蔽gpu驱动

```
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```

开启IOMMU

编辑 `/etc/kernel/cmdline` (编辑`/etc/default/grub`里的`GRUB_CMDLINE_LINUX_DEFAULT`没有启作用)

`lspci -nn`找到显卡的vendor\_id, device\_id

```
intremap=no_x2apic_optout intel_iommu=on vfio-pci.ids=<vendor_id>:<device_id>,<vendor_id>:<device_id> disable_vga=1
```

例如

```
intremap=no_x2apic_optout intel_iommu=on vfio-pci.ids=10de:1287,10de:0e0f disable_vga=1
```

更新、重启

```
update-initramfs -u -k allpve-efiboot-tool refreshreboot
```

验证

```
cat /proc/cmdlinedmesg |grep -E "DMAR|IOMMU"dmesg |grep -i vfiolspci -nnkfind /sys/kernel/iommu_groups/ -type l
```

设置

显卡插上显示器,此时如果显示器没有信号,插拔一下显示器

OSK值

`ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc`

-   [OSX-KVM](https://github.com/kholia/OSX-KVM)