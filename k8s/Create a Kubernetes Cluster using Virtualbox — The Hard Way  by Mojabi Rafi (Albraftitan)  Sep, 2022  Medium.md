Hi everyone! Have you ever thought about having a production like Kubernetes cluster privately so that you can play with Kubernetes with no fear and run various tests at ease?

To achieve this goal, we can go for following approaches:

1.  Run Kubernetes cluster on local machine using mini-kube
2.  Run Kubernetes cluster on of the cloud service like GoogleCloud, AWS etc.
3.  Run Kubernetes cluster on your local machine using Virtualbox and virtual machines.

The first solution has its own limitations. The major limitation of Minikube is that **the local cluster can consist of only one node**, and therefore is not a great simulation of a production-level, multi-node Kubernetes cluster.

The second solution works, yet it is costly. Learning Kubernetes and testing on it would take months. Therefore non of the cloud services free trial would be enough and at the end it gonna costs us:)

The third solution is more complex that other two. Yet it provides us with significant flexibility. It is very like production level kubernetes clusters. We are able to add nodes, drain nodes, add master nodes, work with master’s etcd and use our desired Kubernetes network. The other important benefit of setting up Kubernetes cluster with this approach is that as we have to setup everything from the scratch, we understand better what is happening under-hood. Lets go for it then!

## **Setting up Kubernetes cluster in action**

In the nutshell, we are going to implement the following architecture for setting up our Kubernetes cluster.

