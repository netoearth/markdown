**本章目录:**

0x04 基础使用

-   邮箱&钉钉&微信消息通知 集成配置与实践
    

0x05 补充说明

-   (1) 内置环境变量
    
-   (2) Jenkins 管理员密码忘记重置
    
-   (3) Jenkins 升级迁移
    

0x06 入坑&出坑

-   问题1.jenkins depends on daemon; however Package daemon is not installed.
    
-   问题2:Jenkins 启动时显示 ERROR: No Java executable found in current PATH: /bin:/usr/bin:/sbin:/usr/sbin
    
-   问题3.安装Jenkins后或者安装插件时候一直在加载;
    
-   问题4: 未正确配置Jenkins基础URL等相关信息;
    
-   问题5.无法连接仓库：Command "git ls-remote -h -- git@gitlab.weiyigeek.top:ci-cd/blog.git HEAD" returned status code 128:
    
-   问题6.Jenkins 内置邮件通知发信测试 Failed to send out e-mail javax.mail.AuthenticationFailedException: 535 Error:
    
-   问题7.Jenkins 内置邮件通知发信测试 com.sun.mail.smtp.SMTPSenderFailedException: 501 mail from address must be same as authorization user
    
-   问题8.Jenkins 内置邮件通知发信测试com.sun.mail.smtp.SMTPAddressFailedException: 501 Bad address syntax
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: 在Jenkins中我们还有最重要的一步还没有完成, 即消息通知(让我们知道是构建成功还是、构建失败)等等, 常规的方式有邮箱通知、Shell自定义脚本通知，WebHook通知等;

通知插件插件安装:

```
DingTalk : 钉钉 Jenkins 插件 2.4.3 (https://github.com/jenkinsci/dingtalk-plugin)
Qy Wechat Notification Plugin : 这个插件是一个机器人，可以发布构建状态和发送消息给qy微信. 1.0.2 （https://www.jenkins.io/doc/pipeline/steps/qy-wechat-notification/）
```

**(0) 邮箱通知实践配置**  
描述: 此处以腾讯企业邮箱为例进行配置,首先需要登陆将要被使用的邮箱，注意`必须要使用微信绑定后才能正常生成客户端专用密码`，然后开启SMTP服务;

```
# 客户端设置方法
接收服务器：
imap.exmail.qq.com(使用SSL，端口号993)
发送服务器：
smtp.exmail.qq.com(使用SSL，端口号465)
```

