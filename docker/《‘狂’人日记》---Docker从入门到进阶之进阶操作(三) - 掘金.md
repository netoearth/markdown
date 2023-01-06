   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

2022年08月28日 00:10 ·  阅读 15365

![《‘狂’人日记》---Docker从入门到进阶之进阶操作(三)](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/442554e2fe784a3fa36f027440f51789~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第28天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 上一章节讲到了dockerfile,用于构建镜像，那么这一章节来介绍一下使用 `supermin5`来构建镜像

## 1.什么是supermin

Supermin是用于构建Supermin设备的工具。这些是小型设备（类似于虚拟机），通常大小约为100KB，当您需要启动其中一个设备时，它们会在短短几秒钟内完全实例化。

## 1.1 基本操作

supermin工具可以在两种模式下使用，准备一个小型的supermin设备，这是在构建系统上完成的。以及building，它使用supermin设备并构建一个完整的可引导设备，这是在最终用户的系统上完成的。 Supermin不需要以root身份运行，通常不应以root身份进行运行。它不会影响主机系统或主机系统上安装的软件包。

## 1.2 准备模式

\--prepare在给定的输出目录中创建了一个小的supermin设备。你给它一个你想要安装的软件包的列表，supermin会自动找到依赖项。必须在主机上安装软件包列表。

```
supermin --prepare bash coreutils -o supermin.d
复制代码
```

## 1.3 构建模式

\--build（以前是一个名为“supermin helper”的单独程序）从supermin设备构建整个设备:

```
supermin --build --format ext2 supermin.d -o appliance.d
复制代码
```

## 1.4 构建和缓存

通常，您希望仅根据需要在最终用户计算机上重建设备。Supermin有一些额外的选项来简化此操作：

```
supermin --build \
          --if-newer --lock /run/user/`id -u`/supermin.lock \
          --format ext2 supermin.d -o appliance.d
复制代码
```

## 2.使用supermin5来构建一个基础镜像

在这个案例中，我将使用 `supermin5` 命令构建 centos7系统的 docker 镜像，镜像名称为 centos-7，镜像预装 yum、net-tools、initscripts 和 vi 命令

```
# 安装supermin
yum -y install supermin*

# supermin5 添加预装工具 yum net-tools initscripts vi
supermin5 -v --prepare bash coreutils  yum  net-tools initscripts vim-minial  -o supermin.d
# supermin5 构建
supermin5 -v --build --format chroot supermin.d -o appliance.d

echo 7 >  appliance.d/etc/yum/vars/releasever
# 镜像打包
tar --numeric-owner -cpf centos-7.tar -C appliance.d .

# 导入镜像
cat centos-7.tar | docker import - centos-7
# 运行
docker run -dit --name centos7 centos-7 /bin/bash
cat /etc/redhat-release
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/850a3f962f3340a0a31f352d77c47746~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82a37476dd6b49f6a1eaca2ca0b96067~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**至此，您已成功使用supermin5构建了一个docker镜像，那么可以尝试构建其他镜像，例如ubuntu debian此类镜像**

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 3

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