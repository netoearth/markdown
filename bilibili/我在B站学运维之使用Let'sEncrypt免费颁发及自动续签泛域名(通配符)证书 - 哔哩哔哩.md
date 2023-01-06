GIF

![](https://i0.hdslb.com/bfs/article/4a10568f28aa362fc3ddc7f871b07daf847767da.gif)

帅哥（靓仔）、美女，点个关注后续不迷路！

**本章目录**

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

-   0x00 前言简述
    
-   0x01 快速安装配置
    

-   1.acme.sh
    

-   0x02 证书自动签发实践
    

-   1.acme.sh + Cloudflare 实现自动签发泛域名证书。
    

-   0x03 使用实例
    

-   1.简单示例
    
-   2.扩展补充
    

-   0x04 入坑出坑
    

-   1.Cloudflare 的API 不技持 .cf, .ga, .gq, .ml, or .tk 的域名申请证书
    

![](https://i0.hdslb.com/bfs/article/4aa545dccf7de8d4a93c2b2b8e3265ac0a26d216.png@progressive.webp)

描述: Let's Encrypt 是免费、开放和自动化的证书颁发机构由Linux基金会(`Linux Foundation`)进行日常管理维护，它为1.8亿个网站提供TLS证书的非盈利性证书颁发机构, 通过它我们可以免费申请网站证书，并您的网站上启用 HTTPS (SSL/TLS) 提供支持。

![](https://i0.hdslb.com/bfs/article/403be3c4d8cfe529497d6ac4c6d5ee885aa688c1.png@942w_632h_progressive.webp)

-   域名所有者 : Let’s Encrypt 是一个证书颁发机构（CA）, 要从 Let’s Encrypt 获取您网站域名的证书，您必须证明您对域名的实际控制权。
    
-   ACME 协议软件 : 在Let’s Encrypt 使用 ACME 协议来验证您对给定域名的控制权并向您颁发证书, 要从Let’s Encrypt 获得证书，您需要选择一个要使用的 ACME 客户端软件。 例如官方推荐的客户端 Certbot 、或者使用得最多的 acme.sh
    

**参考来源**  
Let's Encrypt 网站: https://letsencrypt.org/  
ACME 协议客户端: https://letsencrypt.org/zh-cn/docs/client-options/  
acme.sh WIKI: https://wiki.acme.sh  
certbot Github: https://github.com/certbot/certbot

描述: 此处我们采用 acmesh-official 提供的 acme.sh 项目来快速搭建证书自动颁发、续签证书，其使用简单、强大且非常易于使用，它纯粹用 Shell（Unix shell）语言编写的 ACME 协议客户端，安装方式主要有`二进制文件`或者是`acme.sh💕码头工人`(https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker)。

温馨提示: 签发的证书有效期为60天，我们可以设置cron 作业以检查和更新证书，通常证书将每60天自动更新一次。

**快速安装:**  
安装参考地址: https://github.com/acmesh-official/acme.sh/wiki/How-to-install

```
# 从 git 安装 acme
git clone https://github.com/acmesh-official/acme.sh.git && cd ./acme.sh
./acme.sh --install -m master@weiyigeek.top
  # [Thu 10 Mar 2022 02:23:16 PM CST] Installing to /root/.acme.sh
  # [Thu 10 Mar 2022 02:23:16 PM CST] Installed to /root/.acme.sh/acme.sh
  # [Thu 10 Mar 2022 02:23:16 PM CST] Installing alias to '/root/.bashrc'
  # [Thu 10 Mar 2022 02:23:16 PM CST] OK, Close and reopen your terminal to start using acme.sh
  # [Thu 10 Mar 2022 02:23:16 PM CST] Installing cron job
  # [Thu 10 Mar 2022 02:23:16 PM CST] Good, bash is found, so change the shebang to use bash as preferred.
  # [Thu 10 Mar 2022 02:23:17 PM CST] OK

# email环境变量 & 别名设置
export email="mater@weiyigeek.top"
alias acme.sh=~/.acme.sh/acme.sh

# 查看acme.sh版本
acme.sh --version
  # https://github.com/acmesh-official/acme.sh
  # v3.0.2

# 使用 letsencrypt 提供的默认CA,当然你也可以自己提供（PS: 白嫖免费、谁不爱？）
acme.sh --set-default-ca --server letsencrypt
  # [Thu 10 Mar 2022 02:34:09 PM CST] Changed default CA to: https://acme-v02.api.letsencrypt.org/directory
```

至此，您现在可以发布证书了。

**Q: 什么是通配符证书?**  
A: 在没有出现通配符证书之前，Let’s Encrypt 支持两种单域名证书、SAN证书。

> 1）单域名证书：证书仅仅包含一个主机。  
> 2）SAN 证书：域名通配符证书类似 DNS 解析的泛域名概念，通配符证书就是证书中可以包含一个通配符(\*.exmaple.com)。主域名签发的通配符证书可以在所有子域名中使用，比如 www.example.com、bbs.example.com。

**申请通配符证书流程**  
步骤 01.如果您的 DNS 提供商支持 API 访问，我们可以使用该 API 自动颁发证书，目前 acme.sh 支持大多数 dns 提供者（https://github.com/acmesh-official/acme.sh/wiki/dnsapi），如果您的 dns 提供商不支持任何 api 访问，您只能手动添加 txt 记录。

步骤 02.此处我已经将 weiyigeek.top域名DNS解析商设置为 Cloudflare (免费)，其DNS提供上支持API访问, 在使用 acme.sh 实现自动签发证书前，我们需要再Cloudflare网站上获取用于访问 Cloudflare API 的密钥，首先`点击我的个人资料 -> API 令牌`。

步骤 03.设置 acme.sh 用于访问 Cloudflare 的 dnsapi 的相关环境变量。

```
# 注意, 如下环境变量,你需要根据你自己注册的 Cloudflare 邮箱地址（CF_Email）以及获取的 Cloudflare API 密钥（CF_Key）。
export CF_Email="dns@weiyigeek.top"
export CF_Key="be587222s8qa58asd87asd20b55a30cd653"
```

步骤 04.执行如下命令，可以实现自动颁发通配符&ECDSA格式证书，Let's Encrypt 支持颁发 EC 格式证书，

```
acme.sh --issue --dns dns_cf -d weiyigeek.top -d *.weiyigeek.top --keylength ec-256
# --issue是 acme.sh 脚本用来颁发证书的指令；
# -d是--domain的简称，其后面须填写已备案的域名；
# --keylength 证书格式;
  # [Thu 10 Mar 2022 02:37:27 PM CST] Using CA: https://acme-v02.api.letsencrypt.org/directory
  # [Thu 10 Mar 2022 02:37:27 PM CST] Create account key ok.
  # [Thu 10 Mar 2022 02:37:27 PM CST] Registering account: https://acme-v02.api.letsencrypt.org/directory
  # [Thu 10 Mar 2022 02:37:29 PM CST] Registered
  # [Thu 10 Mar 2022 02:37:29 PM CST] ACCOUNT_THUMBPRINT='3BKx-SF-zF1gbbkXpT9F4yfItz7V7JYKsPAkv8mxR2o'
  # [Thu 10 Mar 2022 02:37:29 PM CST] Creating domain key
  # [Thu 10 Mar 2022 02:37:29 PM CST] The domain key is here: /root/.acme.sh/weiyigeek.top_ecc/weiyigeek.top.key
  # [Thu 10 Mar 2022 02:37:29 PM CST] Multi domain='DNS:weiyigeek.top,DNS:*.weiyigeek.top'
  # [Thu 10 Mar 2022 02:37:29 PM CST] Getting domain auth token for each domain
  # [Thu 10 Mar 2022 02:37:33 PM CST] Getting webroot for domain='weiyigeek.top'
  # [Thu 10 Mar 2022 02:37:33 PM CST] Getting webroot for domain='*.weiyigeek.top'
  # [Thu 10 Mar 2022 02:37:33 PM CST] Adding txt value: iobyZpnA9bfxgM_tW_P8hjDye677KzjyQHClMka4Btw for domain:  _acme-challenge.weiyigeek.top
  # [Thu 10 Mar 2022 02:37:36 PM CST] Adding record
  # [Thu 10 Mar 2022 02:37:37 PM CST] Added, OK
  # [Thu 10 Mar 2022 02:37:37 PM CST] The txt record is added: Success.
  # [Thu 10 Mar 2022 02:37:37 PM CST] Adding txt value: NrksC2yEFDGFFMQ8_p76tAhdjWWWLkj1yKpTjUq4TcM for domain:  _acme-challenge.weiyigeek.top
  # [Thu 10 Mar 2022 02:37:40 PM CST] Adding record
  # [Thu 10 Mar 2022 02:37:41 PM CST] Added, OK
  # [Thu 10 Mar 2022 02:37:41 PM CST] The txt record is added: Success.
  # [Thu 10 Mar 2022 02:37:41 PM CST] Let's check each DNS record now. Sleep 20 seconds first.
  # [Thu 10 Mar 2022 02:38:02 PM CST] You can use '--dnssleep' to disable public dns checks.
  # [Thu 10 Mar 2022 02:38:02 PM CST] See: https://github.com/acmesh-official/acme.sh/wiki/dnscheck
  # [Thu 10 Mar 2022 02:38:02 PM CST] Checking weiyigeek.top for _acme-challenge.weiyigeek.top
  # [Thu 10 Mar 2022 02:38:03 PM CST] Domain weiyigeek.top '_acme-challenge.weiyigeek.top' success.
  # [Thu 10 Mar 2022 02:38:04 PM CST] Checking weiyigeek.top for _acme-challenge.weiyigeek.top
  # [Thu 10 Mar 2022 02:38:06 PM CST] Domain weiyigeek.top '_acme-challenge.weiyigeek.top' success.
  # [Thu 10 Mar 2022 02:38:06 PM CST] All success, let's return
  # [Thu 10 Mar 2022 02:38:06 PM CST] Verifying: weiyigeek.top
  # [Thu 10 Mar 2022 02:38:07 PM CST] Pending, The CA is processing your order, please just wait. (1/30)
  # [Thu 10 Mar 2022 02:38:11 PM CST] Success
  # [Thu 10 Mar 2022 02:38:11 PM CST] Verifying: *.weiyigeek.top
  # [Thu 10 Mar 2022 02:38:12 PM CST] Pending, The CA is processing your order, please just wait. (1/30)
  # [Thu 10 Mar 2022 02:38:16 PM CST] Success
  # [Thu 10 Mar 2022 02:38:16 PM CST] Removing DNS records.
  # [Thu 10 Mar 2022 02:38:16 PM CST] Removing txt: iobyZpnA9bfxgM_tW_P8hjDye677KzjyQHClMka4Btw for domain: _acme-challenge.weiyigeek.top
  # [Thu 10 Mar 2022 02:38:22 PM CST] Removed: Success
  # [Thu 10 Mar 2022 02:38:22 PM CST] Removing txt: NrksC2yEFDGFFMQ8_p76tAhdjWWWLkj1yKpTjUq4TcM for domain: _acme-challenge.weiyigeek.top
  # [Thu 10 Mar 2022 02:38:30 PM CST] Removed: Success
  # [Thu 10 Mar 2022 02:38:30 PM CST] Verify finished, start to sign.
  # [Thu 10 Mar 2022 02:38:30 PM CST] Lets finalize the order.
  # [Thu 10 Mar 2022 02:38:30 PM CST] Le_OrderFinalize='https://acme-v02.api.letsencrypt.org/acme/finalize/444241360/70248671870'
  # [Thu 10 Mar 2022 02:38:31 PM CST] Downloading cert.
  # [Thu 10 Mar 2022 02:38:31 PM CST] Le_LinkCert='https://acme-v02.api.letsencrypt.org/acme/cert/03740b271962634e99cbcefe7b74351b952c'
  # [Thu 10 Mar 2022 02:38:32 PM CST] Cert success.
  # -----BEGIN CERTIFICATE-----
  # MIIEZTCCA02gAwIBAgISA3QLJxliY06Zy87+e3Q1G5UsMA0GCSqGSIb3DQEBCwUA
  # .................................
  # jvhX8eE7xRHgXppOwoEPdbZ29uSJWUD9yQ==
  # -----END CERTIFICATE-----
  # [Thu 10 Mar 2022 02:38:32 PM CST] Your cert is in: /root/.acme.sh/weiyigeek.top_ecc/weiyigeek.top.cer
  # [Thu 10 Mar 2022 02:38:32 PM CST] Your cert key is in: /root/.acme.sh/weiyigeek.top_ecc/weiyigeek.top.key
  # [Thu 10 Mar 2022 02:38:32 PM CST] The intermediate CA cert is in: /root/.acme.sh/weiyigeek.top_ecc/ca.cer
  # [Thu 10 Mar 2022 02:38:32 PM CST] And the full chain certs is there: /root/.acme.sh/weiyigeek.top_ecc/fullchain.cer
```

步骤 05.利用openssl查看颁发的CA及其证书。

```
$  acme.sh list
Main_Domain    KeyLength  SAN_Domains      CA               Created                          Renew
weiyigeek.top  "ec-256"   *.weiyigeek.top  LetsEncrypt.org  Thu 10 Mar 2022 06:38:32 AM UTC  Mon 09 May 2022 06:38:32 AM UTC

$ openssl x509 -in ca.cer -noout -text
  # X509v3 extensions:
  #   X509v3 Key Usage: critical
  #       Digital Signature, Certificate Sign, CRL Sign
  #   X509v3 Extended Key Usage:
  #       TLS Web Client Authentication, TLS Web Server Authentication
  #   X509v3 Basic Constraints: critical
  #       CA:TRUE, pathlen:0
  #   X509v3 Subject Key Identifier:
  #       14:2E:B3:17:B7:58:56:CB:AE:50:09:40:E6:1F:AF:9D:8B:14:C2:C6
  #   X509v3 Authority Key Identifier:
  #       keyid:79:B4:59:E6:7B:B6:E5:E4:01:73:80:08:88:C8:1A:58:F6:E9:9B:6E

  #   Authority Information Access:
  #       CA Issuers - URI:http://x1.i.lencr.org/

  #   X509v3 CRL Distribution Points:

  #       Full Name:
  #         URI:http://x1.c.lencr.org/

  #   X509v3 Certificate Policies:
  #       Policy: 2.23.140.1.2.1
  #       Policy: 1.3.6.1.4.1.44947.1.1.1

$ openssl x509 -in ca.cer -noout -text
$ openssl x509 -in weiyigeek.top.cer -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            03:74:0b:27:19:62:63:4e:99:cb:ce:fe:7b:74:35:1b:95:2c
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, O = Let's Encrypt, CN = R3
        Validity
            Not Before: Mar 10 05:38:30 2022 GMT
            Not After : Jun  8 05:38:29 2022 GMT
```

步骤 06.使用签发的证书，ingress为指定主机名称设置tls, 然后通过浏览器访问 https://demo.weiyigeek.top 站点，验证tls配置是否正常。

```
# 1.创建一个tls类型的secret ，并使用--cert指定证书文件，--key指定证书密钥
kubectl create secret tls wildcard-weiyigeek-top --cert=weiyigeek.top.cer --key=weiyigeek.top.key -n devtest

# 2.分别为两个虚拟主机分别配置tls，需要再sepc字段下进行如下配置(注意缩进)
$ kubectl edit ingress -n devtest demo-myweb-blog
spec:
  tls:
  - hosts:
      - demo.weiyigeek.top
    secretName: wildcard-weiyigeek-top
```

![WeiyiGeek.验证TLS配置服务](https://i0.hdslb.com/bfs/article/e58071aaf9b1d2949be9db65bf34f2131119dfeb.png@942w_830h_progressive.webp)

步骤 07.于此同时我们还可，修改 Nginx 配置文件启用 ssl，记得修改完成后需要重启下 Nginx。

```
# 在 Nginx 主机上一条命令解决。
acme.sh  --installcert -d weiyigeek.top \
  --key-file /etc/nginx/ssl/weiyigeek.top.key \
  --fullchain-file /etc/nginx/ssl/fullchain.cer \
  --reloadcmd "service nginx force-reload"

# nginx.conf 的配置参考:
server {
  listen 443 ssl;
  server_name weiyigeek.top;

  ssl on;
  ssl_certificate      /etc/nginx/ssl/fullchain.cer;
  ssl_certificate_key  /etc/nginx/ssl/weiyigeek.top.key;

  root /home/wwwroot/weiyigeek.top;
  index index.html;

  location / {
    try_files $uri $uri/ @router;
    index index.html;
  }

  location @router {
    rewrite ^.*$ /index.html last;
  }
}

server {
    listen 80;
    server_name weiyigeek.top;
    return 301 https://$server_name$request_uri;
}
```

温馨提示: Nginx 的配置 ssl\_certificate 和 ssl\_trusted\_certificate 使用 fullchain.cer ，而非`<domain>.cer`，否则 SSL Labs 的测试会报 Chain issues Incomplete 错误

步骤 08.创建每日 cron 作业以检查和更新证书，添加force参数60天后强制更新。

```
20 0 */60 * * /root/.acme.sh/acme.sh --cron --force --home "/root/.acme.sh" > /dev/null
```

```
# 0.注册用户
acme.sh --register-account -m tls@weiyigeek.top

# 1.颁发单域&多域证书(注意设置DNS指向)为其颁发证书的主机，您必须将所有域指向并绑定到同一个 webroot dir (/home/wwwroot/example.com).
acme.sh --issue -d example.com -w /home/wwwroot/example.com
acme.sh --issue -d example.com -d www.example.com -d cp.example.com -w /home/wwwroot/example.com

# 2.证书生成后利用acme.sh提供的--install-cert选项我们可以直接将证书安装到 Apache/Nginx/Proxmox 服务。
# - Apache example:
# --installcert命令，指定目标位置，然后证书文件会被 copy 到相应的位置。
acme.sh --install-cert -d example.com \
--cert-file      /path/to/certfile/in/apache/cert.pem  \
--key-file       /path/to/keyfile/in/apache/key.pem  \
--fullchain-file /path/to/fullchain/certfile/apache/fullchain.pem \
--reloadcmd     "service apache2 force-reload"

# Nginx example:
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
# Proxmox example:
/root/.acme.sh/acme.sh --installcert -d example.com \
--certpath /etc/pve/local/pveproxy-ssl.pem \
--keypath /etc/pve/local/pveproxy-ssl.key  \
--capath  /etc/pve/local/pveproxy-ssl.pem  \
--reloadcmd  "systemctl restart pveproxy"

# 3.使用独立服务器颁发证书，端口80(TCP)必须可以自由监听，否则系统会提示您释放它并重试。
acme.sh --issue --standalone -d example.com -d www.example.com -d cp.example.com

# 4.使用独立的ssl服务器颁发证书，端口443(TCP)必须可以自由监听，否则系统会提示您释放它并重试。
acme.sh --issue --alpn -d example.com -d www.example.com -d cp.example.com

# 5.dns手动模式,签发证书时，如果您的 dns 提供商不支持任何 api 访问，使用--dns会填写需要添加的txt记录，此时您可以手动添加 txt 记录
acme.sh --issue --dns -d example.com -d www.example.com -d cp.example.com

# 6.颁发ECC证书 （ECDSA）,只需keylength使用前缀设置参数ec-。
# - 单域ECC证书
acme.sh --issue -w /home/wwwroot/example.com -d example.com --keylength ec-256
# - SAN多域ECC证书
acme.sh --issue -w /home/wwwroot/example.com -d example.com -d www.example.com --keylength ec-256

# 7.颁发通配符证书非常的简单,你只需要给一个通配符域作为-d参数。
acme.sh --issue -d example.com -d '*.example.com' --dns dns_cf --keylength ec-256

# 8.检查证书到期时间
acme.sh --cron --home ~/.acme.sh/
  # [Thu 10 Mar 2022 04:15:54 PM CST] ===Starting cron===
  # [Thu 10 Mar 2022 04:15:54 PM CST] Renew: 'weiyigeek.top'
  # [Thu 10 Mar 2022 04:15:54 PM CST] Skip, Next renewal time is: Mon 09 May 2022 06:38:32 AM UTC
  # [Thu 10 Mar 2022 04:15:54 PM CST] Add '--force' to force to renew.
  # [Thu 10 Mar 2022 04:15:54 PM CST] Skipped weiyigeek.top_ecc
  # [Thu 10 Mar 2022 04:15:54 PM CST] ===End cron===

# 9.重新申请签发证书, 设置定时任务后所有证书将每60天自动更新一次，当然您也可以强制更新证书：
acme.sh --renew -d example.com --force [--ecc]  # ecc 参数, 表明只是针对颁发的ECC证书。
acme.sh --renew -d example.com --force

# 10.停止更新证书，您可以执行以下操作从更新列表中删除证书：
acme.sh --remove -d example.com [--ecc]  # 注意, 证书/密钥文件不会从磁盘中删除。

# 11.启用自动升级保持acme.sh为最新版本，禁用自动更新。
acme.sh --upgrade --auto-upgrade
acme.sh --upgrade --auto-upgrade 0


# 12.查看证书列表
acme.sh --list

# 13.删除证书
acme.sh remove <SAN_Domains>
```

**Q: 将默认 CA 更改为 ZeroSSL?**  
A: 通常情况下acme.sh使用letsencrypt作为默认CA, 当前可以将默认CA更改为 ZeroSSL（https://github.com/acmesh-official/acme.sh/wiki/ZeroSSL.com-CA），但实际上并不建议这样做，因为，Let's Encrypt 可以颁发 EC 证书，而 ZeroSSL.com 则不支持颁发。

**Q: 如何颁发包含多个域的单个证书，每个域使用不同的验证方法`多域、SAN模式、Hybrid模式`。**

```
$ acme.sh  --issue  \
-d aa.com  -w /home/wwwroot/aa.com \
-d bb.com  --dns dns_cf \
-d cc.com  --apache \
-d dd.com  -w /home/wwwroot/dd.com
```

**Q: 如何生成pkcs12(pfx) 格式证书?**

> A: 颁发证书后可使用toPkcs命令将证书转换为 pkcs12(pfx) 格式，执行 `acme.sh --toPkcs -d example.com [--password pfx-password]` 命令即可。

**Q: 如何从从现有 CSR 颁发证书?**

> A: 从 v2.4.4 开始，acme.sh 支持从现有 csr 颁发证书, 具体操作如下所示:

```
# 显示 csr 中的主题和域名。
acme.sh --showcsr --csr /path/to/mycsr.csr

# 本地web root目录验证
acme.sh --signcsr --csr /path/to/mycsr.csr -w /path/to/webroot/

# DNS TXT 解析验证
acme.sh --signcsr --csr /path/to/mycsr/csr --dns dns_cf
```

**Q: 如何进行证书签发通知?**  
A: acme.sh 可以在 cronjob 中发送通知, 通知可以是电子邮件或任何其他支持的方式，例如请求 webhook 等， 参考地址：https://github.com/acmesh-official/acme.sh/wiki/notify

```
# 1.邮件通知
export MAIL_FROM="xxx@xxx.com" # or "Xxx Xxx <xxx@xxx.com>", currently works only with sendmail
export MAIL_TO="xxx@xxx.com"   # your account e-mail will be used as default if available
acme.sh --set-notify  --notify-hook mailgun  --notify-hook mail  \
  --notify-level 2 \
  --notify-mode 0

# 2.设置钉钉通知(钉钉),通过群机器人 webhook api 向钉钉群推送通知, 能力开通 https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq
export DINGTALK_WEBHOOK='https://oapi.dingtalk.com/robot/send?access_token=b05ccexxxxx'
export DINGTALK_KEYWORD=acme
acme.sh  --set-notify  --notify-hook dingtalk

# 3. QQ with self-built CQHTTP API, 通过 CoolQ 的插件 CQHTTP 将消息推送到 QQ, 需要您自行部署 CQHTTP 服务端.
# 四个环境变量可供传入:
* CQHTTP_TOKEN: 建议非空，将 CQHTTP 配置文件中您设置的 Access Token 填入。
* CQHTTP_USER: 必需，接收推送通知的 QQ 号码。您需要自行保证机器人号码可以向接收者的 QQ 号码发送消息。
* CQHTTP_APIROOT: 必需，您搭建的 CQHTTP 服务器的 URL (不包含斜杠结尾)。
* CQHTTP_CUSTOM_MSGHEAD: 可选，自定义的消息开头。默认值是 "A message from acme.sh:".

export CQHTTP_TOKEN="Itsjustat0ken,qwq"       # That's the access token
export CQHTTP_USER="10086"     # That's your QQ number (receiver)
export CQHTTP_APIROOT="http://cqhttp-server.local:5700"     # That's your server address
acme.sh  --set-notify  --notify-hook cqhttp
```

**Q: acme.sh 除了支持Cloudflare还支持那些DNSAPI供应商**  
描述: acme.sh 目前支持 cloudflare, dnspod, cloudxns, godaddy 以及 ovh 等数十种解析商的自动集成。

-   使用万网/阿里云的 NDS 解析操作方法：点击右上角头像 -> 选择 `AccessKey` -> 点击开始使用子用户 `AccessKey` -> 起个自定义名称 -> 搜索 NDS -> 选择 系统 `AliyunDNSFullAccess`
    

```
# 1.阿里云 --dns dns_ali ，获取阿里云的DNS API key，首先开通阿里云AccessKeys子账户，复制这里的AccessKeyID和AccessKeySecert值。
export Ali_Key="YourKey"
export Ali_Secret="YourSecert"
```

-   使用的是 DNSPod 解析服务，那就登录 DNSPod 官网生，成所需的 api id 和 api key, 操作方法点击右上角头像 -> 我的账号 -> 账号中心 -> 密钥管理 -> 创建密钥
    

```
# 2.DNSPod --dbs dns_dp
export DP_Id="YourID"
export DP_Key="YourKey"
```

-   错误信息:
    

```
"errors": [{
  "code": 1038,
  "message": "You cannot use this API for domains with a .cf, .ga, .gq, .ml, or .tk TLD (top-level domain). To configure the DNS settings for this domain, use the Cloudflare Dashboard."
}],
```

-   解决办法: 不能自动申请则只能WEB页面上进行申请。
    

至此本节完毕，敬请期待下一小节内容。

![](https://i0.hdslb.com/bfs/article/4adb9255ada5b97061e610b682b8636764fe50ed.png@progressive.webp)

> 欢迎各位志同道合的朋友一起学习交流，如文章有误请在下方留下您宝贵的经验知识，个人邮箱地址【master#weiyigeek.top】或者 个人公众号【WeiyiGeek】联系我。

![](https://i0.hdslb.com/bfs/article/02db465212d3c374a43c60fa2625cc1caeaab796.png@progressive.webp)

更多文章来源于【WeiyiGeek Blog - 为了能到远方，脚下的每一步都不能少】 个人博客。

博客地址: https://weiyigeek.top 

![](https://i0.hdslb.com/bfs/article/9a9cef9d789c08c7d7cb209475b46586777c9d26.png@942w_579h_progressive.webp)

专栏书写不易，如果您觉得这个专栏还不错的，请给这篇专栏【点个赞、投个币、收个藏、关个注，转个发】，这将对我的肯定，谢谢！。

-   echo  "【点个赞】，动动你那粗壮的拇指或者芊芊玉手，亲！"
    
-   printf("%s", "【投个币】，万水千山总是情，投个硬币行不行，亲！")
    
-   fmt.Printf("【收个藏】，阅后即焚不吃灰，亲！")  
    
-   System.out.println("【关个注】，后续浏览查看不迷路哟，亲！")
    
-   console.info("【转个发】，让更多的志同道合的朋友一起学习交流，亲！")