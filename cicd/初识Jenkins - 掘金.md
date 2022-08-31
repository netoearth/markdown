## 背景

大家知道我之前搭了个博客，地址是 [www.xiaojieboshi.com](https://link.juejin.cn/?target=http%3A%2F%2Fwww.xiaojieboshi.com%2F "http://www.xiaojieboshi.com/") ，搭建的过程大家可以看[《1小时带你搭建vuepress高大上个人博客》](https://juejin.cn/post/6962520062758486024 "https://juejin.cn/post/6962520062758486024") 这篇文章。但是有个问题，就是我每次写完博客发布到服务器上很繁琐，我一直想搞一个自动发布，持续集成，但是一直在忙别的事耽误了。这次我决定研究一下，把这件事给做了。

## 目的

大家可以看下之前我的一种工作模式，首先代码是我写的，然后我要本地执行run npm build编译一下，然后推到github上面去，然后把生成的aaa文件夹打成压缩包，这个就是可以直接运行查看的文件，然后通过工具上传到linux服务器，然后在服务器上解压放到对应的文件夹下，然后重启下nginx，一套流程下来，十分的繁琐，粉色的步骤都是需要我做的。 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de3ef854f3874a5da5b5fc452b4614e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

但是我要达到的效果就是自动化部署，就是下面粉色的，我只要做编写代码以及push到github，剩下的通通由jenkins来帮我做完，这个就是本篇文章需要实现的效果。 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a37577a6c654be7a9ec9aff98345ccc~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

最后交代一下我买了一台腾讯云的服务器，43.128.34.245，jenkins和nginx都部署在上面，如下面所示： ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f82d05fb3eec44b49dcf66223eab6d9b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

接下来就是开始操作

## Linux安装jenkins

**1、安装jdk**

记得安装jdk11或者17，因为这里有一个坑，从2022年7月2日起，jenkins新版本不再支持java8，仅支持java11和java17，否则会报错。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/762d158822264475b9b246ced7260370~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**2、上官网查找自己想要的版本**，[mirrors.jenkins-ci.org/redhat/](https://link.juejin.cn/?target=https%3A%2F%2Fmirrors.jenkins-ci.org%2Fredhat%2F "https://mirrors.jenkins-ci.org/redhat/")

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df00062a843846c3af5191879dd5956e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) **3、选择需要的版本下载到本地，然后上传到linux服务器**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/810495f1da464b3ab7c6610af584bca1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) **4、直接安装**

```
rpm -ivh jenkins-2.364-1.1.noarch.rpm
复制代码
```

安装成功 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30e2e6800b79424aa9cb08d02ac371da~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**5、修改启动端口**

```
vim /etc/syscofig/jenkins
复制代码
```

我这里把端口号改成了1234

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec1ea47603b461cbd97e6b6442d2179~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**6、配置jdk路径**

```
vim /etc/init.d/jenkins
复制代码
```

找到candidates这个变量，然后把安装的jdk11或者17路径添加进去

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51500b42cfbc495cb3eeba0f787d46ad~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**7、重新加载一下**

```
systemctl daemon-reload
复制代码
```

**8、启动jenkins**

```
cd /etc/init.d

# 启动
./jenkins start
# 停止
./jenkins stop
# 查看状态
./jenkins status

复制代码
```

已经成功跑起来了 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d9c732a8dcf468a922e2a0991e45c82~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 访问和初始化jenkins

**1、输入[http://43.128.34.245:1234/](https://link.juejin.cn/?target=http%3A%2F%2F43.128.34.245%3A1234%2F "http://43.128.34.245:1234/")**

IP是服务器IP地址，端口号是我刚刚设置的端口号

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e63a9c782a448f7bef891bff0500250~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**2、输入密码**

进入上面红色的路径`/var/lib/jenkins/secrets/initialAdminPassword`

```
# 查看秘钥
cat /var/lib/jenkins/secrets/initialAdminPassword
复制代码
```

然后把秘钥拷出来放进去，点击继续

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c87c4f48c2d8458b8a40b5434c6c56ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) **3、安装插件**

我们选择推荐的即可

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4a66860301a414999c35ff1da9b0e77~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

这个就是正在安装的页面 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d7c73ea649c4fe686046ed74125cfd7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) **4、设置admin账户**

我们可以用默认的admin账号，但是建议自己重新创建一个

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d22f367ba63437abca158f945585de4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

保存并完成

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66e2bd1c21bb47cf89cab53f5fb69937~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

成功进入到界面 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b96721a3448b4946a270501650737024~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 准备工作

### 增加权限

jenkins在执行shell脚本的时候会报权限不够，这时候我们需要给jenkins root的权限。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02d73ba70b304376961cb872846f9483~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

修改配置文件

```
vim /etc/sysconfig/jenkins
复制代码
```

把JENKINS\_USER="jenkins"  改成 root

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28e782ac20b1469aa36453ebd426ec4e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

修改Jenkins相关文件夹用户权限

```
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
复制代码
```

`重启下jenkins`

### 安装nodejs

因为我们的项目是要用到node打包的，所以先在jenkins中安装nodeJs插件

