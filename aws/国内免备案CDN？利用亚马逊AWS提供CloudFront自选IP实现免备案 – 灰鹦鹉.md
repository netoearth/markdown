## 一、原理介绍

AWS提供CloudFront的CDN服务是不支持自选IP的,由于CloudFront边缘节点只提供东南亚导致速度十分捉鸡，甚至不如cloudflare提供的免费服务好用。很多市面上的免备案CDN都是采用亚洲的节点，比如香港的、日本的，但是都没有大陆的，因此这里说的是实现大陆内节点的免备案CDN。

因为某大佬意外发现可以通过类似于Cloudflare方式将备用域名Cname到cloudfrontCDN服务器的方式实现自选IP，且AWS在国内部署了不少的边缘节点(未开放)，这就提供了一个可能，即通过自选IP的方式将域名DNS解析到AWS国内边缘节点，实现国内免备案CDN。

虾皮路友情提醒：本方法公开后，估计很多人都会去用该方法实现免备案CDN，有可能什么时候官方堵住了这个漏洞。因此建议不要随意用在生产环境。

在实现这一操作中，面临以下几个问题：

1.如何找到AWS在国内的边缘节点2.如何进行CDN侧的自选IP设置3.如何实现自动push优选后的IP到DNS上

如果发送一个request到AWS边缘节点的https端口，返回的headers信息中,server会显示为CloudFront，而大部分都会是nginx或其他web服务器，可以以此作为区分。

### 1.扫描开放443端口的IP段

Cloudfront在国内的边缘节点肯定部署在IDC机房，但目前国内IDC机房所在的IP段与家宽基本混为一起，所以无法直接找到国内的IDC机房IP段，只能盲扫全国IP，好在Zmap扫描速度很快，我们要做的就是扫描IP段内所有开放了443端口的IP。可以通过以下命令安装Zmap并扫描IP

sudo apt-get install bison ed gawk gcc libc6-dev make

#扫描开放了443端口的IP,具体格式查看 zmap -h

zmap -w iplist.txt -p 443 -B 30M -o ip-new.txt

#安装Zmap运行环境 sudo apt-get install bison ed gawk gcc libc6-dev make #安装Zmap sudo apt install zmap #扫描开放了443端口的IP,具体格式查看 zmap -h zmap -w iplist.txt -p 443 -B 30M -o ip-new.txt

```
#安装Zmap运行环境
sudo apt-get install bison ed gawk gcc libc6-dev make
#安装Zmap
sudo apt install zmap
#扫描开放了443端口的IP,具体格式查看 zmap -h
zmap -w iplist.txt -p 443 -B 30M -o ip-new.txt
```

iplist.txt 为通过sftp上传的包含了某个IP段的txt文件,格式为X.X.X.0 /24

### 2.扫描AWSCloudFront边缘节点

可以构造一个python脚本,使用脚本进行扫描，脚本从github抄的,核心代码很简单，有能力的大佬可以改成python3

for x in xrange(0, int(SETTHREAD)):

threadl.append(tThread(queue))

class tThread(threading.Thread):

def \_\_init\_\_(self, queue):

threading.Thread.\_\_init\_\_(self)

while not self.queue.empty():

header ={'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36'}

aimurl = "http://"+host+":443"

response = requests.get(url=aimurl,headers=header,timeout=10)

serverText = response.headers\['server'\]

print "-"\*50+"\\n"+aimurl +"\\nServer: "+serverText

if (serverText == "CloudFront"):

with open("result.txt","a+") as file:

if \_\_name\_\_ == '\_\_main\_\_':

print '\\n############# Cloud Front Scan ################'

print ' Author hostloc.com'

print '################################################\\n'

with open(filepath, "r") as f:

iplist = f.read().splitlines()

#iplist = ip\_range(sys.argv\[1\].split('-')\[0\], sys.argv\[1\].split('-')\[1\])

print '\\n\[Note\] Will scan '+str(len(iplist))+" host...\\n"

except KeyboardInterrupt:

print 'Keyboard Interrupt!'

