## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8-SSH-%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3 "史上最全 SSH 暗黑技巧详解")史上最全 SSH 暗黑技巧详解

我见过太多的老鸟、新手对ssh 基本只限于 ssh到远程机器，实际这个命令我们一天要用很多次，但是对它的了解太少了，他的强大远远超出你的想象。当于你也许会说够用就够了，确实没错，但是你考虑过效率没有，或者还有哪些脑洞大开的功能会让你爱死他，这些功能又仅仅是一行命令就够了。

疫情期间一行SSH命令让我节省了70%的出差时间，来，让我们一起走一遍，看看会不会让你大开眼界

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%9C%AC%E6%96%87%E8%AF%95%E5%9B%BE%E8%A7%A3%E5%86%B3%E7%9A%84%E9%97%AE%E9%A2%98 "本文试图解决的问题")本文试图解决的问题

-   如何通过ssh命令科学上网
-   docker 镜像、golang仓库总是被墙怎么办
-   公司跳板机要输入动态token，太麻烦了，如何省略掉这个token；
-   比如多机房总是要走跳板机，如何`绕过`跳板机直连；
-   我的开发测试机器如何免打通、免密码、直达；
-   如何访问隔离环境中(k8s)的Web服务 – 将隔离环境中的web端口映射到本地
-   如何让隔离环境的机器用上yum、apt
-   如何将服务器的图形界面映射到本地(类似vnc的作用)
-   ssh如何调试诊断，这才是终极技能……

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9 "注意事项")注意事项

-   ssh是指的openSSH 命令工具
-   本文适用于各种Linux、MacOS下命令行操作，Windows的话各种可视化工具都可以复制session、配置tunnel来实现类似功能。
-   如果文章中提到的文件、文件夹不存在可以直接创建出来。
-   所有配置都是在你的笔记本上（相当于ssh client上）

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91 "科学上网")科学上网

有时候科学上网还得靠自己，一行ssh命令来科学上网:

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>nohup ssh -qTfnN -D 127.0.0.1:38080 root@1.1.1.1 "vmstat 10" 2&gt;&amp;1 &gt;/dev/null &amp;</p></pre></td></tr></tbody></table>

上面的 1.1.1.1 是你在境外的一台服务器，已经做好了免密登陆（免密见后面，要不你还得输一下密码），这句话的意思就是在本地启动一个38080的端口，上面收到的任何东西都会转发给 1.1.1.1:22（做了ssh加密），1.1.1.1:22 会解密收到的东西，然后把他们转发给google之类的网站（比如你要访问的是google），结果依然通过原路返回

127.0.0.1:38080 socks5 就是要填入到你的浏览器中的代理服务器，什么都不需要装，非常简单

