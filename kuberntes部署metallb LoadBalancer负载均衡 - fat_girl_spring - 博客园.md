metallb简介

官方网站：https://metallb.universe.tf/  
MetalLB是使用标准路由协议的裸机Kubernetes集群的软负载均衡器，目前处于测试版本阶段。

私有云裸金属架构的kubernetes集群不支持LoadBalance  
Kubernetes没有为裸机群集提供网络负载均衡器（类型为LoadBalancer的服务）的实现，如果你的kubernetes集群没有在公有云的IaaS平台（GCP，AWS，Azure …）上运行，则LoadBalancers将在创建时无限期地保持“挂起”状态，也就是说只有公有云厂商自家的kubernetes支持LoadBalancer。  
裸机群集运营商留下了两个较小的工具来将用户流量带入其集群，“NodePort”和“externalIPs”服务。这两种选择都对生产使用产生了重大影响，这使得裸露的金属集群成为Kubernetes生态系统中的二等公民。

MetalLB旨在解决这种不平衡  
MetalLB旨在通过提供与标准网络设备集成的网络LB实现来纠正这种不平衡，以便裸机集群上的外部服务也“尽可能”地工作。即MetalLB能够帮助你在kubernetes中创建LoadBalancer类型的kubernetes服务。

Metallb基本原理  
Metallb 会在 Kubernetes 内运行，监控服务对象的变化，一旦察觉有新的LoadBalancer 服务运行，并且没有可申请的负载均衡器之后，就会完成两部分的工作：  
1.地址分配  
用户需要在配置中提供一个地址池，Metallb 将会在其中选取地址分配给服务。  
2.地址广播  
根据不同配置，Metallb 会以二层（ARP/NDP）或者 BGP 的方式进行地址的广播。  
关于裸金属集群的负载均衡方案介绍  
https://kubernetes.github.io/ingress-nginx/deploy/baremetal/  
基本原理图：  
![](https://img2020.cnblogs.com/blog/1411165/202104/1411165-20210426173159963-994373680.png)

###  部署metallb负载均衡器

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

### 创建config.yaml提供IP池

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[centos@k8s-master ~]$ vim example-layer2-config.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.92.200-192.168.92.210   #跟k8s集群节点的宿主机ip地址段保持一致
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

注意：这里的 IP 地址范围需要跟集群实际情况相对应。  
**执行yaml文件**

```
kubectl apply -f example-layer2-config.yaml
```

### 创建后端应用和服务测试

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[centos@k8s-master ~]$ vim tutorial-2.yaml 
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - name: http
          containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**查看service分配的EXTERNAL-IP**

```
[centos@k8s-master ~]$ kubectl get service 
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>           443/TCP        3d15h
nginx        LoadBalancer   10.101.112.1   192.168.92.201   80:31274/TCP   123m
```

**集群内访问该IP地址**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
[centos@k8s-master ~]$ curl 192.168.92.201
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
......
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

**从集群外访问该IP地址**

![](https://img2020.cnblogs.com/blog/1411165/202104/1411165-20210426174045126-1828966551.png)

 另外使用nodeip+nodeport也可以访问

![](https://img2020.cnblogs.com/blog/1411165/202104/1411165-20210426174111516-946821211.png)

**注：EXTERNAL-IP**相当于负载均衡对外的ip地址即客户端访问地址，**speaker**3个pod相当于客户端需要访问的后台地址，客户端通过访问**EXTERNAL-IP**，**EXTERNAL-IP**绑定宿主机地址访问到服务完成负载均衡

转载自：https://blog.csdn.net/networken/article/details/85928369