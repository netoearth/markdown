Yacht 是一个容器管理工具，它提供了一个图形用户界面 (GUI)，用于轻松部署 Docker 容器。 它使用模板提供容器的一键部署，以便在您的服务器上轻松设置和管理。 在本指南中，您将在搬瓦工 VPS 服务器上安装带有 Docker 的 Yacht。Yacht 这个是最近看到的一个可视化的 Docker 容器管理界面，感觉比较新奇，搜了一下好像中文教程也比较少，所以在此分享一下。Yacht 中文是游艇的意思，Docker 的 logo 是个大鲸鱼，这个是个大游艇，还挺有意思。

首先需要安装 Docker，相关教程：

-   《[在搬瓦工 VPS 的 Ubuntu 16.04 LTS 系统上安装最新版 Docker CE](https://www.bandwagonhost.net/2159.html "在搬瓦工 VPS 的 Ubuntu 16.04 LTS 系统上安装最新版 Docker CE-Bandwagonhost中文网")》
-   《[搬瓦工 Linux CentOS 7 系统安装最新版 Docker 教程](https://www.bandwagonhost.net/2163.html "搬瓦工 Linux CentOS 7 系统安装最新版 Docker 教程-Bandwagonhost中文网")》
-   《[搬瓦工 VPS 上安装部署 Docker 集群管理工具 Kubernetes (K8S) 教程](https://www.bandwagonhost.net/3991.html "搬瓦工 VPS 上安装部署 Docker 集群管理工具 Kubernetes (K8S) 教程-Bandwagonhost中文网")》

## 一、在 Docker 上安装 Yacht

Yacht 官网：[https://yacht.sh](https://yacht.sh/)

### 选项一：安装为 Docker Volume

创建 Yacht 卷：

```
docker volume create yacht
```

安装 Yacht，并分配监听端口，推荐使用 8000 端口。

```
docker run -d -p 8000:8000 -v /var/run/docker.sock:/var/run/docker.sock -v yacht:/config --name yacht selfhostedpro/yacht
```

### 选项二：通过 Docker compose 安装

创建 Yacht 目录：

```
mkdir /opt/yacht
```

在 Yacht 目录下创建文件：

```
nano /opt/yacht/docker-compose.yml
```

粘贴下面内容：

```
version: "3"
services:
  yacht:
    container_name: yacht
    restart: unless-stopped
    ports:
      - 8000:8000
    volumes:
      - yacht:/config
      - /var/run/docker.sock:/var/run/docker.sock
    image: selfhostedpro/yacht

volumes:
  yacht:
```

保存文件。

切换目录：

```
cd /opt/yacht
```

启动应用：

```
docker-compose up -d
```

## 二、设置 Yacht

首先需要放行 8000 端口，这个在搬瓦工 VPS 上一般都没有问题的。为防万一也可以再次操作一下。

Ubuntu：

```
ufw allow 8000/tcp
```

CentOS：

```
firewall-cmd --zone=public --add-port=8000/tcp --permanent
```

然后通过浏览器打开下面地址：

```
http://bwh-server-ip:8000
```

把其中的 `bwh-server-ip` 改成你的 VPS 的 IP 地址即可。

使用下面的默认账户名和密码进行登录：

-   **Username:** admin@yacht.local
-   **Password:** pass

为了保护您的服务器，请通过右上角的用户设置更改默认管理员用户名和密码。

现在，通过选择左侧边栏上的模板创建您的第一个模板，输入应用程序标题和 URL。 要使用默认模板，请复制列出的 Yacht Github URL，然后单击提交以创建模板。

![](https://www.bandwagonhost.net/wp-content/uploads/2022/03/bandwagonhostnet_yacht-1024x283.png)

然后，单击您的模板，选择要部署的应用程序，浏览应用程序详细信息，在 Networking 下分配网络端口，设置目录，然后单击 Deploy 以在您的服务器上创建容器。

![](https://www.bandwagonhost.net/wp-content/uploads/2022/03/bandwagonhostnet_yacht2-1024x586.jpeg)

一旦成功，您的应用程序将在 Yacht 仪表板中显示其 CPU 和内存使用统计信息。

## 三、搬瓦工优惠套餐和补货通知

**搬瓦工推荐方案**

搬瓦工实时库存：[https://stock.bwg.net](https://stock.bwg.net/)

温馨提醒 如果您有选择困难症，直接选中间的 CN2 GIA-E方案，季付 $49.99，多达 12 个机房任意切换

选择建议：

-   入门：洛杉矶 CN2 套餐，目前最便宜，可选 CN2 GT 机房，入门之选。
-   **推荐：**洛杉矶 CN2 GIA-E 套餐，速度超快，可选机房多（DC6、DC9、日本软银、荷兰联通等），性价比最高。
-   高端：香港 CN2 GIA 套餐，价格较高，但是无可挑剔。东京 CN2 GIA 套餐也是非常不错的高端选择。

**搬瓦工新手教程**

1.  **搬瓦工新手入门：《[搬瓦工新手入门完全指南：方案推荐、机房选择、优惠码和购买教程](https://www.bandwagonhost.net/4518.html)》（推荐阅读）**
2.  **搬瓦工购买教程：《[2022 年最新搬瓦工购买教程和支付宝支付教程](https://www.bandwagonhost.net/716.html)》**
3.  **搬瓦工优惠码：BWH3HYATVBJW**
4.  搬瓦工补货通知：《[欢迎订阅搬瓦工补货通知（补货提醒）/ 加入搬瓦工交流群](https://www.bandwagonhost.net/1215.html)》
5.  搬瓦工方案推荐：《[搬瓦工高性价比 VPS 推荐：目前哪款方案最值得买？](https://www.bandwagonhost.net/1967.html)》

**搬瓦工优惠通知**

目前搬瓦工一共有两个限量版套餐，分别是 DC9 CN2 GIA 限量版和 DC6 CN2 GIA-E 限量版，这两个套餐价格分别为 79.99 和 89.99 美元/年，目前都是处于缺货状态，所以如果需要购买的话可以关注下面的补货通知，有货了会第一时间通知的。

-   **搬瓦工补货通知 QQ 群 8（全员禁言，仅发送通知）**：[697178487](https://jq.qq.com/?_wv=1027&k=58ATYA5)
-   **搬瓦工补货通知 QQ 群 10（全员禁言，仅发送通知）**：[451796455](https://jq.qq.com/?_wv=1027&k=ZledJHen)
-   搬瓦工补货通知 TG 群：[@BandwagonHostNews](https://t.me/BandwagonHostNews)
-   搬瓦工补货通知**邮件订阅 1**：[点击订阅（Google Groups）](https://groups.google.com/d/forum/bandwagonhost)
-   搬瓦工补货通知**邮件订阅 2：[点击此处提交邮箱地址](https://www.wjx.top/jq/74781112.aspx)**

[![](https://www.bandwagonhost.net/wp-content/uploads/2021/04/bandwagonhost_banner_new3.png)](https://www.bandwagonhost.net/4518.html)

未经允许不得转载：[Bandwagonhost中文网](https://www.bandwagonhost.net/) » [Yacht 教程：如何使用 Yacht 通过图形界面管理 Docker 容器](https://www.bandwagonhost.net/12424.html)