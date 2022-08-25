## 前言

我们在前面使用Jenkins集合Gogs来进行持续集成的时候，选择的是Jenkins定时检测git仓库是否有更新来决定是否构建。也就是说，我们提交了代码Jenkins并不会马上知道，那么我们可以通过webhook来解决。Jenkins的插件中心已经有对gogs的支持，真的是非常赞。

> [https://plugins.jenkins.io/gogs-webhook](https://plugins.jenkins.io/gogs-webhook)

## 安装Gogs webhook 插件

打开 系统管理 -> 管理插件 -> 可选插件 ，在右上角的输入框中输入“gogs”来筛选插件：

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323123620877-1149383690.png)

## 在gogs中配置

1.  进入我们的仓库，点击仓库设置

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323124010192-1728342384.png)

2.添加webhook

点击 管理Web钩子 -> 添加Web钩子 ->选择Gogs

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323124238564-968104769.png)

添加如下配置：

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323124510238-2056032717.png)

推送地址的格式为：`http(s)://<你的Jenkins地址>/gogs-webhook/?job=<你的Jenkins任务名>`

3.配置Jenkins

进入主面板，点击我们的任务：

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323124945286-673566783.png)

选择配置：

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323125006824-679229001.png)

选择Gogs Webhook 根据自己的需要进行配置，如果没有设置密钥那么什么都不用动。

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323125050591-1026292103.png)

## 测试

我们回到gogs，点击 推送测试 ，推送成功之后会看到一条推送记录

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323125152960-591204673.png)

回到我们的Jenkins可以看到已经成功进行了一次构建：

![](https://images2018.cnblogs.com/blog/668104/201803/668104-20180323125246826-165056931.png)