WARP是CloudFlare提供的一项基于WireGuard的网络流量安全及加速服务，能够让你通过连接到CloudFlare的边缘节点实现隐私保护及链路优化。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-04_21-05-50.jpg)

其连接入口为双栈（IPv4/IPv6均可），且连接后能够获取到由CF提供基于NAT的IPv4和IPv6地址，因此我们的单栈服务器可以尝试连接到WARP来获取额外的网络连通性支持。这样我们就可以让仅具有IPv6的服务器访问IPv4，也能让仅具有IPv4的服务器获得IPv6的访问能力。

> 题图来源微博[@Ratto](https://weibo.com/u/5042576866)，本站仅做引用，如果喜欢请务必关注画师太太啊≧ ﹏ ≦！

___

## 零、前言

在这之前我就写过利用WARP为流量计费教育网节省流量开销的文章，后来听有人说可以安装在服务器上来为服务器增加连通性支持。想了一下也确实是这个道理，毕竟双栈任选入口、连接后可提供双栈网络，而且CF本身接驳有主流的上游，不论是从打通v4还是打通v6的角度，其各方面稳定性都是其他方案无法比拟的。

作为测试，这一次为IPv4实例打通IPv6网络选择的是腾讯云香港区域的轻量应用服务器，为IPv6服务器打通IPv4网络则使用的是Scaleway的`Stardust`实例。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-06_09-34-49.jpg)

___

## 一、WGCF配置

首先我们需要通过WGCF（[点击前往](https://github.com/ViRb3/wgcf)）注册WARP账户并提取为WG配置文件。在这之前我写过一次有关教育网利用其IPv6连接以减少计费流量开销的文章。

#### 【WGCF】提取WARP配置为CERNET提供IPv6流量转发

校园网的IPv4收费不少高校都比较贵，而IPv6是不限速不限量的。经过2020年9月10日教育网的路由变动后，CERNET在HKIX与HE以Peer via IX的形式进行了互联。因 ...

[https://luotianyi.vc/4500.html](https://luotianyi.vc/4500.html)

文中第【一】、【二】两节即为此步骤的详细操作，最终目标是获取到如图所示的`wgcf-profile.conf`。最近不知道什么原因经常出现429报错，在`wgcf register`执行之后看到目录里生成`wgcf-account.toml`即可，然后多次执行`wgcf generate`直至正常生成即可。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-04_21-16-04.png)

___

## 二、处理配置文件

WG连接后是内核层级的软件，会建立自己的虚拟网卡，且WARP客户端均为内网NAT地址，当双栈流量均被WG接管后我们就无法再从原有的IP连接到服务器了。因此在IPv4与IPv6之间我们必须做一个取舍，以防这样的情况发生。所以这个应用场景也就被限制到了在原有基础上提供另一种协议的连通性支持，后文会以配图以解释。

修改配置文件就两种情况：

> ①上文图中`11`行Endpoint修改为`162.159.192.1:2408`，删除掉第`9`行接管本地IPv4路由的配置  
> ②上文图中`11`行Endpoint修改为`[2606:4700:d0::a29f:c001]:2408`，删除掉第`10`行接管本地IPv6路由的配置

### 为仅IPv4服务器添加IPv6

原理如图，由于`AllowedIPs = ::/0`的参数使得IPv6的流量均被WARP网卡接管，实现了让IPv6的流量通过WARP访问外部网络。这样的需求请参考情况①对配置文件进行修改。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-04_21-45-44.png)

### 为仅IPv6服务器添加IPv4

原理如图，由于`AllowedIPs = 0.0.0.0/0`的参数使得IPv4的流量均被WARP网卡接管，实现了让IPv4的流量通过WARP访问外部网络。这样的需求请参考情况②对配置文件进行修改。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-04_21-45-45.png)

### 双栈服务器置换网络

有时我们的服务器本身就是双栈的，但是由于种种原因我们可能并不想使用其中的某一种网络，这时也可以通过WARP接管其中的一部分网络连接隐藏自己的IP地址。至于这样做的目的，最大的意义是减少一些滥用严重机房出现验证码的概率；同时部分内容提供商将WARP的落地IP视为真实用户的原生IP对待，能够解除一些基于IP识别的封锁。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-04_21-45-45-1.png)

