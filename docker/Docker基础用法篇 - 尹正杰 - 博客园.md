　　　　　　　　　　　　　　　　　　　　**Docker基础用法篇**

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.安装docker**

**1>.依赖的基础环境**

```
　　64 bits CPU
　　Linux Kerner 3.10+(虽说Redhat 2.6.x内核也可以运行Docker，这是由于红帽为其打了很多补丁，但运行的稳定性极差，因此不推荐在生产环境中使用CentOS 6.x版本)
　　Linux Kernel cgroups and namespaces
```

**2>.CentOS 7**

```
    “extras” repository仓库中存在docker的安装包，但是版本较低，因此我们不推荐使用CentOS自带的仓库软件包。
```

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191015214303435-174462352.png)

**3>.下载docker-ce的仓库**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# cd /etc/yum.repos.d/
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# 
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
--2019-10-16 21:32:50--  https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
Resolving mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)... 101.6.8.193, 2402:f000:1:408:8100::1
Connecting to mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)|101.6.8.193|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2424 (2.4K) [application/octet-stream]
Saving to: ‘docker-ce.repo’

100%[=============================================================================================>] 2,424       --.-K/s   in 0s      

2019-10-16 21:32:50 (291 MB/s) - ‘docker-ce.repo’ saved [2424/2424]

