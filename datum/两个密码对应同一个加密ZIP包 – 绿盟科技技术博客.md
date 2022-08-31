阅读： 30

参看

https://twitter.com/\_mohemiv/status/1561044393880178689

@\_mohemiv分享了一则消息，特定条件下两个解压密码对应同一个加密zip包。云海跟我说的这事，我几乎不上推特，各种原因吧。

Linux环境中测试步骤如下

————————————————————————–  
a) 加密压缩7z a x.zip /etc/passwd -mem=AES256 -p

提示输入密码时，用这个

Nev1r-G0nna-G2ve-Y8u-Up-N5v1r-G1nna-Let-Y4u-D1wn-N8v4r-G5nna-D0sert-You

b) 解压7z e x.zip

提示输入密码时，用这个

pkH8a0AqNbHcdw8GrmSp  
————————————————————————–

步骤a、b中的两组密码可以互换，加密时用短的，解密时用长的。

Windows环境中用7-Zip GUI测试时，注意选压缩格式 zip (默认是7z) 加密算法 AES-256 (默认是ZipCrypto)

看着有点神奇，但这不是魔法，不是前述URL所说的”后门口令”。它有一个合理的解释，参看

https://twitter.com/Unblvr1/status/1561112433812463616

@Unblvr1解释了原理

ZIP uses PBKDF2, which hashes the input if it’s too big. That hash (as rawbytes) becomes the actual password. Try to hash the first password withSHA1 and decode the hexdigest to ASCII.

他的意思是

$ echo -ne “Nev1r-G0nna-G2ve-Y8u-Up-N5v1r-G1nna-Let-Y4u-D1wn-N8v4r-G5nna-D0sert-You” | shasum706b4838613041714e62486364773847726d5370 -$ echo -ne “Nev1r-G0nna-G2ve-Y8u-Up-N5v1r-G1nna-Let-Y4u-D1wn-N8v4r-G5nna-D0sert-You” | shasum | cut -f1 -d’ ‘ | xxd -r -ppkH8a0AqNbHcdw8GrmSp

在限定条件下zip的密码逻辑有点奇怪，明文口令超长时对之求SHA1，用SHA1当密码；明文口令足够短时，直接用作密码。看7-Zip源码可以找出这段逻辑，了解多长算超长，我懒。

仔细看这个逻辑，理论上对步骤b中密码的字节流求”SHA1 Collision”，有无穷多个碰撞等着你，只要”碰撞”对应可打印字符串，就可用作步骤a中密码。这种运算量太大，对普通人不现实，我连MD5碰撞都没试过，更别说SHA1碰撞。

换个思路，穷举超长字符串，计算SHA1，只要20个字节全部位于ASCII范围，就制造出了一对zip解压密码。这个运算量比”哈希碰撞 (SHA1 Collision)”小，我接着懒。

版权声明  
本站“技术博客”所有内容的版权持有者为绿盟科技集团股份有限公司（“绿盟科技”）。作为分享技术资讯的平台，绿盟科技期待与广大用户互动交流，并欢迎在标明出处（绿盟科技-技术博客）及网址的情形下，全文转发。  
上述情形之外的任何使用形式，均需提前向绿盟科技（010-68438880-5462）申请版权授权。如擅自使用，绿盟科技保留追责权利。同时，如因擅自使用博客内容引发法律纠纷，由使用者自行承担全部法律责任，与绿盟科技无关。