___

## 三、安装客户端

WireGuard是内核级别的工具，来自官方的包需要加载内核模块，所以在安装前请保证你的服务器是KVM/HyperV/XEN HVM这样完全虚拟化的服务器。

OpenVZ和LXC因为不具有内核权限，如果确有需要自己装WireGuard-Go作为内核模块的替代，其提供的文档要求自己配置go环境进行编译，并没有现成的二进制预编译包。当然安装后的配置操作都是相差无几的，在这因为并不推荐就补充在最后面好了。

博主在这些”玩具”机器上使用的均是Debian系统，所以作为一篇记录我就只考虑Ubuntu和Debian系统了~

安装前先确定`kernel-header`已安装，并且检查下`resolv`是正确安装上的：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>apt</span>-<span>get</span><span> </span><span>install </span><span>sudo </span><span>net</span>-<span>tools </span><span>openresolv</span><span> </span>-<span>y</span></p></div></td></tr></tbody></table>

安装主程序，Debian需要添加unstable源，Ubuntu则只需要添加库即可：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#Debian添加unstable源</span></p><p><span>echo</span><span> </span><span>"deb http://deb.debian.org/debian/ unstable main"</span><span> </span><span>&gt;</span><span> </span>/<span>etc</span>/<span>apt</span>/<span>sources</span><span>.</span><span>list</span><span>.</span><span>d</span>/<span>unstable</span>-<span>wireguard</span><span>.</span><span>list</span></p><p><span>printf</span><span> </span><span>'Package: *\nPin: release a=unstable\nPin-Priority: 150\n'</span><span> </span><span>&gt;</span><span> </span>/<span>etc</span>/<span>apt</span>/<span>preferences</span><span>.</span><span>d</span>/<span>limit</span>-<span>unstable</span></p><p><span>#更新源并安装</span></p><p><span>apt</span>-<span>get </span><span>update</span></p><p><span>apt</span>-<span>get </span><span>install </span><span>wireguard</span>-<span>dkms </span><span>wireguard</span>-<span>tools</span></p><p><span>#Ubuntu添加库</span></p><p><span>add</span>-<span>apt</span>-<span>repository </span><span>ppa</span><span>:</span><span>wireguard</span>/<span>wireguard</span></p><p><span>#更新源并安装</span></p><p><span>apt</span>-<span>get </span><span>update</span></p><p><span>apt</span>-<span>get </span><span>install </span><span>wireguard</span></p></div></td></tr></tbody></table>

经过群友的补充，CentOS 7因为内核比较老不能支持，需要安装添加`elrepo`源并更换`kernel-ml`内核。个人不推荐使用CentOS进行安装，7得换内核，8还成了弃子，有点麻烦。

<table><tbody><tr><td data-settings="show"><div><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p></div></td><td><div><p><span>#安装elrepo源</span></p><p><span>rpm</span><span> </span>-<span>import </span><span>https</span><span>:</span>//<span>www</span><span>.elrepo</span><span>.org</span>/<span>RPM</span>-<span>GPG</span>-<span>KEY</span>-<span>elrepo</span><span>.org</span></p><p><span>rpm</span><span> </span>-<span>Uvh </span><span>https</span><span>:</span>//<span>www</span><span>.elrepo</span><span>.org</span>/<span>elrepo</span>-<span>release</span>-<span>7.0</span>-<span>4.el7.elrepo.noarch.rpm</span></p><p><span>#安装额外包组件</span></p><p><span>yum </span><span>install</span><span> </span>-<span>y</span><span> </span><span>epel</span>-<span>release </span><span>elrepo</span>-<span>release </span><span>yum</span>-<span>plugin</span>-<span>elrepo</span><span></span></p><p><span>#安装内核</span></p><p><span>yum</span><span> </span>--<span>enablerepo</span>=<span>elrepo</span>-<span>kernel</span><span> </span>-<span>y</span><span> </span><span>install </span><span>kernel</span>-<span>ml </span><span>kernel</span>-<span>ml</span>-<span>headers&nbsp;&nbsp;</span><span>kernel</span>-<span>ml</span>-<span>devel</span></p><p><span>#查看启动顺序</span></p><p><span>cat</span><span> </span>/<span>boot</span>/<span>grub2</span>/<span>grub</span><span>.cfg</span><span> </span><span>|</span><span> </span><span>grep</span><span> </span><span>menuentry</span></p><p><span>#设置启动顺序</span></p><p><span>grub2</span>-<span>set</span>-<span>default</span><span> </span><span>0</span></p><p><span>#查看设置成功</span></p><p><span>grub2</span>-<span>editenv </span><span>list</span></p><p><span>#更新grub</span></p><p><span>grub2</span>-<span>mkconfig</span><span> </span>-<span>o</span><span> </span>/<span>boot</span>/<span>grub2</span>/<span>grub</span><span>.cfg</span></p><p><span>#重启</span></p><p><span>reboot</span></p><p><span>#查看已安装内核</span></p><p><span>rpm</span><span> </span>-<span>qa</span><span> </span><span>|</span><span> </span><span>grep</span><span> </span><span>kernel</span></p><p><span>#卸载无用的</span></p><p><span>yum </span><span>remove</span><span> </span>或<span> </span><span>rpm</span><span> </span>-<span>e</span></p><p><span>#安装WireGuard组件</span></p><p><span>yum </span><span>install </span><span>kmod</span>-<span>wireguard </span><span>wireguard</span>-<span>tools</span></p></div></td></tr></tbody></table>

