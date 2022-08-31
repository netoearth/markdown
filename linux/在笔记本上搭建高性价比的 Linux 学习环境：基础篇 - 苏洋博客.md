本篇文章会尽可能详细的介绍如何在相对廉价的笔记本上搭建高性价比的 Linux 学习环境，让学习和工作都轻松和高效一些。尤其是针对国内网络环境下，如何快速的完成系统的安装和基础配置。

使用 Linux 的好处和必要性，我想应该不必过多赘述了，希望本文能够帮助你节约大量不必要的折腾的时间，腾出更多时间来思考、休息、以及打游戏。

## 写在前面

相信我的读者中不论是学生党还是已经工作的朋友，一定有憧憬 Linux ，但是迫于使用习惯、工作需求，而一直在使用 Windows 或 MacOS 的情况。

其中不少的朋友，应该也并不是完全没有碰过 Linux，甚至日常中可能会选择使用“多系统引导”、“虚拟机”、“远程云服务器”等方式来体验 Linux，而多过为 Linux 准备一台专用的本地的物理机，用 “裸金属 （Bare Metal）” 的方式运行系统。

既然有如此多的方式来运行 Linux，那么使用“裸金属”的方式来安装和使用 Linux，好处有哪些呢？

### 使用“裸金属”的方式运行 Linux 的好处

想知道使用“裸金属”方式安装 Linux 的好处，自然要知道其他方式的不足。

**先来聊聊犹如 “轮班制” 的“多系统引导”。** 倘若你使用切换“多系统引导”的方式来在同一台设备运行 Linux，虽然一时半刻看上去没有什么问题，系统运行互不干扰。但是随着你深度使用系统，你很快会发现想同时使用相同设备上的操作系统是不可能的事情，为了让两个系统中的数据进行共享，你也需要额外付出一些努力，并且这个方式的可靠性或许并不是那么好。

