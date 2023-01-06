## **1\. EMQ X**

## **EMQ X 与 emqttd 的关系**

EMQ X 全称 Erlang/Enterprise/Elastic MQTT Broker，它是基于 Erlang/OTP 语言平台开发，支持大规模连接和分布式集群，发布订阅模式的百万级开源 MQTT 消息服务器。

说起 EMQ-X，其它的前身就是 emqttd 消息服务器，自 emqttd 3.0 版本起更名为 EMQ-X。

![](https://ask.qcloudimg.com/http-save/yehe-1088047/qwajnyr54z.jpeg?imageView2/2/w/1620)

EMQ X

## **EMQ X 支持的协议**

EMQ X 消息服务器**完整支持 MQTT V3.1/V3.1.1/V5.0 版本协议规范**，并扩展支持 MQTT-SN 、WebSocket、CoAP、LwM2M、Stomp 以及私有 TCP/UDP 协议。

其中，MQTT-SN、CoAP 协议已在 2.0-rc.1 版本发布，LwM2M、LoRaWan 协议在 3.0 版本中发布。

## **EMQ X 支持的部署平台**

EMQ X 的每个版本都提供如下平台的软件包：

-   Linux：CentOS、Ubuntu、Debian、FreeBSD、OpenSUSE
-   MacOS
-   Windows

部署时直接在**官方下载链接\[1\]**下载 zip 压缩包，解压后直接运行即可。

另外，EMQ X 还**提供** [**Docker**](https://cloud.tencent.com/product/tke?from=10680) **镜像**，可以在 Docker 中直接部署。

关于各个平台的部署方法，可以参考**官方部署文档\[2\]**，本文中只讲述如何在 Linux 服务器上使用 zip 方式部署和使用 docker 方式部署，本文所使用的是腾讯[云服务器](https://cloud.tencent.com/product/cvm?from=10680)，配置 1 核 2G，系统是 Ubuntu 18.04 64 位。

## **2\. 使用 zip 压缩包部署**

> 使用 v3.1.0 版本，其它高版本测试有问题！

## **获取 zip 包下载链接**

访问**官方下载链接\[3\]**，选择需要下载的版本和系统，之后复制下载链接：

![](https://ask.qcloudimg.com/http-save/yehe-1088047/gg63f7qpml.png?imageView2/2/w/1620)

获取下载链接

## **下载软件包**

登录云服务器，使用 wget 工具下载：

比如这里我的下载命令是：

```
wget https://www.emqx.io/downloads/broker/v3.1.0/emqx-ubuntu18.04-v3.1.0.zip
```

![](https://ask.qcloudimg.com/http-save/yehe-1088047/7riqrki1j3.png?imageView2/2/w/1620)

下载zip包

## **解压 zip 包**

zip 包需要使用 unzip 工具解压，使用如下命令查询 unzip 是否安装：

我的电脑上已安装，所以查询结果如图：

![](https://ask.qcloudimg.com/http-save/yehe-1088047/b5ibmvc4ea.png?imageView2/2/w/1620)

unzip查询结果

如果没有查询到，请使用如下命令安装：

```
sudo apt-get install unzip
```

确保已经安装 unzip 之后，解压刚刚下载的压缩包：

![](https://ask.qcloudimg.com/http-save/yehe-1088047/72yqblsa4g.png?imageView2/2/w/1620)

解压zip包

## **启动 EMQ X**

进入解压出的文件夹：

然后使用如下命令启动 emqx：

启动成功之后如图：

![](https://ask.qcloudimg.com/http-save/yehe-1088047/4eaaektv09.png?imageView2/2/w/1620)

启动成功

查询一下 emqx 的状态，检查一下是否真正成功启动：

```
sudo ./bin/emqx_ctl status
```

![](https://ask.qcloudimg.com/http-save/yehe-1088047/7i5tjyd79h.png?imageView2/2/w/1620)

查询启动成功

> 特别注意：如果云服务器默认有安全组配置（阿里云），或者开启了宝塔面板，一定要记得放行如下 TCP 端口。

EMQ X 消息服务器默认占用的 TCP 端口包括:

|  |  |
| --- | --- |
| 
1883



 | 

MQTT 协议端口



 |
| 

8883



 | 

MQTT/SSL 端口



 |
| 

8083



 | 

MQTT/WebSocket 端口



 |
| 

8080



 | 

HTTP API 端口



 |
| 

18083



 | 

Dashboard 管理控制台端口



 |

> 接下来可以跳至第 4 节，登录后面控制面板。

## **3\. 使用 docker 部署**

## **安装 docker**

参考我的博客：**Docker-ce 最新版在 Ubuntu18.04 上的安装、更新、卸载方法（存储库方式）\[4\]**。（今天的第二篇文章）

## **获取 docker 镜像**

通过 Docker Hub 获取 docker 镜像：

```
sudo docker pull emqx/emqx:v3.1.0
```

![](https://ask.qcloudimg.com/http-save/yehe-1088047/4pgkfrvmac.png?imageView2/2/w/1620)

拉取docker镜像

## **启动 docker 容器**

使用如下命令启动 docker 容器：

```
sudo docker run -d --name emqx31 -p 1883:1883 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 emqx/emqx:v3.1.0
```

启动之后如图：

![](https://ask.qcloudimg.com/http-save/yehe-1088047/kyafgxszbn.png?imageView2/2/w/1620)

启动docker容器

启动之后查看 docker 进程，检查一下是否运行：

查看docker运行进程

> 特别注意：如果云服务器默认有安全组配置（阿里云），或者开启了宝塔面板，一定要记得放行如下 TCP 端口。

EMQ X 消息服务器默认占用的 TCP 端口包括:

|  |  |
| --- | --- |
| 
1883



 | 

MQTT 协议端口



 |
| 

8883



 | 

MQTT/SSL 端口



 |
| 

8083



 | 

MQTT/WebSocket 端口



 |
| 

8080



 | 

HTTP API 端口



 |
| 

18083



 | 

Dashboard 管理控制台端口



 |

## **停止 docker 服务**

如果不需要使用 EMQ-X，使用如下命令停止 docker 服务：

```
sudo docker stop <查看到的进程号>
```

如图：

停止docker服务

## **4\. 访问 DashBoard 并进行简单设置**

## **访问 DashBoard**

访问http://<服务器 ip 地址或[域名](https://cloud.tencent.com/act/pro/domain-sales?from=10680)\>:18083即可访问到 EMQ-X 的后台登录界面，使用用户名 admin 和密码 public 登录：

登录界面

登陆成功之后，后台界面如图：

后台界面

## **语言和主题设置**

默认是英文和 dark-themes，可以在 setting 界面进行更改：

设置界面

中文界面如下：

中文界面

## **用户设置**

刚刚登录面板使用的是默认用户名和密码，安全起见，可以在 user 界面修改：

设置用户名和密码

### **参考资料**

\[1\]**官方下载链接: https://www.emqx.io/cn/downloads**

\[2\]**官方部署文档: https://docs.emqx.io/broker/v3/cn/install.html**

\[3\]**官方下载链接: https://www.emqx.io/downloads/broker/v3.1.0/**

\[4\]**Docker-ce最新版在Ubuntu18.04上的安装、更新、卸载方法（存储库方式）: https://blog.csdn.net/Mculover666/article/details/103642407**

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_843921b4351a)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。