[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# 
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# cat docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/stable　　　　　　　　#不难发现这个地址依旧时Docker官网的地址，下载镜像速度可能还是很慢，我们需要手动修改。
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://download.docker.com/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://download.docker.com/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://download.docker.com/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://download.docker.com/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# 
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# 
```

\[root@node101.yinzhengjie.org.cn /etc/yum.repos.d\]# wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo --2019-10-16 21:32:50-- https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo

**![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191015214546770-549080609.png)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# cat docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/gpg
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# 
```

\[root@node101.yinzhengjie.org.cn /etc/yum.repos.d\]# cat docker-ce.repo

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# yum repolist
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.huaweicloud.com
base                                                                                                            | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                | 3.5 kB  00:00:00     
Not using downloaded docker-ce-stable/repomd.xml because it is older than what we have:
  Current   : Sat Oct 12 03:17:20 2019
  Downloaded: Thu Sep  5 01:48:48 2019
extras                                                                                                          | 2.9 kB  00:00:00     
updates                                                                                                         | 2.9 kB  00:00:00     
repo id                                                        repo name                                                         status
base/7/x86_64                                                  CentOS-7 - Base                                                   10,097
docker-ce-stable/x86_64                                        Docker CE Stable - x86_64                                             59　　　　　　#我们发现是有docker安装包的
extras/7/x86_64                                                CentOS-7 - Extras                                                    304
updates/7/x86_64                                               CentOS-7 - Updates                                                   332
repolist: 10,792
[root@node101.yinzhengjie.org.cn /etc/yum.repos.d]# 
```

\[root@node101.yinzhengjie.org.cn /etc/yum.repos.d\]# yum repolist　　　　　　　　#验证是否docker-ce的包

![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191015215305057-218967429.png)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  -O /etc/yum.repos.d/docker-ce.repo 　　　　#除了使用上面的清华源，咱们还可以使用阿里源，一步到位，无需修改。
--2019-10-16 21:47:36--  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
Resolving mirrors.aliyun.com (mirrors.aliyun.com)... 1.180.31.241, 124.116.187.114, 36.99.142.195, ...
Connecting to mirrors.aliyun.com (mirrors.aliyun.com)|1.180.31.241|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2640 (2.6K) [application/octet-stream]
Saving to: ‘/etc/yum.repos.d/docker-ce.repo’

100%[=============================================================================================>] 2,640       --.-K/s   in 0s      

2019-10-16 21:47:36 (29.0 MB/s) - ‘/etc/yum.repos.d/docker-ce.repo’ saved [2640/2640]

[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# cat /etc/yum.repos.d/docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-debuginfo]
name=Docker CE Edge - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-edge-source]
name=Docker CE Edge - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/edge
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo 　　　　#除了使用上面的清华源，咱们还可以使用阿里源，一步到位，无需修改。

**4>.安装Docker-ce**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# yum -y install docker-ce　　　　　　#安装Docker免费版本
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.huaweicloud.com
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 3:19.03.3-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2:2.74 for package: 3:docker-ce-19.03.3-3.el7.x86_64
--> Processing Dependency: containerd.io >= 1.2.2-3 for package: 3:docker-ce-19.03.3-3.el7.x86_64
--> Processing Dependency: docker-ce-cli for package: 3:docker-ce-19.03.3-3.el7.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.107-3.el7 will be installed
---> Package containerd.io.x86_64 0:1.2.10-3.2.el7 will be installed
---> Package docker-ce-cli.x86_64 1:19.03.3-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================================================================
 Package                            Arch                    Version                            Repository                         Size
=======================================================================================================================================
Installing:
 docker-ce                          x86_64                  3:19.03.3-3.el7                    docker-ce-stable                   24 M
Installing for dependencies:
 container-selinux                  noarch                  2:2.107-3.el7                      extras                             39 k
 containerd.io                      x86_64                  1.2.10-3.2.el7                     docker-ce-stable                   23 M
 docker-ce-cli                      x86_64                  1:19.03.3-3.el7                    docker-ce-stable                   39 M

Transaction Summary
=======================================================================================================================================
Install  1 Package (+3 Dependent packages)

Total size: 87 M
Total download size: 23 M
Installed size: 362 M
Downloading packages:
No Presto metadata available for docker-ce-stable
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/containerd.io-1.2.10-3.2.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key
 ID 621e9f35: NOKEYPublic key for containerd.io-1.2.10-3.2.el7.x86_64.rpm is not installed
containerd.io-1.2.10-3.2.el7.x86_64.rpm                                                                         |  23 MB  00:00:04     
Retrieving key from https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : 2:container-selinux-2.107-3.el7.noarch                                                                              1/4 
  Installing : containerd.io-1.2.10-3.2.el7.x86_64                                                                                 2/4 
  Installing : 1:docker-ce-cli-19.03.3-3.el7.x86_64                                                                                3/4 
  Installing : 3:docker-ce-19.03.3-3.el7.x86_64                                                                                    4/4 
  Verifying  : 3:docker-ce-19.03.3-3.el7.x86_64                                                                                    1/4 
  Verifying  : 1:docker-ce-cli-19.03.3-3.el7.x86_64                                                                                2/4 
  Verifying  : containerd.io-1.2.10-3.2.el7.x86_64                                                                                 3/4 
  Verifying  : 2:container-selinux-2.107-3.el7.noarch                                                                              4/4 

Installed:
  docker-ce.x86_64 3:19.03.3-3.el7                                                                                                     

Dependency Installed:
  container-selinux.noarch 2:2.107-3.el7       containerd.io.x86_64 0:1.2.10-3.2.el7       docker-ce-cli.x86_64 1:19.03.3-3.el7      

Complete!
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# yum -y install docker-ce　　　　　　#安装Docker免费版本

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker version
Client: Docker Engine - Community　　　　　　#客户端版本信息
 Version:           19.03.3
 API version:       1.40
 Go version:        go1.12.10
 Git commit:        a872fc2f86
 Built:             Tue Oct  8 00:58:10 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community　　　　　　#服务端版本信息
 Engine:
  Version:          19.03.3
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.10
  Git commit:       a872fc2f86
  Built:            Tue Oct  8 00:56:46 2019
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
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.使用阿里docker**镜像加速器**（需要登录阿里云账号，每个人都有自己的加速器）******

******![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191015221136678-1089035283.png)******

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# mkdir /etc/docker
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# vi /etc/docker/daemon.json
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json　　　　　　#将阿里云的加速地址写入该配置文件中。
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"]
}
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl daemon-reload
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# systemctl start docker
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker info　　　　　　　　#查看docker更详细的信息，可以在"Registry Mirrors"看到加速URL是否配置正确。
Client:
 Debug Mode: false

Server:
 Containers: 0　　　　　　　　　　#当前容器个数
  Running: 0　　　　　　　　　　　#当前处于运行状态的容器
  Paused: 0　　　　　　　　　　　　#暂停状态的容器
  Stopped: 0　　　　　　　　　　　　#停止状态的容器
 Images: 0　　　　　　　　　　　　　#镜像版本
 Server Version: 19.03.3　　　　 #服务器版本
 Storage Driver: overlay2　　　　#存储去打动后端，docker镜像是封层构建联合挂载的，他们要求必须使用特殊的文件系统，我们使用的xfs和ext是不支持的！目前支持aufs和overlay2文件系统，默认使用的就是overlay2文件系统。
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-957.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 3.683GiB
 Name: node101.yinzhengjie.org.cn
 ID: S6JF:PNVN:QCOA:5YM4:URXE:RK5C:S6F6:H3TD:BBMW:BVLK:ZGIN:2KTW
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:　　　　　　　　　　#加速镜像信息，这是我使用的阿里镜像加速，如果你也有这个信息说明你配置是成功的
  https://tuv7rqqq.mirror.aliyuncs.com/
 Live Restore Enabled: false

[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker info　　　　　　　　#查看docker更详细的信息，可以在"Registry Mirrors"看到加速URL是否配置正确。

**二.docker常用操作案例**

**1>.docker search(可以根据关键词搜索镜像文件的命令)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker search --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker search --help　　　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker search nginx
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                             Official build of Nginx.                        12059               [OK]                
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   1673                                    [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   742                                     [OK]
linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   79                                      
bitnami/nginx                     Bitnami nginx Docker Image                      71                                      [OK]
tiangolo/nginx-rtmp               Docker image with Nginx using the nginx-rtmp…   57                                      [OK]
nginxdemos/hello                  NGINX webserver that serves a simple page co…   30                                      [OK]
jlesage/nginx-proxy-manager       Docker container for Nginx Proxy Manager        25                                      [OK]
jc21/nginx-proxy-manager          Docker container for managing Nginx proxy ho…   24                                      
nginx/nginx-ingress               NGINX Ingress Controller for Kubernetes         22                                      
privatebin/nginx-fpm-alpine       PrivateBin running on an Nginx, php-fpm & Al…   18                                      [OK]
schmunk42/nginx-redirect          A very simple container to redirect HTTP tra…   17                                      [OK]
blacklabelops/nginx               Dockerized Nginx Reverse Proxy Server.          12                                      [OK]
centos/nginx-18-centos7           Platform for running nginx 1.8 or building n…   11                                      
centos/nginx-112-centos7          Platform for running nginx 1.12 or building …   10                                      
nginxinc/nginx-unprivileged       Unprivileged NGINX Dockerfiles                  9                                       
nginx/nginx-prometheus-exporter   NGINX Prometheus Exporter                       7                                       
sophos/nginx-vts-exporter         Simple server that scrapes Nginx vts stats a…   5                                       [OK]
1science/nginx                    Nginx Docker images that include Consul Temp…   5                                       [OK]
mailu/nginx                       Mailu nginx frontend                            4                                       [OK]
pebbletech/nginx-proxy            nginx-proxy sets up a container running ngin…   2                                       [OK]
travix/nginx                      NGinx reverse proxy                             2                                       [OK]
ansibleplaybookbundle/nginx-apb   An APB to deploy NGINX                          1                                       [OK]
centos/nginx-110-centos7          Platform for running nginx 1.10 or building …   0                                       
wodby/nginx                       Generic nginx                                   0                                       [OK]
[root@node101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.docker pull(下载docker镜像到本地，该命令等效于"docker image pull")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image push --help　　　　　　#查看该命令的帮助信息

Usage:    docker image push [OPTIONS] NAME[:TAG]

Push an image or a repository to a registry

Options:
      --disable-content-trust   Skip image signing (default true)
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image push --help　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image pull nginx:1.14-alpine　　　　　　　　#注意：alpine版本表示专门用于构建容器小镜象的发行版，生产环境不推荐使用，学习阶段为了节省带宽可以这样玩。
1.14-alpine: Pulling from library/nginx
bdf0201b3a05: Pull complete 
3d0a573c81ed: Pull complete 
8129faeb2eb6: Pull complete 
3dc99f571daf: Pull complete 
Digest: sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
Status: Downloaded newer image for nginx:1.14-alpine
docker.io/library/nginx:1.14-alpine
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image pull nginx:1.14-alpine　　　　　　　　#注意：alpine版本表示专门用于构建容器小镜象的发行版，生产环境不推荐使用，学习阶段为了节省带宽可以这样玩。另外：nginx发行版本编号偶数为稳定版本。

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker pull busybox　　　　　　　　　　　　　　　　　　#注意：busybox镜像可以理解为微小型的Linux虚拟机，据说安卓系统就用到了它。另外：如果我们只是指定仓库名称busybox，它默认回去下载busybox:latest这个tag版本。
Using default tag: latest
latest: Pulling from library/busybox
7c9d20b9b6cd: Pull complete 
Digest: sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker pull busybox　　　　　　　　　　　　　　　　　　#注意：busybox镜像可以理解为微小型的Linux虚拟机，据说安卓系统就用到了它。另外：如果我们只是指定仓库名称busybox，它默认回去下载busybox:latest这个tag版本。

**3>.docker images(列出本地所有可用镜像。该命令等效于"docker image ls")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls --help　　　　　　#查看该命令的帮助信息

Usage:    docker image ls [OPTIONS] [REPOSITORY[:TAG]]

List images

Aliases:
  ls, images, list

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image ls --help　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker images　　　　　　#列出已经下载的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker images　　　　　　　　　　　　#列出已经下载的镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls　　　　　 #列出已经下载的镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image ls　　　　　 　　　　　#列出已经下载的镜像

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls --no-trunc　　　　　　#显示完整的IMAGE ID信息
REPOSITORY          TAG                 IMAGE ID                                                                  CREATED            
 SIZEbusybox             latest              sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d   6 weeks ago        
 1.22MBnginx               1.14-alpine         sha256:8a2fb25a19f5dc1528b7a3fabe8b3145ff57fe10e4f1edac6c718a3cf4aa4b73   6 months ago       
 16MB[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image ls --no-trunc　　　　#显示完整的IMAGE ID信息

**4>.docker rmi(删除指定镜像，该命令等效于"docker image rm")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image rm --help　　　　　　#查看该命令帮助信息

Usage:    docker image rm [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Aliases:
  rm, rmi, remove

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image rm --help　　　　　　　　　　 #查看该命令帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image rm nginx:1.14-alpine　　　　#删除已经下载的镜像，需要指定版本号，若不指定默认会删除nginx:latest镜像哟
Untagged: nginx:1.14-alpine
Untagged: nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
Deleted: sha256:8a2fb25a19f5dc1528b7a3fabe8b3145ff57fe10e4f1edac6c718a3cf4aa4b73
Deleted: sha256:f68a8bcb9dbd06e0d2750eabf63c45f51734a72831ed650d2349775865d5fc20
Deleted: sha256:cbf2c7789332fe231e8defa490527a7b2c3ae8589997ceee00895f3263f0a8cf
Deleted: sha256:894f3fad7e6ecd7f24e88340a44b7b73663a85c0eb7740e7ade169e9d8491a4c
Deleted: sha256:a464c54f93a9e88fc1d33df1e0e39cca427d60145a360962e8f19a1dbf900da9
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker image rm nginx:1.14-alpine　　　　#删除已经下载的镜像，需要指定版本号，若不指定默认会删除nginx:latest镜像哟

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              19485c79a9bb        6 weeks ago         1.22MB
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker rmi busybox　　　　　　#删除busybox镜像
Untagged: busybox:latest
Untagged: busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
Deleted: sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d
Deleted: sha256:6c0ea40aef9d2795f922f4e8642f0cd9ffb9404e6f3214693a1fd45489f38b44
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               1.14-alpine         8a2fb25a19f5        6 months ago        16MB
[root@node101.yinzhengjie.org.cn ~]#  
```

\[root@node101.yinzhengjie.org.cn ~\]# docker rmi busybox　　　　　　　　　　　　　#删除busybox镜像

**5>.docker create(创建容器，该命令等效于"docker container create")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container create --help　　　　　　#查看该命令的帮助信息

Usage:    docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]

