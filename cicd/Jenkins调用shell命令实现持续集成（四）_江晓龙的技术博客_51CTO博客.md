## 6.创建jenkins的job任务

### 6.1.实现jenkins+shell集成

#### 6.1.1新建一个job任务

![Jenkins调用shell命令实现持续集成（四）_初始化](https://s2.51cto.com/images/blog/202111/18144807_6195f727a610916076.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.1.2.创建一个freestyle project

![Jenkins调用shell命令实现持续集成（四）_远程仓库_02](https://s2.51cto.com/images/blog/202111/18144807_6195f727e2c599242.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.1.3.调用shell命令进行构建

![Jenkins调用shell命令实现持续集成（四）_初始化_03](https://s2.51cto.com/images/blog/202111/18144808_6195f7281db8823923.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![Jenkins调用shell命令实现持续集成（四）_初始化_04](https://s2.51cto.com/images/blog/202111/18144808_6195f7284bdf343314.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.1.4.点击立即构建

![Jenkins调用shell命令实现持续集成（四）_git_05](https://s2.51cto.com/images/blog/202111/18144808_6195f7288e12e55460.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.1.5.查看构建详情

![Jenkins调用shell命令实现持续集成（四）_git_06](https://s2.51cto.com/images/blog/202111/18144808_6195f728d1c6997997.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.1.6.查看文件存放路径

![Jenkins调用shell命令实现持续集成（四）_git_07](https://s2.51cto.com/images/blog/202111/18144809_6195f729129ee35321.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```
所有的项目都会存放在/var/lib/jenkins/workspace/中
[root@jenkins ~]# ll /var/lib/jenkins/workspace/freestyle-jobs1/
总用量 4
-rw-r--r-- 1 jenkins jenkins 13 8月  24 09:23 file1
```

### 6.2.实现jenkins+gitlab集成

#### 6.2.1.在gitlab上配置一个公钥

![Jenkins调用shell命令实现持续集成（四）_git_08](https://s2.51cto.com/images/blog/202111/18144809_6195f729560285757.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.2.新建一个项目

![Jenkins调用shell命令实现持续集成（四）_远程仓库_09](https://s2.51cto.com/images/blog/202111/18144809_6195f72973b9649827.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.3.上传monitor代码

```
1）初始化git仓库
[root@jenkins monitor]# git init
初始化空的 Git 版本库于 /root/monitor/.git/

2）添加远程仓库
[root@jenkins monitor]# git remote -v
[root@jenkins monitor]# git remote add origin git@gitlab.jiangxl.com:kaifa1zu/monitor.git

3）提交本地代码
[root@jenkins monitor]# git add .
[root@jenkins monitor]# git commit -m "首次提交"

4）推送至远程仓库
[root@jenkins monitor]# git push -u origin master
Counting objects: 414, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (407/407), done.
Writing objects: 100% (414/414), 4.31 MiB | 3.85 MiB/s, done.
Total 414 (delta 47), reused 0 (delta 0)
remote: Resolving deltas: 100% (47/47), done.
To git@gitlab.jiangxl.com:kaifa1zu/monitor.git
 * [new branch]      master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```

![Jenkins调用shell命令实现持续集成（四）_远程仓库_10](https://s2.51cto.com/images/blog/202111/18144809_6195f729be90d52769.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.4.jenkins新建一个项目

![Jenkins调用shell命令实现持续集成（四）_git_11](https://s2.51cto.com/images/blog/202111/18144810_6195f72ac55b790478.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.5.添加私钥与gitlab建立连接

源码管理选择git然后添加一个私钥

![Jenkins调用shell命令实现持续集成（四）_本地代码_12](https://s2.51cto.com/images/blog/202111/18144811_6195f72b1ee2d26355.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```
类型选择ssh，通过[root@jenkins monitor]# cat ~/.ssh/id_rsa这个命令获取私钥
```

![Jenkins调用shell命令实现持续集成（四）_git_13](https://s2.51cto.com/images/blog/202111/18144811_6195f72b4adf644783.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

选择好对应的私钥即可，在点击保存

![Jenkins调用shell命令实现持续集成（四）_远程仓库_14](https://s2.51cto.com/images/blog/202111/18144811_6195f72b7cbe636489.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.6.点击立即构建

点击立即构建

![Jenkins调用shell命令实现持续集成（四）_源码管理_15](https://s2.51cto.com/images/blog/202111/18144811_6195f72bc0bcc9836.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.7.查看控制台输出

![Jenkins调用shell命令实现持续集成（四）_git_16](https://s2.51cto.com/images/blog/202111/18144812_6195f72c0574354134.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6.2.7.查看工作区文件代码

![Jenkins调用shell命令实现持续集成（四）_git_17](https://s2.51cto.com/images/blog/202111/18144812_6195f72c4edcc21758.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)