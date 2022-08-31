![](https://miro.medium.com/max/1400/1*kqY65sTiETTDZwRwIODJSw.jpeg)

Photo by [Todd Diemer](https://unsplash.com/@todd_diemer) on Unsplash

At PayPal, we recently started testing the waters with Kubernetes. A majority of our workloads run on Apache Mesos, and as part of this migration, we needed to understand several performance aspects of clusters running Kubernetes with a PayPal-specific control plane. Chief amongst these aspects is understanding the scalability of the platform as well as identifying opportunities for improvement through tuning the cluster.

Unlike Apache Mesos, which can scale up to 10,000 nodes out of the box, scaling Kubernetes is challenging. Kubernetes’ scalability is not just limited to the number of nodes and pods, but several aspects like the number of resources created, the number of containers per pod, the total number of services, and the pod deployment throughput. This post describes some challenges we faced when scaling and how we solved them.

## Cluster Topology

We have clusters of varying sizes in production spanning thousands of nodes. Our setup consisted of three master nodes and an external three-node etcd cluster all running on Google Cloud Platform (GCP). The control plane was placed behind a load balancer and all data nodes belonged to the same zone as the control plane.

## Workload

For the purpose of performance testing, we used an open-source workload generator, [k-bench](https://github.com/vmware-tanzu/k-bench), modified for our use cases. The resource objects we used were simple pods and deployments. We deployed them both in batches and sequentially with varying batch sizes and inter-deployment times.

## Scale

We started with a low number of pods and a low number of nodes. Through stress testing, we found opportunities for improvement and continued to scale up the cluster as we observed improved performance. Each worker node had four CPU cores and could hold a maximum of 40 pods. We scaled to about 4,100 nodes. The application used for benchmarking was a stateless service running on 100 millicores of guaranteed Quality of Service (QoS).

We started with 2,000 pods on 1,000 nodes, followed by 16,000 pods, then 32,000 pods. After this, we jumped to 150,000 pods on 4100 nodes, followed by 200,000 pods. We had to increase the number of cores on each node to accommodate more pods.

## API Server

The API server proved to be a bottleneck when several connections to the API server returned 504 gateway timeouts, in addition to local client level throttling (exponential backoff). These increased **exponentially** during ramp-ups:

```
I0504 17:54:55.731559       1 request.go:655] Throttling request took 1.005397106s, request: POST:https://<>:443/api/v1/namespaces/kbench-deployment-namespace-14/pods..I0504 17:55:05.741655       1 request.go:655] Throttling request took 7.38390786s, request: POST:https://<>:443/api/v1/namespaces/kbench-deployment-namespace-13/pods..I0504 17:55:15.749891       1 request.go:655] Throttling request took 13.522138087s, request: POST:https://<>:443/api/v1/namespaces/kbench-deployment-namespace-13/pods..I0504 17:55:25.759662       1 request.go:655] Throttling request took 19.202229311s, request: POST:https://<>:443/api/v1/namespaces/kbench-deployment-namespace-20/pods..I0504 17:55:35.760088       1 request.go:655] Throttling request took 25.409325008s, request: POST:https://<>:443/api/v1/namespaces/kbench-deployment-namespace-13/pods..I0504 17:55:45.769922       1 request.go:655] Throttling request took 31.613720059s, request: POST:https://<>:443/api/v1/namespaces/kbench-deployment-namespace-6/pods..
```

The size of the queue which governs rate-limiting at the API Server was updated via _max-mutating-requests-inflight_ and _max-requests-inflight_. These two flags at the API server govern how the [Priority and Fairness](https://kubernetes.io/docs/concepts/cluster-administration/flow-control/) feature introduced as Beta in 1.20 divides the total queue size amongst its queue classes. For example, requests for leader election get priority over pod requests. Within each priority, there is fairness with configurable queues. There is room for further fine-tuning in the future via _PriorityLevelConfiguration_ & _FlowSchema_ API objects.

## Controller Manager

Controller Manager is responsible for providing controllers for native resources like Replica Set, Namespaces, etc., with a high number of deployments (which are managed by replica sets). The rate at which the controller manager could sync its state with the API server was limited. Multiple knobs were used to tune this behavior:

-   _kube-api-qps —_ The number of queries the controller manager could make to the API server in a given second.
-   _kube-api-burst —_ The controller manager burst, which is additional concurrent calls above _kube-api-qps._
-   _concurrent-deployment-syncs —_ Concurrency in the synchronization call for objects like deployment, replicaset, etc.

## Scheduler

When tested independently as an independent component, the scheduler can support a high throughput rate of up to 1,000 pods per second. However, upon deploying the scheduler to a live cluster, we noticed a real-world throughput reduction. A **slow etcd instance caused increased binding latencies** for the scheduler, leading to an increased pending queue size in the order of thousands of pods. The idea was to keep this number under 100 during test runs, as a higher count affects pod startup latencies. We also ended up tuning leader election parameters to be resilient against spurious restarts on short-lived network partitions or network congestion.

## etcd

etcd is the single most critical part of a Kubernetes cluster. This is evident from the plethora of issues etcd can cause all across the cluster manifested in different ways. It needed careful investigation to surface the root causes and scale etcd to handle our intended scale.

When ramping up, a lot of Raft proposals started failing:

![](https://miro.medium.com/max/1400/1*a2TwQWzWPnwRzdn0XEVj2A.png)

Through investigation and analysis, we determined that GCP throttled PD-SSD disk throughput to about 100MiB per second (as shown below) on our disk size of 100G. GCP doesn’t provide a way to increase the throughput limit — it only [increases with the size of the disk](https://cloud.google.com/compute/docs/disks). Even though etcd node only takes < 10G of space, we first tried with 1TB PD-SSD. However, even the larger disk became a **bottleneck** when all 4k nodes joined the Kubernetes control plane at once. We decided to use local SSD, which has very high throughput at the cost of a slightly higher chance of data loss in case of failures as it is [not persistent](https://cloud.google.com/compute/docs/disks/local-ssd#data_persistence).

![](https://miro.medium.com/max/1400/1*4OuITPZunPJcFA3em5-jfQ.png)

After moving to local SSD, we didn’t see the expected performance out of the fastest SSD. Some benchmarks were done directly on the disk with FIO and the numbers there were expected. However, etcd benchmarks showed a different story for concurrent all member writes:

```
LOCAL SSDSummary: Total: 8.1841 secs. Slowest: 0.5171 secs. Fastest: 0.0332 secs. Average: 0.0815 secs. Stddev: 0.0259 secs. Requests/sec: 12218.8374PD SSDSummary: Total: 4.6773 secs. Slowest: 0.3412 secs. Fastest: 0.0249 secs. Average: 0.0464 secs. Stddev: 0.0187 secs. Requests/sec: 21379.7235
```

The local SSD performed worse! After deeper investigation, this was attributed to _ext4_ file system’s write barrier cache commits. Since etcd uses write-ahead logging and calls _fsync_ every time it commits to the raft log, it’s okay to disable the write barrier. Additionally, we have DB backup jobs at the file system level and application level for DR. After this change, the numbers with local SSD improved comparable to PD-SSD:

```
LOCAL SSDSummary:  Total: 4.1823 secs.  Slowest: 0.2182 secs.  Fastest: 0.0266 secs.  Average: 0.0416 secs.  Stddev: 0.0153 secs.  Requests/sec: 23910.3658
```

The effect of this improvement was seen in etcd’s Write-Ahead Logging (WAL) sync duration and backend commit latencies, which decreased by more than 90% around the 15:55 time mark as seen below:

![](https://miro.medium.com/max/1400/1*CONYmPoRi1KmszTZXjh6kw.png)

![](https://miro.medium.com/max/1400/1*BbuL6oW-a4SdZDTdfx0zBQ.png)

The default MVCC DB size in etcd is 2 GB. This was increased to a maximum of 8 GB upon setting off out of DB space alarms. With the utilization of about **~60% of this DB**, we were able to scale up to 200k stateless pods.

With all the above optimizations the cluster was much more stable at our intended scale, however, we were far behind in SLIs for API latencies.

The etcd server would still restart occasionally and just a single restart can spoil the benchmark results, especially the P99 numbers. A closer look revealed that there is a liveness probe [bug in etcd](https://github.com/kubernetes/kubernetes/pull/97034) YAML for v1.20. A workaround was applied to fix this by increasing the failure threshold count.

After exhausting all avenues to scale etcd vertically, mainly resource-wise (CPU, memory, disk) we found that the etcd’s performance was affected with range queries. Etcd doesn’t perform well when there are many range queries and writes to the Raft log suffer, thereby slowing down the cluster’s latencies. The following are the number of range queries per Kubernetes resource that affect performance in one of the test runs:

```
etcd$ sudo grep -ir "events" 0.log.20210525-035918 | wc -l130830etcd$ sudo grep -ir "pods" 0.log.20210525-035918 | wc -l107737etcd$ sudo grep -ir "configmap" 0.log.20210525-035918 | wc -l86274etcd$ sudo grep -ir "deployments" 0.log.20210525-035918 | wc -l6755etcd$ sudo grep -ir "leases" 0.log.20210525-035918 | wc -l4853etcd$ sudo grep -ir "nodes" 0.log.20210525-035918 | wc -l
```

The etcd backend latencies were majorly impacted due to these time-consuming queries. After sharding the etcd server on the events resource, we saw an improvement in cluster stability when there is high contention of pods. In the future, there is room to further shard the etcd cluster on the pods resource. It is easy to configure the API server to contact the relevant etcd for interacting with a sharded resource.

## Results

After optimizing and tuning various components of Kubernetes, we observed a huge improvement in latencies. The following charts demonstrate the performance gain achieved over time to meet SLOs. The workload here is 150k pods total with 250 replicas per deployment on 10 concurrent workers. As long as the P99 latencies for pod startup are within five seconds, per Kubernetes SLOs, we are good.

![](https://miro.medium.com/max/1400/1*QWE90InNMb6Tzq5qp9a3qA.png)

The following chart demonstrates the API call latencies well within [SLOs](https://github.com/kubernetes/community/blob/master/sig-scalability/slos/slos.md#steady-state-slisslos) at 200k pods in the cluster.

> We also achieved P99 pod startup latencies of around **five seconds** for 200k pods at a much higher pod deployment rate than what K8s tests for 5k nodes claim at 3000 pods/min.

![](https://miro.medium.com/max/1400/1*UIwqe64oosj7hMq8XNvqNA.png)

## Conclusion

Kubernetes is a complex system and a deep understanding of the control plane is necessary to know how to scale each component. We have learned a lot through this exercise and will continue to optimize our clusters.