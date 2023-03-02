Kubernetes dashboard is a web-based user interface which provides information on the state of the Kubernetes cluster resources and any errors that may occur. The dashboard can be used to deploy containerized applications to the cluster, troubleshoot deployed applications, as well as the general management of the cluster resources.

The deployment of Deployments, StatefulSets, DaemonSets, Jobs, Services and Ingress can be done from the dashboard or from the terminal with **kubectl.** if you want to scale a Deployment, initiate a rolling update, restart a pod, create a persistent volume and persistent volume claim, you can do all from the Kubernetes dashboard.

Also checkout the article on [Authenticate Kubernetes Dashboard Users With Active Directory](https://computingforgeeks.com/kubernetes-and-active-directory-integration/)

## Step 1: Configure kubectl

We’ll use the kubectl kubernetes management tool to deploy dashboard to the Kubernetes cluster. You can configure kubectl using our guide below.

-   [Easily Manage Multiple Kubernetes Clusters with kubectl & kubectx](https://computingforgeeks.com/manage-multiple-kubernetes-clusters-with-kubectl-kubectx/)

The guide in the link demonstrates how you can configure and access multiple clusters with same kubectl configuration file.

## Step 2: Deploy Kubernetes Dashboard

The default Dashboard deployment contains a minimal set of RBAC privileges needed to run. You can deploy Kubernetes dashboard with the command below.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
```

You can as well download and apply the file locally:

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
kubectl apply -f recommended.yaml
```

### Set Service to use NodePort

This will use the default values for the deployment. The services are available on ClusterIPs only as can be seen from the output below:

```
$ kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
dashboard-metrics-scraper   ClusterIP   10.104.220.36   <none>        8000/TCP   23s
kubernetes-dashboard        ClusterIP   10.108.11.22    <none>        443/TCP    25s
```

Patch the service to have it listen on NodePort:

```
kubectl --namespace kubernetes-dashboard patch svc kubernetes-dashboard -p '{"spec": {"type": "NodePort"}}'
```

Confirm the new setting:

```
$ kubectl get svc -n kubernetes-dashboard kubernetes-dashboard -o yaml
...
spec:
  clusterIP: 10.108.11.22
  clusterIPs:
  - 10.108.11.22
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30506
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

-   **NodePort** exposes the Service on each Node’s IP at a static port (the NodePort). A _ClusterIP_ Service, to which the NodePort Service routes, is automatically created.

```
$ vim nodeport_dashboard_patch.yaml
spec:
  ports:
  - nodePort: 32000
    port: 443
    protocol: TCP
    targetPort: 8443
EOF
```

Apply the patch

```
kubectl -n kubernetes-dashboard patch svc kubernetes-dashboard --patch "$(cat nodeport_dashboard_patch.yaml)"
```

Check deployment status:

```
$ kubectl get deployments -n kubernetes-dashboard                              
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
dashboard-metrics-scraper   1/1     1            1           86s
kubernetes-dashboard        1/1     1            1           86s
```

Two pods should be created – One for dashboard and another for metrics.

```
$ kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-7b64584c5c-xvtqp   1/1     Running   0          2m4s
kubernetes-dashboard-566f567dc7-w59rn        1/1     Running   0          2m4s
```

Since I changed service type to NodePort, let’s confirm if the service was actually created.

```
$ kubectl get service -n kubernetes-dashboard  
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.103.159.77   <none>        8000/TCP        8m40s
kubernetes-dashboard        NodePort    10.101.194.22   <none>        443:32000/TCP   8m40s
```

## Step 3: Accessing Kubernetes Dashboard

My Service deployment was assigned a port **32000**/**TCP**.

```
# Example
https://192.168.200.14:32000
```

Let’s confirm if access to the dashboard is working.

[![install kubernetes dashboard 01](https://computingforgeeks.com/wp-content/uploads/2020/01/install-kubernetes-dashboard-01-1024x575.png?ezimgfmt=rs:696x391/rscb23/ng:webp/ngcb23 "How To Install Kubernetes Dashboard with NodePort 1")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%221024%22%20height=%22575%22%3E%3C/svg%3E)

You need a token to access the dashboard, check our guides:

-   [How To Create Admin User to Access Kubernetes Dashboard](https://computingforgeeks.com/create-admin-user-to-access-kubernetes-dashboard/)
-   [Create Kubernetes Service / User Account restricted to one Namespace](https://computingforgeeks.com/restrict-kubernetes-service-account-users-to-a-namespace-with-rbac/)

You should see a web dashboard which looks similar to below.

[![install kubernetes dashboard 02](https://computingforgeeks.com/wp-content/uploads/2020/01/install-kubernetes-dashboard-02-1024x521.png?ezimgfmt=rs:696x354/rscb23/ng:webp/ngcb23 "How To Install Kubernetes Dashboard with NodePort 2")](data:image/svg+xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20width=%221024%22%20height=%22521%22%3E%3C/svg%3E)

**Nginx Ingress:**

```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: k8s-dashboard
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  tls:
    - hosts:
      - k8sdash.mydomain.com
      secretName: tls-secret
  rules:
    - host: k8sdash.mydomain.com
      http:
        paths:
        - path: /
          backend:
            serviceName: kubernetes-dashboard
            servicePort: 443
```

Check other Kubernetes guides:

-   [Top Minimal Container Operating Systems for running Kubernetes](https://computingforgeeks.com/minimal-container-operating-systems-for-kubernetes/)
-   [Install Production Kubernetes Cluster with Rancher RKE](https://computingforgeeks.com/install-kubernetes-production-cluster-using-rancher-rke/)
-   [Install Minikube Kubernetes on CentOS 8 / CentOS 7 with KVM](https://computingforgeeks.com/how-to-install-minikube-on-centos-linux-with-kvm/)
-   [How To Schedule Pods on Kubernetes Control plane (Master) Nodes](https://computingforgeeks.com/how-to-schedule-pods-on-kubernetes-control-plane-node/)