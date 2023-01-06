Linux 有各種 top 來查看系統的各種 loading，例如：htop、iotop、atop、glances(這個沒用 top 的名字)

而現在 Docker 盛行的時代，自然也有為 Docker Container 的 top：ctop (監控容器運行狀況專用)

這次介紹的 ctop 並不是 Debian / Ubuntu Linux 預設 apt 安裝的 ctop

-   Debian / Ubuntu Linux 預設的 ctop：[Ubuntu Manpage: ctop - Command line / text based Linux Containers monitoring tool](https://manpages.ubuntu.com/manpages/xenial/man1/ctop.1.html)
    -   這個 [ctop](https://github.com/yadutaf/ctop) 也不是不能用，這個也有為每個 container 的資源列出來，但是在那邊跳來跳去，只能按 p 暫停來觀看，不太方便

這次介紹的 ctop 有圖形化的動態顯示，點進去又有更多的選項可以操作，非常實用~

-   官方網站：[ctop：concise commandline monitoring for containers](https://ctop.sh/)
    -   GitHub：[GitHub - bcicen/ctop: Top-like interface for container metrics](https://github.com/bcicen/ctop)

### ctop 執行有幾個常用的快速鍵

-   s 選擇 cpu / mem ... 排序方式
-   f 可以 輸入關鍵字做篩選(太多 container 時，這樣子可以列在一起比較)
-   enter 可以有更多選項 (不過這用法有 stop / restart 比較危險，建議使用 ←→ 的快速鍵即可)

### ctop 安裝方式

這三種方式都可以考慮，我是習慣下載 binary 直接使用~

#### ctop 直接下載

1.  sudo wget [https://github.com/bcicen/ctop/releases/download/v0.7.7/ctop-0.7.7-linux-amd64](https://github.com/bcicen/ctop/releases/download/v0.7.7/ctop-0.7.7-linux-amd64) -O /usr/local/bin/ctop
2.  sudo chmod +x /usr/local/bin/ctop

#### ctop Docker 執行

-   docker run --rm -ti  
    \--name=ctop  
    \--volume /var/run/docker.sock:/var/run/docker.sock:ro  
    quay.io/vektorlab/ctop:latest

#### ctop 設定使用 Repository

1.  sudo apt-get install ca-certificates curl gnupg lsb-release
2.  curl -fsSL [https://azlux.fr/repo.gpg.key](https://azlux.fr/repo.gpg.key) | sudo gpg --dearmor -o /usr/share/keyrings/azlux-archive-keyring.gpg
3.  echo  
    "deb \[arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/azlux-archive-keyring.gpg\] [http://packages.azlux.fr/debian](http://packages.azlux.fr/debian)  
    $(lsb\_release -cs) main" | sudo tee /etc/apt/sources.list.d/azlux.list >/dev/null
4.  sudo apt-get update
5.  sudo apt-get install docker-ctop

## 文章導覽