搭建 Linux 虚拟机，别再用 VirtualBox 从 .iso 文件安装了。

## 概述

2020 年了，也许你已经习惯了 docker，习惯了在 XX 云上快速创建云主机，但是如果你想在个人电脑上安装虚拟机来搭建开发/测试环境，Vagrant 仍然不失高效之选。

> 文末有和 docker 的对比说明。

**本篇内容字数过万，收藏的同时别忘了点个赞，谢谢！**

## 安装软件

**安装 VirtualBox**

进入 VirtualBox 的[主页](https://link.zhihu.com/?target=https%3A//www.virtualbox.org/)，点击大大的下载按钮，即可进入下载页面。

VirtualBox 是一个跨平台的虚拟化工具，支持多个操作系统，根据自己的情况选择对应的版本下载即可。

注意，除了主程序，还要把对应的**扩展包程序**也一并下载了。有些高级特性，比如 USB 3.0 等需要扩展包的支持。

![](https://pic3.zhimg.com/v2-a1225a9c7e8d405a35588542342cc6da_b.jpg)

在安装完主程序后，直接双击扩展包文件即可安装扩展包。

> 下载页面先别关，后面还要用到。

**安装 Vagrant**

在 [Vagant 网站](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/)下载最新的版本，根据自己的操作系统选择对应的版本下载即可。

注意，Vagrant 是没有图形界面的，所以安装完成后也没有桌面快捷方式。具体使用方法，接下来会详细说明。

Vagrant 的安装程序会自动把安装路径加入到 PATH 环境变量，所以，这时候可以通过命令行执行 `vagrant version` 检查是否安装成功：

```
> vagrant version
Installed Version: 2.2.7
Latest Version: 2.2.8
```

## 配置虚机存放位置

创建虚拟机会占用较多的磁盘空间，在 Windows 系统下默认的虚机创建位置是在 C 盘，所以最好配置到其它地方。

**配置 VirtualBox**

启动 VirtualBox 后，通过菜单 `管理` -> `全局设定`，或者按下快捷键 `Ctrl + g`，在全局设定对话框中，修改 `默认虚拟电脑位置`，指定一个容量较大的磁盘。

![](https://pic3.zhimg.com/v2-4371dddf89e26ad3de5dded485a28eaa_b.jpg)

**配置 Vagrant**

通过 Vagrant 创建虚机需要先导入镜像文件，也就是 `box`，它们默认存储的位置在用户目录下的 `.vagrant.d` 目录下，对于 Windows 系统来说，就是 `C:\Users\用户名\.vagrant.d`。

如果后续可能会用到较多镜像，或者你的 C 盘空间比较紧缺，可以通过设置环境变量 `VAGRANT_HOME` 来设置该目录。

在 Windows 系统中，可以这样操作：新建系统环境变量，环境变量名为 `VAGRANT_HOME`，变量值为 `E:\VirtualBox\.vagrant.d`

![](https://pic4.zhimg.com/v2-50a5223ae160aa6452991357a29b62b3_b.jpg)

> **注意**，最后这个 `.vagrant.d` 目录名称不是必须的，但是建议保持一致，这样一眼看上去就能知道这个目录是做什么用处的了。

## 下载虚机镜像

使用 Vagrant 创建虚机时，需要指定一个镜像，也就是 `box`。开始这个 box 不存在，所以 Vagrant 会先从网上下载，然后缓存在本地目录中。

Vagrant 有一个[镜像网站](https://link.zhihu.com/?target=https%3A//app.vagrantup.com/boxes/search)，里面列出了都有哪些镜像可以用，并且提供了操作文档。

但是这里默认下载往往会比较慢，所以下面我会介绍如何在其它地方下载到基础镜像，然后按照自己的需要重置。如果网速较好，下载顺利的朋友可以选择性地跳过部分内容。

下面我给出最常用的两个 Linux 操作系统镜像的下载地址：

**CentOS**

CentOS 的镜像下载网站是： [http://cloud.centos.org/centos/](https://link.zhihu.com/?target=http%3A//cloud.centos.org/centos/)

在其中选择自己想要下载的版本，列表中有一个 `vagrant` 目录，里面是专门为 vagrant 构建的镜像。选择其中的 `.box` 后缀的文件下载即可。这里可以使用下载工具，以较快的速度下载下来。

这里我们选择下载的是 [CentOS 7 的最新版本](https://link.zhihu.com/?target=http%3A//cloud.centos.org/centos/7/vagrant/x86_64/images/CentOS-7.box)

**Ubuntu**

Ubuntu 的镜像下载网站是： [http://cloud-images.ubuntu.com/](https://link.zhihu.com/?target=http%3A//cloud-images.ubuntu.com/)

同样先选择想要的版本，然后选择针对 vagrant 的 `.box` 文件即可。

如果这里官网的速度较慢，还可以从 [清华大学的镜像站](https://link.zhihu.com/?target=https%3A//mirror.tuna.tsinghua.edu.cn/ubuntu-cloud-images/) 下载。

下面的例子以 CentOS 7 为例，使用其它版本操作系统的也可以参考。

## 添加 box

接下来我们需要将下载后的 `.box` 文件添加到 vagrant 中。

Vagrant 没有 GUI，只能从命令行访问，先启动一个命令行，然后执行:

```
$ vagrant box list
There are no installed boxes! Use `vagrant box add` to add some.

```

提示现在还没有 box。如果这是第一次运行，此时 `VAGRANT_HOME` 目录下会自动生成若干的文件和文件夹，其中有一个 `boxes` 文件夹，这就是要存放 box 文件的地方。

执行 `vagrant box add` 命令添加 box:

```
$ vagrant box add e:\Downloads\CentOS-7.box --name centos-7
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos-7' (v0) for provider:
    box: Unpacking necessary files from: file:///e:/Downloads/CentOS-7.box
    box:
==> box: Successfully added box 'centos-7' (v0) for 'virtualbox'!
```

命令后面跟着的是下载的文件路径，并且通过 `--name centos-7` 为这个 box 指定一个名字。

后面创建虚机都需要指定这个名字，所以尽量把名字取得简短一点，同时也要能标识出这个镜像的信息（我们后面会定制自己的基础镜像，所以这里可以简单点）。

再次查询，可以看到有了一个 box：

```
$ vagrant box list
centos-7 (virtualbox, 0)
```

### 新建虚机

创建一个目录，先执行 `vagrant init`：

```
$ mkdir demo
$ cd demo
$ vagrant init centos-7
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

其中的 `centos-7` 就是我们要使用的 `box` 名字。

这个命令只是为我们生成一个 `Vagrantfile`，所以，这里的名字没指定或者写错了都没关系，后面会介绍如何编辑这个 `Vagrantfile` 来修改。

### 启动虚机

我们等会再来细看这个文件，现在直接按照提示执行 `vagrant up`：

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'centos-7'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: demo_default_1588406874156_65036
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
```

正常的情况下，不到一分钟应该就能启动成功了。

> 这里我遇到点问题，2222 端口转发出现未知错误，造成 vagrant 的启动超时。检查了这个端口并没被占用，同时更改大一点的端口可以转发成功，所以应该是系统哪里有点问题。在后面我介绍了如何处理，如果你也遇到和我一样的情况，可先跳到下面查看。  

注意到这里包含的信息：

-   虚机名称：`demo_default_1588406874156_65036`（想改？最后有提）
-   网卡：`Adapter 1: nat`，第一块网卡，NAT 模式，这是固定的
-   端口转发：`22 (guest) => 2222 (host) (adapter 1)`，把虚机的 22 端口，映射到宿主机的 2222 端口上，这样就可以通过 `127.0.0.1:2222` 访问虚拟机了
-   SSH 用户名：`vagrant`，这里使用 `private key` 登录

> 密码也是 `vagrant`，但是密码方式仅供直接登录，是不能通过 SSH 登录的。  

### 查看虚机状态

执行下面的命令可以查看虚机的状态：

```
vagrant status

Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.

```

该命令还提示了如何操作虚机，我们继续一一介绍

### 连接虚机

如果启动没问题，接下来执行 `vagrant ssh` 就能以 `vagrant` 用户直接登入虚机中。

`root` 用户没有默认密码，也不能直接登录。需要 root 权限的命令可以通过在命令前添加 `sudo` 来执行，也可以执行 `sudo -i` 直接切换到 `root` 用户。

这时候打开 VirtualBox 程序，可以看到自动创建的虚机：

![](https://pic4.zhimg.com/v2-34eb6f9e2fc34aea3ae2d4d80328ff7f_b.jpg)

我们也可以在 VirtualBox 的终端上登录系统，默认的登录用户名和密码都是 `vagrant`。

当然还可以使用其它的 SSH 连接工具例如 XShell，SecureCRT 连接，但是这里默认网卡使用的是 NAT 模式，没有指定 IP，实际应用并不方便，我们在后面介绍网络配置时再详细介绍如何连接虚机。

### 停止虚机

执行下面的命令可以关闭虚机：

直接在 VirtualBox 上关闭虚机，或者直接在虚机内部执行 `poweroff` 命令也都是可以的。

### 暂停虚机

执行下面的命令可以暂停虚机：

### 恢复虚机

执行下面的命令把暂停状态的虚机恢复运行：

**注意：** 不管虚机是关闭还是暂停状态，甚至是 error 状态，都可以执行 `vagrant up` 来让虚机恢复运行。

### 重载虚机

执行下面的命令会重启虚机，并且重新加载 `Vagrantfile` 中的配置信息：

### 删除虚机

最后，执行下面的命令可以彻底删除虚机，包括整个虚机文件：

**注意：** 在当前这个小例子中，上面所有的 `vagrant` 命令都需要在 `Vagrantfile` 所在的目录下执行。

## 初识 Vagrantfile

先来认识一下默认的 `Vagrantfile` 文件，使用带语法高亮的文本编辑器（例如 VSCode） 打开:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos-7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

这是一个 Ruby 语法的文件，因为 Vagrant 就是用 Ruby 编写的。如果编辑器没有语法高亮可以手动设置文件类型为 Ruby。

这个缺省文件内容几乎都是注释，提示有哪些配置项可以修改，我们不需要去学 Ruby 编程也可以照葫芦画瓢的完成基本的配置。

> 当然，如果会 Ruby 编程的可以在此实现更高级的作用，但是绝大多数人用不着。  

刨除注释，这个文件的实际生效内容实际只有 3 行:

```
Vagrant.configure("2") do |config|
  config.vm.box = "centos-7"
end
```

首尾两行组成一个代码块结构，不要去动它，除非你知道自己在干什么。我们平常只需要编辑这其中的配置项。

这里的 `config.vm.box` 对应的就是虚机的镜像，也就是 box 文件，这是唯一必填的配置项。

特别提醒，`Vagrantfile` 文件名是固定的写法，大小写也要完全一样，修改了就不认识了。

## 自定义配置 Vagrantfile

下面我将针对这份默认的 `Vagrantfile` 内容，逐个讲解其中的配置含义和如何根据实际情况修改。

### 配置端口转发

端口转发（Port forward）又叫端口映射，就是把虚机的某个端口，映射到宿主机的端口上。这样就能在宿主机上访问到虚拟机中的服务。

例如启动虚机时，默认的 `22 (guest) => 2222 (host) (adapter 1)` 就是把虚机的 SSH 服务端口（`22`）映射到宿主机的 `2222` 端口，这样直接在宿主机通过 ssh 客户端访问 `127.0.0.1:2222` 端口就等价于访问虚拟机的 `22` 端口。

下面这两段配置就是教我们如何配置额外的端口转发规则，例如把 Web 服务也映射出来：

```
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
```

实际上设置端口转发这个功能并不实用，一个很明显的问题就是如果启动多个虚机，很容易就出现宿主机上端口冲突的问题。即使没有端口冲突，使用起来也不方便，我个人不推荐使用的，可以把这部分配置直接删掉。直接使用下面的私有网络。

这个功能是虚拟机软件提供的，可以在虚机的网卡设置中展开高级选项，找到相关的配置：

![](https://pic1.zhimg.com/v2-4d1b277ecb3544b2b438b98388fea778_b.jpg)

还有个地方需要注意，默认的 SSH 端口映射在这里没法直接修改。比如像我这样，2222 端口出现莫名问题，如果想要把 22 端口转发到其它端口如 22222，直接添加下面这样的配置是**没用**的：

```
  config.vm.network "forwarded_port", guest: 22, host: 22222
```

它会在原来的基础上新加一个端口转发规则，而不是替代原来的，必须要先强制关闭掉默认的那条规则:

```
  config.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: "true"
  config.vm.network "forwarded_port", guest: 22, host: 22222
```

### 配置私有网络

下面这段配置用来配置私有网络，实际上对应的是 VirtualBox 的主机网络，也就是 HostOnly 网络。

```
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
```

取消注释最下面一行，就可以为虚机设置指定的私有网络地址：

```
  config.vm.network "private_network", ip: "192.168.33.10"
```

如果这个网段的主机网络在 VirtualBox 中不存在，Vagrant 会在启动虚机时自动创建。所以，如果你想要利用已有的网络，请查看现有**主机网络**配置：

![](https://pic3.zhimg.com/v2-a7962b522cc5584e95340f0aee31e902_b.jpg)

最好这个网络也不要启用 DHCP，完全由自己来分配地址，这样更加清楚。

```
  config.vm.network "private_network", ip: "192.168.56.10"
```

修改完成后，执行 `vagrant reload` 命令重建虚机，就能看到多出来的网卡了。

私有网络实际也可以直接使用 DHCP，但是并不推荐：

```
  config.vm.network "private_network", type: "dhcp"
```

### 配置公共网络

下面这条配置用来配置公共网络：

```
  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"
```

正如注释所说，这里通常对应的就是桥接网络。实际开发场景下，我们极少会需要把虚机暴露到公共网络上，这样既不安全，也没有必要。

默认所起的第 1 个 NAT 网络已经保证了虚机可以上互联网，而私有网络保证了宿主机和虚机，以及虚机和虚机之间的通信。如果有对外暴露服务的需求，还可以使用端口转发。我实在想不出什么情况下是必须要用桥接网络的。

所以这部分配置可以直接删除，如确有使用的，可以参考 [官方文档](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/networking/public_network.html)。

### 配置同步文件夹

下面的配置项用来配置同步文件夹:

```
  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"
```

在启动虚机的时候，我们可以看到这样的提示：

```
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: No guest additions were detected on the base box for this VM! Guest
    default: additions are required for forwarded ports, shared folders, host only
    default: networking, and more. If SSH fails on this machine, please install
    default: the guest additions and repackage the box to continue.
    default:
    default: This is not an error message; everything may continue to work properly,
    default: in which case you may ignore this message.
==> default: Configuring and enabling network interfaces...
==> default: Rsyncing folder: /cygdrive/c/Users/Davy/demo/ => /vagrant
```

先注意最后一行的提示：`Rsyncing folder: /cygdrive/c/Users/Davy/demo/ => /vagrant`

-   `/cygdrive/c/Users/Davy/demo/` 这是宿主机的本地目录，也就是 `Vagrantfile` 所在的目录。
-   `/vagrant` 是虚拟机内部的路径
-   `Rsyncing` 表示同步的方式是 Rsync

> 宿主机目录中出现 `/cygdrive` 是因为 Vagrant 程序用到了 Cygwin，它是在 Windows 系统中兼容 Linux/POSIX 的模拟层。可以把 `/cygdrive` 看成是虚拟的根目录。

这是 Vagrant 默认的同步文件夹设置，别忘了 Vagrant 的作用是用来搭建开发环境的。所以它假定了当前目录是我们的开发项目所在目录，自动把本地的项目目录同步到虚机中，就可以快速的开始开发调试工作了。

1.  在 `demo` 目录下创建一些文件，例如 `hello.py`
2.  执行 `vagrant reload`，重启虚机
3.  在虚机启动完成后登录到虚机内，操作如下：

```
$ vagrant ssh
Last login: Sat May  2 16:25:00 2020 from 10.0.2.2
[vagrant@localhost ~]$ cd /vagrant/
[vagrant@localhost vagrant]$ ls
hello.py  Vagrantfile
[vagrant@localhost vagrant]$ python hello.py
helloworld
```

这种同步方式在大多数情况下都能提供便利，不过也有不足之处：

-   同步是一次性的，即只有启动虚机的时候执行，也就是说改了代码必须要重启一次虚机
-   单向的，即只能从宿主机同步到虚拟机，也就是说在虚机内的改动不会同步到外面
-   需要拷贝文件，如果要同步的文件数量较多，会占用更多的磁盘空间

让我们按照默认配置的提示来新加一条同步文件夹配置试试：

```
config.vm.synced_folder "../data", "/vagrant_data"
```

注意，别忘了先在宿主机上创建 `data` 文件夹，重启虚机可能看到下面的错误提示：

```
==> default: Rsyncing folder: /cygdrive/c/Users/Davy/demo/ => /vagrant
==> default: Mounting shared folders...
    default: /vagrant_data => C:/Users/Davy/data
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 vagrant_data /vagrant_data

The error output from the command was:

mount: unknown filesystem type 'vboxsf'
```

这是因为 Vagrant 提供了多种同步方式，在使用 VirtualBox 的时候，缺省同步类型是 [vboxsf 挂载文件系统](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/synced-folders/virtualbox.html)，它需要在虚拟机内部安装客户机增强包，也就是 `VirtualBox Guest Additions`（输出信息中也提示了）。

如何在虚机系统中安装 `guest additions` 要分操作系统而定，有点小坑，后面会细说，现在修改一下配置，明确指定同步类型是 `rsync`：

```
config.vm.synced_folder "../data", "/vagrant_data", type: "rsync"
```

这样表示仍然使用 `rsync` 来单向同步。

### 更改虚机规格

VirtualBox 等虚拟机软件在 Vagrant 中被称为 Provider，虚机的规格等配置是和 Provider 相关的。因为 VirtualBox 用的最多，所以默认的配置提示是以 VirtualBox 举例。

如果想要了解其它 Provider 的配置，请参考 [文档](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/providers/)

把中间那一段取消注释，其它的可以删掉：

```
  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.
```

使用 VSCode 时，选中它们，然后按下快捷键 `Ctrl + /` 即可：

```
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "1024"
  end
```

`vb.gui = true` 是在虚机启动时自动打开 VirtualBox 的图形界面，这对服务器来说没什么用，直接删掉。

添加 CPU 的配置，同时修改内存大小：

```
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end
```

注意到，内存的大小单位是 MB，值是数字，默认的示例中有引号，实际也可以不加。

特别提醒一下，在 [说明文档](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/virtualbox/configuration.html) 里给的例子，其中的变量名是 `v`，这其实是在双竖线中定义的，直接拷贝的时候要看清楚。

```
config.vm.provider "virtualbox" do |v|
  v.memory = 1024
  v.cpus = 2
end
```

### Provision

Provision 是指在虚机初次创建的时候，Vagrant 自动去执行的构造任务，比如安装软件，更新系统配置等。

因为 box 往往只提供基础的系统（虽然我们可以自定义 box，但是并不是每次都要这么做，而且这样做会丧失一部分灵活性），有些东西仍然需要在创建虚机的时候完成。

```
  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
```

因为这部分完全是个开放的内容，所以我们这里不过多讨论，来看一下什么情况下会触发 provision 的操作：

-   某个环境初次执行 `vagrant up` 的时候
-   执行 `vagrant provision` 命令
-   重启的时候 `vagrant reload --provision`，带上 `--provision` 选项

除了上面这些默认提示给出的配置项，Vagrantfile 还支持其它很多配置，具体请 [查看文档](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/vagrantfile/)。

由于 Vagrantfile 本身是 Ruby 脚本，所以它并不仅仅是静态的配置文件，而且可以包含程序逻辑，例如在 [如何创建多个虚机](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/multi-machine/) 中就有应用，有兴趣的可以自行研究。

## 定制带客户机增强的 box

### 下载 Guest Addition

VirtualBox 的[下载页面](https://link.zhihu.com/?target=https%3A//www.virtualbox.org/wiki/Downloads)并没有直接给出 Guest Addtion 的下载链接，我们先在 VirtualBox 的任一下载链接上右键，复制链接地址，例如得到 `https://download.virtualbox.org/virtualbox/6.1.6/VirtualBox-6.1.6-137129-Win.exe`，去掉最后的文件名，把其中的路径 `https://download.virtualbox.org/virtualbox/6.1.6/` 在浏览器中打开，就能看到所有可下载的版本，在其中找到 `VBoxGuestAdditions_6.1.6.iso` 直接下载。

![](https://pic2.zhimg.com/v2-6a207372c58ba6ff0bc7c8da9f5f015d_b.jpg)

### 安装 Guest Addition

重新使用 Vagrant 从原始的镜像启动一个干净的虚机。

`VBoxGuestAdditions_6.1.6.iso` 需要以光盘的形式挂载到虚机上，但是默认启动的这个虚机是没有光驱的。添加虚拟光驱需要先将虚机关闭。

然后在 VirtualBox 界面上操作，打开 `设置`，选择 `存储`，点击 `添加虚拟光驱`，点击 `控制器： IDE`，选择 `VBoxGuestAdditions_6.1.6.iso`，点击 `OK`

直接在 VirtualBox 上启动虚机，如果你在虚机菜单上选择 `设备` - `安装增强功能`，大概率是会遇到下面这样的错误，别管它，我们手动来安装。

![](https://pic2.zhimg.com/v2-a12c3083ad70b943faaed4f822e6c7c5_b.jpg)

![](https://pic2.zhimg.com/v2-8be73777016fdb8ffc17ce6eb58e3179_b.jpg)

使用 `vagrant/vagrant` 登录到虚机内，切换到 `root` 用户，查看虚拟光盘是否已经挂载上来了：

```
$ sudo -i
# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0  40G  0 disk
└─sda1   8:1    0  40G  0 part /
sr0     11:0    1  57M  0 rom
```

下面的 `sr0` 就是光盘设备，要把它挂载到文件系统中

```
# mount /dev/sr0 /mnt/
mount: /dev/sr0 is write-protected, mounting read-only
# cd /mnt/
# ls
AUTORUN.INF  cert  OS2           TRANS.TBL                VBoxDarwinAdditionsUninstall.tool  VBoxSolarisAdditions.pkg        VBoxWindowsAdditions.exe
autorun.sh   NT3x  runasroot.sh  VBoxDarwinAdditions.pkg  VBoxLinuxAdditions.run             VBoxWindowsAdditions-amd64.exe  VBoxWindowsAdditions-x86.exe
```

在 Linux 系统中要运行 `VBoxLinuxAdditions.run`，这时候直接运行仍然会报错：

![](https://pic2.zhimg.com/v2-48cba2e05faeddc156215d4a4896e61d_b.jpg)

大致意思是要它需要内核的头文件来构建。

头文件通过安装 `kernel-devel` 就可以了，但是直接安装的版本是最新版本的，可能会当前的内核版本不一致，仍然是没用的。例如：

```
[root@localhost mnt]# uname -r
3.10.0-1062.12.1.el7.x86_64
[root@localhost mnt]# yum info kernel-devel
Installed Packages
Name        : kernel-devel
Arch        : x86_64
Version     : 3.10.0
Release     : 1127.el7
Size        : 38 M
Repo        : installed
From repo   : base

```

这里 `kernel-devel` 的 `Release` 版本号和系统内核版本不一致，所以仍然不行。

需要先更新一把，然后再安装（也可以先安装再更新）：

```
# yum update -y
# yum install -y gcc kernel-devel
```

> 这里选择的是更新到最新版本，如果你的开发环境需要特定的内核版本，你也可以根据情况安装指定的版本，只要**保证内核版本和头文件版本完全匹配**。
> 
> Ubuntu 系统的安装方式有所不同，可以参考 [Vagrant 的文档](https://link.zhihu.com/?target=https%3A//www.vagrantup.com/docs/virtualbox/boxes.html)

升级内核后需要重启虚机，然后再次尝试安装 Additions：

```
[root@localhost mnt]# ./VBoxLinuxAdditions.run
Verifying archive integrity... All good.
Uncompressing VirtualBox 6.1.6 Guest Additions for Linux........
VirtualBox Guest Additions installer
Removing installed version 6.1.6 of VirtualBox Guest Additions...
Copying additional installer modules ...
Installing additional modules ...
VirtualBox Guest Additions: Starting.
VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel
modules.  This may take a while.
VirtualBox Guest Additions: To build modules for other installed kernels, run
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup <version>
VirtualBox Guest Additions: or
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup all
VirtualBox Guest Additions: Building the modules for kernel
3.10.0-1127.el7.x86_64.
VirtualBox Guest Additions: Running kernel modules will not be replaced until
the system is restarted

# 检查模块是否已经加载了
[root@localhost mnt]# lsmod |grep vbox
vboxvideo              35867  1
ttm                    96673  1 vboxvideo
drm_kms_helper        186531  1 vboxvideo
drm                   456166  4 ttm,drm_kms_helper,vboxvideo
vboxguest             349038  1
[root@localhost mnt]#
```

能够看到 `vboxguest` 就代表安装成功了。

### 测试

在 Vagrantfile 中添加同步文件夹设置，这次不再指定同步类型

```
config.vm.synced_folder "../data", "/vagrant_data"
```

然后执行 `vagrant reload`

```
$ vagrant reload
...
==> default: Checking for guest additions in VM...
==> default: Rsyncing folder: /cygdrive/c/Users/Davy/demo/ => /vagrant
==> default: Mounting shared folders...
    default: /vagrant_data => C:/Users/Davy/data
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```

可以看到关于 additions 的提示信息没有了，新的同步文件夹也能正常同步了，这次的同步是双向的。我们可以先去同步文件夹随便创建一个文件：

```
$ vagrant ssh
Last login: Tue May  5 12:16:39 2020 from 10.0.2.2
[vagrant@localhost ~]$ cd /vagrant_data/
[vagrant@localhost vagrant_data]$ ls
data.txt
[vagrant@localhost vagrant_data]$ touch vm.txt   # 新建一个文件
[vagrant@localhost vagrant_data]$ ls
data.txt  vm.txt
```

回到 Windows 宿主机上，`data` 文件夹下面也能看到 `vm.txt` 文件，

```
Davy@Davy-Desktop MINGW64 ~/data
$ ls
data.txt  vm.txt
```

这样，我们的 Guest Addition 就安装成功了。

### 清理磁盘

接下来我们来把这个好不容易才装上了增强包的虚机保存为新的镜像。

> 这里你可能还想安装点其它软件，但是作为基础镜像，最好还是保持干净一点，不宜安装太多东西。后续可以在这个基础之上，再次构建其它特定的镜像。  

为了使做出来的镜像文件大小紧凑点，我们把刚才安装过程中的缓存也删掉：

我在打包过程中还遇到过 swapfile 占用磁盘空间的问题，导致镜像文件过大，可以这样检查：

```
du /swapfile   # 查看是否有占用
swapoff -a     # 关闭 SWAP
rm -f /swapfile
```

使用 `df -h` 命令查看磁盘占用情况，像我这次操作，磁盘根目录不到 2GB 的空间占用：

```
# df -h
/dev/sda1        40G  1.5G   39G   4% /
```

### 打包为 box

执行 `vagrant package` 就可以把当前环境打包生成新的 box 文件：

```
$ vagrant  package
    ==> default: Attempting graceful shutdown of VM...
==> default: Clearing any previously set forwarded ports...
==> default: Exporting VM...
==> default: Compressing package to: C:/Users/Davy/demo/package.box

```

生成的文件就在 Dockerfile 所在的目录，文件名默认是 `package.box`。大小不到 600MB。

**注意：**，因为默认启动虚机时本地目录会 rsync 到虚机中，`package.box` 文件也会同步（拷贝）到虚机中，占用虚机磁盘。这时候如果二次执行打包，生成的文件大小会翻倍。

### 添加到 vagrant 中

继续使用下面的命令把新建的 box 添加到 vagrant 中：

```
$ vagrant box add package.box --name davy/centos-7-base
```

为了区分这是个人创建的基础镜像，加了个人用户名作为前缀，同时加了 `base` 后缀。

添加成功后，本地的 `package.box` 就可以删除了。

后续再创建虚机，使用下面的命令就可以了：

```
$ vagrant init davy/centos-7-base
```

## 使用 SSH 客户端

`vagrant ssh` 命令虽然很方便，但是在 Windows 环境下，因为默认的命令行终端不太好用，所以往往还需要使用更专业的 SSH 客户端例如 XShell 或 SecureCRT。

默认的镜像只支持 private\_key 的方式登录，`vagrant/vagrant` 可以在 VirtualBox 上登录系统，但是如果用来登录 SSH，会被拒绝。

当然你可以在制作镜像的时候修改 ssh 服务的配置，让它能够用密码登录，但是实际上用密钥更加方便。

先使用 `vagrant ssh-config` 命令可以看到 SSH 的配置：

```
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 22222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile E:/VirtualBox/.vagrant.d/boxes/davy-VAGRANTSLASH-centos-7-base/0/virtualbox/vagrant_private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

可以看到其中的 `IdentityFile` 就是私钥文件。

发现这个自定义 box 启动的虚机的密钥文件是固定在 `VAGRANT_HOME` 下的相关目录下。那么就好办了，直接在 SSH 客户端软件上导入这个私钥文件就可以了。

以 SecureCRT 为例：

![](https://pic1.zhimg.com/v2-26a045d8d3ae2db9f3f90e55050cfc9c_b.jpg)

## 使用模板文件

`vagrant init` 命令只是用来生成 Vagrantfile 文件，但是默认的配置选项每次都要修改也很麻烦。该命令提供了 `--template` 选项，可以指定一个模板文件，我们可以在自定义自己的模板文件。

这个模板文件的格式 `ERB` 是 `Ruby` 的模板语法，如果有 `Ruby on Rails` 开发经验的可能会比较熟悉。但是我们不用去学习这些细节。

可以从 [这里](https://link.zhihu.com/?target=https%3A//github.com/hashicorp/vagrant/blob/master/templates/commands/init/Vagrantfile.erb) 拷贝一份原文件，还能在 Vagrant 的安装位置 `Vagrant\embedded\gems\2.2.7\gems\vagrant-2.2.7\templates\commands\init` 底下找到，并且有一个 `Vagrantfile.min.erb` 是去掉所有注释的：

```
Vagrant.configure("2") do |config|
  config.vm.box = "<%= box_name %>"
  <% if box_version -%>
  config.vm.box_version = "<%= box_version %>"
  <% end -%>
  <% if box_url -%>
  config.vm.box_url = "<%= box_url %>"
  <% end -%>
end

```

可以看到其中是怎么配置 `config.vm.box` 的。 像 `<%` 这样的语法有兴趣可以自己去了解，这里我们只要把自己想要的配置项原样写上去就行了。

下面是我按照自己的需要写的：

```
Vagrant.configure("2") do |config|
  config.vm.box = "<%= box_name %>"
  config.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: "true"
  config.vm.network "forwarded_port", guest: 22, host: 22222

  # config.vm.network "private_network", ip: "192.168.56.10"
  # config.vm.synced_folder "../data", "/data"

  config.vm.provider "virtualbox" do |vb|
    # vb.name = "give me a better name"
    vb.memory = "1024"
  end
end
```

> 不要有中文，不然会遇到编码的麻烦。  

其中像私有网络和同步文件夹配置，几乎每次基本都要，但是又不好固定，所以仍然以注释的形式保留，每次稍微改一下也很方便。

把这个文件找个目录，保存为 `vagrant.erb`。

接着在使用 `vagrant init` 的时候通过 `--tempate` 指定它就可以了，例如：

```
vagrant init davy/centos-7-base --tempate "C:\Users\Davy\vagrant.erb"
```

显然，每次要记住并且输入这个模板文件也很麻烦的。可以通过设置环境变量 `VAGRANT_DEFAULT_TEMPLATE` 来一劳永逸地解决这个问题。

## 尾声

费了莫大的力气，终于可以比较愉快地玩耍了。虽然也只是刚把基础镜像搞定了，后面可能还要针对不同用途的环境编写更加复杂的 Vagrantfile。

现在很多人刚认识到 Vagrant 之后都会问，[Vagrant 和 Docker 的区别是什么？](https://www.zhihu.com/question/32324376)

在容器流行之前，Vagrant 就是用来编排虚机和自动部署开发环境的，有了 Docker/Kubernetes 之后，直接用容器来编排应用确实更香。但是还有一些工作，例如容器平台自身的安装，多节点集群的部署测试等，更方便用虚机解决。

此外，现在 Windows 中还可以通过 WSL 使用 Linux 系统，但是使用场景上还是有所不同。Vagrant 更多地用于快速搭建可重用的开发环境，从这个角度看，Vagrant 其实好比 IaaS 云平台，只不过规模局限在个人电脑上。

___

感谢您的阅读，请继续关注「**云计算实验室**」。