DKMS会将WG的内核模块编译并加载上去，出现`DKMS: install completed.`并且无错误即可。如果出现内核模块未加载成功，可以尝试`reinstall`一下`wireguard-dkms`，最终以如下命令检查：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#加载内核模块</span></p><p><span>modprobe </span><span>wireguard</span></p><p><span>#检查WG模块加载是否正常</span></p><p><span>lsmod</span><span> </span><span>|</span><span> </span><span>grep </span><span>wireguard</span></p></div></td></tr></tbody></table>

确认模块有返回后，就可以把上文【二】中你需要的配置文件拿出来，配置文件目录放在`/etc/wireguard`下。比如我选择命名为`wgcf.conf`，则开启与关闭命令与配置文件对应：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#开启隧道</span></p><p><span>sudo </span><span>wg</span>-<span>quick </span><span>up </span><span>wgcf</span></p><p><span>#关闭隧道</span></p><p><span>sudo </span><span>wg</span>-<span>quick </span><span>down </span><span>wgcf</span></p></div></td></tr></tbody></table>

开启后可通过`ifconfig`看到名称为你配置文件的WARP虚拟网卡，此时便可以通过WARP进行外部访问了。

___

### 安装Go版本模块

OpenVZ和LXC因为不具有内核权限，如果确有需要自己装WireGuard-Go作为内核模块的替代，WG主程序在检测不到内核的模块时会fallback到go的模块启动程序；同时这个也可以解决部分人不想给内核加载模块的问题，只是效率略低。

在这之前我们要检查一下你的服务器是否开启了TUN支持，如果未开启的话需要联系你的服务商开启：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#尝试加载内核模块</span></p><p><span>modprobe </span><span>tun</span><span></span></p><p><span>#检查TUN模块加载是否正常</span></p><p><span>lsmod</span><span> </span><span>|</span><span> </span><span>grep</span><span> </span><span>tun</span></p></div></td></tr></tbody></table>

既然是go语言写的，自己编译和用现成别人编译好的都可以，自己编译的话需要go语言版本高于`1.13`才可以。

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#下载代码</span></p><p><span>git </span><span>clone</span><span> </span><span>https</span><span>:</span>//<span>git</span><span>.zx2c4</span><span>.com</span>/<span>wireguard</span>-<span>go</span></p><p><span>#进入源码目录</span></p><p><span>cd</span><span> </span><span>wireguard</span>-<span>go</span></p><p><span>#编译</span></p><p><span>make</span></p><p><span>#将编译好文件移到执行目录</span></p><p><span>mv</span><span> </span><span>.</span>/<span>wireguard</span>-<span>go</span><span> </span>/<span>usr</span>/<span>bin</span>/<span>wireguard</span>-<span>go</span></p><p><span>#添加执行权限</span></p><p><span>chmod</span><span> </span>+<span>x</span><span> </span>/<span>usr</span>/<span>bin</span>/<span>wireguard</span>-<span>go</span></p><p><span>#检查运行是否有返回</span></p><p><span>wireguard</span>-<span>go</span></p></div></td></tr></tbody></table>

