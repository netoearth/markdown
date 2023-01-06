![](https://p2.ssl.qhimg.com/t0149b9210315f40d58.png)

## Serverless简介

Serverless（又称为无服务器）架构是一种全新的云计算模式，它是在容器技术和当前服务模式基础之上发展起来的，它更多的是强调后端服务与函数服务相结合，使开发者无需关注后端服务具体实现，而更侧重关注自己业务逻辑代码的实现。

随着云原生技术的不断发展，应用部署模式已逐渐趋向于“业务逻辑实现与基础设施分离”的设计原则。Serverless架构完美诠释了这种新型的应用部署模式和设计原则。从云原生整体发展路线来看，Serverless模式更趋近于云原生最终的发展方向。

## Serverless实现方式：BaaS和FaaS

云计算发展历程从IaaS(Infrastructure as a Service，基础设施即服务)到PaaS(Platform as a Service，平台即服务)，再到SaaS(Software-as-a-Service，软件即服务)，逐渐的将去服务器化的趋势表现的愈发明显，而SaaS的下一个阶段可能就是

BaaS+FaaS+Others，即Serverless。

提到Serverless，我们首先得解下什么是BaaS和FaaS。

BaaS(Backend as a Service，后端即服务)和FaaS(Functions as a Service，函数即服务)是Serverless两种主要的实现方式。BaaS可以理解为是一个整合和开放各种在应用开发中需要的服务能力的平台，它通过创建大量重复的代码功能，方便应用基于服务的快速开发和构建。FaaS是Serverless主要的实现方式，开发者通过编写一段逻辑代码来定义函数调用方式，当事件触发时函数被调用执行。

FaaS本质上是一种事件驱动并由消息触发的服务，事件类型可以是一个HTTP请求，也可以是一次用户操作，函数可以看作是完成某个功能或任务的代码片段。相比传统应用运行模式，Serverless业务代码被拆分成了函数粒度，不同函数表示不同的功能，函数之间调用关系也更加复杂。

![](https://p0.ssl.qhimg.com/t01731b891d85419a17.png)

云计算发展历程与FaaS所处的地位

## Serverless应用架构

通常Serverless应用都是基于无服务器应用框架Serverless Framework构建完成的。开发者无需关心底层资源，即可快速部署完整可用的Serverless应用架构，同时Serverless平台具有资源编排、自动伸缩、事件驱动等能力，覆盖编码、调试、测试、部署等全生命周期，帮助开发者联动各类云组件资源，迅速构建完整的 Serverless 应用。

目前常见的云厂商Serverless应用服务支持各类 Web 框架快速创建、迁移上云，可以实现Express、Next.js、Python Flask、PHP Laravel、Koa、Egg.js、Nuxt.js等框架应用的快速部署。开发者无需进行复杂的配置，通过Web Fuction即可快速搭建各类场景下的Serverless应用，轻松实现云函数、API网关、COS、DB 等资源的创建、配置和部署。Serverless提供从初始化、编码、调试、资源配置和部署发布，到业务监控告警、实时日志、故障排查的一站式解决方案。

从架构层面来看，Serverless由BaaS和FaaS共同构成了完整的应用架构。Serverless计算平台可以帮助开发者完成构建服务运行环境，开发者无需购买服务器，云厂商负责提供并维护基础设施资源和后端服务组件。Serverless架构具有自动扩缩容的特性，当应用部署后云计算平台会为Serverless应用提供足够的资源来支持应用稳定运行。

![](https://p5.ssl.qhimg.com/t011d74669f04e2db8e.png)

Serverless组件架构图

无服务器云函数(Serverless Cloud Function，SCF)是一种为企业和开发者们提供的无服务器执行环境。开发者通过Web IDE或者本地IDE编写代码，然后将代码和所需依赖一起打包部署到云函数平台。开发者会在业务代码中会提前定义好函数具体调用方式，如访问数据库、对象存储、第三方服务等接口，用户通过提前定义好的请求方式去访问对应的服务，平台会根据用户请求去拉起相应的计算资源运行业务代码。

![](https://p2.ssl.qhimg.com/t019f10adafeca5cb32.png)

Serverless运行原理图

## 安全风险共担模型

由于Serverless后端基础设施和服务主要是由云厂商负责的，但是Serverless应用本身是面向企业和开发者的，Serverless应用允许开发者上传自己的代码到服务端，同时允许开发者对应用配置做修改，若开发者上传的代码中包含漏洞或应用配置错误，将导致应用面临风险，因此Serverless安全问题相对复杂。

Serverless的安全风险责任划分可以参照Serverless安全风险共担模型。对于云厂商来说，首先需要保障云基础环境安全，包括所有的底层基础设施和后端服务软件的安全性。同时云厂商也担负着Serverless平台应用整体安全防护责任，借助API网关、云防护等优势来保障Serverless应用安全。对于用户来说，需要保证上传到服务端的代码是安全的、同时确保应用策略配置安全，避免代码中存在漏洞或策略配置不当导致安全风险。所以从本质上来说，Serverless的安全性是需要云厂商和用户双方共同承担的。

![](https://p2.ssl.qhimg.com/t0143d49ced8e80d8a1.png)

Serverless安全风险共担模型

## Serverless安全风险

Serverless一般的攻击流程：攻击者通过应用程序漏洞或者组件漏洞实现初始访问权限，当获取到服务器权限后，攻击者会尝试查找并窃取用户凭据或服务凭据，然后利用可用凭证进一步横向攻击其他云服务。整个攻击过程包含但不限于以下攻击方式：

![](https://p3.ssl.qhimg.com/t017daafdce2b7ab255.png)

腾讯安全云鼎实验室根据自身安全实践，结合国内外众多相关案例，总结出了Serverless常见风险项，用以帮助开发以及运维人员识别各类风险，从而有效的保障Serverless应用安全。主要安全风险可参照下图：

![](https://p2.ssl.qhimg.com/t016cca3644c0628cbf.png)

Serverless安全风险

**1.应用程序漏洞**

对于Serverless应用而言，SQL注入、命令注入、XSS等传统漏洞风险在Serverless应用中同样存在。若Serverless应用在实现过程中没有对外部输入进行严格校验（包括用户输入和各应用间交互）则可能导致注入风险的发生。传统应用的注入防护一般是对用户输入进行过滤和限制，但是Serverless应用内部网络相比传统网络更加复杂，事件输入可能来自任何云服务（如服务器、云存储、电子邮件、消息服务等），因此仅仅依靠编写安全的代码和依赖传统WAF防护并不能完全杜绝注入风险的发生。

示例: 文件上传处未过滤用户输入导致命令执行漏洞。由于开发者在编写代码时使用了命令拼接的方式来构造路径，但是没有对文件名进行严格过滤，导致攻击者可以控制传入的内容，导致命令执行漏洞的产生。

![](https://p2.ssl.qhimg.com/t01f184ecc4b583eadc.png)

![](https://p1.ssl.qhimg.com/t01c386c3f82e0e28a7.png)

**2.拒绝钱包攻击**

DoW（Denial of Wallet,拒绝钱包攻击）是针对云平台账户的DoS方式，其目的是通过高并发来耗尽用户账户可用余额。DoW攻击与传统拒绝服务(DoS)攻击方式类似，恶意攻击者向通过构造大量并发请求来触发函数调用，由于Serverless是按照资源使用量和函数调用次数来收费的，当有大量调用产生时，服务会自动扩展，这将导致用户账户可用余额快速被消耗。DoW攻击的一个典型的场景是在用户订阅web程序上生成大量虚假用户，通过大量用户访问API端点造成大量资源消耗导致账户金额耗尽。

**3.资源滥用风险**

近年来，随着Severless的快速发展，各种服务滥用现象也相继出现，主要体现在云函数滥用和Serverless基础设施滥用。云函数常被恶意人员用作构建代理池、隐藏C2和构建webshell，恶意人员通过构建此类应用来达到隐藏其客户端真实IP的目的。Serverless应用也常被用于构建扫描、钓鱼等攻击平台，攻击者通过构建此类应用来实现对外部系统的扫描探测、钓鱼窃取用户数据等目的。这些滥用行为导致云基础设施资源被恶意利用和消耗，给云服务的正常使用和监测带来了极大的困扰。

示例: 使用Serverless构建扫描应用导致服务滥用。恶意攻击人员通过上传代码构建扫描应用，通过本地调用云函数资源来实现动态IP扫描的目的。

![](https://p1.ssl.qhimg.com/t014d162eafd9d18a6b.png)

**4.第三方API和组件不安全接入**

由于Serverless服务一般会接入多个云服务组件，包括云API、API网关、事件触发器等。若云服务或组件在接入Serverless服务时未对组件身份或接收数据进行校验，则可能导致安全风险的发生。接入数据源的增加会导致攻击面扩大，传统应用只能从单一服务器上获取敏感数据，而Serverless应用通常会接入大量数据源，若攻击者针对各数据源进行攻击，则可能获取到大量敏感数据。

**5.供应链攻击风险**

近年来，供应链安全事件时有发生，一些严重的漏洞可能来自于开源库和框架，nodejs生态系统的一项统计表明通常我们部署的97%的代码来自于开源库和开源组件，所以供应链攻击对Serverless来说是一项非常重要的安全风险。

**6.运行时安全风险**

若攻击者已经通过某种方式获得了Serverless服务器权限后，则可能通过替换引导程序的方式攻击Serverless应用服务，从而导致Serverless应用实例被接管。如果攻击者可以修改ServerlessACL策略，则可以通过修改函数超时时间，增加服务冷启动时间，或者通过Serverless预热插件增加运行时时长等方式实现持久化控制。

**7.配置不当导致权限滥用**

当Serverless存在不安全的IAM配置时，将导致可执行恶意操作以及进行云平台身份越权。Serverless构建的应用程序通常包含数十或数百个函数，每个函数都有自己特定的功能和用途，这些功能被编织在一起并被编排以形成整个系统逻辑。一些函数可能会公开公共的WEB API接口，因此需要强大的身份验证方案为相关功能、事件触发提供访问控制保护。当创建IAM策略时，如果没有遵循最小权限原则，则可能导致分配给函数的IAM角色过于宽松，攻击者可能会利用函数中的漏洞横向移动到云账户中的其它资源。

**8.日志和监控不足**

传统的应用已经有比较成熟的日志管理和分析工具，而Serverless函数级别的日志分析工具还未被广泛采用。由于Serverless应用函数数量多、生命周期短，各函数间调用关系复杂，因此任何一个点都可能成为攻击突破口。对于云厂商来说，需要具备完备的安全防护和应用监测体系，才能更好的保障Serverless应用和服务的安全。

**9.云环境网络攻击风险**

攻击者可利用漏洞或错误的云平台架构配置，从公有云环境横向移动到云管理内部网络(如公有云到IDC网络)；或者从公有云网络横向移动到客户构建的网络环境中，导致严重的攻击事件产生。

**10.云特性攻击风险**

利用云上服务的特性，也可展开更多的攻击，例如通过Serverless服务窃取到云服务凭据或角色临时访问凭据等信息后，可利用这些凭据获取对应服务的相应控制权限；又或是利用容器逃逸漏洞进行更深入攻击操作等。

**11.云资源消耗攻击风险**

挖矿攻击作为一种较为常见的攻击形式，其威胁在云环境中也同样存在，Serverless服务同样存在云资源消耗攻击风险，攻击者攻击Serverless服务用以进行挖矿操作，消耗客户的资源和资金，给客户造成影响。攻击者瞄准云上脆弱资产，将挖矿木马植入云资源中进行挖矿，这将导致客户资源受到较大威胁。

**12.密钥存储风险**

Serverless服务中同样存在密钥以及凭据的存储安全风险，攻击者可以利用漏洞，在Serverless服环境变量中获取凭证、或是在代码中查找明文编写的密钥。

**13.后门持久化风险**

利用Serverless服务的弹性原则，与传统攻击所部署的后门相比，攻击者可以在Serverless服务中构建更加隐蔽的后门。通过利用这些后门，可以达到持久化攻击的效果。

## Serverless防护措施

我们根据大量数据和实际案例，结合Serverless各类风险总结出了Serverless风险防护措施。通过了解这些安全防护手段，可以帮助开发以及运维人员识别并消除各类风险，从而更好的保障云上资产安全。主要的安全防护措施总结如下：

![](https://p1.ssl.qhimg.com/t01753d2eeed6c2b34f.png)

Serverless安全防护

**1.使用安全漏洞缓解措施**

Serverless的安全性依靠用户和云厂商共同来保障，对于开发人员来说，最基础的要求开发人员编写代码时遵循安全开发原则，保证业务代码本身不存在安全漏洞；其次需要保证Serverless应用配置安全，避免因为配置不当导致不安全风险的发生。对于云厂商来说，需要保证Serverless应用与其他云服务组件的接口调用安全，对于重要的功能需要在功能模块之间放置防火墙做好隔离，当基础应用存在注入等问题时，防火墙会起到一定缓解作用。其次由于Serverless通常接入应用组件和数据较多，因此需要使用https/tls来保障数据在传输过程中的安全性，同时使用KMS（Key Management Service，密钥管理系统）来保障服务运行时的密钥使用安全，避免将密钥等敏感数据硬编码或写入环境变量中。

**2.Dos攻击缓解与防护**

开发者通过编写高效的Serverless函数来执行离散的目标任务，为Serverless功能执行设置适当的超时时间和磁盘使用限制，通过对API调用设置请求限制，对Serverless功能实施适当的访问控制等来缓解Dos攻击风险。同时使用不易受到ReDos等应用层Dos攻击的API、模块和库来避免Dos问题的产生。

**3.Serverless滥用防护**

针对Serverless滥用问题，需要从Serverless应用本身做限制，如限制某些库和方法的使用，通过有效监控和阻断来提高Serverless服务滥用的门槛，完善异常事件发现和监测机制，当发现滥用行为时及时触发告警和阻断滥用行为。

**4.第三方依赖库防护**

由于Serverless依赖于第三方组件和库构建应用程序和运行环境，因此第三方依赖库的安全性直接影响到Serverless应用和平台的安全。构建完善的第三方依赖库防护和监测机制对保障Serverless应用安全起到极大的意义。

**5.IAM访问控制防护**

在Serverless中，运行的最小单元通常为一个个函数，Serverless中最小特权原则通过事先定义一组具有访问权限的角色，并赋予函数不同的角色，从而可以实现函数层面的访问控制，避免统一的权限分配导致的各类安全风险。

**6.Serverless平台防护**

对于云厂商而言，要避免使用过时的函数和云资源，重复利用资源虽然有助于节约成本，但是会导致Serverless攻击面增加，因此必须定期清理服务器环境，删除未使用的角色，身份和依赖项等。其次要避免重用执行环境，对于云厂商而言，在两次调用之间保留执行环境，可以提高调用效率，但是当执行环境被保留下来时，部分敏感数据可能会被保留下来，这将导致一定的安全风险。

**7.完善安全监控和日志记录**

由于函数的生命周期极短，并且随着扩展部署越来越多的函数，函数调用数量不断增加，各函数功能之间存在复杂的关联性，攻击可能从任何一个点发起。因此需要建立完善监控机制，例如使用函数级别的日志分析工具来提高监控能力，及时发现攻击行为。

腾讯云Serverless应用服务目前已经具备完善的安全防护体系。以腾讯云云函数(SCF)为例，在密钥安全管理上，腾讯云使用了密钥管理系统（Key Management Service，KMS）。KMS使用经过第三方认证的硬件安全模块 HSM（Hardware Security Module）来生成和保护密钥，实现密钥全生命周期管理和保障数据安全能力。同时云函数也完善了配套的监控和告警机制，提供如调用次数、内存使用、并发使用、超时、代码错误等多维度的监控和告警能力，帮助运维人员轻松实现应用后期维护。对于基础设施、资源管理、安全管控、容灾等能力是云函数平台必备的基础能力，也是云平台的核心能力。

![](https://p2.ssl.qhimg.com/t01778f7bfad6149930.png)

密钥管理系统（KMS）产品架构图

**云安全攻防矩阵V3.0发布**

腾讯安全云鼎实验室根据自身安全实践，结合国内外相关案例，对云上主流应用安全风险进行抽象，绘制出了云安全攻防矩阵。企业和开发者可参照该矩阵了解云服务攻击手法，帮助开发以及运维人员识别各类风险。我们的云安全攻防矩阵V3.0已经发布了，此次新增了Serverless安全矩阵模块，目前3.0版本已经涵盖云服务器、容器、cos存储桶、Serverless等主流云服务。也欢迎大家积极踊跃与我们沟通交流，一起为维护云安全贡献力量。矩阵详情可查看云鼎实验室官网：

https://cloudsec.tencent.com/home/

## 写在后面

Serverless作为一种新的技术和架构模式，虽然起步较晚但发展迅猛，短短几年时间已经推出多款备受开发者追捧的应用，吸引了大批开发者的关注。作为一个相对比较新鲜的事物，Serverless还有很多领域和价值值得我们去探索。随着容器技术、IoT、5G、区块链等新兴技术的发展，技术上也逐渐趋向于中心化、轻量虚拟化、细粒度计算等，而Serverless也将借势发展，相信不久的将来Serverless必将在云计算的舞台上大放异彩。

## 参考链接

http://www.ccopsa.cn/QiantaiZiyuanXiazaiWendang/WenjianXiazai?key=1166d80a-da24-4a59-8fdc-9d15c01ca9af

https://www.youtube.com/watch?v=ILJozDEQ-aw

[https://mp.weixin.qq.com/s/kNawzZowQt8hwiE5Z8wIQQ](https://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247488567&idx=1&sn=6bd22d362c85cc801b7b3588ceca3635&scene=21#wechat_redirect)

[https://mp.weixin.qq.com/s/rbS0\_42RBiFu8UFFQW4kew](https://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247488798&idx=1&sn=485e2131f347ff4d8c3b5b3286b36c97&scene=21#wechat_redirect)

[https://mp.weixin.qq.com/s/dzvQQNFGBTfF7TvowaowFA](https://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247489053&idx=1&sn=4b1e59880a6b0afa61a89e614856bf28&scene=21#wechat_redirect)

[https://mp.weixin.qq.com/s/0Lq7hX2WdC96rVQgkBXunA](https://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247488971&idx=1&sn=52de9e039212a612f40564d0b50dc103&scene=21#wechat_redirect)

[https://mp.weixin.qq.com/s/duF1Z0EDC3n\_G378Aq\_XYA](https://mp.weixin.qq.com/s?__biz=MzIyODYzNTU2OA==&mid=2247488901&idx=1&sn=4d7bdb1ddf015cb77ea4e8bc0978712f&scene=21#wechat_redirect)

https://owasp.org/www-project-serverless-top-10/

https://github.com/neargle/my-re0-k8s-security

https://snyk.io/blog/10-serverless-security-best-practices/

https://project-awesome.org/pmuens/awesome-serverless

https://toutiao.io/posts/jx6yyi3/preview

https://github.com/puresec/sas-top-10

https://www.youtube.com/watch?v=ILJozDEQ-aw

https://www.youtube.com/watch?v=tlZ2PIXTHxc

[https://mp.weixin.qq.com/s/XuAlWNhrGvRrge4JdEZ62A](https://mp.weixin.qq.com/s?__biz=MzI4NDY2MDMwMw==&mid=2247502290&idx=1&sn=67537300fc0d9f76477ccd23a8c4f769&scene=21#wechat_redirect)

https://www.sciencedirect.com/science/article/pii/S221421262100079X

https://unit42.paloaltonetworks.com/gaining-persistency-vulnerable-lambdas/