项目组已实现通过[Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020)进行构建，最近构建任务加入了sonarQube进行代码扫描的内容。

开发每次推送代码到仓库时（git push），都需要执行一次构建，以产生sonarQube扫描报告。

考虑到Gogs 的[webhook](https://so.csdn.net/so/search?q=webhook&spm=1001.2101.3001.7020)可以检测push事件后进行处理，我们决定把整个构建做成自动化，实现开发推送代码到仓库后，自动触发Jenkins构建，提高开发效率。

**一、Jenkins安装Generic Webhook Trigger Plugin**

使用管理员账号登录Jenkins后，在系统管理-》管理插件，搜索“Generic Webhook Trigger”后安装，顺利完成插件安装：

![](https://img-blog.csdn.net/20180605184321996?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

**二、Jenkins项目配置**

![](https://img-blog.csdn.net/20180605184851663?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

其中在“触发远程构建”勾选后，需要填写身份验证令牌(TOKEN\_NAME)，这里配置为"abc123"，后续在webhook配置时用到。

**三、Gogs仓库配置**

(1) 使用管理员账号登录Gogs，点击“仓库设置”

![](https://img-blog.csdn.net/20180605185715541?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

(2) 选择左侧菜单的“管理Web钩子”，选择右上角的“添加Web钩子”，下拉菜单展开后，选择Gogs：

![](https://img-blog.csdn.net/2018060518584440?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

主要配置推送地址，例如：

http://username:password@192.168.1.111:8888/generic-webhook-trigger/invoke?token=abc123  

其中步骤二配置的TOKEN\_NAME就是这里应用上了。

![](https://img-blog.csdn.net/20180605190539590?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

配置好后，点击“添加Web钩子”保存内容。

(3) 验证配置内容

重新选中刚才配置的内容，进入配置页面

![](https://img-blog.csdn.net/20180605191049172?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

页面底部有一个“测试推送”的按钮，点击按钮可以模拟一次推送（push request），然后检查Jenkins是否收到推送请求并处理：

![](https://img-blog.csdn.net/20180605190955647?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

![](https://img-blog.csdn.net/20180605191409359?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZXVuZ1pa/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

**四、后记**

我们本来想实现精确到对git某个分支推送才触发构建的，但是Gogs的webhook配置暂时不支持，目前的配置仓库任何分支推送都会触发构建。