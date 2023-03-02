[docker使用教程相关系列 目录](https://blog.csdn.net/shi_hong_fei_hei/article/details/113954640)

[给技术经理找了几款Docker开源镜像仓库，为什么经理选中了Sonatype Nexus(上)](https://blog.csdn.net/shi_hong_fei_hei/article/details/115049919)

___

**目录**

[一、登录系统后台](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t0)

[二、创建docker仓库](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t1)

[操作步骤一：创建仓库](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t2)

[操作步骤二：配置仓库](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t3)

[操作步骤二：客户端配置](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t4)

[三、客户端使用](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t5)

[客户端使用报错](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t6)

[解决方案](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t7)

[push镜像到私服](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t8)

[pull 从私服拉镜像](https://blog.csdn.net/shi_hong_fei_hei/article/details/115050485?spm=a2c6h.12873639.article-detail.4.714259beCpklDw#t9)

___

## 一、登录系统后台

打开浏览器，访问 http://:8081/

输入账号密码

![0](https://img-blog.csdnimg.cn/img_convert/878dad56bafbfe2cf8d9294ebfcd81d9.png)

## 二、创建[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)仓库

## 操作步骤一：创建仓库

 ![0](https://img-blog.csdnimg.cn/img_convert/40b09ce75f28dcf0e7d140a23c402a39.png)

![0](https://img-blog.csdnimg.cn/img_convert/3e4d1fdfb245e13ae03c6573faa4d7b8.png)

## 操作步骤二：配置仓库

设置仓库名称

仓库指定一个唯一的名字，然后是选择http或https，这里只是测试用，所以走的http

注意：端口那里要确定好。

![0](https://img-blog.csdnimg.cn/img_convert/b66ff2671aa9cebc1188634c0c5d2607.png)

 创建成功后，可以在仓库列表查看到

![0](https://img-blog.csdnimg.cn/img_convert/90968b89ecd48e162542ef69efd3a9f1.png)

## 操作步骤二：客户端配置

因为使用的是http,所以需要在客户端编辑docker配置文件，比如

```
vim /etc/systemd/system/multi-user.target.wants/docker.service
```

 ![0](https://img-blog.csdnimg.cn/img_convert/120c090bc387a68d5ac5571a835c890f.png)

找到ExecStart属性，在dockerd后面添加--insecure-registry 服务器IP:Docker仓库端口

```
ExecStart=/usr/bin/dockerd --insecure-registry=ip:9021
```

![0](https://img-blog.csdnimg.cn/img_convert/f409ea88b15d362f1bf130856fa22d00.png)

保存，重新装载配置并重启docker服务

```
systemctl daemon-reload
```

![0](https://img-blog.csdnimg.cn/img_convert/bcd342f0e8d4121d17bc6793dfe47d31.png)

```
systemctl restart docker
```

![0](https://img-blog.csdnimg.cn/img_convert/23af338b2cd5f2cb5aa586731df508e3.png)

## 三、客户端使用

先docker login ip:9021,输入有权限的账号和密码，提示成功后即可docker push镜像到仓库中了。

例如

```
docker login -u admin -p 你的admin密码 192.168.88.131:9021
```

## 客户端使用报错

这时候会报错

```
Error response from daemon: Get http://192.168.88.131:9021/v1/users/: dial tcp 192.168.88.131:9021: connect: connection refused
```

### 解决方案

1、docker配置文件有没有配置

      如果没有配置，请按照 操作步骤二：配置仓库

2、docker仓库的端口有没有映射出来

      把9021端口映射出来

```
docker run -p 8081:8081 -p 9021:9021 --privileged=true --name nexus -v /usr/local/docker/nexus/nexus-data:/nexus-data 8716903d1912
```

## push镜像到私服

```
#先tag镜像，把镜像名变成包含本地仓库名（如192.168.88.131:9021）的镜像#docker tag 本地镜像名:镜像tag 私有库地址/镜像名:镜像tag tag centos_tomcat8:v1 192.168.88.131:9021/centos_tomcat8:v1
```

推送的时候，需要使用docker push 私有库地址/镜像名:镜像tag

```
再push镜像至仓库中： push 192.168.88.131:9021/centos_tomcat8:v1
```

![0](https://img-blog.csdnimg.cn/img_convert/312012d5c431317fe5b44baef4d63c33.png)

## pull 从私服拉镜像

```
docker pull 192.168.88.131:9021/centos_tomcat8:v1
```

![0](https://img-blog.csdnimg.cn/img_convert/be8f34f87598ba1fa116eb845efe5a03.png)

走到这一步，说明已经安装nexus成功，并能在客户端pull和push了。

感谢：

       在整理这两篇博文时，遇到了一些问题，感谢热心的杨同事和网友的支援，感受到了技术人的可爱！特此感谢！