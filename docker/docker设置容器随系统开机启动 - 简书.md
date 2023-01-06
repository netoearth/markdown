## docker设置容器随系统开机启动

[![](https://upload.jianshu.io/users/upload_avatars/8777535/de5e64f5-f857-4856-96ce-dad74e426cf2?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/0b9c36ba7cc6)

[愤愤的有痣青年](https://www.jianshu.com/u/0b9c36ba7cc6)

2020.12.29 18:44:26字数 179阅读 2,149

docker 设置容器自启动很简单,只需要在容器创建时的`docker run`命令中加上  
`--restart=always`即可.  
对于已经创建的容器,可以执行`docker update --restart=always <容器名>`  
这样当容器因为异常关闭时,会自动再启动起来,但前提是该容器不是手动关闭或者运行时间不超过10s的容器.

你以为这样的完了吗?不,经过测试,当系统重启后,需要登入ssh后容器才会自动启动,经过网上找原因,发现是由于docker服务没有开机启动,所以这里需要将docker服务设置为开机启动,即  
`systemctl enable docker.service`

0人点赞

[日记本](https://www.jianshu.com/nb/18314340)

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/8777535/de5e64f5-f857-4856-96ce-dad74e426cf2?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/0b9c36ba7cc6)

[愤愤的有痣青年](https://www.jianshu.com/u/0b9c36ba7cc6 "愤愤的有痣青年")

总资产1共写了3.4W字获得13个赞共5个粉丝

### 

全部评论0只看作者

按时间倒序

按时间正序