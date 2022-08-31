©著作权归作者所有：来自51CTO博客作者品鉴初心的原创作品，请联系作者获取转载授权，否则将追究法律责任

## Harbor 安装环境说明

| 环境名称 | 版本 | 备注 |
| --- | --- | --- |
| Linux发行版 | 18.04.5 LTS (Bionic Beaver) | 暂无 |
| docker-ce | Docker version 20.10.7, build 20.10.7-0ubuntu1~18.04.1 | apt install docker.ro，数据目录/data/docker |
| docker-compose | docker-compose version 1.24.0, build 0aa59064 |  [https://hkzww.com/ubuntu18-04下安装docker-compose-最新版.html](https://hkzww.com/ubuntu18-04%E4%B8%8B%E5%AE%89%E8%A3%85docker-compose-%E6%9C%80%E6%96%B0%E7%89%88.html) |
| Harbor | 2.3.2 | 最新2021/8/31 |
| harbor安装方式 | 在线安装 | 暂无 |
| harbor安装位置 | /usr/local/harbor-2.3.2 | 暂无 |
| harbor数据目录 | /data/harbor | 暂无 |

## 安装 Harbor

Harbor 的在线或者离线安装程序下载地址可以从 [这里](https://github.com/goharbor/harbor/releases)获取。

完成以上操作，保存退出！

最终目录结构：

## 查看Harbor容器的运行状态

## docker-compose 基本命令

## Harbor测试访问

浏览器输入以下地址或者域名访问Harbor的Web界面，账号密码：admin/xxxxxxxx

 [http://harbor.test.com](http://harbor.test.com/)

![使用docker-compose部署最新版Harbor v2.3.2_docker-compose](https://s2.51cto.com/images/blog/202109/02/7be922ee69ae334e35cd8aef8ea441bd.jpeg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 生成TLS证书，用于Harbor配置Https

## Harbor安装后配置Https

## 测试通过Https协议访问Harbor

## 通过浏览器访问

这里首先需要将上面产生的`/data/harbor/cert/ca.crt`导入到浏览器的受信任的根证书中，然后就可以通过Https协议访问Harbor的Web界面了，但不能保证所有浏览器都支持。访问地址是：`https://harbor.test.com`

关于`火狐浏览器`添加证书操作：点击”三“ --> 设置 --> 隐私与安全 --> 查看证书 --> 导入证书

关于`google浏览器`添加证书操作：点击”三“ --> 设置 --> 隐私设置和安全性 --> 安全 --> 管理证书 --> 导入证书

## 通过Docker命令来访问

## 集成LDAP

可直接登入 harbor 在”配置管理“中修改”认证模式“，大概形式如下：

![使用docker-compose部署最新版Harbor v2.3.2_harbor v2.3.2_02](https://s2.51cto.com/images/blog/202109/02/cd46c970748b52aafba186bd31b33395.jpeg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 参考文档

-    [Harbor仓库配置https访问](https://www.cnblogs.com/sanduzxcvbnm/p/11956347.html)
    
-    [离线安装 Harbor v2](https://blog.atompi.com/2020/08/03/%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85%20Harbor%20v2/)
    

-   **打赏**
-   ****1**赞**
-   ****1**收藏**
-   **评论**
-   **举报**