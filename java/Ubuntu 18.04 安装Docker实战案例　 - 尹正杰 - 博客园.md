　　　　　　　　　　　　　　　　　　**Ubuntu 18.04 安装Docker实战案例**　

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.通过ubantu安装docker**

**1>.查看操作平台**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112155911850-1198955692.png)**

**2>.****`安装必要的一些系统工具`**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# apt-get update
Hit:1 http://mirrors.aliyun.com/ubuntu bionic InRelease
Hit:2 http://mirrors.aliyun.com/ubuntu bionic-security InRelease
Hit:3 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease                  
Hit:4 http://mirrors.aliyun.com/ubuntu bionic-proposed InRelease                 
Hit:5 http://mirrors.aliyun.com/ubuntu bionic-backports InRelease               
Reading package lists... Done                                                   
root@docker101:~# 
```

root@docker101:~# apt-get update

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# apt-get -y install apt-transport-https ca-certificates curl software-properties-common
Reading package lists... Done
Building dependency tree       
Reading state information... Done
ca-certificates is already the newest version (20180409).
ca-certificates set to manually installed.
curl is already the newest version (7.58.0-2ubuntu3.8).
curl set to manually installed.
The following additional packages will be installed:
  python3-software-properties
The following NEW packages will be installed:
  apt-transport-https
The following packages will be upgraded:
  python3-software-properties software-properties-common
2 upgraded, 1 newly installed, 0 to remove and 71 not upgraded.
Need to get 35.3 kB of archives.
After this operation, 153 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic-updates/universe amd64 apt-transport-https all 1.6.12 [1,692 B]
Get:2 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 software-properties-common all 0.96.24.32.12 [10.0 kB]
Get:3 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 python3-software-properties all 0.96.24.32.12 [23.6 kB]
Fetched 35.3 kB in 0s (126 kB/s)                  
Selecting previously unselected package apt-transport-https.
(Reading database ... 66944 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_1.6.12_all.deb ...
Unpacking apt-transport-https (1.6.12) ...
Preparing to unpack .../software-properties-common_0.96.24.32.12_all.deb ...
Unpacking software-properties-common (0.96.24.32.12) over (0.96.24.32.9) ...
Preparing to unpack .../python3-software-properties_0.96.24.32.12_all.deb ...
Unpacking python3-software-properties (0.96.24.32.12) over (0.96.24.32.9) ...
Setting up apt-transport-https (1.6.12) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Setting up python3-software-properties (0.96.24.32.12) ...
Processing triggers for dbus (1.12.2-1ubuntu1.1) ...
Setting up software-properties-common (0.96.24.32.12) ...
root@docker101:~# 
```

root@docker101:~# apt-get -y install apt-transport-https ca-certificates curl software-properties-common

**3>.****`安装GPG证书`**

```
root@docker101:~# curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
OK
root@docker101:~# 
```

