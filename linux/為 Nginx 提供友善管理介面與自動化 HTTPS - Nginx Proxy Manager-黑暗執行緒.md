<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-12-17 11:04 AM"><time datetime="2022-12-17T03:04:54" itemprop="datePublished">2022-12-17 11:04 AM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/nginx-proxy-manager" title="5,998 pageviews">5,998</span></td><td><a href="https://blog.darkthread.net/Blog/nginx-proxy-manager"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

試了上回分享 [HTTPS Nginx Docker 懶人安裝法](https://blog.darkthread.net/blog/nginx-certbot-auto-setup/)時許多讀者大推的方便選擇 - [Nginx Proxy Manager](https://nginxproxymanager.com/)。

Nginx Proxy Manager 用 [tabler 樣版](https://tabler.io/) 幫 Nginx 做了一個漂亮的網頁管理介面，對初學者來說，網頁操作比編輯 .conf 檔好上手。Nginx Proxy Manager 的另一大特色是它整合了 Let's Encrypt 憑證申請，不費吹灰之力就可以將網站轉為 HTTPS。除此之外，它還提供使用者管理、權限管控、稽核記錄... 等功能。簡單來說，是個讓 Nginx 更好管理的好工具。

Nginx Proxy Manager 有提供 Docker 容器，建個 docker-compose.yml 跑 `sudo docker-compose up -d` 即可無腦使用：

```
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

管理介面在 81 Port，設定資料會保存在本機 ./data 目錄，設定資料使用 SQLite 資料庫保存，Let's Encrypt 憑證則存在 ./letsencrypt 資料夾。

![](https://blog.darkthread.net/Posts/files/Fig1_638068430868279703.png)

使用瀏覽器連上 81 Port，一切順利的話會看到登入晝面，初始登入帳號、密碼為 `admin@example.com` 及 `changeme`，第一次登入後會被要求修改帳號及設定密碼：

![](https://blog.darkthread.net/Posts/files/Fig2_638068430870132477.png)

建立 Reverse Proxy 的操作程序如下：

先輸入要綁定的網域名稱。我在同台機器跑了 ASP.NET Core 範例網站容器聽 VM 本機的 5000 Port，Forward Hostname / IP 建議填 VM 本機 IP 較省事(本例為 10.3.0.4)，若要走 localhost/127.0.0.1 需修改 Nginx Proxy Manager 容器走 [host 網路模式](https://docs.docker.com/network/host/)。

![](https://blog.darkthread.net/img/loading.svg)

Custom location 可設定子網站：[使用 Nginx 將 ASP.NET Core 掛為子網站](https://blog.darkthread.net/blog/aspnetcore-as-nginx-subdir/)

![](https://blog.darkthread.net/img/loading.svg)

SSL 部分勾選設定並輸入 Email 就好了。

![](https://blog.darkthread.net/img/loading.svg)

原本 conf 檔常會有些 Proxy Header 設定需求，可以填在 Advanced：

![](https://blog.darkthread.net/img/loading.svg)

你可以在 data/nginx/proxy\_host/\*.conf 找到 Proxy 設定，如果有手工設定過 Nginx 設定檔，對它內容應該不陌生，所有操作設定最後要轉成 conf 形式。

![](https://blog.darkthread.net/img/loading.svg)

不出意外的話，這樣就能輕鬆用 HTTPS 連上網站了。

![](https://blog.darkthread.net/img/loading.svg)

另外還有稽核記錄可查看設定修改記錄，十分貼心。

![](https://blog.darkthread.net/img/loading.svg)

最後說說我的個人看法。Nginx Proxy Manager 很適用合日常管理或 Nginx 初學上手，提供了非常棒的使用體驗，但由於它需要多跑一個管理網站，會多耗用一些主機資源，用在雲端省錢小 VM 會增加非必要性負擔。另外，網頁管理介面多少會增被攻擊的表面積，需限制存取謹慎管理。

基於以上考量，在正式環境我應該還是會回歸用設定檔設定，節省主機資源，至於 Nginx 複雜難以駕御就歸因於管理者需提升本職學能，一切以精簡優先。