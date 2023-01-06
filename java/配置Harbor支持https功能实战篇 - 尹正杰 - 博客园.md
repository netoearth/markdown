　　　　　　　　　　　　******配置Harbor支持https功能实战篇******

　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　**作者：尹正杰**

**版权声明：原创作品，谢绝转载！否则将追究法律责任。**

**一.部署Harbor**

```
　　博主推荐阅读:
　　　　https://www.cnblogs.com/yinzhengjie/p/12233594.html
```

**二.创建证书文件**

**1>.创建CA证书**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn ~]# cd /usr/local/src/harbor/
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ll
total 572840
drwxr-xr-x 4 root root        37 Jan 28 05:10 common
-rw-r--r-- 1 root root       939 Apr  1  2019 docker-compose.chartmuseum.yml
-rw-r--r-- 1 root root       975 Apr  1  2019 docker-compose.clair.yml
-rw-r--r-- 1 root root      1434 Apr  1  2019 docker-compose.notary.yml
-rw-r--r-- 1 root root      5608 Apr  1  2019 docker-compose.yml
-rw-r--r-- 1 root root      8071 Jan 28 19:28 harbor.cfg
-rw-r--r-- 1 root root 585234819 Apr  1  2019 harbor.v1.7.5.tar.gz
-rwxr-xr-x 1 root root      5739 Apr  1  2019 install.sh
-rw-r--r-- 1 root root     11347 Apr  1  2019 LICENSE
-rw-r--r-- 1 root root   1263409 Apr  1  2019 open_source_license
-rwxr-xr-x 1 root root     36337 Apr  1  2019 prepare
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# mkdir certs
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# openssl genrsa -out certs/harbor-ca.key 2048
Generating RSA private key, 2048 bit long modulus
.........................................................+++
.........................................................................................+++
e is 65537 (0x10001)
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ll certs/
total 4
-rw-r--r-- 1 root root 1679 Jan 28 23:58 harbor-ca.key
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**2>.根据自建的ca证书文件创建认证证书**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# openssl req -x509 -new -nodes -key certs/harbor-ca.key -subj "/CN=docker103.yinzhengjie.org.cn" -days 7120 -out certs/harbor-ca.crt
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ll certs/
total 8
-rw-r--r-- 1 root root 1147 Jan 29 00:11 harbor-ca.crt
-rw-r--r-- 1 root root 1679 Jan 28 23:58 harbor-ca.key
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.查看证书信息**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# openssl x509 -in certs/harbor-ca.crt -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            83:d7:7a:2c:52:07:ae:05
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=docker103.yinzhengjie.org.cn
        Validity
            Not Before: Jan 28 16:11:27 2020 GMT
            Not After : Jul 27 16:11:27 2039 GMT
        Subject: CN=docker103.yinzhengjie.org.cn
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d2:57:e4:f7:97:28:84:7e:c7:d4:f6:90:ad:48:
                    93:d5:1f:30:07:8f:09:b1:a9:28:34:4a:f8:59:4c:
                    e1:d6:f0:fb:62:67:b2:24:d9:c1:8f:ea:38:27:f5:
                    40:87:f3:9f:30:b5:2d:cb:cf:2f:c0:c5:e1:98:2d:
                    e1:d6:3d:cd:40:75:d0:ad:e5:d7:1a:1f:ff:28:9f:
                    58:cf:21:c5:af:d1:53:20:9d:89:67:66:bf:ea:3c:
                    ef:34:cc:02:06:e0:20:29:e4:6a:c9:04:88:f8:c9:
                    b7:f9:e7:3d:68:3b:63:86:e5:82:2d:cd:8b:da:45:
                    b5:93:fe:6a:f7:a9:81:4e:1d:d8:b1:9d:f4:97:2f:
                    4b:97:4a:7a:03:70:e9:55:b6:07:fe:db:10:aa:43:
                    50:f4:04:a5:0b:db:83:27:87:1a:ce:f4:54:63:b9:
                    98:c0:34:06:62:a4:3b:15:14:69:ff:89:b1:9c:8c:
                    82:2e:e4:20:03:d6:bb:01:e2:05:f3:bd:d6:98:8e:
                    0a:83:76:d4:72:44:33:f3:d5:a3:b8:98:14:77:55:
                    91:f4:04:ab:ee:93:4a:59:94:50:4d:ec:34:76:9a:
                    64:58:73:3a:7d:c0:50:b3:6a:cf:84:9b:14:f1:f1:
                    0f:e0:e1:40:a8:89:ee:4c:7e:c8:97:f9:26:e9:95:
                    e6:65
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                1E:DC:63:87:F0:37:DC:67:58:65:46:30:69:49:EE:FD:85:74:45:5E
            X509v3 Authority Key Identifier: 
                keyid:1E:DC:63:87:F0:37:DC:67:58:65:46:30:69:49:EE:FD:85:74:45:5E

            X509v3 Basic Constraints: 
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         49:0b:a9:58:d5:99:74:2b:92:b2:72:52:ea:36:3c:a6:e9:a0:
         7d:fe:70:44:85:1b:0c:7b:55:37:e5:fb:b3:8f:1f:8d:5a:3c:
         89:bd:b6:86:30:61:a6:59:86:60:df:22:34:7b:b1:b6:83:53:
         a0:b7:86:cc:00:13:f6:22:29:3c:98:60:7d:61:8e:e9:41:fb:
         5c:ac:77:42:71:ac:3c:8b:de:de:9f:21:6f:60:fd:99:df:cb:
         a6:34:b4:bc:03:31:25:fe:db:6d:5c:dc:58:c2:7f:2e:5f:6b:
         df:2b:00:fc:cd:93:b5:c3:f4:0b:9e:c6:5e:d9:b3:bb:9c:49:
         84:90:95:fe:4d:59:1c:33:47:0b:33:5f:cf:17:31:f2:45:3c:
         e1:ba:52:f0:17:4d:f6:58:0e:ca:3c:84:a5:c4:4d:b3:c9:a9:
         92:19:a6:94:83:1b:dd:c0:08:62:82:b1:07:c2:62:2c:cb:cd:
         da:2a:b7:12:ed:a6:5f:4e:a1:aa:4f:e0:3b:91:7c:12:e6:f8:
         1a:6b:c7:4a:ee:48:ec:1b:35:c7:fc:93:c7:0d:4b:e9:91:a5:
         5a:4c:fb:40:a0:b7:c5:49:20:17:46:8a:8b:f1:d0:e4:b3:2d:
         07:d8:72:16:e2:10:1f:d1:40:3c:bb:54:bc:62:82:60:f5:a1:
         45:f6:ec:5a
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200128161545916-1667180796.png)

