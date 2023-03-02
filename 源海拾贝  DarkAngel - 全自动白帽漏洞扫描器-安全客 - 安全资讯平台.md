![](https://p0.ssl.qhimg.com/t0199042c4883e0c1ae.png)

___

DarkAngel 是一款全自动白帽漏洞扫描器，从hackerone、bugcrowd资产监听到漏洞报告生成、企业微信通知。

DarkAngel 下载地址：[github.com/Bywalks/DarkAngel](https://github.com/Bywalks/DarkAngel)

当前已支持的功能：

-   hackerone资产监听；
-   bugcrowd资产监听；
-   自定义资产添加；
-   子域名扫描；
-   网站指纹识别；
-   漏洞扫描；
-   漏洞报告自动生成；
-   企业微信通知扫描结果；
-   前端显示扫描结果；

## 自动生成漏洞报告

自动生成漏洞报告 – MarkDown格式 – 存放地址/root/darkangel/vulscan/results/report

![](https://p3.ssl.qhimg.com/t01480c48d655c8faa5.png)

支持自添加漏洞报告模板，目前已添加漏洞报告模板如下，漏洞名配置为nuclei模板文件名即可

![](https://p1.ssl.qhimg.com/t0185bcd44951f28027.png)

自定义漏洞报告模板格式

![](https://p3.ssl.qhimg.com/t0146f2ecf75c1d885e.png)

## 企业微信通知

可先查看如何获取配置：[企业微信开发接口文档](https://developer.work.weixin.qq.com/document/path/90487)

获取参数后，在/root/darkangel/vconfig/config.ini中配置参数，即可启用企业微信通知

微信通知 – 漏洞结果

![](https://p0.ssl.qhimg.com/t01404087713903b72d.png)

微信通知 – 扫描进程

![](https://p0.ssl.qhimg.com/t0114aca2c2d06fbcc6.png)

## 安装

整体项目架构ES+Kibana+扫描器，所以安装需要三个部分

ES镜像：

```
拉取ES镜像
docker pull bywalkss/darkangel:es7.9.3

部署ES镜像
docker run -e ES_JAVA_OPTS="-Xms1024m -Xms1024m" -d -p 9200:9200 -p 9300:9300 --name elasticsearch elasticsearch:7.9.3

查看日志
docker logs -f elasticsearch

出现问题，执行命令
sysctl -w vm.max_map_count=262144

重启docker
docker start elasticsearch
```

Kibana镜像：

```
拉取Kibana镜像
docker pull bywalkss/darkangel:kibana7.9.3

部署Kibana镜像（修改一下es-ip）
docker run --name kibana -e ELASTICSEARCH_URL=http://es-ip:9200 -p 5601:5601 -d docker.io/bywalkss/darkangel:kibana7.9.3

查看日志
docker logs -f elasticsearch

出现问题，执行命令
sysctl -w vm.max_map_count=262144

重启docker
docker start elasticsearch
```

扫描器镜像：

```
拉取扫描器镜像
docker pull bywalkss/darkangel:v1

部署扫描器
docker run -it -d -v /root/darkangel:/root/darkangel --name darkangel bywalkss/darkangel:v1
```

docker容器内挂载目录无权限  
运行容器时：—privileged=true

## 用法

```
usage:  [-h] [--scan-new-domain]
        [--add-domain-and-scan ADD_DOMAIN_AND_SCAN [ADD_DOMAIN_AND_SCAN ...]]
        [--offer-bounty {yes,no}] [--nuclei-file-scan]
        [--nuclei-file-scan-by-new-temp NUCLEI_FILE_SCAN_BY_NEW_TEMP]
        [--nuclei-file-scan-by-new-add-temp NUCLEI_FILE_SCAN_BY_NEW_ADD_TEMP]
        [--nuclei-file-scan-by-temp-name NUCLEI_FILE_SCAN_BY_TEMP_NAME]
        [--nuclei-file-polling-scan] [--delete]

DarkAngel is a white hat scanner. Every user makes the Internet more secure.

--------------------------------------------------------------------------------

optional arguments:
  -h, --help            show this help message and exit
  --scan-new-domain     scan new domain from h1 and bc
  --add-domain-and-scan ADD_DOMAIN_AND_SCAN [ADD_DOMAIN_AND_SCAN ...]
                        scan new domain from h1 and bc
  --offer-bounty {yes,no}
                        set add domain is bounty or no bounty
  --nuclei-file-scan    scan new domain from h1 and bc
  --nuclei-file-scan-by-new-temp NUCLEI_FILE_SCAN_BY_NEW_TEMP
                        use new template scan five file by nuclei
  --nuclei-file-scan-by-new-add-temp NUCLEI_FILE_SCAN_BY_NEW_ADD_TEMP
                        add new template scan five file by nuclei
  --nuclei-file-scan-by-temp-name NUCLEI_FILE_SCAN_BY_TEMP_NAME
                        use template scan five file by nuclei
  --nuclei-file-polling-scan
                        five file polling scan by nuclei
```

### —scan-new-domain

`$ python3 darkangel.py --scan-new-domain`

-   监听hackerone和bugcrowd域名并进行扫描（第一次使用时会把hackerone和bugcrowd域名全部添加进去，资产过多的情况下做好准备，扫描时间很长）

![](https://p0.ssl.qhimg.com/t01290d4cdb660368ad.png)

### —add-domain-and-scan

`$ python3 darkangel.py --add-domain-and-scan program-file-name1 program-file-name2 --offer-bounty yes/no`

-   自定义添加扫描域名，并对这些域名进行漏洞扫描
-   文件名为厂商名称，文件内存放需扫描域名
-   需提供—offer-bounty参数，设置域名是否提供赏金

![](https://p5.ssl.qhimg.com/t01d70625e3b9179932.png)

![](https://p0.ssl.qhimg.com/t01dfae4169675cc90a.png)

扫描结束后，会把子域名结果存在在/root/darkangel/vulscan/results/urls目录，按照是否提供赏金分别存放在，bounty\_temp\_urls\_output.txt、nobounty\_temp\_urls\_output.txt文件内

### —nuclei-file-scan

`$ python3 darkangel.py --nuclei-file-scan`

-   用nuclei扫描20个url文件

![](https://p1.ssl.qhimg.com/t01836ded981e4ec1c9.png)

url列表存放位置

![](https://p1.ssl.qhimg.com/t0106bf1419121f720e.png)

### —nuclei-file-polling-scan

`$ python3 darkangel.py --nuclei-file-polling-scan`

-   轮询用nuclei扫描20个url文件，可把该进程放在后台，轮询扫描，监听是否url列表是否存在新漏洞出现

### —nuclei-file-scan-by-new-temp

`$ python3 darkangel.py --nuclei-file-scan-by-new-temp nuclei-template-version`

-   监听nuclei-template更新，当更新时，对url列表进行扫描

当前nuclei-template版本为9.3.1

![](https://p4.ssl.qhimg.com/t0139d00912a7ad44df.png)

执行命令，监听9.3.2版本更新

![](https://p2.ssl.qhimg.com/t0167cae9c1511bc8ff.png)

企业微信通知

![](https://p3.ssl.qhimg.com/t01f714a79e38dc0d8f.png)

url列表存放位置

![](https://p4.ssl.qhimg.com/t01877d394675bab8b6.png)

### —nuclei-file-scan-by-new-add-temp

`$ python3 darkangel.py --nuclei-file-scan-by-new-add-temp nuclei-template-id`

-   监听nuclei单template更新，当更新时，用该template对url列表进行扫描，这里是打了个时间差，某些时候先提交tempalte，验证后才会加入nuclei模板，在还未加入时，我们已经监听并进行扫描，扫描后id会自动增加，监听并进行扫描

查看nuclei单template的id，这里为6296

![](https://p2.ssl.qhimg.com/t0134a208f2fc8db3ab.png)

执行命令，对该template进行扫描

![](https://p4.ssl.qhimg.com/t01d356bae15ce4400f.png)

url列表存放位置

![](https://p1.ssl.qhimg.com/t01b12b36d4111086fd.png)

### —nuclei-file-scan-by-temp-name

`$ python3 darkangel.py --nuclei-file-scan-by-temp-name nuclei-template-name`

-   用单template对url列表进行扫描

![](https://p0.ssl.qhimg.com/t01cc3167c439175959.png)

## 结果显示

前端 – 扫描厂商

![](https://p1.ssl.qhimg.com/t01d1223480cf998c76.png)

前端 – 扫描域名

![](https://p2.ssl.qhimg.com/t011fa07fef728896fe.png)

前端 – 扫描结果

![](https://p4.ssl.qhimg.com/t0165d2e7b6627dde06.png)

微信通知 – 扫描进程

![](https://p0.ssl.qhimg.com/t017d97f71ffbc23d19.png)

微信通知 – 漏洞结果

![](https://p2.ssl.qhimg.com/t012642830ce1644c36.png)

## 注意事项

-   本工具仅用于合法合规用途，严禁用于违法违规用途。