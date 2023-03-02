Hosting and running your applications in-cloud has many benefits, such as scalability and not having to manage your own server's infrastructure. However, the benefits of using Kubernetes come with the costs of deploying computing resources. Pods need enough memory and computing resources to operate optimally. Implementing resource requests and limits is vital to reducing costs and maintaining good resource consumption metrics.

Kubernetes resource limits are crucial for making sure that the cluster functions optimally. Resource limits help administrators keep a cluster healthy by preventing pods from overconsuming CPU, memory, and temporary storage.

In this tutorial, you will learn how to monitor resource consumption and implement the resource request and limits mechanism, which prevents resource overconsumption.

## Prerequisites

You must install [Kubectl](https://www.vultr.com/docs/beginners-guide-to-kubernetes-using-kubectl/) and have a running cluster to follow this guide.

Resource requests specify the amount of memory and CPU a container needs to run. Kubelet will schedule the container on a node that has enough resources. Without resource limits, the container will consume more memory and CPU. You must set resource limits to prevent a container from over-consuming resources and starving other containers. Kubelet will ensure that containers do not exceed the set memory limits.

With resource limits, pods cannot use more CPU or memory than you allow. CPU and memory limits prevent one pod from using all available resources. If you scan a pod that does not have resource requests and limits using a Kubernetes misconfiguration tool like [Datree](https://www.vultr.com/docs/how-to-avoid-kubernetes-misconfiguration-with-datree-on-windows/) you will get the following similar results:

```
    âŒ  Ensure each container has a configured CPU limit  [1 occurrence]
        - metadata.name: pod-example (kind: Pod)
    ðŸ’¡  Missing property object `limits.cpu` - value should be within the accepted boundaries recommended by the organization

    âŒ  Ensure each container has a configured CPU request  [1 occurrence]
        - metadata.name: pod-example (kind: Pod)
    ðŸ’¡  Missing property object `requests.cpu` - value should be within the accepted boundaries recommended by the organization
```

## Resource Consumption Monitoring

To set the proper resource limits and requests, you have to monitor the rate at which containers consume memory so that you don't set resource limits too high or low.

There are various Kubectl commands to fetch resource consumption metrics. Use the following command to view how objects consume resources available in a specific node:

```
    kubectl describe node [enter the name of your node here]
```

You will get the following similar output:

```
      Namespace                   Name                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
      ---------                   ----                                         ------------  ----------  ---------------  -------------  ---
      earthly                     boemo-app-59d4567b6b-kkcs7                   100m (5%)     100m (5%)   128M (0%)        256M (1%)      7d22h
      earthly                     nginx-78449c65d4-qfzlm                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         7d22h
      earthly                     pod-example                                  250m (12%)    500m (25%)  64Mi (0%)        128Mi (1%)     12d
      kube-system                 coredns-64897985d-cc9s9                      100m (5%)     0 (0%)      70Mi (0%)        170Mi (1%)     30d
      kube-system                 etcd-minikube                                100m (5%)     0 (0%)      100Mi (0%)       0 (0%)         30d
      kube-system                 kube-apiserver-minikube                      250m (12%)    0 (0%)      0 (0%)           0 (0%)         30d
      kube-system                 kube-controller-manager-minikube             200m (10%)    0 (0%)      0 (0%)           0 (0%)         30d
      kube-system                 kube-proxy-7q5bp                             0 (0%)        0 (0%)      0 (0%)           0 (0%)         30d
      kube-system                 kube-scheduler-minikube                      100m (5%)     0 (0%)      0 (0%)           0 (0%)         30d
      kube-system                 metrics-server-6b76bd68b6-87rfn              100m (5%)     0 (0%)      300Mi (2%)       0 (0%)         30d
      kube-system                 storage-provisioner                          0 (0%)        0 (0%)      0 (0%)           0 (0%)         30d
      kubernetes-dashboard        dashboard-metrics-scraper-58549894f-n7fqd    0 (0%)        0 (0%)      0 (0%)           0 (0%)         13d
      kubernetes-dashboard        kubernetes-dashboard-ccd587f44-mcz98         0 (0%)        0 (0%)      0 (0%)           0 (0%)         13d
    Allocated resources:
      (Total limits may be over 100 percent, i.e., overcommitted.)
      Resource           Requests        Limits
      --------           --------        ------
      cpu                1200m (60%)     600m (30%)
      memory             687939584 (5%)  568475648 (4%)
      ephemeral-storage  0 (0%)          0 (0%)
      hugepages-1Gi      0 (0%)          0 (0%)
      hugepages-2Mi      0 (0%)          0 (0%)
```

You can also use the `kubectl top` command to view node and pod resource metrics. For example, use the following command to get node resource metrics:

```
    kubectl top node
```

You will get the following output:

```
    NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
    minikube   598m         29%    1129Mi          9%
```

## How to Implement Resource Limits

When creating a Pod, add the request and limit maps as they contain the key-pair values for the memory and CPU limits and requests. Make sure that you add optimum values for the requests and resources key-value pairs. Here is a snippet of the resource limits and request mechanism:

```
    spec:
      containers:
      - name: app
        image: hashicorp/http-echo:0.2.3
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

Here is a full pod configuration manifest that has the resource limit and requests implemented:

```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-example
      namespace: earthly
    spec:
      containers:
      - name: app
        image: gcr.io/google-samples/gb-frontend:v4
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        serviceaccount: kubernetes-service-accounts
        ports:
        - containerPort: 8080
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
```

### Setting Resource Limits for Ephemeral Storage

Ephemeral storage resource limits and requests are also specified in the resources map. After adding the resource limits and request, make sure that you add the `volumeMount` and the `Volumes` fields as shown in the pod below:

```
    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend
    spec:
      containers:
      - name: app
        image: hashicorp/http-echo:0.2.3
        resources:
          requests:
            ephemeral-storage: "2Gi"
          limits:
            ephemeral-storage: "4Gi"
        volumeMounts:
        - name: ephemeral
          mountPath: "/tmp"
      - name: log-aggregator
        image: images.my-company.example/log-aggregator:v6
        resources:
          requests:
            ephemeral-storage: "2Gi"
          limits:
            ephemeral-storage: "4Gi"
        volumeMounts:
        - name: ephemeral
          mountPath: "/tmp"
      volumes:
        - name: ephemeral
          emptyDir: {}
```

## How to Implement Limit Ranges

Starting from Kubernetes 1.10, limit ranges have been enabled by default. Limit Ranges dictate how much memory and CPU containers are allowed to consume. Unlike the resource limit mechanism, which is implemented on a specific pod, limit ranges are applied to all pods being applied to your cluster or namespace at once. Limit ranges specify:

-   The maximum amount of CPU and memory that all containers within a pod can consume.
-   The minimum amount of CPU and memory consumed across all containers within a pod.

When you use limit ranges, you have to state the type of Kubernetes object being limited, such as pods. Create a YAML file and add the following contents that will create a limit range:

```
    apiVersion: "v1"
    kind: "LimitRange"
    metadata:
      name: "resource-limits"
    spec:
      limits:
        - type: "Pod"
          max:
            cpu: "500m"
            memory: "750Mi"
          min:
            cpu: "10m"
            memory: "5Mi"
        - type: "Container"
          max:
            cpu: "500m"
            memory: "750Mi"
          min:
            cpu: "10m"
            memory: "5Mi"
          default:
            cpu: "100m"
            memory: "100Mi"
          defaultRequest:
            cpu: "20m"
            memory: "20Mi"
        - type: "PersistentVolumeClaim"
          min:
            storage: "1Gi"
          max:
            storage: "50Gi"
```

Use the following command to apply the above limitrange to your cluster:

```
    kubectl apply -f resource-limits.yaml
```

You will get the following output:

```
    limitrange/resource-limits created
```

To check if the limit range has been applied, use the describe keyword to get the details:

```
    kubectl describe limitrange resource-limits
```

You will get the following output:

```
    Name:                  resource-limits
    Namespace:             earthly
    Type                   Resource  Min  Max    Default Request  Default Limit  Max Limit/Request Ratio
    ----                   --------  ---  ---    ---------------  -------------  -----------------------
    Pod                    cpu       10m  500m   -                -              -
    Pod                    memory    5Mi  750Mi  -                -              -
    Container              cpu       10m  500m   20m              100m           -
    Container              memory    5Mi  750Mi  20Mi             100Mi          -
    PersistentVolumeClaim  storage   1Gi  50Gi   -                -              -
```

Use the following command to delete an active limitrange:

```
    kubectl delete limitrange resource-limits
```

## Learn More

To learn more about limit ranges, resource requests, and limits see:

-   [Resource Management for Pods and Containers | Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
-   [Limit range documentation](https://kubernetes.io/docs/concepts/policy/limit-range/)