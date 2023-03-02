<table><tbody><tr><td><img src="https://blog.darkthread.net/img/calendar.svg"></td><td><span title="Published at 2023-01-02 02:46 PM"><time datetime="2023-01-02T06:46:44" itemprop="datePublished">2023-01-02 02:46 PM</time></span></td><td><img src="https://blog.darkthread.net/img/comment.svg"></td><td><span title="0 comments">0</span></td><td><img src="https://blog.darkthread.net/img/eye.svg"></td><td><span data-ajax-url="/blog/pageviewcount/docker-invalid-tar-header-error" title="1,056 pageviews">1,056</span></td><td><a href="https://blog.darkthread.net/Blog/docker-invalid-tar-header-error/"><img src="data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==" id="nllbtn"></a></td></tr></tbody></table>

換了新工作機，跑 Docker Desktop 是件輕鬆愉快的小事。於是我把部署到 Linux 主機的操作步驟改成：在 Windows Build Docker 容器 Image，匯出 Image 並壓縮成 tar.gz / tgz，scp 上傳後用 docker load 或 import 載入，但有時會失敗：

![](https://blog.darkthread.net/img/loading.svg)

錯誤訊息為：archive/tar: invalid tar header

查到這是 [Windows 與 Unix 的 STDOUT/STDIN 差異](https://stackoverflow.com/a/53193561/288936)造成的問題，我習慣使用 `docker save image-name > image-name.tar` 匯出 Image，Winows 匯出的檔案拿到 Linux 匯入時便會出錯。解法方法很簡單：

```
docker save image-name -o image-name.tar
docker save image-name --output image-name.tar
```

改用 -o 或 --output 參數指定輸出檔案，即可避免問題。

Windows / Linux 混合應用容易發生的小問題，解法很簡單，有遇到才會知道，留篇文章增加 Google 找到答案的機率。

Explain the issue of 'invalid tar header' error when loading docker image saved from Windows and how to avoid it.