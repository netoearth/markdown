©著作权归作者所有：来自51CTO博客作者一口Linux的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[最强内网穿透工具frp](https://blog.51cto.com/u_15169172/2952058)  
[https://blog.51cto.com/u\_15169172/2952058](https://blog.51cto.com/u_15169172/2952058)

什么是frp

frp 是一个高性能的反向代理应用，可以帮助您轻松地进行内网穿透，对外网提供服务，支持 tcp, http, https 等协议类型，并且 web 服务支持根据域名进行路由转发。frp 采用 C/S 模式，将服务端部署在具有公网 IP 机器上，客户端部署在内网或防火墙内的机器上，通过访问暴露在服务器上的端口，反向代理到处于内网的服务。 在此基础上，frp 支持 TCP, UDP, HTTP, HTTPS 等多种协议，提供了加密、压缩，身份认证，代理限速，负载均衡等众多能力。

下图是frp官网：https://gofrp.org/

![最强内网穿透工具frp_frp](https://s7.51cto.com/images/blog/202106/28/f94c3628b4b2c2d3f14f158b8097fc23.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

GitHub地址：  
https://github.com/fatedier/frp，看下图就知道认可度有多高吧。

![最强内网穿透工具frp_frp_02](https://s8.51cto.com/images/blog/202106/28/423760c70a0edaf89a6f9019877e4f41.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

安装frp

由于frp是go语言开发，因此可以直接下载可执行程序，没有任何依赖。一般通过GitHub的releases下载：  
https://github.com/fatedier/frp/releases，我一般用下图这两个版本。

![最强内网穿透工具frp_穿透工具_03](https://s3.51cto.com/images/blog/202106/28/9be213f78abadda1611bfe0cf052cbc2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

Linux用通用解压命令：tar -xf  
frp\_0.37.0\_linux\_amd64.tar.gz

window直接用工具进行解压吧。最终得到frps、frpc这两个可执行程序，以及示例配置文件。

配置frps

按照如下配置frps.ini文件，在我的公网Linux服务器上执行frps -c frps.ini就可以成功运行。当然也可以使用systemd\\frps.service将frps加入系统自启之类的，不过我还是喜欢用supervisor工具管理这些程序。

```
[common]
# 与客户端建立连接端口
bind_port = 7000
# 与客户端校验的token
token = my_token
# 提供web界面端口
dashboard_port = 7500
# web界面登录用户名
dashboard_user = admin
# web界面登录密码
dashboard_pwd = password1.2.3.4.5.6.7.8.9.10.11.
```

配置frpc

按照如下配置在我的window电脑上的frpc.ini文件，值得注意的是remote\_port实际是公网Linux服务器那边监听的端口，也就是说frpc通过server\_port连接到frps上，通知frps启用remote\_port来穿透到内网。运行客户端程序：frpc -c frpc.ini

```
[common]
# 连接frps的IP地址
server_addr = 192.168.1.100
# 连接frps的端口
server_port = 7000
# 和frps服务端校验的token
token = my_token

# 这里取名随意,一般有意义就行
[stream]
# 看官网有这些类型: TCP,UDP,HTTP,HTTPS,STCP,SUDP
# 去这里开始看实例: https://gofrp.org/docs/examples/ssh/
type = tcp
# 本地访问的IP地址
local_ip = 127.0.0.1
# 本地访问的端口
local_port = 8800
# 在frps服务器上对应的端口
remote_port = 66001.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.
```

访问frps的web服务

根据frps的dashboard\_port端口访问web服务（  
http://192.168.1.100:7500/），可以看到如下的tcp里面有一条内网穿透的服务可以使用。可以看到左侧的tab页，可以支持好几种内网穿透的协议，不过我一般只用到tcp协议，例如访问内网ssh后台，用起来杠杠的。

![最强内网穿透工具frp_frp_04](https://s4.51cto.com/images/blog/202106/28/f89de67e522d38defb60e90a397d1535.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

内网启动tcp服务

为了方便演示，编写了内网的http服务，在/test的方法里面，打印了远程地址，以及http的get请求的一个参数。

```
package main

import (
"fmt"
"net/http"
)

func main() {
http.HandleFunc("/test", func(w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "RemoteAddr:%s\n", r.RemoteAddr)
fmt.Fprintf(w, "data:%s\n", r.FormValue("data"))
})
http.ListenAndServe("127.0.0.1:8800", nil)
}1.2.3.4.5.6.7.8.9.10.11.12.13.14.
```

测试结果

如下图所示当访问服务器的6600端口，实际穿透到访问内网127.0.0.1:8800的服务器。我配置的是tcp协议，因此只要内网的服务是基于tcp协议的，那么就可以对6600端口使用相应的工具去访问。官网示例是访问内网ssh服务：  
https://gofrp.org/docs/examples/ssh/。

![最强内网穿透工具frp_frp_05](https://s3.51cto.com/images/blog/202106/28/9582e3d8836f5defe5d0aad780700c46.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

数据流向图

按照我的理解，画了一张数据流向图。按照数字进行先后顺序进行处理。

![最强内网穿透工具frp_frp_06](https://s3.51cto.com/images/blog/202106/28/c70b038511cf65b1bc8b19a5c4eb3e96.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

总结

内网穿透在ipv6没有普及之前都是刚需啊，例如很多人都会用内网穿透将内网window的3389端口映射，然后就可以在远端使用window自带的mstsc进行远程桌面的访问。也可以将内网Linux服务器的ssh端口穿透到公网，就可以在远程访问内网的ssh后台了。爱了爱了frp牛逼。