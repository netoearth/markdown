<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2022-12-16 09:21 PM"><time datetime="2022-12-16T13:21:11" itemprop="datePublished">2022-12-16 09:21 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/cloudflare-cdn" title="2,643 pageviews">2,643</span></td><td><a href="https://blog.darkthread.net/Blog/cloudflare-cdn"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

上回分享 [HTTPS Nginx Docker 之懶人安裝法](https://blog.darkthread.net/blog/nginx-certbot-auto-setup/)，許多讀者大推 Cloudflare 的免費 CDN 服務，網站開 HTTP 就好，由 Cloudflare Reverse Proxy 負責啟用 TLS 憑證對瀏覽器走 HTTPS，無腦搞定。

走這條路的先決條件是你有註冊專屬網域名稱(不適用 Azure VM 提供的 \*.cloundapp.azure.com FQDN)，要將網域名稱伺服器(Domain Name Server)設為 Cloudflare 代管。網路上相關教學不少，這裡就不詳細介紹步驟，著重觀察 Cloudflare 如何實現自動 HTTPS 及 CDN 效果。

我買了一個網域名稱 dot-net.cloud 來做實驗，從網域名稱註冊廠商管理介面將名稱伺服器改成 Cloudflare 提供的主機(Cloudflare 註冊免費 CDN 服務時會提供，我被指派的是 jake.ns.cloudflare.com 及 olivia.ns.cloudflare.com)，等待大約十分鐘，便可改用 Cloudflare 的 DNS 管理介面設定及修改 DNS 名稱。介面中的 「Proxy 狀態」欄位是重點，設成「通過 Proxy 處理」時，當有人要解析網站網域名稱(例如： `www.dot-net.cloud`)對映 IP，Cloudflare DNS 會回傳其 CDN Proxy 主機的 [Anycast IP](https://blog.darkthread.net/blog/bgp-anycast/) 而非網站的真實 IP。Anycast IP 看似只有一個，Cloudflare 在全球各地放了很多台相同 IP 的主機，從美國、日本、台灣連這個 IP 會連上距離最近，速度最快的那一台，可快取的靜態檔案會直接從主機上傳回，動態內容才轉發到實際網站，CDN Proxy 主機接收到網站回應後轉送給瀏覽器。

![](https://blog.darkthread.net/img/loading.svg)

CDN Proxy 放第一線還有一個好處，客戶端查網域名稱看到的永遠是 CDN Proxy 主機的 IP，網站真實 IP 將被隱藏，減少真實主機被騷擾或攻擊風險。因此，若在 Cloudflare DNS 為 IP 另設不經 Proxy 的對映名稱，代表外界有機會藉此得知真實 IP，系統將會貼心警示：

![](https://blog.darkthread.net/Posts/files/Fig6_638067939286160269.png)

接著來看 Couldflare 好用的自動 HTTPS 功能，SSL/TLS 設定介面提供四種模式：

1.  關閉：瀏覽器到 Proxy、Proxy 到網站都走 HTTP，不加密
2.  彈性：瀏覽器到 Proxy 走 HTTPS，Proxy 到網站走 HTTP，網站完全不需設憑證，對客戶端無腦直上 HTTPS
3.  完整：瀏覽器到 Proxy 走 HTTPS(用公開有效 TLS 憑證)、Proxy 到網站也走 HTTPS，但用自簽憑證就可以了，不需花功夫申請
4.  完整(嚴格)：瀏覽器到 Proxy、Proxy 到網站都走 HTTPS，使用的是網站所安裝的 TLS 憑證，Cloundflare 無法解密傳輸資訊

第 2, 3 種方法較方便，不用花錢花功夫申請正式憑證，但缺點是 Proxy 是傳輸中間人，有能力偷看內容，一旦被突破資訊可能外洩；第 2 種做法最省事，但 Proxy 到網站段的傳輸未加密，有被攔截竊聽的風險。

![](https://blog.darkthread.net/Posts/files/Fig2_638067939288036846.png)

啟用彈性或完整模式，若是用免費方案，Cloudflare 會幫你向 Let's Encrypt 申請一張[萬用字元憑證](https://blog.darkthread.net/blog/wildcard-tls-cert-risk/) (本例為 \*.dot-net.cloud)，TLS 憑證最後還是靠 Let's Encrypt 佛心提供，但申請安裝更新的麻煩事由 Cloudflare 代勞。

![](https://blog.darkthread.net/Posts/files/Fig3_638067939289924232.png)

架了一個 ASP.NET Core 範例網站，Nginx 設定只開 HTTP：

![](https://blog.darkthread.net/Posts/files/Fig8_638067947704023442.png)

實測啟用彈性模式並設定強制轉 HTTPS，透過 CDN Proxy，瀏覽器會改走 HTTPS 並顯示為 Let's Encrypt 憑證，網站端完全不用設定：

![](https://blog.darkthread.net/Posts/files/Fig4_638067939291703862.png)

用 [https://www.iplocation.net/ip-lookup](https://www.iplocation.net/ip-lookup) 查詢 `www.dot-net.cloud` 對映的 IP 及地理位置，IP2Location、ipinfo.io 測到的 104.21.79.234 在舊金山、DB-IP 測到的 104.21.79.234 在加拿大，但都註冊在 Cloudflare 公司名下：

![](https://blog.darkthread.net/Posts/files/Fig7_638067939293649520.png)

使用 bunny.net 的 Global Latency Test 測試從全世界各角落連到 104.21.79.234 的延遲，大部分都是 0ms，驗證了 IP Anycast，也驗證了 CDN：

![](https://blog.darkthread.net/Posts/files/Fig5_638067939295979573.png)

觀察完畢。