©著作权归作者所有：来自51CTO博客作者a772304419的原创作品，请联系作者获取转载授权，否则将追究法律责任

> -   使用`sudo apt install com.qq.weixin.work.deepin`方式无法安装企业微信，所以用docker的方式来解决。
> -   在ubuntu22.04下只能使用`fcitx`，不然输入法无法输入中文。

```
docker run -d --name wechat --device /dev/snd --ipc host \    -v /tmp/.X11-unix:/tmp/.X11-unix \    -v $HOME/WXWork:/WXWork \    -v $HOME:/HostHome \    -v $HOME/wine-WXWork:/home/wechat/.deepinwine/Deepin-WXWork \    -e DISPLAY=unix$DISPLAY \    -e XMODIFIERS=@im=fcitx \    -e QT_IM_MODULE=fcitx \    -e GTK_IM_MODULE=fcitx \    -e AUDIO_GID=`getent group audio | cut -d: -f3` \    -e GID=`id -g` \    -e UID=`id -u` \    -e DPI=96 \    -e WAIT_FOR_SLEEP=1 \    boringcat/wechat:work1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.
```

拷贝以上代码，保存为.sh脚本，chmod +x xxx.sh 增加执行权限，执行脚本(前提是已安装好docker)

FAQ：

**企业微信关闭后如何重新打开？**

1.docker ps -a 查看所有容器进程

2.找到企业微信对应进程，拿到其对应的容器ID(CONTAINER ID列)，执行docker rm

3.重新执行上面的.sh脚本，即可重新呼出企业微信

**企业微信中接收到的文件存放位置？**

整体目录在wine-WXWork/drive\_c/users/wechat/，下面仍然会有 Desktop，Download…等文件夹，这里的文件夹就是在企业微信里点击接收的文件“另存为”时弹出的目录

-   **打赏**
-   **赞**
-   **收藏**
-   **评论**
-   **举报**

**相关文章**