**4>.****`写入软件源信息`**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
Hit:1 http://mirrors.aliyun.com/ubuntu bionic InRelease
Hit:2 http://mirrors.aliyun.com/ubuntu bionic-security InRelease                 
Hit:3 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease                                        
Get:4 https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic InRelease [64.4 kB]                                                    
Hit:5 http://mirrors.aliyun.com/ubuntu bionic-proposed InRelease                                                                      
Hit:6 http://mirrors.aliyun.com/ubuntu bionic-backports InRelease                                                   
Get:7 https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic/stable amd64 Packages [9,852 B]
Fetched 74.3 kB in 1s (103 kB/s)      
Reading package lists... Done
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.****`更新并安装Docker-CE`**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# apt-get -y update
Hit:1 http://mirrors.aliyun.com/ubuntu bionic InRelease
Hit:2 http://mirrors.aliyun.com/ubuntu bionic-security InRelease                                                                   
Hit:3 https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic InRelease                                                           
Hit:4 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease                                                                    
Hit:5 http://mirrors.aliyun.com/ubuntu bionic-proposed InRelease                
Hit:6 http://mirrors.aliyun.com/ubuntu bionic-backports InRelease               
Reading package lists... Done                                                   
root@docker101:~# 
```

root@docker101:~# apt-get -y update

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
root@docker101:~# apt-get -y install docker-ce
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  aufs-tools cgroupfs-mount containerd.io docker-ce-cli libltdl7 pigz
The following NEW packages will be installed:
  aufs-tools cgroupfs-mount containerd.io docker-ce docker-ce-cli libltdl7 pigz
0 upgraded, 7 newly installed, 0 to remove and 71 not upgraded.
Need to get 85.5 MB of archives.
After this operation, 384 MB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu bionic/universe amd64 pigz amd64 2.4-1 [57.4 kB]
Get:2 https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic/stable amd64 containerd.io amd64 1.2.10-3 [20.0 MB]
Get:3 http://mirrors.aliyun.com/ubuntu bionic/universe amd64 aufs-tools amd64 1:4.9+20170918-1ubuntu1 [104 kB]
Get:4 http://mirrors.aliyun.com/ubuntu bionic/universe amd64 cgroupfs-mount all 1.4 [6,320 B]
Get:5 http://mirrors.aliyun.com/ubuntu bionic/main amd64 libltdl7 amd64 2.4.6-2 [38.8 kB]
Get:6 https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic/stable amd64 docker-ce-cli amd64 5:19.03.5~3-0~ubuntu-bionic [42.5 MB]
Get:7 https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic/stable amd64 docker-ce amd64 5:19.03.5~3-0~ubuntu-bionic [22.8 MB]
Fetched 85.5 MB in 8s (11.0 MB/s)                                                                                                                                                                                                                                            
Selecting previously unselected package pigz.
(Reading database ... 66948 files and directories currently installed.)
Preparing to unpack .../0-pigz_2.4-1_amd64.deb ...
Unpacking pigz (2.4-1) ...
Selecting previously unselected package aufs-tools.
Preparing to unpack .../1-aufs-tools_1%3a4.9+20170918-1ubuntu1_amd64.deb ...
Unpacking aufs-tools (1:4.9+20170918-1ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../2-cgroupfs-mount_1.4_all.deb ...
Unpacking cgroupfs-mount (1.4) ...
Selecting previously unselected package containerd.io.
Preparing to unpack .../3-containerd.io_1.2.10-3_amd64.deb ...
Unpacking containerd.io (1.2.10-3) ...
Selecting previously unselected package docker-ce-cli.
Preparing to unpack .../4-docker-ce-cli_5%3a19.03.5~3-0~ubuntu-bionic_amd64.deb ...
Unpacking docker-ce-cli (5:19.03.5~3-0~ubuntu-bionic) ...
Selecting previously unselected package docker-ce.
Preparing to unpack .../5-docker-ce_5%3a19.03.5~3-0~ubuntu-bionic_amd64.deb ...
Unpacking docker-ce (5:19.03.5~3-0~ubuntu-bionic) ...
Selecting previously unselected package libltdl7:amd64.
Preparing to unpack .../6-libltdl7_2.4.6-2_amd64.deb ...
Unpacking libltdl7:amd64 (2.4.6-2) ...
Setting up aufs-tools (1:4.9+20170918-1ubuntu1) ...
Setting up containerd.io (1.2.10-3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Processing triggers for ureadahead (0.100.0-21) ...
Setting up cgroupfs-mount (1.4) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for systemd (237-3ubuntu10.29) ...
Setting up libltdl7:amd64 (2.4.6-2) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Setting up docker-ce-cli (5:19.03.5~3-0~ubuntu-bionic) ...
Setting up pigz (2.4-1) ...
Setting up docker-ce (5:19.03.5~3-0~ubuntu-bionic) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Processing triggers for systemd (237-3ubuntu10.29) ...
root@docker101:~# 
root@docker101:~# 
```

root@docker101:~# apt-get -y install docker-ce

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# systemctl status docker　　　　　　　　　　　　　　　　#安装成功后默认docker是启动的
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-01-12 08:03:46 UTC; 2min 2s ago
     Docs: https://docs.docker.com
 Main PID: 5634 (dockerd)
    Tasks: 10
   CGroup: /system.slice/docker.service
           └─5634 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.061865687Z" level=warning msg="Your kernel does not support swap memory limit"
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.061970486Z" level=warning msg="Your kernel does not support cgroup rt period"
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.062000817Z" level=warning msg="Your kernel does not support cgroup rt runtime"
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.062480825Z" level=info msg="Loading containers: start."
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.340909890Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.578008607Z" level=info msg="Loading containers: done."
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.670981992Z" level=info msg="Docker daemon" commit=633a0ea838 graphdriver(s)=overlay2 version=19.03.5
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.671108044Z" level=info msg="Daemon has completed initialization"
Jan 12 08:03:46 docker101.yinzhengjie.org.cn systemd[1]: Started Docker Application Container Engine.
Jan 12 08:03:46 docker101.yinzhengjie.org.cn dockerd[5634]: time="2020-01-12T08:03:46.756901996Z" level=info msg="API listen on /var/run/docker.sock"
root@docker101:~# 
root@docker101:~# systemctl enable docker　　　　　　　　　　　　#咱们可以将docker设置为开启自启动
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**6>.查看docker的版本**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# docker version 
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea838
 Built:             Wed Nov 13 07:29:52 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea838
  Built:            Wed Nov 13 07:28:22 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**二.docker镜像加速配置**

