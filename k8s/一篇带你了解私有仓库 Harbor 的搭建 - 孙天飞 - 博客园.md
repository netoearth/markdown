## 一、Harbor简介

-   虽然Docker官方提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境内的Registry也是非常必要的。
    
-   Harbor是由VMware公司开源的企业级的Docker Registry管理项目，相比docker官方拥有更丰富的权限权利和完善的架构设计，适用大规模docker集群部署提供仓库服务。
    
-   它主要提供 Dcoker Registry 管理界面UI，可基于角色访问控制,镜像复制， AD/LDAP 集成，日志审核等功能，完全的支持中文。  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200423143002665.png)
    

## 二、Harbor 的主要功能

-   基于角色的访问控制

> 用户与Docker镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。

-   基于镜像的复制策略

> 镜像可以在多个Registry实例中复制（可以将仓库中的镜像同步到远程的Harbor，类似于MySQL主从同步功能），尤其适合于负载均衡，高可用，混合云和多云的场景。

-   图形化用户界面

> 用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间。

-   支持 AD/LDAP

> Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理。

-   镜像删除和垃圾回收

> Harbor支持在Web删除镜像，回收无用的镜像，释放磁盘空间。image可以被删除并且回收image占用的空间。

-   审计管理

> 所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。

-   RESTful API

> RESTful API 提供给管理员对于Harbor更多的操控, 使得与其它管理软件集成变得更容易。

-   部署简单

> 提供在线和离线两种安装工具， 也可以安装到vSphere平台(OVA方式)虚拟设备。

Harbor 的所有组件都在 Docker 中部署，所以 Harbor 可使用 Docker Compose 快速部署。  
注意： 由于 Harbor 是基于 Docker Registry V2 版本，所以 docker 版本必须 > = 1.10.0 docker-compose >= 1.6.0

## 三、Harbor 架构组件

架构组件图：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200423135758799.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4MzYxNTQx,size_16,color_FFFFFF,t_70)  
1、Proxy：反向代理工具

2、Registry：负责存储docker镜像，处理上传/下载命令。对用户进行访问控制，它指向一个token服务，强制用户的每次docker pull/push请求都要携带一个合法的token，registry会通过公钥对token进行解密验证。

3、Core service：Harbor的核心功能：

-   UI：图形界面
-   Webhook：及时获取registry上image状态变化情况，在registry上配置 webhook，把状态变化传递给UI模块。
-   Token服务：复杂根据用户权限给每个docker push/p/ull命令签发token。Docker客户端向registry服务发起的请求，如果不包含token，会被重定向到这里，获得token后再重新向registry进行请求。

4、Database：提供数据库服务，存储用户权限，审计日志，docker image分组信息等数据

5、Log collector：为了帮助监控harbor运行，复责收集其他组件的log，供日后进行分析

## 四、Harbor 部署

### 4.1、环境准备

两台虚拟机

harbor (harbor服务端，用于搭建私有仓库)

20.0.0.10 docker-ce、docker-compose（必须安装）、Harbo

client（客户端，用于远程访问私有仓库） 20.0.0.30 docker-ce

4.2、安装compose 和 harbor

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></td><td><div><p><code>-rw-r--r--. 1 root root 533765727 12月&nbsp; 1 18:58 harbor-offline-installer-v1.2.2.tgz</code></p><p><code>将安装包解压缩</code></p><p><code>-rw-r--r--. 1 root root&nbsp; 10867152 12月&nbsp; 1 19:02 docker-compose</code></p><p><code>tar</code> <code>zxvf harbor-offline-installer-v1.2.2.tgz -C </code><code>/usr/local/</code></p><p><code>cd</code> <code>/usr/local/harbor/</code>&nbsp;</p></div></td></tr></tbody></table>

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201201232359047-1437838520.png)

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></td><td><div><p><code>1 上传docker-compose到</code><code>/root</code><code>目录下</code></p><p><code>2</code></p><p><code>3 将docker-compose移动到</code><code>/usr/local/bin</code></p><p><code>4 [root@server1 ~]</code></p><p><code>5 [root@server1 ~]</code></p></div></td></tr></tbody></table>

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></td><td><div><p><code>对harbor配置文件进行修改</code></p><p><code>vi</code> <code>harbor.cfg</code></p><p><code>hostname</code> <code>= 20.0.0.10</code></p><p><code>//</code><code>启动harbor，</code><code>install</code><code>.sh会直接调用docker-compose.yml，进行编排容器服务</code></p><p><code>sh </code><code>/usr/local/harbor/install</code><code>.sh</code></p></div></td></tr></tbody></table>

