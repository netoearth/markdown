## Introduction

Kubernetes, also known as K8s, is an open-source platform for container orchestration. It manages clusters meant to run containerized applications, scaling deployments, and provides a platform for deployment automation. A container is a form of operating system virtualization that isolates code and all its dependencies, so the application runs securely, quickly, and reliably from other computing environments. Due to the deprecation of Docker runtime by Kubernetes, a new API plugin for container runtimes called CRI (Container Runtime Interface) has been introduced, enabling kubelet to use a wide variety of container runtimes. CRI-O is an implementation of the Kubernetes CRI to enable using OCI (Open Container Initiative).

After deploying a small application in a container, you might need to scale it on many servers. Due to the complexity of this operation, there should be an automation tool responsible for allocating resources to the application on different servers, monitoring the application and servers. This is where Kubernetes comes in. You need at least a **master node** and a **worker node** to have a cluster. Later on, you can expand the cluster by adding as many worker nodes as you need. This enables your application's containers to achieve high performance and high availability by distributing it between available hosts within the cluster.

In this article, you will learn how to deploy a Kubernetes cluster on Ubuntu 20.04.

## Prerequisites

-   Deploy two or more [fully updated](https://www.vultr.com/docs/update-ubuntu-server-best-practices) Vultr Ubuntu 20.04 servers.
    
-   Create a [non-root](https://www.vultr.com/docs/create-a-sudo-user-on-ubuntu-best-practices) user with sudo access on all servers.
    

## 1\. Install CRI-O

Kubernetes requires CRI-O to run. Perform all these steps on all the nodes.

Update the system.

```
$ sudo apt update
```

Load the `overlay` and `br_netfilter` modules.

```
$ sudo modprobe overlay



$ sudo modprobe br_netfilter
```

Create a sysctl config file to enable IP forwarding.

```
$ sudo nano /etc/sysctl.d/99-kubernetes-cri.conf
```

Add the following code to the file. Save and close the file.

```
net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

net.ipv4.ip_forward = 1
```

Reload sysctl.

```
$ sudo sysctl --system
```

Change user to `root`.

```
$ sudo -i
```

Add repository version information.

```
# export OS=xUbuntu_20.04



# export VERSION=1.18
```

Add CRI-O repositories.

```
# sudo echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

# sudo echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list



# sudo curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key add -

# sudo curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -
```

Exit `root`.

```
# exit
```

Update the system.

```
$ sudo apt update
```

Install CRI-O.

```
$ sudo apt install cri-o cri-o-runc -y
```

Find the path of container monitoring utility.

```
$ which conmon
```

Edit the CRI-O configuration file `/etc/crio/crio.conf`.

```
$ sudo nano /etc/crio/crio.conf
```

The file should have the following changes:

```
conmon = "/usr/bin/conmon"



registries = [

    "quay.io",

    "docker.io",

    "gcr.io",

    "eu.gcr.io",

    "k8s.gcr.io"

]
```

Reload the daemon.

```
$ sudo systemctl daemon-reload
```

Start the CRI-O service.

```
$ sudo systemctl start crio
```

Enable CRI-O service to startup at system boot.

```
$ sudo systemctl enable crio
```

Check the CRI-O service status.

```
$ sudo systemctl status crio
```

## 2\. Disable Swap Memory

Swap memory needs to be disabled for Kubernetes to run normally. Perform these steps for all the nodes.

Disable the swap memory.

```
$ sudo swapoff -a
```

Reboot the system for changes to take effect.

```
$ sudo reboot
```

## 3\. Install Kubernetes

Perform these steps on all the nodes.

Install all required packages.

```
$ sudo apt install -y apt-transport-https curl
```

Add Kubernetes GPG Key.

```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Add Kubernetes APT Repository.

```
$ sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

Install Kubeadm, Kubelet, and Kubectl. This also installs Kubernetes-cni to enable container networking in the cluster.

```
$ sudo apt install -y kubelet kubeadm kubectl



$ sudo apt-mark hold kubelet kubeadm kubectl
```

Reload the daemon.

```
$ sudo systemctl daemon-reload
```

Restart the CRI-O service.

```
$ sudo systemctl restart crio
```

Restart the kubelet service.

```
$ sudo systemctl restart kubelet
```

Initialize the **master node** using kubeadm. This should only be run on the master node. Make sure you copy and save the worker node join command and tokens where you can retrieve them for later use.

```
$ sudo kubeadm init --pod-network-cidr=192.168.10.0/24 --ignore-preflight-errors=all
```

Create new `.kube` configuration directory. This should only be run on the master node.

```
$ sudo mkdir -p $HOME/.kube
```

Copy the configuration `admin.conf` from `/etc/kubernetes` directory to the new `.kube` directory. This should only be run on the master node.

```
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Change the permissions to the configuration directory. This should only be run on the master node.

```
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Check kubeadm version. This should only be run on the master node.

```
$ kubeadm version
```

## 4\. Configure Pod Network

Install the **Weave network plugin** to allow communication between the master and worker nodes. This should only be run on the master node.

```
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

## 5\. Join Worker Node to the Cluster

If you lost the join command, you could find it by running this command on the master node.

```
$ kubeadm token create --print-join-command
```

Join worker nodes to master node. Perform this step for worker nodes only. Use the join instructions generated and saved in the kubernetes initialization step. Make sure you add the `--ignore-preflight-errors=all` flag. For example:

```
$ sudo kubeadm join 192.0.2.10:6443 --token 2qmwga.qyejfbo9vouiowlt \ --discovery-token-ca-cert-hash  sha256:083a2a20c8de9254100f1b37b4be1999946aee6f34791985c80d9eced9618e94 --ignore-preflight-errors=all
```

From the **master** node, check all nodes status.

```
$ kubectl get nodes
```

From the **master** node, verify pod namespaces.

```
$ kubectl get pods --all-namespaces
```

## 6\. Deploy Nginx in Kubernetes

Deploy an Nginx application in Kubernetes on the master node using YAML.

Create a pod.

```
$ kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

Check pod status.

```
$ kubectl get pods
```

Check all pods' information.

```
$ kubectl describe pods
```

## Conclusion

You have successfully learned how to install Container Runtime Interface, Kubernetes cluster, join worker nodes to the cluster and deploy a containerized application on your server. You can now deploy containerized applications and scale them to handle more user traffic.

## More Information

For more information on Kubernetes, please see the [official documentation](https://kubernetes.io/docs/home/).