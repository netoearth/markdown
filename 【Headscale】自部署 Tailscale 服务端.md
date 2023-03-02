自部署开源 Tailscale 服务端 —— Headscale。

Tailscale 是一个优秀的内网穿透和异地组网工具，基于 Wireguard ，配合 Tailscale 的穿透策略，基本上可以应用于大多数NAT场景。

Tailscale 是一款商用产品，对个人提供免费服务，就个人使用来说基本上是完全够用了（限制20台设备）。但是其后端服务器全都位于国外，在国内延迟实在有点高，而且完全依赖于第三方SSO认证，且官方支持的SSO平台也都在国外，在国内每况愈下的网络环境想快速的连接这些平台实在是有些困难，于是我们就需要自己部署后端了。

有人认为 Tailscale 后台不开源不够透明不够开放，于是自行开发了一套后端，命名为`Headscale`，很恶趣味的名字，这套后端完全开源，可自行部署，最重要的是比 Netmaker 部署要容易得多，而且打洞失败可以自动转中继，比较智能。

## [](https://lxnchan.cn/headscale.html#%E9%85%8D%E7%BD%AE%E6%9C%8D%E5%8A%A1%E7%AB%AF "配置服务端")配置服务端

### [](https://lxnchan.cn/headscale.html#%E5%9F%BA%E7%A1%80%E9%85%8D%E7%BD%AE "基础配置")基础配置

