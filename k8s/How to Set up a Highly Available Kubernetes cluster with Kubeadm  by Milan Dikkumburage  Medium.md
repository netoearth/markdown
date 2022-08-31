## **Overview**

This article explains how to set up a highly available kubernetes cluster using kubeadm.

You can set up an HA cluster:

-   With stacked control plane nodes, where etcd nodes are colocated with control plane nodes
-   With external etcd nodes, where etcd runs on separate nodes from the control plane

**Stacked etcd topology**

In stacked etcd mode, control plane node having apiserver ,controller-manager,scheduler & etcd.

However, a stacked cluster runs the risk of failed coupling. If one node goes down, both an etcd member and a control plane instance are lost, and redundancy is compromised. You can mitigate this risk by adding more control plane nodes

![](https://miro.medium.com/max/1400/0*y5536PsznYgax1AL)

**External etcd topology**

In external etcd mode, etcd is deployed on a separate set of nodes from the Kubernetes control plane components as shown below .

![](https://miro.medium.com/max/1400/0*gYdAeyyGH8Nbxz6T)

> _In this article we are using external etcd method_

**Minimum Server Requirements**

-   A compatible Linux host. (Linux distributions based on Debian and Red Hat, and those distributions without a package manager)
-   2 GB or more of RAM per machine
-   Swap disabled. You **MUST** disable swap in order for the kubelet to work properly
-   SSH access between each master,etcd & worker nodes

**Setup Environment**

I have tested this setup DigitalOcean & aws cloud.But you can also work with bare metal or virtual machines. In this article i’m using centos 7 EC2 in aws.I’m using aws ELB as the load balancer.

![](https://miro.medium.com/max/1052/1*4Tu9ZdCg1hK1wYzGMba8mg.png)

**Set Hostname**

Execute below command each node. you can change the hostname as you like

**Disable swap**

> **Note** :- Please execute these steps on all Master,Etcd,Worker nodes

```
swapoff -a  
```

**Letting iptables see bridged traffic**

> **Note** :- Please execute these steps on all Master,Etcd,Worker nodes

**Installing a container runtime**

> **Note** :- Please execute these steps on all Master,Etcd,Worker nodes

## **Install docker engine**

## Post-installation steps for Linux docker

```
sudo usermod -aG docker $USER
```

**Configure Docker to start on boot**

```
sudo systemctl enable docker.service
```

**Installing kubeadm, kubelet and kubectl**

\# Set SELinux in permissive mode (effectively disabling it)

#Enable the cgroup driver

**Setup ETCD external nodes**

**Configure the kubelet to be a service manager for etcd**

> **Note** :- Please execute these steps on all Etcd nodes

Check the kubelet status to ensure it is running.

```
sudo systemctl status kubelet
```

**Create Configuration file for kubeadm**

> **Note** :- This step is to be performed on One of first etcd node.

Generate one kubeadm configuration file for each host that will have an etcd member running on it using the following script

Create a file called bash1.sh and add the below file & replace with your etcd private IPs & hostname

#Execute the Script

```
chmod +x bash1.sh./bash1.sh
```

**Generate the certificate authority**

```
kubeadm init phase certs etcd-ca
```

This creates two files

-   /etc/kubernetes/pki/etcd/ca.crt
-   /etc/kubernetes/pki/etcd/ca.key

**Create certificates for each member**

Create bash2.sh and add the below file & replace with your etcd private IPs

#Execute the Script

```
chmod +x bash2.sh./bash2.sh
```

#In the etcd node 1 go to the /root/.ssh path & create ssh file & config file as below

Then ssh to each from etcd-01

ssh centos@172.31.80.58

![](https://miro.medium.com/max/1400/1*8EdVjxpd2NKheWcnxihoXQ.png)

ssh centos@172.31.91.72

![](https://miro.medium.com/max/1400/1*muicqUzOq0IBdmaDJ4NaIg.png)

ssh centos@172.31.84.201

![](https://miro.medium.com/max/1400/1*OsZfesQQisOdoW7C2FOAwQ.png)

**Copy certificates and kubeadm configs**

Create bash3.sh and add the below file & replace with your etcd private IPs

#Execute the Script

```
chmod +x bash3.sh./bash3.sh
```

Create bash4.sh and add the below file & replace with your etcd private IPs

#Execute the Script

```
chmod +x bash4.sh./bash4.sh
```

Copy the config to master node

> **Note** :- This step is to copy certificate to only one master node.

Create bash5.sh and add the below file & replace with your etcd private IPs

#Execute the Script

```
chmod +x bash5.sh./bash5.sh
```

**Create the static pod manifests**

> **Note** :- run commands on etcd each hosts

**#Etcd node 01**

![](https://miro.medium.com/max/1400/1*swCHWfUjvmgiod6t_NLQxw.png)

**#Etcd node 02**

![](https://miro.medium.com/max/1400/1*-Z7vLZWzr3KsYqTbWiyL1Q.png)

**#Etcd node 03**

![](https://miro.medium.com/max/1400/1*wHnQk_aGVAe8xro_8Z_TZw.png)

**Check the cluster health**

Create bash5.sh in etcd node 01 and add the below file & replace with your etcd private IPs

#Execute the Script

```
chmod +x bash6.sh./bash6.sh
```

Result as below .

![](https://miro.medium.com/max/1400/1*wO2eOK6Z8zfcckmwZPRhWQ.png)

You can get the ETCD\_TAG execute below command

```
kubeadm config images list
```

**Create Network Load Balancer in AWS & attach the master nodes to LB.**

Create a network load balancer

![](https://miro.medium.com/max/1400/1*ISJrYHNcotq7Ll7ShIevOg.png)

![](https://miro.medium.com/max/1400/1*nPVdWbMKe9mul5x_CbjQmw.png)

![](https://miro.medium.com/max/1400/1*DXYuxGQIrdpqkSAKpnNejw.png)

#Set the port 6443 and create a new target & attached the master nodes.

![](https://miro.medium.com/max/1400/1*WmRPd4mXgFozAfQqzYIVmA.png)

![](https://miro.medium.com/max/1400/1*xFGDuEKtt1XEYuu95XPH3Q.png)

#Select include as pending & create target group

![](https://miro.medium.com/max/1400/1*od2xOxB3iqnfipqWsZu5ZQ.png)

#once target create attach the target group to ELB & create load balancer

![](https://miro.medium.com/max/1400/1*78jjDD3Ud3BG_RjFTk3VBQ.png)

#Once load balancer active get the load balancer dns endpoint point the & Create CNAME record.

![](https://miro.medium.com/max/774/1*x93z7ANHMXzBT_Bp6Xp6aw.png)

**Set up the first control plane node**

> **Note** :- This step is to be performed on certificate copy master node.

Create vi kubeadm-config.yaml Replace Load Balancer DNS & etcd private IP.

Deploy kubernetes Cluster .

```
sudo kubeadm init — config kubeadm-config.yaml — upload-certs
```

Output result as below

![](https://miro.medium.com/max/1400/1*xnVdbUfX6n1JqHU0BItI_Q.png)

#once deploy the cluster execute below command

**Adding master nodes**

Copy the master node join command and execute in each master 02 & master 03 node

![](https://miro.medium.com/max/1400/1*hZDFywoKKvUdBtJdPdsnAg.png)

**Adding worker nodes**

Copy the worker node join command and execute in each worker 01 ,worker 02 & worker 03 node

![](https://miro.medium.com/max/1400/1*COwEonxf72wXlQrqnPNsdQ.png)

**Install wavent**

#You can verify load balancer working execute below command any of master node

```
kubectl cluster-info
```

![](https://miro.medium.com/max/1400/1*2LoHyGCo5ozEsuRdSaf8ig.png)

Cluster details

```
kubectl get node -o wide
```

![](https://miro.medium.com/max/1400/1*Hn-ZMzsgygghz52YuAofpw.png)

![](https://miro.medium.com/max/1400/1*MntjpGnrahhJY7ErQnVH3w.png)

Referance