**1>.获取加速器(https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200112161843018-569005798.png)**

**2>.生成配置文件并重启docker服务**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# sudo mkdir -p /etc/docker
root@docker101:~# sudo tee /etc/docker/daemon.json <<-'EOF'
> {
>   "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"]
> }
> EOF
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"]
}
root@docker101:~# sudo systemctl daemon-reload
root@docker101:~# sudo systemctl restart docker
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**三.下载docker镜像**

**1>.下载一个centos镜像**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# docker image ls　　　　　　　　　　#查看当前的docker镜像为空
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
root@docker101:~# 
root@docker101:~# docker pull centos　　　　　　　　#从官方下载一个docker镜像，本次我们下载的是官方提供的centos镜像。
Using default tag: latest
latest: Pulling from library/centos
729ec3a6ada3: Pull complete 
Digest: sha256:f94c1d992c193b3dc09e297ffd54d8a4f1dc946c37cbeceb26d35ce1647f88d9
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
root@docker101:~# 
root@docker101:~# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              0f3e07c0138f        3 months ago        220MB
root@docker101:~# 
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.基于刚刚下载的docker镜像启动一个容器**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# docker ps　　　　　　　　#查看当前容器情况
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@docker101:~# 
root@docker101:~# docker run -d -it centos　　　　　　#在后端基于centos镜像运行一个可交互的容器
968e7ecc39f277a3d3f98b658f8f496de622edccfa4ef45d8ec64c46f5012d4c
root@docker101:~# 
root@docker101:~# docker ps　　　　　　　　#再次查看当前运行的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
968e7ecc39f2        centos              "/bin/bash"         3 seconds ago       Up 2 seconds                            keen_meitner
root@docker101:~# 
root@docker101:~# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.查看已经创建的容器所在的默认路径**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
root@docker101:~# ll /var/lib/containerd/io.containerd.runtime.v1.linux/moby/　　　　　　　　#注意,这个“moby”就是docker项目名称。
total 12
drwx--x--x 3 root root 4096 Jan 12 08:24 ./
drwx--x--x 3 root root 4096 Jan 12 08:24 ../
drwx--x--x 2 root root 4096 Jan 12 08:24 968e7ecc39f277a3d3f98b658f8f496de622edccfa4ef45d8ec64c46f5012d4c/　　　　　　　　　　#这个数字就是上面我们创建的容器编号。
root@docker101:~# 
root@docker101:~# 
root@docker101:~# ll /var/lib/containerd/io.containerd.runtime.v1.linux/moby/968e7ecc39f277a3d3f98b658f8f496de622edccfa4ef45d8ec64c46f5012d4c/
total 8
drwx--x--x 2 root root 4096 Jan 12 08:24 ./
drwx--x--x 3 root root 4096 Jan 12 08:24 ../
prwx------ 1 root root    0 Jan 12 08:24 shim.stderr.log|
prwx------ 1 root root    0 Jan 12 08:24 shim.stdout.log|
root@docker101:~# 
root@docker101:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
968e7ecc39f2        centos              "/bin/bash"         About a minute ago   Up About a minute                       keen_meitner
root@docker101:~# 
[root@968e7ecc39f2 /]# ls -l
total 48
lrwxrwxrwx   1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x   5 root root  360 Jan 12 08:24 dev
drwxr-xr-x   1 root root 4096 Jan 12 08:24 etc
drwxr-xr-x   2 root root 4096 May 11  2019 home
lrwxrwxrwx   1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx   1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------   2 root root 4096 Sep 27 17:13 lost+found
drwxr-xr-x   2 root root 4096 May 11  2019 media
drwxr-xr-x   2 root root 4096 May 11  2019 mnt
drwxr-xr-x   2 root root 4096 May 11  2019 opt
dr-xr-xr-x 184 root root    0 Jan 12 08:24 proc
dr-xr-x---   2 root root 4096 Sep 27 17:13 root
drwxr-xr-x  11 root root 4096 Sep 27 17:13 run
lrwxrwxrwx   1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 May 11  2019 srv
dr-xr-xr-x  13 root root    0 Jan 12 08:24 sys
drwxrwxrwt   7 root root 4096 Sep 27 17:13 tmp
drwxr-xr-x  12 root root 4096 Sep 27 17:13 usr
drwxr-xr-x  20 root root 4096 Sep 27 17:13 var
[root@968e7ecc39f2 /]# 
[root@968e7ecc39f2 /]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186