**三.**配置Harbor支持https功能****

**1>.**修改harbor的主机名****

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# egrep -v "^$|^#" harbor.cfg  | grep hostname
hostname = docker103.yinzhengjie.org.cn
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

**2>.修改url的请求协议为https**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# egrep -v "^$|^#" harbor.cfg  | grep ui_url_protocol
ui_url_protocol = http
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# sed -r -i 's#(ui_url_protocol = )http#\1https#' harbor.cfg 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# egrep -v "^$|^#" harbor.cfg  | grep ui_url_protocol
ui_url_protocol = https
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**3>.指定证书路径**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# grep "ssl_cert = " harbor.cfg 
ssl_cert = /data/cert/server.crt
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# sed -r -i 's#(ssl_cert = )/data/cert/server.crt#\1/usr/local/src/harbor/certs/harbor-ca.crt#' harbor.cfg 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# grep "ssl_cert = " harbor.cfg 
ssl_cert = /usr/local/src/harbor/certs/harbor-ca.crt
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**4>.指定CA证书路径**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# grep "ssl_cert_key" harbor.cfg 
ssl_cert_key = /data/cert/server.key
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# sed -r -i 's#(ssl_cert_key = )/data/cert/server.key#\1/usr/local/src/harbor/certs/harbor-ca.key#' harbor.cfg 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# grep "ssl_cert_key" harbor.cfg 
ssl_cert_key = /usr/local/src/harbor/certs/harbor-ca.key
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**5>.自定义Harbor的密码**

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# grep "harbor_admin_password" harbor.cfg 
harbor_admin_password = yinzhengjie
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

**6>.重新执行安装命令(\[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor\]# ./install.sh)**

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)

