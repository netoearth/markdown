深度應用 Docker 容器後常會走到這一步 - 自建 Docker Registry (容器 Image 倉庫)。

不管用 docker 指令或跑 Docker Compose，建立容器都需先載入 Docker Image，若是公開 Image，透過 docker pull 指令、在 docker-compose.yml 指定 image 來源、或 Dockerfile 寫 FROM imageName:tag，都可從 [Docker Hub](https://www.docker.com/products/docker-hub/)下載，私有程式或內部應用系統不適合上傳第三方系統，企業內部主機則可能位於無法連 Internet 的環境，從 Docker Hub 下載這條路便不可行。簡單做法是用 `docker save container-name > container-name.tar` 再 scp 上傳到 Docker 主機用 `docker load -i container-name.tar` 載入，但儲存、上傳、載入(有時還需要壓縮解壓縮)程序複雜，若要部署多台主機格外沒效率。因此，建立私有 Docker Registry 取代 Docker Hub 的角色，是較省事有效率的實務做法。  
(補充：關於 Container Image 部署，可參考保哥這篇[遷移容器映像(Container Image)到另一台主機的各種作法](https://blog.miniasp.com/post/2023/01/02/How-to-Move-Container-Image-to-another-Docker-Engine))

Docker 支援自建私有 Registry 伺服器，不意外地有安裝成 Docker 容器的便捷做法。經簡單設定與安裝，架一台私有 Docker Registry 儲存非公開容器 Image 並不困難。說是一回事，做是一回事，這篇來實地演練一次。

參考資料：

-   [Deploy a registry server by docker docs](https://docs.docker.com/registry/deploying/)
-   [使用 Private Registry 分享 image by Miles/iThome 鐵人賽](https://ithelp.ithome.com.tw/articles/10248854)

首先準備一台 Docker 主機，裝好 Nginx + Certbot 服務，我是在 Azure 開一個 B1s 小 VM 來跑。(參考：[HTTPS Nginx Docker 之懶人安裝法](https://blog.darkthread.net/blog/nginx-certbot-auto-setup/))

註：Azure 有 [Azure Container Registry](https://azure.microsoft.com/zh-tw/products/container-registry?WT.mc_id=DOP-MVP-37580) 的現成雲端服務，做法更簡便。學習在 Linux 跑 Docker 做法是為將來在離線環境建立 Docker Registry 做準備。

以我的標準，即便內部伺服器也要控管存取身分，不能讓閒雜人等隨便用。Docker Registry 支援帳號密碼登入，但前題要啟甪 TLS HTTPS 連線，故除了安裝 htpasswd 工具設定密碼雜湊檔，並需要安裝 mkcert 建立 TLS 憑證：

```
sudo apt-get install apache2-utils mkcert
```

我在主機端建了一個 /var/registry 資料夾放 Docker Registry 相關檔案，接著用 mkcert 建立自簽憑證給 Registry 用：

![](https://blog.darkthread.net/Posts/files/2023/Fig1_638094716259010929.png)

設定帳號密碼：

![](https://blog.darkthread.net/Posts/files/2023/Fig2_638094716260952346.png)

準備好 Docker Compose 設定檔，REGISTRY\_HTTP\_TLS\_CERTIFICATE 及 REGISTRY\_HTTP\_TLS\_KEY 環境變數指向 mkcert 剛才製作的兩個 pem 檔名、REGISTRY\_AUTH\_HTPASSWD\_PATH 指向密碼檔名；volumes 部分將 Docker 的 Image 資料、憑證、認證資料對應到 /var/registry 實體路徑保存，以免容器關閉後資料消失：

```
version: "3"
services:
  registry:
    restart: always
    image: registry:2
    ports:
      - 5000:5000
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/localhost.pem
      REGISTRY_HTTP_TLS_KEY: /certs/localhost-key.pem
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    volumes:
      - /var/registry/data:/var/lib/registry
      - /var/registry/certs:/certs
      - /var/registry/auth:/auth
```

用 docker-compose up -d 啟動容器，若一切正常，用`curl https://localhost:5000/v2/`測試，若傳回`{"errors":[{"code":"UNAUTHORIZED","message":"authentication required","detail":null}]}`就代表成功了。

接著來測試 Push Image 到私有 Registry，操作程序是用 docker tag 為己載入容器加上 `localhost:5000/aspnetapp` 這種標籤，此時等同同一個 IMAGE ID 有兩個 Tag，由於 Registry 需要登入，用剛才的設定帳號密碼以指令 docker login 登入，接著 `docker push localhost:5000/aspnetsapp` 就能完成上傳：

![](https://blog.darkthread.net/Posts/files/2023/Fig3_638094716264480901.png)

檢查資料夾 /var/registry/data/docker/registry/v2/repositories/aspnetapp/\_layers/sha256 可看見上傳過程出現的四個 Layer ID，成功!

![](https://blog.darkthread.net/Posts/files/2023/Fig4_638094716266415706.png)

接下來試試從 Windows Docker Desktop 從 VM 的對外網域名稱下載容器 Image 執行，一樣輕鬆秒殺：

![](https://blog.darkthread.net/Posts/files/2023/Fig5_638094716268462984.png)

就醬，一台自建 Docker Registry 便上線服役囉~

Tutorial of how to builld a private docker registry and upload private container to it.