[![image.png](https://plantegg.github.io/images/oss/e4a2fdad5b04542dc657b96e195a2b45.png)](https://plantegg.github.io/images/oss/e4a2fdad5b04542dc657b96e195a2b45.png)

原理图如下(灰色矩形框就是你本地ssh命令，ssh 线就是在穿墙， 国外服务器就是命令中的1.1.1.1)：  
[![undefined](https://plantegg.github.io/images/oss/1561367815573-0b793473-67fa-4edc-ae58-04e7c4c51b87.png)](https://plantegg.github.io/images/oss/1561367815573-0b793473-67fa-4edc-ae58-04e7c4c51b87.png)

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E4%B9%8Bhttp%E7%89%B9%E6%AE%8A%E4%BB%A3%E7%90%86%E2%80%93%E5%88%A9%E7%94%A8ssh-%E6%9C%AC%E5%9C%B0%E8%BD%AC%E5%8F%91%E6%98%AFHTTP%E5%8D%8F%E8%AE%AE "科学上网之http特殊代理–利用ssh 本地转发是HTTP协议")科学上网之http特殊代理–利用ssh 本地转发是HTTP协议

前面所说的代理是socks5代理，一般浏览器都有插件支持，但是比如你的docker（或者其他程序）需要通过http去拉取镜像就会出现如下错误：

```
Sending build context to Docker daemon 8.704 kB
Step 1 : FROM k8s.gcr.io/kube-cross:v1.10.1-1
Get https://k8s.gcr.io/v1/_ping: dial tcp 108.177.125.82:443: i/o timeout
```

[如果是git这样的应用内部可以配置socks5和http代理服务器，请参考另外一篇文章](https://www.atatech.org/articles/102153)，但是有些应用就不能配置了，当然最终通过ssh大法还是可以解决这个问题：

```
sudo ssh -L 443:108.177.125.82:443 root@1.1.1.1 //在本地监听443，转发给远程108.177.125.82的443端口
```

然后再在 /etc/hosts 中将域名 k8s.gcr.io 指向 127.0.0.1， 那么本来要访问 k8s.gcr.io:443的，变成了访问本地 127.0.0.1:443 而 127.0.0.1:443 又通过ssh重定向到了 108.177.125.82:443 这样就实现了http代理或者说这种特殊情况下的科学上网。这个方案不需要装任何东西，但是每个访问目标都要这样处理，好在这种情况不多

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%86%85%E9%83%A8%E5%A0%A1%E5%9E%92%E6%9C%BA%E3%80%81%E8%B7%B3%E6%9D%BF%E6%9C%BA%E9%83%BD%E9%9C%80%E8%A6%81%E5%AF%86%E7%A0%81-%E5%8A%A8%E6%80%81%E7%A0%81%EF%BC%8C%E5%A4%AA%E5%A4%8D%E6%9D%82%E4%BA%86%EF%BC%8C%E6%80%8E%E4%B9%88%E8%A7%A3%EF%BC%9F "内部堡垒机、跳板机都需要密码+动态码，太复杂了，怎么解？")内部堡垒机、跳板机都需要密码+动态码，太复杂了，怎么解？

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p></pre></td><td><pre><p><span>$</span><span> cat ~/.ssh/config </span></p><p><span>#</span><span>reuse the same connection --关键配置</span></p><p>ControlMaster auto</p><p>ControlPath ~/tmp/ssh_mux_%h_%p_%r</p><p>#<span>查了下ControlPersist是在OpenSSH5.6加入的，5.3还不支持</span></p><p><span>#</span><span>不支持的话直接把这行删了，不影响功能</span></p><p><span>#</span><span>keep one connection <span>in</span> 72hour</span></p><p><span>#</span><span>ControlPersist 72h</span></p><p><span>#</span><span>复用连接的配置到这里，后面的配置与复用无关</span></p><p>#<span>其它也很有用的配置</span></p><p>GSSAPIAuthentication=no</p><p>StrictHostKeyChecking=no</p><p>TCPKeepAlive=yes</p><p>CheckHostIP=no</p><p><span>#</span><span> <span>"ServerAliveInterval [seconds]"</span> configuration <span>in</span> the SSH configuration so that your ssh client sends a <span>"dummy packet"</span> on a regular interval so that the router thinks that the connection is active even <span>if</span> it<span>'s particularly quiet</span></span></p><p>ServerAliveInterval=15</p><p><span>#</span><span><span>ServerAliveCountMax=6</span></span></p><p>ForwardAgent=yes</p><p>UserKnownHostsFile /dev/null</p></pre></td></tr></tbody></table>

在你的ssh配置文件增加上述参数，意味着72小时内登录同一台跳板机只有第一次需要输入密码，以后都是重用之前的连接，所以也就不再需要输入密码了。

加了如上参数后的登录过程就有这样的东东(默认没有，这是debug信息)：

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p></pre></td><td><pre><p>debug1: setting up multiplex master socket</p><p>debug3: muxserver_listen: temporary control path   /home/ren/tmp/ssh_mux_10.16.*.*_22_corp.86g3C34vy36tvCtn</p><p>debug2: fd 3 setting O_NONBLOCK</p><p>debug3: fd 3 is O_NONBLOCK</p><p>debug3: fd 3 is O_NONBLOCK</p><p>debug1: channel 0: new [/home/ren/tmp/ssh_mux_10.16.*.*_22_corp]</p><p>debug3: muxserver_listen: mux listener channel 0 fd 3</p><p>debug1: control_persist_detach: backgrounding master process</p><p>debug2: control_persist_detach: background process is 15154</p><p>debug2: fd 3 setting O_NONBLOCK</p><p>debug1: forking to background</p><p>debug1: Entering interactive session.</p><p>debug2: set_control_persist_exit_time: schedule exit in 259200 seconds</p><p>debug1: multiplexing control connection</p></pre></td></tr></tbody></table>

/home/ren/tmp/ssh\_mux\_10.16._._\_22\_corp 这个就是保存好的socket，下次可以重用，免密码。 in 259200 seconds 对应 72小时

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%88%91%E6%9C%89%E5%BE%88%E5%A4%9A%E4%B8%8D%E5%90%8C%E6%9C%BA%E6%88%BF%EF%BC%88%E6%88%96%E8%80%85%E8%AF%B4%E4%B8%8D%E5%90%8C%E5%AE%A2%E6%88%B7%EF%BC%89%E7%9A%84%E6%9C%BA%E5%99%A8%E9%83%BD%E9%9C%80%E8%A6%81%E8%B7%B3%E6%9D%BF%E6%9C%BA%E6%9D%A5%E7%99%BB%E5%BD%95%EF%BC%8C%E8%83%BD%E4%B8%80%E6%AC%A1%E7%9B%B4%E6%8E%A5ssh%E4%B8%8A%E5%8E%BB%E5%90%97%EF%BC%9F "我有很多不同机房（或者说不同客户）的机器都需要跳板机来登录，能一次直接ssh上去吗？")我有很多不同机房（或者说不同客户）的机器都需要跳板机来登录，能一次直接ssh上去吗？

比如有一批客户机房的机器IP都是192.168._._, 然后需要走跳板机100.10.1.2才能访问到，那么我希望以后**在笔记本上直接 ssh 192.168.1.5 就能直接连上**

```
$ cat /etc/ssh/ssh_config

Host 192.168.*.*
ProxyCommand ssh -l ali-renxijun 100.10.1.2 exec /usr/bin/nc %h %p
```

上面配置的意思是执行 ssh 192.168.1.5的时候命中规则 Host 192.168._._ 所以执行 ProxyCommand 先连上跳板机再通过跳板机连向192.168.1.5 。这样在你的笔记本上就跟192.168._._ 的机器仿佛在一起，ssh可以上去，但是ping不通这个192.168.1.5的ip

**划重点：公司的线上跳板机做了特殊限制，限制了这个技能。日常环境跳板机支持这个功能**

比如我的跳板配置：

```
#到美国的机器用美国的跳板机速度更快
Host 10.74.*
ProxyCommand ssh -l user us.jump exec /bin/nc %h %p 2>/dev/null
#到中国的机器用中国的跳板机速度更快
Host 10.70.*
ProxyCommand ssh -l user cn.jump exec /bin/nc %h %p 2>/dev/null

Host 192.168.0.*
ProxyCommand ssh -l user 1.1.1.1 exec /usr/bin/nc %h %p
```

其实我的配置文件里面还有很多规则，懒得一个个隐藏IP了，这些规则是可以重复匹配的

来看一个例子

```
ren@ren-VirtualBox:/$ ping -c 1 10.16.1.*
        PING 10.16.1.* (10.16.1.*) 56(84) bytes of data.^C
    --- 10.16.1.* ping statistics ---
    1 packets transmitted, 0 received, 100% packet loss, time 0ms

ren@ren-VirtualBox:~$ ssh -l corp 10.16.1.* -vvv
OpenSSH_6.7p1 Ubuntu-5ubuntu1, OpenSSL 1.0.1f 6 Jan 2014
debug1: Reading configuration data /home/ren/.ssh/config
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 28: Applying options for *
debug1: /etc/ssh/ssh_config line 44: Applying options for 10.16.*.*
debug1: /etc/ssh/ssh_config line 68: Applying options for *
debug1: auto-mux: Trying existing master
debug1: Control socket "/home/ren/tmp/ssh_mux_10.16.1.*_22_corp" does not exist
debug1: Executing proxy command: exec ssh -l corp 139.*.*.* exec /usr/bin/nc 10.16.1.* 22
```

本来我的笔记本跟 10.16.1. _是不通的(ping 不通），但是ssh可以直接连上，实际ssh登录过程中自动走跳板机139._._._ 就连上了

\-vvv 参数是debug，把ssh登录过程的日志全部打印出来。

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#ProxyJump "ProxyJump")ProxyJump

需要 `OpenSSH 7.3` 以上版本才可以使用 `ProxyJump`, 相对 ProxyCommand 更简洁方便些

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p></pre></td><td><pre><p>#ssh 116 就可以通过 jumpserver:50023 连上 root@1.116.2.1:22</p><p>Host 116</p><p>    HostName 1.116.2.1</p><p>    Port 22</p><p>    User root</p><p>    ProxyJump admin@jumpserver:50023</p><p>#ssh 1.112.任意ip 都会默认走 jumpserver 跳转过去</p><p>Host 1.112.*.*</p><p>    Port 22</p><p>    User root</p><p>    ProxyJump root@jumpserver</p></pre></td></tr></tbody></table>

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%B0%86%E9%9A%94%E7%A6%BB%E7%8E%AF%E5%A2%83%E4%B8%AD%E7%9A%84web%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84%E5%88%B0%E6%9C%AC%E5%9C%B0%EF%BC%88%E6%9C%AC%E5%9C%B0%E4%BB%A3%E7%90%86%EF%BC%89 "将隔离环境中的web端口映射到本地（本地代理）")将隔离环境中的web端口映射到本地（本地代理）

远程机器部署了WEB Server，需要通过浏览器来访问这个WEB服务，但是server在隔离环境中，只能通过ssh访问到。一般来说会在隔离环境中部署一个windows机器，通过这个windows机器来访问到这个web server。能不能省掉这个windows机器呢？

现在我们试着用ssh来实现本地浏览器直接访问到这个隔离环境中的WEB Server。

假设web server是：10.1.1.123:8083， ssh账号是：user

先配置好本地直接 ssh user@10.1.1.123 （参考前面的 ProxyCommand配置过程，最好是免密也配置好），然后在你的笔记本上执行：

```
ssh -CNfL 0.0.0.0:8088:10.1.1.123:8083 user@10.1.1.123
```

或者：(root@100.1.2.3 -p 54900 是可达10.1.1.123的代理服务器)

```
ssh -CNfL 0.0.0.0:8089:10.1.1.123:8083 root@100.1.2.3 -p 54900
```

这表示在本地启动一个8088的端口，将这个8088端口映射到10.1.1.123的8083端口上，用的ssh账号是user

然后在笔记本上的浏览器中输入： 127.0.0.1：8088 就看到了如下界面：

[![image.png](https://plantegg.github.io/images/oss/1acbd09b4b45dbd478ddabc0e001a15e.png)](https://plantegg.github.io/images/oss/1acbd09b4b45dbd478ddabc0e001a15e.png)

反过来，**也可以让隔离环境机器通过代理上网，比如安装yum**

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E4%B8%BA%E4%BB%80%E4%B9%88%E6%9C%89%E6%97%B6%E5%80%99ssh-%E6%AF%94%E8%BE%83%E6%85%A2%EF%BC%8C%E6%AF%94%E5%A6%82%E6%80%BB%E6%98%AF%E9%9C%80%E8%A6%8130%E7%A7%92%E9%92%9F%E5%90%8E%E6%89%8D%E8%83%BD%E6%AD%A3%E5%B8%B8%E7%99%BB%E5%BD%95 "为什么有时候ssh 比较慢，比如总是需要30秒钟后才能正常登录")为什么有时候ssh 比较慢，比如总是需要30秒钟后才能正常登录

先了解如下知识点，在 ~/.ssh/config 配置文件中：

```
GSSAPIAuthentication=no
```

禁掉 GSSAPI认证，GSSAPIAuthentication是个什么鬼东西请自行 Google(多一次没必要的授权认证过程，然后等待超时)。 这里要理解ssh登录的时候有很多种认证方式（公钥、密码等等），具体怎么调试请记住强大的命令参数 ssh -vvv 上面讲到的技巧都能通过 -vvv 看到具体过程。

比如我第一次碰到ssh 比较慢总是需要30秒后才登录，不能忍受，于是登录的时候加上 -vvv明显看到控制台停在了：GSSAPIAuthentication 然后Google了一下，禁掉就好了

当然还有去掉每次ssh都需要先输入yes

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%89%B9%E9%87%8F%E6%89%93%E9%80%9A%E6%89%80%E6%9C%89%E6%9C%BA%E5%99%A8%E4%B9%8B%E9%97%B4%E7%9A%84ssh%E7%99%BB%E5%BD%95%E5%85%8D%E5%AF%86%E7%A0%81 "批量打通所有机器之间的ssh登录免密码")批量打通所有机器之间的ssh登录免密码

**Expect在有些公司是被禁止的**

ssh免密码的原理是将本机的pub key复制到目标机器的 ~/.ssh/authorized\_keys 里面。可以手工复制粘贴，也可以 ssh-copy-id 等

如果有100台机器，互相两两打通还是比较费事（大概需要100\*99次copy key）。 下面通过 expect 来解决输入密码，然后配合shell脚本来批量解决这个问题。

[![](https://plantegg.github.io/images/951413iMgBlog/S9jLW7B.png)](https://plantegg.github.io/images/951413iMgBlog/S9jLW7B.png)

这个脚本需要四个参数：目标IP、用户名、密码、home目录，也就是ssh到一台机器的时候帮我们自动填上yes，和密码，这样就不需要人肉一个个输入了。

再在外面写一个循环对每个IP执行如下操作：

[![](https://plantegg.github.io/images/951413iMgBlog/4SZcnvc.png)](https://plantegg.github.io/images/951413iMgBlog/4SZcnvc.png)

if代码部分检查本机~/.ssh/下有没有id\_rsa.pub，也就是是否以前生成过密钥对，没生成的话就帮忙生成一次。

for循环部分一次把生成的密钥对和authorized\_keys复制到所有机器上，这样所有机器之间都不需要输入密码就能互相登陆了（当然本机也不需要输入密码登录所有机器）

最后一行代码：

```
ssh $user@$n "hostname -i"
```

验证一下没有输密码是否能成功ssh上去。

**思考一下，为什么这么做就可以打通两两之间的免密码登录，这里没有把所有机器的pub key复制到其他所有机器上去啊**

> 答案：其实这个脚本做了一个取巧投机的事，那就是让所有机器共享一套公钥、私钥。  
> 有时候我也会把我的windows笔记本和我专用的某台虚拟机共享一套秘钥，这样任何新申请的机器打通一次账号就可以在两台机器上随便登录。请保护好自己的私钥

如果免密写入 authorized\_keys 成功，但是通过ssh pubkey认证的时候还是有可能失败，这是因为pubkey认证要求：

-   authorized\_keys 文件权限要对
-   .ssh 文件夹权限要对
-   /home/user 文件夹权限要对 —-这个容易忽视掉

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E7%95%99%E4%B8%AA%E4%BD%9C%E4%B8%9A%EF%BC%9A%E7%AC%AC%E4%B8%80%E6%AC%A1ssh%E6%9F%90%E5%8F%B0%E6%9C%BA%E5%99%A8%E7%9A%84%E6%97%B6%E5%80%99%E6%80%BB%E6%98%AF%E5%87%BA%E6%9D%A5%E4%B8%80%E4%B8%AA%E8%AD%A6%E5%91%8A%EF%BC%8C%E9%9C%80%E8%A6%81yes%E7%A1%AE%E8%AE%A4%E6%89%8D%E8%83%BD%E5%BE%80%E4%B8%8B%E8%B5%B0%EF%BC%8C%E6%80%8E%E4%B9%88%E5%B9%B2%E6%8E%89%E4%BB%96%EF%BC%9F "留个作业：第一次ssh某台机器的时候总是出来一个警告，需要yes确认才能往下走，怎么干掉他？")留个作业：第一次ssh某台机器的时候总是出来一个警告，需要yes确认才能往下走，怎么干掉他？

> StrictHostKeyChecking=no  
> UserKnownHostsFile=/dev/null

如果按照文章操作不work，推荐就近问身边的同学。问我的话请cat 配置文件 然后把ssh -vvv user@ip (user、ip请替换成你的），再截图发给我。\*\*

测试成功的同学也请留言说下什么os、版本，以及openssl版本，我被问崩溃了

___

**这里只是帮大家入门了解ssh，掌握好这些配置文件和-vvv后有好多好玩的可以去挖掘，同时也请在留言中说出你的黑技能**

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#ssh-config-%E5%8F%82%E8%80%83%E9%85%8D%E7%BD%AE "~/.ssh/config 参考配置")~/.ssh/config 参考配置

下面是我个人常用的ssh config配置

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p></pre></td><td><pre><p><span>$</span><span>cat ~/.ssh/config</span></p><p><span>#</span><span>GSSAPIAuthentication=no</span></p><p>StrictHostKeyChecking=no</p><p><span>#</span><span>TCPKeepAlive=yes</span></p><p>CheckHostIP=no</p><p><span>#</span><span> <span>"ServerAliveInterval [seconds]"</span> configuration <span>in</span> the SSH configuration so that your ssh client sends a <span>"dummy packet"</span> on a regular interval so that the router thinks that the connection is active even <span>if</span> it<span>'s particularly quiet</span></span></p><p>ServerAliveInterval=15</p><p><span>#</span><span><span>ServerAliveCountMax=6</span></span></p><p>ForwardAgent=yes</p><p>UserKnownHostsFile /dev/null</p><p>#<span><span>reuse the same connection</span></span></p><p>ControlMaster auto</p><p>ControlPath /tmp/ssh_mux_%h_%p_%r</p><p>#<span><span>keep one connection in 72hour</span></span></p><p>ControlPersist 72h</p><p>Host 192.168.1.*</p><p>ProxyCommand ssh user@us.jump exec /usr/bin/nc %h %p 2&gt;/dev/null</p><p>Host 192.168.2.*</p><p>ProxyCommand ssh user@cn.jump exec /usr/bin/nc %h %p 2&gt;/dev/null</p><p><span>#</span><span><span>ProxyCommand /bin/nc -x localhost:12346 %h %p</span></span></p></pre></td></tr></tbody></table>

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#etc-ssh-ssh-config-%E5%8F%82%E8%80%83%E9%85%8D%E7%BD%AE "/etc/ssh/ssh_config 参考配置")/etc/ssh/ssh\_config 参考配置

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p></pre></td><td><pre><p>Host *</p><p>Protocol 2</p><p>ServerAliveInterval 30</p><p>User admin</p><p>host 10.10.55.*</p><p>ProxyCommand ssh -l admin admin.jump  exec /usr/bin/nc %h %p</p><p># uos is a hostname</p><p>Host 10.10.1.13* 192.168.2.133 uos</p><p>ProxyCommand ssh -l root -p 54900 1.1.1.1 exec /usr/bin/nc %h %p</p><p>#debug for git proxy</p><p>Host github.com</p><p>#    LogLevel DEBUG3</p><p>#    ProxyCommand ssh  -l root gfw.jump exec /usr/bin/nc %h %p</p><p>#    ProxyCommand ssh -oProxyCommand='ssh -l admin gfw.jump:22' -l root gfw.jump2 exec /usr/bin/nc %h %p</p><p>ForwardAgent yes</p><p>ForwardX11 yes</p><p>ForwardX11Trusted yes</p><p>    SendEnv LANG LC_*</p><p>    HashKnownHosts yes</p><p>    GSSAPIAuthentication no</p><p>    GSSAPIDelegateCredentials no</p><p>    Compression yes</p></pre></td></tr></tbody></table>

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%85%B6%E4%BB%96%E7%9F%A5%E8%AF%86%E7%82%B9 "其他知识点")其他知识点

参数的优先级是：命令行配置选项 > ~/.ssh/config > /etc/ssh/ssh\_config

在SSH的**身份验证阶段，SSH只支持服务端保留公钥，客户端保留私钥的方式，**所以方式只有两种：客户端生成密钥对，将公钥分发给服务端；服务端生成密钥对，将私钥分发给客户端。只不过出于安全性和便利性，一般都是客户端生成密钥对并分发公钥（阿里云服务器秘钥对–服务器将一对密钥中的公钥放在 authorized\_keys, 私钥给client登陆用）

服务器上的 /etc/ssh/ssh\_host _是用来验证服务器身份的秘钥对（对应client的 known\_hosts), _\*在主机验证阶段，服务端持有的是私钥，客户端保存的是来自于服务端的公钥。注意，这和身份验证阶段密钥的持有方是相反的。__

SSH支持多种身份验证机制，**它们的验证顺序如下：gssapi-with-mic,hostbased,publickey,keyboard-interactive,password**，但常见的是密码认证机制(password)和公钥认证机制(public key). 当公钥认证机制未通过时，再进行密码认证机制的验证。这些认证顺序可以通过ssh配置文件(注意，不是sshd的配置文件)中的指令PreferredAuthentications改变。

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%B0%B8%E4%B9%85%E9%9A%A7%E9%81%93 "永久隧道")永久隧道

大多时候隧道会失效，或者断开，我们需要有重连机制，一般可以通过autossh（需要单独安装）搞定自动重连，再配合systemd或者crond搞定永久自动重连

比如以下代码在gf开启2个远程转发端口

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p></pre></td><td><pre><p>remote_port=(30081 30082)</p><p>for port in "${remote_port[@]}"</p><p>do</p><p>    line=`ps aux |grep ssh |grep $port | wc -l`</p><p>    if [[ "$line" -lt 1 ]]; then</p><p>        autossh -M 0 -fNR gf:$port:127.0.0.1:22 root@gf</p><p>    fi;</p><p>done</p><p>line=`ps aux |grep ssh |grep 13129 | wc -l`</p><p>if [[ "$line" -lt 1 ]]; then</p><p>    nohup ssh -fNR gf:13129:172.16.1.2:3129 root@gf</p><p>fi;</p><p>#cat /etc/cron.d/jump</p><p>#* * * * * root sh /root/drds_private_cloud/jump.sh</p></pre></td></tr></tbody></table>

或者另外创建一个service服务

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p></pre></td><td><pre><p>[Unit]</p><p>Description=AutoSSH tunnel on 31081 to gf server</p><p>After=network.target</p><p>[Service]</p><p>Environment="AUTOSSH_GATETIME=0"</p><p>ExecStart=/usr/bin/autossh -M 0 -q -N -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" -NR gf:31081:172.16.1.2:22 -i /root/.ssh/id_rsa root@gf</p><p>[Install]</p><p>WantedBy=multi-user.target</p></pre></td></tr></tbody></table>

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E8%B0%83%E8%AF%95ssh%E2%80%93%E7%BB%88%E6%9E%81%E5%A4%A7%E6%8B%9B "调试ssh–终极大招")调试ssh–终极大招

好多问题我都是debug发现的

-   客户端增加参数 -vvv 会把所有流程在控制台显示出来。卡在哪个环节；密码不对还是key不对一看就知道
-   server端还可以：/usr/sbin/sshd -ddd -p 2222 在2222端口对sshd进行debug，看输出信息验证为什么pub key不能login等. 一般都是权限不对，/root 以及 /root/.ssh 文件夹的权限和owner都要对，更不要说 /root/.ssh/authorized\_keys 了

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>/usr/sbin/sshd -ddd -p 2222</p></pre></td></tr></tbody></table>

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#ssh-%E6%8F%90%E7%A4%BA%E4%BF%A1%E6%81%AF "ssh 提示信息")[ssh 提示信息](https://www.tecmint.com/ssh-warning-banner-linux/)

可以用一下脚本生成一个彩色文件，放到 /etc/motd 中就行

Basic colors are numbered:

-   1 – Red
-   2 – Green
-   3 – Yellow
-   4 – Blue
-   5 – Magenta
-   6 – Cyan
-   7 – White

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p></pre></td><td><pre><p><span>#</span><span>!/bin/sh</span></p><p>export TERM=xterm-256color</p><p>read one five fifteen rest &lt; /proc/loadavg</p><p>echo "$(tput setaf 2)</p><p>Kernel: `uname -v | awk -v OFS=' ' '{print $4, $5}'`</p><p>        \\   ^__^</p><p>         \\  (oo)\\_______</p><p>            (__)\\       )\\\/\\</p><p>                ||----w |</p><p>                ||     ||</p><p>本机器为长稳测试环境, 千万不要kill进程, 不要跑负载过重的任务</p><p>有任何需要请联系 ** 多谢!</p><p>$<span>(tput setaf 4)Load Averages......: <span>${one}</span>, <span>${five}</span>, <span>${fifteen}</span> (1, 5, 15 min)</span></p><p><span>$</span><span>(tput setaf 5)</span></p><p> ______________</p><p>本机器为长稳测试环境, 千万不要kill进程, 不要跑负载过重的任务</p><p>有任何需要请联系 ** 多谢!</p><p> --------------</p><p>        \\   ^__^</p><p>         \\  (oo)\\_______</p><p>            (__)\\       )\\\/\\</p><p>                ||----w |</p><p>                ||     ||</p><p>$<span>(tput sgr0)<span>"</span></span></p></pre></td></tr></tbody></table>

以上脚本运行结果

[![image-20210902224011450](https://plantegg.github.io/images/951413iMgBlog/image-20210902224011450.png)](https://plantegg.github.io/images/951413iMgBlog/image-20210902224011450.png)

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#sshd-Banner "sshd Banner")sshd Banner

`Banner`指定用户登录后，sshd 向其展示的信息文件（`Banner /usr/local/etc/warning.txt`），默认不展示任何内容。

或者配置：

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></pre></td><td><pre><p>cat /etc/ssh/sshd_config</p><p># no default banner path</p><p>#Banner none</p><p>#在配置文件末尾添加Banner /etc/ssh/my_banner这一行内容：</p><p>Banner /etc/ssh/my_banner</p></pre></td></tr></tbody></table>

/etc/ssh/my\_banner 中可以放置提示内容

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E9%AA%8C%E8%AF%81%E7%A7%98%E9%92%A5%E5%AF%B9 "验证秘钥对")验证秘钥对

**\-y** Read a private OpenSSH format file and print an OpenSSH public key to stdout.

> cd ~/.ssh/ ; ssh-keygen -y -f id\_rsa | cut -d’ ‘ -f 2 ; cut -d’ ‘ -f 2 id\_rsa.pub

`ssh-keygen -y -e -f <private key>`获取一个私钥并打印相应的公钥，该公钥可以直接与您可用的公钥进行比较

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#ssh-keygen "ssh-keygen")ssh-keygen

[静默生成](https://stackoverflow.com/questions/43235179/how-to-execute-ssh-keygen-without-prompt)

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></pre></td><td><pre><p>ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa &lt;&lt;&lt;y</p><p>ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa &lt;&lt;&lt;y &gt;/dev/null 2&gt;&amp;1</p><p>//修改 passphrase</p><p>ssh-keygen -p -P "12345" -N "abcde" -f .ssh/id_rsa</p><p>//ssh-keygen -p [-P old_passphrase] [-N new_passphrase] [-f keyfile]</p><p>//或者直接通过提示一步步修改：</p><p>ssh-keygen -p</p></pre></td></tr></tbody></table>

删除或者修改 passphrase

> run `ssh-keygen -p` in a terminal. It will then prompt you for a keyfile (defaulted to the correct file for me, `~/.ssh/id_rsa`), the old passphrase (enter what you have now) and the new passphrase (enter nothing).

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#ssh-agent "ssh-agent")ssh-agent

私钥设置了密码以后，每次使用都必须输入密码，有时让人感觉非常麻烦。比如，连续使用`scp`命令远程拷贝文件时，每次都要求输入密码。

`ssh-agent`命令就是为了解决这个问题而设计的，它让用户在整个 Bash 对话（session）之中，只在第一次使用 SSH 命令时输入密码，然后将私钥保存在内存中，后面都不需要再输入私钥的密码了。

第一步，使用下面的命令新建一次命令行对话。

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>$ eval `ssh-agent`</p></pre></td></tr></tbody></table>

上面命令中，`ssh-agent`会先自动在后台运行，并将需要设置的环境变量输出在屏幕上，类似下面这样。

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p></pre></td><td><pre><p>$ ssh-agent</p><p>SSH_AUTH_SOCK=/tmp/ssh-barrett/ssh-22841-agent; export SSH_AUTH_SOCK;</p><p>SSH_AGENT_PID=22842; export SSH_AGENT_PID;</p><p>echo Agent pid 22842;</p></pre></td></tr></tbody></table>

`eval`命令的作用，就是运行上面的`ssh-agent`命令的输出，设置环境变量。

第二步，在新建的 Shell 对话里面，使用`ssh-add`命令添加默认的私钥（比如`~/.ssh/id_rsa`，或`~/.ssh/id_dsa`，或`~/.ssh/id_ecdsa`，或`~/.ssh/id_ed25519`）。

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p></pre></td><td><pre><p>$ ssh-add</p><p>Enter passphrase for /home/you/.ssh/id_dsa: ********</p><p>Identity added: /home/you/.ssh/id_dsa (/home/you/.ssh/id_dsa)</p></pre></td></tr></tbody></table>

上面例子中，添加私钥时，会要求输入密码。以后，在这个对话里面再使用密钥时，就不需要输入私钥的密码了，因为私钥已经加载到内存里面了。

如果添加的不是默认私钥，`ssh-add`命令需要显式指定私钥文件。

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>$ ssh-add my-other-key-file</p></pre></td></tr></tbody></table>

上面的命令中，`my-other-key-file`就是用户指定的私钥文件。

SSH agent 程序能够将您的已解密的私钥缓存起来，在需要的时候用它来解密key chanllge返回给 SSHD [https://webcache.googleusercontent.com/search?q=cache:7OfvSBFki10J:https://www.ibm.com/developerworks/cn/linux/security/openssh/part2/+&cd=7&hl=en&ct=clnk&gl=hk](https://webcache.googleusercontent.com/search?q=cache:7OfvSBFki10J:https://www.ibm.com/developerworks/cn/linux/security/openssh/part2/+&cd=7&hl=en&ct=clnk&gl=hk) keychain介绍

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%AE%89%E8%A3%85sshd%E5%92%8Cdebug "安装sshd和debug")安装sshd和debug

sshd 有自己的一对或多对密钥。它使用密钥向客户端证明自己的身份。所有密钥都是公钥和私钥成对出现，公钥的文件名一般是私钥文件名加上后缀`.pub`。

DSA 格式的密钥文件默认为`/etc/ssh/ssh_host_dsa_key`（公钥为`ssh_host_dsa_key.pub`），RSA 格式的密钥为`/etc/ssh/ssh_host_rsa_key`（公钥为`ssh_host_rsa_key.pub`）。如果需要支持 SSH 1 协议，则必须有密钥`/etc/ssh/ssh_host_key`。

如果密钥不是默认文件，那么可以通过配置文件`sshd_config`的`HostKey`配置项指定。默认密钥的`HostKey`设置如下。

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></pre></td><td><pre><p># HostKey for protocol version 1</p><p># HostKey /etc/ssh/ssh_host_key</p><p># HostKeys for protocol version 2</p><p># HostKey /etc/ssh/ssh_host_rsa_key</p><p># HostKey /etc/ssh/ssh_host_dsa_ke</p></pre></td></tr></tbody></table>

注意，如果重装 sshd，`/etc/ssh`下的密钥都会重新生成（这些密钥对用于验证Server的身份），导致客户端重新 ssh 连接服务器时，会跳出警告，拒绝连接。为了避免这种情况，可以在重装 sshd 时，先备份`/etc/ssh`目录，重装后再恢复这个目录。

> 调试：非后台(-D)和debug(-d)模式启动sshd，同时监听2222和3333端口
> 
> sshd -D -d -p 2222 -p 3333

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#scp%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0%E6%9D%A5%E8%AE%BE%E7%BD%AEsocks%E4%BB%A3%E7%90%86 "scp可以通过命令行参数来设置socks代理")scp可以通过命令行参数来设置socks代理

> scp -o “ProxyCommand=nc -X 5 -x **\[SOCKS\_HOST\]**:**\[SOCKS\_PORT\]** %h %p” **\[LOCAL/FILE/PATH\]** **\[REMOTE\_USER\]**@**\[REMOTE\_HOST\]**:**\[REMOTE/FILE/PATH\]**

其中\[SOCKS\_HOST\]和\[SOCKS\_PORT\]是socks代理的LOCAL\_ADDRESS和LOCAL\_PORT。\[LOCAL/FILE/PATH\]、\[REMOTE\_USER\]、\[REMOTE\_HOST\]和\[REMOTE/FILE/PATH\]分别是要复制文件的本地路径、要复制到的远端主机的用户名、要复制到的远端主机名、要复制文件的远端路径，这些参数与不使用代理时一样。“ProxyCommand=nc”表示当前运行命令的主机上需要有nc命令。

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#ProxyCommand "ProxyCommand")ProxyCommand

> Specifies the proxy command for the connection. This command is launched prior to making the connection to Hostname. %h is replaced with the host defined in HostName and %p is replaced with 22 or is overridden by a Port directive.

在ssh连接目标主机前先执行ProxyCommand中的命令，比如 .ssh/config 中有如下配置

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></pre></td><td><pre><p>host remote-host</p><p>ProxyCommand ssh -l root -p 52146 1.2.3.4 exec /usr/bin/nc %h %p</p><p>//以上配置等价下面的命令</p><p>ssh -o ProxyCommand="ssh -l root -p 52146 1.2.3.4 exec /usr/bin/nc %h %p" remote-host</p><p>//or 等价</p><p>ssh -o ProxyCommand="ssh -l root -p 52146 -W %h:%p 1.2.3.4 " remote-host</p><p>//or 等价 debug1: Setting implicit ProxyCommand from ProxyJump: ssh -l root -p 52146 -vvv -W '[%h]:%p' 1.2.3.4</p><p>ssh -J root@1.2.3.4:52146 remote-host</p></pre></td></tr></tbody></table>

如上配置指的是，如果执行ssh remote-host 命中host规则，那么先执行命令 ssh -l root -p 52146 1.2.3.4 exec /usr/bin/nc 同时把remote-host和端口(默认22)传给nc

ProxyCommand和ProxyJump很类似，ProxyJump使用：

<table><tbody><tr><td><pre><p>1</p><p>2</p></pre></td><td><pre><p>//ssh到centos8机器上，走的是gf这台跳板机，本地一般和centos8不通</p><p>ssh -J gf:22 centos8</p></pre></td></tr></tbody></table>

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95 "加密算法")[加密算法](http://www.openssh.com/legacy.html)

列出本地所支持默认的加密算法

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p></pre></td><td><pre><p>#ssh -Q key                                                            </p><p>ssh-ed25519</p><p>ssh-ed25519-cert-v01@openssh.com</p><p>ssh-rsa</p><p>ssh-dss</p><p>ecdsa-sha2-nistp256</p><p>ecdsa-sha2-nistp384</p><p>ecdsa-sha2-nistp521</p><p>ssh-rsa-cert-v01@openssh.com</p><p>ssh-dss-cert-v01@openssh.com</p><p>ecdsa-sha2-nistp256-cert-v01@openssh.com</p><p>ecdsa-sha2-nistp384-cert-v01@openssh.com</p><p>ecdsa-sha2-nistp521-cert-v01@openssh.com</p><p>ssh -Q cipher       # List supported ciphers</p><p>ssh -Q mac          # List supported MACs</p><p>ssh -Q key          # List supported public key types</p><p>ssh -Q kex          # List supported key exchange algorithms</p></pre></td></tr></tbody></table>

比如连服务器报如下错误：

<table><tbody><tr><td><pre><p>1</p><p>2</p></pre></td><td><pre><p>debug1: kex: algorithm: (no match)</p><p>Unable to negotiate with server port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1,diffie-hellman-group14-sha1</p></pre></td></tr></tbody></table>

表示服务端支持 diffie-hellman-group1-sha1,diffie-hellman-group14-sha1 加密，但是client端不支持，那么可以指定算法来强制client端使用某种和server一致的加密方式

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></pre></td><td><pre><p>ssh  -oKexAlgorithms=+diffie-hellman-group14-sha1 -l user</p><p>或者config中配置：</p><p>host server_ip</p><p>KexAlgorithms +diffie-hellman-group1-sha1</p></pre></td></tr></tbody></table>

如果仍然报以下错误：

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></pre></td><td><pre><p>debug2: first_kex_follows 0</p><p>debug2: reserved 0</p><p>debug1: kex: algorithm: diffie-hellman-group14-sha1</p><p>debug1: kex: host key algorithm: (no match)</p><p>Unable to negotiate with server_ip port 22: no matching host key type found. Their offer: ssh-rsa</p></pre></td></tr></tbody></table>

那么可以配置来解决：

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p></pre></td><td><pre><p>Host *</p><p>    HostKeyAlgorithms +ssh-rsa</p><p>    PubkeyAcceptedKeyTypes +ssh-rsa</p></pre></td></tr></tbody></table>

When an SSH client connects to a server, each side offers lists of connection parameters to the other. These are, with the corresponding [ssh\_config](https://man.openbsd.org/ssh_config.5) keyword:

-   `KexAlgorithms`: the key exchange methods that are used to generate per-connection keys
-   `HostkeyAlgorithms`: the public key algorithms accepted for an SSH server to authenticate itself to an SSH client
-   `Ciphers`: the ciphers to encrypt the connection
-   `MACs`: the message authentication codes used to detect traffic modification

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%97%A0%E6%89%80%E4%B8%8D%E8%83%BD%E7%9A%84-SSH-%E4%B8%89%E5%A4%A7%E8%BD%AC%E5%8F%91%E6%A8%A1%E5%BC%8F "无所不能的 SSH 三大转发模式")无所不能的 SSH 三大转发模式

了解完前面的一些小知识，再来看看无所不能的三大杀招。上面的各种代理基本都是由这三种转发模式实现的。

SSH能够做动态转发、本地转发、远程转发。先简要概述下他们的特点和使用场景

**三个转发模式的比较：**

-   动态转发完全可以代替本地转发，只是动态转发是`socks5协议`，本地转发是tcp协议
-   本地转发完全是把动态转发特例化到访问某个固定目标的转发
-   远程转发是启动转端口的机器同时连上两端的两个机器，把本来不连通的两端拼接起来，中间显得多了个节点。
-   三个转发模式可以串联使用

动态转发常用来科学上网，本地转发用来打洞，这两种转发启动的端口都是在本地；远程转发也是打洞的一种，只不过启用的端口在远程机器上。

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%8A%A8%E6%80%81%E8%BD%AC%E5%8F%91-D-SOCKS5-%E5%8D%8F%E8%AE%AE "动态转发 (-D)   SOCKS5 协议")动态转发 (-D) SOCKS5 协议

动态转发指的是，本机与 SSH 服务器之间创建了一个加密连接，然后本机内部针对某个端口的通信，都通过这个加密连接转发。它的一个使用场景就是，访问所有外部网站，都通过 SSH 转发。

动态转发需要把本地端口绑定到 SSH 服务器。**至于 SSH 服务器要去访问哪一个网站，完全是动态的，取决于原始通信，所以叫做动态转发**。

动态的意思就是：需要访问的目标、端口还不确定。后面要讲的本地转发、远程转发都是针对具体IP、port的转发。

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p></pre></td><td><pre><p><span>$</span><span> ssh -D 4444 ssh-server -N</span></p><p>//或者如下方式：</p><p>nohup ssh -qTfnN -D *:13658 root@jump vmstat 10  &gt;/dev/null 2&gt;&amp;1</p></pre></td></tr></tbody></table>

注意，这种转发采用了 SOCKS5 协议。访问外部网站时，需要把 HTTP 请求转成 SOCKS5 协议，才能把本地端口的请求转发出去。`-N`参数表示，这个 SSH 连接不能执行远程命令，只能充当隧道。

[![image-20210913143129749](https://plantegg.github.io/images/951413iMgBlog/image-20210913143129749.png)](https://plantegg.github.io/images/951413iMgBlog/image-20210913143129749.png)

下面是 ssh 隧道建立后的一个**使用实例**。

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p></pre></td><td><pre><p>curl -x socks5://localhost:4444 http://www.example.com</p><p>or</p><p>curl --socks5-hostname localhost:4444 https://www.twitter.com</p></pre></td></tr></tbody></table>

上面命令中，curl 的`-x`参数指定代理服务器，即通过 SOCKS5 协议的本地`3000`端口，访问`http://www.example.com`。

官方文档关于 -D的介绍

> \-D \[bind\_address:\]port  
> Specifies a local “dynamic” application-level port forwarding. This works by allocat‐  
> ing a socket to listen to port on the local side, optionally bound to the specified  
> bind\_address. Whenever a connection is made to this port, the connection is forwarded  
> over the secure channel, and the application protocol is then used to determine where  
> to connect to from the remote machine. Currently the SOCKS4 and SOCKS5 protocols are  
> supported, and ssh will act as a SOCKS server. Only root can forward privileged ports.  
> Dynamic port forwardings can also be specified in the configuration file.

特别注意，如果ssh -D 要启动的本地port已经被占用了是不会报错的，但是实际socks代理会没启动成功

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E6%9C%AC%E5%9C%B0%E8%BD%AC%E5%8F%91-L "本地转发 (-L)")本地转发 (-L)

本地转发（local forwarding）指的是，SSH 服务器作为中介的跳板机，建立本地计算机与特定`目标网站`之间的加密连接。本地转发是在本地计算机的 SSH 客户端建立的转发规则。

典型使用场景就是，打洞，经过跳板机访问无法直接连通的服务。

它会指定一个本地端口（local-port），所有发向那个端口的请求，都会转发到 SSH 跳板机（ssh-server），然后 SSH 跳板机作为中介，将收到的请求发到目标服务器（target-host）的目标端口（target-port）。

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p><span>$</span><span> ssh -L :<span>local</span>-port:target-host:target-port ssh-server  //target-host是ssh-server的target-host, target-host 域名解析、路由都是由ssh-server完成</span></p></pre></td></tr></tbody></table>

上面命令中，`-L`参数表示本地转发，`local-port`是本地端口，`target-host`是你想要访问的目标服务器，`target-port`是目标服务器的端口，`ssh-server`是 SSH 跳板机。当你访问localhost:local-port 的时候会通过ssh-server把请求转给target-host:target-port

[![img](https://plantegg.github.io/images/951413iMgBlog/vgaakWbKC9OPXugAR9oPnotTq1L4jBRDEg.JPG)](https://plantegg.github.io/images/951413iMgBlog/vgaakWbKC9OPXugAR9oPnotTq1L4jBRDEg.JPG)

上图对应的命令是：

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>ssh -L 53682:remote-server:53682 ssh-server</p></pre></td></tr></tbody></table>

然后，访问本机的53682端口，就是访问`remote-server`的53682端口.

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>$ curl http://localhost:53682</p></pre></td></tr></tbody></table>

注意，**本地端口转发采用 HTTP 协议，不用转成 SOCKS5 协议**。如果需要HTTP的动态代理，可以先起socks5动态代理，然后再起一个本地转发给动态代理的socks5端口，这样就有一个HTTP代理了，能给yum、docker之类的使用。

这个命令最好加上`-N`参数，表示不在 SSH 跳板机执行远程命令，让 SSH 只充当隧道。另外还有一个`-f`参数表示 SSH 连接在后台运行。

如果经常使用本地转发，可以将设置写入 SSH 客户端的用户个人配置文件。

<table><tbody><tr><td><pre><p>1</p><p>2</p></pre></td><td><pre><p>Host test.example.com</p><p>LocalForward client-IP:client-port server-IP:server-port</p></pre></td></tr></tbody></table>

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E8%BF%9C%E7%A8%8B%E8%BD%AC%E5%8F%91-R "远程转发(-R)")远程转发(-R)

远程端口指的是在远程 SSH 服务器建立的转发规则。主要是执行ssh转发的机器别人连不上，所以需要一台client能连上的机器当远程转发端口，要不就是本地转发了。

由于本机无法访问内网 SSH 跳板机，就无法从外网发起 SSH 隧道，建立端口转发。必须反过来，从 SSH 跳板机发起隧道，建立端口转发，这时就形成了远程端口转发。

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>ssh -fNR 30.1.2.3:30081:166.100.64.1:3128 root@30.1.2.3 -p 2728</p></pre></td></tr></tbody></table>

上面的命令，首先需要注意，**不是在30.1.2.3 或者166.100.64.1 上执行的，而是找一台能联通 30.1.2.3 和166.100.64.1的机器来执行**，在执行前Remote clients能连上 30.1.2.3 但是 30.1.2.3 和 166.100.64.1 不通，所以需要一个中介将 30.1.2.3 和166.100.64.1打通，这个中介就是下图中的MobaXterm所在的机器，命令在MobaXterm机器上执行

[![image-20210913163036410](https://plantegg.github.io/images/951413iMgBlog/image-20210913163036410.png)](https://plantegg.github.io/images/951413iMgBlog/image-20210913163036410.png)

执行上面的命令以后，跳板机30.1.2.3 到166.100.64.1的隧道已经建立了，这个隧道是依赖两边都能连通的MobaXterm机器。然后，就可以从Remote Client访问目标服务器了，即在Remote Client上执行下面的命令。

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>$ curl http://30.1.2.3:30081</p></pre></td></tr></tbody></table>

执行上面的命令以后，命令就会输出服务器 166.100.64.1 的3128端口返回的内容。

如果经常执行远程端口转发，可以将设置写入 SSH 客户端的用户个人配置文件。

<table><tbody><tr><td><pre><p>1</p><p>2</p></pre></td><td><pre><p>Host test.example.com</p><p>RemoteForward local-IP:local-port target-ip:target-port</p></pre></td></tr></tbody></table>

注意远程转发需要：

> 1.  sshd\_config里要打开`AllowTcpForwarding`选项，否则`-R`远程端口转发会失败。
> 2.  默认转发到远程主机上的端口绑定的是`127.0.0.1`，[如要绑定`0.0.0.0`需要打开sshd\_config里的`GatewayPorts`选项(然后ssh -R 后加上\*:port )](https://serverfault.com/questions/997124/ssh-r-binds-to-127-0-0-1-only-on-remote)。这个选项如果由于权限没法打开也有办法，可配合`ssh -L`将端口绑定到`0.0.0.0`。

开通远程转发后，如果需要动态代理（比如访问所有web服务），那么可以在30081端口机器上(30.1.2.3)执行：

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>nohup ssh -qTfnN -D *:13658 root@127.0.0.1 -p 30081 vmstat 10  &gt;/dev/null 2&gt;&amp;1</p></pre></td></tr></tbody></table>

表示在30081机器上(30.1.2.3)启动了一个socks5动态代理服务

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E8%B0%83%E8%AF%95%E8%BD%AC%E5%8F%91%E3%80%81%E4%BB%A3%E7%90%86%E6%98%AF%E5%90%A6%E8%83%BD%E8%81%94%E9%80%9A "调试转发、代理是否能联通")调试转发、代理是否能联通

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#curl "curl")[curl](https://docs.google.com/document/d/1lSeScMYw9I7Pj_OgXEugfwp-taeF4b72WF_CGp4ey5s/edit#heading=h.n7jhdk88a6rk)

> curl -I –socks5-hostname 127.0.0.1:13659 twitter.com
> 
> curl -x socks5://localhost:13659 twitter.com

Suppose you have a socks5 proxy running on localhost:8001.

[In curl >= 7.21.7, you can use](https://blog.emacsos.com/use-socks5-proxy-in-curl.html)

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>curl -x socks5h://localhost:8001 http://www.google.com/</p></pre></td></tr></tbody></table>

In curl >= 7.18.0, you can use

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>curl --socks5-hostname localhost:8001 http://www.google.com/</p></pre></td></tr></tbody></table>

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#wget "wget")wget

**指定命令行参数**,通过命令行指定HTTP代理服务器的方式如下：

> wget -Y on -e “http\_proxy=[http://\*\*\[HTTP\_HOST\]\*\*:\*\*\[HTTP\_PORT\]\*\*](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89//http://**[HTTP_HOST]**:**[HTTP_PORT]**)“ [http://facebook.com/其中：\[HTTP\_HOST\]和\[HTTP\_PORT\]是http](http://facebook.com/%E5%85%B6%E4%B8%AD%EF%BC%9A[HTTP_HOST]%E5%92%8C[HTTP_PORT]%E6%98%AFhttp) proxy的ADDRESS和PORT。

\-Y表示是否使用代理，on表示使用代理。

\-e执行后面跟的命令，相当于在.wgetrc配置文件中添加了一条命令，将http\_proxy设置为需要使用的代理服务器。

wget –limit-rate=2.5k 限制下载速度，进行测试

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#PKI-Public-Key-Infrastructure-%E8%AF%81%E4%B9%A6 "PKI (Public Key Infrastructure)证书")PKI (Public Key Infrastructure)证书

X.509 只是一种常用的证书格式，一般以PEM编码，PEM 编码的证书通常以 **`.pem`、`.crt` 或 `.cer`** 为后缀。再次提醒，这只是“通常”情况，实际上某些工具可能并不遵循这些惯例。通过pem证书可以访问需要认证的https服务(比如etcd、apiserver等)

-   **ASN.1 用于定义数据类型**，例如证书（certificate）和秘钥（key）——就像用 JSON 定义一个 request body —— X.509 用 ASN.1 定义。
-   DER 是一组将 ASN.1 编码成二进制（比特和字节）的编码规则（encoding rules）。
-   PKCS#7 and PKCS#12 是比 X.509 更大的数据结构（封装格式），也用 ASN.1 定义，其中能包含除了证书之外的其他东西。二者分别在 Java 和 Microsoft 产品中使用较多。
-   DER 编码之后是二进制数据，不方便复制粘贴，因此大部分证书都是用 PEM 编码的，它用 base64 对 DER 进行编码，然后再加上自己的 label。
-   私钥通常用是 PEM 编码的 PKCS#8 对象，但有时也会用密码来加密。

通过命令 cat /etc/kubernetes/pki/ca.crt | openssl x509 -text 也可以得到下图信息

[![image](https://plantegg.github.io/images/951413iMgBlog/step-certificate-inspect.png)](https://plantegg.github.io/images/951413iMgBlog/step-certificate-inspect.png)

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%85%AC%E9%92%A5%E3%80%81%E7%A7%81%E9%92%A5%E5%B8%B8%E8%A7%81%E6%89%A9%E5%B1%95%E5%90%8D "公钥、私钥常见扩展名")公钥、私钥常见扩展名

-   公钥：`.pub` or `.pem`，`ca.crt`
-   私钥：`.prv,` `.key`, or `.pem` , `ca.key`。

### [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E8%AF%81%E4%B9%A6%E7%94%9F%E6%88%90%E8%BF%87%E7%A8%8B%E6%BC%94%E7%A4%BA "证书生成过程演示")证书生成过程演示

并不是所有的场景都需要向这些大型的 CA 机构申请公钥证书，在任何一个企业，组织或是团体内都可以自己形这样的“小王国”，也就是说，你可以自行生成这样的证书，只需要你自己保证自己的生成证书的私钥的安全，以及不需要扩散到整个互联网。下面，我们用 `openssl`命令来演示这个过程。

1）生成 CA 机构的证书（公钥） `ca.crt` 和私钥 `ca.key`

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></pre></td><td><pre><p>openssl req -newkey rsa:2048 \</p><p>    -new -nodes -x509 \</p><p>    -days 365 \</p><p>    -out ca.crt \</p><p>    -keyout ca.key \</p><p>    -subj "/C=SO/ST=Earth/L=Mountain/O=CoolShell/OU=HQ/CN=localhost"</p></pre></td></tr></tbody></table>

2) 生成 alice 的私钥

<table><tbody><tr><td><pre><p>1</p></pre></td><td><pre><p>openssl genrsa -out alice.key 2048</p></pre></td></tr></tbody></table>

3）生成 Alice 的 CSR – Certificate Signing Request

<table><tbody><tr><td><pre><p>1</p><p>2</p></pre></td><td><pre><p>openssl req -new -key alice.key -days 365 -out alice.csr \</p><p>    -subj "/C=CN/ST=Beijing/L=Haidian/O=CoolShell/OU=Test/CN=localhost.alice"</p></pre></td></tr></tbody></table>

4）使用 CA 给 Alice 签名证书

<table><tbody><tr><td><pre><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></pre></td><td><pre><p>openssl x509  -req -in alice.csr \</p><p>    -extfile &lt;(printf "subjectAltName=DNS:localhost.alice") \ </p><p>    -CA ca.crt -CAkey ca.key  \</p><p>    -days 365 -sha256 -CAcreateserial \</p><p>    -out alice.crt</p></pre></td></tr></tbody></table>

## [](https://plantegg.github.io/2019/06/02/%E5%8F%B2%E4%B8%8A%E6%9C%80%E5%85%A8_SSH_%E6%9A%97%E9%BB%91%E6%8A%80%E5%B7%A7%E8%AF%A6%E8%A7%A3--%E6%94%B6%E8%97%8F%E4%BF%9D%E5%B9%B3%E5%AE%89/#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99%EF%BC%9A "参考资料：")参考资料：

[http://docs.corp-inc.com/pages/editpage.action?pageId=203555361](http://docs.corp-inc.com/pages/editpage.action?pageId=203555361)  
[https://wiki.archlinux.org/index.php/SSH\_keys\_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87](https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

[https://wangdoc.com/ssh/key.html](https://wangdoc.com/ssh/key.html)

[https://robotmoon.com/ssh-tunnels/](https://robotmoon.com/ssh-tunnels/)

[通过SSH动态转发来建立Socks代以及各种场景应用案例](https://blog.gwlab.page/vpn-over-ssh-the-socks-proxy-8a8d7bdc7028)

[https://daniel.haxx.se/blog/2020/05/26/curl-ootw-socks5/](https://daniel.haxx.se/blog/2020/05/26/curl-ootw-socks5/)

[SSH Performance](http://www.allanjude.com/bsd/AsiaBSDCon2017_-_SSH_Performance.pdf)

[Why when I transfer a file through SFTP, it takes longer than FTP?](https://stackoverflow.com/questions/8849240/why-when-i-transfer-a-file-through-sftp-it-takes-longer-than-ftp)

[一行代码解决scp在Internet传输慢的问题](https://zhuanlan.zhihu.com/p/413732839)

[关于证书（certificate）和公钥基础设施（PKI）的一切](https://www.cnxct.com/everything-about-pki-zh/)

[网络数字身份认证术](https://coolshell.cn/articles/21708.html)