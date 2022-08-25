在平时的工作中，不知道你有没有经常需要构建容器镜像进行测试，并且不一定是在构建环境中使用镜像。这时候就需要将镜像推送到镜像仓库做中转，然后在别处拉取并运行容器。久而久之，因为忘记清理镜像仓库中的“垃圾”镜像越来越多。

当然，也可以使用类似 [Harbor](https://goharbor.io/) 这种带有自动清理功能镜像仓库。但只是作为临时镜像的中转，Harbor 这种未免太重了。

今天要介绍的 [ttl.sh](https://ttl.sh/) 正适合处理这种场景。

## ttl.sh

ttl.sh 是一个匿名的临时镜像仓库，免费使用无需登录，并且已经[开源](https://github.com/replicatedhq/ttl.sh)。无需登录，镜像名称本身就提供了保密性，比如你可以使用 UUID 来作为镜像名称，使用同一个 UUID 来推送和拉取镜像。

## 使用

ttl.sh 的使用格外简单，跟平时使用 Docker Hub 或者 Docker Registry 没差别，只是 tag 的需要注意一下。

1.  `docker build` 构建镜像时通过 tag 为镜像指定有效期，比如 `ttl.sh/b0a2c1c3-5751-4474-9dfe-6a9e17dfb927:1h`。有效期默认是 1 小时，最长是 24 小时。有效的 tag 可以是 `5m`、`300s`、`4h`、`1d`，如果超过 24 小时有效期会被设置为 24 小时；如果时间格式无效，有效期设置为默认的 1 小时；
2.  使用 `docker push` 推送镜像；
3.  使用 `docker pull` 拉取镜像。

比如：

```
# macOS 下默认生成大写的 UUID，需要转成小写；Linux 下直接使用 uuidgen 即可
# docker 镜像不支持大写镜像名
$ IMAGE_NAME=$(uuidgen | tr "[:upper:]" "[:lower:]")
$ docker build -t ttl.sh/${IMAGE_NAME}:5m .
$ docker push ttl.sh/${IMAGE_NAME}:5m
```

## 实现

ttl.sh 的源码开源在 GitHub，实现也不复杂。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/05/29/ttlsh.png)

ttl.sh 基于 _Registry_ v2 的镜像仓库，利用 _Registry_ 的 [_notification_](https://docs.docker.com/registry/notifications/) 功能，将镜像的 _push event_ 发送给 _Hooks_ web 服务。

_Hooks_ 将 _event_ 中的镜像信息解析并记录在 Redis 中，主要是记录镜像的过期时间；同时有个 _Reaper_ 的定时任务定期从 Redis 获取镜像的信息，过期的镜像会调用 _Registry_ 的 REST API 进行清理。