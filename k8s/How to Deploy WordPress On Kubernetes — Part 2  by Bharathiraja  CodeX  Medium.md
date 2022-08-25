## Learn how to deploy the WordPress on Kubernetes and connect with MySQL Pod.

![](https://miro.medium.com/max/700/1*VpxbX3hVaUuz9jDNjBLUiQ.png)

WordPress On Kubernetes

WordPress needs MySQL to store data(text content). In the [previous article](https://medium.com/codex/how-to-deploy-wordpress-on-kubernetes-part-1-62cc5bd74410), you have learned how to deploy MySQL on Kubernetes. If you have not done the MySQL setup then please visit this [link](https://medium.com/codex/how-to-deploy-wordpress-on-kubernetes-part-1-62cc5bd74410) and create MySQL Pod. In this tutorial, you will learn how to deploy the WordPress on the Kubernetes cluster and how to connect with the MySQL Pod. MySQL credentials are passed to the WordPress while creating the Pod configuration file for WordPress. So do not forget the MySQL username, password, and database name.

In this article, you will learn the following topics.

1\. Create PVC for WordPress

2\. Create Deployment for WordPress

3\. Create Service for WordPress

4\. Remove All Pods, PVCs, and Services

Enough theory. Let’s start the deployment of WordPress.

## **1\. Create PVC for WordPress**

WordPress stores Post data(text content) in MySQL. And stores the images on the hard disk. So WordPress also needs separate storage for storing images. So create PV and PVC for WordPress using the following code. Here PV is automatically created using the PVC. Here I am using Linode LKE cloud provider.

```
apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: wordpress-volumespec:  accessModes:  - ReadWriteOnce  resources:    requests:      storage: 10Gi  storageClassName: block-storage-class-name
```

Save the code in a file name **wp-volume.yaml** and execute using the following command.

```
kubectl apply -f wp-volume.yaml
```

Get the list of PV and PVC.

```
kubectl get pvkubectl get pvc
```

Here the command displays both MySQL and WordPress PV and PVC.

## **2\. Create Deployment for WordPress**

This is the meat of this article. Use the following code to create a WordPress Pod.

```
---apiVersion: apps/v1kind: Deploymentmetadata:  name: wordpressspec:  replicas: 1  selector:    matchLabels:      app: wordpress  template:    metadata:      labels:        app: wordpress    spec:      containers:        - name: wordpress          image: wordpress:5.8.3-php7.4-apache          ports:          - containerPort: 80            name: wordpress          volumeMounts:            - name: wordpress-data              mountPath: /var/www          env:            - name: WORDPRESS_DB_HOST              value: mysql-service.default.svc.cluster.local            - name: WORDPRESS_DB_PASSWORD              valueFrom:                secretKeyRef:                  name: wp-db-secrets                  key: MYSQL_ROOT_PASSWORD            - name: WORDPRESS_DB_USER              value: root            - name: WORDPRESS_DB_NAME              value: wordpress      volumes:        - name: wordpress-data          persistentVolumeClaim:            claimName: wordpress-volume
```

Here a few important parameters are explained.

1\. **image** => It tells what Docker image to install. Use the latest version of WordPress from Docker Hub.

2\. **volumeMounts** => Use to mount the path of WordPress folder and send the data(images, pdfs) to PV.

3\. **WORDPRESS\_DB\_HOST** => Used to connect the MySQL Pod. Please pass the correct value. The value for this parameter will be created using the following syntax.

```
#syntaxservice-name.namespace.svc.cluster.local#Actual Valuemysql-service.default.svc.cluster.local
```

Save the code in a file name **wp.yaml** and execute using the following command.

```
kubectl apply -f wp.yaml
```

The WordPress Pod is created. And it works fine.

Get the list of running Pods using the following command.

```
kubectl get pods
```

## **3\. Create Service for WordPress**

The WordPress Pod is up and running. You can not directly access the WordPress Pod. If you want to access the WordPress application, then you need to create a Load Balancer service for the WordPress Pod.

Create a Load Balancer service for WordPress Pod using the following code.

```
---kind: ServiceapiVersion: v1metadata:  name: wordpress-servicespec:  type: LoadBalancer  selector:    app: wordpress  ports:    - name: http      protocol: TCP      port: 80      targetPort: 80
```

Save the code in a file name **wp-service.yaml** and execute using the following command.

```
kubectl apply -f wp-service.yaml
```

Just look at the running services, then it will show an IP address for the load balancer service. Access the application using the IP address.

```
kubectl get svc
```

Enter the IP address given by the Load Balancer service in the browser and you will get a page like the below image. Click Continue to proceed further.

![](https://miro.medium.com/max/700/1*m6j6jbQCbADBBCdD7gAaCA.png)

WordPress Installation

Then set up the WordPress username and password to install the application.

![](https://miro.medium.com/max/700/1*FbCfi351IrTRGxKRdnduCA.png)

WordPress Create User

That’s all. Your WordPress application is ready now. You can publish your post/articles using WordPress.

## **4\. Remove All Pods, PVCs, and Services**

If you don’t want to keep the resource after practicing, then you can delete the Kubernetes components using the below commands.

```
kubectl delete svc --allkubectl delete pod --all kubectl delete pvc --all kubectl delete pv --all
```

## **Summary:**

WordPress is a popular CMS(Content Management System) software and open-source too. It is developed using PHP and MySQL. Many organisations are using WordPress and customising WordPress is very easy. The WordPress deployment on Kubernetes is split into two parts. This is the second part of WordPress deployment. Use [this](https://medium.com/codex/how-to-deploy-wordpress-on-kubernetes-part-1-62cc5bd74410) link for the first part of the WP deployment.

I hope this article will help you to install and run the WordPress on Kubernetes.

Stay tuned for more articles.

Thank you for reading.