```
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      20480                                                                                               127.0.0.1:1514                                                                                                                    *:*                  
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      20480                                                                                                      :::80                                                                                                                     :::*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
LISTEN     0      20480                                                                                                      :::443                                                                                                                    :::*                  
LISTEN     0      20480                                                                                                      :::4443                                                                                                                   :::*                  
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ll
total 572840
drwxr-xr-x 2 root root        48 Jan 29 00:11 certs
drwxr-xr-x 4 root root        37 Jan 28 05:10 common
-rw-r--r-- 1 root root       939 Apr  1  2019 docker-compose.chartmuseum.yml
-rw-r--r-- 1 root root       975 Apr  1  2019 docker-compose.clair.yml
-rw-r--r-- 1 root root      1434 Apr  1  2019 docker-compose.notary.yml
-rw-r--r-- 1 root root      5608 Apr  1  2019 docker-compose.yml
-rw-r--r-- 1 root root      8112 Jan 29 00:26 harbor.cfg
-rw-r--r-- 1 root root 585234819 Apr  1  2019 harbor.v1.7.5.tar.gz
-rwxr-xr-x 1 root root      5739 Apr  1  2019 install.sh
-rw-r--r-- 1 root root     11347 Apr  1  2019 LICENSE
-rw-r--r-- 1 root root   1263409 Apr  1  2019 open_source_license
-rwxr-xr-x 1 root root     36337 Apr  1  2019 prepare
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# docker-compose down 
Stopping nginx              ... done
Stopping harbor-portal      ... done
Stopping harbor-jobservice  ... done
Stopping harbor-core        ... done
Stopping harbor-db          ... done
Stopping registry           ... done
Stopping harbor-adminserver ... done
Stopping registryctl        ... done
Stopping redis              ... done
Stopping harbor-log         ... done
Removing nginx              ... done
Removing harbor-portal      ... done
Removing harbor-jobservice  ... done
Removing harbor-core        ... done
Removing harbor-db          ... done
Removing registry           ... done
Removing harbor-adminserver ... done
Removing registryctl        ... done
Removing redis              ... done
Removing harbor-log         ... done
Removing network harbor_harbor
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# ss -ntl
State      Recv-Q Send-Q                                                                                          Local Address:Port                                                                                                         Peer Address:Port              
LISTEN     0      128                                                                                                         *:22                                                                                                                      *:*                  
LISTEN     0      128                                                                                                        :::22                                                                                                                     :::*                  
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor]# 
```

\[root@docker103.yinzhengjie.org.cn /usr/local/src/harbor\]# docker-compose down　　　　　　　　#先停掉Harbor服务

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200128163327834-1985538569.png)

**7>.登录页面测试**

**![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200128163559447-1174894210.png)**

**8>.登录Harbor成功，数据依旧还在,如下图所示。**

![](https://img2018.cnblogs.com/i-beta/795254/202001/795254-20200128163928266-645003583.png)

**四.客户端测试**

**1>.将harbor服务器设置https协议后，发现无法正常登录啦,报错如下图所示**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://tuv7rqqq.mirror.aliyuncs.com"]
}
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker login docker103.yinzhengjie.org.cn
Authenticating with existing credentials...
Login did not succeed, error: Error response from daemon: Get https://docker103.yinzhengjie.org.cn/v2/: x509: certificate signed by unknown authority
Username (admin): jason
Password: 
Error response from daemon: Get https://docker103.yinzhengjie.org.cn/v2/: x509: certificate signed by unknown authority
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202110254469-610489186.png)**

**2>.拷贝证书到本地**

![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202105916327-78326257.png)

**3>.将上一步下载的证书上传到服务器端并运行命令，更新信任ca(需要重启docker，使docker对新的ca可见)**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[root@docker101.yinzhengjie.org.cn ~]# ll /etc/pki/ca-trust/source/anchors/
total 0
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll
total 2040
-rw-r--r-- 1 root root 2083917 Jan 24 06:12 haproxy-1.8.20.tar.gz
-rw-r--r-- 1 root root     805 Feb  2 10:58 harbor.pub.cer
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# cp harbor.pub.cer /etc/pki/ca-trust/source/anchors/
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# ll /etc/pki/ca-trust/source/anchors/
total 4
-rw-r--r-- 1 root root 805 Feb  2 19:00 harbor.pub.cer
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# update-ca-trust extract
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# systemctl restart docker
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# docker login docker103.yinzhengjie.org.cn
Authenticating with existing credentials...
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker101.yinzhengjie.org.cn ~]# 
[root@docker101.yinzhengjie.org.cn ~]# 
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**![](https://img2018.cnblogs.com/i-beta/795254/202002/795254-20200202110728689-1360151500.png)**

当你的才华还撑不起你的野心的时候，你就应该静下心来学习。当你的能力还驾驭不了你的目标的时候，你就应该沉下心来历练。问问自己，想要怎样的人生。 欢迎加入基础架构自动化运维：598432640，大数据SRE进阶之路：959042252，DevOps进阶之路：526991186