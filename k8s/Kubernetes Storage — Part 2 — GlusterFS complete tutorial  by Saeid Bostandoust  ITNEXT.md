![](https://miro.medium.com/max/1400/1*i5Tp6PHEccn70-4NDRe5ww.jpeg)

If you are interested in Kubernetes Storage, this series of articles is for you. In this part, I will explain how to up and run GlusterFS and use it in Kubernetes. So, let’s get started.

[Kubernetes Storage — Part 1 — NFS complete tutorial](https://itnext.io/kubernetes-storage-part-1-nfs-complete-tutorial-75e6ac2a1f77)

## Resource Requirements:

-   A running Kubernetes cluster. 1.18+ is suggested.
-   Three nodes for up and run GlusterFS with Replication.

I will run the steps on **Ubuntu**\-based systems. I suggest you do too.

**Tip: k3s does not support GlusterFS volumes.**

**A three-nodes Kubernetes cluster:**

```
Node1OS: Ubuntu 20.04Kubernetes: 1.20.7 (installed via kubespray)FQDN: node001.b9tcluster.localIP Address: 192.168.12.4Node2OS: Ubuntu 20.04Kubernetes: 1.20.7 (installed via kubespray)FQDN: node002.b9tcluster.localIP Address: 192.168.12.5Node3OS: Ubuntu 20.04Kubernetes: 1.20.7 (installed via kubespray)FQDN: node003.b9tcluster.localIP Address: 192.168.12.6
```

**Three nodes for GlusterFS:**

```
Storage1OS: Ubuntu 20.04FQDN: node004.b9tcluster.localIP Address: 192.168.12.7Storage: /dev/sdb1 mounted on /gluster/volumeStorage: /dev/sdb2 mounted on /gluster/heketi (Heketi)Role: ReplicaStorage2OS: Ubuntu 20.04FQDN: node005.b9tcluster.localIP Address: 192.168.12.8Storage: /dev/sdb1 mounted on /gluster/volumeStorage: /dev/sdb2 mounted on /gluster/heketi (Heketi)Role: ReplicaStorage3OS: Ubuntu 20.04FQDN: node006.b9tcluster.localIP Address: 192.168.12.9Storage: /dev/sdb1 mounted on /gluster/volumeStorage: /dev/sdb2 mounted on /gluster/heketi (Heketi)Role: Arbiter
```

In the above config that is considered for the GlusterFS cluster, the **Arbiter** node is a node that is not really replicate data! Instead of data, it saves metadata of the files. We use it to prevent storage **split-brain** exactly with two replicas only.

## 1- Up and Run GlusterFS cluster:

To install and configure GlusterFS, follow the following steps:

```
# Install GlusterFS server on all STORAGE nodes.apt install -y glusterfs-serversystemctl enable --now glusterd.service
```

Setup a Gluster volume with 2 replicas and 1 arbiter:

```
# Only on Node1 of STORAGE nodes.gluster peer probe node005.b9tcluster.localgluster peer probe node006.b9tcluster.localgluster volume create k8s-volume replica 2 arbiter 1 transport tcp \  node004:/gluster/volume \  node005:/gluster/volume \  node006:/gluster/volumegluster volume start k8s-volume
```

To view the volume information, run the following command:

```
gluster volume info k8s-volumeVolume Name: k8s-volumeType: ReplicateVolume ID: dd5aac80-b160-4281-9b22-00ae95f4bc0cStatus: StartedSnapshot Count: 0Number of Bricks: 1 x (2 + 1) = 3Transport-type: tcpBricks:Brick1: node004:/gluster/volumeBrick2: node005:/gluster/volumeBrick3: node006:/gluster/volume (arbiter)Options Reconfigured:transport.address-family: inetstorage.fips-mode-rchecksum: onnfs.disable: onperformance.client-io-threads: off
```

## 2- Prepare Kubernetes worker nodes:

Now, to able Kubernetes workers to connect and use GlusterFS volume, you need to install glusterfs-client in WORKER nodes.

```
apt install glusterfs-client
```

**Important tip!** Each storage solution may need client packages to connect to the storage server. You should install them in all Kubernetes worker nodes. For GlusterFS, the **glusterfs-client** package is needed.

## 3- Discovering GlusterFS in Kubernetes:

GlusterFS cluster should be discovered in the Kubernetes cluster. To do that, you need to add an Endpoints object that points to the servers of the GlusterFS cluster.

```
apiVersion: v1kind: Endpointsmetadata:  name: glusterfs-cluster  labels:    storage.k8s.io/name: glusterfs    storage.k8s.io/part-of: kubernetes-complete-reference    storage.k8s.io/created-by: ssbostansubsets:  - addresses:      - ip: 192.168.12.7        hostname: node004      - ip: 192.168.12.8        hostname: node005      - ip: 192.168.12.9        hostname: node006    ports:      - port: 1
```

## 4- Using GlusterFS in Kubernetes:

## Method 1 — Connecting to GlusterFS directly with Pod manifest:

To connect to the GlusterFS volume directly with Pod manifest, use the GlusterfsVolumeSource in the Pod. Here is an example:

```
apiVersion: v1kind: Podmetadata:  name: test  labels:    app.kubernetes.io/name: alpine    app.kubernetes.io/part-of: kubernetes-complete-reference    app.kubernetes.io/created-by: ssbostanspec:  containers:    - name: alpine      image: alpine:latest      command:        - touch        - /data/test      volumeMounts:        - name: glusterfs-volume          mountPath: /data  volumes:    - name: glusterfs-volume      glusterfs:        endpoints: glusterfs-cluster        path: k8s-volume        readOnly: no
```

## Method 2 — Connecting using the PersistentVolume resource:

To create the PersistentVolume object for the GlusterFS volume, use the following manifest. The storage size does not take any effect.

```
apiVersion: v1kind: PersistentVolumemetadata:  name: glusterfs-volume  labels:    storage.k8s.io/name: glusterfs    storage.k8s.io/part-of: kubernetes-complete-reference    storage.k8s.io/created-by: ssbostanspec:  accessModes:    - ReadWriteOnce    - ReadOnlyMany    - ReadWriteMany  capacity:    storage: 10Gi  storageClassName: ""  persistentVolumeReclaimPolicy: Recycle  volumeMode: Filesystem  glusterfs:    endpoints: glusterfs-cluster    path: k8s-volume    readOnly: no
```

## Method 3 — Dynamic provisioning using StorageClass:

It’s time to explain the hardest section, GlusterFS with Kubernetes StorageClass, to achieve dynamic storage provisioning. To achieve this model, in addition to the GlusterFS cluster, we need Heketi. Heketi is RESTful-based volume management for GlusterFS. We have to use Heketi because Kubernetes needs to access it to create volumes dynamically.

**Requirements:**

-   A running GlusterFS for storing the Heketi database.
-   Available raw storage devices on GlusterFS cluster nodes.
-   SSH key to connect Heketi to GlusterFS nodes.

**Architecture and Scenario:**

To construct volumes, the Kubernetes-deployed Heketi must connect with GlusterFS nodes. This communication should be accomplished by using SSH. So, we need a private key that is authorized on GlusterFS nodes. On the other hand, Heketi should be aware of which raw devices are accessible for the creation of partitions and GlusterFS bricks. A topology file should contain a list of cluster nodes as well as a list of accessible raw storage. Heketi has its own database to save all about created bricks. To save Heketi data, I have used the existing GlusterFS cluster that we deployed in previous sections.

Let’s assume some existing materials:

-   Private key for connecting to GlusterFS nodes via SSH: **heketi-ssh-key**
-   Available raw devices on all GlusterFS nodes: **/dev/sdc (100GB)**

**3.1: Create a GlusterFS volume to store the Heketi database:**

```
gluster volume create heketi-db-volume replica 3 transport tcp \  node004:/gluster/heketi \  node005:/gluster/heketi \  node006:/gluster/heketigluster volume start heketi-db-volume
```

**3.2: Create a Kubernetes secret to store SSH private key:**

```
kubectl create secret generic heketi-ssh-key-file \  --from-file=heketi-ssh-key
```

**3.3: Create Heketi “config.json” file:**

```
{  "_port_comment": "Heketi Server Port Number",  "port": "8080",  "_use_auth": "Enable JWT authorization.",  "use_auth": true,  "_jwt": "Private keys for access",  "jwt": {    "_admin": "Admin has access to all APIs",    "admin": {      "key": "ADMIN-HARD-SECRET"    }  },  "_glusterfs_comment": "GlusterFS Configuration",  "glusterfs": {    "executor": "ssh",    "_sshexec_comment": "SSH username and private key file",    "sshexec": {      "keyfile": "/heketi/heketi-ssh-key",      "user": "root",      "port": "22"    },    "_db_comment": "Database file name",    "db": "/var/lib/heketi/heketi.db",    "loglevel" : "debug"  }}
```

**3.4: Create the cluster “topology.json” file:**

More than one raw device can be used. The Heketi knows how to manage them. In addition, Heketi can manage several clusters simultaneously.

```
{  "clusters": [    {      "nodes": [        {          "node": {            "hostnames": {              "manage": [                "node004.b9tcluster.local"              ],              "storage": [                "192.168.12.7"              ]            },            "zone": 1          },          "devices": [            "/dev/sdc"          ]        },        {          "node": {            "hostnames": {              "manage": [                "node005.b9tcluster.local"              ],              "storage": [                "192.168.12.8"              ]            },            "zone": 1          },          "devices": [            "/dev/sdc"          ]        },        {          "node": {            "hostnames": {              "manage": [                "node006.b9tcluster.local"              ],              "storage": [                "192.168.12.9"              ]            },            "zone": 1          },          "devices": [            "/dev/sdc"          ]        }      ]    }  ]}
```

**3.5: Create a Kubernetes ConfigMap of Heketi config and topology:**

```
kubectl create configmap heketi-config \  --from-file=heketi.json \  --from-file=topology.json
```

**3.6: Up and Run the Heketi in Kubernetes:**

```
apiVersion: v1kind: Servicemetadata:  name: heketi  labels:    app.kubernetes.io/name: heketi    app.kubernetes.io/part-of: glusterfs    app.kubernetes.io/origin: kubernetes-complete-reference    app.kubernetes.io/created-by: ssbostanspec:  type: NodePort  selector:    app.kubernetes.io/name: heketi    app.kubernetes.io/part-of: glusterfs    app.kubernetes.io/origin: kubernetes-complete-reference    app.kubernetes.io/created-by: ssbostan  ports:    - port: 8080---apiVersion: apps/v1kind: Deploymentmetadata:  name: heketi  labels:    app.kubernetes.io/name: heketi    app.kubernetes.io/part-of: glusterfs    app.kubernetes.io/origin: kubernetes-complete-reference    app.kubernetes.io/created-by: ssbostanspec:  replicas: 1  selector:    matchLabels:      app.kubernetes.io/name: heketi      app.kubernetes.io/part-of: glusterfs      app.kubernetes.io/origin: kubernetes-complete-reference      app.kubernetes.io/created-by: ssbostan  template:    metadata:      labels:        app.kubernetes.io/name: heketi        app.kubernetes.io/part-of: glusterfs        app.kubernetes.io/origin: kubernetes-complete-reference        app.kubernetes.io/created-by: ssbostan    spec:      containers:        - name: heketi          image: heketi/heketi:10          ports:            - containerPort: 8080          volumeMounts:            - name: ssh-key-file              mountPath: /heketi            - name: config              mountPath: /etc/heketi            - name: data              mountPath: /var/lib/heketi      volumes:        - name: ssh-key-file          secret:            secretName: heketi-ssh-key-file        - name: config          configMap:            name: heketi-config        - name: data          glusterfs:            endpoints: glusterfs-cluster            path: heketi-db-volume
```

**3.7: Load the cluster topology into Heketi:**

```
kubectl exec POD-NAME -- heketi-cli \  --user admin \  --secret ADMIN-HARD-SECRET \  topology load --json /etc/heketi/topology.json
```

Replace **POD-NAME** with the name of your Heketi pod.

If everything goes well, you should be able to get the cluster id from Heketi.

```
kubectl exec POD-NAME -- heketi-cli \  --user admin \  --secret ADMIN-HARD-SECRET \  cluster listClusters:Id:c63d60ee0ddf415097f4eb82d69f4e48 [file][block]
```

**3.8: Get Heketi NodePort info:**

```
kubectl get svcheketi NodePort 10.233.29.206 <none> 8080:31310/TCP 41d
```

**3.9: Create Secret of Heketi “admin” user:**

```
kubectl create secret generic heketi-admin-secret \  --type=kubernetes.io/glusterfs \  --from-literal=key=ADMIN-HARD-SECRET
```

**3.10: Create StorageClass for GlusterFS dynamic provisioning:**

Replace needed info with your own.

```
apiVersion: storage.k8s.io/v1kind: StorageClassmetadata:  name: glusterfs  labels:    storage.k8s.io/name: glusterfs    storage.k8s.io/provisioner: heketi    storage.k8s.io/origin: kubernetes-complete-reference    storage.k8s.io/created-by: ssbostanprovisioner: kubernetes.io/glusterfsparameters:  resturl: http://127.0.0.1:31310  clusterid: c63d60ee0ddf415097f4eb82d69f4e48  restauthenabled: !!str true  restuser: admin  secretNamespace: default  secretName: heketi-admin-secret  volumetype: replicate:3
```

**3.11: Create PersistentVolumeClaim to test dynamic provisioning:**

```
apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: testvol  labels:    storage.k8s.io/name: glusterfs    storage.k8s.io/provisioner: heketi    storage.k8s.io/origin: kubernetes-complete-reference    storage.k8s.io/created-by: ssbostanspec:  accessModes:    - ReadWriteOnce  storageClassName: glusterfs  resources:    requests:      storage: 10Gi # Storage size takes effect.
```

## 4- Kubernetes and GlusterFS storage specification:

Before using the GlusterFS, please consider the following tips:

-   **ReadWriteOnce**, **ReadOnlyMany**, **ReadWriteMany** access modes.
-   GlusterFS volumes can be isolated with partitions and bricks.
-   GlusterFS can also be deployed on Kubernetes like native storage.

## The final words:

Congratulations! You have passed all about the Kubernetes and GlusterFS. To learn more about Kubernetes, follow this GitHub repository. This complete Kubernetes reference will be riched soon. All contributions are welcomed. If you find this article is useful, STARgaze the project GitHub repository. Good luck.