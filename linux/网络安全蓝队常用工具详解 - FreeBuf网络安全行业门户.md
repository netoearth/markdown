## 网络安全蓝队常用工具详解

这个github存储库包含了**35+****工具**和**资源**，可以用于**蓝色团队活动**。

有些工具可能是专门为蓝色团队设计的，而其他工具则更通用，可以在蓝色团队上下文中进行调整使用。

> **Warning**
> 
> \*本资料库中的资料仅供参考及教育用途。它们不打算用于任何非法活动

## 工具列表

**网络发现与映射**

-   -   **[Nmap](https://wxweb.vercel.app/#nmap)**网络扫描
    -   **[Nuclei](https://wxweb.vercel.app/#nuclei)**漏洞扫描工具
    -   **[Masscan](https://wxweb.vercel.app/#masscan)**快速网络扫描工具
    -   **[快速IP扫描](https://wxweb.vercel.app/#angry-ip-scanner)**IP/port scanner
    -   **[ZMap](https://wxweb.vercel.app/#zmap)**大型网络扫描器
    -   **[Shodan](https://wxweb.vercel.app/#shodan)**面向互联网的资产搜索引擎

**漏洞管理**

-   -   **[OpenVAS](https://wxweb.vercel.app/#openvas)**开源漏洞扫描器
    -   **[Nessus Essentials](https://wxweb.vercel.app/#nessus-essentials)**系统漏洞扫描工具
    -   **[Nexpose](https://wxweb.vercel.app/#nexpose)**漏洞管理工具

**安全监视-日志管理/日志分析**

-   -   **[Sysmon](https://wxweb.vercel.app/#sysmon)**windows系统监视器
    -   **[Kibana](https://wxweb.vercel.app/#kibana)**数据探索和可视化工具
    -   **[Logstash](https://wxweb.vercel.app/#logstash)**数据收集与处理工具

**威胁工具和技术**

-   -   **[lolbas-project.github.io](https://wxweb.vercel.app/#lolbas-projectgithubio)**Living Off The Land Windows Binaries
    -   **[gtfobins.github.io](https://wxweb.vercel.app/#gtfobinsgithubio)**Living Off The Land Linux Binaries
    -   **[filesec.io](https://wxweb.vercel.app/#filesecio)**攻击者文件扩展名
    -   **[KQL Search](https://wxweb.vercel.app/#kql-search)**KQL查询聚合器
    -   **[Unprotect Project](https://wxweb.vercel.app/#unprotect-project)**恶意软件规避技术知识库

**威胁情报工具集合**

-   -   **[Maltego](https://wxweb.vercel.app/#maltego)**威胁情报工具平台
    -   **[MISP](https://wxweb.vercel.app/#misp)**恶意软件信息共享平台
    -   **[ThreatConnect](https://wxweb.vercel.app/#threatconnect)**威胁数据聚合工具

**应急响应计划**

-   -   **[NIST](https://wxweb.vercel.app/#nist)**网络安全框架
    -   **[Incident Response Plan](https://wxweb.vercel.app/#incident-response-plan)**应急响应框架
    -   **[Ransomware Response Plan](https://wxweb.vercel.app/#ransomware-response-plan)**勒索事件应急响应框架

**恶意软件检测和分析工具/平台**

-   -   **[VirusTotal](https://wxweb.vercel.app/#virustotal)**恶意软甲IOCs（妥协指标）分享平台
    -   **[IDA](https://wxweb.vercel.app/#ida)**恶意软件反汇编器和调试器
    -   **[Ghidra](https://wxweb.vercel.app/#ghidra)**恶意软件逆向工程工具

**数据恢复工具集合**

-   -   **[Recuva](https://wxweb.vercel.app/#recuva)**文件恢复工具
    -   **[Extundelete](https://wxweb.vercel.app/#extundelete)**Ext3或ext4分区恢复工具
    -   **[TestDisk](https://wxweb.vercel.app/#testdisk)**数据恢复工具

**数字取证工具集合**

-   -   **[SANS SIFT](https://wxweb.vercel.app/#sans-sift)**法医工具包
    -   **[The Sleuth Kit](https://wxweb.vercel.app/#the-sleuth-kit)**磁盘映像分析工具
    -   **[Autopsy](https://wxweb.vercel.app/#autopsy)**数字取证平台

**网络安全意识培训**

-   -   **[TryHackMe](https://wxweb.vercel.app/#tryhackme)**网络安全挑战平台
    -   **[HackTheBox](https://wxweb.vercel.app/#hackthebox)**网络安全挑战平台
    -   **[PhishMe](https://wxweb.vercel.app/#phishme)**网络钓鱼训练平台

**网络安全交流和合作**

-   -   **[Twitter](https://wxweb.vercel.app/#twitter)**Twitter上的安全账户
    -   **[Facebook TheatExchange](https://wxweb.vercel.app/#facebook-theatexchange)**恶意指标共享平台

## 网络发现与映射

用于扫描和绘制网络、发现设备和服务以及识别潜在漏洞的工具

### (#tool-list)Nmap

Nmap (Network Mapper的缩写)是一个免费的开源网络扫描工具，用于发现计算机网络上的主机和服务，并探测有关它们特征的信息。

它可以用来确定网络上哪些端口是开放的，以及哪些服务正在这些端口上运行。包括识别网络安全漏洞的能力。

**安装：**

您可以从这里下载最新版本。

**使用：**

```
# Scan a single IPnmap 192.168.1.1# Scan a rangenmap 192.168.1.1-254# Scan targets from a filenmap -iL targets.txt# Port scan for port 21nmap 192.168.1.1 -p 21# Enables OS detection, version detection, script scanning, and traceroutenmap 192.168.1.1 -A
```

不错的用法小抄。

![image](https://image.3001.net/images/20230115/1673718576_63c2eb308188c8aa17771.png!small "null")image

图片来自https://kirelos.com/nmap-version-scan-determining-the-version-and-available-services/

### (#tool-list)Nuclei

一个专门的工具，旨在自动检测web应用程序、网络和基础设施中的漏洞。 nucleus使用预定义模板探测目标并识别潜在漏洞。它可用于测试单个主机或一系列主机，并可配置为运行各种测试以检查不同类型的漏洞。

**安装**

```
git clone https://github.com/projectdiscovery/nuclei.git; \cd nuclei/v2/cmd/nuclei; \go build; \mv nuclei /usr/local/bin/; \nuclei -version;
```

**使用**

```
# All the templates gets executed from default template installation path.nuclei -u https://example.com# Custom template directory or multiple template directorynuclei -u https://example.com -t cves/ -t exposures/# Templates can be executed against list of URLsnuclei -list http_urls.txt# Excluding single templatenuclei -list urls.txt -t cves/ -exclude-templates cves/2020/CVE-2020-XXXX.yaml
```

完整的使用信息可以在这里(https://nuclei.projectdiscovery.io/nuclei/get-started/#running-nuclei)找到。

![image](https://image.3001.net/images/20230115/1673718597_63c2eb45e1f4b8d693f9f.png!small "null")image

Image used from https://www.appsecsanta.com/nuclei

### (#tool-list)Masscan

类似于nmap的端口扫描程序，但速度要快得多，可以在短时间内扫描大量端口。

Masscan使用一种称为“SYN扫描”的新技术来扫描网络，这使它能够非常快速地扫描大量端口。

**安装: (Apt)**

```
sudo apt install masscan
```

**安装: (Git)**

```
sudo apt-get install clang git gcc make libpcap-devgit clone https://github.com/robertdavidgraham/masscancd masscanmake
```

**使用:**

```
# Scan for a selection of ports (-p22,80,445) across a given subnet (192.168.1.0/24)masscan -p22,80,445 192.168.1.0/24# Scan a class B subnet for ports 22 through 25masscan 10.11.0.0/16 -p22-25# Scan a class B subnet for the top 100 ports at 100,000 packets per secondmasscan 10.11.0.0/16 ‐‐top-ports 100 ––rate 100000# Scan a class B subnet, but avoid the ranges in exclude.txtmasscan 10.11.0.0/16 ‐‐top-ports 100 ‐‐excludefile exclude.txt
```

![image](https://image.3001.net/images/20230115/1673718638_63c2eb6e6268a09e3b343.png!small "null")image

Image used from https://kalilinuxtutorials.com/masscan/

### (#tool-list)Angry IP Scanner

一个免费的开源工具，用于扫描IP地址和端口。

这是一个跨平台的工具，被设计成快速且易于使用，并且可以扫描整个网络或一系列IP地址来找到活动的主机。

Angry IP Scanner还可以检测设备的主机名和MAC地址，并可用于执行基本的ping扫描和端口扫描。

**安装:**

您可以从这里下载最新版本。

**使用:**

Angry IP Scanner可以通过GUI使用。

完整的使用信息和文档可以在这里找到。

![image](https://image.3001.net/images/20230115/1673718652_63c2eb7c9c53066667bb9.png!small "null")image

Image used from https://angryip.org/screenshots/

### (#tool-list)ZMap

ZMap是一个网络扫描器，用于对IPv4地址空间或大部分地址空间进行全面扫描。

在一台具有千兆以太网连接的典型台式计算机上，ZMap能够在45分钟内扫描整个公共IPv4地址空间。

**安装:**

您可以从这里下载最新版本。

**使用:**

```
# Scan only 10.0.0.0/8 and 192.168.0.0/16 on TCP/80zmap -p 80 10.0.0.0/8 192.168.0.0/16
```

完整的使用信息可以在这里(https://github.com/zmap/zmap/wiki)找到。

![image](https://image.3001.net/images/20230115/1673718668_63c2eb8c24960838effa3.png!small "null")image

Image used from https://www.hackers-arise.com/post/zmap-for-scanning-the-internet-scan-the-entire-internet-in-45-minutes

### (#tool-list)Shodan

Shodan是一个针对联网设备的搜索引擎。

它在互联网上搜索资产，允许用户搜索特定的设备并查看有关它们的信息。

这些信息可以包括设备的IP地址、正在运行的软件和版本，以及设备的类型。

**安装:**

搜索引擎可通过https://www.shodan.io/dashboard访问。

**使用:**

Shodan query fundamentals

Shodan query examples

Nice query cheatsheet

![image](https://image.3001.net/images/20230115/1673718677_63c2eb959130115f687a9.png!small "null")image

Image used from https://www.shodan.io/

## 脆弱性管理

用于识别、优先排序和减轻网络和单个设备上的漏洞的工具

### (#tool-list)OpenVAS

OpenVAS是一个开源漏洞扫描器，可以帮助识别软件和网络中的安全漏洞。

它是一种可用于执行网络安全评估的工具，通常用于识别系统和应用程序中的漏洞，以便对其进行修补或缓解。

OpenVAS由Greenbone Networks公司开发，是一款免费的开源软件应用程序。

**安装: (Kali)**

```
apt-get updateapt-get dist-upgradeapt-get install openvasopenvas-setup
```

**使用:**

```
openvas-start
```

Visit https://127.0.0.1:9392, accept the SSL certificate popup and login with admin credentials:

-   • username:admin
-   • password:(Password in openvas-setup command output)

![image](https://image.3001.net/images/20230115/1673718687_63c2eb9fc77d9bdffd8d6.png!small "null")image

Image used from https://www.kali.org/blog/openvas-vulnerability-scanning/

### (#tool-list)Nessus Essentials

Nessus是一个漏洞扫描器，可以帮助识别和评估存在于网络或计算机系统中的漏洞。

它是一种用于执行安全评估的工具，可用于识别系统和应用程序中的漏洞，以便对其进行修补或缓解。

Nessus由Tenable, Inc.开发，有免费和付费两种版本:

\-免费版本，称为Nessus Essentials，仅供个人使用，与付费版本相比，其功能有限。 -付费版，称为Nessus专业版，功能更全面，旨在专业环境中使用。

**安装:**

Register for a Nessus Essentials activation code here and download.

Purchase Nessus Professional from here.

**使用:**

Extensive documentation can be found here.

Nessus Plugins Search

Tenable Community

![image](https://image.3001.net/images/20230115/1673718698_63c2ebaa23141ef30594a.png!small "null")image

Image used from https://www.tenable.com

### (#tool-list)Nexpose

Nexpose是Rapid7开发的漏洞管理工具。它旨在帮助组织识别和评估其系统和应用程序中的漏洞，以降低风险并提高安全性。

Nexpose可用于扫描网络、设备和应用程序，以识别漏洞并提供补救建议。

它还提供了资产发现、风险优先级以及与Rapid7漏洞管理平台中的其他工具集成等功能。

**安装:**

For detailed installation instructions see here.

**使用:**

For full login information see here.

For usage and scan creation instructions see here.

![image](https://image.3001.net/images/20230115/1673718723_63c2ebc366e79c4e6b30f.png!small "null")image

Image used from https://www.rapid7.com/products/nexpose/

## 安全监视

用于收集和分析安全日志和其他数据源以识别潜在威胁和异常活动的工具

### (#tool-list)Sysmon

Sysmon是一个Windows系统监视器，它跟踪系统活动并将其记录到Windows事件日志中。

它提供了关于系统活动的详细信息，包括进程创建和终止、网络连接以及文件创建时间的更改。

Sysmon可配置为监视特定事件或进程，并可用于向管理员警告系统上的可疑活动。

**安装:**

Download the sysmon binary from here.

**使用:**

```
# Install with default settings (process images hashed with SHA1 and no network monitoring)sysmon -accepteula -i# Install Sysmon with a configuration file (as described below)sysmon -accepteula -i c:\windows\config.xml# Uninstallsysmon -u# Dump the current configurationsysmon -c
```

Full event filtering information can be found here.

The Microsoft documentation page can be found here.

![image](https://image.3001.net/images/20230115/1673718736_63c2ebd04b0df9ea0770e.png!small "null")image

Image used from https://nsaneforums.com/topic/281207-sysmon-5-brings-registry-modification-logging/

### (#tool-list)Kibana

Kibana是一个开源的数据可视化和探索工具，经常与Elasticsearch结合使用，用于日志分析。

Kibana提供了一个用户友好的界面，用于搜索、可视化和分析日志数据，这有助于识别可能表明安全威胁的模式和趋势。

Kibana可用于分析广泛的数据源，包括系统日志、网络日志和应用程序日志。它还可以用于创建自定义仪表板和警报，以帮助安全团队随时了解潜在威胁并快速响应事件。

**安装:**

You can download Kibana from here.

Installation instructions can be found here.

**使用: (Visualize and explore log data)**

Kibana provides a range of visualization tools that can help you identify patterns and trends in your log data. You can use these tools to create custom dashboards that display relevant metrics and alerts.

**使用: (Threat Alerting)**

Kibana可以配置为在日志数据中检测到特定模式或异常时发送警报。您可以设置警报，以通知您潜在的安全威胁，例如失败的登录尝试或到已知恶意IP地址的网络连接。

Nice blog about querying and visualizing data in Kibana.

![image](https://image.3001.net/images/20230115/1673718744_63c2ebd8aafc9070ceaf6.png!small "null")image

Image used from https://www.pinterest.co.uk/pin/analysing-honeypot-data-using-kibana-and-elasticsearch--684758318328369269/

### (#tool-list)Logstash

Logstash是一个具有实时流水线功能的开源数据收集引擎。它是一个服务器端数据处理管道，可以同时从多个数据源中摄取数据，对其进行转换，然后将其发送到Elasticsearch之类的“存储”中。

Logstash有一组丰富的插件，允许它连接到各种源，并以多种方式处理数据。它可以解析和转换日志，将数据转换为结构化格式，或将其发送到另一个工具进行进一步处理。

凭借其快速处理大量数据的能力，Logstash是ELK堆栈(Elasticsearch、Logstash和Kibana)不可分割的一部分，通常用于集中、转换和监视日志数据。

**安装:**

Download logstash from here.

**使用:**

Full logstash documentation here.

Configuration examples here.

![image](https://image.3001.net/images/20230115/1673718751_63c2ebdf8b071623f119c.png!small "null")image

Image used from https://www.elastic.co/guide/en/logstash/current/logstash-modules.html

## 威胁工具和技术

Tools for identifying and implementing detections against TTPs used by threat actors.

### (#tool-list)lolbas-project.github.io

依赖于土地二进制文件(lolbin)是合法的Windows可执行文件，威胁行为者可以使用它们在不引起怀疑的情况下进行恶意活动。

使用lolbin可以使攻击者混入正常的系统活动并逃避检测，使其成为恶意行为者的流行选择。

LOLBAS项目是一个MITRE映射的lolbin列表，其中包含防御者的命令、用法和检测信息。

Visit https://lolbas-project.github.io/.

**使用:**

利用这些信息进行检测，以针对LOLBIN的使用加强基础设施。

这里有一些项目链接可以开始:

-   • Bitsadmin.exe
-   • Certutil.exe
-   • Cscript.exe

![image](https://image.3001.net/images/20230115/1673718758_63c2ebe6a61d8cb8dc2a6.png!small "null")image

Image used from https://lolbas-project.github.io/

### (#tool-list)gtfobins.github.io

GTFOBins (Get The F\* Out Binaries的缩写)是Unix二进制文件的集合，可用于提升特权、绕过限制或在系统上执行任意命令。

威胁行为者可以使用它们来获得对系统的未经授权的访问并进行恶意活动。

GTFOBins项目是一个Unix二进制文件列表，其中包含针对攻击者的命令和使用信息。此信息可用于实现unix检测。

Visit https://gtfobins.github.io/.

**使用:**

Here are some project links to get started:

-   • base64
-   • curl
-   • nano

![image](https://image.3001.net/images/20230115/1673718763_63c2ebeb724a0df38e453.png!small "null")image

Image used from https://gtfobins.github.io/

### (#tool-list)filesec.io

Filesec是一个文件扩展名列表，攻击者可以使用它进行网络钓鱼、执行、宏等。

这是一个很好的资源，用于了解常见文件扩展名的恶意用例以及可以防御它们的方法。

每个文件扩展名页面包含一个描述，相关的操作系统和建议。

Visit https://filesec.io/.

**使用:**

Here are some project links to get started:

-   • .Docm
-   • .Iso
-   • .Ppam

![image](https://image.3001.net/images/20230115/1673718766_63c2ebee4bcc8672faf0b.png!small "null")image

Image used from https://filesec.io/

### (#tool-list)KQL Search

KQL代表“Kusto查询语言”，它是一种用于搜索和过滤Azure Monitor日志中的数据的查询语言。它类似于SQL，但更适合日志分析和时间序列数据。

KQL查询语言对蓝队人员特别有用，因为它允许您快速轻松地搜索大量日志数据，以识别可能表明威胁的安全事件和异常。

KQL Search is a web app created by @ugurkocde that aggregates KQL queries that are shared on GitHub.

You can visit the site at https://www.kqlsearch.com/.

More information about Kusto Query Language (KQL) can be found here.

![image](https://image.3001.net/images/20230115/1673718768_63c2ebf038ffc299d4a7e.png!small "null")image

Image used from https://www.kqlsearch.com/

### (#tool-list)Unprotect Project

恶意软件作者花费大量的时间和精力来开发复杂的代码，以对目标系统执行恶意操作。对于恶意软件来说，保持不被发现并避免沙盒分析、反病毒或恶意软件分析师是至关重要的。

使用这种技术，恶意软件能够在雷达下通过，并在系统中不被发现。这个免费数据库的目标是集中关于恶意软件规避技术的信息。

该项目旨在为恶意软件分析师和防御者提供可操作的见解和检测能力，以缩短他们的响应时间。V

The project can be found at https://unprotect.it/.

The project has an API - Docs here.

![image](https://image.3001.net/images/20230115/1673718777_63c2ebf939c40503c0e04.png!small "null")image

Image used from https://unprotect.it/map/

## 威胁情报

用于收集和分析有关当前和新出现威胁的情报，以及生成关于潜在威胁的警报的工具

### (#tool-list)Maltego

Maltego是一个由Paterva开发的商业威胁情报和取证工具。安全专业人员使用它来收集和分析有关域、IP地址、网络和个人的信息，以识别可能不会立即明显的关系和连接。

Maltego使用可视化界面将数据表示为实体，这些实体可以链接在一起，形成一个关系网络。它包括一系列转换，这些转换是可用于从各种来源收集数据的脚本，如社交媒体、DNS记录和WHOIS数据。

Maltego通常与其他安全工具(如SIEMs和漏洞扫描器)一起使用，作为综合威胁情报和事件响应策略的一部分。.

You can schedule a demo here.

Maltego handbook Handbook for Cyber Threat Intelligence

![image](https://image.3001.net/images/20230115/1673718780_63c2ebfc64263b06ee6ec.png!small "null")image

Image used from https://www.maltego.com/reduce-your-cyber-security-risk-with-maltego/

### (#tool-list)MISP

MISP (Malware Information Sharing Platform，恶意软件信息共享平台)是一个开源平台，用于共享、存储和关联有针对性的攻击、威胁和恶意活动的危害指标(ioc)。

MISP包括一系列特性，例如IOCs的实时共享，支持多种格式，以及从其他工具导入和导出数据的能力。

它还提供了一个RESTful API和各种数据模型，以促进MISP与其他安全系统的集成。除了用作威胁情报平台外，MISP还用于事件响应、取证分析和恶意软件研究。

**安装:**

```
# Kaliwget -O /tmp/misp-kali.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh && bash /tmp/misp-kali.sh# Ubuntu 20.04.2.0-serverwget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.shbash /tmp/INSTALL.sh
```

Full installation instructions can be found here.

**使用:**

MISP documentation can be found here.

MISP user guide

MISP Training Cheat sheet

![image](https://image.3001.net/images/20230115/1673718784_63c2ec00998c06045083d.png!small "null")image

Image used from http://www.concordia-h2020.eu/blog-post/integration-of-misp-into-flowmon-ads/

### (#tool-list)ThreatConnect

ThreatConnect是一个威胁情报平台，可帮助组织聚合、分析威胁数据并对其采取行动。它旨在为组织的威胁状况提供单一、统一的视图，并使用户能够协作和共享有关威胁的信息。

该平台包括一系列用于收集、分析和传播威胁情报的功能，例如可定制的仪表板、与第三方数据源的集成以及创建自定义报告和警报的能力。

它旨在通过向组织提供识别、优先排序和响应潜在威胁所需的信息来帮助组织改进其安全态势。

You can request a demo from here.

ThreatConnect for Threat Intel Analysts - PDF

![image](https://image.3001.net/images/20230115/1673718791_63c2ec070f7521629b3a2.png!small "null")image

Image used from https://threatconnect.com/threat-intelligence-platform/

## 应急响应计划

创建和维护事件响应计划的工具，包括用于响应不同类型事件的模板和最佳实践

### (#tool-list)NIST

NIST网络安全框架(CSF)是由美国国家标准与技术研究院(NIST)开发的一个框架，旨在帮助组织管理网络安全风险。它为实施和维护强大的网络安全计划提供了一套指导方针、最佳实践和标准。

该框架围绕五个核心功能进行组织:识别、保护、检测、响应和恢复。这些功能为理解和解决网络安全风险的各个组成部分提供了一个结构。

CSF的设计是灵活的和可适应的，它可以被定制以适应组织的特定需求和目标。它旨在作为改善组织网络安全态势的工具，并帮助组织更好地理解和管理其网络安全风险。

**Useful Links:**

NIST Quickstart Guide

Framework for Improving Critical Infrastructure Cybersecurity

Data Breach Response: A Guide for Business

NIST Events and Presentations

Twitter - @NISTcyber

![image](https://image.3001.net/images/20230115/1673718811_63c2ec1bd617186d22965.png!small "null")image

Image used from https://www.dell.com/en-us/blog/strengthen-security-of-your-data-center-with-the-nist-cybersecurity-framework/

### (#tool-list)Incident Response Plan

事件响应计划是公司为管理和减轻安全事件(如数据泄露或网络攻击)影响而制定的一套程序。

事件响应计划背后的理论是，它可以帮助公司为安全事件做好准备并有效地做出响应，从而最大限度地减少损害，并降低未来再次发生此类事件的几率。

企业需要事件响应计划有以下几个原因:

1.  1. \*\*最大限度地减少安全事件的影响:\*\*事件响应计划有助于公司尽快识别和解决安全事件的来源，这有助于最大限度地减少损害并减少其蔓延的机会。
2.  2. \*\*为满足监管要求:\*\*许多行业的法规要求公司制定事件响应计划。例如，支付卡行业数据安全标准(PCI DSS)要求商户和其他接受信用卡的组织制定事件响应计划。

3.\*\*保护声誉:\*\*安全事件会损害公司的声誉，从而导致客户和收入的损失。事件响应计划可以帮助公司管理情况，并将对其声誉的损害降至最低。

1.  1. \*\*降低安全事件的成本:\*\*安全事件的成本可能很大，包括补救成本、法律费用和业务损失。事件响应计划可以通过提供响应事件的路线图来帮助公司将这些成本降至最低。

**Useful Links:**

National Cyber Security Centre - Incident Response overview

SANS - Security Policy Templates

SANS - Incident Handler's Handbook

FRSecure - Incident Response Plan Template

Cybersecurity and Infrastructure Security Agency - CYBER INCIDENT RESPONSE

FBI - Incident Response Policy

![image](https://image.3001.net/images/20230115/1673718816_63c2ec20725abf330e536.png!small "null")image

Image used from https://www.ncsc.gov.uk/collection/incident-management/incident-response

### (#tool-list)Ransomware Response Plan

勒索软件是一种对受害者的文件进行加密的恶意软件。然后，攻击者向受害者索要赎金，以恢复对文件的访问;因此得名勒索软件。

勒索软件响应计划背后的理论是，它可以帮助公司做好准备，并有效地应对勒索软件攻击，这可以最大限度地减少攻击的影响，并降低未来再次发生攻击的几率。

企业需要勒索软件响应计划的原因有几个:

1.  1. \*\*为了最大限度地减少勒索软件攻击的影响:\*\*勒索软件响应计划有助于公司尽快识别和解决勒索软件攻击，这有助于最大限度地减少损害，减少勒索软件传播到其他系统的机会。
2.  2. \*\*防止数据丢失:\*\*勒索软件攻击可能导致重要数据丢失，这对企业来说代价高昂且具有破坏性。勒索软件响应计划可以帮助公司从攻击中恢复，避免数据丢失。

3.\*\*保护声誉:\*\*勒索软件攻击可以损害公司的声誉，这可能导致客户和收入的损失。勒索软件响应计划可以帮助公司管理情况，并将对其声誉的损害降至最低。

1.  1. \*\*降低勒索软件攻击的成本:\*\*勒索软件攻击的成本可能很大，包括补救成本、法律费用和业务损失。勒索软件响应计划可以通过提供响应攻击的路线图来帮助公司将这些成本降至最低。

**有用的链接:**

National Cyber Security Centre - Mitigating malware and ransomware attacks

NIST - Ransomware Protection and Response

Cybersecurity and Infrastructure Security Agency - Ransomware Guide

Microsoft Security - Ransomware response

Blog - Creating a Ransomware Response Plan

![image](https://image.3001.net/images/20230115/1673718821_63c2ec252b44e7b720951.png!small "null")image

Image used from https://csrc.nist.gov/Projects/ransomware-protection-and-response

## 恶意软件检测和分析

Tools for detecting and analyzing malware, including antivirus software and forensic analysis tools.

### (#tool-list)VirusTotal

VirusTotal是一个基于云的网站和工具，可以分析和扫描文件、url和软件中的病毒、蠕虫和其他类型的恶意软件。

当文件、URL或软件提交给VirusTotal时，该工具会使用各种防病毒引擎和其他工具来扫描和分析恶意软件。然后，它提供一个包含分析结果的报告，可以帮助安全专业人员和蓝队识别和响应潜在威胁。

VirusTotal还可用于检查文件或URL的声誉，并监视网络上的恶意活动。

Visit https://www.virustotal.com/gui/home/search

**Usage:**

```
# Recently created documents with macros embedded, detected at least by 5 AVs(type:doc OR type: docx) tag:macros p:5+ generated:30d+# Excel files bundled with powershell scripts and uploaded to VT for the last 10days(type:xls OR type:xlsx) tag:powershell fs:10d+# Follina-like exploit payloadsentity:file magic:"HTML document text" tag:powershell have:itw_url# URLs related to specified parent domain/subdomain with a specific header inthe responseentity:url header_value:"Apache/2.4.41 (Ubuntu)" parent_domain:domain.org# Suspicious URLs with a specific HTML titleentity:url ( title:"XY Company" or title:"X.Y. Company" or title:"XYCompany" ) p:5+
```

Full documentation can be found here.

VT INTELLIGENCE CHEAT SHEET

![image](https://image.3001.net/images/20230115/1673718823_63c2ec27cd4b7bd139a3c.png!small "null")image

Image used from https://www.virustotal.com/gui/home/search

### (#tool-list)IDA

交互式反汇编器(Interactive Disassembler)是一个强大的工具，用于反向工程和分析已编译和可执行代码。

它可以用来检查软件的内部工作，包括恶意软件，并了解它是如何工作的。IDA允许用户反汇编代码，将其反编译为高级编程语言，并查看和编辑生成的源代码。这对于识别漏洞、分析恶意软件和理解程序是如何工作的很有用。

IDA还可以用于生成图形和图表，以可视化代码的结构和流程，这可以使其更容易理解和分析。

**安装:**

Download IDA from here.

**使用:**

IDA Practical Cheatsheet

IDAPython cheatsheet

IDA Pro Cheatsheet

![image](https://image.3001.net/images/20230115/1673718833_63c2ec319a0819e9a1a51.png!small "null")image

Image used from https://www.newton.com.tw/wiki/IDA%20Pro

### (#tool-list)Ghidra

Ghidra是由美国国家安全局(NSA)开发的免费开源软件逆向工程工具。它用于分析已编译和可执行代码，包括恶意软件。

Ghidra允许用户反汇编代码，将其反编译为高级编程语言，并查看和编辑生成的源代码。这对于识别漏洞、分析恶意软件和理解程序是如何工作的很有用。

Ghidra还包括一系列支持SRE任务的特性和工具，例如调试、代码图形化和数据可视化。Ghidra是用Java编写的，可用于Windows、MacOS和Linux。

**安装:**

1.  1. Download the latest release from here.
2.  2. Extract the zip

Full installation and error fix information can be found here.

**使用:**

1.  1. Navigate to the unzipped folder

```
# WindowsghidraRun.bat# Linux./ghidraRun
```

If Ghidra failed to launch, see the Troubleshooting link.

![image](https://image.3001.net/images/20230115/1673718843_63c2ec3b92b13c969434d.png!small "null")image

Image used from https://www.malwaretech.com/2019/03/video-first-look-at-ghidra-nsa-reverse-engineering-tool.html

## 数据恢复

从损坏或损坏的系统和设备中恢复数据的工具

### (#tool-list)Recuva

Recuva是一个数据恢复工具，可以用来恢复从您的计算机删除的文件。

它通常用于恢复可能包含有价值信息的已删除文件，例如可用于调查安全事件的已删除日志或文档。

Recuva可以从硬盘、USB驱动器和内存卡中恢复文件，适用于Windows和Mac操作系统。

**安装:**

You can download the tool from here.

**使用:**

Nice step by step guide.

![image](https://image.3001.net/images/20230115/1673718864_63c2ec50e7e1c032ef1c4.png!small "null")image

Image used from https://www.softpedia.com/blog/recuva-explained-usage-video-and-download-503681.shtml

### (#tool-list)Extundelete

Extundelete是一个实用工具，可用于从ext3或ext4文件系统中恢复已删除的文件。

它的工作原理是在文件系统中搜索曾经属于某个文件的数据块，然后尝试使用这些数据块重新创建该文件。它通常用于恢复被意外或恶意删除的重要文件。

**安装:**

You can download the tool from here.

**使用:**

```
# Prints information about the filesystem from the superblock.--superblock# Attemps to restore the file which was deleted at the given filename, called as "--restore-file dirname/filename".--restore-file path/to/deleted/file# Restores all files possible to undelete to their names before deletion, when possible. Other files are restored to a filename like "file.NNNN".--restore-all
```

Full usage information can be found here.

![image](https://image.3001.net/images/20230115/1673718883_63c2ec631dcf458fdb7ba.png!small "null")image

Image used from https://theevilbit.blogspot.com/2013/01/backtrack-forensics-ext34-file-recovery.html

### (#tool-list)TestDisk

TestDisk是一个免费的开源数据恢复软件工具，旨在帮助恢复丢失的分区，并使非引导磁盘重新启动。它对计算机取证和数据恢复都很有用。

它可用于恢复由于各种原因(如意外删除、格式化或分区表损坏)而丢失的数据。

TestDisk还可以用于修复损坏的引导扇区，恢复删除的分区，以及恢复丢失的文件。它支持广泛的文件系统，包括FAT、NTFS和ext2/3/4，并可用于从损坏或格式化的磁盘恢复数据，这些磁盘使用的文件系统与创建它们时使用的文件系统不同。

**安装:**

You can download the tool from here.

**使用:**

Full usage examples here.

Step by step guide

TestDisk Documentation PDF - 60 Pages

![image](https://image.3001.net/images/20230115/1673718892_63c2ec6caee271a202a2c.png!small "null")image

Image used from https://www.cgsecurity.org/wiki/

## 数字取证

对数字设备和系统进行法医调查的工具，包括收集和分析证据的工具

### (#tool-list)SANS SIFT

SANS SIFT (SANS Investigative Forensic Toolkit)是一个用于法医分析和事件响应的强大工具包。

它是一个开源和商业工具的集合，可用于在广泛的系统上执行取证分析，包括Windows、Linux和Mac OS x。SANS SIFT工具包设计用于在取证工作站上运行，这是一个专门的计算机，用于对数字证据执行取证分析。

SANS SIFT工具包对蓝队人员特别有用，因为它提供了广泛的工具和资源，可用于调查事件，响应威胁，并对受损系统进行取证分析。

**安装:**

1.  1. 访问\[https://www.sans.org/tools/sift-workstation/\] (https://www.sans.org/tools/sift-workstation/)。
2.  2. 点击“登录下载”按钮并输入(或创建)您的SANS Portal帐户凭据来下载虚拟机。

3.启动虚拟机后，使用下面的凭据来获得访问权限。

```
Login = sansforensicsPassword = forensics
```

**笔记:**Use to elevate privileges to root while mounting disk images.

Additional install options here.

**使用:**

```
# Registry Parsing - Regripperrip.pl -r <HIVEFILE> -f <HIVETYPE># Recover deleted registry keysdeleted.pl <HIVEFILE># Mount E01 Imagesewfmount image.E01 mountpointmount -o# Stream Extractionbulk_extractor <options> -o output_dir
```

Full usage guide here.

![image](https://image.3001.net/images/20230115/1673718944_63c2eca0e4512e9d7bd1d.png!small "null")image

Image used from https://securityboulevard.com/2020/08/how-to-install-sift-workstation-and-remnux-on-the-same-system-for-forensics-and-malware-analysis/

### (#tool-list)The Sleuth Kit

Sleuth Kit是一组命令行工具，可用于分析磁盘映像并从中恢复文件。

它主要用于法医调查人员在计算机被扣押或磁盘图像被制作后检查数字证据。它很有用，因为它可以帮助了解安全事件期间发生了什么，并识别任何恶意活动。

侦探工具包中的工具可用于提取已删除的文件，分析磁盘分区结构，并检查文件系统以寻找篡改或异常活动的证据。

**安装:**

Download tool from here.

**使用:**

Link to documentation.

![image](https://image.3001.net/images/20230115/1673718965_63c2ecb588e613c46c7b3.png!small "null")image

Image used from http://www.effecthacking.com/2016/09/the-sleuth-kit-digital-forensic-tool.html

### (#tool-list)Autopsy

尸检是一个数字取证平台和图形界面的侦探套件和其他数字取证工具。

它被执法部门、军队和公司检查人员用来调查计算机上发生的事情。您可以使用它来分析磁盘映像和恢复文件，以及识别系统和用户活动。

尸检被“蓝队”(保护组织免受攻击的网络安全专业人员)用于进行法医分析和事件响应。它可以帮助蓝队了解攻击的性质和范围，并识别可能发生在计算机或网络上的任何恶意活动。

**安装:**

Download the tool from here.

**使用:**

Autopsy User Guide

SANS - Introduction to using the AUTOPSY Forensic Browser

![image](https://image.3001.net/images/20230115/1673718971_63c2ecbbdc9ace5126504.png!small "null")image

Image used from https://www.kitploit.com/2014/01/autopsy-digital-investigation-analysis.html

## 网络安全意识培训

Tools for training employees and other users on how to recognize and prevent potential security threats.

### (#tool-list)TryHackMe

TryHackMe是一个提供各种虚拟机(被称为“房间”)的平台，旨在通过实际学习来教授网络安全概念和技能。

这些房间是交互式和游戏化的，允许用户通过解决挑战和完成任务来了解诸如web漏洞、网络安全和密码学等主题。

该平台通常用于安全意识培训，因为它为用户提供了一个安全和可控的环境来练习技能，并学习不同类型的网络威胁以及如何防御它们。

Visit https://tryhackme.com/ and create an account.

TryHackMe - Getting Started Guide

**Useful links:**

Pre-Security Learning Path

introduction to Cyber Security Learning Path

Visit the hacktivities tab for a full list of available rooms and modules.

![image](https://image.3001.net/images/20230115/1673719002_63c2ecda53af264bcde7e.png!small "null")image

Image used from https://www.hostingadvice.com/blog/learn-cybersecurity-with-tryhackme/

### (#tool-list)HackTheBox

HackTheBox是一个练习和提高黑客技能的平台。

它由一系列模拟现实场景的挑战组成，要求你使用各种黑客技术的知识来解决它们。这些挑战旨在测试您对网络安全、密码学、web安全等主题的知识。

HackTheBox经常被安全专业人士用作练习和提高技能的一种方式，它也可以作为安全意识培训的有用资源。通过应对挑战并学习如何解决它们，个人可以更好地了解如何识别和减轻常见的安全威胁。

Visit https://app.hackthebox.com/login and create an account.

**Useful links:**

Blog - Introduction to Hack The Box

Blog - Learn to Hack with Hack The Box: The Beginner's Bible

Blog - Introduction to Starting Point

![image](https://image.3001.net/images/20230115/1673719013_63c2ece5d25713c8a6e09.png!small "null")image

Image used from https://www.hackthebox.com/login

### (#tool-list)PhishMe

PhishMe是一家提供安全意识培训的公司，帮助组织教育其员工如何识别和预防网络钓鱼攻击。

PhishMe的培训项目旨在教员工如何识别和报告网络钓鱼企图，以及如何保护他们的个人和专业账户免受这类攻击。

该公司的培训计划可以定制，以适应不同组织的需求，并可以通过各种媒介提供，包括在线课程、现场培训和模拟。

Request a demo from here.

**有用的链接:**

Cofense Blog

Cofense Knowledge Center

![image](https://image.3001.net/images/20230115/1673719016_63c2ece847181da2228cb.png!small "null")image

Image used from https://cofense.com/product-services/phishme/

## 交流和合作

Tools for coordinating and communicating with team members during an incident, including chat, email, and project management software.

### (#tool-list)Twitter

Twitter是一个分享网络安全信息的好平台。

这是一个被安全专业人员、研究人员和专家广泛使用的平台，可以让您访问无穷无尽的新信息。

以下是值得关注的优秀账号:

-   • @vxunderground
-   • @Alh4zr3d
-   • @3xp0rtblog
-   • @C5pider
-   • @\_JohnHammond
-   • @mrd0x
-   • @TheHackersNews
-   • @pancak3stack
-   • @GossiTheDog
-   • @briankrebs
-   • @SwiftOnSecurity
-   • @schneierblog
-   • @mikko
-   • @campuscodi

### (#tool-list)Facebook TheatExchange

Facebook ThreatExchange是一个供安全专业人员分享和分析网络威胁信息的平台。

它的目的是通过允许组织以私密和安全的方式相互共享威胁情报，帮助组织更好地防御威胁。

它旨在供“蓝队”使用，这些“蓝队”负责组织的安全，并致力于预防、检测和响应网络威胁。

**使用:**

To request access to ThreatExchange, you have to submit an application via https://developers.facebook.com/products/threat-exchange/.

**有用的链接:**

Welcome to ThreatExchange!

ThreatExchange UI Overview

ThreatExchange API Reference

GitHub - ThreatExchange

原文链接：https://github.com/A-poc/BlueTeam-Tools/blob/main/README.md