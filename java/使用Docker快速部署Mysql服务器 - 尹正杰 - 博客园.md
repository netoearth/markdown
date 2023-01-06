　　　　　　　　　　　　**使用Docker快速部署Mysql服务器**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

　　**请原谅我是一个标题党，我个人觉得没有必要从头做MySQL的镜像，可以在官方提供的MySQL镜像的基础之上更迭咱们自己的软件。接下来我们谈谈一下如何下载官方的镜像并启动过程。**

**一.下载MySQL镜像并启动**

**1>.访问Docker的官方镜像仓库(https://hub.docker.com/)**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200118172102316-1863225087.png)**

**2>.选择好MySQL分支后,点击"tag",如下图所示，会列出响应的版本**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200118172253236-507312034.png)

**3>.选择相应的MySQL版本**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200118172547009-1713339270.png)

**4>.二话不说，直接使用Docker****下载对应的Mysql镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY            TAG                  IMAGE ID            CREATED             SIZE
nginx                 v0.2-20200118-1225   35c0c66e03ac        2 hours ago         448MB
jason/centos7-nginx   v0.1.20200114        7372d16c99bc        27 hours ago        448MB
jason/centos7-nginx   latest               c4e0980a825a        28 hours ago        448MB
centos                centos7.6.1810       f1cb7c7d58b7        10 months ago       202MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image pull mysql:5.6.44
5.6.44: Pulling from library/mysql
0a4690c5d889: Pull complete 
98aa2fc6cbeb: Pull complete 
0777e6eb0e6f: Pull complete 
2464189c041c: Pull complete 
b45df9dc827d: Pull complete 
8f57052b58bf: Pull complete 
ee774b34419e: Pull complete 
bcb6c29a9771: Pull complete 
f5eace967cb6: Pull complete 
fe457a2a894d: Pull complete 
a3266082cf3b: Pull complete 
Digest: sha256:02b3ddb41d6e5d48d24aa8e59e1cd5870ddaca8ba7cdedf6602f7e6266240d64
Status: Downloaded newer image for mysql:5.6.44
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY            TAG                  IMAGE ID            CREATED             SIZE
nginx                 v0.2-20200118-1225   35c0c66e03ac        2 hours ago         448MB
jason/centos7-nginx   v0.1.20200114        7372d16c99bc        28 hours ago        448MB
jason/centos7-nginx   latest               c4e0980a825a        28 hours ago        448MB
mysql                 5.6.44               c30095c52827        6 months ago        256MB
centos                centos7.6.1810       f1cb7c7d58b7        10 months ago       202MB
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.启动MySQL容器的镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker container run -it --rm mysql:5.6.44 　　　　　　　　　　　　#直接启动时可能会报错，如下所示，那是因为咱们没有为容器的MySQL指定密码，使用"-e"参数指定密码即可。
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container run -it --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=yinzhengjie mysql:5.6.44 　　　　　　　　　　#注意，该容器退出后会自动删除哟~
Initializing database
2020-01-18 14:26:19 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-01-18 14:26:19 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2020-01-18 14:26:19 0 [Note] /usr/sbin/mysqld (mysqld 5.6.44) starting as process 34 ...
2020-01-18 14:26:19 34 [Note] InnoDB: Using atomics to ref count buffer pool pages
2020-01-18 14:26:19 34 [Note] InnoDB: The InnoDB memory heap is disabled
2020-01-18 14:26:19 34 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2020-01-18 14:26:19 34 [Note] InnoDB: Memory barrier is not used
2020-01-18 14:26:19 34 [Note] InnoDB: Compressed tables use zlib 1.2.11
2020-01-18 14:26:19 34 [Note] InnoDB: Using Linux native AIO
2020-01-18 14:26:19 34 [Note] InnoDB: Using CPU crc32 instructions
2020-01-18 14:26:19 34 [Note] InnoDB: Initializing buffer pool, size = 128.0M

......
MySQL init process done. Ready for start up.

