### 文章目录

-   -   -   -   -   [一、安装docker](https://blog.csdn.net/weixin_40816738/article/details/117652736#docker_1)
                -   -   [1\. 在线安装docker](https://blog.csdn.net/weixin_40816738/article/details/117652736#1_docker_2)
                    -   [2\. 换镜像源](https://blog.csdn.net/weixin_40816738/article/details/117652736#2__13)
                -   [二、安装redis](https://blog.csdn.net/weixin_40816738/article/details/117652736#redis_29)
                -   [三、安装mysql](https://blog.csdn.net/weixin_40816738/article/details/117652736#mysql_53)
                -   [四、安装RabbitMq](https://blog.csdn.net/weixin_40816738/article/details/117652736#RabbitMq_77)
                -   [五、安装ElasticSearch](https://blog.csdn.net/weixin_40816738/article/details/117652736#ElasticSearch_91)
                -   -   [5.1. 修改服务器配置](https://blog.csdn.net/weixin_40816738/article/details/117652736#51__98)
                    -   [5.2. 创建容器并启动 ES](https://blog.csdn.net/weixin_40816738/article/details/117652736#52__ES_103)
                    -   [5.3. 查看启动日志](https://blog.csdn.net/weixin_40816738/article/details/117652736#53__109)
                    -   [5.4. 进入镜像](https://blog.csdn.net/weixin_40816738/article/details/117652736#54__116)
                    -   [5.5. 修改cluster-name](https://blog.csdn.net/weixin_40816738/article/details/117652736#55_clustername_120)
                    -   [5.6. 安装中文分词插件](https://blog.csdn.net/weixin_40816738/article/details/117652736#56__129)
                    -   [5.7. 退出并重启镜像](https://blog.csdn.net/weixin_40816738/article/details/117652736#57__136)
                    -   [5.8. 查看启动日志](https://blog.csdn.net/weixin_40816738/article/details/117652736#58__145)
                -   [六、构建eblog的docker镜像](https://blog.csdn.net/weixin_40816738/article/details/117652736#eblogdocker_148)
                -   -   [6.1. 克隆项目到本地](https://blog.csdn.net/weixin_40816738/article/details/117652736#61__153)
                    -   [6.2.下载maven依赖](https://blog.csdn.net/weixin_40816738/article/details/117652736#62maven_160)
                    -   [6.3. 修改yml配置和本地运行](https://blog.csdn.net/weixin_40816738/article/details/117652736#63_yml_162)
                    -   [6.4. 采用容器别名通信](https://blog.csdn.net/weixin_40816738/article/details/117652736#64__171)
                    -   [6.5. 跳过测试打包](https://blog.csdn.net/weixin_40816738/article/details/117652736#65___183)
                    -   [6.6. 上传jar包](https://blog.csdn.net/weixin_40816738/article/details/117652736#66__jar_190)
                    -   [6.7. 制作Dockerfile](https://blog.csdn.net/weixin_40816738/article/details/117652736#67__Dockerfile_192)
                    -   [6.8. 制作eblog镜像](https://blog.csdn.net/weixin_40816738/article/details/117652736#68_eblog_225)
                    -   [6.9. 查看镜像列表](https://blog.csdn.net/weixin_40816738/article/details/117652736#69__282)
                -   [七、启动eblog项目](https://blog.csdn.net/weixin_40816738/article/details/117652736#eblog_289)
                -   -   [7.1. 容器运行状态查看](https://blog.csdn.net/weixin_40816738/article/details/117652736#71__291)
                    -   [7.2. 修改websoket地址](https://blog.csdn.net/weixin_40816738/article/details/117652736#72_websoket_293)
                    -   [7.3. 启动eblog](https://blog.csdn.net/weixin_40816738/article/details/117652736#73_eblog_296)
                    -   [7.4. 查看eblog打印日志](https://blog.csdn.net/weixin_40816738/article/details/117652736#74_eblog_304)
                    -   [7.5. 浏览器验证](https://blog.csdn.net/weixin_40816738/article/details/117652736#75__310)

##### 一、安装[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)

###### 1\. 在线安装docker

```
#安装
yum install docker

#检验安装是否成功
[root@localhost opt]# docker --version
Docker version 1.13.1, build 7f2769b/1.13.1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607105324530.png)

###### 2\. 换镜像源

```
sudo vim /etc/docker/daemon.json
内容如下：
{
 "registry-mirrors": ["https://m9r2r2uj.mirror.aliyuncs.com"]
}
保存退出，重启docker

#重启
sudo systemctl daemon-reload
sudo systemctl restart docker
```

具体地址获取方式：  
[Centos7 解决Docker拉取镜像慢的问题](https://blog.csdn.net/weixin_40816738/article/details/104587121)

##### 二、安装redis

> 首先上dockerHub搜索redis，点击进入详情页之后，拉到下面就可以看到how to use，如果需要选择特定的版本，有Supported tags给我们选择，然后如果拉取最新的版本的话，拉倒下面就教程。  
> [https://hub.docker.com/\_/redis](https://hub.docker.com/_/redis)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607105928384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

```
#拉取redis的镜像
docker pull redis
#查看本地redis镜像
docker images
#运行redis
docker run --name myredis -p 6379:6379 -d redis redis-server --appendonly yes
```

-   docker run表示运行的意思
-   –name myredis 表示起个名字叫myredis
-   \-p 6379:6379表示把服务器的6379映射到docker的6379端口，这样就可以通过服务器的端口访问docker的端口
-   \-d 表示以后台服务形式运行redis
-   redis redis-server --appendonly yes表示开启持久化缓存模式，可以存到硬盘

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607155301531.png)

##### 三、安装mysql

> 如果想下载指定版本的mysql镜像请一部官网选择版本  
> [https://hub.docker.com/\_/mysql](https://hub.docker.com/_/mysql)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607110217432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607110322191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

```
docker pull mysql:5.7.27
docker run --name mymysql -e MYSQL_ROOT_PASSWORD=admin -d -p 3306:3306  mysql:5.7.27 
```

-   MYSQL\_ROOT\_PASSWORD=admin表示root的初始密码
-   mysql:5.7.27表示操作的是mysql的5.7.27版本，没有后面的版本号的话，默认是拉取最新版本的mysql。

连上mysql，创建数据库`eblog`，然后把数据库脚本导入进去。 脚本位置：  
[https://github.com/MarkerHub/eblog/blob/master/eblog.sql](https://github.com/MarkerHub/eblog/blob/master/eblog.sql)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607110512641.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607110602606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
如果遇到异常，请移步  
[1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contai](https://gblfy.blog.csdn.net/article/details/117654908)

##### 四、安装[RabbitMq](https://so.csdn.net/so/search?q=RabbitMq&spm=1001.2101.3001.7020)

```
docker run -d --hostname my-rabbit --name myrabbit -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management
```

-   指定hostname为my-rabbit ： --hostname my-rabbit
-   创建用户 ： -e RABBITMQ\_DEFAULT\_USER
-   设置密码： -e RABBITMQ\_DEFAULT\_PASS
-   可视化窗口版本 ： rabbitmq:management
-   端口映射：-p 5672:5672

一行命令搞定，注意RABBITMQ\_DEFAULT\_PASS=password是设置密码  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607145849427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607145905174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

##### 五、安装[ElasticSearch](https://so.csdn.net/so/search?q=ElasticSearch&spm=1001.2101.3001.7020)

> docker 安装 Elasticsearch6.4.3版本 及中文插件安装。  
> 系统配置  
> 不配置的话，可能会启动失败  
> 具体报错：max virtual memory areas vm.max\_map\_count \[65530\] is too low, increase to at least \[262144\]。  
> 解决：

###### 5.1. 修改服务器配置

```
sudo sysctl -w vm.max_map_count=262144
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607150152177.png)

###### 5.2. 创建容器并启动 ES

```
docker run -p 9200:9200 -p 9300:9300 -d --name es_643 elasticsearch:6.4.3
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607150858798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 5.3. 查看启动日志

```
docker logs -f es_643 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607151008865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 5.4. 进入镜像

```
docker exec -it es_643 /bin/bash
```

###### 5.5. 修改cluster-name

es配置文件位置： `/usr/share/elasticsearch/config/elasticsearch.yml`

```
# 修改cluster-name保持和eblog程序yml配置文件一致，如果不需要，可以不改
vi /usr/share/elasticsearch/config/elasticsearch.yml
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607151345495.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607151218776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607151324540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 5.6. 安装[中文分词](https://so.csdn.net/so/search?q=%E4%B8%AD%E6%96%87%E5%88%86%E8%AF%8D&spm=1001.2101.3001.7020)插件

```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.3/elasticsearch-analysis-ik-6.4.3.zip
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607151425570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 5.7. 退出并重启镜像

```
# 退出
exit

# 重启es_643容器
docker restart es_643
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607153043913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 5.8. 查看启动日志

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607153512513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

##### 六、构建eblog的docker镜像

> 前提：mysql、redis、RabbitMq、elasticsearch容器都处于启动状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607155406377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 6.1. 克隆项目到本地

```
git clone git@github.com:MarkerHub/eblog.git
```

或者直接下载zip包  
[https://github.com/MarkerHub/eblog/archive/refs/heads/master.zip](https://github.com/MarkerHub/eblog/archive/refs/heads/master.zip)

###### 6.2.下载[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)依赖

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607153919379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 6.3. 修改yml配置和本地运行

将mysql、redis、RabbitMq、elasticsearch 地址链接、名称、用户、密码等信息地址修改为容器信息  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607154630827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607154554970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607155532806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607155514525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
看到以上截图，说明服务都部署成功！

###### 6.4. 采用容器别名通信

> 为了保证容器内部之间可以通信，因此，采用给容器起别名方式±-link参数

| 调整前 | 调整后 | 说明 |
| --- | --- | --- |
| 192.168.223.128 | emysql | mysql地址 |
| 192.168.223.128 | eredis | redis地址 |
| 192.168.223.128 | ees | elasticsearch地址 |
| 192.168.223.128 | erabbit | rabbit地址 |

如下图所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607160057774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 6.5. 跳过测试打包

```
cd D:\vue\eblog-master
mvn clean package -Dmaven.test.skip=true
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607160539710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607160610618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 6.6. 上传jar包

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607160914124.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 6.7. 制作Dockerfile

```
cd /app
vim Dockerfile
```

```
# Docker image for springboot file run
# VERSION 1.0.0
# Author: gblfy
FROM java:8
EXPOSE 8080
MAINTAINER gblfy <xxx@qq.com>
ENV TZ=Asia/Shanghai
RUN ln -sf /usr/share/zoneinfo/{TZ} /etc/localtime && echo '{TZ}' > /etc/timezone
ADD eblog-0.0.1-SNAPSHOT.jar /app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607162409371.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

> 注：jar包需要和Dockerfile在同一级目录  
> FROM java:8 表示基于jdk8环境  
> EXPOSE 8080 表示对外暴露的端口是8080

-   VOLUME /tmp 表示挂载到/tmp目录
-   ADD eblog-0.0.1-SNAPSHOT.jar /app.jar 表示把jar包复制到镜像服务里面的根目录，并改名称app.jar
-   RUN bash -c ‘touch /app.jar’ 表示执行创建app.jar
-   ENTRYPOINT \[“java”,"-jar","/app.jar"\] 表示执行启动命令java -jar

###### 6.8. 制作eblog镜像

接下来，我们安装Dockrfile的命令，把eblog-0.0.1-SNAPSHOT.jar构建成docker的镜像。

```
cd /app

# 构建镜像，注意后面有个点哈
docker build -t eblog .
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607161446313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

> 注：Dockerfile文件内容可以自定义  
> 制作镜像也添加添加版本信息

```
docker build -t eblog:1.0 .
```

操作记录：

```
[root@localhost app]# 
[root@localhost app]# docker build -t eblog:1.0 .
Sending build context to Docker daemon  73.7 MB
Step 1/8 : FROM java:8
 ---> d23bdf5b1b1b
Step 2/8 : EXPOSE 8080
 ---> Using cache
 ---> e91404d45de4
Step 3/8 : MAINTAINER gblfy <xxx@qq.com>
 ---> Running in 5b79b4ead8e2
 ---> 0e711f0534b6
Removing intermediate container 5b79b4ead8e2
Step 4/8 : ENV TZ Asia/Shanghai
 ---> Running in e23d8325cc54
 ---> c9eba2cd377e
Removing intermediate container e23d8325cc54
Step 5/8 : RUN ln -sf /usr/share/zoneinfo/{TZ} /etc/localtime && echo '{TZ}' > /etc/timezone
 ---> Running in 4b82ff8fab7c

 ---> 3ea55048b889
Removing intermediate container 4b82ff8fab7c
Step 6/8 : ADD eblog-0.0.1-SNAPSHOT.jar /app.jar
 ---> d6d02fb42305
Removing intermediate container c95686e0b3dc
Step 7/8 : RUN bash -c 'touch /app.jar'
 ---> Running in ea7e06619867

 ---> 60f8478bc78b
Removing intermediate container ea7e06619867
Step 8/8 : ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom -jar /app.jar
 ---> Running in 0a92b68b6034
 ---> 9394516f581d
Removing intermediate container 0a92b68b6034
Successfully built 9394516f581d
[root@localhost app]# 
```

###### 6.9. 查看镜像列表

```
docker images
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607162637153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

复制代码这步骤完成之后，我们就可以在准备工作就已经完成啦，接下来，我们就直接启动我们的项目哈。

##### 七、启动eblog项目

> 前提：mysql、redis、RabbitMq、elasticsearch容器都处于启动状态

###### 7.1. 容器运行状态查看

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607161810834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 7.2. 修改websoket地址

`static/res/js/im.js`  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607170639635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 7.3. 启动eblog

> 因为eblog容器需要调用mysql/redis/rabbit/es\_643容器服务，相当于跨容器通信，因此，采用–link方式

```
docker run -p 8080:8080 -p 9326:9326 --name eblog --link es_643:ees --link myrabbit:erabbit --link mymysql:emysql --link myredis:eredis -d eblog:1.0
```

-   \-p 8080:8080 -p 9326:9326 ：9326是因为即时聊天需要用到的ws端口
-   –link es:ees 表示关联容器，把容器es起别名为ees，前面是容器，后面是别名（别名在eblog容器中的yml已经配置）  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607164407853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 7.4. 查看eblog打印日志

```
docker logs -f eblog
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021060717092325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)

###### 7.5. 浏览器验证

[http://192.168.223.128:8080/](http://192.168.223.128:8080/)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021060717005840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607171557840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDgxNjczOA==,size_16,color_FFFFFF,t_70)