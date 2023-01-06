[Docker](https://blog.yixun.store/tag/Docker)[Jenkins](https://blog.yixun.store/tag/Jenkins)

推荐使用 Pipeline ，使用场景更为广泛 功能也更强大

Jenkins很多人跑不起来，其实主要还是脚本的编写上有问题，所以我记录这篇博客的初衷也是记录脚本，因为安装不难 网上的教程一抓一大把。但是真的可用的的部署脚本却是少之又少，至少我研究jenkins的这两三天是没找到。

我也是这两天刚玩的jenkins，如果有什么地方写的不对，欢迎指出问题，共同进步

我这使用的Ubuntu系统，jenkins安装网上的资料也很全，我这个行不通的话安装可以试试别的，脚本是自己跑通之后记录的，其实脚本本身也就只是服务器命令的结合体，大体上也没什么难度

## 安装

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install openjdk-8-jdk 
sudo apt-get install jenkins
```

安装成功就可以启动了，Jenkins的默认端口是8080，配置文件地址`sudo vim /etc/default/jenkins`，可以视实际情况按需修改。 启动成功直接`http://IP:端口`就可以直接访问了。

## nginx配置

```
server {
    listen 80;
    server_name 你的域名; #需要将yourdomain.com替换成证书绑定的域名。
    rewrite ^(.*)$ https://$host$1; #将所有HTTP请求通过rewrite指令重定向到HTTPS。
    location / {
        index index.html index.htm;
    }
}

server
    {
        listen 80;
listen 443 ssl;
        server_name 你的域名;
#rewrite ^(.*) https://$server_name$1 permanent;
        #index index.html;
#root  /data/abl/system/page;

# 开启gzip
gzip  on;
# 低于1kb的资源不压缩
gzip_min_length 1k;
# 设置压缩所需要的缓冲区大小
gzip_buffers 4 16k;
# 压缩级别【1-9】，越大压缩率越高，同时消耗cpu资源也越多，建议设置在4左右。
gzip_comp_level 4;
# 需要压缩哪些响应类型的资源，缺少的类型自己补。
gzip_types text/css text/javascript application/javascript;
# 配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
gzip_disable "MSIE [1-6]\.";
# 是否添加“Vary: Accept-Encoding”响应头，
gzip_vary on;
# 设置gzip压缩针对的HTTP协议版本，没做负载的可以不用
# gzip_http_version 1.0;

index index.html index.htm;
ssl_certificate /data/ssl/证书.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
ssl_certificate_key /data/ssl/证书.key; #需要将cert-file-name.key替换成已上传的证书密钥文件的名称。
ssl_session_timeout 5m;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
#表示使用的加密套件的类型。
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
ssl_prefer_server_ciphers on;

if ($ssl_protocol = "") { return 301 https://$host$request_uri; } 
 # 接口
        location / {
proxy_pass http://localhost:8080;
        }
    }

```

## jenkins配置

1.  第一次访问`Jenkins`，会让你输入密码，按照给出的地址 去服务器上复制填入即可 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10174739.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10174739.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10174739.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10174739.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10174739.png")
2.  进去之后，会出现两个选项，第一个选项：安装建议的插件 第二个选项：自选插件去安装 。在此为了方便，我们选择第一个选项，安装建议的插件。然后等他安装完就可以了 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10192271.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10192271.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10192271.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10192271.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10192271.png")
3.  安装完所有插件后，设置好账号密码 就可以使用了 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10202263.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10202263.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10202263.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10202263.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10202263.png")

> 项目基础环境配置

我部署jenkins的时候，看网上的文章都在说需要配置`JDK`、`Git`，但是实际使用的其部署`JAVA`项目的时候，什么都没有配置的情况下，也是可以正常部署运行的 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10294387.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10294387.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10294387.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10294387.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10294387.png") 只需要使用其默认的`Maven`配置就可以了 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1031559.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1031559.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1031559.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1031559.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1031559.png")

