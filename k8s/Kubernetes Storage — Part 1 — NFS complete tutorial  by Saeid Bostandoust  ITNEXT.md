![](https://miro.medium.com/max/1400/1*i5Tp6PHEccn70-4NDRe5ww.jpeg)

In this series of articles, I intend to talk about available Kubernetes storage solutions with the complete manual to deploying and connecting them to the Kubernetes. Each article contains a different storage solution that can be used in combination with Kubernetes. This series of tutorials is useful for everyone who knows the Kubernetes storage architecture and concepts. Please note that this series of the tutorial is not a conceptual one about the Kubernetes and I’m not talking about the Kubernetes concepts. So, let’s get the party started with the NFS storage.

## Resource Requirements:

-   A running Kubernetes cluster. 1.18+ is suggested.
-   A running node with some available storage.

I will run the steps on **Ubuntu**\-based systems. I suggest you do too.

For this demonstration, I use two nodes with the following configuration:

**A running node using as the NFS server:**

```
OS: Ubuntu 20.04FQDN: node004.b9tcluster.localIP Address: 192.168.12.7
```

**A single-node Kubernetes cluster:**

```
OS: Ubuntu 20.04Kubernetes: 1.21.5 (k3s distribution)FQDN: node005.b9tcluster.localIP Address: 192.168.12.8
```

## 1- Deploy and configuring the NFS server:

Run the following commands on the node you considered as an NFS server. Please note that you can deploy the NFS server in a clustered way with high availability support. Search it if you want.

```
apt update && apt -y upgradeapt install -y nfs-servermkdir /datacat << EOF >> /etc/exports/data 192.168.12.8(rw,no_subtree_check,no_root_squash)EOFsystemctl enable --now nfs-serverexportfs -ar
```

That commands install the NFS server and export `/data` which is accessible by the Kubernetes cluster. In the case of a multi-node Kubernetes cluster, you should allow all Kubernetes **worker** nodes.

## 2- Prepare Kubernetes worker nodes:

Now to connect to the NFS server, the Kubernetes nodes need an NFS client package to be able to connect to the NFS server. You should run the following commands only on the Kubernetes worker nodes. Control-plane nodes do not need it! Just WORKERS.

```
apt install -y nfs-common
```

**Important tip!** Each storage solution may need client packages to connect to the storage server. You should install them in all Kubernetes worker nodes. For NFS, the **nfs-common** package is needed.

## 3- Using NFS in Kubernetes:

## Method 1 — Connecting to NFS directly with Pod manifest:

To connect to the NFS volume directly with Pod manifest, use the NFSVolumeSource in the Pod. Here is an example:

```
apiVersion: v1kind: Podmetadata:  name: test  labels:    app.kubernetes.io/name: alpine    app.kubernetes.io/part-of: kubernetes-complete-reference    app.kubernetes.io/created-by: ssbostanspec:  containers:    - name: alpine      image: alpine:latest      command:        - touch        - /data/test      volumeMounts:        - name: nfs-volume          mountPath: /data  volumes:    - name: nfs-volume      nfs:        server: node004.b9tcluster.local        path: /data        readOnly: no
```

## Method 2 — Connecting using the PersistentVolume resource:

To create the PersistentVolume object for the NFS volume, use the following manifest. You should note that the storage size does not take any effect.

```
apiVersion: v1kind: PersistentVolumemetadata:  name: nfs-volume  labels:    storage.k8s.io/name: nfs    storage.k8s.io/part-of: kubernetes-complete-reference    storage.k8s.io/created-by: ssbostanspec:  accessModes:    - ReadWriteOnce    - ReadOnlyMany    - ReadWriteMany  capacity:    storage: 10Gi  storageClassName: ""  persistentVolumeReclaimPolicy: Recycle  volumeMode: Filesystem  nfs:    server: node004.b9tcluster.local    path: /data    readOnly: no
```

## Method 3 — Dynamic provisioning using StorageClass:

To provision PersistentVolume dynamically using the StorageClass, you need to install the NFS provisioner. I use the **nfs-subdir-external-provisioner** to achieve that. The following commands install all things we need by using the honey Helm package manager.

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisionerhelm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \  --create-namespace \  --namespace nfs-provisioner \  --set nfs.server=node004.b9tcluster.local \  --set nfs.path=/data
```

![](https://miro.medium.com/max/1400/1*lmzdyjISeeZIekaRc5hisQ.png)

Kubernetes with the NFS StorageClass.

To create the PersistentVolumeClaim use the following manifest:

```
apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: nfs-test  labels:    storage.k8s.io/name: nfs    storage.k8s.io/part-of: kubernetes-complete-reference    storage.k8s.io/created-by: ssbostanspec:  accessModes:    - ReadWriteMany  storageClassName: nfs-client  resources:    requests:      storage: 1Gi
```

![](https://miro.medium.com/max/1400/1*mwSiM3A-OGMr23-bOcmKtg.png)

Kubernetes NFS volume dynamic provisioning.

## 4- Kubernetes and NFS storage specification:

The NFS has the following specifications in the Kubernetes world. You need to consider them before using the NFS storage.

-   **ReadWriteOnce**, **ReadOnlyMany**, **ReadWriteMany** access modes.
-   The storage size does not take any effect!
-   In the case of dynamic provisioning, volumes are separated into different directories. But without any access controls!

## The final words:

To learn more about Kubernetes, follow this GitHub repository. This complete Kubernetes reference will be riched soon. All contributions are welcomed. If you find this article is useful, STARgaze the project GitHub repository. Good luck.