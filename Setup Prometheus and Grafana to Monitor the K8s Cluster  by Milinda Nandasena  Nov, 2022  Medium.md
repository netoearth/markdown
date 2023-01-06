Hi guys 👋, in this article I’ll show you how to install Prometheus and Grafana on your K8s cluster

![](https://miro.medium.com/max/842/1*YACFHEnfQBYChIPExOZ0qA.png)

Monitoring is a key part of SRE (Site Reliability Engineering) and DevOps. It is more important when considering environments like Kubernetes and Microservices because engineers must monitor critical information about the platform’s performance and health.

Pod resources, memory usage, CPU utilization, network bandwidth, disk pressure, and other parameters must be monitored on a regular basis. Kubernetes does not self-monitor. In this case, there should be some mechanism in place to detect problems early on.

It is not a practical solution to monitor the thousands of applications manually. Prometheus and Grafana come at this point.

So, Prometheus and Grafana are heavily used by SRE/DevOps engineers because they allow them to set up alerts, live views, and custom dashboards in addition to collecting performance metrics.

If you want to learn more about Prometheus and Grafana, check out their official documentation.

-   [https://prometheus.io/](https://prometheus.io/)
-   [https://grafana.com/](https://grafana.com/)

There are two prerequisites you must meet before starting this.

**Prerequisites**:

1.  **Kubernetes cluster** — We have already built our cluster.

![](https://miro.medium.com/max/842/1*ay2C-x_l2eoo-SOVcOxtqw.png)

I’ll use the previously created K8s cluster for this setup. In my previous article ([https://milindasenaka96.medium.com/setup-your-k8s-cluster-with-aws-ec2-3768d78e7f05](https://milindasenaka96.medium.com/setup-your-k8s-cluster-with-aws-ec2-3768d78e7f05)), I used AWS EC2 to set up the K8s cluster.

So, if you read that document and then come back here, you will have a better understanding of the setup that we created.

![](https://miro.medium.com/max/842/1*LbkNTXW7jE0Dq67HJqJMFQ.png)

2\. "helm" **installation —** Using the **_helm_**, the monitoring stack can be installed with a single command line. Install helm on your master node with the command below.

```
snap install helm --classic
```

Output:

![](https://miro.medium.com/max/842/1*HD1YC0IIdkss9Y4BxGgXzA.png)

Now you know the prerequisites for the setup.

So, we can start, right? Let’s get started!

For better understanding, I will divide the article into a few sections.

1.  Deploying the **_kube-prometheus-stack_** Helm Chart
2.  Accessing the Prometheus UI
3.  Accessing the Grafana Dashboard

1.  **Deploying the _kube-prometheus-stack_ helm chart**

The **_kube-prometheus-stack_** is a collection of Kubernetes manifests, Prometheus, Grafana, Alertmanager, Prometheus operator, Prometheus rules, and other monitoring resources. Since all the mandatory features are included here, this provides easy E2E Kubernetes cluster monitoring. ([https://artifacthub.io/packages/helm/prometheus-worawutchan/kube-prometheus-stack](https://artifacthub.io/packages/helm/prometheus-worawutchan/kube-prometheus-stack))

**Step 1 — Create the namespace — _monitoring_**

The first step is to create a new namespace in the Kubernetes cluster. It will assign a location in our Kubernetes cluster for the Prometheus and Grafana services to be deployed. So, run the command below.

Output:

![](https://miro.medium.com/max/842/1*ShgG9GCk20X7tKSqFQxSIA.png)

**Step 2 — Installing _kube-prometheus-stack_ with Helm**

-   Next, run the below command to add the Prometheus-community helm repo.

```
helm repo add prometheus-community https://prometheus community.github.io/helm-charts
```

-   Next, run the below command after adding the Helm repo to deploy the **_kube-prometheus-stack_** Helm chart.

```
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```

-   To confirm and check your **_kube-prometheus-stack_** deployment, use the command below.

```
kubectl get pods -n monitoring
```

As you can see, the below image is shown each component in the stack running in your cluster.

Output:

![](https://miro.medium.com/max/842/1*8T0NOSPRU0pUBlRxM0YDlA.png)

-   To view all the services deployed in the monitoring namespace, run the below command

```
Kubectl get svc -n monitoring
```

And the output is shown below,

![](https://miro.medium.com/max/842/1*AaJOD3MPSlP_nJs1_OKtFA.png)

Our monitoring stack with Prometheus and Grafana is up and ready!

**2\. Accessing the Prometheus UI**

Now, you have successfully deployed your Prometheus services to the cluster, and your Kubernetes cluster is almost ready for monitoring.

How can you access your Prometheus dashboard, though? The “Type” of the following service must be changed from **ClusterIP** to **NodePort**.

**— prometheus-kube-prometheus-prometheus**

For that, run the below command.

```
kubectl --namespace monitoring patch svc prometheus-kube-prometheus-prometheus -p '{"spec": {"type": "NodePort"}}'
```

Output:

![](https://miro.medium.com/max/842/1*bBgTuvyJ_SA2ijwjb1Z-Dw.png)

-   Open your web browser and navigate to the URLs below to access Prometheus UI.

[http://YOUR-MASTER-NODE-PUBLIC-IP:NodePort](https://milindasenaka96.medium.com/setup-prometheus-and-grafana-to-monitor-the-k8s-cluster-e1d35343d7a9http://YOUR-MASTER-NODE-PUBLIC-IP:NodePort)

Ex: **http:// 18.143.107.40:31187**

If your Prometheus service is working, your web browser should display the following page.

![](https://miro.medium.com/max/842/1*Qhk5MaY2bmUhy-Ldk-FVaw.png)

-   Now you can navigate to this path **Status > Targets,** you will see the configured monitoring components which are Prometheus Targets

![](https://miro.medium.com/max/842/1*5qWTUrrR15slwBFjgNIznQ.png)

**3\. Accessing the Grafana Dashboard**

When we experience the visualization capabilities of the Prometheus, we can see that they are limited, as we are stuck to the Graph option. Prometheus is useful for gathering metrics from targets that have been configured with it.

But for the resource monitoring and visualization, we have to configure Prometheus with Grafana, and both work well together

Grafana was automatically installed and configured during the **_kube-prometheus-stack_** helm deployment, so you can configure Grafana access on your Cluster.

To access your Grafana dashboard,

-   Run the following command to change the “Type” of the **_prometheus-grafana_** service from **_ClusterIP_** to **_NodePort_**.

```
kubectl --namespace monitoring patch svc prometheus-grafana -p '{"spec": {"type": "NodePort"}}'
```

Output:

![](https://miro.medium.com/max/842/1*huswSjWdoEjG-ZVguDwDNQ.png)

-   Open your web browser and navigate to the URLs below to access the Grafana dashboard.

[http://YOUR-MASTER-NODE-PUBLIC-IP:NodePort](https://milindasenaka96.medium.com/setup-prometheus-and-grafana-to-monitor-the-k8s-cluster-e1d35343d7a9http://YOUR-MASTER-NODE-PUBLIC-IP:NodePort)

Ex: **http:// 18.143.107.40:32334**

If your Grafana service is working, your web browser should display the following login page.

![](https://miro.medium.com/max/563/1*OLk8VZ7O1t7CqyrZjuqd7A.png)

The default.

— Username: **admin**

— Password: **prom-operator**

-   After logging in, you will see the Grafana dashboard, as shown below.

![](https://miro.medium.com/max/842/1*7vJKByoP5qLn_m9BsrDzhg.png)

**3.1** **Interacting with Grafana**

By default, the **_kube-prometheus-stack_** deploys Grafana with some pre-configured dashboards for each target configured in Prometheus.

When you select “Browse” from the dashboard icon, your browser brings you to a page with a list of dashboards.

![](https://miro.medium.com/max/488/1*nseFkyLchasm4oZla9_ssg.png)

Below is shown the default dashboard list.

![](https://miro.medium.com/max/842/1*XTi2xw1eVvlNE5Iy_WfVAw.png)

I will select one of the dashboards. As you can see below image, the data source is set to Prometheus (default data source), and the namespace for visualization is set to monitoring.

![](https://miro.medium.com/max/842/1*rhSYus8lM5f-aOoeabKLtA.png)

**3.2** **Import pre-defined dashboard into Grafana**

In the previous section, you are interacting with the Grafana UI; I think you can now try to import some pre-defined dashboards from there [https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/).

**_Note_**: **You can select any dashboard you want from that list based on your requirement and later you can customize the dashboard as you want.**

-   Navigate to the URL [https://grafana.com/grafana/dashboards/6417-kubernetes-cluster-prometheus/](https://grafana.com/grafana/dashboards/6417-kubernetes-cluster-prometheus/) (this is my selection since it provides more visualization on the Kubernetes cluster) and copy the Dashboard ID (6417)

![](https://miro.medium.com/max/842/1*wAPt5Ne4kdlahTdGPewtRg.png)

-   Now go to the dashboard icon, and click the browse option.
-   Next from the top of the right corner, click the new button and select the import option.

![](https://miro.medium.com/max/842/1*HaMae2Kpab7yf9BCmCmWuA.png)

You will get window show as below, and put the ID into the field and click “Load” button.

![](https://miro.medium.com/max/842/1*XWuT2LCEzvkT0lY3s0K2FA.png)

-   Next, you will get the window shown below and fill the fields same as in the image.

![](https://miro.medium.com/max/842/1*WDCdQ4X0H-KweLiwW265aQ.png)

-   Finally, you can click the import button and it will be redirecting to your imported dashboard.

![](https://miro.medium.com/max/842/1*AhOQ3TxzWr6zRFqluVWCkA.png)

Cool… right? 😎

In this article, you learned how to deploy Prometheus and Grafana using Helm, as well as how to configure Grafana and view your cluster metrics by configuring your Grafana dashboard.

Thanks for reading! 🙏 I hope this article was helpful.

Happy reading guys! Bye Bye! 🙋♂️[️](https://emojipedia.org/man-raising-hand/)