**制作windows7+10-qcow2镜像**  
1.工具准备：vmware 14、virtio-win-0.1.141.iso（虚拟驱动）、cloudbase-init（虚拟机初始化工具）、win-7.iso/win-10.iso。

vmware：[https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html](https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html)  
virtio：[https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html](https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html)  
cloudbase-init：[https://cloudbase.it/cloudbase-init/#download](https://cloudbase.it/cloudbase-init/#download)

2.创建一台VMware虚拟机（**桌面版**）， 开启VT和CPU性能计数器。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190218133130973.png)  
3.安装kvm，并启动libvirtd并设置开机启动

```
# yum install epel* -y 
# yum -y install qemu-kvm libvirt virt-install bridge-utils virt-manager libguestfs-tools-c
# systemctl start libvirtd 
# systemctl enable libvirtd 

如果libvirtd 启动失败请执行：yum update librbd1  然后再次启动libvirtd 
如果字体乱码请执行： yum install dejavu-lgc-sans-fonts -y
```

4.创建qcow2文件，并启动虚拟机。

```
方法一：
# qemu-img create -f qcow2 ws7.qcow2 15G

# virt-install --connect qemu:///system \
  --name ws7 --ram 2048 --vcpus 2 \
  --network network=default,model=virtio \
  --disk path=ws7.qcow2,format=qcow2,device=disk,bus=virtio \
  --cdrom win-7.iso \
  --disk path=virtio-win-0.1-XX.iso,device=cdrom \
  --vnc --os-type windows
  这里启动可能会显示找不到引导CD，下面进行问题解决。
```

在命令行输入：

```
# virt-manager
这里会看到你开启的虚拟机。
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222163921669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
点击打开  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222164138775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
选择关机选项，在选择**强制关机**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222163955947.png)  
关机后，点击上图标记的按钮。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222164303840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
设置引导选项，哪个是**系统引导的光盘**就把那个选中（都选中也不影响），完成后再开机，问题解决。

5.加载virtio驱动程序。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222165656459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222165727177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222165928837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
在virtio-CD中找到 viostor文件夹选择win7子文件夹，再选择对应驱动版本（我的是admin64）点击确定。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222170328430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222170356371.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTk4Mjkw,size_16,color_FFFFFF,t_70)  
点击下一步，安装完成后，再安装虚拟网卡驱动，位置 **virtio-CD/NetKvm/win7/admin64**安装完成后，进行操作系统安装，安装过程不提供。

7.系统安装完成后，安装cloudbase-init（不提供安装方法），安装完成后关机，上传至openstack中。