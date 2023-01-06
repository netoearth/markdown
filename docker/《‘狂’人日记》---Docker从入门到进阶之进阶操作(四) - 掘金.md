   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

2022年08月29日 00:21 ·  阅读 431

![《‘狂’人日记》---Docker从入门到进阶之进阶操作(四)](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01705529f3fa413da14f1e55fb786bc1~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第29天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 前两章dockerfile和supermin都是针对docker镜像的操作，那么这篇文章将向您演示对docker数据的管理

在容器中管理数据主要有**两种**方式：

1.  数据卷（Data volumes）
2.  数据卷容器（Data volume containers）

## 1.数据卷

## 1.1 什么是数据卷？

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

1.  数据卷可以在容器之间共享和重用
2.  对数据卷的修改会立马生效
3.  对数据卷的更新，不会影响镜像
4.  卷会一直存在，直到没有容器使用

数据卷的使用，类似于 `Linux` 下对目录或文件进行 `mount`。

## 1.2 创建数据卷

```
# 创建一个名为 new-volume 的数据卷
docker volume create new-volume
# 列出
docker volume ls

# 查看卷信息
docker volume inspect new-volume
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f1cb72700a34df8a7d29c5dfef0d909~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

## 1.3 挂载数据卷

```
# 启动一个挂载新卷的nginx容器
docker run -dit --name nginx -v new-volume:/mnt nginx

# 查看容器的详细信息 grep -a 这里显示具体前后行数
 docker inspect nginx|grep Mounts -a5
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a5e1de73f7e4b538c3cb83898a0186f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/953b1c41b1704437811ab22063483639~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

## 1.4 删除数据卷

数据卷 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在 容器被删除后自动删除 数据卷 ，并且也不存在垃圾回收这样的机制来处理没有任 何容器引用的 数据卷 。如果需要在删除容器的同时移除数据卷。可以在删除容器 的时候使用 **docker rm -v** 这个命令。

```
docker volume rm new-volume
复制代码
```

无主的数据卷可能会占据很多空间，要清理请使用以下命令

```
docker volume prune
复制代码
```

## 2.数据卷容器

## 2.1 什么是数据卷容器？

数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的。

## 2.1、容器间共享卷

在这个案例中，将使用三个nginx容器进行演示，分别为 nginxweb1、nginxweb2、nginxweb3

```

# 启动nginxweb1 创建共享卷
docker run -dit --name nginxweb1 -p 8081:80 -v /usr/share/nginx/html nginx
# 启动nginxweb2、nginxweb3 并指定共享卷为 nginxweb1
docker run -dit --name nginxweb2 --volumes-from nginxweb1 -p 8082:80 nginx
docker run -dit --name nginxweb3 --volumes-from nginxweb1 -p 8083:80 nginx
# curl测试
curl localhost:8081
curl localhost:8082
curl localhost:8083
# 进入nginxweb1 进行修改index.html文件，并测试
docker exec -it nginxweb1 bash
容器内操作
echo "This is nginxweb1 website" > /usr/share/nginx/html/index.html
# 退出容器测试 exit
curl localhost:8081
curl localhost:8082
curl localhost:8083
复制代码
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f49f50eb81a4b25aa984a04866327ed~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3a061eea0f449b7ac5ae00a97a5f931~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 1

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