　　![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201201232548236-1518241473.png)

 ![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201201232645971-1713859685.png)

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p></td><td><div><p><code>//</code><code>到这里，harbor安装已经完成，可以查看Harbor启动的镜像和容器服务</code></p><p><code>docker images&nbsp;&nbsp;&nbsp;&nbsp;</code></p><p><code>docker </code><code>ps</code> <code>-a&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code></p></div></td></tr></tbody></table>

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p></td><td><div><p><code>也可用docker-compose </code><code>ps</code><code>查看容器状态，但是需要在</code><code>/usr/local/harbor</code><code>目录下执行</code></p><p><code>[root@harbor harbor]</code></p><p><code>/usr/local/harbor</code></p><p><code>[root@harbor harbor]</code></p></div></td></tr></tbody></table>

4.3、harbor 图形化管理

在harbor.cfg文件里可以找到登录UI界面的默认用户、密码。

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201201232934878-941788377.png)

 打开浏览器输入harbor的IP地址登录UI界面。用账户密码进行登录

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201201233004155-1748877347.png)

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202184234944-826536824.png)

 添加用户

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202184354642-345230309.png)

4.4、推送镜像

此时可使用 Docker 命令在本地通过 127.0.0.1 来登录和推送镜像。默认情况下，Register 服务器在端口 80 上侦听。

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p></td><td><div><p><code>[root@node1 harbor]</code></p><p><code>WARNING! Using --password via the CLI is insecure. Use --password-stdin.</code></p><p><code>WARNING! Your password will be stored unencrypted </code><code>in</code> <code>/root/</code><code>.docker</code><code>/config</code><code>.json.</code></p><p><code>Configure a credential helper to remove this warning. See</code></p><p><code>https:</code><code>//docs</code><code>.docker.com</code><code>/engine/reference/commandline/login/</code></p><p><code>Login Succeeded</code></p></div></td></tr></tbody></table>

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p></td><td><div><p><code>[root@node1 harbor]</code></p><p><code>给镜像打标签</code></p><p><code>[root@node1 harbor]</code></p><p><code>[root@node1 harbor]</code></p><p><code>The push refers to repository [127.0.0.1</code><code>/test/httpd</code><code>]</code></p><p><code>c74375f55aa8: Pushed</code></p><p><code>211b9be55a20: Pushed</code></p><p><code>aa0b3e4b6d3b: Pushed</code></p><p><code>540171a10c83: Pushed</code></p><p><code>f5600c6330da: Pushed</code></p><p><code>v1: digest: sha256:4c7c70926e2f2e10a9f78b63f344c83ae97a22c7fefa96afed46c63e4e607c51 size: 1366　　</code></p></div></td></tr></tbody></table>

进入网页中查看镜像是否上传成功

　![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202190823892-1067028772.png)

4.5、尝试其他服务器登录Harbor仓库

　上面都是本地操作，尝试在其他客户端尝试登录，会出现如下报错

<table><tbody><tr><td><p>1</p><p>2</p></td><td><div><p><code>[root@server3 ~]</code></p><p><code>Error response from daemon: Get https:</code><code>//20</code><code>.0.0.10</code><code>/v2/</code><code>: dial tcp 20.0.0.10:443: connect: connection refused　　</code></p></div></td></tr></tbody></table>

解决方法

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></td><td><div><p><code>[root@server3 ~]</code></p><p><code>ExecStart=</code><code>/usr/bin/dockerd</code> <code>-H fd:</code><code>//</code> <code>--insecure-registry 20.0.0.10 --containerd=</code><code>/run/containerd/containerd</code><code>.sock&nbsp;&nbsp;&nbsp;</code></p><p><code>[root@server3 ~]</code></p><p><code>[root@server3 ~]</code></p></div></td></tr></tbody></table>

