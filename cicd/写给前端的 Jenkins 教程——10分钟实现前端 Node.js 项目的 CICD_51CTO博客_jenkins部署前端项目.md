## 前言

> 又是令人窒息的毕业设计，导致我走上了折腾 CI/CD 的道路。
> 
> 为了做毕设把两台云服务器都直接重置当成试验品了，期间重置了好几次，还遇到了一些奇奇怪怪的问题，于是决定写篇文章记录一下，也给想折腾的同学填下坑。

众所周知，我们在开发的过程中，写代码其实只占很小的一部分，更多的时间其实是在设计代码、构建和部署。

代码的设计非常考验代码功底，本人才疏学浅，就不讲解这个部分了。

构建和部署通常来讲没那么复杂，但是却十分繁琐，尤其是手动的方式进行构建部署。重复操作多，流程长，非常消耗耐心和精力。

细心的同学应该发现标题中出现了一个新的名词：**CI/CD**。

> 在软件工程中，CI/CD 或 CICD 通常指的是持续集成和持续交付或持续部署的组合实践。CI/CD 通过在应用程序的构建、测试和部署中实施自动化，在开发和运营团队之间架起了桥梁。
> 
> —— 引用自维基百科

**CI（Continuous Integration）** 指的是持续集成，即项目代码的新更改会定期构建、测试并合并到代码仓库中，有效解决一次开发多个项目分支导致代码冲突问题。

**CD（Continuous Delivery/Continuous Deployment）** 指的是持续持续交付/持续部署，即项目代码的新更改可以自动或手动合并到主分支，并在合并至主分支后自动执行构建、测试流程，检测新更改是否对主分支代码产生影响。构建测试通过后，会自动发布并部署至生产环境，有效减轻运维团队负担。

概念说了这么多，肯定很多同学直呼看不懂。没关系，我们找个实际场景。

相信很多初学前端的同学一定有过一个想法：写一个自己的网站放到服务器上。

实现这个想法通常需要以下几个步骤：

> 编写代码 -> （单元测试/集成测试） -> 上传至代码仓库 -> 打包构建 -> 上传至服务器 -> 配置 Nginx/Apache 将 80 端口映射至网站文件夹

是不是一个非常长的流程？

当我们有了 CI/CD 的系统之后，我们就**只需要编写代码，剩下的步骤都交给 CI/CD 系统**来处理，这极大地解放了我们的双手，提升了开发效率。