![](https://miro.medium.com/max/1400/1*7udyNeQkQQQpsbJuYiV8Sg.png)

Figure 1-Cluster architecture

There are some key-points apparent in the figure 1.

-   We start off by 1 master node
-   we have 3 worker nodes
-   Each VM has 2 network interfaces.
-   One network interface is attached to default NAT network in virtualbox. the purpose of this network is to provide the node internet access.
-   Another network interface is attached to a Host-Only network in virtualbox. All inside cluster communications would be through this network.

**Prepare VirtualBox machines for deploying Kubernetes cluster**

According to figure-1, We need a private network that nodes can communicate with each other though. To create one go to File -> Host Network Manager. Create new network there with no DHCP Enable option. Remember the network range created! we gonna use it soon. Mine is _192.168.60.1/24 ._

![](https://miro.medium.com/max/1268/1*9ZnFgOdCt8SkqWtXZffr-g.png)

**VM’s OS and configuration:**

I assume you are familiar with creating new virtual machine in virtual box. if not, no worries! just Do google it :). We gonna use following setup as of our VMs:  
Master(s):  
— Name: master# (number of master in the cluster)  
— OS: ubuntu-server-18.04  
— CPU: 2core  
— Memory: 2GB  
Workers:  
— OS: ubuntu-server-18.04  
— CPU: 1core  
— Memory: 1GB

Please note each of the VMs should benefit from 2 networks. One default NAT network which provides internet access to our node, Another attached to Host Only Network created in previous step.

![](https://miro.medium.com/max/1292/1*3Rjm2DAYcdNTjWiOP-2TTA.png)

![](https://miro.medium.com/max/1354/1*Ptr-yb33HBmzJaB5I-KXbA.png)

![](https://miro.medium.com/max/1400/1*B9RjJbRyGzNBI_beiLumjw.gif)

After setting up the VM virtual hardware, it is time to install our operating system. There is no trick on it, except we have to assign our desired IP address in the range provided by Host Only adapter manually. In my case, I will assign _192.168.60.11_ to my first Master node. We will extend this lab in the next tutorials so lets reserve some IPs from _192.168.60.11_ to _192.168.60.20._ Consequently, worker01 node will be 192.168.60.21 and so on.

![](https://miro.medium.com/max/1400/1*mhnGMc44Hnes37iWepXBEQ.gif)

Note: You should not set gateway 192.168.60.1 for any Node, Or you will have to change the default route (The route we need for Internet connection) to the NAT interface after OS install.

Note: Among All packages the installer ask you about, only select OpenSSH package to be installed.

Having OS installed and VM up and running, you should be able to ping the IP address of the VM.

![](https://miro.medium.com/max/1400/1*S43p0nbfq7Ji06g_YCwfaQ.gif)

We repeat the same steps for the workers too. At the end, the final node should represent the following table

![](https://miro.medium.com/max/608/1*nQ3gRRVKfbGV04iz-n-0fg.png)

After finishing the setup process, you should be able to ping all the targets from your host machine and from inside each of the nodes.

**Setup Kubernetes cluster using kubeadm tool**

So far we prepared the required infrastructure to deploy our Kubernetes cluster. Lets begin setting up the cluster :). There are several ways to setup Kubernetes cluster. We are going to leverage \`_kubeadm\`_ tool. kubeadm enable us to create Kubernetes cluster that will pass [Kubernetes Conformance tests](https://kubernetes.io/blog/2017/10/software-conformance-certification/). kubeadm also support providing tokens, cluster upgrade etc.

Install required components

Each of the nodes need the following components available:

-   kubelet: The kubelet is the primary “node agent” that runs on each node. It can register the node with the apiserver using one of: the hostname; a flag to override the hostname; or specific logic for a cloud provider.
-   kubectl: The Kubernetes command-line tool, [kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/), allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.
-   kubeadm: Using `kubeadm`, you can create a minimum viable Kubernetes cluster that conforms to best practices.

**Step 1- Installing required components**

On each nodes, add the kubernetes repository

```
sudo apt update && sudo apt install -y curlcurl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key addsudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

Having required repository added, install required components. At the time, the latest version of kubernets is 1.25.0. Yet as for more stability, we are going to use kubernetes version 1.22.10 for our cluster.  
First, lets list what versions are available:

```
apt list -a kubeadm
```

![](https://miro.medium.com/max/1000/1*66wf57gAOa0825lgMRI70w.png)

```
sudo apt install -y kubeadm=1.22.10-00 kubelet=1.22.10-00 kubectl=1.22.10-00
```

Note: In case you copy the command above, please note the “-” inside the code. It can be copied as “ — ” which fail the command above.

It is recommended to hold the version on installed packages. We really don’t want to mess up with kubernetes auto updates :)

```
sudo apt-mark hold kubeadm kubelet kubectl
```

Check the version of installed components and it should be as version we just installed

`kubead version`

![](https://miro.medium.com/max/1400/1*1ZNDOOwVAcI64vXF9uX8aQ.png)

Next step is to install Docker on the nodes. The official document on installing docker on Ubunut can be found [here](https://docs.docker.com/engine/install/ubuntu/). Yer for the sake of simplicity, we install it using a simple command as follow:

```
sudo apt install -y docker.io
```

You can verify installation using

> docker version

![](https://miro.medium.com/max/1400/1*c0H-L7BcY68KdJzfBNgxWg.png)

Docker will not start on system startup. meanwhile it is a requisite for kubernetes cluster to be functional. Do Not forget to enable its service to make sure docker will get up and work on each node restart

```
sudo systemctl enable docker
```

**Changing Docker Cgroup Driver**

It is crucial to change docker cgroup Driver after install or you get error regarding “Docker group driver detected cgroupfs instead systemd” on cluster initialization. It is easy, and should be done on all nodes (master and workers).

```
sudo cat <<EOF | sudo tee /etc/docker/daemon.json{ "exec-opts": ["native.cgroupdriver=systemd"],"log-driver": "json-file","log-opts":{ "max-size": "100m" },"storage-driver": "overlay2"}EOF
```

Restart Docker and make sure it is up and running

> sudo systemctl restart docker

You can check if the Cgroup driver is changes to systemd

> sudo docker info | grep -i cgroup

![](https://miro.medium.com/max/1342/1*x_ysV8Ga9Ka6xt9jRO5EtA.gif)

**Disable swap and enable ip\_forward**

Kubernetes cluster needs the to disabled swap for the sake of security issues and performance. To disable swap permanently on all node, run the following commands:

> sudo swapoff -a

Now to disable swap permanently, open _/etc/fstab_ and comment line containing swap by adding # in the beginning.

![](https://miro.medium.com/max/1342/1*Lf7u9f9pNNwp0UCjOdNp6g.gif)

Now we need to enable IP\_Forward in kernel

```
sudo modprobe overlaysudo modprobe br_netfiltersudo tee /etc/sysctl.d/kubernetes.conf<<EOFnet.bridge.bridge-nf-call-ip6tables = 1net.bridge.bridge-nf-call-iptables = 1net.ipv4.ip_forward = 1EOFsudo sysctl --system
```

**Step 2- Initializing Kubernetes Master Node**

Having are required components installed and configured, We shall initialize the master node. Please note that in our lab and for now, our control plane consists of only one node (master01). All the required components for [control plane](https://kubernetes.io/docs/concepts/overview/components/) including etcd will be installed on master node as a docker container.

Please note that from the first place, the secondary private network was added for internal kubernetes components communications. Therefore we shall bound the _api-adrvertisement-address_ to this network only.

```
sudo kubeadm init --apiserver-advertise-address 192.168.60.11 --control-plane-endpoint 192.168.60.11
```

![](https://miro.medium.com/max/1400/1*GDIFEb-y4DxgFBC5WYY7QQ.gif)

it takes some time to download all the required docker images and initialize it depends on you Internet connection speed.

![](https://miro.medium.com/max/1400/1*ABPPGEtQ9NDZVZphTUtuwA.gif)

When the master node initialization process done, the commands to run on workers and probable other masters to join the cluster will be provided by kubeadm

![](https://miro.medium.com/max/1400/1*ith0CDN6-I3tSAulvrwnjA.png)

To connect to our cluster using kubectl, we need to apply following changes on the remote machine.

> mkdir -p $HOME/.kube  
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
> sudo chown $(id -u):$(id -g) $HOME/.kube/config

lets setup kubectl installed on master machine and then run _kubectl get node command to see the output._

![](https://miro.medium.com/max/1400/1*wzljYSsGReXabiNkrcK11A.gif)

As you can see the master node is already added to the cluster.

Copy and paste the command to join workers to the cluster somewhere. We gonna need it to join workers to our clusters

Note: All the previous steps should be applied to workers as well, except that workers dont need kubectl at all.

Having all the above steps done for workers, use the command you copied and join the workers to the cluster. After joining the workers to the cluster, the output of _kubectl get node_ should show new joined nodes as worker.

![](https://miro.medium.com/max/1330/1*82P6lGX2BLVC-1ZHqJeTCQ.png)

**Step 3- Installing Kubernetes Network**

You probably noticed in kubectl get node command output that all the nodes are in “NotReady” state. The reason is we still did not create any Kubernetes network for the cluster. This network is different from our infrastructure network, and is responsible to create an internal network for all inside-cluster components so that they can communicate with each other through it.

We are going to use [Flannel](https://github.com/flannel-io/flannel), one of most straight forward networks for kubernetes. There are [other network-add on](https://kubernetes.io/docs/concepts/cluster-administration/addons/) with more feature and capabilities. Yet Flannel serve us best and also dosent add up any unnecessary complexity.

To install Flannel network add-on:

1.  `kubectl apply -f [https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml](https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml)`

It takes some time to download all required docker images and bring the containers up.

![](https://miro.medium.com/max/1400/1*OfHckbX_0Av-U8E1RAvIKA.gif)

Congratulations! The Kubernetes cluster is up and ready to use.

Final Words:  
I started to write about what I get during last 3 years working in DevOps in a series. We will go through most of experiences I get by doing. For that purpose, we need this cluster. I started to explain it in hard-way, so we can get what exactly is happening under hood. Because performing all the steps every time manually is a time consuming process, I provided [an automated way to deploy the exact same cluster](https://medium.com/@mojabi.rafi/setup-kubernetes-cluster-on-virtual-box-an-automated-way-7a62138ed36c), which will take few minutes based on your Internet connection speed. We will use this automated way to do our experiments on Kubernetes until we start working with cloud based Kubernetes engines such as GKE.  
Please provide me feed backs and ask questions to encourage me.