重新登录，登录成功

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p></td><td><div><p><code>[root@server3 ~]</code></p><p><code>WARNING! Your password will be stored unencrypted </code><code>in</code> <code>/root/</code><code>.docker</code><code>/config</code><code>.json.</code></p><p><code>Configure a credential helper to remove this warning. See</code></p><p><code>https:</code><code>//docs</code><code>.docker.com</code><code>/engine/reference/commandline/login/</code></p><p><code>Login Succeeded　　</code></p></div></td></tr></tbody></table>

下载镜像

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p></td><td><div><p><code>[root@server3 ~]</code></p><p><code>对镜像进行打标签</code></p><p><code>[root@server3 ~]</code></p><p><code>[root@server3 ~]</code></p></div></td></tr></tbody></table>

登录网页进行查看，镜像是否上传成功

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202192658230-845098569.png)

##  4.6、harbor的关闭与开启

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p><p>31</p><p>32</p><p>33</p><p>34</p><p>35</p><p>36</p><p>37</p><p>38</p><p>39</p><p>40</p><p>41</p><p>42</p><p>43</p><p>44</p><p>45</p></td><td><div><p><code>关闭（修改配置文件必须先关闭服务）</code></p><p><code>[root@node1 harbor]</code></p><p><code>Stopping harbor-jobservice&nbsp; ... </code><code>done</code></p><p><code>Stopping nginx&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping harbor-ui&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping harbor-adminserver ... </code><code>done</code></p><p><code>Stopping harbor-db&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping registry&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping harbor-log&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-jobservice&nbsp; ... </code><code>done</code></p><p><code>Removing nginx&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-ui&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-adminserver ... </code><code>done</code></p><p><code>Removing harbor-db&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing registry&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-log&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing network harbor_harbor</code></p><p><code>查看容器的状态</code></p><p><code>[root@node1 harbor]</code></p><p><code>CONTAINER ID&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IMAGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; COMMAND&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CREATED&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; STATUS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PORTS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NAMES</code></p><p><code>再次开始harbor</code></p><p><code>[root@node1 harbor]</code></p><p><code>Creating network </code><code>"harbor_harbor"</code> <code>with the default driver</code></p><p><code>Creating harbor-log ... </code><code>done</code></p><p><code>Creating harbor-adminserver ... </code><code>done</code></p><p><code>Creating registry&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Creating harbor-db&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Creating harbor-ui&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Creating nginx&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Creating harbor-jobservice&nbsp; ... </code><code>done</code></p><p><code>[root@node1 harbor]</code></p><p><code>重新查看容器的状态</code></p><p><code>[root@node1 harbor]</code></p><p><code>CONTAINER ID&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IMAGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; COMMAND&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CREATED&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; STATUS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PORTS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NAMES</code></p><p><code>0e89e37ae455&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/harbor-jobservice</code><code>:v1.2.2&nbsp;&nbsp;&nbsp; </code><code>"/harbor/harbor_jobs…"</code>&nbsp;&nbsp; <code>37 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 35 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; harbor-jobservice</code></p><p><code>1ceaa3c7bdac&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/nginx-photon</code><code>:1.11.13&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code>"nginx -g 'daemon of…"</code>&nbsp;&nbsp; <code>37 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 35 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0.0.0.0:80-&gt;80</code><code>/tcp</code><code>, 0.0.0.0:443-&gt;443</code><code>/tcp</code><code>, 0.0.0.0:4443-&gt;4443</code><code>/tcp</code>&nbsp;&nbsp; <code>nginx</code></p><p><code>6f70d6b9379b&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/harbor-ui</code><code>:v1.2.2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code>"/harbor/harbor_ui"</code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <code>37 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 36 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; harbor-ui</code></p><p><code>348dc4a931e8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/harbor-db</code><code>:v1.2.2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code>"docker-entrypoint.s…"</code>&nbsp;&nbsp; <code>38 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 36 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 3306</code><code>/tcp</code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <code>harbor-db</code></p><p><code>519de971e723&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/registry</code><code>:2.6.2-photon&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code>"/entrypoint.sh serv…"</code>&nbsp;&nbsp; <code>38 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 36 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 5000</code><code>/tcp</code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <code>registry</code></p><p><code>47b28cc9a461&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/harbor-adminserver</code><code>:v1.2.2&nbsp;&nbsp; </code><code>"/harbor/harbor_admi…"</code>&nbsp;&nbsp; <code>38 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 36 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; harbor-adminserver</code></p><p><code>65c16de60a34&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vmware</code><code>/harbor-log</code><code>:v1.2.2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code>"/bin/sh -c 'crond &amp;…"</code>&nbsp;&nbsp; <code>38 seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up 37 seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 127.0.0.1:1514-&gt;514</code><code>/tcp</code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <code>harbor-log</code></p></div></td></tr></tbody></table>

