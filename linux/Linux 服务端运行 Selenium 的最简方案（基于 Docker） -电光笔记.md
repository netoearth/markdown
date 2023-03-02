2022-09-19 11:38:50   [Cyberbolt](https://www.cyberlight.xyz/user/Cyberbolt)

此方案可能是全网最简的 Linux 服务端 Selnenium 运行方案(无图形界面)。您无需安装任何额外环境，拉取 Docker 镜像即可运行 Selenium 代码。

该镜像基于 Python 3.7，Selenium 4.4.0(可以使用 pip3 更新)，内置 Chrome 浏览器及驱动。

该方案仅支持在无图形界面的 Linux 终端运行 Selenium，不支持测试代码，请先在您的本机图形界面中完善代码。

### 链接

-   GitHub: [https://github.com/Cyberbolt/selenium-linux-server](https://github.com/Cyberbolt/selenium-linux-server)
-   dockerhub: [https://hub.docker.com/r/cyberbolt/selenium](https://hub.docker.com/r/cyberbolt/selenium)
-   电光笔记: [https://www.cyberlight.xyz/](https://www.cyberlight.xyz/)

### 测试运行

确保机器已安装 Docker 环境，首先拉取镜像

```
docker pull cyberbolt/selenium
```

运行 Selenium 测试代码

```
docker run --rm cyberbolt/selenium python3 /test/test.py
```

之后收到以下提示

```
Selenium automates browsers. That's it!
```

已经成功运行！本测试访问了 [Selenium](https://www.selenium.dev/) 官网并获取 h1 标题的内容。

### 推荐使用方案

将自己测试好的代码挂载至该容器中，使用 Docker 指定运行自己的代码文件。

您可以像使用 Python 镜像一样使用该 Selenium 镜像。（Docker 运行 Python 镜像的[官方教程](https://docs.docker.com/language/python/)）

PS: 如果需要手动指定 chromedriver 的位置，请选择 /opt/google/chrome/chromedriver