## `Jenkins` 简介

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务](https://s2.51cto.com/images/blog/202202/16183325_620cd2f5a42c988072.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)Jenkins 官网

> `Jenkins` 是开源 CI&CD 软件领导者，提供超过 1000 个插件来支持构建、部署、自动化，满足任何项目的需要。

一句话概括：`Jenkins` 是一款以插件化的方式实现 CI/CD 的软件。

## 前期工作

准备一台**干净的**装有 **CentOS 7+** 的物理机/虚拟机/云服务器。

**注意：后续操作建议在 `root` 用户下进行，避免出现权限问题！**

## 安装宝塔面板

宝塔面板是一款比较好用的服务器运维软件，建议安装宝塔面板后使用面板来安装各种服务器软件。

```
# 物理机/虚拟机的同学直接在终端执行# 云服务器的同学可以远程连接服务器后在终端执行yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh1.2.3.
```

**面板安装好后终端会输出面板的随机入口 `URL` 和初始化账号密码，可以将 `URL` 输入浏览器，即可进入面板的登录界面。**

使用云服务器的同学如果将 `URL` 输入浏览器后无法访问面板，记得**进入云服务器厂商的管理控制台将 `8888` 端口放行**。

## 安装 `LNMP`

`LNMP` 其实指的是一套网站运行的服务器架构：L（`Linux`）、N（`Nginx`）、M（`MySQL`）、P（`PHP`）。

进入面板后，面板会推荐我们安装两套服务器架构的其中之一：`LNMP/LAMP`，这个时候我们选择左边的 `LNMP`。

`MySQL` 和 `PHP` 我们可能暂时用不到，不过宝塔推荐我们安装，我们就先安装上。

安装完成之后，我们就可以开始安装 `Jenkins` 了。

## 安装 `Jenkins`

首先，我们先检查一下机器是否安装了 `Java`。

很简单，终端输入 `java`，输出不是 `command not found` 就说明我们的机器已经安装了 `Java`。

不过我还是建议使用 `yum` 重新安装一下：

接着，安装下载工具 `wget`：

```
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo1.
```

然后，导入下载密匙：

```
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key1.
```

等上述步骤都执行完成后，就可以开始安装 `Jenkins` 了：

安装过程中可能会跳出几个提示，输入 `yes` 或者 `y` 放行就好。

## 启动 `Jenkins`

经过了漫长的等待，我们可以启动 `Jenkins` 了：

`Jenkins` 运行在机器的 `8080` 端口，**使用云服务器的同学记得到防火墙放行端口**。

## 初始化 `Jenkins`

在浏览器输入 `http://<你的服务器 IP>:8080` 就可以访问到 `Jenkins` 的解锁界面了。

初始化 `Jenkins` 需要输入一段命令来查看密码：

```
cat /var/lib/jenkins/secrets/initialAdminPassword1.
```

把控制台输出的密码复制到 `Jenkins` 解锁界面中，插件安装界面了。

## 安装插件

我们选择左边的**安装推荐插件**，然后静等插件安装完成。

如果有安装失败的插件，点击重试就好，一般多试几次就可以。

当然不排除有多试几次也不行的，建议重置一下服务器从头再来一次。

不嫌麻烦的话也可以一个一个手动安装，插件下载地址：手动下载地址。

## 创建用户

都搞定之后就是创建一个管理员账号了，输入自己喜欢的用户名和密码，输入全名和电子邮箱就可以创建了。

## 安装 `Node.js` 插件

创建完用户之后就能够进入到欢迎页了，我们找到左边的 **管理 `Jenkins`**，然后找到 **插件管理**。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_02](https://s2.51cto.com/images/blog/202202/16183326_620cd2f6022ac6504.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)插件管理

在 **插件管理** 页我们点击 **可选插件** Tab，然后在搜索栏中输入 `NodeJS`，只会命中一个插件，我们安装它。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_03](https://s2.51cto.com/images/blog/202202/16183326_620cd2f6439b675556.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)搜索 NodeJS

等待安装完成。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_04](https://s2.51cto.com/images/blog/202202/16183326_620cd2f68d1295348.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)NodeJS 安装完成

## 配置 `Node.js` 插件

紧接着我们就要去配置 `Node.js` 了，点击 **管理 `Jenkins`**，找到 **全局工具配置**，然后翻到最底下，有一个 `NodeJS`的配置区域。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_05](https://s2.51cto.com/images/blog/202202/16183326_620cd2f6e9f2522261.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)配置 NodeJS

我们点击 **新增 `NodeJS`**，给它取个名字，选个版本，建议选 `LTS` 的版本。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_06](https://s2.51cto.com/images/blog/202202/16183327_620cd2f74399531907.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)新增 NodeJS

## 安装 `Publish Over SSH` 插件

配置好 `Node.js` 之后继续回到 **插件管理**，搜索 `Publish Over SSH` 并安装。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务_07](https://s2.51cto.com/images/blog/202202/16183327_620cd2f78a72f41783.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)Publish Over SSH 安装完成

## 配置 `Publish Over SSH` 插件

安装好之后就要配置 `SSH` 了，还是点击 **管理 `Jenkins`**，找到 **系统配置**，配置好云服务器的 `SSH` 连接信息，点击右下角的 **Test Configuration** 测试一下连接是否正常，显示 **Success** 就说明配置正确了。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务_08](https://s2.51cto.com/images/blog/202202/16183327_620cd2f7c708091695.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)配置 Publish Over SSH

## 安装 `GitHub API` 插件

跟前面安装插件的步骤一样，我们安装好 `GitHub API` 插件。

## 配置 `GitHub API` 插件

在配置之前，我们先要到 `GitHub` 生成 **Personal access token**。

我们点击右上角 **头像** - **Settings**，找到 **Developer settings**，然后选中 **Personal access tokens**，点击右上角 **Generate new token**，按图中所示勾选对应的内容。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_09](https://s2.51cto.com/images/blog/202202/16183328_620cd2f81605763212.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)生成 Personal access token

点击 **Generate token** 之后就会生成一段 **token**。\*\*注意：token 只会显示一次！token 只会显示一次！token 只会显示一次！\*\*建议用记事本记一下。

既然是要实现代码 `push` 到仓库就自动构建并发布，那么我们肯定得用到 `Webhook`，不过我们不需要手动创建 `Webhook`，插件会帮我们自动创建。

现在我们继续来配置插件，还是到 **系统配置** 当中，找到 `GitHub` 配置的部分，点击 **添加 `GitHub` 服务器**，点击 **凭据** 右侧的 **添加** 按钮，选择 **`Jenkins`**。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_10](https://s2.51cto.com/images/blog/202202/16183328_620cd2f8592925844.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)添加 GitHub 服务器

点击后会弹出一个添加凭据的窗口，**类型** 选择为 **Secret text**，将我们刚才生成的 **Personal access token** 复制到 **Secret** 一栏中，点击添加。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_11](https://s2.51cto.com/images/blog/202202/16183328_620cd2f8874b452776.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)添加凭据

添加后我们在 **凭据** 一栏选中 **Secret text**，勾选 **管理 Hook**，点击 **连接测试**，如果正确显示了你的 `GitHub` 用户名，就说明配置成功了。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_12](https://s2.51cto.com/images/blog/202202/16183328_620cd2f8c814955193.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)GitHub 配置完成

## 新建任务

现在我们可以新建任务了，点击主界面左侧的 **新建任务**，选择 **构建一个自由风格的软件项目**，给任务取个名字。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_13](https://s2.51cto.com/images/blog/202202/16183329_620cd2f90ded97812.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)新建任务

我们勾选 **`GitHub` 项目**，输入 **项目 `URL`**（就是项目的浏览器地址）。将下面的 **源码管理** 选中为 **`Git`**，将你要构建部署的项目的 **`clone`** 地址填到 **Repository URL** 一栏中（就是项目的浏览器地址加上 `.git` 后缀名）。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_14](https://s2.51cto.com/images/blog/202202/16183329_620cd2f943f928558.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)配置任务

**注意：如果是公开的仓库，Credentials 一栏可以选择无；如果是私有的仓库，需要先添加一个可以访问该仓库的 `GitHub` 账号，方法类似配置 `GitHub API` 插件，只不过类型一栏选择 用户名密码，然后在下方输入 用户名 密码。**

紧接着我们勾选 **构建触发器** 一栏中的 **GitHub hook trigger for GITScm polling**，勾选 **构建环境** 一栏中的 **Use secret text(s) or file(s)**，在 **凭据** 一栏中选中我们之前添加的 **Secret text**，勾选 **Provide Node & npm bin/ folder to PATH** 为构建项目提供 `Node.js` 环境。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务_15](https://s2.51cto.com/images/blog/202202/16183329_620cd2f98502326210.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)配置任务

然后我们到 **构建** 一栏中，**增加构建步骤**，选择 **执行 `shell`**，在命令中输入：

```
node -vnpm -vrm -rf node_modulesnpm installnpm run testnpm run build1.2.3.4.5.6.7.
```

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务_16](https://s2.51cto.com/images/blog/202202/16183329_620cd2f9b423e53665.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)添加构建步骤

**注意：命令中有一条 `npm run test` 命令可以不加，如果是编写好了测试用例的项目，就需要加上，测试代码功能是否正常。**

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_17](https://s2.51cto.com/images/blog/202202/16183330_620cd2fa1ce4132251.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)添加构建命令

最后一步，我们点击 **增加构建后操作步骤**，选择 **Send build artifacts over SSH**，使用 **SSH** 的方式将代码上传至服务器。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_18](https://s2.51cto.com/images/blog/202202/16183330_620cd2fa6de1353836.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)添加构建后步骤![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_19](https://s2.51cto.com/images/blog/202202/16183330_620cd2fab168969425.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)添加构建后步骤

最后，点击 **保存**，配置完成~

## 立即构建

**新建任务** 配置完成之后，我们点击左侧的 **立即构建**，就会发现左下方的 **Build History** 中多了一个名为 **#1** 的记录。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务_20](https://s2.51cto.com/images/blog/202202/16183330_620cd2fadfd7511513.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)#1

点击 **#1**，选择左侧的 **控制台输出**，就可以看到我们打包构建过程中的所有控制台输出了。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_21](https://s2.51cto.com/images/blog/202202/16183331_620cd2fb2ff1656569.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)#1 控制台输出![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_22](https://s2.51cto.com/images/blog/202202/16183331_620cd2fb6c7ee48519.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)#1 Console

然后我们就能到自己的网站上查看效果了！我部署的是《试试前端自动化测试（React 实战）》中的 Demo。

## 测试 `Webhook`

既然要实现自动化构建部署，那就得在每次代码 `push` 到远程仓库的时候自动执行，所以我们要测试一下 `Webhook` 是否生效，是否可以触发构建部署。

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_github_23](https://s2.51cto.com/images/blog/202202/16183331_620cd2fbcd2ef41017.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)测试

这个时候再回到 `Jenkins`，你会惊讶地发现有个构建正在进行！

![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_云服务_24](https://s2.51cto.com/images/blog/202202/16183332_620cd2fc1436722633.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)#2

**大功告成！**

## 总结

这算是我提前学习工作内容？毕竟毕业后入职也会涉及到相关平台的搭建和配置。

折腾 `Jenkins` 花了整整一天，期间遇到各种各样的问题不断重置服务器，最后踩完了所有的坑，一次跑通了。

写这篇文章花了一天半的样子 ，文中好多图，一直截图上传，好难 QAQ，**这么详细厚着脸皮要个赞不过分吧**~

不知道是不是我服务器性能不够，有时候会出现构建到一半 `Jenkins` 服务挂掉的情况，偶现。

如果有条件的话还是建议搞台高性能的服务器，如果是学生党的话可以考虑把 `Jenkins` 服务放在另外一台干净的服务器上。

## ![写给前端的 Jenkins 教程——10分钟实现前端/ Node.js 项目的 CI/CD_服务器_25](https://s2.51cto.com/images/202205/a8a6cc102a17cd53424471569eb78395df6ac3.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

