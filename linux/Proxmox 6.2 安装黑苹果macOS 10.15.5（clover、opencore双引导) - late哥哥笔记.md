一、准备工具  
链接:[https://pan.baidu.com/s/1f9lppZUr\_Jq6qgiCXyoJ6A](https://pan.baidu.com/s/1f9lppZUr_Jq6qgiCXyoJ6A)  密码:cvzw

![下载镜像文件](https://www.lategege.com/2020/06/image-1024x263.png)

四个文件都放在网盘里了。

二、将三个镜像传入pve

![上传镜像文件](https://www.lategege.com/2020/06/image-2-1024x490.png)

如果需要clover引导，将pie-edk2-fimware\_2.20191127-1\_all.deb 上传至pve宿主机  可通过scp、ftp等工具 随意。opencore引导可忽略  
三、创建虚拟机

![](https://www.lategege.com/2020/06/image-3-1024x750.png)

![](https://www.lategege.com/2020/06/image-5-1024x748.png)

![系统设置](https://www.lategege.com/2020/06/image-1-1024x745.png)

![硬盘设置](https://www.lategege.com/2020/06/image-4-1024x746.png)

![cpu设置](https://www.lategege.com/2020/06/image-6-1024x747.png)

![内存设置](https://www.lategege.com/2020/06/image-7-1024x743.png)

![网卡设置](https://www.lategege.com/2020/06/image-8-1024x744.png)

由于本文是clover、opencore双引导教程  所以创建三个CD-ROM，如果单一引导只需两个

![添加opencore镜像](https://www.lategege.com/2020/06/image-9.png)

![添加clover镜像](https://www.lategege.com/2020/06/image-10.png)

![添加mac镜像](https://www.lategege.com/2020/06/image-11.png)

最终结果

![](https://www.lategege.com/2020/06/image-13-1024x577.png)

四、配置虚拟机  
由于mac驱动不支持CD驱动器, 所以这边要把CD驱动设备改成硬盘设备、另外黑苹果引导需要对cpu参数做一些处理，图形话界面无法修改，需要进入shell修改，这两个修改是同一个配置文件 我们创建的虚拟起id是105 那么这个虚拟机的配置文件路径为 /etc/pve/qemu-server/105.conf  
通过vim工具对/etc/pve/qemu-server/105.conf进行修改

![添加pve虚拟机参数](https://www.lategege.com/2020/06/image-12-1024x217.png)

新增 args参数  Intel cpu:     在 -cpu 部分可如下配置: -cpu host,kvm=on,vendor=GenuineIntel,+kvm\_pv\_unhalt,+kvm\_pv\_eoi,+hypervisor,+invtsc  
amd 和intel cpu 通用配置:  在 -cpu部分可如下配置: -cpu Penryn,kvm=on,vendor=GenuineIntel,+kvm\_pv\_unhalt,+kvm\_pv\_eoi,+hypervisor,+invtsc,+pcid,+ssse3,+sse4.2,+popcnt,+avx,+avx2,+aes,+fma,+fma4,+bmi1,+bmi2,+xsave,+xsaveopt,check  
intel cpu 配置为host 就是直接使用宿主cpu 在虚拟机中可运行虚拟机、docker 实现嵌套虚拟化 amd cpu 必须模拟成penryn架构、无法嵌套虚拟化，intel cpu也可以模拟penny 当然性能会较host有所损耗 media-cdrom 修改为 cache-unsafe 使得cd-rom设备变为硬盘设备

五、配置宿主机  
echo “options kvm ignore\_msrs=Y” >> /etc/modprobe.d/kvm.conf && update-initramfs -k all -u  
以上命令是避免引导循环启动的  
clover引导请安装刚刚上传进pve的包  
shell执行以下命令  
安装:dpkg -i pve-edk2-firmware\_2.20191127-1\_all.deb   
阻止更新:apt-mark hold pve-edk2-firmware 

六、安装

![修改磁盘格式](https://www.lategege.com/2020/06/image-14-1024x499.png)

Ide0 clover  ide1 open core

![pve引导顺序](https://www.lategege.com/2020/06/image-15.png)

修改引导顺序 如果你选择clover 引导 选择clover对应的ide 如果选择opencore引导 选择opencore对应的ide

修改分辨率 进入ovmf画面 按esc 

![ovmf引导](https://www.lategege.com/2020/06/image-16-1024x714.png)

ov

Device manager ->ovum platform configuration->change preferred ->1920\*1080 修改分辨率    
修改分辨率是为乐防止引导界面出现显示问题、如果你那边正常可不修改  
接下来就是正常引导安装

![安装macos分区](https://www.lategege.com/2020/06/image-17-1024x602.png)

就这一步需要注意下 抹盘分区选guid 格式选apfs 名字随便起，其他安装部分略过  
两种引导方式安装大同小异，新手不用怕错、因为这是虚拟机，虚拟机你可以随便折腾，硬盘有问题就重新建一个就行了，不需要重新创建虚拟机。

![磁盘工具](https://www.lategege.com/2020/06/image.jpeg)

安装完成将efi引导写入虚拟机系统磁盘的efi分区 sudo dd if=/dev/disk1s1 of=/dev/disk2s1 然后删除掉ide clover和 ide opencore 改为实际硬盘启动即可

后续会有pve黑苹果显卡直通教程，请留意本网站[https://lategege.com](https://lategege.com/) late哥哥笔记