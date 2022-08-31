本文简要介绍 SeaweedFS 的搭建指南，以及使用方法。搭建部分主要参考[这篇文章](http://www.diyhi.com/article-7.html)。本文基于以下版本：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>Go 1.17.5</span><br><span>SeaweedFS 2.84</span><br><span>Redis 6.2.6</span><br></pre></td></tr></tbody></table>

安装完成后，master 的运行服务如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>redis_6379</span><br><span>weed-master</span><br></pre></td></tr></tbody></table>

所有 worker 的运行服务如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>weed-volume</span><br><span>weed-filer</span><br><span>weed-s3</span><br></pre></td></tr></tbody></table>

## 准备工作

我们在这里准备了四台机器，一台 master 与三台 worker，已经全部配置好 hosts。

### 安装 Go

请参考[官网的安装教程](https://go.dev/doc/install)。命令简要列举如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>wget https://go.dev/dl/go1.17.5.linux-amd64.tar.gz</span><br><span>rm -rf /usr/<span>local</span>/go &amp;&amp; tar -C /usr/<span>local</span> -xzf go1.17.5.linux-amd64.tar.gz</span><br></pre></td></tr></tbody></table>

将以下命令添加到 `/etc/profile`：

/etc/profile

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span><span>export</span> PATH=<span>$PATH</span>:/usr/<span>local</span>/go/bin</span><br></pre></td></tr></tbody></table>

Go 需要在所有机器上安装。

### 安装 Redis

我们使用 Redis 存储文件映射关系。安装请参考[官网的安装教程](https://redis.io/topics/quickstart)。命令简要列举如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><span>yum -y install gcc</span><br><span>mkdir /data</span><br><span>wget http://download.redis.io/redis-stable.tar.gz</span><br><span>tar xvzf redis-stable.tar.gz -C /data</span><br><span><span>cd</span> /data/redis-stable</span><br><span>make install</span><br></pre></td></tr></tbody></table>

如果遇到 `zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory` 错误，请执行 `make distclean && make install`。

然后继续执行：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>mkdir /etc/redis</span><br><span>mkdir /var/redis</span><br><span>cp /data/redis-stable/utils/redis_init_script /etc/init.d/redis_6379</span><br><span>cp /data/redis-stable/redis.conf /etc/redis/6379.conf</span><br><span>mkdir /var/redis/6379</span><br></pre></td></tr></tbody></table>

修改配置文件：

/etc/redis/6379.conf

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br></pre></td><td><pre><span># ...</span><br><span># IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES</span><br><span># JUST COMMENT OUT THE FOLLOWING LINE.</span><br><span># ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~</span><br><span><span>- bind 127.0.0.1 -::1</span></span><br><span><span>+ # bind 127.0.0.1 -::1</span></span><br><span># ...</span><br><span># By default protected mode is enabled. You should disable it only if</span><br><span># you are sure you want clients from other hosts to connect to Redis</span><br><span># even if no authentication is configured, nor a specific set of interfaces</span><br><span># are explicitly listed using the "bind" directive.</span><br><span><span>- protected-mode yes</span></span><br><span><span>+ protected-mode no</span></span><br><span># ...</span><br><span># By default Redis does not run as a daemon. Use 'yes' if you need it.</span><br><span># Note that Redis will write a pid file in /var/run/redis.pid when daemonized.</span><br><span># When Redis is supervised by upstart or systemd, this parameter has no impact.</span><br><span><span>- daemonize no</span></span><br><span><span>+ daemonize yes</span></span><br><span># ...</span><br><span># Specify the log file name. Also the empty string can be used to force</span><br><span># Redis to log on the standard output. Note that if you use standard</span><br><span># output for logging but daemonize, logs will be sent to /dev/null</span><br><span><span>- logfile ""</span></span><br><span><span>+ logfile "/var/log/redis_6379.log"</span></span><br><span># ...</span><br><span># The working directory.</span><br><span>#</span><br><span># The DB will be written inside this directory, with the filename specified</span><br><span># above using the 'dbfilename' configuration directive.</span><br><span>#</span><br><span># The Append Only File will also be created inside this directory.</span><br><span>#</span><br><span># Note that you must specify a directory here, not a file name.</span><br><span><span>- dir ./</span></span><br><span><span>+ dir /var/redis/6379</span></span><br><span># ...</span><br><span># IMPORTANT NOTE: starting with Redis 6 "requirepass" is just a compatibility</span><br><span># layer on top of the new ACL system. The option effect will be just setting</span><br><span># the password for the default user. Clients will still authenticate using</span><br><span># AUTH &lt;password&gt; as usually, or more explicitly with AUTH default &lt;password&gt;</span><br><span># if they follow the new protocol: both will work.</span><br><span>#</span><br><span># The requirepass is not compatable with aclfile option and the ACL LOAD</span><br><span># command, these will cause requirepass to be ignored.</span><br><span>#</span><br><span><span>- # requirepass foobared</span></span><br><span>requirepass redisredisredis</span><br><span># ...</span><br></pre></td></tr></tbody></table>

最后执行：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>systemctl daemon-reload</span><br><span>systemctl start redis_6379</span><br><span>systemctl <span>enable</span> redis_6379</span><br></pre></td></tr></tbody></table>

Redis 只需要在 master 上安装。

### 配置防火墙

在 master 节点开启以下配置（IP 段需要自行修改）：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="6379" accept'</span></span><br><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="9333" accept'</span></span><br><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="19333" accept'</span></span><br><span>firewall-cmd --reload</span><br><span>firewall-cmd --list-all </span><br></pre></td></tr></tbody></table>

在 worker 节点开启以下配置（IP 段需要自行修改）：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br></pre></td><td><pre><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="8080" accept'</span></span><br><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="18080" accept'</span></span><br><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="8888" accept'</span></span><br><span>firewall-cmd --permanent --add-rich-rule=<span>'rule family="ipv4" source address="xxx.xxx.xxx.0/24" port protocol="tcp" port="18888" accept'</span></span><br><span>firewall-cmd --permanent --zone=public --add-port=80/tcp</span><br><span>firewall-cmd --reload</span><br><span>firewall-cmd --list-all </span><br></pre></td></tr></tbody></table>

如果嫌麻烦可以直接关闭防火墙：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>systemctl stop firewalld.service</span><br><span>systemctl <span>disable</span> firewalld.service</span><br></pre></td></tr></tbody></table>

首先需要在所有机器上下载 SeaweedFS 并且创建所需目录。命令如下：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>wget https://github.com/chrislusf/seaweedfs/releases/download/2.84/linux_amd64.tar.gz</span><br><span>mkdir /data/weed</span><br><span>tar zxvf linux_amd64.tar.gz -C /data/weed/</span><br><span>mkdir /data/weed/meta</span><br><span>mkdir /data/weed/data</span><br></pre></td></tr></tbody></table>

随后分别配置 master 与 worker。我们在 master 安装 `weed-master`，在 worker 安装 `weed-volume`、`weed-filer` 与 S3 网关。

### 在 master 节点安装 `weed-master`

首先直接新建以下文件配置服务，其中 IP 地址根据你的设置自行更改：

/usr/lib/systemd/system/weed-master.service

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><span>[Unit]</span><br><span>Description=SeaweedFS Master</span><br><span>After=network.target</span><br><span></span><br><span>[Service]</span><br><span>Type=simple</span><br><span>User=root</span><br><span>Group=root</span><br><span></span><br><span>ExecStart=/data/weed/weed -v=0 master -ip=master -port=9333 -defaultReplication=001 -mdir=/data/weed/meta</span><br><span>WorkingDirectory=/data/weed</span><br><span>SyslogIdentifier=seaweedfs-master</span><br><span></span><br><span>[Install]</span><br><span>WantedBy=multi-user.target</span><br></pre></td></tr></tbody></table>

随后运行以下命令开启服务：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>systemctl daemon-reload</span><br><span>systemctl start weed-master</span><br><span>systemctl <span>enable</span> weed-master</span><br></pre></td></tr></tbody></table>

### 在 worker 节点安装 `weed-volume`，`weed-filer` 与 S3 网关

直接新建以下文件配置服务，其中 IP 地址与 mserver 地址根据你的设置自行更改：

/usr/lib/systemd/system/weed-volume.service

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><span>[Unit]</span><br><span>Description=SeaweedFS Volume</span><br><span>After=network.target</span><br><span></span><br><span>[Service]</span><br><span>Type=simple</span><br><span>User=root</span><br><span>Group=root</span><br><span></span><br><span>ExecStart=/data/weed/weed -v=0 volume -mserver=master:9333 -ip=[worker] -port=8080 -dir=/data/weed/data -dataCenter=dc1 -rack=rack1</span><br><span>WorkingDirectory=/data/weed</span><br><span>SyslogIdentifier=seaweedfs-volume</span><br><span></span><br><span>[Install]</span><br><span>WantedBy=multi-user.target</span><br></pre></td></tr></tbody></table>

然后运行以下命令生成 filer 配置文件：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>/data/weed/weed scaffold -config filer -output=<span>"/data/weed/"</span></span><br></pre></td></tr></tbody></table>

编辑此文件，配置 `[redis2]` 部分：

/data/weed/filer.toml

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br></pre></td><td><pre><span># ...</span><br><span>[leveldb2]</span><br><span># local on disk, mostly for simple single-machine setup, fairly scalable</span><br><span># faster than previous leveldb, recommended.</span><br><span><span>- enabled = true</span></span><br><span><span>+ enabled = false</span></span><br><span>dir = "./filerldb2"                    # directory to store level db files</span><br><span># ...</span><br><span>[redis2]</span><br><span><span>- enabled = false </span></span><br><span><span>- address = "localhost:6379"</span></span><br><span><span>- password = ""</span></span><br><span><span>+ enabled = true </span></span><br><span><span>+ address = "master:6379"</span></span><br><span><span>+ password = "redisredisredis"</span></span><br><span>database = 0</span><br><span># This changes the data layout. Only add new directories. Removing/Updating will cause data loss.</span><br><span>superLargeDirectories = []</span><br><span># ...</span><br></pre></td></tr></tbody></table>

新建以下文件配置服务，其中 master 地址根据你的设置自行更改：

/usr/lib/systemd/system/weed-filer.service

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><span>[Unit]</span><br><span>Description=SeaweedFS Filer</span><br><span>After=network.target</span><br><span></span><br><span>[Service]</span><br><span>Type=simple</span><br><span>User=root</span><br><span>Group=root</span><br><span></span><br><span>ExecStart=/data/weed/weed -v=0 filer -master=master:9333 -port=8888 -defaultReplicaPlacement=001</span><br><span>WorkingDirectory=/data/weed</span><br><span>SyslogIdentifier=seaweedfs-filer</span><br><span></span><br><span>[Install]</span><br><span>WantedBy=multi-user.target</span><br></pre></td></tr></tbody></table>

随后运行以下命令开启服务：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>systemctl daemon-reload</span><br><span>systemctl start weed-volume</span><br><span>systemctl <span>enable</span> weed-volume</span><br><span>systemctl start weed-filer</span><br><span>systemctl <span>enable</span> weed-filer</span><br></pre></td></tr></tbody></table>

接下来配置 S3 网关。首先建立如下文件并写入内容：

/data/weed/config.json

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br></pre></td><td><pre><span>{</span><br><span>  <span>"identities"</span>: [</span><br><span>    {</span><br><span>      <span>"name"</span>: <span>"anonymous"</span>,</span><br><span>      <span>"actions"</span>: [</span><br><span>        <span>"Read"</span></span><br><span>      ]</span><br><span>    },</span><br><span>    {</span><br><span>      <span>"name"</span>: <span>"root"</span>,</span><br><span>      <span>"credentials"</span>: [</span><br><span>        {</span><br><span>          <span>"accessKey"</span>: <span>"testak"</span>,</span><br><span>          <span>"secretKey"</span>: <span>"testsk"</span></span><br><span>        }</span><br><span>      ],</span><br><span>      <span>"actions"</span>: [</span><br><span>        <span>"Admin"</span>,</span><br><span>        <span>"Read"</span>,</span><br><span>        <span>"List"</span>,</span><br><span>        <span>"Tagging"</span>,</span><br><span>        <span>"Write"</span></span><br><span>      ]</span><br><span>    }</span><br><span>  ]</span><br><span>}</span><br></pre></td></tr></tbody></table>

新建以下文件配置服务，其中 master 地址根据你的设置自行更改：

/usr/lib/systemd/system/weed-s3.service

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br></pre></td><td><pre><span>[Unit]</span><br><span>Description=SeaweedFS S3</span><br><span>After=network.target</span><br><span></span><br><span>[Service]</span><br><span>Type=simple</span><br><span>User=root</span><br><span>Group=root</span><br><span></span><br><span>ExecStart=/data/weed/weed -v=0 s3 -port=8333 -filer=localhost:8888 -config=/data/weed/config.json</span><br><span>WorkingDirectory=/data/weed</span><br><span>SyslogIdentifier=seaweedfs-s3</span><br><span></span><br><span>[Install]</span><br><span>WantedBy=multi-user.target</span><br></pre></td></tr></tbody></table>

随后运行以下命令开启服务：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>systemctl daemon-reload</span><br><span>systemctl start weed-s3</span><br><span>systemctl <span>enable</span> weed-s3</span><br></pre></td></tr></tbody></table>

## 使用方法

### UI

以下 UI 可以访问：

-   [http://master:9333](http://master:9333/)
-   [http://worker](http://worker/)\[1-3\]:8080
-   [http://worker](http://worker/)\[1-3\]:8888

相关截图显示如下：  
![](https://s2.loli.net/2022/01/06/OSXt89YEeCnKZoT.png)  
![](https://s2.loli.net/2022/01/06/OxleWtQHsCvImAd.png)  
![](https://s2.loli.net/2022/01/06/uTb4gqV7GRWw2Qp.png)

### 使用 `weed`

以下是使用 `weed` 命令上传与下载出现的结果：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br></pre></td><td><pre><span>$ /data/weed/weed upload /data/weed/weed</span><br><span>[{<span>"fileName"</span>:<span>"weed"</span>,<span>"url"</span>:<span>"worker1:8080/12,ce488bc6ce"</span>,<span>"fid"</span>:<span>"12,ce488bc6ce"</span>,<span>"size"</span>:82229387}]</span><br><span></span><br><span>$ <span>cd</span> ~; mkdir weed-test; <span>cd</span> weed-test</span><br><span>$ /data/weed/weed download 12,ce488bc6ce</span><br></pre></td></tr></tbody></table>

### 使用 `curl`

以下只展示命令和结果：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br></pre></td><td><pre><span>$ <span>cd</span> ~/weed-test; <span>echo</span> <span>"hello world"</span> &gt; hello.txt</span><br><span>$ curl http://master:9333/dir/assign</span><br><span>{<span>"fid"</span>:<span>"10,e4e7c9db81"</span>,<span>"url"</span>:<span>"worker1:8080"</span>,<span>"publicUrl"</span>:<span>"worker1:8080"</span>,<span>"count"</span>:1}</span><br><span></span><br><span>$ curl -F file=@hello.txt http://worker1:8080/10,e4e7c9db81 -v</span><br><span>{<span>"name"</span>:<span>"hello.txt"</span>,<span>"size"</span>:12,<span>"eTag"</span>:<span>"f0ff7292"</span>,<span>"mime"</span>:<span>"text/plain"</span>}</span><br><span>$ curl http://worker1:8080/10,e4e7c9db81</span><br><span>hello world</span><br><span></span><br><span>$ curl -X DELETE http://worker1:8080/10,e4e7c9db81</span><br><span>{<span>"size"</span>:43}</span><br><span></span><br><span>$ curl http://master:9333/cluster/status?pretty=y</span><br><span>{</span><br><span>  <span>"IsLeader"</span>: <span>true</span>,</span><br><span>  <span>"Leader"</span>: <span>"master:9333"</span>,</span><br><span>  <span>"MaxVolumeId"</span>: 12</span><br><span>}</span><br><span>$ curl http://master:9333/dir/lookup?volumeId=12</span><br><span>{<span>"volumeOrFileId"</span>:<span>"12"</span>,<span>"locations"</span>:[{<span>"url"</span>:<span>"worker1:8080"</span>,<span>"publicUrl"</span>:<span>"worker1:8080"</span>},{<span>"url"</span>:<span>"worker2:8080"</span>,<span>"publicUrl"</span>:<span>"worker2:8080"</span>}]}</span><br><span></span><br><span>$ curl -F file=@hello.txt http://worker1:8888/text/</span><br><span>{<span>"name"</span>:<span>"hello.txt"</span>,<span>"size"</span>:12}</span><br><span>$ curl http://worker1:8888/text/hello.txt</span><br><span>hello world</span><br><span></span><br><span>$ curl -X DELETE http://worker1:8888/text/hello.txt</span><br></pre></td></tr></tbody></table>

### 使用 `s3cmd`

首先安装 `S3cmd`：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>yum install -y epel-release</span><br><span>yum install -y s3cmd</span><br></pre></td></tr></tbody></table>

然后编辑 `$HOME/.s3cfg`，其中 Worker IP 可以是任意配置了 S3 网关的节点：

$HOME/.s3cfg

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br></pre></td><td><pre><span>host_base = [worker1]:8333</span><br><span>host_bucket = [worker1]:8333</span><br><span>bucket_location = us-east-1</span><br><span>use_https = False</span><br><span></span><br><span>access_key = testak</span><br><span>secret_key = testsk</span><br><span></span><br><span>signature_v2 = False</span><br></pre></td></tr></tbody></table>

试着新建桶并上传文件：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br></pre></td><td><pre><span>s3cmd mb s3://test</span><br><span>s3cmd put /data/weed/weed s3://test/</span><br><span>s3cmd ls s3://test</span><br></pre></td></tr></tbody></table>

也可以使用其他 S3 工具或客户端。

### 使用 HDFS Client

这部分参考了[官方 Wiki](https://github.com/chrislusf/seaweedfs/wiki/Hadoop-Compatible-File-System#test-seaweedfs-on-hadoop)。这里假设你已经有一套可用的 Hadoop 集群。

首先在[这个地址](https://repo1.maven.org/maven2/com/github/chrislusf/seaweedfs-hadoop3-client/)下载相应的 jar 包，随后将它放入 Hadoop classpath：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br></pre></td><td><pre><span>wget https://repo1.maven.org/maven2/com/github/chrislusf/seaweedfs-hadoop3-client/2.84/seaweedfs-hadoop3-client-2.84.jar</span><br><span>cp seaweedfs-hadoop3-client-2.84.jar <span>$HADOOP_HOME</span>/share/hadoop/common/lib/</span><br></pre></td></tr></tbody></table>

然后输入以下命令进行测试：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>hdfs dfs -Dfs.defaultFS=seaweedfs://worker[1-3]:8888 -Dfs.seaweedfs.impl=seaweed.hdfs.SeaweedFileSystem -ls /</span><br></pre></td></tr></tbody></table>

可以修改 `$HADOOP_HOME/etc/hadoop/core-site.xml`，让默认值指向 SeaweedFS：

$HADOOP\_HOME/etc/hadoop/core-site.xml

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br></pre></td><td><pre><span><span>&lt;<span>configuration</span>&gt;</span></span><br><span>  <span>&lt;<span>property</span>&gt;</span></span><br><span>    <span>&lt;<span>name</span>&gt;</span>fs.defaultFS<span>&lt;/<span>name</span>&gt;</span></span><br><span>    <span>&lt;<span>value</span>&gt;</span>seaweedfs://worker[1-3]:8888<span>&lt;/<span>value</span>&gt;</span></span><br><span>  <span>&lt;/<span>property</span>&gt;</span></span><br><span>  <span>&lt;<span>property</span>&gt;</span></span><br><span>    <span>&lt;<span>name</span>&gt;</span>fs.seaweedfs.impl<span>&lt;/<span>name</span>&gt;</span></span><br><span>    <span>&lt;<span>value</span>&gt;</span>seaweed.hdfs.SeaweedFileSystem<span>&lt;/<span>value</span>&gt;</span></span><br><span>  <span>&lt;/<span>property</span>&gt;</span></span><br><span>  <span>&lt;<span>property</span>&gt;</span></span><br><span>    <span>&lt;<span>name</span>&gt;</span>fs.AbstractFileSystem.seaweedfs.impl<span>&lt;/<span>name</span>&gt;</span></span><br><span>    <span>&lt;<span>value</span>&gt;</span>seaweed.hdfs.SeaweedAbstractFileSystem<span>&lt;/<span>value</span>&gt;</span></span><br><span>  <span>&lt;/<span>property</span>&gt;</span></span><br><span>  <span>&lt;<span>property</span>&gt;</span></span><br><span>    <span>&lt;<span>name</span>&gt;</span>fs.seaweed.buffer.size<span>&lt;/<span>name</span>&gt;</span></span><br><span>    <span>&lt;<span>value</span>&gt;</span>4194304<span>&lt;/<span>value</span>&gt;</span></span><br><span>  <span>&lt;/<span>property</span>&gt;</span></span><br><span>  <span>&lt;<span>property</span>&gt;</span></span><br><span>    <span>&lt;<span>name</span>&gt;</span>fs.seaweed.volume.server.access<span>&lt;/<span>name</span>&gt;</span></span><br><span>    </span><br><span>    <span>&lt;<span>value</span>&gt;</span>direct<span>&lt;/<span>value</span>&gt;</span></span><br><span>  <span>&lt;/<span>property</span>&gt;</span></span><br><span><span>&lt;/<span>configuration</span>&gt;</span></span><br></pre></td></tr></tbody></table>

重启 Hadoop 集群，即可正常使用。如果出现以下错误：

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>java.io.IOException: java.util.concurrent.ExecutionException: java.io.IOException: assign volume: rpc error: code = Unknown desc = no free volumes left for {"replication":{},"ttl":{"Count":0,"Unit":0}}</span><br></pre></td></tr></tbody></table>

可以尝试在 `weed-master` 服务的启动脚本增加参数：`-volumeSizeLimitMB=512`，减小 volume 容量，然后再 `weed-volume` 服务的启动脚本增加参数 `-max=0`，让系统自行决定此参数。修改完服务脚本后要做 daemon-reload 与 restart 操作，可以在 UI 查看变化是否生效。