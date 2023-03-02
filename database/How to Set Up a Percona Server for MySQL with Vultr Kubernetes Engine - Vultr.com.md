## Introduction

Percona Server for MySQL is a free, fully compatible, enhanced, and open-source drop-in replacement for any MySQL database. By using the [Percona Distribution for MySQL Operator](https://www.percona.com/doc/kubernetes-operator-for-pxc/index.html) in Kubernetes environments, you can ensure data availability for your MySQL database while simplifying the complex deployments using Kubernetes configuration files. This guide explains how to deploy, configure, and backup a high-availability MySQL cluster with Vultr Kubernetes Engine. Backups are stored separately on Vultr Object Storage with the ability to restore with point-in-time recovery.

## Prerequisites

Before you begin, you should:

-   Deploy a [Vultr Kubernetes Cluster](https://my.vultr.com/kubernetes/) with at least three nodes.
    
-   Configure `kubectl` and `git` in your machine.
    
-   Create a [Vultr Object Storage](https://my.vultr.com/objectstorage/).
    
-   Prepare the access key and secret key for your Object Storage
    

## 1\. Install Percona Distribution for MySQL Operator

Install Percona Distribution for MySQL Operator version 1.10.0 using Kubectl:

```
$ kubectl apply -f https://raw.githubusercontent.com/percona/percona-xtradb-cluster-operator/v1.10.0/deploy/bundle.yaml
```

## 2\. Create MySQL Cluster

-   Step 1: Download the source code for this guide.
    
    ```
    $ git clone -b v1.10.0 https://github.com/quanhua92/k8s-mysql-cluster
    ```
    
-   Step 2: Deploy your database cluster using the following command:
    
    ```
    $ kubectl apply -f 001_cr_minimal.yaml
    ```
    

The CR spec file for your database cluster is defined in the file `001_cr_minimal.yaml`.

```
apiVersion: pxc.percona.com/v1-10-0

kind: PerconaXtraDBCluster

metadata:

  name: db-cluster

spec:

  crVersion: 1.10.0

  secretsName: db-cluster-secrets

  allowUnsafeConfigurations: true

  upgradeOptions:

    apply: 8.0-recommended

    schedule: "0 4 * * *"

  pxc:

    size: 3

    image: percona/percona-xtradb-cluster:8.0.23-14.1

    volumeSpec:

      persistentVolumeClaim:

        storageClassName: vultr-block-storage

        accessModes: ["ReadWriteOnce"]

        resources:

          requests:

            storage: 15G

  haproxy:

    enabled: true

    size: 3

    image: percona/percona-xtradb-cluster-operator:1.10.0-haproxy

    #serviceType: LoadBalancer

    #replicasServiceType: LoadBalancer

  logcollector:

    enabled: true

    image: percona/percona-xtradb-cluster-operator:1.10.0-logcollector
```

The `001_cr_minimal.yaml` is designed specifically for Vultr Kubernetes Engine.

By default, this spec file creates a cluster named `db-cluster` with 03 pods for MySQL Instances and 03 HAProxy pods for high availability. A Vultr Block Storage are used as the data storage for each MySQL instance.

The results of running `kubectl get pods -o wide` are similar too:

```
NAME                                               READY   STATUS    RESTARTS   AGE     IP              NODE                NOMINATED NODE   READINESS GATES

db-cluster-haproxy-0                               2/2     Running   0          5m7s    10.244.80.197   node-1f0f02fd5303   <none>           <none>

db-cluster-haproxy-1                               2/2     Running   0          3m16s   10.244.19.131   node-1e43b600d18b   <none>           <none>

db-cluster-haproxy-2                               2/2     Running   0          2m56s   10.244.182.4    node-a2ed9ec0ccf8   <none>           <none>

db-cluster-pxc-0                                   3/3     Running   0          5m7s    10.244.182.3    node-a2ed9ec0ccf8   <none>           <none>

db-cluster-pxc-1                                   3/3     Running   0          3m23s   10.244.80.198   node-1f0f02fd5303   <none>           <none>

db-cluster-pxc-2                                   3/3     Running   0          100s    10.244.19.132   node-1e43b600d18b   <none>           <none>

percona-xtradb-cluster-operator-566848cf48-tl4fh   1/1     Running   0          5m25s   10.244.80.196   node-1f0f02fd5303   <none>           <none>
```

## 3\. Access MySQL Service

-   Step 1: Get the root password to access the MySQL service under the secret named `db-cluster-secrets`.
    
    ```
    $ kubectl get secret db-cluster-secrets -o yaml
    ```
    

The output should look as follows:

```
apiVersion: v1

data:

  clustercheck: MjFuTUZmblMyVTRJeW11Ng==

  monitor: bnJLUFNrVmgxcE9NTFd6ZVI=

  operator: N3p3M0hqbVYyUlVuZ1dSZzEycA==

  proxyadmin: dGducHllSnZPdXM2cDhYY1Y=

  replication: ZXhrS0tidEJ3N29QZE5kRHA=

  root: MUVtTnBDVkdJeWI4cVlraGVY

  xtrabackup: RFBJQ0FWRkIyazJCTmFONEM=

kind: Secret

metadata:

  creationTimestamp: "2022-01-09T13:23:42Z"

  name: db-cluster-secrets

  namespace: default

  resourceVersion: "1011"

  uid: 5e05580e-cab7-4b3d-bf18-f978614d73dd

type: Opaque
```

Get the base64 encoded password as root. It is `MUVtTnBDVkdJeWI4cVlraGVY` in this example case here.

-   Step 2: Decode the value for the root password using the following command:
    
    ```
    $ echo 'MUVtTnBDVkdJeWI4cVlraGVY' | base64 --decode
    ```
    
-   Step 3: Create a pod in the Kubernetes cluster to access the MySQL service.
    
    ```
    $ kubectl run -it --rm percona-client --image=percona:8.0 -* bash
    ```
    
-   Step 4: Login using the mysql tool as follows (replace <root\_password> with your decoded password in the previous step):
    
    ```
    $ mysql -h db-cluster-haproxy -uroot -p<root_password>
    ```
    

## 4\. (Optional) Expose MySQL Service for External Usage With Load Balancer

Add the serviceType under haproxy in CR YAML spec to expose the MySQL Service for applications outside of Kubernetes

```
haproxy:

    serviceType: LoadBalancer

    replicasServiceType: LoadBalancer
```

Install the provided `002_cr_load_balancer.yaml` spec to enable load balancers. This command creates [Vultr Load Balancers in your account](https://my.vultr.com/loadbalancers/).

```
$ kubectl apply -f 002_cr_load_balancer.yaml
```

## 5\. Day-2 Operations

This section shows how to manage backups and restores to have a good disaster recovery plan with a 3-2-1 backup strategy.

A 3-2-1 backup strategy means having at least three total copies of your data (one production data and two backup copies) on two different media and one offsite backup.

For the sake of simplicity, this guide takes advantage of the scheduled backup feature to automatically archive three copies of data:

1.  Production data: Vultr Block Storage.
    
2.  Vultr Object Storage: the first backup location.
    
3.  Offsite S3-compatible Object Storage: Vultr Object Storage or any S3-compatible storage.
    

### 5.1 Setup Vultr Object Storage

Get the access id and access key of the [Vultr Object Storage](https://my.vultr.com/objectstorage/).

-   Step 1: Store the access key and secret key as a secret named `vultr-s3-secret`.

Here is the `003_vultr_s3_secrets.yaml` file as an example.

```
apiVersion: v1

kind: Secret

metadata:

  name: vultr-s3-secret

type: Opaque

stringData:

  AWS_ACCESS_KEY_ID: "4hrVSwKAYtfcjFu92r"

  AWS_SECRET_ACCESS_KEY: "sIgPx!z#%9V&l65TRV"
```

-   Step 2: Save the secret to Kubernetes
    
    ```
    $ kubectl apply -f 003_vultr_s3_secrets.yaml
    ```
    
-   Step 3: Create a bucket with the name `db-cluster-backup` in your Vultr Object Storage. This bucket is used to store the backup data in a later section.
    

### 5.2 Setup S3-compatible Object Storage

Create another [Vultr Object Storage](https://my.vultr.com/objectstorage/) or use any S3-compatible Object Storage.

-   Step 1: Store the access key and secret key as a secret named `offsite-s3-secret`

Here is the example spec, `004_offsite_s3_secrets.yaml`

```
apiVersion: v1

kind: Secret

metadata:

  name: offsite-s3-secret

type: Opaque

stringData:

  AWS_ACCESS_KEY_ID: "4hrVSwKAYtfcjFu92r"

  AWS_SECRET_ACCESS_KEY: "sIgPx!z#%9V&l65TRV"
```

-   Step 2: Save the secret to Kubernetes
    
    ```
    $ kubectl apply -f 004_offsite_s3_secrets.yaml
    ```
    
-   Step 3: Create a bucket with the name `db-cluster-offsite-backup` in your Object Storage. This bucket are used to store the backup data in a later section.
    

### 5.3 Setup Automatic Backup

The file `005_cr_with_backup_s3.yaml` contains a new `backup` section with the secrets in the previous step.

```
backup:

  image: perconalab/percona-xtradb-cluster-operator:main-pxc8.0-backup

  pitr:

    enabled: true

    storageName: s3-vultr

    timeBetweenUploads: 30

  storages:

    s3-vultr:

      type: s3

      s3:

        bucket: db-cluster-backup

        credentialsSecret: vultr-s3-secret

        endpointUrl: https://<insert_vultr_s3_endpoint_here>

    s3-offsite:

      type: s3

      s3:

        bucket: db-cluster-backup

        credentialsSecret: offsite-s3-secret

        endpointUrl: https://<insert_offsite_s3_endpoint_here>

  schedule:

    - name: "daily-backup-vultr"

      schedule: "0 0 * * *"

      keep: 10

      storageName: s3-vultr

    - name: "daily-backup-offsite"

      schedule: "0 0 * * *"

      keep: 10

      storageName: s3-offsite
```

In the above spec, a scheduled backup runs every day and there are at most 10 copies of those in the storage. The point-in-time recovery feature is enabled to give you the ability to restore to any specific date and time.

-   Step 1: Make sure that the `bucket`, `endpointUrl` are correct and apply the changes with the following command:
    
    ```
    $ kubectl apply -f 005_cr_with_backup_s3.yaml
    ```
    

The results of running `kubectl get pods -o wide` are similar too:

```
NAME                                               READY   STATUS    RESTARTS   AGE     IP              NODE                NOMINATED NODE   READINESS GATES

db-cluster-haproxy-0                               2/2     Running   0          7m52s   10.244.80.197   node-1f0f02fd5303   <none>           <none>

db-cluster-haproxy-1                               2/2     Running   0          6m1s    10.244.19.131   node-1e43b600d18b   <none>           <none>

db-cluster-haproxy-2                               2/2     Running   0          5m41s   10.244.182.4    node-a2ed9ec0ccf8   <none>           <none>

db-cluster-pitr-6cfb7549f7-bgsr5                   1/1     Running   0          16s     10.244.182.6    node-a2ed9ec0ccf8   <none>           <none>

db-cluster-pxc-0                                   3/3     Running   0          7m52s   10.244.182.3    node-a2ed9ec0ccf8   <none>           <none>

db-cluster-pxc-1                                   3/3     Running   0          6m8s    10.244.80.198   node-1f0f02fd5303   <none>           <none>

db-cluster-pxc-2                                   3/3     Running   0          4m25s   10.244.19.132   node-1e43b600d18b   <none>           <none>

percona-client                                     1/1     Running   0          98s     10.244.182.5    node-a2ed9ec0ccf8   <none>           <none>

percona-xtradb-cluster-operator-566848cf48-tl4fh   1/1     Running   0          8m10s   10.244.80.196   node-1f0f02fd5303   <none>           <none>
```

-   Step 2: Check the logs of the backup pod PITR with the following command. You need to change the pod name to your corresponding pod name.
    
    ```
    $ kubectl logs db-cluster-pitr-6cfb7549f7-bgsr5
    ```
    

The result shows that binary logs have been successfully written to S3. These are the backup for the Point-In-Time Recovery feature.

```
2022/01/09 13:30:08 run binlog collector

2022/01/09 13:30:10 Reading binlogs from pxc with hostname= db-cluster-pxc-0.db-cluster-pxc.default.svc.cluster.local

2022/01/09 13:30:10 Starting to process binlog with name binlog.000003

2022/01/09 13:30:10 Successfully written binlog file binlog.000003 to s3 with name binlog_1641594293_733acbcaf56c45ed9cb41518b6b90cc0

2022/01/09 13:30:10 Starting to process binlog with name binlog.000004

2022/01/09 13:30:11 Successfully written binlog file binlog.000004 to s3 with name binlog_1641595077_bf4ecf614e5dc157128e7495cb590d4a

2022/01/09 13:30:11 Starting to process binlog with name binlog.000012

2022/01/09 13:30:13 Successfully written binlog file binlog.000012 to s3 with name binlog_1641595331_c4261dced1631cc839d4ef97b4b60931
```

-   Step 3: Go to the [Vultr Object Storage](https://my.vultr.com/objectstorage/) to browse for the files.

### 5.4 Create a Manual Backup

-   Step 1: Create a new spec file `006_manual_backup.yaml`, to make a manual backup
    
    ```
    apiVersion: pxc.percona.com/v1
    
    kind: PerconaXtraDBClusterBackup
    
    metadata:
    
    finalizers:
    
            - delete-s3-backup
    
    name: backup-001
    
    spec:
    
    pxcCluster: db-cluster
    
    storageName: s3-vultr
    ```
    
-   Step 2: Run the command to perform the backup:
    
    ```
    $ kubectl apply -f 006_manual_backup.yaml
    ```
    
-   Step 3: Run the following command to see the backup progress:
    
    ```
    $ kubectl get pxc-backup -w
    ```
    

The result should look similar to:

```
NAME         CLUSTER      STORAGE    DESTINATION                                                  STATUS    COMPLETED   AGE

backup-001   db-cluster   s3-vultr   s3://db-cluster-backup/db-cluster-2022-01-09-13:32:16-full   Running               9s

backup-001   db-cluster   s3-vultr   s3://db-cluster-backup/db-cluster-2022-01-09-13:32:16-full   Succeeded   4s        35s
```

-   Step 4: After the status has changed to `Succeeded`, you can go to the [Vultr Object Storage](https://my.vultr.com/objectstorage/) to browse for the files.

### 5.5 Add Data to the Database

-   Step 1: Create a pod to access the database. (replace <root\_password> with your decoded password in the previous step)
    
    ```
    $ kubectl run -it --rm percona-client --image=percona:8.0 -- bash
    
    $ mysql -h db-cluster-haproxy -uroot -p<root_password>
    ```
    
-   Step 2: Create a new database and table by running the following MYSQL commands
    
    ```
    mysql> create database data;
    
    mysql> use data;
    
    mysql> create table first_table(id INT PRIMARY KEY);
    ```
    
-   Step 3: Add a few rows to the database table
    
    ```
    mysql> insert into data.first_table values (1), (2), (3);
    
    mysql> select * from data.first_table;
    ```
    
-   Step 4: Make sure that the binary logs are uploaded to [Vultr Object Storage](https://my.vultr.com/objectstorage/) before deleting the database
    
-   Step 5: Drop the database for test purpose
    
    ```
    mysql> drop database data;
    ```
    

### 5.6 Restore

-   Step 1: Disable PITR before doing any restore, even if the backup was not made with point-in-time recovery. Set the property `spec.backup.pitr.enabled` to `false` in CR YAML file.
    
-   Step 2: Apply the `007_cr_disable_pitr.yaml` file using the following command:
    
    ```
    $ kubectl apply -f 007_cr_disable_pitr.yaml
    ```
    
-   Step 3: Prepare the restore spec file to restore to the time right after you drop the database. Here is my restore spec file `008_restore.yaml`. You need to change the date-time to match your setup.
    
    ```
    apiVersion: pxc.percona.com/v1
    
    kind: PerconaXtraDBClusterRestore
    
    metadata:
    
    name: restore-001
    
    spec:
    
    pxcCluster: db-cluster
    
    backupName: backup-001
    
    pitr:
    
            type: date
    
            date: "2022-01-09 13:36:00"
    
            backupSource:
    
            storageName: s3-vultr
    ```
    
-   Step 4: Perform the recovery using the command:
    
    ```
    $ kubectl apply -f .\008_restore.yaml
    ```
    
-   Step 5: Run the following command to see the progress:
    
    ```
    $ kubectl get pxc-restore -w
    ```
    

The result should look similar to:

```
NAME          CLUSTER      STATUS             COMPLETED   AGE

restore-001   db-cluster   Stopping Cluster               4s

restore-001   db-cluster   Restoring                      72s

restore-001   db-cluster   Starting Cluster               101s

restore-001   db-cluster   Point-in-time recovering       2m13s

restore-001   db-cluster   Succeeded           0s         5m46s
```

-   Step 6: After the status changed to `Succeeded`, perform the query to check the database
    
    ```
    $ kubectl run -it --rm percona-client --image=percona:8.0 -- bash
    
    $ mysql -h db-cluster-haproxy -uroot -p<root_password>
    
    mysql> show databases;
    
    mysql> select * from data.first_table;
    ```
    
-   Step 7: Enable the PITR feature by setting the `spec.backup.pitr.enabled` to `true`. Run the following command:
    
    ```
    $ kubectl apply -f .\010_cr_enable_pitr.yaml
    ```
    

### Quick Tips

If you donâ€™t see any changes under the status when running either `kubectl get pxc-restore` or `kubectl get pxc-backup`, try to reset the Percona Operator pod as follows:

1.  Get the pod name of Percona Operator
    
    ```
    $ kubectl get pod -l app.kubernetes.io/instance=percona-xtradb-cluster-operator
    ```
    
2.  Delete the pod. (Replace <pod\_name> with the pod name):
    
    ```
    $ kubectl delete pod <pod_name>
    ```
    

## 6\. Benchmark Your MySQL Service with Sysbench

1.  Create a pod to run the `sysbench` benchmark. Replace <root\_password> with your decoded password in the previous step.
    
    ```
    $ kubectl run -it --rm sysbench-client --image=perconalab/sysbench:latest -- bash
    ```
    
2.  Create a database named `sbtest`
    
    ```
    $ mysql -h db-cluster-haproxy  -uroot -p<root_password>
    
    mysql> create database sbtest;
    
    mysql> exit;
    ```
    
3.  Prepare data tables
    
    ```
    $ sysbench oltp_read_write --mysql-user=root --mysql-password=<root_password> --db-driver=mysql --tables=10 --table-size=100000 --mysql-host=db-cluster-haproxy --mysql-port=3306 prepare
    ```
    
4.  Perform Read-Write test
    
    ```
    $ sysbench oltp_read_write --mysql-user=root --mysql-password=<root_password> --db-driver=mysql --tables=10 --table-size=100000 --mysql-host=db-cluster-haproxy --mysql-port=3306 --time=30 --threads=100 run
    ```
    
5.  Perform Read-only test
    
    ```
    $ sysbench oltp_read_only --mysql-user=root --mysql-password=<root_password> --db-driver=mysql --tables=10 --table-size=100000 --mysql-host=db-cluster-haproxy --mysql-port=3306 --time=30 --threads=100 run
    ```
    
6.  Perform Write-only test
    
    ```
    $ sysbench oltp_write_only --mysql-user=root --mysql-password=<root_password> --db-driver=mysql --tables=10 --table-size=100000 --mysql-host=db-cluster-haproxy --mysql-port=3306 --time=30 --threads=100 run
    ```
    
7.  Perform Select-Random-Points test
    
    ```
    $ sysbench select_random_points --mysql-user=root --mysql-password=<root_password> --db-driver=mysql --tables=10 --table-size=1000 --mysql-host=db-cluster-haproxy --mysql-port=3306 --threads=100 --rand-type=uniform --time=30 run
    ```
    
8.  Clean up the test data
    
    ```
    $ sysbench oltp_read_write --mysql-user=root --mysql-password=<root_password> --db-driver=mysql --tables=10 --table-size=100000 --mysql-host=db-cluster-haproxy --mysql-port=3306 cleanup
    ```
    

## Clean Up

Uninstall everything in this guide as follows:

1.  Delete the cluster:
    
    ```
    $ kubectl delete -f .\010_cr_enable_pitr.yaml
    
    $ kubectl delete deployment.apps/db-cluster-pitr
    ```
    
2.  Delete the PXC backup
    
    ```
    $ kubectl delete pxc-backup backup-001
    ```
    
3.  Delete the PXC restore
    
    ```
    $ kubectl delete pxc-restore restore-001
    
    $ kubectl delete pxc-restore restore-002
    ```
    
4.  Delete the Persistent Volume Claim
    
    ```
    $ kubectl delete pvc datadir-db-cluster-pxc-0
    
    $ kubectl delete pvc datadir-db-cluster-pxc-1
    
    $ kubectl delete pvc datadir-db-cluster-pxc-2
    ```
    
5.  Delete the secrets
    
    ```
    $ kubectl delete secret db-cluster-secrets
    
    $ kubectl delete secret internal-db-cluster
    
    $ kubectl delete secret vultr-s3-secret
    
    $ kubectl delete secret offsite-s3-secret
    ```
    
6.  Delete Percona Distribution for MySQL Operator
    
    ```
    $ kubectl delete -f https://raw.githubusercontent.com/percona/percona-xtradb-cluster-operator/v1.10.0/deploy/bundle.yaml
    ```
    
7.  Delete files in your Object Storage
    

### More Information

-   [Percona Distribution for MySQL Operator Documentation](https://www.percona.com/doc/kubernetes-operator-for-pxc/index.html)