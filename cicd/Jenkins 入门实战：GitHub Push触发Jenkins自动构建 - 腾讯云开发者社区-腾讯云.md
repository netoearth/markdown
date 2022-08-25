## **Jenkins 是什么？**

![](https://ask.qcloudimg.com/http-save/yehe-2778554/nuhkoc6lma.png?imageView2/2/w/1620)

**官方介绍\[1\]**：Jenkins 是一款开源 CI&CD 软件，用于自动化各种任务，包括构建、测试和部署软件

Jenkins 支持各种运行方式，可通过系统包、Docker 或者通过一个独立的 Java 程序

那么 Jenkins 到底帮我们做了哪些东西，解决了团队开发中的哪些痛点呢？

当我们在一个 team 中开发的时候，每个人的本地环境都是有所不同的，比如 node 版本，windows 系统和 Mac 有所区别等等，Jenkins 就可以解决这个问题。Jenkins 就相当于大家的一个统一环境，不会有所差异。

另外，我们平时在部署的之前需要 `npm run build` 打包，Jenkins 中结合 git hook 我们可以做到在我们执行 `git push` 或者合 `master` 的时候帮助我们自动打包。

也就是只需要发起 Git 提交，以下功能自动化完成

-   单元测试
-   打包构建
-   代码部署
-   邮件提醒

本文主要讲我们在 GitHub 提交代码的时候触发 Jenkins 自动打包构建

> 没有购买[服务器](https://cloud.tencent.com/product/cvm?from=10680)...，所以没有部署

## **下载安装**

直接去**官网下载即可\[2\]**

按照提示安装

另外我是用 Java 去启动的，所以也要搭建 Java 环境，这里就不展开了

![](https://ask.qcloudimg.com/http-save/yehe-2778554/oc1jvaslwg.jpeg?imageView2/2/w/1620)

## **Jenkins 启动**

以上安装完，讲道理会自动启动 8080 端口的一个服务，我当时没有，估计是端口占用，直接报错了。

**使用命令启动**

没有的话，切换到这个目录下

```
➜  ~ cd /Applications/Jenkins
```

使用命令启动

```
Java -jar jenkins.war --httpPort=8388
```

其中 8388 是端口号，自己可以自定义

**输入密码**

成功后会让我们输入管理员密码，这个按照它提示的路径下面复制即可

但是苹果下面这个目录有可能是没有权限的，具体的做法是：点开文件的简介，然后最下方点开共享与权限。名称那栏：EVERYONE 的权限设定为读与写就 OK 了

**安装插件**

使用推荐的安装就可以了，我当时可能因为网络问题，很多都失败了（这可能跟我后面踩很多坑有一定关系）在进入系统后，我们还可以安装，所以也还好！

这里啰嗦几句，**如果你后面发现如果有一些配置你的页面没有的，很有可能你缺少了某个插件，因为你的插件很可能决定你页面的配置。**

**创建管理员账户**

上面完成之后，是这个界面。开始我们的 Jenkins 之旅吧

![](https://ask.qcloudimg.com/http-save/yehe-2778554/oauf2dsxfi.png?imageView2/2/w/1620)

## **准备一个 GitHub 项目**

我们这里以 Vue-cli 搭建的项目来讲

![](https://ask.qcloudimg.com/http-save/yehe-2778554/c9pha1g817.jpeg?imageView2/2/w/1620)

这里我们要区分一下，项目主页和仓库地址，下面会有用到

我的项目主页：**https://github.com/GpingFeng/Vue-Jenkins-Test\[3\]**

我的仓库地址：https://github.com/GpingFeng/Vue-Jenkins-Test.git

## **获取 GitHub 的 Personal access token**

-   GitHub 主页
-   点击 setting

![](https://ask.qcloudimg.com/http-save/yehe-2778554/fgl477ncyp.png?imageView2/2/w/1620)

-   点击 Developer settings

![](https://ask.qcloudimg.com/http-save/yehe-2778554/t6ui34oaqh.jpeg?imageView2/2/w/1620)

-   点击 Personal access tokens。再点击 Generate new token.有可能要输入密码

![](https://ask.qcloudimg.com/http-save/yehe-2778554/zl1wqhhwkx.jpeg?imageView2/2/w/1620)

-   勾上 repo 和 admin:repo\_hook

![](https://ask.qcloudimg.com/http-save/yehe-2778554/ow8j33016e.jpeg?imageView2/2/w/1620)

![](https://ask.qcloudimg.com/http-save/yehe-2778554/idegtf3kqm.png?imageView2/2/w/1620)

-   点击 Generate token。生成之后将这个 token 保存。**一定要保存，后面就看不到了**，后面会用到

## **配置 Jenkins**

-   系统设置

![](https://ask.qcloudimg.com/http-save/yehe-2778554/sno8yh0hl1.jpeg?imageView2/2/w/1620)

-   找到 GitHub 这个选项——添加——Jenkins。这里的名称随便填，API URL 填写 https://api.github.com

![](https://ask.qcloudimg.com/http-save/yehe-2778554/lk1i5qy8d.png?imageView2/2/w/1620)

-   弹窗中按以下填写，类型选 Secret text 点击添加

![](https://ask.qcloudimg.com/http-save/yehe-2778554/uz98si0fd8.png?imageView2/2/w/1620)

-   在凭据选上刚刚你添加的，勾上管理 Hook，点击“连接测试”，成功之后如下所示：

![](https://ask.qcloudimg.com/http-save/yehe-2778554/7uplmdilp4.png?imageView2/2/w/1620)

-   最后点击最下面的保存一下

## **Jenkins 新建项目**

-   点击新建按钮

![](https://ask.qcloudimg.com/http-save/yehe-2778554/f6umgs28ow.png?imageView2/2/w/1620)

-   名字随意，勾选 Freestyle project——确认，就可以进入配置页面了

![](https://ask.qcloudimg.com/http-save/yehe-2778554/za7rh5dov7.jpeg?imageView2/2/w/1620)

-   源码管理，基础配置如下

![](https://ask.qcloudimg.com/http-save/yehe-2778554/dmmzpg4zcd.png?imageView2/2/w/1620)

-   源码管理的 Credentials，使用的是 Username with password，配置如下：

![](https://ask.qcloudimg.com/http-save/yehe-2778554/vjj3fzqz7c.png?imageView2/2/w/1620)

-   构造触发器选择：GitHub hook trigger for GITScm polling

![](https://ask.qcloudimg.com/http-save/yehe-2778554/jh9s3zeh2l.png?imageView2/2/w/1620)

-   构建环境和绑定

如下图所示，勾选 Use secret text(s) or file(s)，下面的”凭据”选择我们之前配置过的凭证

![](https://ask.qcloudimg.com/http-save/yehe-2778554/ct3cqqge8l.jpeg?imageView2/2/w/1620)

-   构建，选中 Execute shell，填写构建的命令如下

```
echo $WORKSPACE
node -v
npm -v

npm install&&
npm run build
```

![](https://ask.qcloudimg.com/http-save/yehe-2778554/3m2kla1c4w.png?imageView2/2/w/1620)

## **配置 GitHub 的 webhook 地址**

webhook 是通知 Jenkins 时的请求地址，用来填写到 GitHub 上，这样 GitHub 就能通过该地址通知到 Jenkins

假设 Jenkins 所在服务器的地址是：192.168.0.1，端口为 8080，那么 webhook 地址就是 http://192.168.0.1:8080/github-webhook

-   setting

![](https://ask.qcloudimg.com/http-save/yehe-2778554/yp90tg4ugu.jpeg?imageView2/2/w/1620)

-   Webhooks——Add webhook

![](https://ask.qcloudimg.com/http-save/yehe-2778554/91trgg4l0f.jpeg?imageView2/2/w/1620)

-   在 Payload URL 位置填入 webhook 地址，再点击底部的 Add webhook 按钮，这样就完成 webhook 配置了，今后当前工程有代码提交，GitHub 就会向此 webhook 地址发请求，通知 Jenkins 构建

![](https://ask.qcloudimg.com/http-save/yehe-2778554/yv2q5unk4m.jpeg?imageView2/2/w/1620)

再次提醒，上述地址必须是外网也能访问的，否则 GitHub 无法访问到 Jenkins

在这里我卡了很久，一直都没有成功！需要注意，不能使用 localhost。可以使用 **ngrok 下载链接\[4\]** 启动一个可供外部访问的地址，比如我的端口是 8388。先切换到下载解压好的文件下，使用 ngrok 启动如下：

如下所示，外部就可以通过 http://3043f4fa.ngrok.io 访问到你本地的服务了（但是要注意的是，这个只能维持 8 个小时）

## **验证一下**

我们尝试本地提交代码到 GitHub，可以看到 GitHub 会通知到 Jenkins，Jenkins 就帮我们自动构建了。

切到控制台，可以看到输出如下，说明真的成功了

## **结束语**

以上只是一个小小的尝试，还有很多的坑没踩，比如怎么部署到服务器等等

## **踩过的坑以及参考文档**

**安装插件的方法\[5\]**

**jenkins 源码管理只有 none 选项，怎么弄能看到 Subversion 选项？小白求教。\[6\]**

**webhook 连接不上的原因\[7\]**

**配置 GitHub Push 自动触发 Jenkins 的构建\[8\]**

**Jenkins 在 Mac 上的安装与使用\[9\]**

**macOS Jenkins 安装&配置\[10\]**

**实战笔记：Jenkins 打造强大的前端自动化工作流\[11\]**

### **参考资料**

\[1\]

官方介绍: _https://jenkins.io/zh/doc/_

\[2\]

官网下载即可: _https://jenkins.io/zh/download/_

\[3\]

https://github.com/GpingFeng/Vue-Jenkins-Test: _https://github.com/GpingFeng/Vue-Jenkins-Test_

\[4\]

ngrok 下载链接: _https://ngrok.com/download_

\[5\]

安装插件的方法: _https://www.jianshu.com/p/3b5ebe85c034_

\[6\]

jenkins 源码管理只有none选项，怎么弄能看到Subversion选项？小白求教。: _https://www.oschina.net/question/2352781\_2191336_

\[7\]

webhook 连接不上的原因: _https://stackoverflow.com/questions/42037370/jenkinsgithub-we-couldn-t-deliver-this-payload-couldnt-connect-to-server_

\[8\]

配置GitHub Push自动触发Jenkins的构建: _https://www.cnblogs.com/mingyue5826/p/10768486.html#%E6%9E%84%E5%BB%BA%E7%8E%AF%E5%A2%83%E5%92%8C%E7%BB%91%E5%AE%9A_

\[9\]

Jenkins在Mac上的安装与使用: _https://www.jianshu.com/p/d5121d51a87f_

\[10\]

macOS Jenkins安装&配置: _https://www.jianshu.com/p/9dc3b45fbbec_

\[11\]

实战笔记：Jenkins打造强大的前端自动化工作流: _https://juejin.im/post/5ad1980e6fb9a028c42ea1be_

文章分享自微信公众号：

![](https://open.weixin.qq.com/qr/code?username=gh_9013e08e595a)

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

如有侵权，请联系

cloudcommunity@tencent.com

删除。