##  五、创建harbor用户

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202193334445-492397291.png)

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202193351957-1294691397.png)

 ![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202193503497-759056075.png)

 设为管理员

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202193538997-603050232.png)

 在项目中添加成员

![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202193816248-777908869.png)

 ![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202193918162-1625416058.png)

##  5.1、用stf用户登录

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p><p>23</p><p>24</p><p>25</p><p>26</p><p>27</p><p>28</p><p>29</p><p>30</p></td><td><div><p><code>先退出登录</code></p><p><code>[root@node1 harbor]</code></p><p><code>Removing login credentials </code><code>for</code> <code>20.0.0.10</code></p><p><code>用stf用户进行登录</code></p><p><code>[root@node1 harbor]</code></p><p><code>WARNING! Using --password via the CLI is insecure. Use --password-stdin.</code></p><p><code>WARNING! Your password will be stored unencrypted </code><code>in</code> <code>/root/</code><code>.docker</code><code>/config</code><code>.json.</code></p><p><code>Configure a credential helper to remove this warning. See</code></p><p><code>https:</code><code>//docs</code><code>.docker.com</code><code>/engine/reference/commandline/login/</code></p><p><code>Login Succeeded</code></p><p><code>从私有仓库中下载镜像文件</code></p><p><code>[root@node1 harbor]</code></p><p><code>v1: Pulling from </code><code>test</code><code>/tomcat</code></p><p><code>756975cb9c7e: Pull complete</code></p><p><code>d77915b4e630: Pull complete</code></p><p><code>5f37a0a41b6b: Pull complete</code></p><p><code>96b2c1e36db5: Pull complete</code></p><p><code>27a2d52b526e: Pull complete</code></p><p><code>a867dba77389: Pull complete</code></p><p><code>0939c055fb79: Pull complete</code></p><p><code>0b0694ce0ae2: Pull complete</code></p><p><code>81a5f8099e05: Pull complete</code></p><p><code>c3d7917d545e: Pull complete</code></p><p><code>Digest: sha256:4527a552568f7d706173d8065278cd1abaa7edce186a149a5a2de251e12e6c3c</code></p><p><code>Status: Downloaded newer image </code><code>for</code> <code>20.0.0.10</code><code>/test/tomcat</code><code>:v1</code></p><p><code>20.0.0.10</code><code>/test/tomcat</code><code>:v1</code></p><p><code>拉取成功</code></p></div></td></tr></tbody></table>

　![](https://img2020.cnblogs.com/blog/2091211/202012/2091211-20201202194854306-1434446570.png)

5.2、移除Harbor 服务器同时保留镜像数据和数据库

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p></td><td><div><p><code>[root@node1 harbor]</code></p><p><code>Stopping harbor-jobservice&nbsp; ... </code><code>done</code></p><p><code>Stopping nginx&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping harbor-ui&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping harbor-db&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping registry&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Stopping harbor-adminserver ... </code><code>done</code></p><p><code>Stopping harbor-log&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-jobservice&nbsp; ... </code><code>done</code></p><p><code>Removing nginx&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-ui&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-db&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing registry&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing harbor-adminserver ... </code><code>done</code></p><p><code>Removing harbor-log&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... </code><code>done</code></p><p><code>Removing network harbor_harbor</code></p><p><code>如需重新部署，需要移除 Harbor 服务容器全部数据，持久数据，如镜像，数据库等在宿主机的</code><code>/data/</code><code>目录下,日志在宿主机的</code></p><p><code>1 </code><code>/var/log/Harbor/</code><code>目录下。</code></p><p><code>2 </code><code>rm</code> <code>-rf </code><code>/data/database/</code></p><p><code>3 </code><code>rm</code> <code>-rf </code><code>/data/registry/</code></p></div></td></tr></tbody></table>