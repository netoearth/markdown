![](https://miro.medium.com/max/1400/1*noL_dNYvof9ipkr4joyYtA.jpeg)

If you are familiar with Kubernetes, you must know Kubernetes installation is one of the challenging topics of Kubernetes. This challenge occurs because a multitude of installation methods exists. In this article, I’ll talk about Kubernetes installation methods, available choices, and best practices. Let’s get started.

## Kubernetes Installation Methods:

Because a great many installation methods exist, I divide these methods into five major types. This division has been done based on usage, ease of installation, and where to install.

## 1- Single-node installation:

This type of installation is suitable for those ones to want to preview Kubernetes or it’s suitable for practices, tests and development purposes. There are many single-node Kubernetes distributions on market but I introduce the most popular ones.

**minikube** is a single-node Kubernetes distribution that releases officially by the Kubernetes community. The latest version of Kubernetes can be running with minikube. Alongside installing Kubernetes, Some Kubernetes addons can be running easily. With minikube, Kubernetes can be deployed on VM, Container, or bare-metal systems. Multiple container runtimes (Docker, Containerd, CRI-O) are supported too.

More information on [https://minikube.sigs.k8s.io](https://minikube.sigs.k8s.io/)

**kind** is a distribution for running Kubernetes cluster inside Docker containers on your local system. kind stands for “Kubernetes in Docker” This distribution is suitable for testing, local development and CI systems. kind is distributed officially by the Kubernetes community.

More information on [https://kind.sigs.k8s.io](https://kind.sigs.k8s.io/)

**k3s** is another useful distribution that is released by Rancher. It’s initially built for IoT and edge computing but can be used for any other purpose. You can run a single-node Kubernetes or a cluster with a couple of commands.

More information on [https://k3s.io](https://k3s.io/)

**k3d** is a helper for running k3s on Docker. It launches a Kubernetes cluster on your local machine just like “kind”. k3d is released by Rancher.

More information on [https://github.com/rancher/k3d](https://github.com/rancher/k3d)

**microk8s** is another installation option. Not only a single-node Kubernetes but a cluster can also be deployed using microk8s. This Kubernetes distribution is released by Canonical — the company that releases Ubuntu — and is available on snappy package manager. Kubernetes can be deployed with a couple of commands. Alongside Kubernetes, a list of various addons is available that they can be deployed easily.

More information on [https://microk8s.io](https://microk8s.io/)

## 2- Manual cluster installation:

This type of installation is used for deploying a minimum viable cluster. Some parts of the installation should be done manually. It’s a preferred way to deploy the Kubernetes cluster for the first time.

**kubeadm** is a tool that is used to deploy a cluster by human hands. It is used to bootstrap Kubernetes components, not provisioning machines. Before bootstrapping the cluster, some actions should be done manually.

More information on [https://kubernetes.io/docs/reference/setup-tools/kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm)

## 3- Automatic cluster installation:

This type of installation is done by using automation tools, scripts, or provider distributed installers. It’s a preferred way for those ones want to deploy production-grade Kubernetes clusters on an on-premises environment or want to manage cluster lifecycle manually.

**kubespray** is a collection of Ansible playbooks that are used to deploy production-grades clusters on both bare-metal and the cloud. Alongside the installation, day two operations can be performed with kubespray. This installer is officially maintained by the Kubernetes community. A dozen of addons are available in kubespray and can be deployed along with Kubernetes easily. kubespray is one of the most suitable installation choices.

More information on [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)

**kops** will not only manage the cluster lifecycle, but it also will provision necessary cloud infrastructure. Deploying on AWS is supported officially, deploying on other cloud providers is available but in alpha and beta state.

More information on [https://kops.sigs.k8s.io](https://kops.sigs.k8s.io/)

**RKE** is a Rancher distributed Kubernetes that can deploy production-grade Kubernetes clusters on top of Docker containers. Managing Kubernetes clusters is done easily with RKE. If you want to use the Rancher platform, you should select this distribution.

More information on [https://github.com/rancher/rke](https://github.com/rancher/rke)

**Charmed Kubernetes** is the Canonical way to deploy Kubernetes clusters with Juju. It’s suitable for running Kubernetes both on Multi-cloud environments and bare-metal. If you look for an eligible Kubernetes distribution that can be deployed on OpenStack, this installer is for you.

More information on [https://ubuntu.com/kubernetes](https://ubuntu.com/kubernetes)

**KubeSphere** is not only a Kubernetes distribution, it also is a platform for creating a cloud solution based on Kubernetes. A large number of tools, addons, etc can be deployed using KubeSphere along with Kubernetes. This platform can also be deployed on the existing Kubernetes cluster.

More information on [https://kubesphere.io](https://kubesphere.io/)

**Kubermatic** is a Kubernetes platform just like the Rancher. You can deploy and manage Kubernetes clusters on clouds as well as on-premises. The connection between the master/seed cluster and the downstream clusters is handled by OpenVPN.

More information on [https://github.com/kubermatic/kubermatic](https://github.com/kubermatic/kubermatic)

**KubeOne** is a tool to provision necessary infrastructure and deploying Kubernetes on a couple of providers. It can easily be integrated with Terraform and Kubermatic.

More information on [https://github.com/kubermatic/kubeone](https://github.com/kubermatic/kubeone)

## 4- Managed clusters:

The lifecycle of clusters is managed by the providers. In this type of installation, a production-grade cluster can be deployed with minimum user actions. Providers are responsible for managing the whole cluster as well as the underlying infrastructure. Because of ease of installation and management, this method is suggested to anyone. Another benefit of using managed Kubernetes is access to cloud features. Some managed Kubernetes providers provide a set of useful features which may not be available on on-premises or bare-metal solutions.

**Magnum** is an OpenStack solution to installing managed Kubernetes and other orchestration tools on top of the OpenStack ecosystem. With Magnum, cloud customers can run Kubernetes clusters easily. This method can also be categorized in Automated cluster installation methods. I decided to introduce it here because it also supports the amazing cloud features. In addition, OpenStack based cloud providers can provide this method to their customers to installing managed Kubernetes clusters.

More information on [https://docs.openstack.org/magnum/latest](https://docs.openstack.org/magnum/latest/)

**EKS** stands for Elastic Kubernetes Service is the amazon solution to provide managed Kubernetes cluster. EKS can easily be integrated with other Amazon services. A command-line tool, eksctl, is used to running a production Kubernetes cluster in a couple of minutes.

More information on [https://aws.amazon.com/eks](https://aws.amazon.com/eks)

**GKE** is a Google Cloud version of Kubernetes just like AWS EKS. GKE offers a special operation mode called Autopilot which reduces managing costs and optimizes clusters for production.

More information on [https://cloud.google.com/kubernetes-engine](https://cloud.google.com/kubernetes-engine)

**AKS** is managed by Microsoft Azure and can be deployed easily. This managed Kubernetes solution is good for Azure users because it can integrate with other Azure tools that are available on the Azure ecosystem.

More information on [https://azure.microsoft.com/en-us/services/kubernetes-service](https://azure.microsoft.com/en-us/services/kubernetes-service/)

## 5- Kubernetes the hard way:

The hard way method is used to install Kubernetes from scratch. I suggest this type of installation to everyone who wants to learn all components of Kubernetes to understand Kubernetes deeply. If you want to know the process of deploying a cluster and what happens between cluster components, install Kubernetes with this method at least one time.

**Kelsey Hightower** is the one who released the hard way method for the first time, as far as I know. He explains how to deploy Kubernetes from scratch on the Google Cloud Platform.

More information on [https://github.com/kelseyhightower/kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

**Mumshad Mannambeth** forks the previously explained method and has changed that to deploy to local machines using Vagrant. This version of the hard way is not updated to the latest Kubernetes.

More information on [https://github.com/mmumshad/kubernetes-the-hard-way](https://github.com/mmumshad/kubernetes-the-hard-way)

## Conclusion:

A multitude of other tools and solutions for installing Kubernetes exists on the market however I described more popular ones. Which method you should use is up to you. There is no absolute right solution but if you want my opinion as best practices, here are some options for you in different situations. You wear the pants at the end.

Absolute single-node for testing and development: **minikube**

Running a multi-node Kubernetes on a single-node machine: **kind** or **k3d**

Minimal production cluster based on Ubuntu ecosystem: **microk8s**

A single-node with pre-installed addons: **k3s**

IoT friendly with clustering support: **k3s** or **microk8s**

For using as executor on CI/CD systems: **k3s** or **microk8s**

Manual minimum viable cluster: **kubeadm**

Automated cluster installation based on Ansible: **kubespray**

Self-managed cluster on AWS: **kops**

Self-managed for various infrastructure: **KubeOne**

Rancher friendly cluster on top of Docker: **RKE**

For Ubuntu lovers that using Juju: **Charmed Kubernetes**

Managing Multi-cluster environments: **Rancher** or **Kubermatic**

Cloud-native application management: **KubeSphere**

Clouds that based on OpenStack: **Magnum**

Managed cluster for everyone: **GKE**, **EKS** or **AKS**

Managed cluster with Autopilot support: **GKE**

For who one needs to understand Kubernetes deeply: **The Hard Way**

Day to day practices(all-in-one): **minikube** or **k3s**

If you have some experience or know other good methods, please comment below. I will update this post if any changes happened on the market.