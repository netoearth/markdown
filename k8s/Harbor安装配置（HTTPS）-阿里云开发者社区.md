1、下载harbor

git clone https://github.com/vmware/harbor

2、安装docker、docker-compose

pip uninstall docker docker-py; pip install docker

pip install docker-compose

3、修改/data/harbor/make/harbor.cfg

hostname = registry.niudingfeng.com

ui\_url\_protocol = https

email\_server = smtp.xiaoniu66.com

email\_server\_port = 25

email\_username = ndf.operate@xiaoniu66.com

email\_password = xnkj94nb!

email\_from = ndf.operate <ndf.operate@xiaoniu66.com>

email\_ssl = false

4、创建https证书

cd /data/harbor/cert

\[root@twin-sz01-docker-004 cert\]# openssl req -x509 -days 3650 -nodes -newkey rsa:2048 -keyout /data/harbor/cert/server.key -out /data/harbor/cert/server.crt

Generating a 2048 bit RSA private key

...........................+++

.....................................................................................................................+++

writing new private key to '/data/harbor/cert/server.key'

\-----

You are about to be asked to enter information that will be incorporated

into your certificate request.

What you are about to enter is what is called a Distinguished Name or a DN.

There are quite a few fields but you can leave some blank

For some fields there will be a default value,

If you enter '.', the field will be left blank.

\-----

Country Name (2 letter code) \[XX\]:

State or Province Name (full name) \[\]:

Locality Name (eg, city) \[Default City\]:

Organization Name (eg, company) \[Default Company Ltd\]:

Organizational Unit Name (eg, section) \[\]:

Common Name (eg, your name or your server's hostname) \[\]:registry-backup.niudingfeng.com

Email Address \[\]:

5、生成配置文件

cd /data/harbor/make && ./prepare

6、复制docker-compose文件

cd /data/harbor/make && cp docker-compose.tpl docker-compose.yml

7、执行安装脚本

cd /data/harbor/make && ./install.sh

FQA：

1、登录时报错：Error response from daemon: Get https://registry.niudingfeng.com/v1/users/: x509: certificate signed by unknown authority

此种情况多发生在自签名的证书，报错含义是签发证书机构未经认证，无法识别。

chmod 644 /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem

cat /data/harbor/cert/server.crt >>/etc/pki/tls/certs/ca-bundle.crt

chmod 444 /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem

证书是docker的daemon需要用到的，重启docker服务：service docker restart

     本文转自aaron428 51CTO博客，原文链接：http://blog.51cto.com/aaronsa/1897891，如需转载请自行联系原作者