   

[![](https://p26-passport.byteacctimg.com/img/user-avatar/e3b82b14bcbe9861a2b50f64dcd89de1~300x300.image)](https://juejin.cn/user/3611262085773719)

2022年08月26日 08:34 ·  阅读 408

![《‘狂’人日记》---Docker从入门到进阶之进阶操作(一)](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/767b18832ef84060b32b5226e60ec98a~tplv-k3u1fbpfcp-zoom-crop-mark:3024:3024:3024:1702.awebp?)  

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第26天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 提供前面三章的学习，相信大家已经对docker有了初步了解，那么今天就玩点进阶操作，对Docker的网络进行管理

## 1.容器网络

## 1.1.映射网络

**容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P 或 -p 参数来指定端口映射。  
当使用 -P 标记时，Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口。  
使用 docker ps 可以看到，本地主机的 49153 被映射到了容器的 80 端口。此时访问本机的 49153 端口即可访问容器 内 web 应用提供的界面。**

```

# 指定映射
docker run -dit --name nginxweb1 -p 8081:80 nginx

# 随机映射
docker run -dit --name nginxweb4 -P nginx

# 查看所有容器
docker ps -a
复制代码
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13c0f826b0d74013b40f6a898edb3696~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 1.2.主机网络

可以使用 `--network=host` 参数来直接使用宿主机网络

```
docker run -dit --network=host --name nginx-host nginx
复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67992a98be4a456fa1809e5936686acf~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 1.3.容器互联

### 1.3.1 新建网桥

**\-d 参数指定 Docker 网络类型，有 bridge overlay 。其中 overlay 网络 类型用于 Swarm mode，在本小节中你可以忽略它。**

```
docker network create -d bridge new-net
复制代码
```

### 1.3.2 容器互联创建容器

```
docker run -dit --name box1 --network new-net busybox
docker run -dit --name box2 --network new-net busybox
复制代码
```

### 1.3.3 容器互联测试

```
# 进入box1容器
docker exec -it box1 sh

# ping测试
ping box2
复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88fa9b7947fc4d1daf045e843d16c115~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 2.自定义docker0网络

**当 Docker 启动时，会自动在主机上创建一个 docker0 虚拟网桥，实际上是 Linux 的一个 bridge，可以理解为一个软件交 换机。它会在挂载到它的网口之间进行转发。 同时，Docker 随机分配一个本地未占用的私有网段（在 RFC1918 中定义）中的一个地址给 docker0 接口。比如典型的 172.17.42.1 ，掩码为 255.255.0.0 。此后启动的容器内的网口也会自动分配一个同一网段（ 172.17.0.0/16 ）的 地址。 当创建一个 Docker 容器的时候，同时会创建了一对 veth pair 接口（当数据包发送到一个接口时，另外一个接口也可以 收到相同的数据包）。这对接口一端在容器内，即 eth0 ；另一端在本地并被挂载到 docker0 网桥，名称以 veth 开 头（例如 vethAQI2QT ）。通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。Docker 就创建了在主机和所 有容器之间一个虚拟共享网络。**

## 2.1 移除原有网络

### 2.1.1 安装btctl命令

```
yum install -y bridge-utils
复制代码
```

### 2.1.2 查询网桥信息

```
brctl  show
复制代码
```

### 2.1..3 关闭docker服务

```
systemctl stop docker
复制代码
```

### 2.1.4 停止docker0网桥

```
ip link set dev docker0 down
复制代码
```

### 2.1.5 删除docker0网桥

```
brctl delbr docker0
复制代码
```

### 2.1.6 查询所有网桥信息

```
brctl show
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5635461710c24247bd24d9546f646445~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 2.2 自定义新网络

### 2.2.1 创建新网桥bridge0

```
brctl addbr bridge0
复制代码
```

### 2.2.2 查询创建的bridge0

```
brctl show
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d733f5d044942e4942f422fb53fc005~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

### 2.2.3 设置新网桥的网络段地址

```
ip addr add 192.168.0.1/24 dev bridge0
复制代码
```

### 2.2.4 启动bridge0网桥

```
ip link set dev bridge0 up
复制代码
```

### 2.2.5 查询bridge0网桥信息

```
ifconfig bridge0
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18102857b7f74a4ea4fec033bad7e4cd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

#### 7.4.2.6、添加新网桥到配置文件

```
vi /etc/sysconfig/docker
# 添加 -b=bridge0 到 OPTIONS 中
OPTIONS='-b=bridge0'
复制代码
```

### 2.2.7 加载配置文件，重启docker服务

```
systemctl daemon-reload
systemctl restart docker
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dbaee4b44ec41558e79118a5eeccd30~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 2.3 使用新网络部署应用测试docker0网络

### 2.3.1 创建一个nginx容器

```
docker run -dit --name nginx-net nginx
复制代码
```

### 2.3.2 查看容器的状态

```
docker ps -a
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d829f7d1b4f4efd8fbd326a6e0666aa~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)[](https://link.juejin.cn/?target= "https://link.juejin.cn?target=")

### 2.3.3 查看容器的bridge

```
docker inspect -f {{.NetworkSettings.Networks.bridge}} nginx-net
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70f37a76d03e4f4c9138604f92c9413a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 1

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