2020-01-18 14:26:29 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-01-18 14:26:29 0 [Note] mysqld (mysqld 5.6.44) starting as process 1 ...
2020-01-18 14:26:29 1 [Note] Plugin 'FEDERATED' is disabled.
2020-01-18 14:26:29 1 [Note] InnoDB: Using atomics to ref count buffer pool pages
2020-01-18 14:26:29 1 [Note] InnoDB: The InnoDB memory heap is disabled
2020-01-18 14:26:29 1 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2020-01-18 14:26:29 1 [Note] InnoDB: Memory barrier is not used
2020-01-18 14:26:29 1 [Note] InnoDB: Compressed tables use zlib 1.2.11
2020-01-18 14:26:29 1 [Note] InnoDB: Using Linux native AIO
2020-01-18 14:26:29 1 [Note] InnoDB: Using CPU crc32 instructions
2020-01-18 14:26:29 1 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2020-01-18 14:26:29 1 [Note] InnoDB: Completed initialization of buffer pool
2020-01-18 14:26:29 1 [Note] InnoDB: Highest supported file format is Barracuda.
2020-01-18 14:26:29 1 [Note] InnoDB: 128 rollback segment(s) are active.
2020-01-18 14:26:29 1 [Note] InnoDB: Waiting for purge to start
2020-01-18 14:26:29 1 [Note] InnoDB: 5.6.44 started; log sequence number 1625997
2020-01-18 14:26:29 1 [Note] Server hostname (bind-address): '*'; port: 3306
2020-01-18 14:26:29 1 [Note] IPv6 is available.
2020-01-18 14:26:29 1 [Note]   - '::' resolves to '::';
2020-01-18 14:26:29 1 [Note] Server socket created on IP: '::'.
2020-01-18 14:26:29 1 [Warning] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2020-01-18 14:26:29 1 [Warning] 'proxies_priv' entry '@ root@d1197bf0e557' ignored in --skip-name-resolve mode.
2020-01-18 14:26:29 1 [Note] Event Scheduler: Loaded 0 events
2020-01-18 14:26:29 1 [Note] mysqld: ready for connections.
Version: '5.6.44'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**6>.使用容器启动MySQL时的注意事项（应该单独指定存储卷路径，否则容器消亡后数据也会跟着丢失啦）**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED              STATUS              PORTS                         NAMES
d1197bf0e557        mysql:5.6.44               "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp        vibrant_rosalind
d517b3a369ef        nginx:v0.2-20200118-1225   "nginx"                  2 hours ago          Up 2 hours          0.0.0.0:80->80/tcp, 443/tcp   myNginx
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# docker container exec -it d1197bf0e557 bash
root@d1197bf0e557:/# 
root@d1197bf0e557:/# ls -l /var/lib/mysql/
total 110604
-rw-rw----. 1 mysql mysql       56 Jan 18 14:26 auto.cnf
-rw-rw----. 1 mysql mysql 50331648 Jan 18 14:26 ib_logfile0
-rw-rw----. 1 mysql mysql 50331648 Jan 18 14:26 ib_logfile1
-rw-rw----. 1 mysql mysql 12582912 Jan 18 14:26 ibdata1
drwx------. 2 mysql mysql     4096 Jan 18 14:26 mysql
drwx------. 2 mysql mysql     4096 Jan 18 14:26 performance_schema
root@d1197bf0e557:/# 
root@d1197bf0e557:/# 
root@d1197bf0e557:/# 
root@d1197bf0e557:/# 
root@d1197bf0e557:/# grep data /etc/mysql/ -R
/etc/mysql/my.cnf.fallback:# The MySQL database server configuration file.
/etc/mysql/mysql.conf.d/mysqld.cnf:datadir        = /var/lib/mysql　　　　　　　　　　　　　　#生产环境中，我们应该将该目录挂载到宿主机对应的路径哟~
root@d1197bf0e557:/# 
root@d1197bf0e557:/# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.连接MySQL镜像的服务器**

**1>.宿主机安装mysql的客户端**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# yum -y install mysql
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.bit.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirror.bit.edu.cn
base                                                                                                                                                                                                                                                   | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                                                                                                       | 3.5 kB  00:00:00     
extras                                                                                                                                                                                                                                                 | 2.9 kB  00:00:00     
pouch-stable                                                                                                                                                                                                                                           | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                                                                | 2.9 kB  00:00:00     
updates/7/x86_64/primary_db                                                                                                                                                                                                                            | 5.9 MB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package mariadb.x86_64 1:5.5.64-1.el7 will be installed
--> Processing Dependency: mariadb-libs(x86-64) = 1:5.5.64-1.el7 for package: 1:mariadb-5.5.64-1.el7.x86_64
--> Running transaction check
---> Package mariadb-libs.x86_64 1:5.5.60-1.el7_5 will be updated
---> Package mariadb-libs.x86_64 1:5.5.64-1.el7 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================================================
 Package                                                             Arch                                                          Version                                                                  Repository                                                   Size
==============================================================================================================================================================================================================================================================================
Installing:
 mariadb                                                             x86_64                                                        1:5.5.64-1.el7                                                           base                                                        8.7 M
Updating for dependencies:
 mariadb-libs                                                        x86_64                                                        1:5.5.64-1.el7                                                           base                                                        759 k

Transaction Summary
==============================================================================================================================================================================================================================================================================
Install  1 Package
Upgrade             ( 1 Dependent package)

Total download size: 9.5 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/2): mariadb-libs-5.5.64-1.el7.x86_64.rpm                                                                                                                                                                                                            | 759 kB  00:00:00     
(2/2): mariadb-5.5.64-1.el7.x86_64.rpm                                                                                                                                                                                                                 | 8.7 MB  00:00:01     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                         7.8 MB/s | 9.5 MB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:mariadb-libs-5.5.64-1.el7.x86_64                                                                                                                                                                                                                         1/3 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Installing : 1:mariadb-5.5.64-1.el7.x86_64                                                                                                                                                                                                                              2/3 
  Cleanup    : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                                                                                                                                                                                                                       3/3 
/sbin/ldconfig: /lib64/libnvidia-container.so.1 is not a symbolic link

  Verifying  : 1:mariadb-libs-5.5.64-1.el7.x86_64                                                                                                                                                                                                                         1/3 
  Verifying  : 1:mariadb-5.5.64-1.el7.x86_64                                                                                                                                                                                                                              2/3 
  Verifying  : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                                                                                                                                                                                                                       3/3 

Installed:
  mariadb.x86_64 1:5.5.64-1.el7                                                                                                                                                                                                                                               

Dependency Updated:
  mariadb-libs.x86_64 1:5.5.64-1.el7                                                                                                                                                                                                                                          

Complete!
[root@docker201.yinzhengjie.org.cn ~]# 
```

\[root@docker201.yinzhengjie.org.cn ~\]# yum -y install mysql

**2>.连接MySQL Docker容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker201.yinzhengjie.org.cn ~]# hostname -i
192.168.6.201
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      100                                                                                                 127.0.0.1:25                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::3306                                                                                                                   :::*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
LISTEN     0      100                                                                                                       ::1:25                                                                                                                     :::*                  
[root@docker201.yinzhengjie.org.cn ~]# 
[root@docker201.yinzhengjie.org.cn ~]# mysql -h 192.168.6.201 -uroot -pyinzhengjie
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.44 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> 
MySQL [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

MySQL [(none)]> 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186