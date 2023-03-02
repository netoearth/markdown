[![Velero](https://pic-cdn.ewhisper.cn/img/2022/05/25/a3ddf88245393277760fcd39dbe3ccb5-Velero.svg)](https://pic-cdn.ewhisper.cn/img/2022/05/25/a3ddf88245393277760fcd39dbe3ccb5-Velero.svg "Velero")

| 考量维度 | 基于 CSI 快照 | 基于 Restic 文件复制 |
| --- | --- | --- |
| 应用性能影响 | 低，CSI 接口调用存储系统快照 | 取决于数据量，占用额外资源 |
| 数据可用性 | 依赖于存储系统 | 对象存储和生产环境隔离，独立可用性，支持跨站点可用性 |
| 数据一致性 | 支持 Crash Consistency，配合 hook 机制实现一致性 | 无保障，基于 hook |
|  |  |  |

### 最佳实践[](https://ewhisper.cn/posts/22436/#%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%20-2)

高频本地快照 + 低频 restic 备份到 s3

从应用角度选择合适的备份粒度和备份策略

多集群环境中共享同一对象存储时要防止冲突

### 坑[](https://ewhisper.cn/posts/22436/#%E5%9D%91)

删除长时间未完成的备份或恢复任务，会导致 velero 阻塞无法处理后续任务

### QA[](https://ewhisper.cn/posts/22436/#QA)

#### velero 快照和企业存储提供的快照 (比如 netapp) 的对比?[](https://ewhisper.cn/posts/22436/#velero-%20%E5%BF%AB%E7%85%A7%E5%92%8C%E4%BC%81%E4%B8%9A%E5%AD%98%E5%82%A8%E6%8F%90%E4%BE%9B%E7%9A%84%E5%BF%AB%E7%85%A7%20-%20%E6%AF%94%E5%A6%82%20-netapp-%20%E7%9A%84%E5%AF%B9%E6%AF%94)

答：相比企业级快照，Velero 是可以从应用角度来实现做快照；

另外备份到 s3 的话，可以通过 hook 实现一致性。

推荐的一种最佳实践: 先做快照，然后凌晨 后台 把快照做 s3 数据的复制。

## 系列文章[](https://ewhisper.cn/posts/22436/#%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%20-9)

-   [Velero 系列文章](https://ewhisper.cn/tags/Velero/)

## 📚️参考文档[](https://ewhisper.cn/posts/22436/#%F0%9F%93%9A%EF%B8%8F%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%20-9)

-   [Velero Docs - Overview](https://velero.io/docs/v1.8/index.html)