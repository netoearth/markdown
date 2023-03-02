### 目录

-   [🌲 前言 🌲](https://cloud.tencent.com/developer?from=10680)
-   [🌻 Vagrant 简介 🌻](https://cloud.tencent.com/developer?from=10680)
-   [❤️ 入门玩法：Vagrant 安装 ❤️](https://cloud.tencent.com/developer?from=10680)
-   [⚡️ 初阶玩法：Vagrant 常用命令 ⚡️](https://cloud.tencent.com/developer?from=10680)
    -   [1️⃣ Vagrant 基础命令](https://cloud.tencent.com/developer?from=10680)
    -   [2️⃣ Vagrant Box 管理](https://cloud.tencent.com/developer?from=10680)
    -   [3️⃣ Vagrant 虚拟机系统命令](https://cloud.tencent.com/developer?from=10680)
    -   [4️⃣ Vagrant 插件管理](https://cloud.tencent.com/developer?from=10680)
-   [🌀 进阶玩法：Vagrant 插件 🌀](https://cloud.tencent.com/developer?from=10680)
-   [☁️ 高阶玩法：Vagrant 配套工具 ☁️](https://cloud.tencent.com/developer?from=10680)
    -   [① ngrok](https://cloud.tencent.com/developer?from=10680)
    -   [② packer](https://cloud.tencent.com/developer?from=10680)
-   [💦 超神玩法：Vagrant 卸载 💦](https://cloud.tencent.com/developer?from=10680)
    -   [1️⃣ Windows系统](https://cloud.tencent.com/developer?from=10680)
    -   [2️⃣ Linux系统](https://cloud.tencent.com/developer?from=10680)
    -   [3️⃣ macOS系统](https://cloud.tencent.com/developer?from=10680)
-   [⏰ 写在最后](https://cloud.tencent.com/developer?from=10680)

## 🌲 前言 🌲

**首先下载安装** [**Vagrant**](https://www.vagrantup.com/downloads) **和** [**VirtualBox**](https://www.virtualbox.org/wiki/Downloads) **，不用知道是啥玩意，安装完软件之后，打开 `cmd` 或者 `终端` 执行以下命令即可安装操作系统，就这么简单！**

**🙉 以下列出主流系统安装方式 🙉 ：**

> 🌞 **安装Windows10 系统** 🌞

```
vagrant init galppoc/windows10
vagrant up
```

>  **安装 macOS 系统** 

```
vagrant init jhcook/macos-sierra
vagrant up
```

> 🧡 **安装 Linux/Unix 系统** 🧡

**① centos：**

```
vagrant init generic/centos7
vagrant up
```

**② Ubuntu：**

```
vagrant init generic/ubuntu2010
vagrant up
```

**③ redhat：**

```
vagrant init generic/rhel7
vagrant up
```

**④ fedora：**

```
vagrant init generic/fedora32
vagrant up
```

**⑤ oracle linux：**

```
vagrant init generic/oracle7
vagrant up
```

**看不懂？没有关系，直接输入就行！！！**

**❤️ 以上只列出常用主流操作系统，并不是仅仅这些，Vagrant 支持所有操作系统安装，只要能安装的，都可以！**

**☀️ 想要入门？想要解锁 🔓 更多玩法？看下去，带你玩转 Vagrant ！！！☀️**

## 🌻 Vagrant 简介 🌻

**Vagrant** - 一款用于管理虚拟机的命令行实用软件，用 Ruby 语言开发而成。换言说，可以省去你使用虚拟机创建操作系统的所有操作，比如创建虚拟机，挂载镜像文件，一步步点击安装；使用 Vagrant ，这些都不需要做了，简简单单 2 行命令，快速创建属于你个人的系统。

![](https://ask.qcloudimg.com/http-save/yehe-3615093/207ef7aae88c3bc319ec4105ce625b20.png?imageView2/2/w/1620)

## ❤️ 入门玩法：Vagrant 安装 ❤️

😄 Vagrant 的安装 **Very easy~** ，直接在官网下载安装包👇🏻：

> 官网下载地址：https://www.vagrantup.com/downloads

![](https://ask.qcloudimg.com/http-save/yehe-3615093/3538d5b588ca860659cc8fd98157f7c2.png?imageView2/2/w/1620)

**安装完后，方便使用，启用命令行自动补全：**

```
vagrant autocomplete install --bash --zsh
```

![](https://ask.qcloudimg.com/http-save/yehe-3615093/573e28a82e5b867280efbd80e5313263.png?imageView2/2/w/1620)

o(￣▽￣)ｄ，安装完后，重新启动终端，尝试输入部分命令 vagr ，然后直接按 Tab 键自动带出完整命令。

## ⚡️ 初阶玩法：Vagrant 常用命令 ⚡️

## 1️⃣ Vagrant 基础命令

**查看 Vagrant 帮助**

![](https://ask.qcloudimg.com/http-save/yehe-3615093/9547ea52691e2958bea3386b1b381532.png?imageView2/2/w/1620)

☺️ 显而易见，这个命令就是帮助我们了解 Vagrant 有哪些命令，以及哪些功能，如何使用。

**查看 Vagrant 版本**

![](https://ask.qcloudimg.com/http-save/yehe-3615093/411dbfb017c0d14ad3fa5dcd20fb05ea.png?imageView2/2/w/1620)

查看当前 Vagrant 版本。

**查看 Vagrant 当前所有已安装系统**

![](https://ask.qcloudimg.com/http-save/yehe-3615093/0bad1267cc69ff225c8d578898106cc3.png?imageView2/2/w/1620)

通过该命令可以查看当前系统已安装的虚拟机系统详细信息，非常方便。

## 2️⃣ Vagrant Box 管理

本节主要介绍 Vagrant Box 的基本管理命令：

![](https://ask.qcloudimg.com/http-save/yehe-3615093/41d21984422ba54ded3374430aeb45d2.png?imageView2/2/w/1620)

**查看所有已添加 box**

![](https://ask.qcloudimg.com/http-save/yehe-3615093/8294432987027b1c7cb29bdd4e140873.png?imageView2/2/w/1620)

此命令可以列出所有已添加的 box 。

**添加新的 box**

```
vagrant box add /Volumes/Lucifer/vagrant/centos79-oracle11g-vb/centos7.9 --name=centos7
```

![](https://ask.qcloudimg.com/http-save/yehe-3615093/07a36e60bac065d8a5a28e47bd694669.png?imageView2/2/w/1620)

下载 box 到本地，指定本地 box 添加，名称为centos7。

**移除已添加 box**

```
vagrant box remove centos7
```

![](https://ask.qcloudimg.com/http-save/yehe-3615093/f70cda22255189e61faf896c5c9e81e7.png?imageView2/2/w/1620)

本示例移除名为 centos7 的 box。

## 3️⃣ Vagrant 虚拟机系统命令

**初始化虚拟机系统**

```
vagrant init luciferliu/oracle11g
```

![](https://ask.qcloudimg.com/http-save/yehe-3615093/1d0dfd55e6a81ba58737c01e13567dda.png?imageView2/2/w/1620)

初始化虚拟机系统之后，会自动生成一个Vagrantfile文件，可自定义编辑，玩转 Vagrant 关键。

**校验 Vagrantfile 文件**

当你编辑好一个 Vagrantfile 之后，不确定编写是否正确，可以使用该命令进行验证是否正确。

**启动虚拟机系统**

```
vagrant up --provider=virtualbox
```

用默认虚拟机 virtualbox 启动虚拟机系统，过程中显示，如果本地没有添加过 box，vagrant up 启动后会自动下载添加。

**连接虚拟机系统**

通过该命令可以无需常规的 SSH 方式，快速连接系统，默认用户为 vagrant，密码为 vagrant。

**查看虚拟机系统状态**

显而易见，查看当前虚拟机系统的运行状态。

**重载虚拟机系统**

顾名思义，重新加载你的虚拟机系统，当你修改 Vagrantfile 之后需要生效，无需关闭虚拟机系统，执行该命令即可生效。

**关闭虚拟机系统**

执行关闭命令后，虚拟机系统将会立刻关闭。

**打包虚拟机系统**

为什么要打包系统？很简单，你可以打包后备份它，可以分享它给你的朋友，可以用来工作同步一键部署开发环境。

**删除虚拟机系统**

删除命令将会删除你的虚拟机系统，包括所有文件，全都消失无踪，慎用 ⚠️。

## 4️⃣ Vagrant 插件管理

本节主要介绍 Vagrant 插件的管理命令：

**查看已安装插件**

显示你安装的所有 Vagrant 插件。

**安装插件**

```
vagrant plugin install vagrant-share
```

**卸载插件**

```
vagrant plugin uninstall vagrant-share
```

**修复插件**

插件出现问题时，可以使用修复命令来进行修复。

**更新插件**

既然是插件，当然需要经常更新，使用更新命令可以更新你的插件。

## 🌀 进阶玩法：Vagrant 插件 🌀

既然上面讲到插件了，那就推荐一下常用插件。刚安装的 Vagrant 相当于一个没有武装的战士，想要拥有强大的战斗力，就需要用插件来全副武装。Vagrant 具有许多开箱即用的强大功能，可以让你的环境启动并运行。

> **❤️ 给大家分享下 Plugin 网站 ：https://vagrant-lists.github.io/#plugins ❤️**

👌🏻 **这里分享下我已安装的几个插件：**

-   **vagrant-parallels** (2.2.3, global) 用于 Parallels Desktop 虚拟机的支持插件
-   **vagrant-proxyconf** (2.0.10, global) 用于设置虚拟机代理
-   **vagrant-share** (2.0.0, global) 通过插件可以分享你的虚拟机环境给朋友
-   **vagrant-mutate** (1.2.0, global) 使用插件可以转换你的 box ，比如从 virtualbox 到 kvm

**☀️ 安装方式：**

```
vagrant plugin install vagrant-parallels
vagrant plugin install vagrant-proxyconf
vagrant plugin install vagrant-reload
vagrant plugin install vagrant-share
vagrant plugin install vagrant-mutate
vagrant plugin list
```

❤️ **顺便给大家分享一个解决 Vgrant 插件安装慢的方法：** ❤️

```
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l
```

没错，就是替换 RubyGems 的源，确保只有这一个源，如果失灵，就换回去，😌 慢就慢点，总比不能用强。

## ☁️ 高阶玩法：Vagrant 配套工具 ☁️

> ① ngrok ② packer

**① ngrok：**

**`ngrok` 是什么？别问(\`へ´)ノ，问就告诉你：用来做内网映射的😄。**

接下来教你怎么用 ngrok ，将你的虚拟机环境分享给全世界各地的 People 使用！！！

**第一步**：安装ngrok，官网下载安装包，解压是个可执行文件，直接双击使用 o(￣▽￣)ｄ。

> 官网下载地址：https://dashboard.ngrok.com/get-started/setup

**第二步**：需要注册一个 ngrok 用户，获取授权码，来授权本地机器。

**第三步**：测试 ngrok 是否可用

打开你的浏览器访问 http://127.0.0.1:4040，如果能打开，说明 👌🏻 啦，恭喜你 🎉 ！

**第四步**：使用 **vagrant-share** 插件，分享你的虚拟机：

这里有个小前提：需要环境变量中配置 `ngrok` ，否则 vagrant 无法获取到改命令：

```
export VG_HOME='/opt/vagrant'
export PATH=$PATH:$VG_HOME/bin
```

注意：我是将解压的 `ngrok` 可执行工具放入 `/opt/vagrant/bin` 目录下，参考如上配置即可。

-   使用 ssh 方式分享你的主机：

**服务端开启共享：**

注意：过程中需要输入两次密码，用于提供给客户端来进行登录。

如果服务端执行之后，显示上图这样，说明已经共享成功，客户端可以进行访问。

**客户端连接：**

模拟下其他朋友连接我的虚拟机环境：

```
vagrant connect --ssh orange_amigo:george_botanic@forward
```

这边输入密码后，已经连接进虚拟机环境了，我这是一套 Oracle RAC 环境的其中一个节点：

这里的 `orange_amigo:george_botanic@forward` 是自动生成的，建议根据实际情况填写。

**② packer：**

**`packer` 是什么？别问(\`へ´)ノ，问就告诉你：用来定制你的专属 Box 的😄。**

接下来教你怎么用 packer ，打造为你量身定制的虚拟机环境！！！

**第一步**：安装 packer ，官网下载安装包，解压是个可执行文件，直接双击使用 o(￣▽￣)ｄ。

> 官网下载地址：https://www.packer.io/downloads

这里有个小前提：需要环境变量中配置 `packer` ，否则 vagrant 无法获取到改命令：

```
export VG_HOME='/opt/vagrant'
export PATH=$PATH:$VG_HOME/bin
```

注意：我是将解压的 `packer` 可执行工具放入 `/opt/vagrant/bin` 目录下，参考如上配置即可。

**第二步：** 下载一个centos或者windows镜像，也就是安装包。本次以centos为例吧：

> 下载地址：https://mirrors.163.com/centos/7.9.2009/isos/x86\_64/CentOS-7-x86\_64-Minimal-2009.iso

这里我已经成功下载完成，需要进行一下 checksum 校验一下：

```
shasum -a 256 /Users/lpc/Downloads/CentOS-7-x86_64-Minimal-2009.iso
```

**顺便记录一下校验码：`07b94e6b1a0b0260b94c83d6bb76b26bf7a310dc78d7a9c7432809fb9bc6194a`**

**第三步：** 这里我们借用 `bento` 前辈写好的模板进行打包，首先通过 `git clone` 项目：

> Github 地址：https://github.com/chef/bento

```
git clone https://hub.fastgit.org/chef/bento.git
```

**第四步：** 打开刚 git 的项目，自定义 packer json 文件：

根据 `bento` 大神的 centos-7.9-x86\_64.json ，我们自定义一个新的 json 文件：

```
{
  "builders": [
    {
      "boot_command": [
        "<up><wait><tab> text ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/{{user `ks_path`}}<enter><wait>"
      ],
      "boot_wait": "5s",
      "cpus": "{{ user `cpus` }}",
      "disk_size": "{{user `disk_size`}}",
      "guest_additions_path": "VBoxGuestAdditions_{{.Version}}.iso",
      "guest_additions_url": "{{ user `guest_additions_url` }}",
      "guest_os_type": "RedHat_64",
      "hard_drive_interface": "sata",
      "headless": "{{ user `headless` }}",
      "http_directory": "{{user `http_directory`}}",
      "iso_checksum": "{{user `iso_checksum`}}",
      "iso_url": "{{user `mirror`}}/{{user `mirror_directory`}}/{{user `iso_name`}}",
      "memory": "{{ user `memory` }}",
      "output_directory": "{{ user `build_directory` }}/packer-{{user `template`}}-virtualbox",
      "shutdown_command": "echo 'vagrant' | sudo -S /sbin/halt -h -p",
      "ssh_password": "vagrant",
      "ssh_port": 22,
      "ssh_timeout": "10000s",
      "ssh_username": "vagrant",
      "type": "virtualbox-iso",
      "virtualbox_version_file": ".vbox_version",
      "vm_name": "{{ user `template` }}"
    },
    {
      "boot_command": [
        "<up><wait><tab> text ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/{{user `ks_path`}}<enter><wait>"
      ],
      "boot_wait": "5s",
      "cpus": "{{ user `cpus` }}",
      "disk_size": "{{user `disk_size`}}",
      "guest_os_type": "centos",
      "http_directory": "{{user `http_directory`}}",
      "iso_checksum": "{{user `iso_checksum`}}",
      "iso_url": "{{user `mirror`}}/{{user `mirror_directory`}}/{{user `iso_name`}}",
      "memory": "{{ user `memory` }}",
      "output_directory": "{{ user `build_directory` }}/packer-{{user `template`}}-parallels",
      "parallels_tools_flavor": "lin",
      "prlctl_version_file": ".prlctl_version",
      "shutdown_command": "echo 'vagrant' | sudo -S /sbin/halt -h -p",
      "ssh_password": "vagrant",
      "ssh_port": 22,
      "ssh_timeout": "10000s",
      "ssh_username": "vagrant",
      "type": "parallels-iso",
      "vm_name": "{{ user `template` }}"
    }
  ],
  "post-processors": [
    {
      "output": "{{ user `build_directory` }}/{{user `box_basename`}}.{{.Provider}}.box",
      "type": "vagrant"
    }
  ],
  "provisioners": [
    {
      "environment_vars": [
        "HOME_DIR=/home/vagrant",
        "http_proxy={{user `http_proxy`}}",
        "https_proxy={{user `https_proxy`}}",
        "no_proxy={{user `no_proxy`}}"
      ],
      "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E sh -eux '{{.Path}}'",
      "expect_disconnect": true,
      "scripts": [
        "{{template_dir}}/scripts/update.sh",
        "{{template_dir}}/../_common/motd.sh",
        "{{template_dir}}/../_common/sshd.sh",
        "{{template_dir}}/scripts/networking.sh",
        "{{template_dir}}/../_common/vagrant.sh",
        "{{template_dir}}/../_common/virtualbox.sh",
        "{{template_dir}}/../_common/vmware.sh",
        "{{template_dir}}/../_common/parallels.sh",
        "{{template_dir}}/scripts/cleanup.sh",
        "{{template_dir}}/../_common/minimize.sh"
      ],
      "type": "shell"
    }
  ],
  "variables": {
    "box_basename": "centos-7.9",
    "build_directory": "../../builds",
    "build_timestamp": "{{isotime \"20060102150405\"}}",
    "cpus": "2",
    "disk_size": "65536",
    "git_revision": "__unknown_git_revision__",
    "guest_additions_url": "",
    "headless": "",
    "http_directory": "{{template_dir}}/http",
    "http_proxy": "{{env `http_proxy`}}",
    "https_proxy": "{{env `https_proxy`}}",
    "iso_checksum": "07b94e6b1a0b0260b94c83d6bb76b26bf7a310dc78d7a9c7432809fb9bc6194a",
    "iso_name": "CentOS-7-x86_64-Minimal-2009.iso",
    "ks_path": "7/ks.cfg",
    "memory": "1024",
    "mirror": "",
    "mirror_directory": "Users/lpc/Downloads",
    "name": "centos-7.9",
    "no_proxy": "{{env `no_proxy`}}",
    "template": "centos-7.9-x86_64",
    "version": "TIMESTAMP"
  }
}
```

修改内容如上，读者可根据实际环境填写。

**需要注意：读者主要修改的部分为：** **1、iso的存放位置：**

**2、iso 的 checksum 结果和 iso 名称：**

修改以上两部分即可。

**第四步：** 启动 packer 进行打包：

```
cd /Volumes/DBA/vagrant/packer/packer_templates/centos
packer build -only=virtualbox-iso centos-7.9-x86_64.json
```

这里的 `-only=virtualbox-iso` 是指只创建 virtualbox 的 box 文件。

执行过程中，我们什么都不需要干，只需要安静的做个美男子，等着就完事儿了。😄

**🎥 小剧场：**

闲着也是闲着，研究了一下大神的脚本，发现有一个脚本挺有意思的，拿出来分享一下：

⭐️ 就是这个脚本，干什么的呢❓

> 简单来说：当我们安装完一个虚拟机系统之后，想要进行打包时，系统内一些多余的空间或者垃圾会占空间，导致打包出来的 box 过大，使用此脚本会清理这些垃圾和多余空间，使得打包出来的 box 非常的小，基本跟初始镜像的大小一致。❤️ 可以说是灰常 Nice 的小玩意儿了！！！爱了爱了，收藏一波。

言归正传，经过一段时间的等待后，我们的 `packer` 也跑完了：

😰 报错了，说是访问 GitHub 失败了，阿西吧，外国大神写的脚本，国内 GitHub 难受啊，想办法解决下：

**在 `networking.sh` 脚本中加入如下内容：**

```
# modify by luciferliu for github raw.githubusercontent.com:443; Connection refused
echo '185.199.108.133 raw.githubusercontent.com' >>/etc/hosts;
echo '185.199.109.133 raw.githubusercontent.com' >>/etc/hosts;
echo '185.199.110.133 raw.githubusercontent.com' >>/etc/hosts;
echo '185.199.111.133 raw.githubusercontent.com' >>/etc/hosts;

ping raw.githubusercontent.com
```

**测试一下是否可以：**

👌🏻，没有毛病啊，老铁，哈哈，忘记 linux ping不会自动停止，无限 ping 了。😓 重来吧，如果能重来，我要选一下代码！😄 ，修改为：

```
ping raw.githubusercontent.com -c 10
```

我们只 `ping` 10次哈，意思一下就行！

**再一次，事不过三啊，给爷冲！！！**

**🎉 皇天不负有心人，成了 Nice！👌🏻**

打包完之后的 box 才 300 多M，来测试一下能不能用吧：

```
vagrant box add /Volumes/DBA/vagrant/packer/bento/builds/centos-7.9.virtualbox.box --name=centos79
vagrant init centos79
vagrant up --provider=virtualbox
```

连接使用：

使用没有毛病，很好很强大，到这 `packer` 也就讲完啦。😄

> 关于高阶玩法，我就只讲了两个，一个是 `gnrok` 和 vagrant-share 插件组合使用，一个是 `packer` 打包 box。后续更多的玩法，放到后面慢慢讲，我也在慢慢的研究，慢慢玩，大家有兴趣可以 **关注博主** 一波。😄

## 💦 超神玩法：Vagrant 卸载 💦

**哈哈，都说一切事务的终点就是回到原点，那不就是卸载嘛！😄，为了给大家写 blog，我前前后后卸载安装了好多回了，也算是颇有心得。**

删除 Vagrant 程序将从您的机器中删除 vagrant 二进制文件和所有依赖项。 卸载程序后，您仍然可以使用标准方法重新安装。 卸载 Vagrant 不会删除用户数据。 此部分下面的部分提供了有关如何从系统中删除该目录的更详细说明。

## 1️⃣ Windows系统

使用控制面板的添加/删除程序部分卸载，这就不需要我教了吧，ε=(´ο｀\*))) 实现不行，`geek uninstaller` 走起。

## 2️⃣ Linux系统

```
rm -rf /opt/vagrant
rm -f /usr/bin/vagrant
rm -rf ~/.vagrant.d
```

## 3️⃣ macOS系统

```
sudo rm -rf /opt/vagrant /usr/local/bin/vagrant
sudo pkgutil --forget com.vagrant.vagrant
rm -rf ~/.vagrant.d
```

在所有平台上，此目录位于您的主目录的根目录中，并命名为 `vagrant.d`。 只需删除 **~/.vagrant.d** 目录即可删除用户数据。 如果在 Windows 上，此目录位于 C:\\Users\\YourUsername.vagrant.d，其中 YourUsername 是本地用户的用户名。

## ⏰ 写在最后

Vagrant 我也刚接触不久，如果有玩的厉害的大佬，一起交流下心得，顺便也指点指点。感觉 Vagrant 的可玩度还是挺高的，感兴趣的确实可以入手一哈。😄

**☀️ 最后的最后，分享一下 Vagrant 安装 Oracle**[**数据库**](https://cloud.tencent.com/solution/database?from=10680)**系统：**

> [**Vagrant安装Oracle系列**](https://blog.csdn.net/m0_50546016/category_11236959.html)

本文分享自作者个人站点/博客：https://luciferliu.blog.csdn.net/复制

如有侵权，请联系

cloudcommunity@tencent.com

删除。