Create a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown
                                       (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container create --help　　　　　　#查看该命令的帮助信息

**6>.docker start(启动一个或多个容器，该命令等效于"docker container start")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker start --help　　　　　　#查看该名命令的帮助信息

Usage:    docker start [OPTIONS] CONTAINER [CONTAINER...]

Start one or more stopped containers

Options:
  -a, --attach               Attach STDOUT/STDERR and forward signals
      --detach-keys string   Override the key sequence for detaching a container
  -i, --interactive          Attach container's STDIN
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker start --help　　　　　　#查看该名命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                33 minutes ago      Exited (130) 7 minutes ago                       b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container start -i -a b1　　　　　　#启动一个已经退出的容器
/ # 
/ # 
/ # ps
PID USER TIME COMMAND
1 root 0:00 sh
6 root 0:00 ps
/ #
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container start -i -a b1　　　　　　#启动一个已经退出的容器

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ps　　　　　　　　　　　　　#容器启动成功后，可用看到正在运行的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                35 minutes ago      Up 2 minutes                            b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                35 minutes ago      Up 2 minutes                            b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container ps　　　　　　　　　　　　　#容器启动成功后，可用看到正在运行的容器

**7>.docker stop(停止一个或多个运行中的容器，该命令等效于"docker container stop")**

**8>.docker kill(强行停止一个或多个运行中的容器，该命令等效于"docker container kill")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker kill  --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker kill [OPTIONS] CONTAINER [CONTAINER...]

Kill one or more running containers

Options:
  -s, --signal string   Signal to send to the container (default "KILL")
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker kill --help　　　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                38 minutes ago      Up 5 minutes                            b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker kill  b1　　　　　　#强行停止b1容器的运行
b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker kill b1　　　　　　#强行停止b1容器的运行

**9>.docker run(运行一个新的容器并启动该容器，该命令等效于"docker container run")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker  run --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown
                                       (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --help　　　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name b1 -it busybox:latest　　　　　　#注意，"--name"表示给该容器起一个名称，"-i"选项表示交互式，"-t"表示启动一个终端，启动时指定的Image为"busybox:latest"
/ # 
/ # ps
PID USER TIME COMMAND
1 root 0:00 sh
6 root 0:00 ps
/ # 
/ # ls /bin/ | wc -l
399
/ # 
/ # httpd -h
httpd: option requires an argument -- h
BusyBox v1.31.0 (2019-09-04 17:25:45 UTC) multi-call binary.

Usage: httpd [-ifv[v]] [-c CONFFILE] [-p [IP:]PORT] [-u USER[:GRP]] [-r REALM] [-h HOME]
or httpd -d/-e/-m STRING

Listen for incoming HTTP requests

-i    Inetd mode
-f    Don't daemonize
-v[v]    Verbose
-p [IP:]PORT    Bind to IP:PORT (default *:80)
-u USER[:GRP]    Set uid/gid after binding to port
-r REALM    Authentication Realm for Basic Authentication
-h HOME    Home directory (default .)
-c FILE    Configuration file (default {/etc,HOME}/httpd.conf)
-m STRING    MD5 crypt STRING
-e STRING    HTML encode STRING
-d STRING    URL decode STRING
/ # 
/ #

/ # mkdir /data/html -pv
created directory: '/data/'
created directory: '/data/html'
/ # 
/ # vi /data/html/index.html
/ # 
/ # cat /data/html/index.html
Busybox httpd server.
yinzhengjie dao ci yi you !
/ # 
/ # httpd -f -h /data/html/　　　　　　#此时我们就启动了一个httpd服务，我们可用通过客户端去访问　　
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name b1 -it busybox:latest　　　　　　#注意，"--name"表示给该容器起一个名称，"-i"选项表示交互式，"-t"表示启动一个终端，启动时指定的Image为"busybox:latest"，在busybox中启动一个httpd服务

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container inspect b1 | grep "IPAddress"　　#查看b1容器的IP地址，用于访问b1容器内的httpd服务。　　　　
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 172.17.0.2　　　　　　　　　　#访问b1容器的httpd服务
Busybox httpd server.
yinzhengjie dao ci yi you !
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container inspect b1 | grep "IPAddress"　　#查看b1容器的IP地址，用于访问b1容器内的httpd服务。　

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
/ # 
/ # httpd -f -h /data/html/　　　　　　　　#使用"ctrl + c"终止容器中的进程
^C
/ # 
/ # exit　　　　　　　　　　　　　　　　　　　　#退出b1容器
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker ps　　　　#发现没有正在运行的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker ps -a　　 #不难发现，容器依旧还在只不过状态为Exited啦。
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                25 minutes ago      Exited (130) 11 seconds ago                       b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

/ # httpd -f -h /data/html/　　　　　　　　#使用"ctrl + c"终止容器中的进程

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name web01 -d nginx:1.14-alpine　　　　#启动一个nginx容器之后我们可用访问它
ce713ceaa586a9790b7d11602666826a05e64551bef184cb504dbfcaf5f5ac9a
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   11 seconds ago      Up 11 seconds       80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker inspect web01 | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# curl 172.17.0.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name web01 -d nginx:1.14-alpine　　　　#启动一个nginx容器之后我们可用访问它

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker run --name redis01 -d redis:4-alpine　　　　#启动一个redis容器，注意：如果镜像不存在他会自动去官网下载对应的镜像哟~
Unable to find image 'redis:4-alpine' locally
4-alpine: Pulling from library/redis
9d48c3bd43c5: Pull complete 
6bcae78f4e99: Pull complete 
8cb2d2938e96: Pull complete 
b8ca2dc963b3: Pull complete 
6d2f7873656f: Pull complete 
ad232b4fa0bb: Pull complete 
Digest: sha256:e59e6cab7ada6fa8cfeb9ad4c5b82bf7947bf3620cafc17e19fdbff01b239981
Status: Downloaded newer image for redis:4-alpine
b16c5b671f318bd1d258512df1aa67ff93f2175f3c79067aa59e59128bcfae4c
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   23 seconds ago      Up 22 seconds       6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes        80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container exec -it redis01 /bin/sh
/data # 
/data # ps
PID   USER     TIME  COMMAND
    1 redis     0:00 redis-server
   12 root      0:00 /bin/sh
   17 root      0:00 ps
/data # 
/data # 
/data # netstat -tnl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      
tcp        0      0 :::6379                 :::*                    LISTEN      
/data # 
/data # redis-cli 
127.0.0.1:6379> KEYS *
(empty list or set)
127.0.0.1:6379> 
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> exit
/data # 
/data # exit
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker run --name redis01 -d redis:4-alpine　　　　#启动一个redis容器，注意：如果镜像不存在他会自动去官网下载对应的镜像哟~

**10>.docker rm(删除一个或多个容器，该命令等效于"docker container rm")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker rm --help　　　　　　#查看该命令的帮助信息

Usage:    docker rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove the volumes associated with the container
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker rm --help　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                            PORTS               N
AMEScb5fb9d1adc7        busybox:latest      "sh"                40 minutes ago      Exited (137) About a minute ago                       b
1[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker rm b1　　　　　　　　#删除指定容器
b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker rm b1　　　　　　　　#删除指定容器b1

**11>.docker pause(暂停一个或多个容器，该命令等效于"docker container pause")**

**12>.docker unpause(取消暂停一个或多个容器，该命令等效于"docker container unpause")**

**13>.docker top(显示容器运行的进程信息，可用查看出哪些容器更耗资源，该命令等效于"docker container top")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   9 minutes ago       Up 9 minutes        6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   15 minutes ago      Up 15 minutes       80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker top web01                     #查看web01容器的进程信息
UID                 PID                 PPID                C                   STIME               TTY                 TIME           
     CMDroot                20146               20128               0                   07:42               ?                   00:00:00       
     nginx: master process nginx -g daemon off;100                 20186               20146               0                   07:42               ?                   00:00:00       
     nginx: worker process[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker top web01 #查看web01容器的进程信息

**14>.docker container ls(列出已有容器，等效于"docker container ps")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   13 minutes ago      Up 13 minutes       80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container ls

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   9 minutes ago       Up 9 minutes        6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   14 minutes ago      Up 14 minutes       80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container ps 

**15>.docker ps(显示所有容器进程，等效于"docker container ps",推荐使用"docker container ls")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker ps --help　　　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps　　　　　　　　　　#查看运行的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                11 minutes ago      Up 11 minutes                           b1
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker ps　　　　　　　　　　#查看运行的容器

**16>.docker network(网络管理命令)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker network --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker network --help　　　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker network ls 　　　　　　　　　　#查看可用的网络
NETWORK ID          NAME                DRIVER              SCOPE
c2802fc2d2c3        bridge              bridge              local
9f539144f682        host                host                local
e10670abb710        none                null                local
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker network ls 　　　　　　　　　　#查看可用的网络

**17>.docker inspect(获取指定容器/镜像的元数据信息,该命令等效于"docker container inspect")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                13 minutes ago      Up 13 minutes                           b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker inspect b1　　　　　　#获取指定容器的元数据信息
[
    {
        "Id": "cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15",
        "Created": "2019-10-16T22:58:31.841469017Z",
        "Path": "sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 18948,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-10-16T22:58:32.33407354Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "ResolvConfPath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/hostname",
        "HostsPath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/hosts",
        "LogPath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/cb5fb9d1adc74e1aed93e48
cf463e8f92c5af2e2a08d06a21c668bd12818dd15-json.log",        "Name": "/b1",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f-init/diff:/var/l
ib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/diff",                "MergedDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f/merged",
                "UpperDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f/diff",
                "WorkDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "cb5fb9d1adc7",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "Image": "busybox:latest",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "00e83dbc6f6c139b185206915c601d27d443766de99bda6d643ca6b3ebd76604",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/00e83dbc6f6c",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "2038c26a11ee7376bf03d0dc5d60aab0b2f0519d3098f7413d25a099aea4bf49",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "c2802fc2d2c3198bd6d5257cb83043aabd76e3ecaeea8daafe1b0bb065efb9d4",
                    "EndpointID": "2038c26a11ee7376bf03d0dc5d60aab0b2f0519d3098f7413d25a099aea4bf49",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker inspect b1　　　　　　#获取指定容器的元数据信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cb5fb9d1adc7        busybox:latest      "sh"                18 minutes ago      Up 18 minutes                           b1
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container inspect b1　　　　#获取指定容器的元数据信息
[
    {
        "Id": "cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15",
        "Created": "2019-10-16T22:58:31.841469017Z",
        "Path": "sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 18948,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-10-16T22:58:32.33407354Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:19485c79a9bbdca205fce4f791efeaa2a103e23431434696cc54fdd939e9198d",
        "ResolvConfPath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/hostname",
        "HostsPath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/hosts",
        "LogPath": "/var/lib/docker/containers/cb5fb9d1adc74e1aed93e48cf463e8f92c5af2e2a08d06a21c668bd12818dd15/cb5fb9d1adc74e1aed93e48
cf463e8f92c5af2e2a08d06a21c668bd12818dd15-json.log",        "Name": "/b1",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f-init/diff:/var/l
ib/docker/overlay2/b770b052c1df66b22f2bbc171e637e15b1cc3ff49b4ae3eca3d32f5ad1b778d3/diff",                "MergedDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f/merged",
                "UpperDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f/diff",
                "WorkDir": "/var/lib/docker/overlay2/8fc38355b69e4f7e15dbbdaae09e14bbee4fa8f64507b0ba66d91d7ac7a94f2f/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "cb5fb9d1adc7",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "sh"
            ],
            "Image": "busybox:latest",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "00e83dbc6f6c139b185206915c601d27d443766de99bda6d643ca6b3ebd76604",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/00e83dbc6f6c",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "2038c26a11ee7376bf03d0dc5d60aab0b2f0519d3098f7413d25a099aea4bf49",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "c2802fc2d2c3198bd6d5257cb83043aabd76e3ecaeea8daafe1b0bb065efb9d4",
                    "EndpointID": "2038c26a11ee7376bf03d0dc5d60aab0b2f0519d3098f7413d25a099aea4bf49",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container inspect b1　　　　#获取指定容器的元数据信息

**18>.docker exec(在运行中的容器运行命令,该命令等效于"docker container exec")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker  exec --help　　　　　　　　#查看该命令的帮助信息

Usage:    docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container exec --help

Usage:    docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker exec --help　　　　　　　　#查看该命令的帮助信息

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   23 seconds ago      Up 22 seconds       6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes        80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container exec -it redis01 /bin/sh　　　　　　#注意，-i表示启用交互式，-t表示启动一个终端，解释器类型不支持/bin/bash,因此我们这里需要使用"/bin/sh"
/data # 
/data # ps
PID   USER     TIME  COMMAND
    1 redis     0:00 redis-server
   12 root      0:00 /bin/sh
   17 root      0:00 ps
/data # 
/data # 
/data # netstat -tnl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      
tcp        0      0 :::6379                 :::*                    LISTEN      
/data # 
/data # redis-cli 
127.0.0.1:6379> KEYS *
(empty list or set)
127.0.0.1:6379> 
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> exit
/data # 
/data # exit
[root@node101.yinzhengjie.org.cn ~]# 
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container exec -it redis01 /bin/sh　　　　　　#注意，-i表示启用交互式，-t表示启动一个终端，解释器类型不支持/bin/bash,因此我们这里需要使用"/bin/sh"

**19>.docker logs(获取容器的log信息，该命令等效于"docker container logs")**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@node101.yinzhengjie.org.cn ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b16c5b671f31        redis:4-alpine      "docker-entrypoint.s…"   9 minutes ago       Up 9 minutes        6379/tcp            redis01
ce713ceaa586        nginx:1.14-alpine   "nginx -g 'daemon of…"   15 minutes ago      Up 15 minutes       80/tcp              web01
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]# docker container logs web01　　　　　　#显示web01容器的日志信息
172.17.0.1 - - [16/Oct/2019:23:44:58 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
[root@node101.yinzhengjie.org.cn ~]# 
[root@node101.yinzhengjie.org.cn ~]#  
```

\[root@node101.yinzhengjie.org.cn ~\]# docker container logs web01　　　　　　#显示web01容器的日志信息

**20>.**

**三.****docker event state（docker命令的状态转换关系）******

 ![](https://img2018.cnblogs.com/blog/795254/201910/795254-20191016081612240-952398805.png)

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186