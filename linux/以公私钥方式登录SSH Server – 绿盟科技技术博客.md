阅读： 30

## 一、背景介绍

公网上有台Ubuntu 22，考虑到暴力猜测SSH密码太疯狂，应该只允许公私钥方式登录SSH，端口应该开在非标端口。初始有个非标端口、随机生成的密码，先用密码登录，再配置公私钥登录。意外发现两个Windows客户端公私钥登录失败，排查后发现是服务端OpenSSH版本过高，出现向后不兼容的安全升级，而客户端较旧，两相一凑，歇菜。出于各种综合考虑，以向后兼容为主要矛盾，对此记录一番。

从安全角度讲，C/S两侧都升至最新版是最佳选择，但现实世界中不这样干的原因有很多。

## 二、生成公私钥

ssh-keygen -q -C “<comment>” -t rsa -b 4096 -N “<passphrase>” -m PEM -f rsa\_4096  
ls -l rsa\_4096\*

ssh-keygen生成两个文件，扩展名为.pub的是公钥，没有扩展名的是私钥，公钥需要上传到目标SSH Server。不喜欢所谓免密登录，指定了<passphrase>。

新版ssh-keygen默认不再是”-m PEM”，而是OpenSSH自己的一种新格式。某些旧版SecureCRT不认新格式私钥，为此必须给ssh-keygen显式指定”-m PEM”生成旧版私钥。

rsa\_4096形如

—–BEGIN RSA PRIVATE KEY—–  
Proc-Type: 4,ENCRYPTED  
DEK-Info: AES-128-CBC,…  
…  
—–END RSA PRIVATE KEY—–

作为对比，OpenSSH新版私钥形如

—–BEGIN OPENSSH PRIVATE KEY—–  
…  
—–END OPENSSH PRIVATE KEY—–

从安全性讲，应该用新版私钥，从向后兼容性讲，用旧版私钥。

rsa\_4096.pub形如

ssh-rsa …== <comment>

## 三、/etc/ssh/sshd\_config

PermitRootLogin yes  
StrictModes yes  
Protocol 2  
\# RSAAuthentication yes  
PubkeyAuthentication yes  
PubkeyAcceptedAlgorithms +ssh-rsa  
AuthorizedKeysFile .ssh/authorized\_keys  
PasswordAuthentication yes

将Protocol设为2，避免使用脆弱的1。注释掉RSAAuthentication，该选项只为1所用。测试阶段将PasswordAuthentication设为yes，公私钥方式登录SSH成功后再改成no。是否允许root远程登录看情况。

重启sshd使配置生效

service sshd restart

## 四、 $HOME/.ssh/authorized\_keys

在目标SSH Server上执行

mkdir -p $HOME/.ssh  
chmod 0500 $HOME/.ssh  
cat rsa\_4096.pub >> $HOME/.ssh/authorized\_keys  
chown -R <user> $HOME/.ssh  
chmod 0600 $HOME/.ssh/authorized\_keys

其实就是将公钥内容放入authorized\_keys，chown、chmod是防止sshd以不安全为由拒绝使用authorized\_keys。

$HOME/.ssh/known\_hosts(0644)是Linux作为SSH Client使用时自动生成的，与本文无关。

## 五、WinSCP

WinSCP和PuTTY必须使用some.ppk这种格式的私钥，可用puttygen完成格式转换

————————————————————————–  
<path>\\putty\\puttygen.exe  
Load  
rsa\_4096  
Save private key  
rsa\_4096.ppk  
————————————————————————–

rsa\_4096.ppk形如

————————————————————————–  
PuTTY-User-Key-File-2: ssh-rsa  
Encryption: aes256-cbc  
Comment: imported-openssh-key  
Public-Lines: 12  
…  
Private-Lines: 28  
…  
Private-MAC: …  
————————————————————————–

WinSCP需要对目标站点配置私钥

————————————————————————–  
WinSCP  
Advanced Site Settings  
SSH  
Authentication  
Private key file  
rsa\_4096.ppk  
————————————————————————–

我用WinSCP 5.17.8，Ubuntu 22中是OpenSSH 8.9p1，这两个C/S配对时存在兼容性问题。若服务端没有”PubkeyAcceptedAlgorithms +ssh-rsa”，WinSCP 5.17.8公私钥登录时提示”Server refused our key”；WinSCP 5.20做了安全增强，无需修改服务端配置。

参\[3\]，里面有一段

