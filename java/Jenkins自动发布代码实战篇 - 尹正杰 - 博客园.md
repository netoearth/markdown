 　　　　　　　　　　　　　　　　　　**Jenkins自动发布代码实战篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

 **一.Jenkins服务器配置秘钥对并上传到Gitlab中**

**1>.在Jenkins后端生成秘钥对**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins ~]# ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
e1:75:1f:cf:18:02:5e:b7:16:99:14:75:56:73:ac:ad root@jenkins.yinzhengjie.org.cn
The key's randomart image is:
+--[ RSA 2048]----+
|          . ..=BB|
|         . o .o+=|
|        . o o =o |
|       . o . +.*.|
|        S     o.o|
|              E  |
|                 |
|                 |
|                 |
+-----------------+
[root@jenkins ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.查看服务端的公钥和私钥**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins ~]# cat /root/.ssh/id_rsa.pub 　　　　　　　　　　　　#查看公钥
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCwr6GMG0OYSWyLcraC4FGquTm9C+26a2/SdTysXOTiC6GpqYE9L57NgCSJSYWFalj+s/O6+LqHLY/ORi+LMmh4DtYSD4rUYo5NRd68B3lVs2JU8FqfCK/bJR+Sy/SxDAtKfrP/8gPM+4saB9JXUaljavhwIsyqMJxLkbwmDsdYmf4MHjMoHA4k8qECYKfvL7mep3Cglh0U4dQTubVvjmN/f6oKX7l7yVe+DoSImPyYae16+8AOe0v8+hiL5zB8eBBuCJZfXT/ZQbp5pAWxNCiiHsLI9VQhjsmsLS+bpgEvnCDqbVuXhrYXaLrQDm2EG2YwU1YSES6gJrvs5OkMbwXr root@jenkins.yinzhengjie.org.cn
[root@jenkins ~]# 
[root@jenkins ~]# 
[root@jenkins ~]# cat /root/.ssh/id_rsa　　　　　　　　　　　　　　#查看私钥
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAsK+hjBtDmElsi3K2guBRqrk5vQvtumtv0nU8rFzk4guhqamB
PS+ezYAkiUmFhWpY/rPzuvi6hy2PzkYvizJoeA7WEg+K1GKOTUXevAd5VbNiVPBa
nwiv2yUfksv0sQwLSn6z//IDzPuLGgfSV1GpY2r4cCLMqjCcS5G8Jg7HWJn+DB4z
KBwOJPKhAmCn7y+5nqdwoJYdFOHUE7m1b45jf3+qCl+5e8lXvg6EiJj8mGntevvA
DntL/PoYi+cwfHgQbgiWX10/2UG6eaQFsTQooh7CyPVUIY7JrC0vm6YBL5wg6m1b
l4a2F2i60A5thBtmMFNWEhEuoCa77OTpDG8F6wIDAQABAoIBACTUZ2tKH5v16i9j
ORIs6jkZJST4AJT0YjnPgFc5gehwnkE8eRkk/Qg/Jt6LOc7mrShioHKf+FnHMIvB
65Ura8Vi6lKXdMvyw1DuOJCfSjaIDv4/n9Q4vleC9ILoCtiA9zWPFbxLWWl4IbXK
XZkgB5wWpzTQvDLZsSq7dSnFuN4J2pjR63SJkZCJRgy5g4qpNTH6jzo2D4Q3X/lF
toN6n95He/2yAKRs1eJsvJWn33gnNQbMvKMbtialqQGy476XZomnBjlE2LKj4Kyz
SrIzoJRi1Qq/34vNdDT+ycIvYZmZUo4aOG9QiwykcuxAhbbrrMjtYfMGPpm6KHLA
GbyPQ7kCgYEA5NM9LqY993ie0EjKsIVjptAxHSzh6/BmqFPxXyDpsoZxARR3SCiG
c8sEsFH0zRfF+TS8dza5LVN6tnh+xZcatofbrs+ZIou0oV2+nU46ZO7xBR5ha9rh
mzyEJnypCEdckPMR/Fh9hbl0KEGxK5b6ZZqfcaC4Vr1LLdlASKB2JbUCgYEAxatB
S78IYbQ9Dfv86ok1j4hv+C0gJ7rnqz2Mj+1wZtKUGUgZdXVqFm27nDZobEr6BXCL
8IbMFZXhgm6iM6llRopN0Dm2jDZ42oIe39KjXF7EsrYpDJ41ZgJQxv1sU4gdwOMV
w3lChmCzYtBDP2z2W1Qq6Ln6Ra5pDbG83czHwR8CgYB3Mp6tXUXsUrYP88s59tI5
RDxBYW7yc9FWIBwdHM0ABU56bInSWeHoEbqIirjF2Xt0XIdMZoJB3TmQMeZ/0T3G
FbFXN6ciurnGUUoJMYXzrBB7RR8kiul47yY70jZPLLVIgIY++G2yqi+bBNVgyo33
PXuPOlSsQoEWChSVgJjq/QKBgQCA1zBXS+wNqyqEm/Ptd4O2y6qX6+nim5v3bMXa
5lv2WVl45RrbCa4dcmbv2jLUK0auFv7Pxzzs8OWtW6lT3R0LDojLqWKIH9VEL74q
C6S5R3gUOFGnTNPnaqj2Gybph3ZFTH7aC4bGCe/C/5ZlmAM34jOZv+cWVilZaLl/
JMQq5wKBgQC41olS3GTRuPdNoZCI9rWaR30XafVv3EV7XmAnytNJGJv/VY00a1fk
fXWIUTG/AYQ932qBZQ/PL9eobtuvrZxRj7Xt8p6O6ERRk4mqRERHF+E6yEFdzKef
rXfkvpqaZ63aTLjeolEiVO/Vud6ZEcc6UkpBnajgN1e5vapXARL9pA==
-----END RSA PRIVATE KEY-----
[root@jenkins ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.将Jenkins服务器端的公钥上传到GitLab中**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909104234461-1713034428.png)

 **二.Jenkins关联GitLab的WebUI界面配置**

