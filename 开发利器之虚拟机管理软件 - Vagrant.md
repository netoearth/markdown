## 说明

1.  关于 Vagrant 的安装，请参阅[官方文档](https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-install)
    
2.  如果想动手实践，请先安装你熟悉的虚拟机软件，比如 [Virtualbox](https://www.virtualbox.org/)
    
3.  对于 Windows 用户，请先安装 [Cygwin](http://www.cygwin.com/)
    
4.  本文测试环境：
    
    -   操作系统：macOS 12.6
    -   虚拟机软件：Virtualbox 6.1.40
    -   Vagrant：2.3.2
5.  关于平台虚拟机化技术与容器技术之间的对比，请参阅：[http://timd.cn/k8s/vm-and-container/](http://timd.cn/k8s/vm-and-container/)。容器更轻量，但是有些服务必须跑在物理机/虚拟机上，比如我们可能想要安装一个用于测试的 Kubernetes 集群
    

___

Vagrant 是管理虚拟化软件的软件，其本身没有虚拟化能力。它通过调用虚拟化软件（比如 VMWare、VirtualBox）的 API，来实现虚拟化。

___

## Vagrant 中的基础概念

-   Provider： 为 Vagrant 提供虚拟化支持的具体软件，比如 VMWare、VirtualBox
-   Box： 虚拟机镜像。Vagrant 根据不同 Provider 提供很多基础镜像，用户也可以根据需求使用 `vagrant package` 制作自己的 Box
-   Project： 项目由项目目录及其下的 Vagrantfile 组成。项目可以包含子项目，子项目将继承和重写父项目的配置。项目的虚拟机实例被存储到 `~/.vagrant.d/boxes/`，而非项目目录
-   Vagrantfile： Vagrant 的配置文件，使用 Ruby 语法描述。配置文件里可以定义该项目使用的 Box、网络、共享目录、Provision 脚本等。当执行 `vagrant up` 命令时，Vagrant 会读取当前目录下的 Vagrantfile
-   Provision： 虚拟机实例启动后，需要完成的基础配置工作。Vagrant 支持使用 Shell、Puppet、Chef 来完成 Provisioning 工作
-   Plugin： 用于扩展对宿主机 OS、GuestOS、Provider、Provisioner 的支持，比如 Vagrant 通过 Plugin 实现对 AWS 和 Openstack 的支持

___

## 快速入门

### 1\. 创建项目目录

### 2\. 创建 Vagrantfile

-   当使用 `vagrant init <name>` 的形式时，Vagrant 会去本地仓库（`~/.vagrant.d/boxes/`）中寻找 Box
-   当使用 `vagrant init <user-name>/<box-name>` 的形式时，Vagrant 会去官网下载 Box：[https://app.vagrantup.com](https://app.vagrantup.com/)
-   当使用非本地仓库，并且非官网的 Box 时，可以使用 `vagrant init <box-name> <box-url>` 的形式，其中 <box\-name\> 用来指定 Box 的名字，<box\-url\> 用来指定 Box 的下载地址

`vagrant box add` 命令用于将本地或远程主机上的 Box 添加到本地仓库中。`vagrant box remove` 命令用于从本地仓库移除 Box。

### 3\. 创建和启动虚拟机实例

Vagrant 将 VirtualBox 作为默认的 Provider。可以使用 `vagrant up` 的 `--provider` 选项指定其它 Provider，比如以下命令将启动一个 hyperv 虚拟机：

启动虚拟机后，Vagrant 将在 Vagrantfile 所在的目录下创建 `.vagrant/` 子目录，该目录用于保存虚拟机的基础信息，比如 SSH 私钥、虚拟机实例的 ID 等。

### 4\. 登陆虚拟机

也可以进入 `.vagrant/` 目录，找到 SSH 私钥（`vagrant up` 命令的输出包含用户名），然后通过其它 SSH 客户端，使用私钥进行登陆

### 5\. 关闭虚拟机

### 6\. 销毁虚拟机

销毁虚拟机只删除虚拟机本身，不删除虚拟机使用的 Box。

___

## Vagrantfile 详解

### 1\. 网络配置

Vagrant 支持三种网络配置：

##### 1.1. 端口映射（forwarded port）

用于将宿主机的端口映射到虚拟机的端口上。比如通过下面的配置将宿主机的 8080 端口映射到虚拟机的 80 端口上：

##### 1.2. 私有网络（host\-only）

只有宿主机能访问虚拟机。如果多个虚拟机在同一网段，那么虚拟机之间也可以互相访问。所有虚拟机只有一个出口 \- 宿主机。

##### 1.3. 公有网络（bridge）

虚拟机和宿主机享受相同的待遇，虚拟机相当于局域网中的独立的主机。从 Vagrant 1.3 起，可以设置静态 IP：

使用桥接时，如果宿主机有多张网卡，那么执行 `vagrant up` 时，Vagrant 会询问将要桥接到哪张网卡。如果不想要这种交互，那么可以在 Vagrantfile 中进行配置：

### 2\. 在一个 Vagrantfile 中管理多台虚拟机

在 Vagrantfile 中，可以使用 `config.vm.define` 来定义多台虚拟机。下面是一个示例：

### 3\. 共享目录设置

Vagrant 借助 rsync 同步文件，所以宿主机需要安装 rysnc。

### 4\. Provision

Provision 是指在虚拟机实例启动后，通过某些工具自动地、批量地为虚拟机安装软件或进行配置。Vagrant 提供多种方式对虚拟机进行 Provisioning，包括 Shell、Puppet、Chef、Ansible 等。

以 Shell 为例，既可以直接在 Vagrantfile 中编写 Shell 脚本，也可以引用外部 Shell 文件。

直接在 Vagrantfile 中编写 Shell 脚本时，需要通过 inline 参数指定脚本内容：

其中 `run: "always"` 表示每次执行 `vagrant up` 命令时，都执行 Provision。

下面再看一个引用外部脚本的示例：

> 注意：
> 
> 1.  并非每次执行 `vagrant up` 命令时，都执行 Provision。只有在以下 3 种情况时，Provision 才被执行：
> 
> -   首次执行 `vagrant up` 命令时
> -   执行 `vagrant provision` 命令时
> -   执行 `vagrant reload --provision` 命令时

### 5\. Provider 特定配置

不同 Provider 具有不同特性，因此存在不同的配置方式。以 VirtualBox 为例，可以通过 name 选项设置虚拟机的名字，通过 cpus 选项设置虚拟机的 CPU 数量，通过 memory 选项设置虚拟机的内存：

Provider 的特定配置将覆盖原来的配置：

### 6\. 其它

-   `config.vm.box = <box-name>` 该名称是 `vagrant init` 后面跟的名字
-   `config.vm.hostname = <guest-os-hostname>` 用于设置虚拟机的主机名

___

## 创建自己的 Vagrant Box

接下来以一个示例说明如何创建 Box：

其中，hadoop\_master 是虚拟机实例的名称，hadoop\_master.box 是 Box 名。

创建 Box 时，如果虚拟机实例正在运行，那么 Vagrant 将先 halt，再 export。

用如下命令将 Box 添加到本地仓库：

通过下面的命令查看本地仓库的 Box 列表：

最后，按照下面的流程查看前面创建的 Box：

1.  创建并且切换到用于测试的项目目录：
    
2.  创建 Vagrantfile，使用前面新创建的 hadoop\_master.box 作为 Box：
    
3.  创建、启动虚拟机：
    
4.  登陆虚拟机：
    

___

## 遇到的问题

Q1：执行 `vagrant up` 命令时，出现 `mount: unknown filesystem type 'vboxsf'` 错误

A：使用 `vagrant plugin install vagrant-vbguest` 命令，安装 vbguest 插件