## Docker安装

命令若是执行不了，可以自行百度，docker的一些安装配置，网上资料还是很全的

```
1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
二、Dcoekr配置阿里仓库
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF' { "registry-mirrors": ["https://co73sqx0.mirror.aliyuncs.com"] }
EOF
sudo systemctl daemon-reload sudo systemctl restart docker sudo systemctl enable docker
```

## 项目部署

## 项目添加Dockerfile文件

Dockerfile指令详解：[https://www.runoob.com/docker/docker-dockerfile.html](https://www.runoob.com/docker/docker-dockerfile.html "https://www.runoob.com/docker/docker-dockerfile.html")

[![https://img-blog.csdnimg.cn/26d7f8f7a17741d1bfcc29bb30bb5a40.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16](https://img-blog.csdnimg.cn/26d7f8f7a17741d1bfcc29bb30bb5a40.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16 "https://img-blog.csdnimg.cn/26d7f8f7a17741d1bfcc29bb30bb5a40.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16")](https://img-blog.csdnimg.cn/26d7f8f7a17741d1bfcc29bb30bb5a40.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16 "https://img-blog.csdnimg.cn/26d7f8f7a17741d1bfcc29bb30bb5a40.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16")

```
FROM java:8
ENV TZ Asia/Shanghai
# 这里需要注意 需要将jar包 复制到Dockerfile同一目录下，将Dockerfile移动或者复制到jar同一目录也行，反正这俩要在一起
ADD abl_system.jar abl_system.jar
# 另外使用了证书，比如微信退款证书的，如果一开始是将微信证书置于服务器上的，那么将项目用docker启动，是读不到证书的，需要将证书也放到Docker里
# ADD apiclient_cert.p12  /data/paySSL/
MAINTAINER Create App Docker By "XiaoYiXun"
EXPOSE 8002
ENTRYPOINT ["java","-jar","/abl_system.jar"]
CMD ["--spring.profiles.active=test"]
```

## Jenkins新建任务

> 配置服务器信息

[![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10551319.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10551319.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10551319.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10551319.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10551319.png") 进入系统配置后，找到`SSH Servers`，填入一些基本项即可`Name`,`Hostname`,`Username`,`Proxy password` [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1059425.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1059425.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1059425.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1059425.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_1059425.png")

> 新增 Git 凭据

[![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11020979.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11020979.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11020979.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11020979.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11020979.png")

凭据的类型不要选错，可以使用账号密码（账号一般是指邮箱，比如gitee，我平时都是用手机号登录，但是在这个地方账号填手机号是连不上去的，需要填邮箱），也可以使用令牌 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11025421.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11025421.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11025421.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11025421.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11025421.png")

> 新建一个maven任务

这里只介绍项目部署的一些基本项，其他的不做介绍（因为我也没用明白，滑稽.jpg）

1.  这里不要选第一个，我看网上很多文章都选的第一个，但是对于我们Java程序猿来说，部署项目选第二个其实简单很多 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10481278.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10481278.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10481278.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10481278.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_10481278.png")
    
2.  添加git仓库地址，并选择对应的凭证，如果是Gitee仓库地址的话，需要在 系统管理-插件管理-安装Gitee Plugin 插件，安装完之后重启即可 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11100632.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11100632.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11100632.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11100632.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11100632.png")
    
3.  点击跳转到最下面，构建后操作 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11141724.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11141724.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11141724.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11141724.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11141724.png")
    
4.  选择对应的服务器，在`Exec command`处填入相关脚本即可
    

```
#!/bin/bash WORKSPACE 是自带的环境变量，变量查询列表： https://jenkins.oplai.com/env-vars.html/
# 这里是将项目中的Dockerfile文件，复制到jar包所在文件目录
cp ${WORKSPACE}/abl-system/docker/Dockerfile  ${WORKSPACE}/abl-system/target

echo "开始运行"
echo "停止system_test"
docker stop system_test
echo "删除容器system_test"
docker rm system_test
echo "删除镜像aobaole_system_test:latest"
docker rmi -f aobaole_system_test:latest

#要把jar包制作成镜像就必须使用docker build命令 同时必须有对应的Dockerfile（指定jar包位置是在Dockerfile指定）
# . 表示当前目录 -f 参数指定Dockerfile文件  -t 表示 制作的镜像tag 
cd ${WORKSPACE}/abl-system/target
docker build -f Dockerfile  -t  aobaole_system_test:latest  .

echo "新容器id："
docker run -d -p 8002:8002 --name system_test aobaole_system_test:latest

echo "执行完毕"

```

[![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11164880.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11164880.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11164880.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11164880.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211022_11164880.png")

都弄完之后点击保存 然后执行就可以了

## 使用Pipeline部署

使用这种方式的话 本机上需要装maven

[![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_08150139.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_08150139.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_08150139.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_08150139.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_08150139.png")

[![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09310157.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09310157.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09310157.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09310157.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09310157.png") 将代码粘贴到脚本处，填写好相关参数即可

脚本本身 其实就是普通的命令，写过一次之后基本上就都懂了。

```
git_auth = "xxxxx"
git_branch = "*/master"
git_timeout = "120"
git_url = "xxxxx"
//远程docker仓库地址
docker_url = "ccr.ccs.tencentyun.com"
docker_space = "xiaoyixun"
docker_username = "xx"
docker_password = "xxx"
//目标服务器 本机部署不需要
target_ip = "127.0.0.1"
target_port = "22"
target_user = "root"
target_password = "root"
project_name = "abl_applets"
//dockerFile文件地址
docker_file_path = "docker/test/"
docker_tag = "test"
docker_port = "8003"
//模块名 单模块的时候为空就行
module_name = "abl-applets/"
pipeline {
    agent any
    stages {
        stage('环境检测') {
            steps {
script{
// 标识是否是部署到其他服务器
NEED_UPLOAD_DOCKER = true
}
                sh label: '',
                script: '''
                    mvn -v
                    git version
                    java -version
                    docker -v
                '''

            }
        }
        stage('代码获取') {
            steps {
                checkout([
                $class: 'GitSCM',
                branches: [[name: "${git_branch}"]],
                extensions: [[
                    $class: 'CloneOption', noTags: false, reference: '',
                    shallow: false, timeout: "${git_timeout}"
                ]],
                userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_url}"]]
                ])
            }
        }
        stage('项目构建') {
            steps {
                sh label:'',
                script:'mvn clean package -Dmaven.test.skip=true'
            }
        }
stage('镜像构建') {
steps {
script{
if(NEED_UPLOAD_DOCKER){
sh label:'',
script:"""
cp -r ${module_name}target/${project_name}.jar ${module_name}${docker_file_path}
sudo docker build --rm -t ${docker_url}/${docker_space}/${project_name}:${docker_tag} ${module_name}${docker_file_path}
rm ${module_name}${docker_file_path}${project_name}.jar
exit
"""
}else{
sh label:'',
script:"""
sudo docker rm -f ${project_name}_${docker_tag}
sudo docker rmi ${project_name}:${docker_tag}
cp -r ${module_name}target/${project_name}.jar ${module_name}${docker_file_path}
sudo docker build --rm -t ${project_name}:${docker_tag} ${module_name}${docker_file_path}
rm ${module_name}${docker_file_path}${project_name}.jar
exit
"""
}
}
}
}
stage('镜像上传') {
steps {
script{
if(NEED_UPLOAD_DOCKER){
sh label:'',
script:"""
sudo docker login --username=${docker_username} --password=${docker_password} ${docker_url}
sudo docker push ${docker_url}/${docker_space}/${project_name}:${docker_tag}
sudo docker rmi ${docker_url}/${docker_space}/${project_name}:${docker_tag}
sudo docker logout ${docker_url}
exit
"""
}
}
}
}
stage('项目部署') {
steps {
script{
if(NEED_UPLOAD_DOCKER){
sh label:'',
script:"""
sshpass -p ${target_password} ssh -tt -p ${target_port} -o StrictHostKeyChecking=no ${target_user}@${target_ip} << reallsh
sudo docker rm -f ${project_name}_${docker_tag}
sudo docker rmi ${docker_url}/${docker_space}/${project_name}:${docker_tag}
sudo docker login --username=${docker_username} --password=${docker_password} ${docker_url}
sudo docker pull ${docker_url}/${docker_space}/${project_name}:${docker_tag}
sudo docker run -itd --name ${project_name}_${docker_tag} -p ${docker_port}:${docker_port} --restart always ${docker_url}/${docker_space}/${project_name}:${docker_tag}
sudo docker logout ${docker_url}
exit
reallsh
"""

}else{
sh label:'',
script:"""

sudo docker run -itd --name ${project_name}_${docker_tag} -p ${docker_port}:${docker_port} --restart always ${project_name}:${docker_tag}
exit
reallsh
"""
}
}
}
}
}
    post {
        always {
            sh label:'',
           script:"""
                mvn clean
            """
        }
    }
}


```

这个错误就是因为我没有把目标文件复制到`Dockerfile`所在目录导致，并且可以看到 pwd命令输出的当前目录并不在`Dockerfile`文件所在目录，所以`docker build`命令结尾不能使用 `.` ，而需要指定到`Dockerfile`指定目录 [![https://img-blog.csdnimg.cn/dddc9fe91a754081aaa3bc1ecdcd5832.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16](https://img-blog.csdnimg.cn/dddc9fe91a754081aaa3bc1ecdcd5832.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16 "https://img-blog.csdnimg.cn/dddc9fe91a754081aaa3bc1ecdcd5832.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16")](https://img-blog.csdnimg.cn/dddc9fe91a754081aaa3bc1ecdcd5832.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16 "https://img-blog.csdnimg.cn/dddc9fe91a754081aaa3bc1ecdcd5832.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6JCn5LiA5pes,size_20,color_FFFFFF,t_70,g_se,x_16") 注意点：

1.  脚本代码里有一段被注释的代码，之所以被注释是因为这种方式，不能实现在远程服务器部署，到头来会变成在你Jenkins的安装服务器上去执行脚本代码，但是如果你就只是想在本机部署，那么可以使用第二种
2.  第三方Docker容器服务可以使用阿里云或者是腾讯云的容器服务，阿里云一搜就可以找到，腾讯云稍微藏了一下所以放个截图 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09504867.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09504867.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09504867.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09504867.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_09504867.png")
3.  本机部署的时候 会有一个小bug，在镜像构建的时候 ，如果是第一次部署，`sudo docker rm -f ${project_name}_${docker_tag} sudo docker rmi ${project_name}:${docker_tag}` 这两行代码会报错 ，所以第一次部署的注释掉 然后部署成功之后再加上就行QAQ

## 结果

这个docker镜像上传，还是有点看服务器性能，两张截图是不同服务器，部署了两套jenkins做的测试对比 [![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10011199.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10011199.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10011199.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10011199.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10011199.png")

[![http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10030431.png](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10030431.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10030431.png")](http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10030431.png "http://blogimgbucketxun.oss-cn-beijing.aliyuncs.com/xiaoYiXun/common/20211026_10030431.png")

所以如果只是在本机部署的话，其实是没有这个顾虑的，本机部署就不需要push到公共仓库了，直接本机构建完，然后直接run就可以了

本站文章除注明转载/出处外，均为本站原创或翻译，转载前请务必署名,转载请标明出处  
最后编辑时间为: 2022/07/01 15:39