**1>.创建新任务**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909102324924-190324305.png)

**2>.编辑任务名称**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909102517459-613128283.png)**

**3>.源码管理选择git，并将git的地址填写为GitLab的地址**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909104629089-1948954478.png)**

**4>.****将Jenkins服务器端的私钥上传到Jenkins的webUI中******

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909105039167-1214887554.png)

**5>.关联GitLab**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909105718868-496480174.png)**

**三.小试牛刀-使用Jenkins执行shell命令**

**1>.在Jenkins服务端创建测试目录**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins ~]# mkdir  -p /home/yinzhengjie/data/jenkins/www 
[root@jenkins ~]# 
[root@jenkins ~]# chown -R jenkins.jenkins /home/yinzhengjie/data/jenkins/www/
[root@jenkins ~]# 
[root@jenkins ~]# ll -d /home/yinzhengjie/data/jenkins/www/
drwxr-xr-x 2 jenkins jenkins 6 Sep  9 08:04 /home/yinzhengjie/data/jenkins/www/
[root@jenkins ~]# 
[root@jenkins ~]# chmod +x /home/yinzhengjie/ -R    　　　　　　　　　　　　#这个执行权限必须得加，因为Jenkins默认是没有访问yinzhengjie用户家目录的权限哟！
[root@jenkins ~]# 
[root@jenkins ~]# ll  /home/yinzhengjie/data/jenkins/www/
total 0
[root@jenkins ~]#    
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.构建选择-Execute shell**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909110751496-44081682.png)

**3>.自定义需要执行的shell命令（温馨提示：Jenkins默认的工作目录是：**/var/lib/jenkins/workspace/，如果之前没有执行任务，workspace默认是不存在的，当执行第一个Jenkins任务时自动创建**）**

 ![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909111059299-893889460.png)

**4>.运行任务**

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909111813877-962973742.png)

**5>.Jenkins任务执行成功后查看服务器端测试目录是否有数据**

**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909113150641-1009525363.png)**

**6>.成功执行任务后，我们会发现在服务器端会生成“/var/lib/jenkins/workspace/”目录**

 ![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909113410912-615794457.png)

**四.Jenkins执行任务的具体步骤**

**1>.点击项目名称**![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909115608516-1074357570.png)

**2>.查看编译的历史**

 ![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909115750891-614135147.png)

**3>.查看控制台输出**

 ![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909120130649-1805799260.png)

**4>.**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186