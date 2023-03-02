## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#Preface "Preface")Preface

原本公司內部對於 docker image 的需求很簡單，就是 docker hub mirror 而已，一開始用 [VMware Harbor](https://goharbor.io/) 就可以滿足需求，但後來需求逐漸變多了，大概條列如下：

-   docker hub mirror
    
-   private docker registry
    
-   docker registry proxy of certain public registry(e.g., `quay.io`, `gcr.io`)
    

後來找到 [Sonatype Nexus Repository](https://www.sonatype.com/nexus-repository-oss) 用來解決上面的需求，而且這個不僅可以作為 docker image 的 proxy，還可以作為 npm, rpm, deb … 等軟體套件的 proxy，因此就花了點時間研究一下如何安裝使用。

## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E5%AE%89%E8%A3%9D%E9%9C%80%E6%B1%82 "安裝需求")安裝需求

基本上安裝需求很簡單，只有兩樣：

-   Java Runtime version 8
    
-   很大的硬碟空間 (為了儲存 proxy 下來的 docker image)
    

## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E5%AE%89%E8%A3%9D%E6%B5%81%E7%A8%8B "安裝流程")安裝流程

首先到[官網](https://www.sonatype.com/nexus-repository-oss)下載 Nexus3 Repository OSS，並解壓縮到較大硬碟空間的分割上，以下是這次安裝的版本：

-   官網最新版本：`nexus-3.14.0-04-unix.tar.gz`
    
-   解壓縮路徑：`/data/nexus3`
    

解壓縮後會有兩個目錄，分別是 `nexus-3.14.0-04` & `sonatype-work`

## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E8%A8%AD%E5%AE%9A-HTTPS "設定 HTTPS")設定 HTTPS

### [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E6%BA%96%E5%82%99-certificates "準備 certificates")準備 certificates

這個部份就略過了，需要的人可以去 [SSL For Free](https://www.sslforfree.com/) 申請免費且合法的憑證，比較需要注意的是 SSL For Free 僅會提供 CER 格式的 certificate & private key，因此後面的設定中會需要使用其他工具轉成其他格式。

> 假設申請的 certificate 為 wildcard certificate for `*.example.com`，檔名分別為

### [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E8%A8%AD%E5%AE%9A%E7%9B%AE%E6%A8%99 "設定目標")設定目標

在以下步驟中，我們預設要將 nexus repository 的 DNS 設定為 `nexus.example.com`，因此先建立一個可被 Nexus Repository Manager 用的 certificate，必須是 JKS(Java keystore) 的格式：

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br></pre></td><td><pre><span></span><br><span>$ openssl pkcs12 -<span>export</span> -name nexus.example.com -inkey path_to_private.key -<span>in</span> path_to_wildcards.example.com.crt -out complete_key.p12</span><br><span></span><br><span></span><br><span></span><br><span>$ keytool -importkeystore -deststorepass password -destkeypass password -destkeystore keystore.jks -srckeystore complete_key.p12 -srcstoretype PKCS12 -srcstorepass password -<span>alias</span> nexus.example.com</span><br></pre></td></tr></tbody></table>

### [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E8%A8%AD%E5%AE%9A-HTTPS-1 "設定 HTTPS")設定 HTTPS

這裡有兩個目錄變數需要先定義：

-   `$install-dir`：\*\*/data/nexus3/nexus-3.14.0-04\*\*
    
-   `$data-fir`：\*\*/data/nexus3/sonatype-work/nexus3\*\*
    

設定流程如下：

1.  將上述產生的 `keystore.jks` 複製到 `$install-dir/etc/ssl/` 中，完整路徑應該是 **$install-dir/etc/ssl/keystore.jks**
    
2.  編輯 `$data-dir/etc/nexus.properties`，新增設定 `application-port-ssl=443` (port number 可以自訂)
    
3.  編輯 `$data-dir/etc/nexus.properties`，移除 `nexus-args` 的註解，並加上 `${jetty.etc}/jetty-https.xml` (使用逗號隔開設定)
    
4.  編輯 `$install-dir/etc/jetty/jetty-https.xml`，檢視 `sslContextFactory` section，確保檔名 & 密碼跟上面設定的 certificate 是相同的 (若檔名是 **keystore.jks** & 密碼是 **password** 就可以忽略此步驟)
    

## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E5%95%9F%E5%8B%95-Nexus-Respository-Manager "啟動 Nexus Respository Manager")啟動 Nexus Respository Manager

到此為止，Nexus 還是無法直接以 HTTPS 的形式啟動，因為還需要設定 Base Url，首先要以 HTTP 啟動 Nexus：

> /data/nexus3/nexus-3.14.0-04/bin/nexus start

接著以 `admin` 的身份登入(預設密碼為 `admin123`)，進入 `System -> Capabilities`，新增 **Base Url**，並設定為 `https://nexus.example.com` 並儲存。

最後重新啟動 Nexus Repository Manager：

> /data/nexus3/nexus-3.14.0-04/bin/nexus restart

接著應該就可以用 [https://nexus.example.com](https://nexus.example.com/) 登入了!

## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#%E8%A8%AD%E5%AE%9A-Docker-Image-Mirror-amp-Proxy "設定 Docker Image Mirror & Proxy")設定 Docker Image Mirror & Proxy

這個部份就不細說了，因為下面的參考文章都寫的很清楚，基本上就是設定以下內容：

-   如果要設定 private docker registry，那就設定 docker(hosted) repository
    
-   如果要設定 docker hub mirror，那就設定 docker(proxy) repository (設定 `https://registry-1.docker.io`)
    
-   如果要設定 docker proxy，那就選 docker(proxy) repository(gcr.io 設定 `https://gcr.io`；quay.io 設定 `https://quay.io`)
    
-   如果要設定一個統一的 docker proxy 入口，則設定 docker(group) repository，並把需要的 docker mirror or proxy 加入
    

比較需要注意的是，需要到 `Security -> Realms` 中將 `Docker Bearer Token Realm` 設定為 Active，如此一來才可以使用匿名的方式存取 docker proxy。

## [](https://godleon.github.io/blog/Nexus_Repository/docker-configure-proxy-with-nexus/#References "References")References

-   [Nexus3 Repository Manager 3 > Security > Configuring SSL](https://help.sonatype.com/repomanager3/security/configuring-ssl#ConfiguringSSL-InboundSSL-ConfiguringtoServeContentviaHTTPS)
    
-   [Using Nexus OSS as a proxy/cache for Docker images – Tech by Maarten](https://mtijhof.wordpress.com/2018/07/23/using-nexus-oss-as-a-proxy-cache-for-docker-images/)
    
-   [使用nexus3.x配置docker镜像仓库及仓库代理 - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000015629878)
    
-   [Nexus - 构建并上传docker image至Sonatype Nexus | 若见喻笺](http://zhangyuyu.github.io/2018/01/09/Nexus-%E6%9E%84%E5%BB%BA%E5%B9%B6%E4%B8%8A%E4%BC%A0docker-image%E8%87%B3Sonatype-Nexus/)