#!/usr/bin/env python # coding=utf-8 # python2.7 only import threading import requests import Queue import sys import re import numpy as np # def bThread(iplist): threadl = \[\] queue = Queue.Queue() for host in iplist: queue.put(host) for x in xrange(0, int(SETTHREAD)): threadl.append(tThread(queue)) for t in threadl: t.start() for t in threadl: t.join() #create thread class tThread(threading.Thread): def \_\_init\_\_(self, queue): threading.Thread.\_\_init\_\_(self) self.queue = queue def run(self): while not self.queue.empty(): host = self.queue.get() try: checkServer(host) except: continue def checkServer(host): header ={'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36'} aimurl = "http://"+host+":443" response = requests.get(url=aimurl,headers=header,timeout=10) serverText = response.headers\['server'\] if len(serverText) > 0: print "-"\*50+"\\n"+aimurl +"\\nServer: "+serverText if (serverText == "CloudFront"): if mutex.acquire(3): with open("result.txt","a+") as file: file.write(host+"\\n") file.close() mutex.release() if \_\_name\_\_ == '\_\_main\_\_': print '\\n############# Cloud Front Scan ################' print ' Author hostloc.com' print '################################################\\n' global SETTHREAD global mutex mutex = threading.Lock() try: SETTHREAD = sys.argv\[2\] filepath = sys.argv\[1\] with open(filepath, "r") as f: iplist = f.read().splitlines() #iplist = ip\_range(sys.argv\[1\].split('-')\[0\], sys.argv\[1\].split('-')\[1\]) print '\\n\[Note\] Will scan '+str(len(iplist))+" host...\\n" bThread(iplist) except KeyboardInterrupt: print 'Keyboard Interrupt!' sys.exit()

```
#!/usr/bin/env python
# coding=utf-8
# python2.7 only
import threading
import requests
import Queue
import sys
import re
import numpy as np
#
def bThread(iplist):
threadl = []
queue = Queue.Queue()
for host in iplist:
queue.put(host)
for x in xrange(0, int(SETTHREAD)):
threadl.append(tThread(queue))
for t in threadl:
t.start()
for t in threadl:
t.join()
#create thread
class tThread(threading.Thread):
def __init__(self, queue):
threading.Thread.__init__(self)
self.queue = queue
def run(self):
while not self.queue.empty():
host = self.queue.get()
try:
checkServer(host)
except:
continue
def checkServer(host):
header ={'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36'}
aimurl = "http://"+host+":443"
response = requests.get(url=aimurl,headers=header,timeout=10)
serverText = response.headers['server']
if len(serverText) > 0:
print "-"*50+"\n"+aimurl +"\nServer: "+serverText
if (serverText == "CloudFront"):
if mutex.acquire(3):
with open("result.txt","a+") as file:
file.write(host+"\n")
file.close()
mutex.release()
if __name__ == '__main__':
print '\n############# Cloud Front Scan ################'
print ' Author hostloc.com'
print '################################################\n'
global SETTHREAD
global mutex
mutex = threading.Lock()
try:
SETTHREAD = sys.argv[2]
filepath = sys.argv[1]
with open(filepath, "r") as f:
iplist = f.read().splitlines()
#iplist = ip_range(sys.argv[1].split('-')[0], sys.argv[1].split('-')[1])
print '\n[Note] Will scan '+str(len(iplist))+" host...\n"
bThread(iplist)
except KeyboardInterrupt:
print 'Keyboard Interrupt!'
sys.exit()
```

使用示例： python cfscan.py 443-ip.txt 200其中 cfscan.py 为 脚本名、 443-ip.txt为包含了IP的txt文件 200为线程数(卡顿调小)

最后结果会输出为当前目录下的 result.txt文件,结果即为可以使用的自选IP.

## 三、CDN设置

CDN设置中，首先你要给你要使用的域名申请SSL证书并配置好,确认https情况下可以访问你的网站。

### 1.设置源站解析

cloudfront中有一个源域名设置项，如下

注意这里不要填写你要用的域名，这里其实是要填源站IP的，但是CloudFront不支持直接填写源站IP，所以你必须开个子域名直接指向源站IP，比如这里的 www.vicho.me 就是在DNS解析处指向了源站IP，如下

### 2.设置CloudFront

登录你的AWS账号(不能是edustarter账号,好像现在edustarter账号也进不去Cloudfront了)，进入Cloudfront,创建分配。

原设置就填刚才你设置的，默认缓存配置根据程序不同需要设置不同的方式。

接下来重点就是，备用域名填你要使用的域名，然后通过ACM上传证书(DNS方式解析一条验证记录即可)

后续不表，完成分配之后，会分配给你一个cname地址，如下图所示：

### 3.解析

转到你的DNS服务商处，将要使用的域名cname解析到上面显示的地址处，然后把要使用的域名条件解析到你的自选IP上

等待解析生效即可使用。

## 三、优选IP并将PUSH到你的DNS上

优选IP的原理很简单，ping你在第一步选出来的边缘节点IP，找到最低ping的一个，需要进行以下操作：

找一台电信单线的服务器，ping ip列表，选出最低ping值的IP，即为电信最优IP找一台联通单线的服务器，ping ip列表，选出最低ping值的IP，即为联通最优IP找一台移动单线的服务器，ping ip列表，选出最低ping值的IP，即为移动最优IP

## 文章导航