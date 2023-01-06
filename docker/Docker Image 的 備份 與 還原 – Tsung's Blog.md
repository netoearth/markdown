Docker 平常都是備份 Dockerfile 和 docker-compose.yml 即可，但是某些套件已經無法線上安裝，就是 Dockerfile build 會失敗的話，比較好的做法就是直接把 Image 先存下來~

想要將 Docker Image 備份、還原，要怎麼做呢？

-   官方文件：[Back up and restore data | Docker Documentation](https://docs.docker.com/desktop/backup-and-restore/)

另外，若已經在 Container 做很多事情，但是無從考證起，那只好把 Container commit 然後另外備份成 Image.tar，要怎麼做呢？

### Docker 從 Image 做 備份 與 還原

備份與還原會用到：

-   [docker save | Docker Documentation](https://docs.docker.com/engine/reference/commandline/save/)
-   [docker load | Docker Documentation](https://docs.docker.com/engine/reference/commandline/load/)

Docker Image 備份

-   docker save image-name > image.tar # 挑其一即可
-   **docker image save -o image.tar image-name**
    -   docker image save -o fedora-latest.tar fedora:latest
-   **docker save image-name:latest | gzip > image\_latest.tar.gz** # 壓縮

Docker Image 還原

-   **docker load -i image.tar** # image-name
    1.  載入完後 docker images
    2.  docker run -d image-name
-   docker load --input fedora.tar
-   **docker load < busybox.tar.gz**

### Docker 從 Container commit 做 container image 備份

官方文件：[docker commit | Docker Documentation](https://docs.docker.com/engine/reference/commandline/commit/)

1.  docker ps # 找到 container id (c3f279d17e0a)
2.  **docker commit c3f279d17e0a svendowideit/testimage:version3** # 可於 commit 時 --change 修改 ENV 等參數
3.  docker images # 就會看到 svendowideit/testimage:version3
4.  docker run -d svendowideit/testimage:version3 # 就可以另外跑起來

## 文章導覽