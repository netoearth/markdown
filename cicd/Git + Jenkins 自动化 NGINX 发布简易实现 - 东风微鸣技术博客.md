## 概述[](https://ewhisper.cn/posts/6622/#%E6%A6%82%E8%BF%B0%20-2)

之前基于 GitLab + Jenkins 实现了简单的 NGINX 的自动化发布。  
具体包含如下的组件：

1.  GitLab
    1.  包括 GItLab 的 WebHook；
2.  Jenkins 及其插件：
    1.  Generic Webhook Trigger
    2.  Publish Over SSH

🧠疑问：

为什么不用 Ansible？  
答：这里说明下，之所以不用 Ansible，是因为这个环境默认没有安装 Ansible，而且 Publish Over SSH 也足够用了，就没再用 Ansible 了。

## 详细说明[](https://ewhisper.cn/posts/6622/#%E8%AF%A6%E7%BB%86%E8%AF%B4%E6%98%8E%20-3)

这里有 3 个几个自动化 job，如下：

[![](https://pic-cdn.ewhisper.cn/img/2021/09/26/bb748108348da5f2fcb5420ddcb5766b-20210926161608.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/bb748108348da5f2fcb5420ddcb5766b-20210926161608.png)

1.  [Generic Webhook Trigger _用于和 GitLab 联动, 自动触发 WebHook_](https://plugins.jenkins.io/generic-webhook-trigger)
2.  [Publish Over SSH _用于通过 SSH 发布 NGINX 配置_](https://plugins.jenkins.io/publish-over-ssh)

说明：

配置 **WebHook**  
以 `test-intranet-nginx` 为例进行说明.

1.  进入该项目 -> _设置_ \-> _集成_. 如下图:  
    [![gitlab webhook](https://pic-cdn.ewhisper.cn/img/2021/09/26/744d1857422df566e5058ae8c484d2ff-20210926161821.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/744d1857422df566e5058ae8c484d2ff-20210926161821.png "gitlab webhook")
2.  URL 里填入: `https://jenkins.example.com/generic-webhook-trigger/invoke?token=Jdy0bTQafyfUUBxJw33k`(假设 [jenkins.example.com](http://jenkins.example.com/) 是 Jenkins 的控制台域名，token 可以在对应的 Jenkins 插件 _Generic Webhook Trigger_ 中找到，这里 token 是用于区别具体是哪个 job。)
3.  Trigger 选择: _Push events_ -> _master_.
4.  按需取消勾选 _Enable SSL verification_. 保存. 如下:  
    [![gitlab webhook 填入 jenkins url 和 token](https://pic-cdn.ewhisper.cn/img/2021/09/26/d10c812adad57c8eddc5c2c851c109c7-20210926162208.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/d10c812adad57c8eddc5c2c851c109c7-20210926162208.png "gitlab webhook 填入 jenkins url 和 token")

说明：

URL 地址可以在 Jenkins 的对应插件里找到.  
Trigger 可以按需调整.

在 Jenkins 的 _系统配置_ 里 -> _Publish over SSH_:

如下图:  
[![jenkins 插件 publish over ssh](https://pic-cdn.ewhisper.cn/img/2021/09/26/79f7eb20e8657e570b61e17f7bcef262-20210926162428.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/79f7eb20e8657e570b61e17f7bcef262-20210926162428.png "jenkins 插件 publish over ssh")

填入:

-   Key
-   SSH Server 的:
    -   name（用于给 jenkins 用户识别的用户名）
    -   Hostname（目标机器的 IP 地址）
    -   Username（目标机器的 OS 用户）
    -   Remote Directory（需要把文件发送到的目标机器的目录地址）

说明：

一般情况下，对于 NGINX, 目录是 2 个, 为: `/etc/nginx`（放配置 `*.conf`） 和 `/usr/share/nginx/html`（放静态 web 文件）

以下图 Job 为例:

首先配置 _源码管理_, 如下图:

[![job 源码管理配置](https://pic-cdn.ewhisper.cn/img/2021/09/26/da2334348587b66cd8c32b7e557654ea-20210926162943.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/da2334348587b66cd8c32b7e557654ea-20210926162943.png "job 源码管理配置")

填入:

1.  仓库 URL
2.  认证信息(如果是公开的库, 就不需要认证信息)
3.  分支: `master`（按需调整）
4.  源码库的信息:
    1.  本例中，类型是: `gitlab`
    2.  URL
    3.  GitLab 的 Version

然后配置 _构建触发器_, 如下图:

[![jenkins-webhook-trigger1.png](http://pic-cdn.ewhisper.cn/img/2021/03/11/fbb827c65dc06ff41eec7ee2b1b6fedf-jenkins-webhook-trigger1.png)](http://pic-cdn.ewhisper.cn/img/2021/03/11/fbb827c65dc06ff41eec7ee2b1b6fedf-jenkins-webhook-trigger1.png "jenkins-webhook-trigger1.png")

说明：

详细使用请在浏览器输入图中的 URL 进一步查看.

1.  Variable (使用默认配置)
2.  Expression (使用默认配置)
3.  Token

[![jenkins-webhook-trigger2.png](http://pic-cdn.ewhisper.cn/img/2021/03/11/528aa7f0ddf0396f8a39fd78c8516b7d-jenkins-webhook-trigger2.png)](http://pic-cdn.ewhisper.cn/img/2021/03/11/528aa7f0ddf0396f8a39fd78c8516b7d-jenkins-webhook-trigger2.png "jenkins-webhook-trigger2.png")

4.  Expression(解释如下: 用于进行目录过滤，填入正则后，只有目录匹配正则且发生变化才会触发构建)

[![](https://pic-cdn.ewhisper.cn/img/2021/09/26/141b15948f34a710ced2864e693c6211-20210926163327.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/141b15948f34a710ced2864e693c6211-20210926163327.png)

5.  Text (默认配置)

最后, 是 _构建后操作_（实际「构建」过程没做任何事情）. 如下图:

[![配置 publish over ssh](https://pic-cdn.ewhisper.cn/img/2021/09/26/c4ca405746ce9d63abbd955f5b12ca01-20210926163728.png)](https://pic-cdn.ewhisper.cn/img/2021/09/26/c4ca405746ce9d63abbd955f5b12ca01-20210926163728.png "配置 publish over ssh")

注意：

如果有多台 nginx 要同时发, 就要在这里同时写上多台 SSH Server.

1.  _Name_: 下拉框选择对应 Name
2.  _Transfers_
    1.  _Source files_: 源文件, 位于: `iaas_web_xxxx/conf/**/*`
    2.  _Remove prefix_: 需要移除的前缀, 为: `iaas_web_xxxx/`. 移除后, 示例为: `conf/nginx.conf`
    3.  _Remote directory_: 不填写就是之前填写的目录, 一般为 `/etc/nginx`. 那么示例就发布到: `/etc/nginx` + `conf/nginx.conf`, 即: `/etc/nginx/conf/nginx.conf`
    4.  _Exec command_: 文件传输过去后需要执行的命令. 为: `nginx -t && nginx -s reload`(或：`sudo systemctl reload nginx`). 即, 先 `-t` 验证配置是否有语法错误, 然后再 `reload` 发布. 如果验证有问题, jenkins pipeline 会异常, 变黄或变红.

## 发布流程[](https://ewhisper.cn/posts/6622/#%E5%8F%91%E5%B8%83%E6%B5%81%E7%A8%8B)

1.  用户通过 IDE + Git, 在自己本地修改 NGINX Conf, 并最终 `push` 或 `merge`(也会触发 `push` 的动作) 到 `master`上
2.  GitLab 接收到 `push` event, 触发 webhook 调用: `https://example.com/generic-webhook-trigger/invoke?token=Jdy0bTQafyfUUBxJw33k`
3.  Jenkins 收到 webhook trigger. 并结合 filter 的 Expression 进行判断，确认匹配，则开始自动启动一次 Job.
4.  该 Job 过程为:
    1.  将存有 nginx 配置的仓库 pull 到 jenkins.
    2.  通过 _Publish over SSH_, 将相关目录和文件传输到 SSH Server 的指定目录
    3.  执行 nginx 命令, 进行发布.
5.  结束.

提示：

如果因为其他异常, 导致未自动发布，那么也可以手动点击 Job 页面的: _立即构建_ 进行手动触发

可以通过 [首图](https://pic-cdn.ewhisper.cn/img/2021/09/26/bb748108348da5f2fcb5420ddcb5766b-20210926161608.png) 的 rss 订阅: _Atom feed 失败_, 这样发布失败你就会及时收到邮件.