直接用别人编译好的包也是可以的，loc上的`@52mfzy`提供了一个预编译64位的项目的地址。

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#下载二进制文件（64位Linux）</span></p><p><span>wget</span><span> </span>-<span>P</span><span> </span>/<span>usr</span>/<span>bin </span><span>https</span><span>:</span>//<span>github</span><span>.com</span>/<span>bernardkkt</span>/<span>wg</span>-<span>go</span>-<span>builder</span>/<span>releases</span>/<span>latest</span>/<span>download</span>/<span>wireguard</span>-<span>go</span></p><p><span>#添加执行权限</span></p><p><span>chmod</span><span> </span>+<span>x</span><span> </span>/<span>usr</span>/<span>bin</span>/<span>wireguard</span>-<span>go</span></p><p><span>#检查执行是否正常</span></p><p><span>wireguard</span>-<span>go</span></p></div></td></tr></tbody></table>

其他的架构我找到一个比较老的`20181222`版本，如有需要可以按自己需求选择（[点击前往](https://github.com/AlexanderOMara/wireguard-go-builds/releases/tag/0.0.20181222)），`tar.xz`包用`tar -xf`命令解压。注意这里提供的旧版本软件包因为是测试版本，需要添加确认运行的系统变量（新版本就没问题了）；你可以在启动隧道前执行一次，也可以添加到`/etc/profile`永久生效。

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#每次开机后执行 或 添加到/etc/profile</span></p><p><span>export </span><span>WG_I_PREFER_BUGGY_USERSPACE_TO_POLISHED_KMOD</span>=<span>1</span></p></div></td></tr></tbody></table>

go模块安装正确后，请参考前文添加源并安装主程序，Debian仅安装`wireguard-tools`即可，Ubuntu安装`wireguard`，CentOS仅安装`wireguard-tools`，内核模块无需安装。其余的写配置文件和使用也与前文一致。

___

### 开机自启

我不建议把WARP设置为开机自启，不过也没什么问题，官方提供的文档是通过systemctl管理的，所以比较老的系统就不用考虑了。

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#允许配置文件为wg0的开机自启</span></p><p><span>systemctl </span><span>enable </span><span>wg</span>-<span>quick</span><span>@</span><span>wg0</span><span>.service</span></p><p><span>#重载deamon配置</span></p><p><span>systemctl </span><span>daemon</span>-<span>reload</span></p><p><span>#启动wg0配置文件的进程</span></p><p><span>systemctl </span><span>start </span><span>wg</span>-<span>quick</span><span>@</span><span>wg0</span></p></div></td></tr></tbody></table>

___

## 四、结语

本文最初的目的就是因为买了Scaleway的Stardust纯IPv6实例，实测效果是非常不错的。相比于DNS64等方案，CloudFlare提供的网络非常优秀。

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-05_15-17-54.png)

腾讯云轻量这边，服务器到CF的Endpoint才0.4ms，添加IPv6后到本地谷歌的CDN节点仅仅3ms，基本可以当作本地具备的IPv6使用了。鹅厂的服务器最近用的也蛮舒服的，别具一格的升配活动连学生机都可以参与，有需要的话也可以考虑一下~

![](https://cdn.luotianyi.vc/wp-content/uploads/2021-02-06_09-35-43.png)

经过实际测试，与WARP建立的隧道在5天的时间中均未出现明显的波动，其连通性及稳定性相较于HE.NET的TunnelBroker好很多；并且作为商业化的应用，其速度得到了很好的保证。

至于前文AllowIPs讲的非常简单，只用了仅保留IPv4/IPv6其一，其实可以不必如此，它是以路由表的形式接管流量，你可以自己通过修改参数自己指定路由表的内容。这里就不展开讲了，附一个从loc大佬那里拿来的IP库（[点击下载](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/IP/IP.rar)）。

最后还是那句话，且用且珍惜，不要将公共的资源肆意加以滥用~

___

\*转载请注明原文档出处  
\*文章部分参考自Linode官方文档