2021-10-12 17:36, OpenSSH 8.8 disabled ssh-rsa by default. Until WinSCP  
5.20 is released, add this to server’s sshd\_config to re-enable it:

PubkeyAcceptedAlgorithms +ssh-rsa

Ubuntu 20中是OpenSSH\_8.2p1，没这幺蛾子。

## 六、SecureCRT

SecureCRT 7.3.3密码登录Ubuntu 22时失败

Key exchange failed.

No compatible key exchange method. The server supports these methods:

curve25519-sha256,  
curve25519-sha256@libssh.org,  
ecdh-sha2-nistp256,  
ecdh-sha2-nistp384,  
ecdh-sha2-nistp521,  
sntrup761x25519-sha512@openssh.com,  
diffie-hellman-group-exchange-sha256,  
diffie-hellman-group16-sha512,  
diffie-hellman-group18-sha512,  
diffie-hellman-group14-sha256

No compatible cipher. The server supports these ciphers:

chacha20-poly1305@openssh.com,  
AES-128-CTR,  
AES-192-CTR,  
AES-256-CTR,  
aes128-gcm@openssh.com,  
aes256-gcm@openssh.com

意思是说服务端支持上面这一堆，但客户端当前配置不支持。

检查SecureCRT的”Session Options”，下面是一种方案，充分非必要

————————————————————————–  
Connection  
SSH2  
Authentication  
Password  
Key exchange  
ecdh-sha2-nistp256  
Advanced  
Cipher  
AES-256-CTR  
MAC  
SHA2-256  
————————————————————————–

先用Password登录成功，再配置公私钥登录

————————————————————————–  
Connection  
SSH2  
Authentication  
Publickey  
Properties  
Use session public key setting  
Use identity or certificate file  
rsa\_4096  
————————————————————————–

SecureCRT 7.3.3不认OpenSSH新版私钥

ssh-keygen -q -C “test” -t rsa -b 4096 -N “test” -f test\_4096

用test\_4096时，SecureCRT 7.3.3提示

————————————————————————–  
The private key file could not be found

Note that the public key file and private key file must have the same name(e.g., “Identity.pub” and “Identity”) and must be located in the same  
folder.

Unknown file format  
————————————————————————–

过去的套路突然不灵，起初我挺懵逼的，后来无意中发现新旧私钥格式不一样，问ChatGPT如何生成旧版私钥，从它的回答中意识到ssh-keygen可以指定生成哪种格式的私钥，在man手册中看到

By default OpenSSH will write newly-generated private keys in its own format. Setting a format of “PEM” when generating a supported private key  
type will cause the key to be stored in the legacy PEM private key format.

ssh-keygen的缺省值变了，为了保持向后兼容性，应该”ssh-keygen -m PEM”。

SecureCRT 7.3.3与Ubuntu 22中的OpenSSH 8.9p1，这两个C/S配对时存在兼容性问题。若服务端没有”PubkeyAcceptedAlgorithms +ssh-rsa”，SecureCRT 7.3.3公私钥登录时提示

————————————————————————–  
Public-key authentication with the server for user scz failed. Please verify username and public/private key pair.

The client has disconnected from the server. Reason: Unable to authenticate using any of the configured authentication methods.  
————————————————————————–

假设没有先验知识，只根据上述提示，很难定位PubkeyAcceptedAlgorithms，这个提示不合格。新版SecureCRT应该有安全增强，无需修改服务端配置，未实测。

实测SecureCRT 7.3.3不支持rsa-sha2-512、ed25519(ecdsa)。

## 七、 Linux SSH Client

服务端没有”PubkeyAcceptedAlgorithms +ssh-rsa”时，旧版SecureCRT、WinSCP公私钥登录失败，之前以为是不加此配置时服务端不认RSA公钥。意外发现服务端不加此配置时Linux SSH Client公私钥登录成功，用的是同一套RSA公私钥，那就不是服务端不认RSA公钥，应该有其他合理解释，与协商过程强相关，这让我对公私钥登录流程产生好奇。

用Linux SSH Client时，注意私钥权限最小化，否则拒绝使用

chmod 0400 rsa\_4096  
ssh -i rsa\_4096 <user>@<host>

查看SSH Server/Client版本

$ sshd -V  
OpenSSH\_8.9p1 Ubuntu-3, OpenSSL 3.0.2 15 Mar 2022

$ ssh -V  
OpenSSH\_8.4p1 Debian-3, OpenSSL 1.1.1i 8 Dec 2020

