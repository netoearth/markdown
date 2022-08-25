**一、准备工具**  
`链接: https://pan.baidu.com/s/16gqF-jDWHopRZrzhu8Sl6Q 提取码: rj24`  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202092706585-1669315394.png)

网盘内容下载到本地，并且解压文件  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202092946684-1202880790.png)

安装PVE的教程，可以参考  
[https://www.cnblogs.com/faberbeta/p/proxmox004.html](https://www.cnblogs.com/faberbeta/p/proxmox004.html)  
**二、将二个镜像传入pve的镜像仓库**

```
Catalina-installer.iso
OpenCore.iso
```

![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202093147940-207611099.png)  
**三、创建虚拟机**  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202093817049-481221621.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202093932035-43048628.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202094044350-790031162.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202094233717-1143234456.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202094543323-1840630611.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202094756308-2026579881.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202094859300-456065704.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202095022073-282224426.png)

使用opencore引导， 创建两个CD-ROM  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202095110009-1208870947.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202095205596-1637555461.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202110303077-352670607.png)

最终结果  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202110619357-946746034.png)

**四、配置虚拟机**  
由于mac驱动不支持CD驱动器, 所以这边要把CD驱动设备改成硬盘设备、另外黑苹果引导需要对cpu参数做一些处理，图形话界面无法修改，需要进入shell修改，这两个修改是同一个配置文件 我们创建的虚拟起id是108 那么这个虚拟机的配置文件路径为 /etc/pve/qemu-server/108.conf  
通过vim工具对/etc/pve/qemu-server/110.conf进行修改  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202095728810-560218597.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202100157936-543622548.png)

```
apt update
apt install vim -y
```

![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202100012622-1913256282.png)

修改/etc/pve/qemu-server/下的虚拟机配置文件，与创建时的VM ID同名，比如文中的是110.conf，在文本开头添加一行：  
不会使用vim请自行百度  
vim /etc/pve/qemu-server/110.conf  
`args: -device isa-applesmc,osk="ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc" -smbios type=2 -device usb-kbd,bus=ehci.0,port=2`  
如果cpu是intel，则在同一行行末加上：

`-cpu host,kvm=on,vendor=GenuineIntel,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc`  
如果cpu是amd，则在同一行行末加上：

`-cpu Penryn,kvm=on,vendor=GenuineIntel,+kvm_pv_unhalt,+kvm_pv_eoi,+hypervisor,+invtsc,+pcid,+ssse3,+sse4.2,+popcnt,+avx,+avx2,+aes,+fma,+fma4,+bmi1,+bmi2,+xsave,+xsaveopt,check`  
将两处media=cdrom修改为cache=unsafe  
**最终的结果如图：**  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202110946270-784117639.png)

intel cpu 配置为host 就是直接使用宿主cpu 在虚拟机中可运行虚拟机、docker 实现嵌套虚拟化 amd cpu 必须模拟成penryn架构、无法嵌套虚拟化，intel cpu也可以模拟penny 当然性能会较host有所损耗 media-cdrom 修改为 cache-unsafe 使得cd-rom设备变为硬盘设备  
**五、配置宿主机**

```
echo "options kvm ignore_msrs=Y" >> /etc/modprobe.d/kvm.conf
update-initramfs -k all -u
reboot
```

在PVE的shell命令行处，输入以上命令并重启，是避免引导循环启动  
**六、安装**  
1）修改引导顺序，OpenCore.iso的设备设置为自动第一引导项  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202101219664-1033347555.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202101444285-881070130.png)  
2）开机  
进入ovmf画面 按esc  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202111151403-2119308277.png)  
修改分辨率  
Device manager ->ovum platform configuration->change preferred ->1920\*1080 修改分辨率  
修改分辨率是为乐防止引导界面出现显示问题、如果你那边正常可不修改  
接下来就是正常引导安装  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202111350676-1014540062.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202111549893-1002648842.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202111824688-864221588.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202111906144-1927629749.png)  
![](https://img2020.cnblogs.com/blog/1616576/202012/1616576-20201202111932941-1385224581.png)  
接下来就不继续演示了，可以自行操作了