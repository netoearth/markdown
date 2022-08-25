[Skip to content](https://azhuge233.com/proxmox-ve-%E8%99%9A%E6%8B%9F%E5%8C%96-macos-big-sur/#content)

之前在 [Proxmox VE KVM 虚拟化 macOS](https://azhuge233.com/proxmox-ve-kvm-%e8%99%9a%e6%8b%9f%e5%8c%96-macos/) 中成功新建了 Catalina 虚拟机，Big Sur 则需要改动一些配置步骤

实测 Big Sur 虚拟机没有 Catalina 流畅

下面将展示创建 Big Sur 虚拟机时需要做出的改动

## 环境

-   Proxmox VE 6.2

## 参考

-   [Installing macOS 11 “Big Sur” on Proxmox 6](https://www.nicksherlock.com/2020/06/installing-macos-big-sur-on-proxmox/)

## 步骤

Big Sur 的安装步骤与 Catalina 有两点不同

### 生成安装镜像

镜像创建需要克隆 [thenickdude](https://github.com/thenickdude/OSX-KVM) 源，此源是作者 nicksherlock 对 [kholia/OSX-KVM](https://github.com/kholia/OSX-KVM) 的魔改 fork，支持创建 BIgSur 的离线安装镜像（仅限 macOS），Linux 下只能创建 BaseSystem 镜像（安装时需要互联网连接下载 Big Sur）

1.  安装依赖
    
    \# sudo apt install g++ git qemu-utils libxml2-dev libssl-dev zlib1g-dev cmake libbz2-dev libfuse-dev fuse autoconf unzip
    
    xcode-select --install # Linux # sudo apt install g++ git qemu-utils libxml2-dev libssl-dev zlib1g-dev cmake libbz2-dev libfuse-dev fuse autoconf unzip
    
    ```
    xcode-select --install 
    # Linux 
    # sudo apt install g++ git qemu-utils libxml2-dev libssl-dev zlib1g-dev cmake libbz2-dev libfuse-dev fuse autoconf unzip
    ```
    
2.  克隆镜像
    
    git clone https://github.com/thenickdude/OSX-KVM.git
    
    git clone https://github.com/thenickdude/OSX-KVM.git
    
    ```
    git clone https://github.com/thenickdude/OSX-KVM.git
    ```
    
3.  新建安装包
    
    cd OSX-KVM/scripts/bigsur
    
    \# make BigSur-recovery.img
    
    cd OSX-KVM/scripts/bigsur make BigSur-full.img # Linux # make BigSur-recovery.img
    
    ```
    cd OSX-KVM/scripts/bigsur
    make BigSur-full.img
    # Linux
    # make BigSur-recovery.img
    ```
    
4.  将 BugSur-xxx.img 上传到 PVE
    -   如果是在 macOS 下生成的离线安装包，镜像大小约为 14-15 G

### 新建 Big Sur 虚拟机

本步骤与 [Proxmox VE KVM 虚拟化 macOS](https://azhuge233.com/proxmox-ve-kvm-%e8%99%9a%e6%8b%9f%e5%8c%96-macos/) 的区别在硬盘的配置，[Proxmox VE KVM 虚拟化 macOS](https://azhuge233.com/proxmox-ve-kvm-%e8%99%9a%e6%8b%9f%e5%8c%96-macos/) 中新建的硬盘总线为 SATA 0，这里需要改为 VirtIO Block 0，其他配置不变

其他步骤与 [Proxmox VE KVM 虚拟化 macOS](https://azhuge233.com/proxmox-ve-kvm-%e8%99%9a%e6%8b%9f%e5%8c%96-macos/) 相同，最终可以成功安装 Big Sur

## 效果

![](https://azhuge233.com/wp-content/uploads/2020/12/Snipaste_2020-12-02_13-41-56-768x333.png)![](https://azhuge233.com/wp-content/uploads/2020/12/Snipaste_2020-12-02_14-33-28-768x432.png)![](https://azhuge233.com/wp-content/uploads/2020/12/Snipaste_2020-12-02_14-33-40-768x432.png)

## 文章导航