查看客户端连接指定站点时所用配置

$ ssh -G -i rsa\_4096 <user>@<host>  
…  
hostkeyalgorithms …,rsa-sha2-512,rsa-sha2-256,ssh-rsa  
…  
pubkeyacceptedkeytypes …,rsa-sha2-512,rsa-sha2-256,ssh-rsa  
…

查看客户端支持的(签名)算法

$ ssh -Q sig  
ssh-ed25519  
sk-ssh-ed25519@openssh.com  
ssh-rsa  
rsa-sha2-256  
rsa-sha2-512  
ssh-dss  
ecdsa-sha2-nistp256  
ecdsa-sha2-nistp384  
ecdsa-sha2-nistp521  
sk-ecdsa-sha2-nistp256@openssh.com  
webauthn-sk-ecdsa-sha2-nistp256@openssh.com

rsa-sha2-512/rsa-sha2-256/ssh-rsa三者公私钥都用RSA，签名算法分别用SHA-512、SHA-256、SHA-1。

参\[1\]

————————————————————————–  
HostkeyAlgorithms

the public key algorithms accepted for an SSH server to authenticate itself to an SSH client

PubkeyAcceptedKeyTypes

(ssh/sshd): the public key algorithms that will be attempted by the client, and accepted by the server for public-key authentication (e.g.  
via .ssh/authorized\_keys)  
————————————————————————–

客户端用PubkeyAcceptedKeyTypes指定用哪种(签名)算法，而非HostKeyAlgorithms。

rm $HOME/.ssh/known\_hosts  
ssh -i rsa\_4096 -o PubkeyAcceptedKeyTypes=rsa-sha2-256 <user>@<host>

上述命令登录成功，等价于无”-o”参数的默认情形

rm $HOME/.ssh/known\_hosts  
ssh -i rsa\_4096 -o PubkeyAcceptedKeyTypes=ssh-rsa <user>@<host>

上述命令登录失败，相当于模拟了旧版SecureCRT、WinSCP登录失败的情形，后两者无法指定PubkeyAcceptedKeyTypes。

☆ /var/log/auth.log

前述登录失败是旧版Client对新版Server，也有反过来的，同样可能失败，参\[2\]。C/S两侧在协商过程中向对方展示己方支持的算法，只要存在交集，就能成功，反之失败。从运维角度看，关注auth.log，从中发现失败的深层次原因，再在C/S两侧做相应调整，领会精神，此处不一一排列组合了。

tail -f /var/log/auth.log

旧版SecureCRT登录失败时，auth.log指明应修改PubkeyAcceptedAlgorithms

userauth\_pubkey: key type ssh-rsa not in PubkeyAcceptedAlgorithms \[preauth\]

Linux SSH Client模拟登录失败时，可见同样的日志。登录成功时看到

Accepted publickey for scz from <sip> port <sport> ssh2: RSA SHA256:…

客户端可以这样探测服务端支持哪些算法

$ ssh -o HostkeyAlgorithms=ssh-rsa <any>@<host>

Unable to negotiate with <server> port <port>: no matching host key type found. Their offer: rsa-sha2-512,rsa-sha2-256,ecdsa-sha2-nistp256,ssh-ed25519

与此同时服务端auth.log中有

Unable to negotiate with <client> port <port>: no matching host key type found. Their offer: ssh-rsa \[preauth\]

## 八、ChatGPT

我没看过OpenSSH源码，也不了解公私钥登录流程，索性让ChatGPT忽悠忽悠我。粗看了两眼，答得不算离谱，有一定可信度。

————————————————————————–  
Q: scz

OpenSSH公私钥登录认证时，大致流程是什么？

A: ChatGPT

1\. 用户生成一对公私钥，私钥存储在本地计算机上，公钥复制到远程服务器上  
2\. 上传公钥到服务器，将公钥复制~/.ssh/authorized\_keys  
3\. SSH客户端连接服务器，客户端将本地公钥发送到服务器  
4\. 服务器收到客户端发送的公钥，与authorized\_keys中的公钥列表进行匹配  
5\. 若服务器成功验证了公钥，会用该公钥加密一个随机数，并将其发送给客户端  
6\. 客户端收到服务器发送的加密随机数，用本地私钥对其解密  
7\. 客户端和服务器拥有同一随机数，它们用之作种子，利用密钥协商算法生成对称  
加密密钥  
8\. 客户端和服务器用前述对称加密密钥进行后续通信  
9\. 公私钥只用于登录认证阶段，一旦认证通过，客户端和服务器之间的通信使用对  
称加密通信

