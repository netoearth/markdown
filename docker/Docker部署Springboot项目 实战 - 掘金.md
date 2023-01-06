   

[![](https://p3-passport.byteimg.com/img/user-avatar/fc54186e0dcc5885b3f51192d57df321~100x100.awebp)](https://juejin.cn/user/2859142558267559)

2021年07月28日 11:08 ·  阅读 1712

![Docker部署Springboot项目 实战](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff668af59bee416084bd7dcb9b84497c~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp)  

`封面来源：` ==说唱新时代 鱼翅Fin《我是我最后的目击者》==

> 在学习编程的时候，我们都会想，该怎么把自己写出来的项目和创意分享给大家勒。直接给代码吗？平常人看不懂，环境还需要一大堆。 那么部署到服务器上，让人能够直接访问，我想这应该是最棒的方式了吧。 使用Docker 的话，那么就是让这个方便变得更加的便捷啦。

[阿里云服务器上Docker部署前端Vue项目实战](https://link.juejin.cn/?target=url "url")

## 一、前言

我写这个博客前已经将我需要的mysql、redis等等都安装好了。安装redis的博客、我之前也写啦的。

在这里只讲怎么将项目放到服务器上的docker上去跑，环境还是需要自己搭建的。

项目中用到什么，就要在docker中安装什么。 本人项目环境：

jdk11 、mysql 5.7、redis

项目结构

![<img src="发布项目到服务器上的Docker.assets/image-20210428214235231.png" alt="image-20210428214235231" style="zoom:67%;" />](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb48397cf2834d9d8925a1eb7885d203~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

如果想要idea 中dockerfile 文件高亮的话，在idea下载一下docker插件

![<img src="发布项目到服务器上的Docker.assets/image-20210428214405834.png" alt="image-20210428214405834" style="zoom:67%;" />](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7800209b388f4639951e18656649078b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 二、具体步骤：

#### 1、打成jar包

1、将运行的项目使用maven打成jar包，率先放在本地测试，看有没有问题。

![<img src="发布项目到服务器上的Docker.assets/image-20210428205006468.png" alt="image-20210428205006468" style="zoom:67%;" />](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c5f6787e62d47eaab90aa126087a466~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

我这个是一个springboot项目 点击maven的打包之后 ，就会生成一个jar 包

然后在命令行编译它。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/940cbbce30ec4e29aa9eb00de306178e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

自己然后再在浏览器内进行测试。看是否可以运行。

成功的话，接下来就开始写Dockerfile文件。

#### 2、Dockerfile文件内容

我用的是jdk11

```
FROM openjdk:11 # FROM: 基础镜像，基于jdk8镜像开始

COPY *.jar /app.jar  # COPY: 将应用的配置文件也拷贝到镜像中。

CMD ["--server.port=8080"]

EXPOSE 8080  # EXPOSE：声明端口
 
ENTRYPOINT ["java","-jar","/app.jar"]  
# ENTRYPOINT：docker启动时，运行的命令，这里容器启动时直接运行jar服务。
复制代码
```

#### 3、上传

**上传jar包和dockerfile文件**到服务器上去。

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-uql6kdhI-1619616926791)(发布项目到服务器上的Docker.assets/image-20210428210245926.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18f6c0f74f5d44d297eea785bd0a32a3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 4、打包成docker镜像

我一开始的话 已经在服务器上把文件夹建好了....

就直接去这个文件下查看文件就好拉。

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-KV7KNKkK-1619616926795)(发布项目到服务器上的Docker.assets/image-20210428211321555.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ec65c7ec79c4ce78cd0202763a785de~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

查看无误后 用docker的打包命令 将这两个一起打包成一个镜像。 必须这两个东西都在一个文件下，才可以。

```
docker build -t news_school_web1 .  
复制代码
```

\==注: 先将最重要的， 最后是有一个小数点的，千万不要忘了。==

-   docker bulid 是打包命令
-   `-t` − 给镜像加一个Tag
-   后面跟的 news\_school\_web1 就是为这个镜像取的名字
-   `.` 小数点表示当前目录，即Dockerfile所在目录

成功的话 应该是这样子的

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Dgwv3G3J-1619616926796)(发布项目到服务器上的Docker.assets/image-20210428212212251.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/723f1d075eac4ae7aca66333472486f9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

当然这样是不够的，我们输入命令去查看一下。 看有没有这个镜像。

```
docker images
复制代码
```

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-O4lbs47R-1619616926800)(发布项目到服务器上的Docker.assets/image-20210428212413687.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c3f1f99761e41e096cb2717e922ba43~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 5、启动镜像

```
docker run -d -p 8686:8686 --name news_web_test news_school_web1
复制代码
```

-   \-d 是后台运行
-   \-p 8686:8686 是端口映射
-   \--name 取名字
-   最后跟的 news\_school\_web1 是我打包好的镜像名称。

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-qllErRpi-1619616926803)(发布项目到服务器上的Docker.assets/image-20210428212719782.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f369bcfce204543bdbc7ba782e83595~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

查看

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-hFGy4J3v-1619616926806)(发布项目到服务器上的Docker.assets/image-20210428212737960.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f00067d84db4472a876b4f7b302d313~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 6、测试

-   先在服务器测试 成功返回我的页面
    
    ![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-hVQLrKCN-1619616926807)(发布项目到服务器上的Docker.assets/image-20210428212833550.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf17cc26aeed44aea2514bbfde700fa2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)
    
-   接着在外网测试
    
    ```
    http://IP地址:8686/login
    复制代码
    ```
    
    ![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-SdHLhW7A-1619616926808)(发布项目到服务器上的Docker.assets/image-20210428213108424.png)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25684ec59271435ca8c63d8b63d14591~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)
    

我用的是post测试，成功返回自己的数据，就代表已经成功在运行拉。

## 日常自言自语

想着这一次将过程好好记录下来，在能够帮助到自己的同时，再帮助到其他人。😊

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-nZ3FlXuw-1619616926809)(发布项目到服务器上的Docker.assets/21f15fe11b7a84d2f2121c16dec50a4e4556f865.png@100w_100h.webp)]](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbee9596848b4e898d37b820d4c345c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

文章被收录于专栏：

![cover](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5516ed36df6a424e951525c78197c166~tplv-k3u1fbpfcp-no-mark:160:160:160:120.awebp)

Docker

本栏主要写一些Docker部署安装镜像使用以及一些实战内容。内容偏向实战为更多一些。

相关课程

![cover](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/25/16d67c7db5fc6421~tplv-t2oaga2asx-zoom-mark-crop-v2:0:0:160:224.awebp)

VIP

MySQL 是怎样使用的：从零蛋开始学习 MySQL

[小孩子4919 ![lv-5](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-5.d08789d.png "创作等级")](https://juejin.cn/user/3526889001463006)   

4607购买

![cover](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/20/172d1488323e5a18~tplv-t2oaga2asx-zoom-mark-crop-v2:0:0:160:224.awebp)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 3

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