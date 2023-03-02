相信已经有一部分朋友今天连接到CentOS 6的服务器后执行`yum`后发现出现404的报错，那么发生了什么？

![](https://cdn.luotianyi.vc/wp-content/uploads/2020-12-03_18-12-29.jpg)

原因是CentOS 6已经随着2020年11月的结束进入了EOL（Reaches End of Life），官方便在12月2日正式将CentOS 6相关的软件源移出了官方源，随之而来逐级镜像也会陆续将其删除。

不过有一些老设备依然需要维持在当前系统，CentOS官方也给这些还不想把CentOS 6扔进垃圾堆的用户保留了各个版本软件源的镜像，只是这个软件源不会再有更新了。

___

## 一、选择Vault源

更换的Vault源我选了两个，一个是官方的一个是阿里的。官方的源使用的是AWS位于北美的服务器，没有使用CDN，从国内访问是比较差的；另一个是阿里云的，使用了阿里云位于全球各地的CDN节点分发。

> CentOS官方：[http://vault.centos.org/](http://vault.centos.org/)  
> 阿里云镜像：[http://mirrors.aliyun.com/centos-vault/](http://mirrors.aliyun.com/centos-vault/)

位于海外的服务器建议直接使用官方的源，如果效果不好或位于国内则可以选择阿里云的镜像试一试。实际上阿里云在Developer的软件源页面并没有把Vault源挂出来，不清楚阿里云对于这个源的支持是一个什么样的态度，是否会在日后移除也是未知的。

___

## 二、更换Vault源

首先把`fastestmirror`关了，这个插件默认会寻找离你最近的镜像站去访问：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#编辑文件</span></p><p><span>vi</span><span> </span>/<span>etc</span>/<span>yum</span>/<span>pluginconf</span><span>.</span><span>d</span>/<span>fastestmirror</span><span>.</span><span>conf</span></p><p><span>#修改参数</span></p><p><span>enable</span>=<span>0</span></p></div></td></tr></tbody></table>

然后把系统中原来的源挪为备份或者直接删掉：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>mv</span><span> </span>/<span>etc</span>/<span>yum</span><span>.</span><span>repos</span><span>.</span><span>d</span>/<span>CentOS</span>-<span>Base</span><span>.</span><span>repo</span><span> </span>/<span>etc</span>/<span>yum</span><span>.</span><span>repos</span><span>.</span><span>d</span>/<span>CentOS</span>-<span>Base</span><span>.</span><span>repo</span><span>.</span><span>bak</span></p></div></td></tr></tbody></table>

然后wget下载修改后的源到对应的目录，选择合适你源的执行：

<table><tbody><tr><td data-settings="show"></td><td><div><p><span>#替换为官方Vault源</span></p><p><span>wget</span><span> </span>-<span>O</span><span> </span>/<span>etc</span>/<span>yum</span><span>.</span><span>repos</span><span>.</span><span>d</span>/<span>CentOS</span>-<span>Base</span><span>.</span><span>repo </span><span>https</span><span>:</span><span>//static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/SourcesList/Centos-6-Vault-Official.repo</span></p><p><span>#替换为阿里云Vault镜像</span></p><p><span>wget</span><span> </span>-<span>O</span><span> </span>/<span>etc</span>/<span>yum</span><span>.</span><span>repos</span><span>.</span><span>d</span>/<span>CentOS</span>-<span>Base</span><span>.</span><span>repo </span><span>https</span><span>:</span><span>//static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/SourcesList/Centos-6-Vault-Aliyun.repo</span></p></div></td></tr></tbody></table>

___

## 三、手动更换

如果是全新的系统没有wget的话，请使用SFTP或者直接使用`nano`、`vi`编辑`/etc/yum.repos.d/`目录下`CentOS-Base.repo`这个文件，直接将其中的内容修改为源文件的内容：

> CentOS官方：[点击查看](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/SourcesList/Centos-6-Vault-Official.repo.txt)  
> 阿里云镜像：[点击查看](https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/SourcesList/Centos-6-Vault-Aliyun.repo.txt)

更换后尝试`yum update`，能够正常获取软件列表即可。注意这个源文件选择的系统版本是CentOS 6.10，若你想保持在更低版本的系统不进行升级请在源文件中将`6.10`批量替换为`6.x`，Vault源对各版本均有保留。

___

## 四、结语

最后还是得说一句，虽然更换Vault源能够保证系统基础功能正常，但是已经进入EOL的系统失去了官方的更新和维护，日后有可能会因为一些漏洞而被入侵，如果可以的话还是建议更新到最新版本受支持的系统。

同样的Debian类似的也有`snapshot.debian.org`，相关的源文件刚刚整理了一下也放在了镜像站里（[点击前往](https://static.lty.fun/?%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/SourcesList)），如果需要可以自取。

### Debian

Debian的源备份有两种，一种是像CentOS的Vault一样的最终源`Debian Archive`，还有一种是每天2-4次的整源镜像`Snapshot`，按照自己的需求去选择即可。这里需要提一句，`Archive`里面会把较老的系统的二进制包删掉只保留源码，所以个人还是推荐用`Snapshot`。

> Debian Snapshot：[http://snapshot.debian.org/](http://snapshot.debian.org/)  
> Debian Archive：`http://archive.debian.org/debian/`  
> Archive镜像：`https://mirrors.cloud.tencent.com/debian-archive/`

### Ubuntu

> Ubuntu Old Release：`http://old-releases.ubuntu.com/ubuntu/`  
> Old Release镜像：`http://mirrors.ustc.edu.cn/ubuntu-old-releases/ubuntu/`

___

\*开放转载。无须注明出处