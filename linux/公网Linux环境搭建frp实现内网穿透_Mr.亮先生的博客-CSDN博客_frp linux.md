首先你要有一台服务器或者VPS，还要有个域名，内网主机一台。  
服务器或者VPS、域名的购买，本文不赘述。  
本文所用的服务端是Debian GNU/Linux 10系统，客户端是windows10系统。

### 服务端(Linux)搭建步骤

#### 1：下载服务端的frp:

使用wget命令下载。如果wget command not found，则先安装wget，安装命令如下：

```
apt -y install wget
```

下载frp到服务器，在 https://github.com/fatedier/frp/releases 这里可以查看最新版本和获取下载地址。下载命令:

```
wget https://github.com/fatedier/frp/releases/download/v0.37.0/frp_0.37.0_linux_amd64.tar.gz
```

#### 2.使用tar命令解压下载成功的压缩包文件:

```
 tar -zxvf frp_0.37.0_linux_amd64.tar.gz
```

#### 3.使用mv命令移动文件:

```
 mv frp_0.37.0_linux_amd64 /usr/local/frp
```

#### 4.使用cd命令进入文件夹:

```
cd /usr/local/frp/
```

#### 5.修改服务器配置文件(frps.ini):

```
vim frps.ini
```

按i，进行编辑，将内容修改下面的:

```
[common]
#绑定的端口
bind_port = 7000
authentication_method = token
//验证的token
token = liang
//控制台
dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
#http的访问端口
vhost_http_port = 8080
#https的访问端口(如果需要的话)
vhost_https_port = 4443


```

按Esc键，退出编辑，再按:wq保存退出。

#### .启动frp服务端：

方法一： 启动命令(这种方式不推荐，因为断开与服务器的SSH连接后，frp也就停止运行了

```
./frps -c frps.ini
```

方法二: 建议让frp在后台运行:

```
nohup ./frps -c frps.ini &  > frp.log
```

可通过195.**.**.83:7500访问控制台  
这样即使关掉了SSH，frp依然在后台运行中。  
到此，服务端的搭建已经完成。

#### 另，停止运行frp的方法:

杀掉frps进程即可。使用ps命令，查看进程：

```
ps -ef | grep frp
```

使用kill命令杀掉：

```
kill -9 进程id
```

#### 另,也可通过systemctl来管理启动关闭

```
sudo vim /lib/systemd/system/frps.service
也可 修改 
vim /usr/local/frp/systemd/frps.service
然后创建软连接
ln /usr/local/frp/systemd/frps.service /lib/systemd/system
//写入
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.ini

[Install]
WantedBy=multi-user.target
退出保存
//开启
sudo systemctl start frps
//加入自启动
sudo systemctl enable frps
//关闭
sudo systemctl stop frps
//检测是否运行
systemctl status frps
//或者检查端口是否被监听
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210702111515513.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021070211161360.png)

### 客户端搭建步骤

#### 1.下载客户端的frp：

在这里 https://github.com/fatedier/frp/releases 找到windows对应的版本（版本必须与服务器端版本对应，不然会连接不上），32位或者64位。

#### 2.解压下载成功的压缩包

#### 3.编辑frpc.ini文件，内容如下：

```
[common]
server_addr = 195.**.**.83
server_port = 7000
authentication_method = token
token = liang
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_pwd = admin
[web]
type = http
local_ip = 127.0.0.1
local_port = 8081
#域名必须要有，并解析到你的服务器地址
custom_domains = work.****.xyz
```

保存。

#### 4.启动frp客户端：

在目录下打开命令窗口，执行如下命令：

```
frpc.exe -c frpc.ini
```

到此，客户端的搭建已经完成。

#### 测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210701154917112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bhcmllc2U=,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210701154935768.png)  
访问成功则安装无问题

### 注意事项

可查看frpc\_full.ini和frps\_full.ini了解配置项