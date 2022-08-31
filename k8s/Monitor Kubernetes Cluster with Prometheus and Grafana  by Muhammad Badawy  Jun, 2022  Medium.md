![](https://miro.medium.com/max/700/1*kzVoQlMX5O55ni8195E7wA.png)

Monitoring k8s through prometheus and grafana

**In this article** I will demonstrate how to monitor kubernetes cluster resources with prometheus and grafana.

Let’s first give a bit intro about [Prometheus](https://prometheus.io/docs/introduction/overview/) and [Grafana](https://github.com/grafana/grafana), So prometheus is an open source tool for monitoring targets like kubernetes resources, databases, virtual machines etc … It can be also configured with ALERTMANAGER to handle alerts or integrated with Grafana to visualise metrics and build dashboards based on it.

Prometheus uses promQL to query metrics in the below format:

```
METRIC{condition,anotherCondition}
```

Also Prometheus by design implements a pull-based approach for collecting metrics which makes it easy to enroll new targets dynamically through service discovery.

Grafana as mentioned is the GUI part of the monitoring system through which you can add “datasources” to bring other systems metrics, also you can import dashboards or create it from scratch.

Now let’s jump to the **demo**.

Kubernetes version in this demo will be 1.21.9

I’m assuming that you have your kubernetes cluster up and running and you have [helm](https://helm.sh/docs/intro/install/) and [kubectl](https://kubernetes.io/docs/tasks/tools/) installed also then you can proceed with the below steps.

**Steps:**

1- **Let’s first create new namespace**

```
#kubectl create ns monitor
```

2- **Then** we will use helm charts to apply prometheus and grafana resources in kubernetes, The below command will add prometheus repository to your local.

```
#helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

3- **Then** let’s install prometheus resources based on that repository through helm.

```
#helm install prometheus prometheus-community/prometheus --namespace monitor --set alertmanager.persistentVolume.storageClass="managed-csi-premium" --set server.persistentVolume.storageClass="managed-csi-premium"
```

**A couple of points to be cleared here:**

-   Through passing arguments to helm command, we have the ability to change the default values stored in the values file of the helm chart. So here I needed to pass persistent volume class for alertmanager and prometheus server and because I’m using Azure Kubernetes Service, I passed the value of “managed-csi-premium” so this value depends on which kubernetes service that you use and what are the storage classes available. For example if you are using EKS, so this value will be ‘gp2’
-   You can check the values file [here](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus) to see how values can be overwritten.

4- **Then** let’s add grafana charts to our local repository to be able to install it.

```
#helm repo add grafana https://grafana.github.io/helm-charts
```

5- **Then** let’s install grafana resources based on that repository through helm.

```
#helm install grafana grafana/grafana --namespace monitor --set persistence.enabled=true --set persistence.storageClassName="managed-csi-premium" --set adminPassword='CHANGE_IT' --set service.type=ClusterIP
```

Here also I choose to change the default values of the helm chart to enable data persistency for grafana, also to set storage class name, set grafana admin password and specify the service type.

-   You can check the values file [here](https://github.com/grafana/helm-charts/tree/main/charts/grafana) to see how values can be overwritten.

6- **At this point you have a couple of options to navigate to grafana UI**:

1- Through ‘kubectl port-forward’ command or  
2- Installing ingress controller or  
3- Change the type of grafana service to be LoadBalancer

For the demo, I will choose the first option to port-forward from our local-machine, So all you need to do is:

-   List the pods in monitor namespace to pick grafana pod name:

```
kubectl get pods -n monitor
```

-   _port-forward_ from your local machine to grafana pod port which is 3000

```
kubectl port-forward <GRAFANA_POD_NAME> 3000:3000 -n monitor
```

-   In your terminal, you should be able to see logs like the below:

```
Forwarding from 127.0.0.1:3000 -> 3000Forwarding from [::1]:3000 -> 3000
```

Now you can go to your browser and navigate to localhost:3000 to see grafana UI and login with admin username and the password configured earlier.

![](https://miro.medium.com/max/544/1*HU9yk-GGG5MmzGGvC9JyXw.png)

Grafana login page

7- **Add prometheus as datasource to grafana:**

So now grafana needs prometheus to be defined as _datasource_ so grafana will be able to see all the metrics and visualize them through dashboards.

So after you login, from Configuration tab->choose “Data Sources”-> Then “Add Data Source”, and choose prometheus to see the below screen.

![](https://miro.medium.com/max/700/1*yv8HF3ZArq1t5-0D9sIJJw.png)

Grafana DataSource

In URL variable, we need to add prometheus service URL which should follow the below pattern  
<service-name>.<namespace>.svc.cluster.local:<service-port>

In our case, the value will be:  
[http://prometheus-server.monitor.svc.cluster.local](http://prometheus-server.monitor.svc.cluster.local/)

Finally press “Save & test” which should be successfully done.

8- **Now you should be able to import dashboards to grafana or create it from scratch.**  
In this demo I will give an example how to import a dashboard and in another article I can demonstrate how to create them from scratch.

When you press the import button as below, you will be able to import dashboards by ID or by JSON:

![](https://miro.medium.com/max/378/1*fEYBqa6pJjqCvCulpNcBTw.png)

![](https://miro.medium.com/max/1336/1*9hOVk8mMgw3O8HafQBc8ug.png)

Import Dashboard to Grafana

You can use the JSON file in [here](https://github.com/dotdc/grafana-dashboards-kubernetes/blob/master/dashboards/k8s-views-global.json) to have global views for your kubernetes cluster. Just copy the JSON and drop it into the JSON panel, Then “Load” and you will be redirected to modify the dashboard name and choose the datasource which be **prometheus** in our case.

9- **Finally you will be able to see that beautiful view and more:**

![](https://miro.medium.com/max/700/1*MriiPdxUWMJrx-7ja7RMLw.png)

Grafana Dashboard for kubernetes

At this point we reached the end of our tutorial. I hope you enjoyed reading this article, feel free to add your comments, thoughts or feedback and don’t forget to get in touch on [**linkedin**](https://www.linkedin.com/in/muhammad-badawy-linked1n/).