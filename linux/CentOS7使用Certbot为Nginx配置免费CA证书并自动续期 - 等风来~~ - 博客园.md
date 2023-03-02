**阅读目录**

-   [一、准备工作](https://www.cnblogs.com/cfzy/p/14750087.html#_label0)
-    [二、安装SSL证书](https://www.cnblogs.com/cfzy/p/14750087.html#_label1)
    -   [1、CentOS7系统安装Certbox工具（RPM包安装）](https://www.cnblogs.com/cfzy/p/14750087.html#_label1_0)
    -   [2、CentOS7系统安装certbox-auto脚本（certbot-auto安装）](https://www.cnblogs.com/cfzy/p/14750087.html#_label1_1)
-   [三、证书续期](https://www.cnblogs.com/cfzy/p/14750087.html#_label2)
-   [四、报错解决](https://www.cnblogs.com/cfzy/p/14750087.html#_label3)

___

Certbot 是一个免费的开源软件工具，用于在手动管理的网站上自动使用[Let's Encrypt](https://letsencrypt.org/)证书来启用 HTTPS。

官网地址：[**https://certbot.eff.org/**](https://certbot.eff.org/)，不同系统安装certbot 可参考官网的文档。

我没有按照官方文档来，因为系统内核版本低，执行不成功，可以升级内核版本，但是服务器上有生产程序在运行，所以放弃了。

## 一、准备工作

##  二、安装SSL证书

先将需要添加SSL证书的域名配置到nginx.conf

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p></td><td><div><p><code>server {</code></p><p><code>　　server_name www.aa.com;</code></p><p><code>　　listen 80;</code></p><p><code>　　location / {</code></p><p><code>　　　　root html;</code></p><p><code>　　　　index index.html index.htm;</code></p><p><code>　　}</code></p><p><code>}</code></p></div></td></tr></tbody></table>

### 1、CentOS7系统安装Certbox工具（RPM包安装）

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
如果系统之前安装过certbot或者certbot-auto，需要先删除后再安装，避免冲突yum update  　　　　　　#安装更新软件yum install -y epel-release        #yum安装epel-release
yum install -y certbot       　　　 #安装certbot
certbot --version        　　　　　　#查看版本，说明安装成功为解决可能出现的报错，需要安装一下插件　　pip install urllib3　　easy_install urllib3==1.21.1　　　　pip install --upgrade --force-reinstall 'requests==2.6.0'生成证书，执行完命令后需要填写邮件地址、A（同意）、Y　　certbot certonly --webroot -w /usr/local/nginx/html -d www.aa.com证书存放地址：　　/etc/letsencrypt/live/www.aa.com为多个域名添加证书：　　certbot certonly --webroot -w /usr/local/nginx/html -d 域名      
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

### 2、CentOS7系统安装certbox-auto脚本（certbot-auto安装）

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
下载certbot-auto脚本，自动获取证书　　wget https://dl.eff.org/certbot-auto 
　　mv certbot-auto /usr/local/bin/certbot-auto
　　chown root /usr/local/bin/certbot-auto
　　chmod 0755 /usr/local/bin/certbot-auto 自动检测域名，获取SSL证书：   
　　/usr/local/bin/certbot-auto certonly --nginx  　　#选择需要添加的域名即可
为多个域名添加证书：　　certbot-auto certonly --nginx --no-self-upgrade
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

方式1、2生成证书后都需要修改nginx.conf配置文件：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p></td><td><div><p><code>server {</code></p><p><code>　　listen 443 ssl;</code></p><p><code>　　server_name www.aa.com;</code></p><p><code>　　ssl_certificate /etc/letsencrypt/live/www.aa.com/fullchain.pem;</code></p><p><code>　　ssl_certificate_key /etc/letsencrypt/live/www.aa.com/privkey.pem;</code></p><p><code>　　location / {</code></p><p><code>　　　　root html;</code></p><p><code>　　　　index index.html index.htm;</code></p><p><code>　　}</code></p><p><code>}</code></p></div></td></tr></tbody></table>

```
注解：　　ssl_certificate file;     　　#当前虚拟主机使用PEM格式的证书文件； 
　　ssl_certificate_key file; 　　#当前虚拟主机上与其证书匹配的私钥文件； 　　修改完记得重载nginx配置文件：/usr/local/nginx/sbin/nginx -s reload
```

## 三、证书续期

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
certbot常用命令：
    certbot [子命令] [选项] [-d 域名] 
    (默认) run：获取并安装证书到当前网页服务器；
    certonly：获取或更新证书，但是不安装；
    renew ：更新已经获取但快过期的所有证书；
        
    -d 域名列表：指定证书对应的域名列表，域名之间使用逗号分隔；
    --apache：使用Apache插件进行身份认证和安装
    --standalone：运行一个独立的网页服务器用于身份认证
    --nginx：用Nginx插件进行身份认证和安装
    --webroot：把身份认证文件放置在服务器的网页根目录下；
    –manual： 使用交互式或脚本钩子的方式获取证书；
    -n：非交互式运行；
    --test-cert：从预交付服务器上获取测试证书
    –-dry-run：测试获取或更新证书，但是不存储到本地硬盘；
        
    --force-renew：强制更新证书
    –-disable-hook-validation：更新前检查错误，如有错误将不会执行更新操作
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

证书管理：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
查看证书信息：                                                                                                          
    certbox certificates　 #显示使用Certbot生成的所有证书的信息，包括过期时间   
    certbot-auto --no-self-upgrade --force-renew certificates        
        #--force-renew 强制更新  --no-self-upgrade 不更新letsencrypt
撤销证书：
    certbox revoke
    certbox revoke --cert-path /etc/letsencrypt/archive/www.mydomain.com/cert1.pem        #输入"y"确定撤销证书
删除证书：
    certbot delete     #删除证书。回车后会列出所有证书，选择一个想要删除的证书即可
手动续期证书：续期前需要关闭nginx服务
    certbot renew --force-renew            #强制更新证书
    certbot-auto renew --force-renew    #强制更新证书
自动续期证书：每月的1号1点执行（certbot证书有效期是三个月）
    crontab -e   #写到定时任务
    0 1 1 * * /usr/bin/certbot renew --force-renew --renew-hook "/usr/local/nginx/sbin/nginx -s reload"
    0 1 1 * * /usr/local/bin/certbot-auto renew --no-self-upgrade --force-renew --renew-hook "/usr/local/nginx/sbin/nginx -s reload"   
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 四、报错解决

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
报错1：File "/opt/rh/rh-python36/root/usr/lib64/python3.6/ssl.py", line 689, in do_handshake
    执行：pip install --upgrade --force-reinstall 'requests==2.6.0' urllib3
报错2："Could not find a usable 'nginx' binary. Ensure nginx exists, the binary is executable, and your PATH is set correctly
    需要将nginx放到环境变量中
    　　ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
    　　ln -s /usr/local/nginx/conf/ /etc/nginx
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
繁华尽去，小时候逃离的地方，却是长大后再也回不去的远方。
```