加上多个操作系统共享一块磁盘，即使磁盘再大，你也终究会遇到需要对某一个操作系统“动刀”，为其他操作系统腾空间的需求。并且，在漫长的使用过程中，操作系统的升级过程中可能包含了升级引导程序，而引导程序的变更，可能会让你的 Linux 或者其他的操作系统无法正常启动，为了修复这个问题，你不得不投入更多的精力在原本不需要投入的地方，类似这样复杂的[“折腾一通”](https://soulteary.com/2009/08/04/install-windows-7-on-the-hard-disk-and-realize-the-coexistence-of-multiple-systems.html)。

![虚拟化让多个系统和应用以“合租”模式运行](https://attachment.soulteary.com/2022/06/21/virtualization.jpg)

虚拟化让多个系统和应用以“合租”模式运行

**接下来，我们来聊聊类似“合租房共用资源模式”的“虚拟机”方案。** 使用虚拟机，确实是解决了同时使用不同操作系统的问题，而且时至今日，几乎所有的 CPU 都支持虚拟化技术（Intel-VT、AMD-V 等），设备执行虚拟化的效率并不低。但毕竟要多个系统实时共享 CPU 资源，如果你在宿主机或者虚拟机中持续使用超高 CPU 占用率的程序，设备的用户体验会变的很糟糕，你会得到一个明显卡顿的界面。

而且，虚拟机方案除了和上面“多系统引导”一样，同样存储磁盘共享之外，更会实时共享设备所有的内存空间，如果宿主系统本身就费内存，那么虚拟机便无法充分利用设备性能，如果强行给虚拟机分配了太多内存资源，宿主系统要么无法开启更多的程序，要么选择疯狂进行内存和硬盘的交换，前者损失了系统的功能性，后者如果出现过多的硬盘 IO 占用，系统的流畅体验也会打折扣，甚至极端的时候，虚拟机中的程序会因为 IO 被打满，而出现 Crash 。

**当然，以上两种方式的目的都是为了充分利用硬件资源，以及践行“省钱”最大化，在轻度资源使用率范围内，体验还是 OK 的。**

但是，工欲善其事，必先利其器。**如果我们想深入的使用 Linux，使用凑合的方式其实并不是一个好的选择，因为会在效率和体验上做出非常多的让步和牺牲。**

所以，也有一些同学就会选择使用“云主机”、“远程服务器”的方式来玩 Linux。但倘若我们计算一下成本，便会发现使用本地设备安装 Linux 会远比云端服务器的体验和性能香不少。

不妨先来算一笔账。

### 使用笔记本作为 Linux 设备，能比云服务器能省多少钱

以撰写本文所使用的设备，一台不带显卡的普通笔记本为例，一台搭载了第一代 [AMD Ryzen 7 PRO 4750](https://www.amd.com/en/products/apu/amd-ryzen-7-pro-4750u) 的 ThinkPad（21年购买 4500，今年购买二手 2800，没错，我买了两台）。

这台设备 CPU 有 8 颗核心，因为超线程的缘故，所以一共有 16 颗虚拟核心可用。CPU 时钟在 1.7 GHz～4.1 GHz 范围工作，L3 Cache 高达 8MB。为了让它能跑的更欢脱，我曾花了一千多买了两根 32GB 的内存条，以及花了几百块买了一块 1T 的国产 SSD 安了上去。设备默认有一个千兆的网口，以及支持 Wi-Fi 6。**抽象来看，这台设备有 16 个可用的 CPU vCores，能够在 4.1GHz 高频下工作，64GB 内存，1T 存储，默认支持千兆网络。** 如果是全新的设备，算上内存和硬盘 21 年的总成本在 6700 块左右，如果我们选择使用老设备替换下来的 SSD 和内存条，那么成本可以降低到 5000 块左右，当然，如果是在今年使用这个设备，即使你同样选择装满 1T SSD 和 64GB 内存，**总成本也不会超过 5000 块**。关于稳定性和个人使用经验，我在[这篇文章里有提](https://soulteary.com/2021/07/02/cheap-home-workstation-solution-part-one.html)，就不再赘述啦。

![相关设备和配件的订单记录](https://attachment.soulteary.com/2022/06/21/equipment-order.jpg)

相关设备和配件的订单记录

如果我们不选择本地设备，而是选择云服务器的话，以我曾经工作过的某国内头部云服务商为例：

![和 16c64g 云服务器做比较](https://attachment.soulteary.com/2022/06/21/compare-16c64g-server-one-year-cost.jpg)

和 16c64g 云服务器做比较

在不考虑网络成本的情况下（使用免费的 1Mbps），选择一台 16 核心 64GB 1TB硬盘的、主频相对高的云服务器（主频3.3～3.8GHz），一年的综合成本在 2 万块左右。大概是使用本地设备成本的 3 倍 ～ 4倍。

![和 16c16g 云服务器做比较（高密度合租）](https://attachment.soulteary.com/2022/06/21/compare-16c16g-server-one-year-cost2.jpg)

和 16c16g 云服务器做比较（高密度合租）

如果购买资源远小于本地物理机的 16 核心 16GB 的普通、低主频云服务器，那么一台服务器一年成本则要 1万1千块起步。差不多接近我们本地设备成本的两倍。

![和 16c16g 云服务器做比较（高密度合租）](https://attachment.soulteary.com/2022/06/21/compare-16c16g-server-one-year-cost.jpg)

和 16c16g 云服务器做比较（高密度合租）

即使选择购买资源远小于本地物理机的 16 核心 16GB 的“突发性能实例”（不能长时间满载使用），依旧在不考虑网络成本的情况下（使用免费的 1Mbps），一年的费用也要 6300 块。和上文中本地设备最贵的成本计算方式也是差不多的。

**所以，在个人学习场景下，如果不是需要几十核心，上百 GB 的模型训练、数据并行处理场景，使用云服务器是不如使用本地设备划算的。**

并且，因为是本地设备，不论是直接用该设备的键盘进行交互，还是在局域网中使用 SSH 连接设备使用，都能够获得忽略不计的延时，这也是远程服务器带来不了的体验优势。

好啦，聊了这么多的 Linux 在本地设备安装的好处，回归正题，哪一种 Linux 发行版适合我们做为学习环境呢？

### 关于 Linux 发行版的选择

众所周知，Linux 有许多不同的发行版，Ubuntu、Debian、Rocky、Mint、RHEL、SUSE、Fedora、Arch、Gentoo 等，那么我们选择哪一个发行版会更省心呢？

**显然是选择使用人数最多的那个。**

毋庸置疑，Ubuntu 是当今世界上[最流行的 Linux 发行版](https://w3techs.com/technologies/details/os-ubuntu)，不论是 [IoT 领域](https://ubuntu.com/blog/internet-of-things)、[Kubernetes 和容器相关领域](https://ubuntu.com/kubernetes)、[私有云 OpenStack 领域](https://ubuntu.com/openstack)，Ubuntu 都有大量的社区积累，甚至在一些公有云和虚拟化场景下，还有[专用的 HWE 内核](https://ubuntu.com/kernel/variants)提供。尤其是 Ubuntu 有着极其稳定的 LTS 发布机制，能够在开源的形式下提供非常稳定的“承诺”。

![关于 Ubuntu LTS 和各版本生命周期](https://attachment.soulteary.com/2022/06/21/ubuntu-lts.jpg)

关于 Ubuntu LTS 和各版本生命周期

本篇文章选择的操作系统是 Ubuntu 22.04，一个让我们在 80% 场景下都能偷懒的选择。

接下来，让我们来一起看看如何在笔记本上搭建 Linux 学习环境。

## 安装 Ubuntu 22.04

安装 Ubuntu 22.04 一般分三步：下载镜像，制作启动盘，安装系统。

当然，如果你已经是 Ubuntu 的用户，但是软件还未升级到 Ubuntu 22.04 ，可以参考我4月份写的一篇文章[《抢先体验 Ubuntu 22.04 Jammy Jellyfish》](https://soulteary.com/2022/04/10/early-access-to-ubuntu-22-04-jammy-jellyfish.html)，来完成系统的升级，并跳过这个小节关于安装的部分。

### 下载合适的 Ubuntu 镜像

首先访问 Ubuntu 官方网站，下载所需要的系统版本：

-   桌面版：[https://releases.ubuntu.com/22.04/ubuntu-22.04-desktop-amd64.iso](https://releases.ubuntu.com/22.04/ubuntu-22.04-desktop-amd64.iso)
-   服务器版：[https://releases.ubuntu.com/22.04/ubuntu-22.04-live-server-amd64.iso](https://releases.ubuntu.com/22.04/ubuntu-22.04-live-server-amd64.iso)

一般情况下，我会选择服务器版本，但是在笔记本设备上，我通畅会选择桌面版操作系统。最核心的原因是，**能够更好的控制屏幕休眠，支持我们在不合上笔记本屏幕的时候，获得更好的体验**：让我们既享受良好散热、高性能模式的 CPU，还能附带一个可以直接用于调试的屏幕，同时，桌面版的操作系统，也能够自动的安装完毕所有笔记本必要的硬件驱动（包含非开源的厂商驱动）。

经过多次测试，目前 Ubuntu 22.04 针对我手头的 AMD Zen2 （4750u）和 AMD Zen3（5800H）都有着非常好的支持。以及最近官方应该是增加了 Snap 的 CDN，在安装 Snap 软件的时候，速度也有了极大的提升。

### 使用 Balena Etcher 制作系统安装盘

![访问 Balena Etcher 官网，下载软件](https://attachment.soulteary.com/2022/06/21/balena-etcher-hp.jpg)

访问 Balena Etcher 官网，下载软件

制作安装盘，使用的工具依旧是在以往文章中，我多次推荐的 Balena Etcher，你可以从它的官方网站[下载到它](https://www.balena.io/etcher/)。

![使用 Balena Etcher 制作安装盘](https://attachment.soulteary.com/2022/06/21/balena-etcher.jpg)

使用 Balena Etcher 制作安装盘

下载完毕软件之后，打开软件，选择我们下载好的系统镜像，以及要制作成安装盘的 U 盘，点击“制作”按钮，稍等片刻，安装盘就制作完成啦。

### 进行操作系统安装

![安装 Ubuntu 步骤 1-3](https://attachment.soulteary.com/2022/06/21/install-ubuntu-step1-3.jpg)

安装 Ubuntu 步骤 1-3

将引导盘插到要安装系统的设备上，然后使用引导 U 盘来启动系统，可以看到熟悉的 Ubuntu Logo ，等待安装程序加载完毕之后，我们就能够看到安装引导界面了。选择 “Install Ubuntu”，一路下一步，在安装系统的过程中，会看到是否要安装额外驱动的选项，如果你拿不准，可以一并钩上，节约未来折腾的时间。

![安装 Ubuntu 步骤 4-6](https://attachment.soulteary.com/2022/06/21/install-ubuntu-step4-6.jpg)

安装 Ubuntu 步骤 4-6

当基础信息都填写完毕之后，安装程序会询问我们是否要保留硬盘原始数据和操作系统，因为我的设备只运行 Ubuntu，所以我这里选择完全删除系统。在一路 Next 之后，就是自动安装过程了，根据网络的不同，安装时间会有几分钟到十几分钟到差别。在一切就绪之后，重启系统，我们就能够看到久违的系统界面啦。

## 进行系统基础配置

当安装完毕操作系统后，建议你第一时间打开终端，然后进行系统更新，安装软件日常更新补丁和系统安全补丁。

```
sudo apt update && sudo apt -y upgrade
```

当然，如果你觉得你下载或者更新软件的速度有些慢，可以考虑执行命令替换软件所使用的软件下载源头，我个人比较喜欢替换云主机之外场景的源为“清华源”，然后再执行上面的命令：

```
sudo sed -i -e "s/cn.archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list
sudo sed -i -e "s/security.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list
```

等待软件和系统补丁更新完毕之后，执行重启操作，让补丁生效即可（首次更新，会更新内核）。

### 安装 OpenSSH Server

不论是选择桌面版操作系统，还是选择服务端操作系统，默认情况下系统中不会包含 `openssh-server` 这个组件，如果你和我一样，有从局域网其他设备访问这台 Linux 设备的需求，可以先执行下面的命令，来安装它。

```
sudo apt update && sudo apt install -y openssh-server
```

当程序安装完毕之后，我们执行 `ssh username@host-ip` 就可以直接访问到这台 Linux “服务器”啦。如果你要登录 Linux 使用的设备的用户名和 Linux 允许登录的用户名一致，那么可以省略 “username”。

```
ssh 10.11.12.240

The authenticity of host '10.11.12.240 (10.11.12.240)' can't be established.
ED25519 key fingerprint is SHA256:cYodQ6Chywyna1JbHWfA7XAFonHKAz48cPmjRyVOCFU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.11.12.240' (ED25519) to the list of known hosts.
soulteary@10.11.12.240's password: 
```

首次登录的时候，我们需要先输入 `yes` 让当前的设备信任目标设备的指纹，然后在输入密码，就能够看到熟悉的终端提示信息了：

```
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-25-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

189 updates can be applied immediately.
73 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```

### 配置 SSH 免密码登录

作为一个懒人，我们不会希望每次 `ssh` 连接这台设备，都需要输入密码吧（尤其是你的密码可能比较复杂的时候）：

```
ssh 10.11.12.240
soulteary@10.11.12.240's password: 
```

所以，我们可以使用 `ssh-copy-id` 命令，将我们的登录密钥绑定到目标设备中：

```
ssh-copy-id -i ~/.ssh/keys/2022 10.11.12.240  
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/soulteary/.ssh/keys/2022.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
soulteary@10.11.12.240's password: 

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh '10.11.12.240'"
and check to make sure that only the key(s) you wanted were added.
```

在绑定的过程中，会要求我们输入一次密码，正确输入密码之后，能够看到密钥就被成功绑定到设备上了，与此同时，我们也就登录进了操作系统。

使用 `CTRL+D` 登出系统，然后再次执行 `ssh 10.11.12.240` ，我们就可以不需要输入密码，就能够登录这台设备啦。（仅限我们当前使用的机器）

```
ssh 10.11.12.240
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-39-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 updates can be applied immediately.

Last login: Mon Jun 20 18:02:09 2022 from 10.11.12.90
```

### 配置更简单的主机名

虽然我们可以不再输入密码，就能够登录设备，但是 `10.11.12.240` 这串地址未免也太长了些。为了更简单的登录这台主机，我们可以在要发起登录的设备的 `ssh config` 中配置一个短小精悍的名字，比如：`xiaohei`。（名字随便起，甚至你可以起一个域名叫做 `oa.com / alibaba-inc.com`，随你开心）

使用 `vi` 或者你喜欢的编辑器，打开 `~/.ssh/config` ，然后在其中添加类似下面的配置内容：

```
Host xiaohei
  User            soulteary
  Hostname        10.11.12.240
  Port            22
  IdentityFile    ~/.ssh/keys/2022
  ControlPath     ~/.ssh/home-xh-%r@%h:%p
  ControlPersist  yes
  TCPKeepAlive    yes
  Compression     yes
  ForwardAgent    yes
```

保存内容之后，我们就可以使用 `ssh xiaohei` 来登录设备了。

### 配置执行特权命令免输入密码

当我们 `ssh` 到设备中，或者打开设备的终端后，想要执行软件安全或者获取更新的软件信息的时候，会发现一段时间内使用 `sudo` 命令，会触发输入密码的条件限制：

```
sudo apt update
[sudo] password for soulteary: 
```

想要解决这个问题，可以通过修改系统的 `sudoers` 文件：

```
sudo echo "`whoami` ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

不过更推荐的方式是，在 `sudoers.d` 目录中，创建一个新的用户规则，而不是调整可能随着系统升级，而非覆盖或者恢复默认值的系统文件：

```
echo "`whoami` ALL=(ALL) NOPASSWD:ALL" | sudo tee "/etc/sudoers.d/dont-prompt-$USER-for-sudo-password"
```

执行完毕命令之后，我们使用 `CTRL+D` 登出会话，然后再次执行 `ssh xiaohei` 登录设备，接着随便执行任何需要 `sudo` “加持”的命令，会发现命令就都可以在不输入密码的情况下顺畅执行啦。

### 快速安装和最简化配置 ZSH

关于日常使用的 SHELL，我推荐使用时间节约大师 `ZSH` 和开源软件 `OH-MY-ZSH`，关于它的神奇之处，网上有很多介绍，我在此也就不再赘述了。

但是在安装 `OH-MY-ZSH` 之前，我们首先需要安装 `zsh` 和 `git`：

```
sudo apt install -y zsh git
```

在 `OH-MY-ZSH` 官网中，软件的[安装文档非常简单](https://ohmyz.sh/#install)：

```
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

如果你的网络不能够直接下载安装 ZSH，可以考虑使用[“清华源”中的 “OH-MY-ZSH”](https://mirrors.tuna.tsinghua.edu.cn/help/ohmyzsh.git/) 来解决问题。

```
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh ohmyzsh/tools/install.sh
rm -rf ohmyzsh
```

安装过程中，软件会询问你是否要将 `zsh` 设置为默认 “SHELL”，大家可以根据自己情况来选择。在完成 `OH-MY-ZSH` 的安装之后，就可以参考网上各种攻略进一步进行定制化，来提升效率和终端的颜值啦。

这里，我推荐一款自动补全类型的提示插件，相比较完全的自动补全，这款插件能够在你敲已输入的命令的时候，用提示的方式告诉你，你曾经输入过的命令是什么：

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

在完成插件下载之后，我们需要修改 `~/.zshrc` 将配置文件中的 `plugins=(git)` 替换为下面的内容：

```
plugins=(
git
zsh-autosuggestions
)
```

依旧是使用 `CTRL+D`终结会话，然后重新 `ssh` 到设备上。当我们随便敲一个命令之后，就能够看到命令行中的提示补全啦。

![拥有命令行补全提示的 ZSH](https://attachment.soulteary.com/2022/06/21/zsh-tips.jpg)

拥有命令行补全提示的 ZSH

### 补充安装一些常用工具

为了能够敲更少的命令，来获取更多的信息，有几个不错的调试工具非常值得我们安装。

![能够在一个界面内尽可能多的看到系统基本状况](https://attachment.soulteary.com/2022/06/21/glances.jpg)

能够在一个界面内尽可能多的看到系统基本状况

第一个工具，是来自一位法国开发者的系统监控工具 [glances](https://github.com/nicolargo/glances) ，能够在一个简洁的界面中查看到当前系统的负载、网络、IO 读写、硬件温度、进程详情等信息。

```
sudo apt install -y glances
```

如果你追求更小巧一些的工具，传统的老牌神器 [htop](https://htop.dev/) 自然是不遑多让的，你甚至可以在 htop 中快速的设置每一个进程所能使用的资源最大值，来避免某个进程过分自私，用完整台设备的资源。

然而因为 htop 只专注于传统 “top” 命令范畴相关的功能，所以网络和存储相关，我们想偷懒，就还得安装下面几个工具：

```
sudo apt install -y iftop iotop bmon dstat
```

由于我们免不了在服务器上直接使用 `vi` 之类的命令来修修改改，所以这里为了省事，建议安装 `vim` 来替代 `vi`；当前，因为我们可能会处理不少 `JSON` 相关的数据，安装 `jq` 命令，也是必要的：

```
sudo apt install -y vim jq
```

当然，在设备上运行一些需要保持后台运行的程序也是免不了的，这时我们可以考虑安装并使用老牌的 `tmux` 或者 `screen`：

```
sudo apt install -y tmux screen
```

## 更简单的 Docker 安装

在 Ubuntu 22.04 中，虽然 Docker 的安装总体流程变化不大，但是在 [Docker 官方文档中](https://docs.docker.com/engine/install/ubuntu/)，能够看到安装过程得到了进一步简化。

第一步，清理老版本的软件。

```
sudo apt remove -y docker docker-engine docker.io containerd runc
```

第二步，安装所需工具依赖。

```
sudo apt install -y ca-certificates curl gnupg lsb-release
```

第三步，下载软件包签名使用的 GPG 密钥，并配置系统信任该密钥。

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

如果你无法访问官方地址，可以将密钥下载地址替换为下面的地址。

```
# 清华源
https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg
# 阿里云
https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg
```

第四步，创建一个适合于当前 CPU 架构和系统版本的软件源。

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

同样的，如果你希望能够更快的下载到软件，可以配置软件源来替换官方地址。

```
# 清华源
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 阿里云
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

最后一步，就是安装 Docker 的社区版，以及我们常用的 CLI 命令啦（包括 `compose`）。

```
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 允许当前用户使用 Docker

默认情况下，我们使用非 `docker` 用户执行 `docker` 命令，会得到类似下面的错误提示。

```
docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.8.2-docker)
  compose: Docker Compose (Docker Inc., v2.6.0)
  scan: Docker Scan (Docker Inc., v0.17.0)

Server:
ERROR: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/info": dial unix /var/run/docker.sock: connect: permission denied
errors pretty printing info
```

解决这个问题的方案也很简单，将我们当前的用户加入 `docker` 用户组即可：

```
sudo gpasswd -a ${USER} docker
```

执行完毕命令，我们将会看到类似 `Adding user soulteary to group docker` 的提示信息。然后我们查看 `docker` 的用户组中是否包含我们的用户：

```
sudo cat /etc/group | grep docker
```

当命令执行完毕之后，我们将能够看到类似下面的信息：

在确认用户已经添加到 `docker` 用户组中之后，接着依旧是需要执行 `CTRL+D` 登出会话，并使用 `ssh` 重新登录系统。

在重新登录系统之后，再次执行相同的命令，可以看到 `docker` 命令就能够被正确的执行啦。

```
docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.8.2-docker)
  compose: Docker Compose (Docker Inc., v2.6.0)
  scan: Docker Scan (Docker Inc., v0.17.0)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.17
 Storage Driver: zfs
  Zpool: rpool
  Zpool Health: ONLINE
  Parent Dataset: rpool/ROOT/ubuntu_i5u6fr/var/lib
  Space Used By Parent: 2079424512
  Space Available: 951228137472
  Parent Quota: no
  Compression: lz4
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
 runc version: v1.1.2-0-ga916309
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
  cgroupns
 Kernel Version: 5.15.0-39-generic
 Operating System: Ubuntu 22.04 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 16
 Total Memory: 62.18GiB
 Name: soulteary-ThinkPad-L14-Gen-1-B
 ID: TMPL:U72U:QC6V:E3SV:G4AO:MF4S:O6HL:UU7G:5II5:33RV:MWBG:BTVW
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

### 为 Docker 配置镜像 Mirror

有的时候，我们拉取 docker 官方镜像会十分的慢，甚至会卡顿中断，这个时候，可以考虑为 Docker 配置一些容器镜像。

执行下面的命令，将会创建一个位于`/etc/docker/daemon.json` 的文件，并包含一些通用的容器镜像。

```
echo '{"registry-mirrors": [ 
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn/",
    "https://zrb2xl6u.mirror.aliyuncs.com"
  ]}' | jq | sudo tee /etc/docker/daemon.json
```

接着执行 `cat /etc/docker/daemon.json` ，能够看到配置中添加了上面的镜像地址，并且内容是被格式化过的。

执行下面的命令，重启服务：

```
sudo systemctl restart docker.service
```

然后使用 `docker info` 确认配置是否生效即可：

```
...
 Registry Mirrors:
  https://registry.docker-cn.com/
  http://hub-mirror.c.163.com/
  https://docker.mirrors.ustc.edu.cn/
 Live Restore Enabled: false
...
```

### 为 Docker 配置固定的子网地址

或许你也经常会使用 Docker 创建一些带有自定义网络接口的容器。为了让容器分配 IP 地址范围更确定，从而能够减少一些不必要的网络转发时因为 IP 冲突导致的网络不通的问题，我们需要指定容器子网地址。

假设当前系统中的 `/etc/docker/daemon.json` 文件不为空，我们不能够像配置容器镜像一般，简单的将内容覆盖过去，也不能够使用简单的追加操作符，来向文件末尾添加内容，除了手动修改之外，我们该怎么办呢？

其实解决这个问题的方法也很简单，我们可以通过下面的脚本，来完成“先读取原始配置内容”并序列化成 JSON，然后“将原始配置内容和我们要追加的网络内容进行合并”，最后“将修改后的内容保存至原来文件位置”：

```
( echo $(cat <<< $( jq '. + {"default-address-pools": [ 
    {
      "base": "172.20.0.0/16",
      "size": 24
    },
    {
      "base": "172.30.0.0/16",
      "size": 24
    },
    {
      "base": "172.40.0.0/16",
      "size": 24
    }
  ]}' /etc/docker/daemon.json )) | jq | sudo tee /etc/docker/daemon2.json ) && sudo mv /etc/docker/daemon2.json /etc/docker/daemon.json
```

当命令执行完毕之后，我们同样使用 `cat /etc/docker/daemon.json` 来检查配置文件内容：

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn/",
  ],
  "default-address-pools": [
    {
      "base": "172.20.0.0/16",
      "size": 24
    },
    {
      "base": "172.30.0.0/16",
      "size": 24
    },
    {
      "base": "172.40.0.0/16",
      "size": 24
    }
  ]
}
```

可以看到，上文中仅包含“加速镜像”的配置中，已经成功追加了我们想要配置的网络相关的内容。重启服务：

```
sudo systemctl restart docker.service
```

再次使用 `docker info` 确认配置是否生效：

```
...
 Default Address Pools:
   Base: 172.20.0.0/16, Size: 24
   Base: 172.30.0.0/16, Size: 24
   Base: 172.40.0.0/16, Size: 24
```

### 加速 Docker 容器或局域网内裸金属 Ubuntu / Debian 软件下载

在上一篇文章[《轻量小巧的零配置 APT 加速工具：APT Proxy》](https://soulteary.com/2022/06/14/lightweight-and-small-zero-configuration-apt-acceleration-tool-apt-proxy.html)中，我提到了我做了一个简单的小工具，用来加速需要重复下载的 APT 软件包。

在未来学习 Linux 的过程中，或者我们折腾 CI/CD 的过程中，你会发现我们经常需要不停的构建镜像，出于有的镜像 Dockerfile 追求最少指令的原因，在构建镜像中，会不停的从互联网拉取 APT 软件包。而从互联网下载软件包，其实是非常慢的一件事。

使用 APT Proxy 这个小工具，则可以自动的为我们的容器镜像或者局域网的裸金属服务器，选择相对我们网络而言最快的网络镜像，并将从镜像下载的内存进行缓存。

![使用 APT Proxy 缓存的数据](https://attachment.soulteary.com/2022/06/14/apt-proxy-cache-stats.jpg)

使用 APT Proxy 缓存的数据

当我们需要重复下载数据的时候，软件便会将缓存递给我们，从而大幅减少 `apt update`、`apt-get install` 等命令所需要的时间。

使用这个小工具其实非常简单，在我们的设备中寻找一个合适存放服务配置的目录，然后创建一个名为 `docker-compose.yml` 的文件：

```
version: "3"
services:

  apt-proxy:
    image: soulteary/apt-proxy
    restart: always
    # 分别指定 Ubuntu / Debian 系统的软件源为清华源
    command: --ubuntu=cn:tsinghua --debian=cn:tsinghua
    environment:
      - TZ=Asia/Shanghai
    ports:
      - "3142:3142"
```

当文件创建完毕之后，先使用 `docker compose up -d` 将程序使用容器的方式启动起来。

```
docker compose up -d       
[+] Running 3/3
 ⠿ apt-proxy Pulled                                                                                 10.5s
   ⠿ 824a2625c421 Pull complete                                                                      2.8s
   ⠿ f1d062c4abcc Pull complete                                                                      4.9s
[+] Running 2/2
 ⠿ Network apt-proxy_default        Created                                                          0.0s
 ⠿ Container apt-proxy-apt-proxy-1  Started                                                          0.5s
```

接下来，只要在我们需要使用缓存服务的容器或者局域网的主机里，使用类似下面的命令来执行 `apt update` 和 `apt install` 就能够愉悦的享受快速安装的便利啦：

```
http_proxy=http://host.docker.internal:3142 apt-get -o pkgProblemResolver=true -o Acquire::http=true update && \
http_proxy=http://host.docker.internal:3142 apt-get -o pkgProblemResolver=true -o Acquire::http=true install vim -y
```

完整的例子，可以[参考这里](https://soulteary.com/2022/06/14/lightweight-and-small-zero-configuration-apt-acceleration-tool-apt-proxy.html#%E4%BD%BF%E7%94%A8-apt-proxy-%E6%9D%A5%E5%8A%A0%E9%80%9F-apt-%E6%93%8D%E4%BD%9C)。如果你想了解更多的细节，可以访问 [APT Proxy 的 GitHub 开源仓库](https://github.com/soulteary/apt-proxy)。

## 最后

原本计划一篇文章讲完 Ubuntu 操作系统的安装和配置，Docker 的安装和配置，几种不同的 K8S “发行版”的安装和配置，以及当今世界上流行的编程语言的环境配置。

碍于篇幅太长不好发布，于是只好在接下来的几篇内容中，陆续将“基础知识”讲完啦。希望上面的内容，对于正在学习 Linux 或者准备进入 Linux 世界的你有帮助。

–EOF