下载 Headscale 的服务端二进制文件：[Github Release](https://github.com/juanfont/headscale/releases/) 。

> 目前官方**支持arm架构**（意味着可以部署在路由器和其他嵌入式设备上）和x86架构，原则上来说可以部署在任一节点上，但这里建议部署在具有公网IP的、没有NAT的服务器上以确保服务的稳定性。

将下载好的二进制文件移动到`/usr/local/bin/`目录下并改名`headscale`，然后进行部署准备：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br></pre></td><td><pre><span></span><br><span>chmod +x /usr/<span>local</span>/bin/headscale</span><br><span></span><br><span>mkdir -p /etc/headscale</span><br><span></span><br><span>mkdir -p /var/lib/headscale</span><br><span></span><br><span>touch /var/lib/headscale/db.sqlite</span><br></pre></td></tr></tbody></table>

下载 [示例配置文件](https://github.com/juanfont/headscale/raw/main/config-example.yaml) 存放到`/etc/headscale/`目录下并改名`config.yaml`，需要修改的配置项如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br></pre></td><td><pre><span></span><br><span><span>server_url:</span> <span>http://&lt;server_Address&gt;:8080</span></span><br><span></span><br><span><span>ip_prefixes:</span></span><br><span><span>  - fd7a:</span><span>115</span><span>c:a1e0::/48</span></span><br><span><span>  -</span> <span>10.249</span><span>.249</span><span>.0</span><span>/16</span></span><br><span></span><br><span><span>dns_config:</span></span><br><span><span>  nameservers:</span></span><br><span><span>    -</span> <span>114.114</span><span>.114</span><span>.114</span></span><br><span></span><br><span><span>magic_dns:</span> <span>false</span></span><br><span></span><br><span><span>unix_socket:</span> <span>/var/run/headscale/headscale.sock</span></span><br></pre></td></tr></tbody></table>

接着再创建 gRPC socket ：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>mkdir /var/run/headscale</span><br><span>touch /var/run/headscale/headscale.sock</span><br></pre></td></tr></tbody></table>

权限和安全配置（可选）：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span></span><br><span>adduser --no-create-home --disabled-login --shell /sbin/nologin --disabled-password headscale</span><br><span></span><br><span>chown -R headscale:headscale /var/lib/headscale</span><br></pre></td></tr></tbody></table>

此时就可以试运行一下 Headscale 了：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>headscale serve</span><br></pre></td></tr></tbody></table>

如果没有报错就可以进入下一步了。

### [](https://lxnchan.cn/headscale.html#%E5%90%8E%E5%8F%B0%E5%92%8C%E5%BC%80%E6%9C%BA%E8%87%AA%E5%90%AF "后台和开机自启")后台和开机自启

新建Service文件：`/etc/systemd/system/headscale.service`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br></pre></td><td><pre><span><span>[Unit]</span></span><br><span><span>Description</span>=headscale controller</span><br><span><span>After</span>=syslog.target</span><br><span><span>After</span>=network.target</span><br><span></span><br><span><span>[Service]</span></span><br><span><span>Type</span>=simple</span><br><span><span>User</span>=headscale</span><br><span><span>Group</span>=headscale</span><br><span><span>ExecStart</span>=/usr/local/bin/headscale serve</span><br><span><span>Restart</span>=always</span><br><span><span>RestartSec</span>=<span>5</span></span><br><span></span><br><span></span><br><span><span>NoNewPrivileges</span>=<span>yes</span></span><br><span><span>PrivateTmp</span>=<span>yes</span></span><br><span><span>ProtectSystem</span>=strict</span><br><span><span>ProtectHome</span>=<span>yes</span></span><br><span><span>ReadWritePaths</span>=/var/lib/headscale /var/run/headscale</span><br><span><span>AmbientCapabilities</span>=CAP_NET_BIND_SERVICE</span><br><span><span>RuntimeDirectory</span>=headscale</span><br><span></span><br><span><span>[Install]</span></span><br><span><span>WantedBy</span>=multi-user.target</span><br></pre></td></tr></tbody></table>

随后启动即可：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><span></span><br><span>systemctl start headscale</span><br><span></span><br><span>systemctl <span>enable</span> headscale</span><br><span></span><br><span>systemctl status headscle</span><br></pre></td></tr></tbody></table>

### [](https://lxnchan.cn/headscale.html#%E6%96%B0%E5%BB%BA%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%EF%BC%88Tailnet%EF%BC%89 "新建命名空间（Tailnet）")新建命名空间（Tailnet）

Tailscale 中有一个概念叫 tailnet，你可以理解成租户，租户与租户之间是相互隔离的，具体看参考 Tailscale 的官方文档： [What is a tailnet](https://tailscale.com/kb/1136/tailnet/) 。Headscale 也有类似的实现叫 namespace，即命名空间。我们需要先创建一个 namespace，以便后续客户端接入。

创建命名空间：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span></span><br><span>headscale namespaces create &lt;namespace&gt;</span><br></pre></td></tr></tbody></table>

查看命名空间：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>headscale namespaces list</span><br></pre></td></tr></tbody></table>

## [](https://lxnchan.cn/headscale.html#%E9%85%8D%E7%BD%AE%E5%AE%A2%E6%88%B7%E7%AB%AF "配置客户端")配置客户端

### [](https://lxnchan.cn/headscale.html#%E6%94%AF%E6%8C%81%E7%9A%84%E5%AE%A2%E6%88%B7%E7%AB%AF "支持的客户端")支持的客户端

并非全部客户端都支持（或全部支持）将 Headscale 作为后端，目前适配情况如下：

| 操作系统 | 支持情况 | 解决方案 |
| --- | --- | --- |
| Linux | ✅ | 原生支持 |
| Windows | ✅ | 修改注册表 |
| macOS | ✅ | 需要添加描述文件 |
| Android | ⭕ | 需要自编译APK |
| iOS | ❌ | 目前不支持 |

### [](https://lxnchan.cn/headscale.html#%E6%9F%A5%E8%AF%A2%E5%B7%B2%E6%B3%A8%E5%86%8C%E7%9A%84%E8%8A%82%E7%82%B9 "查询已注册的节点")查询已注册的节点

在经过下面的步骤注册完客户端后可通过这条命令查询已注册的节点：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>headscale nodes list</span><br></pre></td></tr></tbody></table>

输出：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><span>ID | Name                      | NodeKey | Namespace | IP addresses  | Ephemeral | Last seen           | Online  | Expired</span><br><span>1  | lxn-testbench-ubuntu-v34  | [0]     | main      | 192.168.5.1   | <span>false</span>     | 2022-04-27 00:59:57 | offline | no     </span><br><span>2  | lxn-testbench-windows-v35 | [0]     | main      | 192.168.5.2   | <span>false</span>     | 2022-04-27 06:04:50 | online  | no    </span><br><span>3  | lxn-testbench-macos-v37   | [0]     | main      | 192.168.5.3   | <span>false</span>     | 2022-04-27 08:33:23 | offline | no    </span><br><span>4  | lxn-testbench-ubuntu-v36  | [0]     | main      | 192.168.5.4   | <span>false</span>     | 2022-04-27 09:22:11 | online  | no     </span><br><span>5  | lxn-testbench-mia5-v11    | [0]     | main      | 192.168.5.6   | <span>false</span>     | 2022-04-27 11:49:33 | online  | no</span><br></pre></td></tr></tbody></table>

### [](https://lxnchan.cn/headscale.html#Linux "Linux")Linux

> 这一小节操作基于 Ubuntu 20.04(Focal) LTS Desktop Edition。

先下载Linux的Tailscale客户端，可以使用官方的一键脚本：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>curl -fsSL https://tailscale.com/install.sh | sh</span><br></pre></td></tr></tbody></table>

其他的安装说明和安装包下载可以在 [Tailscale Packages - stable track](https://pkgs.tailscale.com/stable/) 找到。

安装完成后可以执行如下命令将 Tailscale 加入 Headscale 网络：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span></span><br><span>tailscale up --login-server=http://&lt;server_Address&gt;:8080 --accept-routes=<span>true</span> --accept-dns=<span>false</span></span><br></pre></td></tr></tbody></table>

如果连接成功会有如下输出：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>root@lxn-testbench-v35:~</span><br><span></span><br><span>To authenticate, visit:</span><br><span></span><br><span>        http://0.0.0.0:8080/register?key=0</span><br></pre></td></tr></tbody></table>

把这段网址复制到浏览器打开，会返回如下输出：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>headscale</span><br><span>Machine registration</span><br><span>Run the command below in the headscale server to add this machine to your network:</span><br><span></span><br><span>headscale -n NAMESPACE nodes register --key 0</span><br></pre></td></tr></tbody></table>

把最后一行的命令复制到 headscale 服务器上并将`NAMESPACE`替换为刚才创建的命名空间后执行，执行成功后会返回`Machine register`，同时在客户机上也应该有返回输出`Success.`，之后再按照上面查看已连接客户端的方法在服务端查询确认设备在线就可以了。

### [](https://lxnchan.cn/headscale.html#Windows "Windows")Windows

> 这一小节操作基于 Windows 11 Dev。

先下载 Windows 的 Tailscale 客户端，[下载地址](https://tailscale.com/download/windows) ，下载后正常安装即可，不必登录。找到状态栏右下角的Tailscale图标，右键退出。

然后在客户端浏览器上打开`http://<server_Address>:8080/windows`，并按照屏幕提示添加对应的注册表项或下载导入由服务器提供的注册表文件，随后再重新打开 Tailscale 客户端。

> 由于众所周知的原因，国内可能无法正常连接到Github，某些必要的文件也就无法从Github下载。如果此时无法正常打开`http://<server_Address>:8080/windows`请按照如下方法添加注册表项。

打开命令提示符（管理员），输入以下命令：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>REG ADD "HKLM\Software\Tailscale IPN" /v UnattendedMode /t REG_SZ /d always</span><br><span># 将&lt;server_Address&gt;替换为你服务器的地址</span><br><span>REG ADD "HKLM\Software\Tailscale IPN" /v LoginURL /t REG_SZ /d "http://&lt;server_Address&gt;:<span>8080</span>"</span><br></pre></td></tr></tbody></table>

此时右键Tailscale的托盘图标，点击登录（Log in…），会返回如下输出：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>headscale</span><br><span>Machine registration</span><br><span>Run the command below in the headscale server to add this machine to your network:</span><br><span></span><br><span>headscale -n NAMESPACE nodes register --key 0</span><br></pre></td></tr></tbody></table>

把最后一行的命令复制到 headscale 服务器上并将`NAMESPACE`替换为刚才创建的命名空间后执行，执行成功后会返回`Machine register`，同时在客户机上Tailscale的托盘图标也会发生变化，右键可以看到`Connected`字样，之后再按照上面查看已连接客户端的方法在服务端查询确认设备在线就可以了。

### [](https://lxnchan.cn/headscale.html#macOS "macOS")macOS

> 这一小节的操作基于 macOS 12.5.7。

macOS加入Headscale网络的方式可能略显麻烦，因为需要一个美区的Apple ID，据说国区是没有 Tailscale 这个软件的。

直接在App Store下载 [Tailscale](https://apps.apple.com/us/app/tailscale/id1475387142?l=zh&mt=12) ，下载后正常安装即可，不必登录，也先不要打开 Tailscale 。

在客户端的浏览器上打开`http://<server_Address>:8080/apple`，并按照屏幕提示在 Terminal 做相应的操作或直接下载导入由服务器提供的描述文件，随后打开 Tailscale 客户端。

> 由于众所周知的原因，国内可能无法正常连接到Github，某些必要的文件也就无法从Github下载。如果此时无法正常打开`http://<server_Address>:8080/apple`请按照如下方法添加注册表项。

打开 Terminal （终端），输入以下命令：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>defaults write io.tailscale.ipn.macos ControlURL http://&lt;server_Address&gt;:8080</span><br></pre></td></tr></tbody></table>

然后打开 Tailscale ，点击`Get Start`，`Allow VPN Configurations`，`Sign in to your network`，然后会自动打开Safari，把最后一行的命令复制到 headscale 服务器上并将`NAMESPACE`替换为刚才创建的命名空间后执行，执行成功后会返回`Machine register`，同时在客户机上Tailscale的托盘图标上的感叹号会消失，右键可以看到`Connected`字样，之后再按照上面查看已连接客户端的方法在服务端查询确认设备在线就可以了。

> 如果打开 Tailscale 后仍然是 Tailscale 官方的登录界面，则重启一下mac就可以了。

### [](https://lxnchan.cn/headscale.html#Android "Android")Android

> 这一小节的操作配置如下：  
> 编译系统：Ubuntu 20.04 LTS Server Edition；  
> 客户端运行环境：Android 5.1.1/MIUI 10 稳定版。

> 编译需要较强的性能，否则可能会被卡CPU检测Kill掉。

Android 的 Tailscale 客户端需要自行编译，方法如下：

安装依赖：

1.  安装GoLang的步骤请看 [Ubuntu 安装 Golang 最新版本](https://lxnchan.cn/ubuntu-go.html) 。
    
2.  安装Git：`apt update&&apt install git`。
    

首先将 Tailscale 的客户端 clone 到本地：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>git <span>clone</span> https://github.com/tailscale/tailscale-android.git</span><br></pre></td></tr></tbody></table>

（可选）预编译和运行确保没有数据错误：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>make tailscale-debug.apk</span><br></pre></td></tr></tbody></table>

修改后端地址。找到文件`/tailscale-android/cmd/tailscale/backend.go`，查找如下代码段：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><span>func (b *backend) Start(notify func(n ipn.Notify)) error {</span><br><span>b.backend.SetNotifyCallback(notify)</span><br><span><span>return</span> b.backend.Start(ipn.Options{</span><br><span>StateKey: <span>"ipn-android"</span>,</span><br><span>})</span><br><span>}</span><br></pre></td></tr></tbody></table>

修改为：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br></pre></td><td><pre><span>func (b *backend) Start(notify func(n ipn.Notify)) error {</span><br><span>    b.backend.SetNotifyCallback(notify)</span><br><span>    prefs := ipn.NewPrefs()</span><br><span>    prefs.ControlURL = <span>"http://&lt;server_Address&gt;:8080"</span></span><br><span>    opts := ipn.Options{</span><br><span>    StateKey: <span>"ipn-android"</span>,</span><br><span>        UpdatePrefs: prefs,</span><br><span>    }</span><br><span>    <span>return</span> b.backend.Start(opts)</span><br><span>}</span><br></pre></td></tr></tbody></table>

其中`<server_Address>`修改为Headscale服务器的地址。

编译：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>make tailscale-debug.apk</span><br></pre></td></tr></tbody></table>

安装：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>adb install tailscale-debug.apk</span><br></pre></td></tr></tbody></table>

安装后在手机上打开，点击`Get start`，登录时选择`Sign in with others`，会自动跳转到浏览器，把最后一行的命令复制到 headscale 服务器上并将`NAMESPACE`替换为刚才创建的命名空间后执行，执行成功后会返回`Machine register`，同时在客户机上会自动跳转到主界面，将主界面上方的开关打开后，再按照上面查看已连接客户端的方法在服务端查询确认设备在线就可以了。

### [](https://lxnchan.cn/headscale.html#iOS "iOS")iOS

iOS暂不支持使用Headscale作为后端，因为Tailscale的iOS客户端没有开源，而且官方的解决方案经过尝试也是不可用的（在iOS上安装由服务端提供的描述文件），所以只能等官方适配了。

## [](https://lxnchan.cn/headscale.html#%E6%80%BB%E7%BB%93 "总结")总结

Headscale 的部署方式实在是简单多了，~建议大力推广~。

## [](https://lxnchan.cn/headscale.html#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99 "参考资料")参考资料

> 排名不分先后

-   [Tailscale 基础教程：Headscale 的部署方法和使用教程](https://icloudnative.io/posts/how-to-set-up-or-migrate-headscale/)

 ######  上一篇

#### [Ubuntu 安装 Transmission 和 WebUI](https://lxnchan.cn/ubuntu-tm.html "Ubuntu 安装 Transmission 和 WebUI")

###### [Linux](https://lxnchan.cn/tags/Linux/)