Q: scz

在此过程中，signature algorithm 体现在哪里？

A: ChatGPT

客户端向服务器发送本地公钥，并使用私钥生成一个数字签名(signature)。服务器  
收到客户端发送的公钥和数字签名，与authorized\_keys中的公钥进行匹配，并校验  
数字签名，若验证通过，认为客户端合法，可以继续。

Q: scz

假设公私钥保持不变，新版OpenSSH服务端需要”PubkeyAcceptedAlgorithms +ssh-rsa”，  
否则某些旧版客户端公私钥登录失败，某些新版客户端公私钥登录成功，请从算法角  
度简述造成这种现象的原因

A: ChatGPT

造成这种现象的原因是OpenSSH公私钥登录认证过程中涉及的算法版本升级导致的不  
兼容。OpenSSH公私钥登录认证涉及到多种算法，包括加密算法、HASH算法和签名算  
法等，这些算法可能随时间推移而发生改变，例如在某个版本的OpenSSH中，可能使  
用SHA-1来计算数字签名，但在新版中，可能改为更安全的算法，例如SHA-256。为了  
确保兼容性，需要使用相同的算法，通过适当的参数配置来确保新旧版本之间可以正  
常通信。

Q: scz

Linux命令行ssh公私钥登录时，如何指定用哪种 signature algorithm

A: ChatGPT

ssh -i <private\_key> -o HostKeyAlgorithms=<algorithm> <user>@<host>

可将参数添加到 ~/.ssh/config 中，避免每次都输入相同的参数。

Q: scz

你不是一般的牛逼

A: ChatGPT

感谢夸奖！我只是一个机器人。  
————————————————————————–

上述ChatGPT回答只是不离谱，但不要当成标准答案，更多细节参看RFC 4252/4253。

## 九、RFC 4252

7\. Public Key Authentication Method: “publickey”

☆ RFC 4253

6.6. Public Key Algorithms

### 参考文献

\[1\] OpenSSH Legacy Options  
https://www.openssh.com/legacy.html

https://www.openssh.com/txt/release-8.8  
(Potentially-incompatible changes)

OpenSSH 8.3 released (and ssh-rsa deprecation notice)  
https://lwn.net/Articles/821544/

\[2\] SSH without password does not work after upgrading from 18.04 to 22.04 – \[2022-04-23\]  
https://askubuntu.com/questions/1404049/ssh-without-password-does-not-work-after-upgrading-from-18-04-to-22-04

Ubuntu 22.04 SSH the RSA key isn’t working since upgrading from 20.04 – \[2022-05-17\]  
https://askubuntu.com/questions/1409105/ubuntu-22-04-ssh-the-rsa-key-isnt-working-since-upgrading-from-20-04

SSH server gives “userauth\_pubkey: key type ssh-rsa not in PubkeyAcceptedAlgorithms \[preauth\]” when connecting with Putty – \[2022-10-19\]  
https://unix.stackexchange.com/questions/721606/ssh-server-gives-userauth-pubkey-key-type-ssh-rsa-not-in-pubkeyacceptedalgorit  
(ssh -Q sig)

\[3\] “Server refused our key” after updating to Ubuntu 22.04 beta on a LAMP stack – \[2022-04-09\]  
https://winscp.net/forum/viewtopic.php?t=32133

Bug 1952 – Support rsa-sha2-256 and rsa-sha2-512 SSH public key algorithms  
https://winscp.net/tracker/1952

\[4\] The Secure Shell (SSH) Protocol Assigned Numbers  
https://www.rfc-editor.org/rfc/rfc4250

The Secure Shell (SSH) Authentication Protocol  
https://www.rfc-editor.org/rfc/rfc4252

The Secure Shell (SSH) Transport Layer Protocol  
https://www.rfc-editor.org/rfc/rfc4253

**版权声明**  
本站“技术博客”所有内容的版权持有者为绿盟科技集团股份有限公司（“绿盟科技”）。作为分享技术资讯的平台，绿盟科技期待与广大用户互动交流，并欢迎在标明出处（绿盟科技-技术博客）及网址的情形下，全文转发。  
上述情形之外的任何使用形式，均需提前向绿盟科技（010-68438880-5462）申请版权授权。如擅自使用，绿盟科技保留追责权利。同时，如因擅自使用博客内容引发法律纠纷，由使用者自行承担全部法律责任，与绿盟科技无关。