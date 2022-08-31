## windows镜像加载驱动

[![](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)](https://www.jianshu.com/u/4d9702991b17)

2021.04.27 10:33:43字数 448阅读 436

背景是制作windows的qcow2镜像，之前的方式是通过vmware启动一个windows 虚拟机之后，导出ova文件，  
通过生成的.vmdk文件，在linux上通过qemu-img的方式转化为.qcow2

```
qemu-img convert -O qcow2 <Windows 虚拟机>.vmdk <输出路径>.qcow2
```

使用这个.qcow2 镜像生成的虚拟机会蓝屏，原因是：从vmware迁移到kvm上缺少必要的存储驱动，redhat给了提供了virtio的  
解决方案，需要现将virtio 先安装到windows虚拟机中。

最新稳定版本的virtio版本为0.1-85 ，但我要安装的windows版本为win7，virtio版本只支持win8.1以上的内容，需要  
去找原来的旧的版本，virtio的所有版本的地址是：

```
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/
```

在比较新的版本0.1.164中发现了对应win7的驱动，但这个版本的iso镜像中，并没有可执行文件，只有.info 以及.vfd的文件，  
我没有找到对应的安装方式。但我找到了如何将这部分驱动写入到 iso镜像的方式

参考文章\[windows镜像如何添加virtio驱动\]（[https://hazx.hmacg.cn/server/windows-iso-add-driver.html](https://links.jianshu.com/go?to=https%3A%2F%2Fhazx.hmacg.cn%2Fserver%2Fwindows-iso-add-driver.html)）  
具体操作是使用windows 自带的dism工具（系统没有的话，需要手动安装）

```
准备 WIM 映像文件
1. 使用 UltraISO 挂载 Windows ISO 镜像到虚拟光驱。（或者使用你电脑上任意一款压缩软件打开 ISO 镜像）
2. 在 sources 文件夹下复制出（或使用压缩软件提取出）boot.wim和install.wim两个文件。
　　　boot.wim 是引导进入的安装环境，它缺少驱动时会在选择安装分区步骤时提示找不到硬盘、需要加载驱动等；install.wim 是系统映像，最终安装的系统文件都在这里面，给它添加驱动，安好系统就不用再手工打驱动了。一次性添加好驱动这对于同配置硬件环境批量安装系统的工程来讲能节省大量时间和精力。

添加驱动
　　1. 在 D 盘（或其他位置）新建一个文件夹mnt。
　　2. 使用 UltraISO 挂载 VirtIO 驱动 ISO 镜像到虚拟光驱。
　　　这里我们一般只需要 3 个驱动：Balloon、NetKVM、viostor。你也可以直接使用你电脑上任意一款压缩软件直接解压出这三个文件夹。然后进入目录看一看，按你的 ISO 镜像中系统的版本信息确定驱动的路径。例如，我的 ISO 是 WindowsServer2012R2 64 位，那么驱动就是2k12r2\amd64\这个目录下面的。
　　3. 以管理员身份运行 cmd 或 PowerShell，并进入 Windows AIK 工具包中 Dism 的所在目录，我的是C:\Program Files\Windows AIK\Tools\amd64\Servicing（如果系统已自带 dism 工具，则无需进入任何目录）
　　4. 查看 wim 映像信息：

dism /get-wiminfo /wimfile:D:\install.wim
　　　列出的信息便是这个 ISO 镜像所包含的所有系统版本。其中的 “索引” 便是我们下面要用到的版本编号。
　　5. 挂载 wim 镜像：

dism /mount-wim /wimfile:D:\install.wim /index:1 /mountdir:D:\mnt
　　　这里的 “index” 就是上一步中看到的索引编号。
　　6. 挂载完毕后，添加驱动：

dism /image:D:\mnt /add-driver /driver:F:\viostor\2k12R2\amd64\viostor.inf /forceunsigned
　　　其他两个或者更多驱动都是执行这个命令来添加驱动。驱动指定到 inf 配置文件。最后的/forceunsigned只有确定是未签名的驱动时才用的参数，微软认证的、签名的驱动不需要加这个参数。VirtIO 的 X64 驱动几乎都是未签名的。
　　7. 添加完驱动，查看一下驱动情况：

dism /image:D:\mnt /get-drivers
　　8. 确定驱动已经添加后，卸载并保存 wim 映像：

dism /unmount-wim /mountdir:D:\mnt /commit
　　9. 重复 4~8 步骤，为 boot.wim 添加驱动。
　　　install.wim 中的多个系统版本，你可以选择性添加驱动。但 boot.wim 中的所有版本建议都添加驱动。

封装新的 ISO 镜像
　　1. 使用 UltraISO 打开原 Windows ISO 安装盘镜像。
　　　注意，这里可就不能使用压缩软件了，否则会丢失 ISO 的引导数据。
　　2. 在 sources 文件夹下，删除boot.wim与install.wim。
　　3. 将添加好驱动的boot.wim与install.wim拖到 sources 下。
　　4. 点击【文件】→【另存为】，保存出一个新 ISO 镜像即可。
```

使用这种方式生成的镜像，我在vmware验证的时候，出现的问题是驱动因为没有签名导致无法正常启动机器，这种方式最后也是有问题的，但是这种加载驱动的方式可以学习

最后的方案是在我们的kvm管理系统上，直接用iso启动的windows虚拟机，可以正常使用

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)](https://www.jianshu.com/u/4d9702991b17)

总资产0.92共写了2.2W字获得16个赞共5个粉丝

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   yum install -y qemu-kvm libvirt virt-install bridge-utils...
    
    [![](https://upload-images.jianshu.io/upload_images/24586457-012948fc20be9155.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/60caf1c2f10a)
-   本章内容 ◆ 虚拟化基础◆ 虚拟化技术之KVM◆ kvm实战案例 一：虚拟化基础 https://www.vmwa...
    
    [![](https://upload-images.jianshu.io/upload_images/20946677-b77906500b2c1caf.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/3b283b340718)
-   本文是Arch Linux 线性安装手册/傻瓜书的附录部分，手册/傻瓜书地址在下面： http://www.jia...
    
    [![](https://upload.jianshu.io/users/upload_avatars/6046966/59fc94c1-7727-469a-b6d5-b7d82f7ad07c.png?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)驿窗](https://www.jianshu.com/u/a58c6640dedb)阅读 3,454评论 0赞 19
    
    [![](https://upload-images.jianshu.io/upload_images/6046966-a99f2f74ba432483.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a082f6e400d6)
-   mmp 容我先说一句 为了这个东西我搞了好几天...... 制作环境 ubuntu16.04镜像来自于 http:...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/8-a356878e44b45ab268a3b0bbaaadeeb7.jpg)墨金\_](https://www.jianshu.com/u/d1ae36865ec9)阅读 544评论 0赞 0
    
    [![](https://upload-images.jianshu.io/upload_images/4655378-b0c9b2460cb18d54.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a3f1ef4e9213)
-   1、搭建kvm虚拟环境 •KVM是一个混合类型的VMM，它能够以模拟方式支持硬件的完全虚拟 化，也能够在Guest...
    
    [![](https://upload-images.jianshu.io/upload_images/16912282-6fa7508309fce3b7.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/57f44a81882d)

-   第八章 教学评价 第一节 从考试文化走向评价文化 一、教学评价的早期发展 （一）传统考试阶段 ★《学记》——我国最...
    
-   今天青石的票圈出镜率最高的，莫过于张艺谋的新片终于定档了。 一张满溢着水墨风的海报一次次的出现在票圈里，也就是老谋...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/2-9636b13945b9ccf345bc98d0d81074eb.jpg)青石电影](https://www.jianshu.com/u/aa52975c0a31)阅读 3,610评论 0赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/680837-5490380964d6549d?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/aed76156f092)
-   今天主要学习了flex布局，学习笔记如下： 1.指定flex布局： display：flex(任意容器)...
    
-   插打法原为少林六合门打法，一代宗师万籁声将少林六合门、罗汉门、自然门等内外家之所长融为一家，自然门本无固定招式，然...
    
    [![](https://upload.jianshu.io/users/upload_avatars/8219244/f2de09a8-de86-45e6-b410-f510b562197d?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)杨会](https://www.jianshu.com/u/4ef00b38e444)阅读 3,028评论 1赞 2
    
    [![](https://upload-images.jianshu.io/upload_images/8219244-957887b6bb5f4b6d.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/25d4c2e24461)