　　　　　　　　　　　　**关于Jenkins部署代码权限三种方案**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

****一.**修改Jenkins进程用户为root**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@jenkins ~]# cat /etc/sysconfig/jenkins  | grep JENKINS_USER
JENKINS_USER="jenkins"
[root@jenkins ~]# 
[root@jenkins ~]# 
[root@jenkins ~]# sed -i 's#JENKINS_USER="jenkins"#JENKINS_USER="root"#' /etc/sysconfig/jenkins 
[root@jenkins ~]# 
[root@jenkins ~]# cat /etc/sysconfig/jenkins  | grep JENKINS_USER
JENKINS_USER="root"
[root@jenkins ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909114637711-1843231313.png)

**二.将代码目录用户改为Jenkins**

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

　　**上述代码摘自我之前的笔记，详情请参考：https://www.cnblogs.com/yinzhengjie/p/9607406.html。**

**三.使用sudo授权**

　　 **这个想必大家都会，要么使用命令“visudo”编辑授权，要么使用命令“vi /etc/sudoers”进行编辑，将Jenkins用户权限提升为管理员权限。**

 ![](https://images2018.cnblogs.com/blog/795254/201809/795254-20180909115353958-447799182.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186