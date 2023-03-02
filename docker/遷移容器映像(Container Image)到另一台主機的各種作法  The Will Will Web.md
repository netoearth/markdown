要遷移 Container Image 到另一台主機的方法很多，最簡單的就是自己架設 Container Registry 或使用雲端的 Container Registry 服務，只要把本機的 image 推送到遠端，就可以到另外一台電腦下載。但如果只是同事之間要遷移 Container Image 而已，或是在企業完全斷網(air-gapped)的環境下要傳輸檔案到另一台電腦，這時就有好幾種作法可以用。本篇文章將分享幾種常見的情境，告訴你怎樣可以順利的移轉容器或容器映像。

![Docker](https://user-images.githubusercontent.com/88981/210248856-c87f65c3-78a3-4a83-8c91-3a11cb4259c1.png)

### 基本的 Container Image 備份與還原

1.  將 Container Image 儲存成 `.tar` 封裝檔
    
    假設我們的 Image Tag 是 `websphere-traditional:8.5.5.16`，我們想將 Image 儲存成 `websphere-traditional_8.5.5.16.tar` 檔案。
    
    ```
    docker save -o "websphere-traditional_8.5.5.16.tar" websphere-traditional:8.5.5.16
    ```
    
    > 注意: 使用 `-o` 參數可以指定一個檔名儲存，副檔名必須使用 `.tar` 封裝檔。
    
    由於有些 image 檔案特別大，建議可以用高強度壓縮後再傳輸到另一台電腦。如果是 Windows 平台，也可以使用 [7-Zip](https://www.7-zip.org/) 將該 tar 檔壓縮成 `*.gz` 格式。由於我個人有習慣安裝 [UnxUtils](https://community.chocolatey.org/packages/unxUtils) 工具的習慣，所以在 Windows 環境下也有 Linux 常見的 CLI 工具可用，例如：`gzip`。以下命令會產生 `websphere-traditional_8.5.5.16.tar.gz` 檔案：
    
    ```
    gzip -9 "websphere-traditional_8.5.5.16.tar"
    ```
    
    > 注意: 如果區域網路的網路速度夠快，壓縮 image 反而會增加移轉的時間，此時就可以考慮完全不要壓縮！
    
2.  將 Container Image 儲存成 `.tar.gz` 壓縮檔
    
    你可以透過以下命令直接透過 `STDOUT` 將輸出導向到 `gzip` 命令，並在匯出 image 時自動壓縮：
    
    ```
    docker save websphere-traditional:8.5.5.16 | gzip -9 > "websphere-traditional_8.5.5.16.tar.gz"
    ```
    
    > 注意: 此命令絕對不能跑在 PowerShell 環境下，因為 PowerShell 的 `>` 導向輸出預設採用 `UTF16-LE` ("Unicode") 編碼，輸出的內容並非 `Raw Bytes`，所以檔案格式會壞掉。也因為 `UTF16-LE` 的關係，輸出檔案大小會比原本原本要輸出的內容大 50% 以上，這點地雷大家千萬小心。你只要使用 **Command Prompt** (`cmd.exe`) 命令提示字元環境來執行上述命令就沒問題！🔥
    
3.  將 `.tar` 或 `.tar.gz` 還原到另一台電腦的本機 Container Registry 之中
    
    ```
    docker load -i "websphere-traditional_8.5.5.16.tar.gz"
    ```
    

### 直接透過 SSH 將 Docker Image 快速移轉到另一台 Docker Engine

我會使用 `pv` 來顯示檔案傳輸進度，你可以用 `sudo apt install pv` 事先安裝好此工具。

> 以下命令僅適用於 Linux、macOS 或 WSL (Windows) 的 Shell 環境下執行。

-   從本機匯出並從遠端匯入
    
    ```
    docker save image:latest | bzip2 | pv | ssh user@host "bunzip2 | docker load"
    ```
    
-   從遠端匯出並從本機匯入
    
    ```
    ssh user@host "docker save image:latest | bzip2" | pv | bunzip2 | docker load
    ```
    
-   快速從本機匯入到 [minikube](https://minikube.sigs.k8s.io/docs/start/) 建立的 VM 內的 Docker
    
    ```
    docker save <image> | (eval $(minikube docker-env) && docker load)
    ```
    

### 快速備份本機所有 Container Images

> 以下命令僅適用於 Linux、macOS 或 WSL (Windows) 的 Shell 環境下執行。

-   快速備份
    
    ```
    ALLIMAGES=($(docker images --format '{{.Repository}}:{{.Tag}}'))
    docker save $(echo "${ALLIMAGES[*]}") | gzip -9 > /dir/images.tar.gz
    ```
    
-   快速還原
    
    ```
    tar xvf images.tar.gz -O | docker load
    ```
    
-   快速移轉到另一台電腦
    
    ```
    ALLIMAGES=($(docker images --format '{{.Repository}}:{{.Tag}}'))
    docker save $(echo "${ALLIMAGES[*]}") | pv | ssh user@host 'sudo docker load'
    ```
    

### 匯出特定 Container 的所有資料並還原成一個 Container Images

-   匯出**容器**的檔案系統內容
    
    其匯出的內容將包含所有 images 的內容，也就是匯出一個完整的檔案系統
    
    ```
    docker export CONTAINER_ID > my_container.tar
    ```
    
    > 注意: Windows 平台請勿使用 PowerShell 執行上述命令！
    
-   將**容器的檔案系統**還原成一個 Container Image
    
    ```
    docker import my_container.tar REPOSITORY:TAG
    ```
    
    > 注意: Windows 平台請勿使用 PowerShell 執行上述命令！
    

### 總結

其實 [Docker Desktop](https://www.docker.com/products/docker-desktop/) 提供的這 4 個命令有點像，因為沒有很常用，有時我也會忘記要用哪一個，所以我特別撰文整理一下。

備份 image 用以下指令：

-   `docker save`
    
    ```
    docker save -o "websphere-traditional_8.5.5.16.tar" websphere-traditional:8.5.5.16
    ```
    
    ```
    docker save websphere-traditional:8.5.5.16 | gzip -9 > "websphere-traditional_8.5.5.16.tar.gz"
    ```
    

將 image backup 還原成 image 用以下指令：

-   `docker load`
    
    ```
    docker load -i "websphere-traditional_8.5.5.16.tar.gz"
    ```
    

備份 container 用以下指令：

-   `docker export`
    
    ```
    docker export CONTAINER_ID > my_container.tar
    ```
    

將 container backup 還原成 image 用以下指令：

-   `docker import`
    
    ```
    docker import my_container.tar REPOSITORY:TAG
    ```
    

### 相關連結

-   Docker Documentation
    -   [docker save](https://docs.docker.com/engine/reference/commandline/save/)
    -   [docker load](https://docs.docker.com/engine/reference/commandline/load/)
    -   [docker export](https://docs.docker.com/engine/reference/commandline/export/)
    -   [docker import](https://docs.docker.com/engine/reference/commandline/import/)
-   [How to copy Docker images from one host to another without using a repository - Stack Overflow](https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository)
-   [Docker save images twice the size when using powershell - saving raw byte streams - Stack Overflow](https://stackoverflow.com/a/51389456/910074)
-   [Windows Docker Image 相關問題：invalid tar header](https://blog.darkthread.net/blog/docker-invalid-tar-header-error/)