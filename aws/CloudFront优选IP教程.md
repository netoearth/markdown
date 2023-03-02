**准备以下：**

AWS账户，服务器，XUI，将服务器IP绑定到域名

### 1.创建Cloud front测速链接

aws首页搜索cloud front![cloudfront](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd728a8c0-476e-4747-964c-94dfad0cc664%2Fcloudfront.png?id=b437b9f2-6baa-4980-843e-68a9c1614eac&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

创建分配

源域填写能测速的地址：cachefly.cachefly.net  
![coudfront2](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe327203b-db43-43a7-b15b-ea7a56c4e2a4%2Fcloudfront02.png?id=c04a1547-8ecf-43c2-b98d-d38ebc7713b8&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

协议：匹配查看器

自动压缩对象：No  
![coudfront3](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F63cdc8b2-5763-43bc-b994-5c5fae95d00f%2Fcloudfront03.png?id=08c866dc-aa12-4e51-9407-f9992faec13f&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

缓存键和源请求（选择第二项）：Legacy cache settings  
![cloudfront4](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Faebb929f-b699-4408-9738-daad458d4e76%2Fcloudfront04.png?id=fac93ebc-9c83-491b-805b-e65d0b58ec21&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

创建

复制得到的域名![cloudfront5](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F49befc04-f4cb-4522-a5ad-6b2185a1c0f5%2Fcloudfront05.png?id=79bbd8e3-3b20-4cfd-a780-a7bb4f416581&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

打开显示为测速地址即可（需要等到部署完成）

### 2.使用优选ip工具优选ip

优选IP工具：[https://github.com/XIU2/CloudflareSpeedTest](https://bulianglin.com/g/aHR0cHM6Ly9naXRodWIuY29tL1hJVTIvQ2xvdWRmbGFyZVNwZWVkVGVzdA)

测速地址：[https://cachefly.cachefly.net/100mb.test](https://bulianglin.com/g/aHR0cHM6Ly9jYWNoZWZseS5jYWNoZWZseS5uZXQvMTAwbWIudGVzdA)

CFT IP段：[https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/LocationsOfEdgeServers.html](https://bulianglin.com/g/aHR0cHM6Ly9kb2NzLmF3cy5hbWF6b24uY29tL0FtYXpvbkNsb3VkRnJvbnQvbGF0ZXN0L0RldmVsb3Blckd1aWRlL0xvY2F0aW9uc09mRWRnZVNlcnZlcnMuaHRtbA)

> 由于CFT的CDN包含了中国IP段，需要备案才能使用，需要将其全部剔除，否则优选出来延迟最低的都是中国的CDN

**下方为剔除中国CDN的IP段，官方可能会定期更新IP段，建议自己进行剔除操作**

```
205.251.249.0/24
204.246.168.0/22
18.160.0.0/15
205.251.252.0/23
54.192.0.0/16
204.246.173.0/24
54.230.200.0/21
116.129.226.128/26
130.176.0.0/17
108.156.0.0/14
99.86.0.0/16
205.251.200.0/21
13.32.0.0/15
13.224.0.0/14
70.132.0.0/18
15.158.0.0/16
13.249.0.0/16
18.238.0.0/15
18.244.0.0/15
205.251.208.0/20
65.9.128.0/18
130.176.128.0/18
54.230.208.0/20
116.129.226.0/25
52.222.128.0/17
18.164.0.0/15
64.252.128.0/18
205.251.254.0/24
54.230.224.0/19
71.152.0.0/17
216.137.32.0/19
204.246.172.0/24
18.172.0.0/15
18.154.0.0/15
54.240.128.0/18
205.251.250.0/23
52.46.0.0/18
52.82.128.0/19
54.230.0.0/17
54.230.128.0/18
54.239.128.0/18
130.176.224.0/20
36.103.232.128/26
52.84.0.0/15
143.204.0.0/16
144.220.0.0/16
54.182.0.0/16
54.239.192.0/19
18.64.0.0/14
99.84.0.0/16
130.176.192.0/19
52.124.128.0/17
204.246.164.0/22
13.35.0.0/16
204.246.174.0/23
36.103.232.0/25
204.246.176.0/20
65.8.0.0/16
65.9.0.0/17
108.138.0.0/15
64.252.64.0/18
```

下载优选IP工具后删除其他，多余文件并替换IP![cloudfront6](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe9189540-015a-4563-862f-08095d89808b%2Fcloudfront06.png?id=1c9b46ca-a368-46fa-8790-8a786dfc7208&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

文件夹内右击在终端打开，将exe文件拖入终端![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe42a1cf6-a9be-4b18-a166-56a815d794fb%2Fcloudfront07.png?id=8e215d2c-dd1c-4220-a587-f9e6c07c2e2e&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

```
 测速工具路径 -t 1 -url 你的测速地址/100m.test
```

等待ip优选完成即可![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F6cbdf936-b9f5-44b0-8409-d7110e47042e%2Fcloudfront08.png?id=e920068f-f4f4-4fb2-9e9f-4c1b2aeae471&table=block&spaceId=365285dc-da8c-4e14-bcc4-e1971dd26d06&width=2000&userId=70883a19-0ae0-43a8-86e8-f1e700bc409e&cache=v2)

### 3.cloud front分配以及使用

**再次分配**

测速完成记得删除原先分配免得被盗刷

重复第一步，唯一区别就是将源域替换为服务器ip解析的域名

等待部署完成即可

使用cloud front分配的域名，打开pach设置的路径

例如

```
例如：d104f6sfjksdie.cloudfront.net/ayy
```

显示 Bad Request，即为成功

**使用**

在伪装域名处填写cloudfront分配的域名

ip填写优选出来的ip

使用443端口，V2选择TLS即可，端口修改为443即可