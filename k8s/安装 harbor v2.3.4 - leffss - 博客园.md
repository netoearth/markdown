-   [安装 docker](https://www.cnblogs.com/leffss/p/15621165.html#%E5%AE%89%E8%A3%85-docker)
-   [安装 docker-compose](https://www.cnblogs.com/leffss/p/15621165.html#%E5%AE%89%E8%A3%85-docker-compose)
-   [下载 harbor 安装包](https://www.cnblogs.com/leffss/p/15621165.html#%E4%B8%8B%E8%BD%BD-harbor-%E5%AE%89%E8%A3%85%E5%8C%85)
-   [安装 harbor](https://www.cnblogs.com/leffss/p/15621165.html#%E5%AE%89%E8%A3%85-harbor)
    -   [http 方式](https://www.cnblogs.com/leffss/p/15621165.html#http-%E6%96%B9%E5%BC%8F)
        -   [修改配置](https://www.cnblogs.com/leffss/p/15621165.html#%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE)
        -   [安装启动](https://www.cnblogs.com/leffss/p/15621165.html#%E5%AE%89%E8%A3%85%E5%90%AF%E5%8A%A8)
        -   [访问 harbor](https://www.cnblogs.com/leffss/p/15621165.html#%E8%AE%BF%E9%97%AE-harbor)
        -   [配置 docker 主机](https://www.cnblogs.com/leffss/p/15621165.html#%E9%85%8D%E7%BD%AE-docker-%E4%B8%BB%E6%9C%BA)
    -   [https 方式](https://www.cnblogs.com/leffss/p/15621165.html#https-%E6%96%B9%E5%BC%8F)
        -   [生成证书颁发机构证书](https://www.cnblogs.com/leffss/p/15621165.html#%E7%94%9F%E6%88%90%E8%AF%81%E4%B9%A6%E9%A2%81%E5%8F%91%E6%9C%BA%E6%9E%84%E8%AF%81%E4%B9%A6)
        -   [生成服务器证书](https://www.cnblogs.com/leffss/p/15621165.html#%E7%94%9F%E6%88%90%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%AF%81%E4%B9%A6)
        -   [配置 harbor](https://www.cnblogs.com/leffss/p/15621165.html#%E9%85%8D%E7%BD%AE-harbor)
        -   [安装启动](https://www.cnblogs.com/leffss/p/15621165.html#%E5%AE%89%E8%A3%85%E5%90%AF%E5%8A%A8-1)
        -   [访问 harbor](https://www.cnblogs.com/leffss/p/15621165.html#%E8%AE%BF%E9%97%AE-harbor-1)
        -   [配置 docker 主机](https://www.cnblogs.com/leffss/p/15621165.html#%E9%85%8D%E7%BD%AE-docker-%E4%B8%BB%E6%9C%BA-1)
    -   [验证](https://www.cnblogs.com/leffss/p/15621165.html#%E9%AA%8C%E8%AF%81)
-   [设置 harbor 开启启动](https://www.cnblogs.com/leffss/p/15621165.html#%E8%AE%BE%E7%BD%AE-harbor-%E5%BC%80%E5%90%AF%E5%90%AF%E5%8A%A8)
-   [harbor 高可用](https://www.cnblogs.com/leffss/p/15621165.html#harbor-%E9%AB%98%E5%8F%AF%E7%94%A8)

## 安装 docker

```
yum -y install yum-utils
yum-config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl enable docker
systemctl start docker
systemctl status docker

$ docker --version
Docker version 20.10.11, build dea9396
```

## 安装 docker-compose

安装 pip，本来可以使用 yum install python-pip，但是 centos 7.9 默认源只有 python3-pip 版本的，所以这里使用源码安装

```
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip2.py
python get-pip2.py
pip install docker-compose

$ docker-compose --version
docker-compose version 1.26.2, build unknown
```

## 下载 harbor 安装包

下载地址：[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)

有两种方式 online 或者 offline 安装方式，这里下载 2.3.4 版本 offline 离线包

```
tar zxvf harbor-offline-installer-v2.3.4.tgz
```

## 安装 harbor

## http 方式

### 修改配置

```
$ cd harbor
$ ls
common.sh  harbor.v2.3.4.tar.gz  harbor.yml.tmpl  install.sh  LICENSE  prepare
$ cp harbor.yml.tmpl harbor.yml
# 修改配置文件
$ vi harbor.yml

# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: hub.leffss.com# 修改为本地郁闷或者本机监听IP

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config# 注释掉 https 的相关配置
#https:
  # https port for harbor, default is 443
#  port: 443
  # The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

# # Uncomment following will enable tls communication between all harbor components
# internal_tls:
#   # set enabled to true means internal tls is enabled
#   enabled: true
#   # put your cert and key files on dir
#   dir: /etc/harbor/tls/internal
...
...
...
```

-   harbor\_admin\_password 管理员初始密码
-   data\_volume 数据存放目录

### 安装启动

```
$ ./install.sh
...
...
...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-db     ... done
Creating harbor-portal ... done
Creating registry      ... done
Creating redis         ... done
Creating registryctl   ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----
```

**harbor的停止与启动**

```
$ cd harbor
$ docker-compose stop  # 停止
$ docker-compose start  # 启动(第一次需要使用 up -d)
$ docker-compose down # 停止并删除容器（慎用）
$ docker-compose up -d# 创建并启动
```

### 访问 harbor

[http://10.10.10.21/](http://10.10.10.21/)

或者域名(需要设置本地 hosts)

[http://hub.leffss.com](http://hub.leffss.com/)

默认账号密码：admin Harbor12345

### 配置 docker 主机

**修改docker主机配置文件，使docker支持harbor**

vi /etc/docker/daemon.json

```
{"insecure-registries":["10.10.10.21:80"]}
或者
{"insecure-registries":["hub.leffss.com:80"]}
```

重启 docker

```
systemctl restart docker
```

## https 方式

默认情况下，Harbor不附带证书。可以在没有安全性的情况下部署Harbor，以便您可以通过HTTP连接到它。但是，只有在没有外部网络连接的空白测试或开发环境中，才可以使用HTTP。在没有空隙的环境中使用HTTP会使您遭受中间人攻击。在生产环境中，请始终使用HTTPS。如果启用Content Trust with Notary来正确签名所有图像，则必须使用HTTPS。

要配置HTTPS，必须创建SSL证书。您可以使用由受信任的第三方CA签名的证书，也可以使用自签名证书

### 生成证书颁发机构证书

在生产环境中，您应该从CA获得证书。在测试或开发环境中，您可以生成自己的CA。要生成CA证书，请运行以下命令。

生成CA证书私钥

```
cd ~
mkdir certs
cd certs
openssl genrsa -out ca.key 4096
```

生成CA证书

```
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=hub.leffss.com" \
 -key ca.key \
 -out ca.crt
```

-   如果是 ip 访问， 将 `hub.leffss.com` 改成 ip 地址

### 生成服务器证书

证书通常包含一个`.crt`文件和一个`.key`文件

生成私钥

```
openssl genrsa -out hub.leffss.com.key 4096
```

生成证书签名请求（CSR）

```
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=hub.leffss.com" \
    -key hub.leffss.com.key \
    -out hub.leffss.com.csr
```

-   如果是 ip 访问， 将 `hub.leffss.com` 改成 ip 地址

生成一个x509 v3扩展文件

```
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=hub.leffss.com
DNS.2=hub.leffss.com
DNS.3=hub.leffss.com
EOF
```

-   如果是 ip 访问

```
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = IP:10.10.10.21
EOF
```

使用该v3.ext文件为您的Harbor主机生成证书

```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in hub.leffss.com.csr \
    -out hub.leffss.com.crt
```

-   如果是 ip 访问， 将 `hub.leffss.com` 改成 ip 地址

### 配置 harbor

```
mkdir -p /data/certs
cp hub.leffss.com.crt /data/certs
cp hub.leffss.com.key /data/certs
```

```
$ cd harbor
$ ls
common.sh  harbor.v2.3.4.tar.gz  harbor.yml.tmpl  install.sh  LICENSE  prepare
$ cp harbor.yml.tmpl harbor.yml
# 修改配置文件
$ vi harbor.yml
...
...
...
hostname: hub.leffss.com
https:
  port: 443
  certificate: /data/certs/hub.leffss.com.crt 
  private_key: /data/certs/hub.leffss.com.key
external_url: https://hub.leffss.com
...
...
...
```

### 安装启动

**运行 `prepare` 脚本以启用 HTTPS**

```
./prepare
```

**开始安装**

```
./install.sh
```

**harbor的停止与启动**

```
$ cd harbor
$ docker-compose stop  # 停止
$ docker-compose start  # 启动(第一次需要使用 up -d)
$ docker-compose down # 停止并删除容器（慎用）
$ docker-compose up -d# 创建并启动
```

### 访问 harbor

[https://10.10.10.21/](https://10.10.10.21/)

或者域名(需要设置本地 hosts)

[https://hub.leffss.com](https://hub.leffss.com/)

默认账号密码：admin Harbor12345

### 配置 docker 主机

首先转换`hub.leffss.com.crt`为`hub.leffss.com.cert`，供Docker使用

```
openssl x509 -inform PEM -in hub.leffss.com.crt -out hub.leffss.com.cert
```

所有需要访问 hub 的 docker 主机都需要配置

```
mkdir -p /etc/docker/certs.d/hub.leffss.com/
cp hub.leffss.com.cert /etc/docker/certs.d/hub.leffss.com/
cp hub.leffss.com.key /etc/docker/certs.d/hub.leffss.com/
cp ca.crt /etc/docker/certs.d/hub.leffss.com/
```

-   如果 hub 是其他端口，则文件夹为：/etc/docker/certs.d/hub.leffss.com:\[端口\]/

重启 docker 生效

```
systemctl restart docker
```

## 验证

登陆 harbor 新建一个私有项目

![image-20211129200123689](https://gitee.com/leffss/pictures/raw/master/images/202111292001231.png)

docker 主机测试上传镜像

```
$ docker images
REPOSITORY                      TAG       IMAGE ID       CREATED       SIZE
goharbor/harbor-exporter        v2.3.4    41f7fb260d0d   2 weeks ago   81.1MB
goharbor/chartmuseum-photon     v2.3.4    f460981da720   2 weeks ago   179MB
goharbor/redis-photon           v2.3.4    e4780c57b230   2 weeks ago   155MB
goharbor/trivy-adapter-photon   v2.3.4    af0652363af0   2 weeks ago   130MB
goharbor/notary-server-photon   v2.3.4    66c118fdbe3e   2 weeks ago   110MB
goharbor/notary-signer-photon   v2.3.4    27d49a4ae0d3   2 weeks ago   108MB
goharbor/harbor-registryctl     v2.3.4    0daeaba57fc6   2 weeks ago   133MB
goharbor/registry-photon        v2.3.4    8497f259228a   2 weeks ago   81.9MB
goharbor/nginx-photon           v2.3.4    2218fcda1ff0   2 weeks ago   45MB
goharbor/harbor-log             v2.3.4    4d507b2e8131   2 weeks ago   159MB
goharbor/harbor-jobservice      v2.3.4    5924b12f0b85   2 weeks ago   211MB
goharbor/harbor-core            v2.3.4    dc8b74f8c4f3   2 weeks ago   193MB
goharbor/harbor-portal          v2.3.4    770e6950323b   2 weeks ago   58.2MB
goharbor/harbor-db              v2.3.4    8e2ed50e4699   2 weeks ago   228MB
goharbor/prepare                v2.3.4    cce1a590410d   2 weeks ago   254MB

$ docker tag goharbor/nginx-photon:v2.3.4 hub.leffss.com/leffss/nginx-photon:v2.3.4
$ docker images
REPOSITORY                           TAG       IMAGE ID       CREATED       SIZE
goharbor/harbor-exporter             v2.3.4    41f7fb260d0d   2 weeks ago   81.1MB
goharbor/chartmuseum-photon          v2.3.4    f460981da720   2 weeks ago   179MB
goharbor/redis-photon                v2.3.4    e4780c57b230   2 weeks ago   155MB
goharbor/trivy-adapter-photon        v2.3.4    af0652363af0   2 weeks ago   130MB
goharbor/notary-server-photon        v2.3.4    66c118fdbe3e   2 weeks ago   110MB
goharbor/notary-signer-photon        v2.3.4    27d49a4ae0d3   2 weeks ago   108MB
goharbor/harbor-registryctl          v2.3.4    0daeaba57fc6   2 weeks ago   133MB
goharbor/registry-photon             v2.3.4    8497f259228a   2 weeks ago   81.9MB
goharbor/nginx-photon                v2.3.4    2218fcda1ff0   2 weeks ago   45MB
hub.leffss.com/leffss/nginx-photon   v2.3.4    2218fcda1ff0   2 weeks ago   45MB
goharbor/harbor-log                  v2.3.4    4d507b2e8131   2 weeks ago   159MB
goharbor/harbor-jobservice           v2.3.4    5924b12f0b85   2 weeks ago   211MB
goharbor/harbor-core                 v2.3.4    dc8b74f8c4f3   2 weeks ago   193MB
goharbor/harbor-portal               v2.3.4    770e6950323b   2 weeks ago   58.2MB
goharbor/harbor-db                   v2.3.4    8e2ed50e4699   2 weeks ago   228MB
goharbor/prepare                     v2.3.4    cce1a590410d   2 weeks ago   254MB

$ docker login hub.leffss.com
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

$ docker push hub.leffss.com/leffss/nginx-photon:v2.3.4
The push refers to repository [hub.leffss.com/leffss/nginx-photon]
e1768f3b0fc8: Pushed 
103405848fd2: Pushed 
v2.3.4: digest: sha256:fde18ca6ae5fd7fb0bf69aaab9a24acdd7d9a5b8725fa612be5a2aa3cab7d3ca size: 740

$ docker logout https://hub.leffss.com
Removing login credentials for hub.leffss.com
```

![image-20211129200457750](https://gitee.com/leffss/pictures/raw/master/images/202111292004651.png)

## 设置 harbor 开启启动

vi /lib/systemd/system/harbor.service

```
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor

[Service]
Type=simple
Restart=on-failure
RestartSec=5
# 需要注意 harbor 的安装位置
ExecStart=/usr/bin/docker-compose -f /root/harbor/docker-compose.yml up
ExecStop=/usr/bin/docker-compose -f /root/harbor/docker-compose.yml stop

[Install]
WantedBy=multi-user.target
```

-   必须使用 docker-compose up 命令启动

```
systemctl daemon-reload
systemctl enable harbor  # 开机自启
systemctl start harbor   # 启动
```

## harbor 高可用

参考：[https://www.cnblogs.com/Gmiaomiao/p/14265246.html](https://www.cnblogs.com/Gmiaomiao/p/14265246.html)

原理是使用 harbor 官方默认提供主从复制的方案