进入到插件管理，搜索nodejs，然后选中安装即可 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/015a81427f854ddf87cb5405089adb4b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

安装完成后进入到**全局工具管理**，选择新增nodejs，**选择版本，这个很重要，太高了可能会报错，选择和本地一样的版本稳妥**，填上别名，打钩自动安装，然后保存即可。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55707a233a034e1595a5271ce7df6f10~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 修改服务器时间

查看服务器时间

```
timedatectl | grep "Time zone"
复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35ec47730f79468ca98cdc228bcfe7f9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

进入jenkins的管理，选择脚本命令行 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a44a6625fd0646699effd16a5fd641f9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

输入命令，点击运行即可

```
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone','Asia/Shanghai')
复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/727149a1b474422cb8e2dcc4ec5ddde9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 创建任务

点击左侧的新建item ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee7ba829625b4042a7a38566fd093a05~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

输入名字，选择第一个，创建一个freestyle

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37eb5ffdf2ad4246a370e84622fac29d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**General**

通用这填下`描述`，构建记录保持`7天`，最多保持`10次`构建，还有`github的地址`。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9c70313f2f340ca868e15e7b7b9f45e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48df9c013ea24033b01799cb4cca965f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**源码管理**

设置github项目的仓库地址，我们看下报错了，蓝色框框里的信息，什么意思呢，`就是你原先的密码凭证从2021年8月13日开始就不能用了，必须使用个人访问令牌（personal access token），就是把你的密码替换成token！`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6830153b0acf4d06be94094b29d9b4ef~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

所以我们在`添加凭证`的时候，这里的密码不能填写密码，而是要填github上生成的`token`。 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c1fb32ad4ec45efb1160453f90ad946~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

添加完成后`报错消失`，选择构建的分支为master分支

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99b48e87ba0f4a6dbf006e77f6a182dc~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**构建环境**

选择全局工具管理里面安装的nodejs

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10583aafb0cc4fbb8e3cdffa038a07f2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**构建步骤**

选择新增shell脚本

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3d7ae0703c04ebda833cb96402acc14~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

在里面输入几行脚本，然后点击保存 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cd114b3b7e7454b8768b9aa62ef5642~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

回到项目的管理界面测试一下，首先点`1 Build Now`，然后看到2处就会有个`构建`，构建完成后会变成绿色打钩，然后点3`工作空间`，可以看到4有个`aaa文件夹`生成了，这个就是jenkins把git上的代码拉下来，然后编译打包放到了aaa文件夹，这个就是可以直接运行的文件。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05fdc4d490b04a079c8437de46c3cb9c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

那么为什么会生成到aaa文件夹，因为这是我在代码里面配置的。 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8adbba5f2d0a44679d4085551e6d0bd8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

好，测试完成了，接下来就是把生成的文件部署到nginx，因为这里我是把jenkins和nginx装到了一台机器上，所以直接把文件夹拷到nginx的工作空间即可，如果是分别装在两台机器上，则需要装`Publish over SSH插件`，由jenkins的机器把打好的包远程传输到nginx的那台机器上部署。

我们看下剩下的命令

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed9bd18a2e864ca683dcac52466e6579~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

那为什么我把页面解压缩到/jack/aaa下，然后重启下nginx就能生效了，其实还是我在nginx里面配的，在nginx.conf里，默认就是访问这个文件夹下面的页面。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/836152c056364ded8064bd53c0b63f6a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

至此，新建一个Job就结束了。

## Webhook

好，还差最后一步，我们看到上面演示的时候构建是手动点的Build Now这个按钮，虽然点一下也不累，但是还是想偷懒，不想手动点怎么办。就是做Webhook，就是我每次开发完，代码推到github，自动触发构建。

**1、打开github，在settings里面增加一个webhook** ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/407585cb31174ce7b064191e8229e9fd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**2、在Payload URL填写**`jenkins的地址+/github-webhook/`，**这是固定格式** ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb16d23b31624b3b859efdb95d56b75e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**3、打开项目的配置，在构建触发器github hook这里打上勾，保存下即可** ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7871a09c4184f1f8099b635ea5af953~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**4、测试一下**，我在源码标题这里加了一个1

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60ae12ce38c748f6ae249215f1931fb7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**5、推送一下github**，我们看commit记录是21点56分

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c6063c1fe4d4e7ab1c8fe7f1463de78~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**6、我们看下jenkins上已经自动创建了一个构建任务**，说明webhook已经成功了。`现在的时间是21点59分，那么为什么差了三分钟，主要是我commit完代码，push了好几次才成功的，github大家都懂的，不是很好访问的，所以真正push成功是在59分。`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d4a6d1ed63e4e06a34669ad53d115d0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

**7、最后访问下博客**，发现标题也成功的改掉了，多了一个1

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5abc903bb011480595063fbe8fe4e243~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

演示圆满成功

## 结束语

以上就是我在实现前端项目自动化部署的全部过程，学习过程总是需要自己动手去操作才能知道有没有真的掌握，期间其实遇到很多问题，通过一步步排查、查阅资料，最终完成搭建，自动化部署的方案有很多种，这里我选用的是jenkins的一种，**最后欢迎小伙们点赞、收藏、关注，一键三连，谢谢~**