![WeiyiGeek.腾讯企业邮箱](https://i0.hdslb.com/bfs/article/fd9bb5b9c91ea6f4370777d6e242091dc6efd31a.png@942w_555h_progressive.webp)

Step 1.在 `Dashboard -> 系统配置 -> Jenkins Location`进行配置系统管理员邮件地址，注意此处管理员邮箱地址必须与smtp服务发信地址一致否则将会报出`501 mail from address must be same as authorization user`错误;

Step 2.设置发信邮箱相关配置，点击 `Dashboard -> 系统配置 -> 邮件通知` 填入 SMTP 发信服务器地址以及企业邮箱后缀，采用SSL协议并输入认证的账号与客户端专用密码，最后测试发信;

![WeiyiGeek.发信邮箱相关配置](https://i0.hdslb.com/bfs/article/a7ef7e8f3a97502ef6696f600d80ca37fd2430d4.png@942w_492h_progressive.webp)

Step 3.构建项目通信发信测试，点击 `Dashboard -> Maven-HelloWorld -> 构建设置 -> 启用E-mail Notification`

```
# 收信人：Recipients
# 什么场景发送信息:
- 构建失败给每一个人发送发送电子邮件 : Send e-mail for every unstable build
- 谁构建失败给谁发送邮件: Send separate e-mails to individuals who broke the build
- 为每个失败的模块发送电子邮件 : Send e-mail for each failed module
```

![WeiyiGeek.项目通信发信测试](https://i0.hdslb.com/bfs/article/962744b4925a396daf10aaad306a1a0f47842228.png@942w_689h_progressive.webp)

**补充方式：**  
描述: 由于Jenkins自带的邮件功能比较鸡肋，因此这里推荐安装专门的邮件插件(`Email Extension`)并介绍如何配置Jenkins自带的邮件功能作用。

插件安装: 系统管理→管理插件→可选插件选择`Email Extension Plugin`插件进行安装  
系统设置:

-   1.通过系统管理→系统设置，进行邮件配置 -> Extended E-mail Notification -> 输入 SMTP Server 相关信息以及Authentication相关设置(`注意:密码一般是邮箱授权码`)
    
-   2.设置其编码格式以及默认内容类型，以及邮件模板配置在 Extended E-mail的 default content -;
    

```
<!-- ^\s*(?=\r?$)\n 正则替换空行 -->
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
</head>
<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
    <table width="95%" cellpadding="0" cellspacing="0" style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
        <tr>
            <td>(本邮件由程序自动下发，请勿回复！)</td>
        </tr>
        <tr>
            <td>
                <h2><font color="#FF0000">构建结果 - ${BUILD_STATUS}</font></h2>
            </td>
        </tr>
        <tr>
            <td><br />
                <b><font color="#0B610B">构建信息</font></b>
                <hr size="2" width="100%" align="center" />
            </td>
        </tr>
        <tr><a href="${PROJECT_URL}">${PROJECT_URL}</a>
            <td>
                <ul>
                    <li>项目名称：${PROJECT_NAME}</li>
                    <li>GIT路径：<a href="${GIT_URL}">${GIT_URL}</a></li>
                    <li>构建编号：${BUILD_NUMBER}</li>
                    <li>触发原因：${CAUSE}</li>
                    <li>构建日志：<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <b><font color="#0B610B">变更信息:</font></b>
               <hr size="2" width="100%" align="center" />
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>上次构建成功后变化 :  ${CHANGES_SINCE_LAST_SUCCESS}</a></li>
                </ul>
            </td>
        </tr>
 <tr>
            <td>
                <ul>
                    <li>上次构建不稳定后变化 :  ${CHANGES_SINCE_LAST_UNSTABLE}</a></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>历史变更记录 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>变更集:${JELLY_SCRIPT,template="html"}</a></li>
                </ul>
            </td>
        </tr>
        <!--
        <tr>
            <td>
                <b><font color="#0B610B">Failed Test Results</font></b>
                <hr size="2" width="100%" align="center" />
            </td>
        </tr>
        <tr>
            <td>
                <pre style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
                <br />
            </td>
        </tr>
        <tr>
            <td>
                <b><font color="#0B610B">构建日志 (最后 100行):</font></b>
                <hr size="2" width="100%" align="center" />
            </td>
        </tr>-->
        <!-- <tr>
            <td>Test Logs (if test has ran): <a
                href="${PROJECT_URL}ws/TestResult/archive_logs/Log-Build-${BUILD_NUMBER}.zip">${PROJECT_URL}/ws/TestResult/archive_logs/Log-Build-${BUILD_NUMBER}.zip</a>
                <br />
            <br />
            </td>
        </tr> -->
        <!--
        <tr>
            <td>
                <textarea cols="80" rows="30" readonly="readonly" style="font-family: Courier New">${BUILD_LOG, maxLines=100,escapeHtml=true}</textarea>
            </td>
        </tr>-->
        <hr size="2" width="100%" align="center" />
    </table>
</body>
</html>
```

-   3.设置`Default Triggers`触发机制,例如下面是失败时候和成功时候发送;
    

![WeiyiGeek.设置](https://i0.hdslb.com/bfs/article/37c1e3c01852bc5ca70ac9363801bdfbadf6bc69.png@942w_569h_progressive.webp)

-   4.在Job任务调用中选择构建后操作进行设置`Email Notification`进行设置通知
    

![WeiyiGeek.Email-Notification](https://i0.hdslb.com/bfs/article/efe2f1ceeed7002a0dc496a64d49dfded665d1a3.png@942w_791h_progressive.webp)

**(1) 钉钉消息通知实践配置**  
Step 0.在钉钉中建立一个群聊并且创建一个群机器人生成一个Webhook地址，操作如下图所示:

```
例如：https://oapi.dingtalk.com/robot/send?access_token=95f707645db08794166ed3aad3eaad363bb1475bf7c91635b7456a0a8c8893c6
```

Step 1.设置钉钉消息通知参数, 点击 `Dashboard -> 系统配置 -> 钉钉` 选择通知时机以及代理通信(`当该主机无法正常连接网络时可采用此方法`) -> 点击新增(填入唯一的id、以及名称和webhook地址，注意`如果在创建机器人时指定了关键字和加密字符串需要填写上`)->然后测试发信;

![WeiyiGeek.钉钉消息通知参数](https://i0.hdslb.com/bfs/article/8d75c02fdeafbf4a6cb70b2cef2b2c59a6d4f202.png@942w_489h_progressive.webp)

Step 2.在FreeStyle风格的项目是可以在通用设置卡点选钉钉消息通知的，而Maven的项目是没有该点选选项，因为`该插件只支持FreeStyle和PIPELINE流水线`(这里有巨坑所以有的时候还是老版本的插件好用)，注意网上博客中关于大多数此问题都是不适用的官方文档才是第一手;

参考连接: https://jenkinsci.github.io/dingtalk-plugin/examples/freestyleAdvanced.html#详细日志

![WeiyiGeek.FreeStyle风格构建的钉钉通知](https://i0.hdslb.com/bfs/article/a1d689a8517e13ad6f337cb39b10299227e91114.png@942w_426h_progressive.webp)

PS : 对于其它项目风格的项目在后面我们将使用流水线PIPEline进行实现钉钉的消息通知；

**(2) 企业微信通知实践配置**

-   Step 1.设置企业微信通知全局参数，点击 `Dashboard -> 系统配置 -> 企业微信通知配置`设置构建环境名称(会在信息中显示)以及默认Webhook地址(全局的)、通知用户的Uid(@ALL表示全部)
    

![WeiyiGeek.企业微信全局参数](https://i0.hdslb.com/bfs/article/0c31d8109e8e36c1813315c74b8fc79854f05849.png@942w_489h_progressive.webp)

-   Step 2.在构建任务中设置相应的通知参数，点击 `Dashboard -> Maven-HelloWorld(项目名称) -> 构建后的操作 -> 选择企业微信`
    

```
# PS:此处输入的Webhook优先级高于全局的企业微信Webhook这样做的好处是便于为每个任务分配不同的Webhook;
Webhook地址: https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=c222f3fc-f645-440a-ad24-0ce8d9626f11
情况通知:
  仅失败才@
  发送开始构建信息
  仅失败才发送
  仅成功才发送
  仅构建中断才发送
  仅不稳定构建才发送
通知UserID: @ALL
通知手机号码: 选填
```

-   Step 3.对 `Maven-HelloWorld` 项目进行构建并查看控制台输出,消息推送;
    

```
# 构建前发送
> git config core.sparsecheckout # timeout=10
> git checkout -f e8d88cf3e222b79259edcfb7ca48cee7b079ee08 # timeout=10
Commit message: "v1.2"
> git rev-list --no-walk e8d88cf3e222b79259edcfb7ca48cee7b079ee08 # timeout=10
推送通知 {"markdown":{"content":"Jenkins-Notify<font color=\"info\">【Maven-HelloWorld】<\/font>开始构建\n >构建参数：<font color=\"comment\">git_version=v1.2, deploy_option=rollback <\/font>\n >预计用时：<font color=\"comment\">0分钟<\/font>\n >[查看控制台](http://jenkins.weiyigeek.top:8080/job/Maven-HelloWorld/16/console)"},"msgtype":"markdown"}
通知结果 {"errcode":0,"errmsg":"ok"}

# 构建部署后发送
+ ssh -p 20211 weiyigeek@10.10.107.202 'rm -rf /nfs/data4/webapps &&  ln -s /nfs/data4/war/Maven-HelloWorld-20201227-021934-v1.2 /nfs/data4/webapps && kubectl delete pod -l app=java-maven'
**************WARNING**************
Authorized only. All activity will be monitored and reported.
pod "deploy-java-maven-0" deleted
pod "deploy-java-maven-1" deleted
pod "deploy-java-maven-2" deleted
推送通知{"markdown":{"content":"Jenkins-Notify<font color=\"info\">【Maven-HelloWorld】<\/font>构建<font color=\"info\">成功~<\/font>👌\n >构建用时：<font color=\"comment\">28 sec<\/font>\n >[查看控制台](http://jenkins.weiyigeek.top:8080/job/Maven-HelloWorld/16/console)"},"msgtype":"markdown"}
通知结果{"errcode":0,"errmsg":"ok"}
项目运行结果[SUCCESS]
Finished: SUCCESS
```

![WeiyiGeek.企业微信通知实现效果](https://i0.hdslb.com/bfs/article/d6ac4e4459c13e8ae9c8b300679f3bf7625ed43b.png@942w_429h_progressive.webp)

PS : Jenkins 默认的环境变量列表 http://jenkins.weiyigeek.top:8080/env-vars.html/

```
BUILD_NUMBER
    #The current build number, such as "153"
BUILD_ID
    # The current build ID, identical to BUILD_NUMBER for builds created in 1.597+, but a YYYY-MM-DD_hh-mm-ss timestamp for older builds
BUILD_DISPLAY_NAME
    # The display name of the current build, which is something like "#153" by default.
JOB_NAME
    # Name of the project of this build, such as "foo" or "foo/bar".
JOB_BASE_NAME
    # Short Name of the project of this build stripping off folder paths, such as "foo" for "bar/foo".
BUILD_TAG
    # String of "jenkins-${JOB_NAME}-${BUILD_NUMBER}". All forward slashes ("/") in the JOB_NAME are replaced with dashes ("-"). Convenient to put into a resource file, a jar file, etc for easier identification.
EXECUTOR_NUMBER
    # The unique number that identifies the current executor (among executors of the same machine) that’s carrying out this build. This is the number you see in the "build executor status", except that the number starts from 0, not 1.
NODE_NAME
    # Name of the agent if the build is on an agent, or "master" if run on master
NODE_LABELS
    # Whitespace-separated list of labels that the node is assigned.
WORKSPACE
    # The absolute path of the directory assigned to the build as a workspace.
WORKSPACE_TMP
    # A temporary directory near the workspace that will not be browsable and will not interfere with SCM checkouts. May not initially exist, so be sure to create the directory as needed (e.g., mkdir -p on Linux). Not defined when the regular workspace is a drive root.
JENKINS_HOME
    # The absolute path of the directory assigned on the master node for Jenkins to store data.
JENKINS_URL
    # Full URL of Jenkins, like http://server:port/jenkins/ (note: only available if Jenkins URL set in system configuration)
BUILD_URL
    # Full URL of this build, like http://server:port/jenkins/job/foo/15/ (Jenkins URL must be set)
JOB_URL
    # Full URL of this job, like http://server:port/jenkins/job/foo/ (Jenkins URL must be set)
GIT_COMMIT
    # The commit hash being checked out.
GIT_PREVIOUS_COMMIT
    # The hash of the commit last built on this branch, if any.
GIT_PREVIOUS_SUCCESSFUL_COMMIT
    # The hash of the commit last successfully built on this branch, if any.
GIT_BRANCH
    # The remote branch name, if any.
GIT_LOCAL_BRANCH
    # The local branch name being checked out, if applicable.
GIT_CHECKOUT_DIR
    # The directory that the repository will be checked out to. This contains the value set in Checkout to a sub-directory, if used.
GIT_URL
    # The remote URL. If there are multiple, will be GIT_URL_1, GIT_URL_2, etc.
GIT_COMMITTER_NAME
    # The configured Git committer name, if any, that will be used for FUTURE commits from the current workspace. It is read from the Global Config user.name Value field of the Jenkins Configure System page.
GIT_AUTHOR_NAME
    # The configured Git author name, if any, that will be used for FUTURE commits from the current workspace. It is read from the Global Config user.name Value field of the Jenkins Configure System page.
GIT_COMMITTER_EMAIL
    # The configured Git committer email, if any, that will be used for FUTURE commits from the current workspace. It is read from the Global Config user.email Value field of the Jenkins Configure System page.
GIT_AUTHOR_EMAIL
    # The configured Git author email, if any, that will be used for FUTURE commits from the current workspace. It is read from the Global Config user.email Value field of the Jenkins Configure System page.
```

测试环境变量:

```
#!/bin/bash
echo BUILD_NUMBER: ${BUILD_NUMBER }

echo BUILD_ID: ${BUILD_ID}

echo BUILD_DISPLAY_NAME: $BUILD_DISPLAY_NAME:

echo JOB_NAME: $JOB_NAME

echo JOB_BASE_NAME: $JOB_BASE_NAME

echo BUILD_TAG: $BUILD_TAG

echo EXECUTOR_NUMBER: $EXECUTOR_NUMBER

echo NODE_NAME: $NODE_NAME

echo NODE_LABELS: $NODE_LABELS

echo WORKSPACE: $WORKSPACE

echo WORKSPACE_TMP: $WORKSPACE_TMP

echo JENKINS_HOME: $JENKINS_HOME

echo JENKINS_URL: $JENKINS_URL

echo BUILD_URL: $BUILD_URL

echo JOB_URL: $JOB_URL

echo GIT_COMMIT: $GIT_COMMIT

echo GIT_PREVIOUS_COMMIT: $GIT_PREVIOUS_COMMIT

echo GIT_PREVIOUS_SUCCESSFUL_COMMIT: $GIT_PREVIOUS_SUCCESSFUL_COMMIT

echo GIT_BRANCH: $GIT_BRANCH

echo GIT_LOCAL_BRANCH: $GIT_LOCAL_BRANCH

echo GIT_CHECKOUT_DIR: $GIT_CHECKOUT_DIR

echo GIT_URL: $GIT_URL

echo GIT_COMMITTER_NAME: $GIT_COMMITTER_NAME

echo GIT_AUTHOR_NAME: $GIT_AUTHOR_NAME

echo GIT_COMMITTER_EMAIL: $GIT_COMMITTER_EMAIL

echo GIT_AUTHOR_EMAIL: $GIT_AUTHOR_EMAIL
```

测试结果:

```
+ /bin/bash /tmp/script/env.sh
BUILD_NUMBER: 22
BUILD_ID: 22
BUILD_DISPLAY_NAME: #22:
JOB_NAME: Maven-HelloWorld
JOB_BASE_NAME: Maven-HelloWorld
BUILD_TAG: jenkins-Maven-HelloWorld-22
EXECUTOR_NUMBER: 0
NODE_NAME: master
NODE_LABELS: master
WORKSPACE: /var/lib/jenkins/workspace/Maven-HelloWorld
WORKSPACE_TMP: /var/lib/jenkins/workspace/Maven-HelloWorld@tmp
JENKINS_HOME: /var/lib/jenkins
JENKINS_URL: http://jenkins.weiyigeek.top:8080/
BUILD_URL: http://jenkins.weiyigeek.top:8080/job/Maven-HelloWorld/22/
JOB_URL: http://jenkins.weiyigeek.top:8080/job/Maven-HelloWorld/
GIT_COMMIT: 0f50b10b09c160a86972178d94ca1f0a704dd767
GIT_PREVIOUS_COMMIT: 0f50b10b09c160a86972178d94ca1f0a704dd767
GIT_PREVIOUS_SUCCESSFUL_COMMIT: 0f50b10b09c160a86972178d94ca1f0a704dd767
GIT_BRANCH: v1.7
GIT_URL: git@gitlab.weiyigeek.top:ci-cd/java-maven.git
GIT_AUTHOR_NAME:
GIT_AUTHOR_EMAIL:
GIT_COMMITTER_NAME:
GIT_COMMITTER_EMAIL:
GIT_LOCAL_BRANCH:
GIT_CHECKOUT_DIR:
Finished: SUCCESS
```

1.找到用户的路径

```
[root@jenkins-node1 ~]# cd /var/lib/jenkins/users/
[root@jenkins-node1 users]# tree
.
├── 552408925_8628634723176281851
│   └── config.xml
├── admin_8092868597319509744
│   └── config.xml
├── jenkins_3327043579358903316     #我使用的jenkins作为管理员(如果你是admin就进admin目录)
│   └── config.xml                  #修改config.xml
└── users.xml

3 directories, 4 files
```

2.修改jenkins用户目录下的config.xml，定位到`<passwordHash>`那行删除，改为如下内容-

```
[root@jenkins-node1 users]# vim config.xml
<passwordHash>#jbcrypt:$2a$10$slYx6.2Xyss6w9LnuiwnNOReuvkcSkaI.Y.Z2AC6Sp7hdF7hhxlsK</passwordHash>
```

3.新密码为bgx.com 记得重启jenkins生效  

描述: 在使用 Jenkins 时候显示新版本的 Jenkins (2.272) 可以下载 (变更记录)，正好可以实践一哈Jenkins的升级&迁移。  
PS : 如果是是在生产环境中升级建议慎重，可能会导致插件和升级版本不兼容的情况;

操作流程:

```
# (1) 下载更新包
wget https://updates.jenkins.io/download/war/2.272/jenkins.war

# (2) 停止 Jenkins 服务
jenkins:/usr/share/jenkins# systemctl stop jenkins && ls
  # jenkins.war

# (3) 备份上一个版本
jenkins:/usr/share/jenkins# mv jenkins.war jenkins.war.2.263.1.bak
jenkins:/usr/share/jenkins# cp /home/weiyigeek/jenkins.war jenkins.war
jenkins:/usr/share/jenkins# ls -alh
  # -rw-r--r--   1 root root  67M Dec 24 02:38 jenkins.war
  # -rw-r--r--   1 root root  65M Dec  2 13:56 jenkins.war.2.263.1.bak

# (4) 启动 Jenkins 服务
jenkins:/usr/share/jenkins# systemctl start jenkins
jenkins:/usr/share/jenkins# systemctl status jenkins
  # ● jenkins.service - LSB: Start Jenkins at boot time
  #     Loaded: loaded (/etc/init.d/jenkins; generated)
  #     Active: active (exited) since Thu 2020-12-24 02:38:50 UTC; 4s ago
  #       Docs: man:systemd-sysv-generator(8)
  #     Process: 448375 ExecStart=/etc/init.d/jenkins start (code=exited, status=0/SUCCESS)

  # Dec 24 02:38:48 gitlab systemd[1]: Starting LSB: Start Jenkins at boot time...
  # Dec 24 02:38:48 gitlab jenkins[448375]: Correct java version found
  # Dec 24 02:38:48 gitlab jenkins[448375]:  * Starting Jenkins Automation Server jenkins
  # Dec 24 02:38:48 gitlab su[448432]: (to jenkins) root on none
  # Dec 24 02:38:48 gitlab su[448432]: pam_unix(su-l:session): session opened for user jenkins by (uid=0)
  # Dec 24 02:38:49 gitlab su[448432]: pam_unix(su-l:session): session closed for user jenkins
  # Dec 24 02:38:50 gitlab jenkins[448375]:    ...done.
  # Dec 24 02:38:50 gitlab systemd[1]: Started LSB: Start Jenkins at boot time.

# (5) 访问 Jenkins UI 界面验证升级版本
http://jenkins.weiyigeek.top:8080/about/
```

![WeiyiGeek.Jenkins UI](https://i0.hdslb.com/bfs/article/53412a1244d93ab1d785233fc3890dc4627e5e5d.png@942w_624h_progressive.webp)

**问题1.jenkins depends on daemon; however Package daemon is not installed.**  
问题描述: 在Ubuntu 采用 dpkg 安装 jenkins\_2.263.1\_all.deb 时报错提示 daemon 包未安装  
问题复原:

```
$ sudo dpkg -i jenkins_2.263.1_all.deb
Selecting previously unselected package jenkins.
(Reading database ... 115038 files and directories currently installed.)
Preparing to unpack jenkins_2.263.1_all.deb ...
Unpacking jenkins (2.263.1) ...
dpkg: dependency problems prevent configuration of jenkins:
 jenkins depends on daemon; however:
  Package daemon is not installed.

dpkg: error processing package jenkins (--install):
 dependency problems - leaving unconfigured
Processing triggers for systemd (245.4-4ubuntu3.2) ...
Errors were encountered while processing:
 jenkins
```

解决办法:sudo apt install -y daemon

**问题2:Jenkins 启动时显示 ERROR: No Java executable found in current PATH: /bin:/usr/bin:/sbin:/usr/sbin**  
问题复原:

```
$ systemctl status jenkins
Dec 23 14:02:57 gitlab systemd[1]: Starting LSB: Start Jenkins at boot time...
Dec 23 14:02:57 gitlab jenkins[356298]: ERROR: No Java executable found in current PATH: /bin:/usr/bin:/sbin:/usr/sbin
Dec 23 14:02:57 gitlab jenkins[356298]: If you actually have java installed on the system make sure the executable is in the aforementioned path and that 'type -p ja>
Dec 23 14:02:57 gitlab systemd[1]: jenkins.service: Control process exited, code=exited, status=1/FAILURE
Dec 23 14:02:57 gitlab systemd[1]: jenkins.service: Failed with result 'exit-code'.
Dec 23 14:02:57 gitlab systemd[1]: Failed to start LSB: Start Jenkins at boot time.
```

问题原因: 未找寻到有效的Java执行环境;  
解决流程:

```
①.先执行echo $PATH 看看环境变量运行结果如下：
/usr/maven/maven/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/java/jdk1.8/bin
如果连这都没有的话重新安装Java。

②.建立软连接：ln -s /usr/java/jdk1.8/bin/java /usr/bin/java（换成你自己的路径）
Please wait while Jenkins is getting ready to work (jenkins)
如果界面提示Jenkins正在启动,请稍后…或者提示
Please wait while Jenkins is getting ready to work…
```

**问题3.安装Jenkins后或者安装插件时候一直在加载;**  
问题描述: 由于Jenkins官方插件下载地址没被墙但是网速很慢，下载时间也长;  
解决方法:换清华的镜像进去之后下载插件即可 (http://updates.jenkins-ci.org/download/)  
操作流程: 需要你进入jenkins的工作目录

```
# 打开 hudson.model.UpdateCenter.xml 把 http://updates.jenkins-ci.org/update-center.json 改成 http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
sed -i "s#updates.jenkins.io#mirrors.tuna.tsinghua.edu.cn/jenkins/updates#g" /var/lib/jenkins/hudson.model.UpdateCenter.xml

# 上面的命令就是将将安装目录下的 hudson.model.UpdateCenter.xml 中改成
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>


# (2) 将updates文件夹下的default.json 中所有 http://updates.jenkins-ci.org/download/替换为 https://mirrors.tuna.tsinghua.edu.cn/jenkins/ PS: 也可以在后台进行设置
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json


# (3) 修改完成后重启 Jenkins 即可
```

![WeiyiGeek.Jenkins Please wait](https://i0.hdslb.com/bfs/article/132571204e66800ed0ba52eed9396da133cb1a01.png@942w_570h_progressive.webp)

**问题4: 未正确配置Jenkins基础URL等相关信息;**  
问题描述: Jenkins的根URL是空的，但是需要Jenkins的许多特性的正确操作，如电子邮件通知、PR状态更新和环境变量，如BUILD\_URL。  
请提供Jenkins配置中的准确值。

```
Jenkins root URL is empty but is required for the proper operation of many Jenkins features like email notifications, PR status update, and environment variables such as BUILD_URL.

Please provide an accurate value in Jenkins configuration.
```

解决办法: Dashboard -> 配置 -> Jenkins Location -> Jenkins 地址 & 邮箱

**问题5.无法连接仓库：Command "git ls-remote -h -- git@gitlab.weiyigeek.top:ci-cd/blog.git HEAD" returned status code 128:**  
问题复原:

```
stdout:
  stderr: Host key verification failed.
  fatal: Could not read from remote repository.
# Please make sure you have the correct access rights and the repository exists.
```

问题原因: 由于采用SSH协议进行代码的拉取和信息的查看，在利用公密钥首次链接时候未绑定其机器的公钥信息, 将会导致 `Host key verification failed.`  
解决办法: 在连接的机器上先执行`git -T git@gitlab.weiyigeek.top`保存其主机的公钥信息;

```
# 例如 首次连接Gitlab时候需要进行主机于公钥绑定
ssh -T git@gitlab.com
# 无法建立主机“gitlab.com(172.65.251.78)”的真实性。
The authenticity of host 'gitlab.com (172.65.251.78)' can\'t be established.
ECDSA key fingerprint is SHA256:HbW3g8zUjNSksFbqTiUWPWg2Bq1x8xdGUrliXFzSnUw.
Are you sure you want to continue connecting (yes/no/[fingerprint])?

$ cat ~/.ssh/known_hosts
gitlab.com,172.65.251.78 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAABFSMqzJeV9rUzU4kWitGjeR4PWSa29SPqJ1fVkhtj3Hw9xjLVXVYrU9QOLXBpQ6KWjbjTDTdDkoohFzgbEYI=
```

**问题6.Jenkins 内置邮件通知发信测试 Failed to send out e-mail javax.mail.AuthenticationFailedException: 535 Error:**  
错误信息：

```
Failed to send out e-mail
javax.mail.AuthenticationFailedException: 535 Error: ÇëÊ¹ÓÃÊÚÈ¨ÂëµÇÂ¼¡£ÏêÇéÇë¿´: http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256

at com.sun.mail.smtp.SMTPTransport$Authenticator.authenticate(SMTPTransport.java:947)
at com.sun.mail.smtp.SMTPTransport.authenticate(SMTPTransport.java:858)
at com.sun.mail.smtp.SMTPTransport.protocolConnect(SMTPTransport.java:762)
at javax.mail.Service.connect(Service.java:364)
```

错误原因: 配置STMP的邮箱账号，输入的认证字符串是邮箱密码而并非生成的客户端密码, 在 腾讯企业邮箱、163邮箱都需要使用生成的客户端密码进行登录;  

**问题7.Jenkins 内置邮件通知发信测试 com.sun.mail.smtp.SMTPSenderFailedException: 501 mail from address must be same as authorization user**  
错误信息:

```
Failed to send out e-mail
com.sun.mail.smtp.SMTPSenderFailedException: 501 mail from address must be same as authorization user
at com.sun.mail.smtp.SMTPTransport.mailFrom(SMTPTransport.java:1817)
Caused: com.sun.mail.smtp.SMTPSendFailedException: 501 mail from address must be same as authorization user
;
  nested exception is:
com.sun.mail.smtp.SMTPSenderFailedException: 501 mail from address must be same as authorization user
at com.sun.mail.smtp.SMTPTransport.issueSendCommand(SMTPTransport.java:2374)
at com.sun.mail.smtp.SMTPTransport.mailFrom(SMTPTransport.java:1808)
at com.sun.mail.smtp.SMTPTransport.sendMessage(SMTPTransport.java:1285)
at javax.mail.Transport.send0(Transport.java:231)
```

错误原因: 最后发现是jenkins url下面的系统管理员邮件地址没有填写或者与STMP发信邮箱不一致  
解决办法: 填写系统管理员邮箱与STMP发信邮箱地址一致就可以了。

**问题8.Jenkins 内置邮件通知发信测试com.sun.mail.smtp.SMTPAddressFailedException: 501 Bad address syntax**

错误信息:

```
ERROR: Invalid Addresses
javax.mail.SendFailedException: Invalid Addresses;
  nested exception is:
com.sun.mail.smtp.SMTPAddressFailedException: 501 Bad address syntax
at com.sun.mail.smtp.SMTPTransport.rcptTo(SMTPTransport.java:2064)
at com.sun.mail.smtp.SMTPTransport.sendMessage(SMTPTransport.java:1286)
at javax.mail.Transport.send0(Transport.java:231)
at javax.mail.Transport.send(Transport.java:100)
at hudson.tasks.MailSender.run(MailSender.java:130)
at hudson.tasks.MailSender.execute(MailSender.java:105)
at hudson.maven.MavenModuleSetBuild$MavenModuleSetBuildExecution.cleanUp(MavenModuleSetBuild.java:1093)
at hudson.model.Run.execute(Run.java:1954)
at hudson.maven.MavenModuleSetBuild.run(MavenModuleSetBuild.java:543)
at hudson.model.ResourceController.execute(ResourceController.java:97)
at hudson.model.Executor.run(Executor.java:429)
Caused by: com.sun.mail.smtp.SMTPAddressFailedException: 501 Bad address syntax

at com.sun.mail.smtp.SMTPTransport.rcptTo(SMTPTransport.java:1917)
... 10 more
Finished: FAILURE
```

错误原因: 输入的接收邮箱地址是无效的格式;

原文地址1:  https://mp.weixin.qq.com/s/1jAu4c\_JI56qPP6QN4hYiQ

原文地址2: https://mp.weixin.qq.com/s/Bhp2NNxkDjxXPQ3JyFQyeA

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

>               如果你觉得这个专栏还不错的，请给这篇专栏点个赞、投个币、收个藏、关个注，这将对我有很大帮助！     
> 
>               欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源: https://weiyigeek.top 【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 

![](https://i0.hdslb.com/bfs/article/c378e4695278fb4375c55247420a5858c85a7a9b.png@942w_483h_progressive.webp)

个人主页：https://weiyigeek.top