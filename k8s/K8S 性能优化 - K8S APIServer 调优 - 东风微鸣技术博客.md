## 前言[](https://ewhisper.cn/posts/51187/#%E5%89%8D%E8%A8%80)

K8S 性能优化系列文章，本文为第二篇：Kubernetes API Server 性能优化参数最佳实践。

系列文章：

1.  [《K8S 性能优化 - OS sysctl 调优》](https://ewhisper.cn/posts/5019/)

## 参数一览[](https://ewhisper.cn/posts/51187/#%E5%8F%82%E6%95%B0%E4%B8%80%E8%A7%88)

kube-apiserver 推荐优化的参数如下：

1.  `--default-watch-cache-size`：默认值 100；用于 List-Watch 的缓存池；建议 1000 或更多；
2.  `--delete-collection-workers`：默认值 1；用于提升 namesapce 清理速度，有利于多租户场景；建议 10；
3.  `--event-ttl`: 默认值 1h0m0s；用于控制保留 events 的时长；集群 events 较多时建议 30m，以避免 etcd 增长过快；
4.  `--max-mutating-requests-inflight`: 默认值 200；用于 write 请求的访问频率限制；建议 800 或更高；
5.  `--max-requests-inflight`: 默认值 400；用于 read 请求的访问频率限制；建议 1600 或更高；
6.  `--watch-cache-sizes`: 系统根据环境启发式的设定；用于 pods/nodes/endpoints 等核心资源，其他资源参考 default-watch-cache-size 的设定； K8s v1.19 开始，该参数为动态设定，建议使用该版本。

EOF