2021-09-06 22:27:20   [Cyberbolt](https://www.cyberlight.xyz/user/Cyberbolt)

此方案可能是全网最快的 宝塔面板 部署方案。 复习计算机网络时轻度折腾了 Linux、Docker、路由器 等，竟突然想到 Docker 部署宝塔面板的完美解决方案。在此使用 Python 开发并制作了宝塔面板 Docker 镜像。

您可能存在疑问，宝塔面板为何不直接装到系统中？我们使用不同的服务器，往往产生较大环境差异，CentOS、Debian、Ubuntu？每次新装宝塔面板，都需要选择对应系统的脚本，漫长的安装过程不提，初次登录需要使用系统生成的账号和密码，每次搭建需要重新配置部分环境，同时保存好每个服务器的 url、账户、密码。如果您和我一样，较高频率地使用服务器开发，便感到十分繁琐。有没有一个工具能一键秒建宝塔面板，并在搭建过程中配置好账户信息？由此便得到了今天的主角，[cyberbolt/baota](https://hub.docker.com/r/cyberbolt/baota)。仍然有疑问，宝塔面板运行的生产环境，如 MySQL，并不适合将数据全部存储在容器中，实际工作中可能遇到安全隐患。在此我提供了 方案二，生产环境中将容器内宝塔面板的运行目录挂载至宿主机硬盘中。您会发现，无论是 方案一 还是 二，对常用 Docker 的开发者来说，都远比直接安装宝塔便捷。在 Docker 中运行宝塔面板，由于容器的隔离性，唯一的不同便是 舍弃了部分系统功能，您直接通过宝塔修改系统，只能作用于容器，并不会影响宿主机系统，但这恰好也是容器的优点。而常用的 Nginx、MySQL(MariaDB) 等均能完美使用。

**Docker 部署宝塔面板**

此方案可能是全网最快的 宝塔面板 部署方案。该镜像基于 宝塔Linux正式版 7.7.0（官方纯净版，可升级） 制作。维护脚本使用 Python 开发，源码和 Dockerfile 均已上传至 [GitHub](https://github.com/Cyberbolt/baota)（欢迎您的 Star）。

本镜像仅保留了最精简的 宝塔面板，未安装任何插件。初始化容器后，您可以根据需要选择安装插件。"Simple is better than complex!" 此外，如果您在生产环境下部署宝塔面板，请务必参考 **方案二** 创建容器。

使用方法如下:

(注：为了方便部署，该镜像去除了安全入口，您可以自行配置)

**方案一（最快化部署）**

```
docker run -itd --net=host --restart=always --name baota cyberbolt/baota:latest -port 端口号 -username 用户名 -password 密码
```

示例如

```
docker run -itd --net=host --restart=always --name baota cyberbolt/baota:latest -port 8888 -username cyberbolt -password abc123456
```

\--net=host : 容器和主机使用同一网络

\--restart=always: 守护进程，容器挂掉将自动重启

\-port : 填写宝塔面板运行的端口号

\-username: 填写宝塔面板的用户名

\-password : 填写宝塔面板的密码

```
该方法的登录方式:

登陆地址: http://{{服务器的ip地址}}:{{您输入的端口号}}

账号: 您填写的用户名

密码: 您填写的密码

```

**如果您未自定义用户名和密码，直接使用的如下命令**

```
docker run -itd --net=host --restart=always --name baota cyberbolt/baota:latest
```

**宝塔面板也会自动创建，此时可通过默认信息登录，默认信息为**

```

登陆地址: http://{{服务器的ip地址}}:8888

账号: cyber

密码: abc12345

```

**方案二（生产环境部署）**

生产环境中，为了避免极小概率的数据丢失，我们将容器内的宝塔文件映射到宿主机的目录中（您之后安装的 Nginx、MySQL 等服务均会挂载到宿主机目录）。该方法是 Docker 部署宝塔面板的最优方案，可以在生产环境中运行。

首先按最简方案创建一个测试容器（为保存宝塔文件到宿主机目录中）

输入命令创建测试容器（这里仅为测试容器，为避免出错，后面几步请原封不动地复制粘贴）

```
docker run -itd --net=host --name baota-test cyberbolt/baota:latest -port 26756 -username cyberbolt -password abc123456
```

将 Docker 容器中的 /www 目录 拷贝至宿主机的 /www

```
docker cp baota-test:/www /www
```

拷贝完成后删除创建的测试容器

```
docker stop baota-test && docker rm baota-test
```

创建宝塔面板容器，并将宿主机目录映射至容器中（自行输入面板的 端口号、用户名 和 密码 后即可完成部署）

```
docker run -itd -v /www:/www --net=host --restart=always --name baota cyberbolt/baota:latest -port 端口号 -username 用户名 -password 密码
```

示例如

```
docker run -itd -v /www:/www --net=host --restart=always --name baota cyberbolt/baota:latest -port 8888 -username cyberbolt -password abc123456
```

\--net=host : 容器和主机使用同一网络

\--restart=always: 守护进程，容器挂掉将自动重启

\-port : 填写宝塔面板运行的端口号

\-username: 填写宝塔面板的用户名

\-password : 填写宝塔面板的密码

```
该方法的登录方式:

登陆地址: http://{{服务器的ip地址}}:{{您输入的端口号}}

账号: 您填写的用户名

密码: 您填写的密码

```

部署成功！

电光笔记官网 [https://www.cyberlight.xyz/](https://www.cyberlight.xyz/)

![](https://www.cyberlight.xyz/static/images/default_profile_picture/default.png)

雪人  
2021-10-30 04:01:48

博主，你好。

执行这个的时候docker容器一直restarting

```
docker run -itd -v /www:/www --net=host --restart=always \
--name baota cyberbolt/baota \
-port 8888 -username cyberbolt -password abc123456
```

这是logs

```
➜  ~ docker logs e862325b7baf
正在设置面板端口
Traceback (most recent call last):
  File "script.py", line 95, in <module>
    main()
  File "script.py", line 79, in main
    bt_init(data['port'], data['username'], data['password'])
  File "script.py", line 35, in bt_init
    child.expect('已将面板端口修改'.encode('utf-8'))
  File "/usr/local/lib/python3.7/dist-packages/pexpect/spawnbase.py", line 344, in expect
    timeout, searchwindowsize, async_)
  File "/usr/local/lib/python3.7/dist-packages/pexpect/spawnbase.py", line 372, in expect_list
    return exp.expect_loop(timeout)
  File "/usr/local/lib/python3.7/dist-packages/pexpect/expect.py", line 179, in expect_loop
    return self.eof(e)
  File "/usr/local/lib/python3.7/dist-packages/pexpect/expect.py", line 122, in eof
    raise exc
pexpect.exceptions.EOF: End Of File (EOF). Exception style platform.
<pexpect.pty_spawn.spawn object at 0xffffbd165c88>
command: /usr/bin/bt
args: ['/usr/bin/bt']
buffer (last 100 chars): b''
before (last 100 chars): b'9520\r\n|-\xe9\x94\x99\xe8\xaf\xaf\xef\xbc\x8c\xe4\xb8\x8e\xe9\x9d\xa2\xe6\x9d\xbf\xe5\xbd\x93\xe5\x89\x8d\xe7\xab\xaf\xe5\x8f\xa3\xe4\xb8\x80\xe8\x87\xb4\xef\xbc\x8c\xe6\x97\xa0\xe9\x9c\x80\xe4\xbf\xae\xe6\x94\xb9\r\n'
after: <class 'pexpect.exceptions.EOF'>
match: None
match_index: None
exitstatus: None
flag_eof: True
pid: 7
child_fd: 5
closed: False
timeout: 30
delimiter: <class 'pexpect.exceptions.EOF'>
logfile: None
logfile_read: None
logfile_send: None
maxread: 2000
ignorecase: False
searchwindowsize: None
delaybeforesend: 0.05
delayafterclose: 0.1
delayafterterminate: 0.1
searcher: searcher_re:
    0: re.compile(b'\xe5\xb7\xb2\xe5\xb0\x86\xe9\x9d\xa2\xe6\x9d\xbf\xe7\xab\xaf\xe5\x8f\xa3\xe4\xbf\xae\xe6\x94\xb9')
```

___

![](https://www.cyberlight.xyz/static/images/profile_picture/Cyberbolt/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20200415171815.jpg)

Cyberbolt  
2021-10-30 10:34:23

回复 雪人: 您好，宿主机的 /www 是怎么得到的？请您复制给我这一步(宿主机得到 /www 目录)的命令，谢谢

![](https://www.cyberlight.xyz/static/images/profile_picture/Cyberbolt/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20200415171815.jpg)

Cyberbolt  
2021-10-30 10:35:51

回复 雪人: 您在 CyberLight 的邮箱填写错误，可能收不到回复。建议在 GitHub 留言

![](http://q1.qlogo.cn/g?b=qq&nk=8021907&s=100)

magellan  
2022-11-01 17:38:02

回复 Cyberbolt: 我和雪人遇到同样的问题了，也是一直重启。请问有了解决办法没有？

![](http://q1.qlogo.cn/g?b=qq&nk=8021907&s=100)

magellan  
2022-11-01 17:38:04

回复 Cyberbolt: 我和雪人遇到同样的问题了，也是一直重启。请问有了解决办法没有？

![](http://q1.qlogo.cn/g?b=qq&nk=8021907&s=100)

magellan  
2022-11-01 17:39:58

回复 Cyberbolt: 我和雪人遇到同样的问题了，也是一直重启。请问有了解决办法没有？

![](http://q1.qlogo.cn/g?b=qq&nk=8021907&s=100)

magellan  
2022-11-01 17:41:07

回复 Cyberbolt: 我和雪人遇到同样的问题了，也是一直重启。请问有了解决办法没有？