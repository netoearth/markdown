Learn how to setup MySQL on Kubernetes.

![](https://miro.medium.com/max/700/1*zwtjZsL6FVo1S4hZtVPeFw.jpeg)

Photo by [Souvik Banerjee](https://unsplash.com/@rswebsols?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/wordpress?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

WordPress is a popular open-source CMS(content management system) software. Many organizations are using WordPress. Why people love WordPress. Because it is open-source software and developed using PHP which is easy to learn. And customization is also very easy if you know the basics of PHP. Many developers are developing both free and paid themes and plugins for WordPress. Installing the themes and plugins is very easy. Just one click to install and uninstall. So you will get plenty of options for your CMS when you use WordPress.

In this tutorial, you will learn, how to deploy the WordPress application to the Kubernetes cluster. Deploying the WordPress application is very if you know the basics of Kubernetes components such as Pods, Volume, Services, and Secrets. The WordPress deployment on Kubernetes is split into two-part. This is the first part. This article covers how to set up MySQL on Kubernetes and how to create a service for MySQL. The next part will focus on the WordPress deployment and how to connect with MySQL.

In this article, you will learn the following topics.

1\. Create Secret for MySQL

2\. Create PVC for MySQL

3\. Create Deployment for MySQL

4\. Create Service for MySQL

Enough theory. Let’s start the deployment of MySQL.

## **1\. Create Secret for MySQL**

MySQL requires a root password while installing it. You can set the root password in the deployment file. However, using a plain password is not recommended. You need to encode the password before using it. You can store your encoded password in the Kubernetes secrets component.

Use the following command to encode a password.

```
echo -n 'mysql1234' | base64
```

The code to create a Kubernetes secret is given below.

```
apiVersion: v1kind: Secretmetadata:  name: wp-db-secretstype: Opaquedata:  MYSQL_ROOT_PASSWORD: bXlzcWwxMjM0
```

In the above code, place your encoded password. Save the code in a file name **mysql-secret.yaml**. And execute the file using the apply command.

```
kubectl apply -f mysql-secret.yaml
```

Now get the list of secrets using the following command.

```
kubectl get secrets
```

Now the secret is ready to use. Next, you need to create a volume for data storage.

## **2\. Create PVC for MySQL**

By default, the Pod in the Kubernetes does not store the data. To store the data in the Kubernetes you need storage. I am using Linode LKE. Here they offer a built-in storage class. So instead of creating Storage Class and PV(Persistent Volume), I am able to create the PVC(Persistent Volume Claim).

Use the below code to create a PVC.

```
---apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: mysql-volumespec:  accessModes:  - ReadWriteOnce  resources:    requests:      storage: 1Gi  storageClassName: your-storage-name
```

In the above code please change the **storage** and **storageClassName** as per your need. Save the code in a file name **mysql-volume.yaml**. And execute the code using the following command.

```
kubectl apply -f mysql-volume.yaml
```

Now the PV and PVC are created.

Get the list of PV using the following command.

```
kubectl get pv
```

Get the list of PVC using the following command.

```
kubectl get pvc
```

## **3\. Create Deployment for MySQL**

The password and storage are ready. Now you can deploy MySQL. Use the following code to create a MySQL Pod.

```
---apiVersion: apps/v1kind: ReplicaSetmetadata:  name: mysql  labels:    app: mysqlspec:  replicas: 1  selector:    matchLabels:      app: mysql  template:    metadata:      labels:        app: mysql    spec:      containers:      - name: database        image: mysql:5.7        args:          # mount volume          - "--ignore-db-dir=lost+found"        # add root password        envFrom:          - secretRef:              name: wp-db-secrets        ports:          - containerPort: 3306        volumeMounts:        - name: mysql-data          mountPath: /var/lib/mysql      volumes:      - name: mysql-data        persistentVolumeClaim:          claimName: mysql-volume
```

1\. Here the deployment type is chosen as **ReplicaSet** using the kind option. Instead of ReplicaSet, you can use Deployment or StatefulSet.

2\. MySQL 5.7 is the version used.

3\. The password is set using the **envFrom**.

4\. The volume is mounted using the **persistentVolumeClaim** option from volumes.

Save the above code in a file name **mysql.yaml** and execute using the below command.

```
kubectl apply -f mysql.yaml
```

The MySQL Pod is created and it is up and running.

Get the list of running Pods.

```
kubectl get pods
```

If you want to know what is happened during the Pod creation, then please use the describe command to know the details.

```
kubectl describe pod pod_name
```

## **4\. Create Service for MySQL**

The MySQL Pod is up and running. If an application wants to use the MySQL database then you need to create a service for the Pod. Why because you can not directly access MySQL. The Service component will give access to the MySQL Pod.

Create a Service for MySQL using the below code.

```
---apiVersion: v1kind: Servicemetadata:  name: mysql-servicespec:  ports:  - port: 3306    protocol: TCP  selector:    app: mysql
```

The service type is **ClusterIP**. Why because the default value is ClusterIP. If ClusterIP means, only internal Pods can access the MySQL Pod. You can access the MySQL Pod from an outside network.

Save the code in a file name **mysql-service.yaml**. And execute using the below command.

```
kubectl apply -f mysql-service.yaml
```

Get the list of running services.

```
kubectl get svc
```

Now you need to create a database in MySQL. So, enter into the MySQL Pod using the Pod name.

```
kubectl exec -it pod_name -- bash#Examplekubectl exec -it mysql-lh4v4 -- bash
```

Use the below command to connect with MySQL and enter the password.

```
mysql -u root -p
```

Create a database named WordPress.

```
CREATE DATABASE wordpress;
```

Get the list of available databases.

```
show databases;
```

Now exit from the database and Pod using the **exit** command.

Now MySQL is ready. You can connect WordPress with MySQL and proceed with the WordPress installation. This can be done in the next article.

## **Summary:**

WordPress is very popular why because it is very easy to use. And you can easily get support from the developer or developer community. There are many free plugins and themes are available for WordPress. If you want to scale the WordPress then using Kubernetes is recommended why because scaling can be done very easily.

I hope this article will help you start the installation process of WordPress in Kubernetes.

Stay tuned for the next part.

Thank you for reading.