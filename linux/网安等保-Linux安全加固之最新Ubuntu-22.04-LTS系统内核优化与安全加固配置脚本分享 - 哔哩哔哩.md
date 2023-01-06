B站关注「[**WeiyiGeek**](https://space.bilibili.com/385802642/dynamic)」[**点我，点我**](https://space.bilibili.com/385802642/dynamic) UP主

设为「**特别关注**」，每天带你在B站玩转网络安全运维、应用开发、物联网IOT学习！

希望各位看友【**关注、点赞、评论、收藏、投币**】，助力每一个梦想。

GIF

![](https://i0.hdslb.com/bfs/article/4a10568f28aa362fc3ddc7f871b07daf847767da.gif)

帅哥（靓仔）、美女，点个关注后续不迷路！

**本章目录：**

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

-   0x00 前言简述
    
-   0x01 加固实践
    

-   📖 帮助文档
    
-   🏃 脚本使用
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

作者: WeiyiGeek

首发地址：https://mp.weixin.qq.com/s/dO1bV0tfXKn4ZmqlMcUrrQ

原文地址：https://blog.weiyigeek.top/2022/8-13-683.html

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: Ubuntu 22.04 LTS 是Canonical于2022年4月21日发布的操作系统，代号为Jammy Jellyfish（果酱水母）, 其采用GNOME电源配置文件和流线型工作空间过渡，提高优化图形驱动程序上的桌面帧速率，使用新的加密算法迁移到OpenSSL v3以提高安全性,并且为内存安全的系统级编程添加了Rus。

而又由于CentOS发行版在最近几年时内将不在进行维护更新了，所以为了考虑到系统的安全性、可维护性、以及后期运维成本，我们企业内部在2020年时就已经将ubuntu作为主要的服务器系统，所以在我公司新上的业务系统基本采用debian系的发行版服务器来承载基础应用业务，而使用最多当然是Ubuntu此发行版。

现在 Ubuntu 推出了22.04 ， 想到原来每次都需要手动一台一台的进行主机安全加固以符合等保要求，所以了节约工作时间提高工作效率，有更多时间进行学习进步，则需要将将我们公司所使用的系统基线镜像进行更新迭代，编写适用于ubuntu 22.04系统的安全加固脚本，并且总结此篇文章并在文章末尾附上自动化安全加固脚本，谢谢大家支持。

此处我依据在我从前编写的 Ubuntu 20.04 系统安全加固脚本中对其根据最新的22.04版本进行更新了等保相关规定策略，以及更新适用最新版本的 Ubuntu，针对脚本进行结构调整，更加方便大家一起参参与维护，若脚本有Bug请大家发送到我的邮箱 master@weiyigeek.top。

想要获取该加固脚本的朋友可以在WX公众号【WeiyiGeek】中回复【ubuntu系统加固】即可获得，或者访问【https://weiyigeek.top/wechat.html?key=ubuntu系统加固】

描述: Ubuntu 22.04 主机系统安全加固的 `Ubuntu22.04-InitializeReinforce.sh` 脚本相关上的使用说明以及实践。

**实践使用视频演示**

![](https://i0.hdslb.com/bfs/article/card/c6500d09f093acf6778da8dd9cfa2e80a42ec8ed.png)

描述: 本工具集主要针对于 Ubuntu 22.04 、20.04 LTS 操作系统进行安全加固以及系统初始化操作。

🛠 **脚本说明:**

```
root@Ubuntu-Security:/home/ubuntu/Ubuntu# ./Ubuntu22.04-InitializeReinforce.sh
     __          __  _       _  _____           _
     \ \        / / (_)     (_)/ ____|         | |
     \ \  /\  / /__ _ _   _ _| |  __  ___  ___| | __
       \ \/  \/ / _ \ | | | | | | |_ |/ _ \/ _ \ |/ /
       \  /\  /  __/ | |_| | | |__| |  __/  __/   <
         \/  \/ \___|_|\__, |_|\_____|\___|\___|_|\_\
                      __/ |
                      |___/
======================================================================
@ Desc: Ubuntu 22.04 Security Reinforce and System initialization
@ Mail bug reports: master@weiyigeek.top or pull request (pr)
@ Author : WeiyiGeek
@ Follow me on Blog   : https://blog.weiyigeek.top/
@ Follow me on Wechat : https://weiyigeek.top/wechat.html?key=欢迎关注
@ Communication group : https://weiyigeek.top/visit.html
======================================================================

Usage: ./Ubuntu22.04-InitializeReinforce.sh [--start ] [--network] [--function] [--clear] [--version] [--help]
Option:
  --start            Start System initialization and security reinforcement.
  --network          Configure the system network and DNS resolution server.
  --function         PCall the specified shell function.
  --clear            Clear all system logs, cache and backup files.
  --version          Print version and exit.
  --help             Print help and exit.

Mail bug reports or suggestions to <master@weiyigeek.top> or pull request (pr).
current version : 1.0

WARNING: 温馨提示：使用前先请配置机器上网环境,若没有配置请在 Ubuntu22.04.conf 文件中进行网络配置.
```

⚛️**脚本函数:**

描述: 如下脚本将根据参数在 `Ubuntu22.04-InitializeReinforce.sh` 分别进行调用执行, 也可采用`--function`参数进行指定调用。

```
❯ grep -r -n  "函数名称" -A 1 *
scripts/os-base.sh:26:# 函数名称: base_hostname
scripts/os-base.sh-27-# 函数用途: 主机名称设置
--
scripts/os-base.sh:55:# 函数名称: ubuntu_mirror
scripts/os-base.sh-56-# 函数用途: ubuntu 系统主机软件仓库镜像源
--
scripts/os-base.sh:126:# 函数名称: ubuntu_software
scripts/os-base.sh-127-# 函数用途: ubuntu 系统主机内核版本升级以常规软件安装
--
scripts/os-base.sh:153:# 函数名称: base_timezone
scripts/os-base.sh-154-# 函数用途: 主机时间同步校准与时区设置
--
scripts/os-base.sh:192:# 函数名称: base_banner
scripts/os-base.sh-193-# 函数用途: 远程本地登陆主机信息展示
--
scripts/os-base.sh:345:# 函数名称: base_reboot
scripts/os-base.sh-346-# 函数用途: 是否进行重启或者关闭服务器
--
scripts/os-clean.sh:27:# 函数名称: system_clean
scripts/os-clean.sh-28-# 函数用途: 删除安全加固过程临时文件清理为基线镜像做准备
--
scripts/os-exceptions.sh:26:# 函数名称: problem_usercrond
scripts/os-exceptions.sh-27-# 函数用途: 解决普通用户定时任务无法定时执行问题
--
scripts/os-exceptions.sh:45:# 函数名称: problem_multipath
scripts/os-exceptions.sh-46-# 函数用途: 解决 ubuntu multipath add missing path 错误
--
scripts/os-network.sh:27:# 函数名称: net_config
scripts/os-network.sh-28-# 函数用途: 主机IP地址与网关设置
--
scripts/os-network.sh:70:# 函数名称: net_dns
scripts/os-network.sh-71-# 函数用途: 设置主机DNS解析服务器
--
scripts/os-optimize.sh:27:# 函数名称: optimize_kernel
scripts/os-optimize.sh-28-# 函数用途: 系统内核参数的优化配置
--
scripts/os-optimize.sh:84:# 函数名称: resources_limits
scripts/os-optimize.sh-85-# 函数用途: 系统资源文件打开句柄数优化配置
--
scripts/os-optimize.sh:115:# 函数名称: swap_partition
scripts/os-optimize.sh-116-# 函数用途: 创建系统swap分区
--
scripts/os-security.sh:27:# 函数名称: sec_usercheck
scripts/os-security.sh-28-# 函数用途: 用于锁定或者删除多余的系统账户
--
scripts/os-security.sh:65:# 函数名称: sec_userconfig
scripts/os-security.sh-66-# 函数用途: 针对拥有ssh远程登陆权限的用户进行密码口令设置。
--
scripts/os-security.sh:131:# 函数名称: sec_passpolicy
scripts/os-security.sh-132-# 函数用途: 用户密码复杂性策略设置 (密码过期周期0~90、到期前15天提示、密码长度至少12、复杂度设置至少有一个大小写、数字、特殊字符、密码三次不能一样、尝试次数为三次）
--
scripts/os-security.sh:166:# 函数名称: sec_sshdpolicy
scripts/os-security.sh-167-# 函数用途: 系统sshd服务安全策略设置
--
scripts/os-security.sh:194:# 函数名称: sec_loginpolicy
scripts/os-security.sh-195-# 函数用途: 用户登陆安全策略设置
--
scripts/os-security.sh:230:# 函数名称: sec_historypolicy
scripts/os-security.sh-231-# 函数用途: 用户终端执行的历史命令记录安全策略设置
--
scripts/os-security.sh:261:# 函数名称: sec_grubpolicy
scripts/os-security.sh-262-# 函数用途: 系统 GRUB 安全设置防止物理接触从grub菜单中修改密码
--
scripts/os-security.sh:304:# 函数名称: sec_firewallpolicy
scripts/os-security.sh-305-# 函数用途: 系统防火墙策略设置, 建议操作完成后重启计算机.
--
scripts/os-security.sh:335:# 函数名称: sec_ctrlaltdel
scripts/os-security.sh-336-# 函数用途: 禁用 ctrl+alt+del 组合键对系统重启 (必须要配置我曾入过坑)
--
scripts/os-security.sh:355:# 函数名称: sec_recyclebin
scripts/os-security.sh-356-# 函数用途: 设置文件删除回收站别名(防止误删文件)(必须要配置,我曾入过坑)
--
scripts/os-security.sh:405:# 函数名称: sec_supolicy
scripts/os-security.sh-406-# 函数用途: 切换用户日志记录和切换命令更改名称为SU(可选)
--
scripts/os-security.sh:425:# 函数名称: sec_privilegepolicy
scripts/os-security.sh-426-# 函数用途: 系统用户sudo权限与文件目录创建权限策略设置
--
scripts/os-service.sh:26:# 函数名称: svc_apport
scripts/os-service.sh-27-# 函数用途: 禁用烦人的apport错误报告
--
scripts/os-service.sh:52:# 函数名称: svc_snapd
scripts/os-service.sh-53-# 函数用途: 不使用snapd容器的环境下禁用或者卸载多余的snap软件及其服务
--
scripts/os-service.sh:75:# 函数名称: svc_cloud-init
scripts/os-service.sh-76-# 函数用途: 非云的环境下禁用或者卸载多余的cloud-init软件及其服务
--
scripts/os-service.sh:101:# 函数名称: svc_debugshell
scripts/os-service.sh-102-# 函数用途: 在系统启动时禁用debug-shell服务
--
scripts/os-software.sh:26:# 函数名称: install_chrony
scripts/os-software.sh-27-# 函数用途: 安装配置 chrony 时间同步服务器
--
scripts/os-software.sh:79:# 函数名称: install_java
scripts/os-software.sh-80-# 函数用途: 安装配置java环境
--
scripts/os-software.sh:110:## 函数名称: install_docker
scripts/os-software.sh-111-## 函数用途: 在 Ubuntu 主机上安装最新版本的Docker
--
scripts/os-software.sh:201:## 函数名称: install_cockercompose
scripts/os-software.sh-202-## 函数用途: 在 Ubuntu 主机上安装最新版本的Dockercompose
```

☕️ **配置文件:**  
描述: 在 Ubuntu22.04.conf 配置文件中定义脚本所需的安全策略以及日志、历史记录存放路径, 以模板的初始密码与防火墙配置等，其中最主要的是一定要配置好IP地址，以成功拉取软件仓库中的工具。

```
$ vim Ubuntu22.04.conf
# Show  Script Execute result (Y/N)
VAR_VERIFY_RESULT=Y

# Modify Script vertify timeout (unit s)
VAR_VERIFY_TIMEOUT=5

# Modify Script run time
VAR_RUNDATE=$(date +%Y%m%d-%s)

# Modify Path to logfile.
LOGFILE=/var/log/weiyigeek-${VAR_RUNDATE}.log

# Modify Path to Backup directory.
BACKUPDIR=/var/log/.backup/${VAR_RUNDATE}

# Modify Path to history record directory.
HISTORYDIR=/var/log/.history

# Modify su command execute log file path.
SU_LOG_FILE=${HISTORYDIR}/su.log

# Modify the hostname
VAR_HOSTNAME="Ubuntu-Security"

# Modify the IP/MASK and Gateway
VAR_IP=10.20.172.152/24
VAR_GATEWAY=10.20.172.1

# Modify the DNS server
# DNSPod: 119.29.29.29      Alidns: 223.5.5.5 223.6.6.6
# Google: 8.8.8.8 8.8.4.4   Cloudflare: 1.1.1.1 1.0.0.1
# Internal : Your intranet domain name resolution server
VAR_DNS_SERVER=("223.5.5.5" "223.6.6.6")

# Modify the SSHD server
VAR_SSHD_PORT=20221

# Modify the super user and normal user
# 建议将密码设置最小长度10（最好设置为12以上，等保要求），数字、大写字母、小写字母、特殊符号，密码包含三种及以上, 且无规律。
# 温馨提示: 下面设置的密码为初始密码，在系统登陆后会要求更改。
VAR_SUPER_USER=root
VAR_SUPER_PASS=R2022.weiyigeek.top
# normal user
VAR_USER_NAME=ubuntu
VAR_USER_PASS=U2022.weiyigeek.top
# low privilege application users
VAR_APP_USER=app
VAR_APP_PASS=A2022.weiyigeek.top

# Modify the NTP server
VAR_NTP_SERVER=( "ntp.aliyun.com" "ntp.tencent.com" "192.168.10.254")

# Modify the timezone
VAR_TIMEZONE=Asia/Shanghai

# Modify Password policy
# 默认密码最大使用为90天、过期前15天提示, 密码最小长度为12
PASS_MIN_DAYS=1
PASS_MAX_DAYS=90
PASS_WARN_AGE=15
PASS_MIN_LEN=12
# 默认加密方式为SHA512, 重试次数为3, 新密码与旧密码至少有6个字符不同, 至少包含3种密码类型，不限制密码中包含大写字母、小写字母、数字、特殊符号的最大数量，记住三次旧密码。
VAR_PASS_ENCRYPT=SHA512
VAR_PASS_RETRY=3
VAR_PASS_DIFOK=6
VAR_PASS_MINCLASS=3
VAR_PASS_UCREDIT=-1
VAR_PASS_LCREDIT=-1
VAR_PASS_DCREDIT=-1
VAR_PASS_OCREDIT=-1
VAR_PASS_REMEMBER=3

# 禁止没有主目录的用户登录
VAR_DEFAULT_HOME=no
# 删除用户时禁止同步删除用户组
VAR_USERGROUPS_ENAB=no
# 启用成功登录的日志记录
VAR_LOG_OK_LOGINS=yes

# Modify file or Dirctory privilege policy
VAR_UMASK=022

# Modify user login failed count policy
# 默认在5分钟之内登陆失败次数超过6次将锁定10分钟,终端超时10分钟
VAR_LOGIN_FAIL_COUNT=6
VAR_LOGIN_FAIL_INTERVAL=300
VAR_LOGIN_LOCK_TIME=600
VAR_LOGIN_TIMEOUT=300

# Modify history record count policy
VAR_HISTSIZE=128

# Modify firewall policy tcp or udp port .
VAR_ALLOW_PORT=("22,80,443,${VAR_SSHD_PORT}/tcp" "53/udp")
```

-   Step 1.上传到需要加固的主机服务器中，此处我上传到ubuntu用户的家目录。
    

```
OperatingSystem\Security> scp -r .\Ubuntu\ ubuntu@10.20.172.152:~
ubuntu@10.20.172.152\'s password:
Ubuntu22.04.conf                                                                      100% 2979   976.9KB/s   00:00
os-base.sh                                                                            100%   14KB   5.4MB/s   00:00
os-clean.sh                                                                           100% 2446     2.1MB/s   00:00
os-exceptions.sh                                                                      100% 2634     2.5MB/s   00:00
os-info.sh                                                                            100% 1169     1.3MB/s   00:00
os-manual.sh                                                                          100% 1860     2.0MB/s   00:00
os-network.sh                                                                         100% 3774     1.8MB/s   00:00
os-optimize.sh                                                                        100% 7752     3.7MB/s   00:00
os-security.sh                                                                        100% 23KB     5.7MB/s   00:00
os-service.sh                                                                         100% 3969     2.0MB/s   00:00
os-software.sh                                                                        100% 8007     3.3MB/s   00:00
```

-   Step 2.登陆服务器并切换到root用户, 查看 `/home/ubuntu` 目录下上传的加固版本。
    

```
$ ssh -p 22 ubuntu@10.20.172.152
ubuntu@Ubuntu-Security:~$ tree Ubuntu/
Ubuntu/
├── Readme.assets
│   ├── image-20220823143235577.png
│   └── image-20220823143354742.png
├── Readme.md
├── Ubuntu22.04-InitializeReinforce.sh
├── config
│   └── Ubuntu22.04.conf
├── example
│   └── 22.04
│       ├── 00-custom-header
│       ├── common-auth
│       ├── common-password
│       ├── issue
│       ├── issue.net
│       ├── login.defs
│       ├── profile
│       ├── resolved.conf
│       ├── sshd_config
│       └── su
└── scripts
    ├── os-base.sh
    ├── os-clean.sh
    ├── os-exceptions.sh
    ├── os-info.sh
    ├── os-manual.sh
    ├── os-network.sh
    ├── os-optimize.sh
    ├── os-security.sh
    ├── os-service.sh
    └── os-software.sh

ubuntu@Ubuntu-Security:~$ sudo -i
```

-   Step 3.切换root用户后进入 `/home/ubuntu/Ubuntu`，安全加固脚本存放目录，首先将所有的sh文件赋予可执行去那些，其次需要在 `Ubuntu22.04.conf` 中进行相应配置，最后运行`Ubuntu22.04-InitializeReinforce.sh --start`即可，最后等待系统重启即可。
    

```
cd /home/ubuntu/Ubuntu
chmod +x -R *
Ubuntu22.04-InitializeReinforce.sh  --start
```

-   Step 4.中途请根据需求输入Y/N，然后等待重启即可，在重启后请注意sshd服务端口更改为20221所以此时你需要指定ssh连接端口。
    

```
ssh -p 20221 ubuntu@10.20.172.152  # Ubuntu22.04.conf 定义的 ubuntu 初始化密码，登陆后会提示你进行更改。
su - root  # 只能有ubuntu用户切换到root用户，其它低权限以及app用户无法通过su进行用户切换
```

温馨提示: 如果执行到密码更新策略时，选择输入了(N) 否将不会更新其在`Ubuntu22.04.conf`脚本中定义的密码。  

温馨提示：脚本中默认root密码为R2022.weiyigeek.top。

温馨提示: 防火墙策略只开放了80，443，22，20221等端口。

需要的盆友，快快来实践吧，获取该加固脚本的朋友可以在WX公众号【WeiyiGeek】中回复【ubuntu系统加固】即可获得，或者访问【https://weiyigeek.top/wechat.html?key=ubuntu系统加固 】

本文至此完毕，更多技术文章，尽情期待下一章节！

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

原文地址：https://blog.weiyigeek.top/2022/8-13-683.html

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

**已发布的相关历史文章（点击即可进入）**

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

[企业运维实践-我在B站学使用kubeadm方式安装高可用的Kubernetes集群稳定的v1.23.7版](https://www.bilibili.com/read/cv18083177)

[运维实践-我在B站学源码编译Nginx利用GeoIP2模块实现显示IP所属地及处理不同地区访问](https://www.bilibili.com/read/cv18082692)

[](https://www.bilibili.com/read/cv18072582)[运维实践-我在B站学在源码编译Nginx里使用lua-nginx模块解析Lua脚本访问Redis数据库](https://www.bilibili.com/read/cv18072582)

[工作效率-我在B站学Markdown轻量级标记语言十五分钟让你快速学习语法到精通排版实践](https://www.bilibili.com/read/cv18072190)

[我在B站学习图像处理之使用EasyOCR库对行程码图片进行批量OCR文字识别](https://www.bilibili.com/read/cv16911816)

[使用NextCloud搭建私有网络云盘并支持Office文档在线预](https://www.bilibili.com/read/cv16835328)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

[9.我在B站学云原生之Kubernetes入门实践Pod模板创建、静态Pod以及Pod生命周期介绍](https://www.bilibili.com/read/cv16634321)

[](https://www.bilibili.com/read/cv16625708)[8.我在B站学云原生之Kubernetes入门实践资源清单与Namespace名称空间介绍](https://www.bilibili.com/read/cv16625708)

[7.我在B站学云原生之Kubernetes手把手教你使用二进制](https://www.bilibili.com/read/cv16625496)[方式部署K8S集群v1.23.6实践(下)](https://www.bilibili.com/read/cv16625253)

[7.我在B站学云原生之Kubernetes手把手教你使用二进制方式部署K8S集群v1.23.6实践(上)](https://www.bilibili.com/read/cv16625253)

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】或者 关注 UP主【WeiyiGeek】联系我。

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源于【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 个人博客。

**个人主页:** 【 https://weiyigeek.top 】  

**博客地址:** 【 https://blog.weiyigeek.top 】

![](https://i0.hdslb.com/bfs/article/8b134e8e411630dce6289ee89a08b3fc6edaa032.png@942w_549h_progressive.webp)

https://weiyigeek.top - Always keep a beginner's mind, don't forget the beginner's mind

专栏书写不易，如果您觉得这个专栏还不错的，请给这篇专栏 **【****点个赞、投个币、收个藏、关个注，转个发，留个言】(人间六大情)**，这将对我的肯定，谢谢！。

-   echo  "【点个赞】，动动你那粗壮的拇指或者芊芊玉手，亲！"
    
-   printf("%s", "【投个币】，万水千山总是情，投个硬币行不行，亲！")
    
-   fmt.Printf("【收个藏】，阅后即焚不吃灰，亲！")
    
-   System.out.println("【关个注】，后续浏览查看不迷路哟，亲！")
    
-   console.info("【转个发】，让更多的志同道合的朋友一起学习交流，亲！")
    
-   cout << "【留个言】，文章写得好不好、有没有错误，一定要留言哟，亲! " << endl;
    

GIF

![](https://i0.hdslb.com/bfs/article/11a629d1bc4369dc810216c5dedac871136167d7.gif@1s.webp)

谢谢，各位帅哥、美女四连支持！！这就是我的动力！

更多网络安全、系统运维、应用开发、物联网实践、网络工程、全栈文章，尽在 【https://blog.weiyigeek.top】 之中，谢谢各位看友支持！