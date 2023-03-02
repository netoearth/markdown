<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2023-01-18 11:13 PM"><time datetime="2023-01-18T15:13:55" itemprop="datePublished">2023-01-18 11:13 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/redis-first-time" title="1,882 pageviews">1,882</span></td><td><a href="https://blog.darkthread.net/Blog/redis-first-time/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

[i5 迷你工作機](https://blog.darkthread.net/blog/deskmini-i5/)上線，原本 CPU 太鳥、RAM 太少、SSD 不夠快等限制一掃而空，加上對 Docker、Hyper-V 逐漸上手，以前沒機會實際摸過的新玩意，花幾分鐘便能在我的電腦上與我相見歡。一則以喜一則以憂，能阻止我學東西的硬體限制變少了，但再也無法以沒環境沒機會學當成推拖藉口...

終於，我摸到大名鼎鼎的光速資料庫 Redis 了!

與 Redis 的第一次接觸，我打算用 Docker 架台 Redis 伺服器，學習使用管理工具查詢，最後寫支 .NET 程式讀寫資料結束第一回合。

到現在才開始看 Redis 起步算晚，但好處是學習資源豐富，我主要參考以下文章：

-   [使用 Docker 安裝 Redis yb m@rcus](https://marcus116.blogspot.com/2019/02/how-to-run-redis-in-docker.html)
-   [Redis 管理工具 - Another Redis Desktop Manager by m@rus](https://marcus116.blogspot.com/2020/04/tool-redis-another-redis-desktop-manager.html)
-   [建立 .NET 6 + Redis 本機開發環境 by 余小章](https://dotblogs.com.tw/yc421206/2022/10/29/how_to_create_redis_net_6_local_development_environment)
-   [docker-compose 安装redis by longxin111](https://blog.51cto.com/u_6192297/3299825)
-   [StackExchange.Redis Basic Usage](https://stackexchange.github.io/StackExchange.Redis/Basics.html)

以下是我在 Windows Docker Desktop 安裝 Redis 及測試的過程記錄。

1.  Docker 容器的資料關閉後會消失，故要對映到本機磁碟才能永久保存。先建立兩個目錄放設定檔及資料，我建了 X:\\dockers\\redis，其下加入 conf 及 data 子目錄。
2.  從 `http://download.redis.io/redis-stable/redis.conf` 下載範例設定檔 redis.conf 放在 X:\\dockers\\redis\\conf\\redis.conf
3.  我只改了三處，拿掉 bind 127.0.0.1 允許非本機連線，data 由 ./ 改為 /data、requirepass 設了密碼  
    ![](https://blog.darkthread.net/Posts/files/2023/Fig1_638096516492929422.png)
4.  建立 x:\\dockers\\redis\\docker-compose.yml，[redis Docker Image](https://hub.docker.com/_/redis) 的標準版 OS 使用 Debian，我則選 OS 只有 5MB 的 alpine 版，更省空間(缺點是功能較少，像是互動環境沒有 bash，只有 sh，但用到機會不多影響不大)。volumes 要將 /data 跟 /etc/redis/redis.conf 對映到 X:\\dockers\\redis 下的目錄與檔案：
    
    ```
     version: '3.8'
     services:
       myredis:
         container_name: myredis
         image: redis:7.0.7-alpine
         restart: always
         ports:
           - 6379:6379
         privileged: true
         command: redis-server /etc/redis/redis.conf --appendonly yes
         volumes:
           - /x/dockers/redis/data:/data
           - /x/dockers/redis/conf/redis.conf:/etc/redis/redis.conf
    ```
    
5.  執行 `docker-compose up -d` 啟動 Redis 伺服器
6.  用 `docker -it` 登入 Docker 容器跑 Redis-cli ping 測試  
    ![](https://blog.darkthread.net/Posts/files/2023/Fig2_638096516494853590.png)
7.  執行 `winget install qishibo.AnotherRedisDesktopManager` 安裝 GUI 管理工具  
    ![](https://blog.darkthread.net/Posts/files/2023/Fig3_638096516496741780.png)
8.  建立 .NET 6 Console 專案，參照 StackExchange.Redis 及 MSTest.TestFramework (借用 Assert API)
    
    ```
    dotnet new console -o redis-demo
    cd redis-demo
    dotnet add package StackExchange.Redis 
    dotnet add package MSTest.TestFramework
    ```
    
    Program.cs 程式如下：
    
    ```
    using Microsoft.VisualStudio.TestTools.UnitTesting;
    using StackExchange.Redis;
    
    var redis = ConnectionMultiplexer.Connect("127.0.0.1:6379,password=P@ssW0rd");
    var db = redis.GetDatabase();
    // 一般字串保存    
    db.StringSet("name", "Jeffrey");
    Assert.AreEqual("Jeffrey", (string)db.StringGet("name")!);
    // 設定 10 分鐘後過期
    db.StringSet("expireIn10Min", "darkthread", TimeSpan.FromMinutes(10));
    Assert.AreEqual("darkthread", (string)db.StringGet("expireIn10Min")!);
    // 設定 1 秒後過期
    db.StringSet("expireIn1Sec", "almost gone", TimeSpan.FromSeconds(1));
    Assert.IsNotNull((string)db.StringGet("expireIn1Sec")!);
    Thread.Sleep(2000);
    // 過期後應為 null
    Assert.IsNull((string)db.StringGet("expireIn1Sec")!);
    // 測試訂閱與發佈
    var sub = redis.GetSubscriber();
    var received = false;
    sub.Subscribe("testChannel", (channel, message) =>
    {
        Console.WriteLine($"channel: {channel}, message: {message}");
        received = true;
    });
    Assert.IsFalse(received);
    sub.Publish("testChannel", "hello");
    Thread.Sleep(1000);
    Assert.IsTrue(received);
    
    Console.WriteLine("test ok");
    ```
    

通過測試：

![](https://blog.darkthread.net/Posts/files/2023/Fig4_638096516498874452.png)

使用 Another Redis Desktop Manager 也能看到寫入資料，成功!

![](https://blog.darkthread.net/Posts/files/2023/Fig5_638096516500793440.png)

Tutorial of setting up Redis Docker container, installing GUI tool and writing C# program to do basic test.