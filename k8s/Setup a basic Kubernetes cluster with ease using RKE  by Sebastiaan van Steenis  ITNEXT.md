In this post, you will go from 3 Ubuntu 16.04 nodes to a basic Kubernetes cluster in a few simple steps. To accomplish this, you will be using Rancher Kubernetes Engine (RKE). To be able to use RKE, you will need 3 Linux nodes with Docker installed (see **Requirements** below).

This won’t be a production ready cluster, but enough to get you familiar with RKE, some Kubernetes and be able to play around with the cluster. Keep an eye out for the post for building a production ready cluster.

This post was also published on the Rancher Blog: [https://rancher.com/blog/2018/2018-09-26-setup-basic-kubernetes-cluster-with-ease-using-rke/](https://rancher.com/blog/2018/2018-09-26-setup-basic-kubernetes-cluster-with-ease-using-rke/)

## Requirements

-   **RKE  
    **You will be using RKE from your workstation. Download the latest version for your platform at:  
    [https://github.com/rancher/rke/releases/latest](https://github.com/rancher/rke/releases/latest)
-   **kubectl  
    **After creating the cluster, we will use the default Kubernetes command-line tool called `kubectl` to interact with the cluster.  
    Get the latest version for your platform at:  
    [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
-   **3 Ubuntu 16.04 nodes with 2(v)CPUs, 4GB of memory and with swap disabled  
    **Most commonly used Linux distribution is Ubuntu 16.04, this is what will be used in this post. Make sure swap is disabled by running `swapoff -a` and removing any `swap` entry in `/etc/fstab`. You must be able to access the node using SSH. As this is a multi-node cluster, [the required ports](https://rancher.com/docs/rke/v0.1.x/en/os/#ports) need to be opened before proceeding.
-   **Docker installed on each Linux node  
    **Kubernetes only validates Docker up to **17.03.2** (See [https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md#external-dependencies](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md#external-dependencies)).  
    You can use [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/) to install Docker (make sure you install **17.03.2**) or use this one-liner to install the correct version:  
    `curl [https://releases.rancher.com/install-docker/17.03.sh](https://releases.rancher.com/install-docker/17.03.sh) | sh`

Make sure the requirements listed above are fulfilled before you proceed.

## How RKE works

RKE can be run from any platform (the binary is available for MacOS/Linux/Windows), in this example it will run on your workstation/laptop/computer. The examples in this post are based on MacOS/Linux.

RKE will connect to the nodes using a configured SSH private key (the nodes should have the matching SSH public key installed for the SSH user) and setup a tunnel to access the Docker socket (`/var/run/docker.sock` by default, but configurable). This means that the configured SSH user must have access to the Docker socket on the machine, we will go over this in **Creating the Linux user account**.

![](https://miro.medium.com/max/505/1*Fk-h09tAgw-LRATG59J1tQ.png)

## Creating the Linux user account

_Note: Make sure Docker is installed following the instructions in the_ **_Requirements_** _section above._

The following steps need to be executed on every node. If you need to use sudo, prefix each command with `sudo`. If you already have users that can access the machine using a SSH key and can access the Docker socket, you can skip this step.

```
# Login to the node$ ssh your_user@hostname# Create a Linux user called rke, create home directory, and add to docker group$ useradd -m -G docker rke# Switch user to rke and create SSH directories$ su - rke$ mkdir $HOME/.ssh$ chmod 700 $HOME/.ssh$ touch $HOME/.ssh/authorized_keys# Test Docker socket access$ docker versionClient: Version:      17.03.2-ce API version:  1.27 Go version:   go1.7.5 Git commit:   f5ec1e2 Built:        Tue Jun 27 03:35:14 2017 OS/Arch:      linux/amd64Server: Version:      17.03.2-ce API version:  1.27 (minimum version 1.12) Go version:   go1.7.5 Git commit:   f5ec1e2 Built:        Tue Jun 27 03:35:14 2017 OS/Arch:      linux/amd64 Experimental: false
```

## Configuring SSH keys

In this post we will create new keys but feel free to use your existing keys. Just make sure you specify them correctly when we configure the keys in RKE.

_Note: If you want to use SSH keys with a passphrase, you will need to have_ `_ssh-agent_` _running with the key added and specify_ `_--ssh-agent-auth_` _when running RKE._

**Creating SSH key pair  
**Creating an SSH key pair can be done by using `ssh-keygen` , you can execute this on your workstation/laptop/computer. It is highly recommended to put a passphrase on your SSH private key. If you lose your SSH private key (and not have a passphrase on it), anyone can use it to access your nodes.

```
ssh-keygenGenerating public/private rsa key pair.Enter file in which to save the key ($HOME/.ssh/id_rsa):Enter passphrase (empty for no passphrase):Enter same passphrase again:Your identification has been saved in $HOME/.ssh/id_rsa.Your public key has been saved in $HOME/.ssh/id_rsa.pub.The key fingerprint is:xxx
```

After creating the SSH key pair, you should have the following files:

-   `$HOME/.ssh/id_rsa` (SSH private key, keep this secure)
-   `$HOME/.ssh/id_rsa.pub` (SSH public key)

**Copy the SSH public key to the nodes:  
**To be able to access the nodes using the created SSH key pair, you will need to install the SSH public key onto the nodes.

Execute this for every node (where `hostname` is the IP/hostname of the node):

```
# Install the SSH public key on the node$ cat $HOME/.ssh/id_rsa.pub | ssh hostname "sudo tee -a /home/rke/.ssh/authorized_keys"
```

_Note: This post is demonstrating how you create a separate user for RKE. Because of this, we can’t use_ `_ssh-copy-id_` _as it only works for installing keys to the same user as is used for the SSH connection._

**Setup ssh-agent  
**_Note: If you chose not to put a passphrase on your SSH private key, you can skip this step._

This needs to be executed on your workstation/laptop/computer:

```
# Run ssh-agent and configure the correct environment variables$ eval $(ssh-agent)Agent pid 5151# Add the private key to the ssh-agent$ ssh-add $HOME/.ssh/id_rsaIdentity added: $HOME/.ssh/id_rsa ($HOME/.ssh/id_rsa)
```

**Test SSH connectivity  
**Last step is to test if we can access the node using the SSH private key. This needs to be executed on your workstation/laptop/computer, replacing `hostname` with each of the nodes IP/hostname):

```
$ ssh -i $HOME/.ssh/id_rsa rke@hostname docker versionClient: Version:      17.03.2-ce API version:  1.27 Go version:   go1.7.5 Git commit:   f5ec1e2 Built:        Tue Jun 27 03:35:14 2017 OS/Arch:      linux/amd64Server: Version:      17.03.2-ce API version:  1.27 (minimum version 1.12) Go version:   go1.7.5 Git commit:   f5ec1e2 Built:        Tue Jun 27 03:35:14 2017 OS/Arch:      linux/amd64 Experimental: false
```

## Configuring and running RKE

Get RKE for your platform at:  
[https://github.com/rancher/rke/releases/latest](https://github.com/rancher/rke/releases/latest).

RKE will run on your workstation/laptop/computer.

For this post I’ve renamed the RKE binary to `rke`, to make the commands generic for each platform. You can do the same by running:

```
mv rke_darwin-amd64 rke
```

Test if RKE can be successfully executed by using the following command:

```
# Download RKE for MacOS (Darwin)$ wget https://github.com/rancher/rke/releases/download/v0.1.9/rke_darwin-amd64# Rename binary to rkemv rke_darwin-amd64 rke# Make RKE binary executable$ chmod +x rke# Show RKE version$ ./rke --versionrke version v0.1.9
```

Next step is to create a cluster configuration file (by default it will be`cluster.yml`). This contains all information to build the Kubernetes cluster, like node connection info, what roles to apply to what node etcetera. All [configuration options](https://rancher.com/docs/rke/v0.1.x/en/config-options/) can be found in the documentation. You can create the cluster configuration file by running `./rke config` and answering the questions. For this post, you will create a 3 node cluster with every role on each node (answer `y` for every role), and we will add the Kubernetes Dashboard as addon (Using [https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml)). To access the Kubernetes Dashboard, you need a Service Account token which will be created by adding [https://gist.githubusercontent.com/superseb/499f2caa2637c404af41cfb7e5f4a938/raw/930841ac00653fdff8beca61dab9a20bb8983782/k8s-dashboard-user.yml](https://gist.githubusercontent.com/superseb/499f2caa2637c404af41cfb7e5f4a938/raw/930841ac00653fdff8beca61dab9a20bb8983782/k8s-dashboard-user.yml) to the addons.

Regarding answering the question to create the cluster configuration file:

-   The values in brackets, for instance `[22]` for SSH Port, are defaults and can just be used by pressing the Enter key.
-   The default SSH Private Key would do, if you have another key, please change it.

```
$ ./rke config[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]: ~/.ssh/id_rsa[+] Number of Hosts [1]: 3[+] SSH Address of host (1) [none]: ip_or_dns_host1[+] SSH Port of host (1) [22]:[+] SSH Private Key Path of host (ip_or_dns_host1) [none]:[-] You have entered empty SSH key path, trying fetch from SSH key parameter[+] SSH Private Key of host (ip_or_dns_host1) [none]:[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa[+] SSH User of host (ip_or_dns_host1) [ubuntu]: rke[+] Is host (ip_or_dns_host1) a Control Plane host (y/n)? [y]: y[+] Is host (ip_or_dns_host1) a Worker host (y/n)? [n]: y[+] Is host (ip_or_dns_host1) an etcd host (y/n)? [n]: y[+] Override Hostname of host (ip_or_dns_host1) [none]:[+] Internal IP of host (ip_or_dns_host1) [none]:[+] Docker socket path on host (ip_or_dns_host1) [/var/run/docker.sock]:[+] SSH Address of host (2) [none]: ip_or_dns_host2[+] SSH Port of host (2) [22]:[+] SSH Private Key Path of host (ip_or_dns_host2) [none]:[-] You have entered empty SSH key path, trying fetch from SSH key parameter[+] SSH Private Key of host (ip_or_dns_host2) [none]:[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa[+] SSH User of host (ip_or_dns_host2) [ubuntu]: rke[+] Is host (ip_or_dns_host2) a Control Plane host (y/n)? [y]: y[+] Is host (ip_or_dns_host2) a Worker host (y/n)? [n]: y[+] Is host (ip_or_dns_host2) an etcd host (y/n)? [n]: y[+] Override Hostname of host (ip_or_dns_host2) [none]:[+] Internal IP of host (ip_or_dns_host2) [none]:[+] Docker socket path on host (ip_or_dns_host2) [/var/run/docker.sock]:[+] SSH Address of host (3) [none]: ip_or_dns_host3[+] SSH Port of host (3) [22]:[+] SSH Private Key Path of host (ip_or_dns_host3) [none]:[-] You have entered empty SSH key path, trying fetch from SSH key parameter[+] SSH Private Key of host (ip_or_dns_host3) [none]:[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa[+] SSH User of host (ip_or_dns_host3) [ubuntu]: rke[+] Is host (ip_or_dns_host3) a Control Plane host (y/n)? [y]: y[+] Is host (ip_or_dns_host3) a Worker host (y/n)? [n]: y[+] Is host (ip_or_dns_host3) an etcd host (y/n)? [n]: y[+] Override Hostname of host (ip_or_dns_host3) [none]:[+] Internal IP of host (ip_or_dns_host3) [none]:[+] Docker socket path on host (ip_or_dns_host3) [/var/run/docker.sock]:[+] Network Plugin Type (flannel, calico, weave, canal) [canal]:[+] Authentication Strategy [x509]:[+] Authorization Mode (rbac, none) [rbac]:[+] Kubernetes Docker image [rancher/hyperkube:v1.11.1-rancher1]:[+] Cluster domain [cluster.local]:[+] Service Cluster IP Range [10.43.0.0/16]:[+] Enable PodSecurityPolicy [n]:[+] Cluster Network CIDR [10.42.0.0/16]:[+] Cluster DNS Service IP [10.43.0.10]:[+] Add addon manifest URLs or YAML files [no]: yes[+] Enter the Path or URL for the manifest [none]: https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml                                                                                                                    [+] Add another addon [no]: yes[+] Enter the Path or URL for the manifest [none]: https://gist.githubusercontent.com/superseb/499f2caa2637c404af41cfb7e5f4a938/raw/930841ac00653fdff8beca61dab9a20bb8983782/k8s-dashboard-user.yml                                                                                                                    [+] Add another addon [no]: no
```

When the last question is answered, the `cluster.yml` file will be created in the same directory as RKE was run:

```
ls -la cluster.yml -rw-r----- 1 user user 3688 Sep 17 12:50 cluster.yml
```

You are now ready to build your Kubernetes cluster. This can be done by running `rke up`. Before you run the command, make sure the [ports required](https://rancher.com/docs/rke/v0.1.x/en/os/#ports) are opened between your workstation/laptop/computer and the nodes, and between each of the nodes. You can now build your cluster using the following command:

```
$ ./rke upINFO[0000] Building Kubernetes cluster...INFO[0151] Finished building Kubernetes cluster successfully
```

If all went well, you should have a lot of output from the command but it should end with `Finished building Kubernetes cluster successfully`. It will also write a kubeconfig file as `kube_config_cluster.yml` . You can use that file to connect to your Kubernetes cluster.

## Exploring your Kubernetes cluster

Make sure you have `kubectl` installed, see [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/) how to get it for your platform.

_Note: When running_ `_kubectl_`_, it automatically tries to use a kubeconfig from the default location;_ `_$HOME/.kube/config_` _. In the examples, we explicitly specify the kubeconfig file using_ `_--kubeconfig kube_config_cluster.yml_` _. If you don’t want to specify the kubeconfig file every time, you can copy the file_ `_kube_config_cluster.yml_` _to_ `_$HOME/.kube/config_`_. (you probably need to create the directory_ `_$HOME/.kube_` _first)_

Start with querying the server for its version:

```
$ kubectl --kubeconfig kube_config_cluster.yml versionClient Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-10T11:44:36Z", GoVersion:"go1.11", Compiler:"gc", Platform:"darwin/amd64"}                                                                   Server Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.1", GitCommit:"b1b29978270dc22fecc592ac55d903350454310a", GitTreeState:"clean", BuildDate:"2018-07-17T18:43:26Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
```

One of the first things to check, is if all nodes are in `Ready` state:

```
$ kubectl --kubeconfig kube_config_cluster.yml get nodesNAME    STATUS    ROLES                      AGE       VERSIONhost1   Ready     controlplane,etcd,worker   11m       v1.11.1host2   Ready     controlplane,etcd,worker   11m       v1.11.1host3   Ready     controlplane,etcd,worker   11m       v1.11.1
```

When you generated the cluster configuration file, you added the Kubernetes dashboard addon to be deployed on the cluster. You can check the status of the deployment using:

```
$ kubectl --kubeconfig kube_config_cluster.yml get deploy -n kube-system -l k8s-app=kubernetes-dashboardNAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGEkubernetes-dashboard   1         1         1            1           17m
```

By default, the deployments are not exposed to the outside. If you want to visit the Kubernetes Dashboard in your browser, you will need to expose the deployment externally (which we will do in our demo application later) or use the built-in proxy functionality of kubectl. This will open the `127.0.0.1:8001` (your local machine on port 8001) and tunnel it to the Kubernetes cluster.

Before you can visit the Kubernetes Dashboard, you need to retrieve the token to login to the dashboard. By default, it runs under a very limited account and will not be able to show you all the resources in your cluster. The second addon we added when creating the cluster configuration file created the account and token we need (this is based upon [https://github.com/kubernetes/dashboard/wiki/Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user))

You can retrieve the token by running:

```
$ kubectl --kubeconfig kube_config_cluster.yml -n kube-system describe secret $(kubectl --kubeconfig kube_config_cluster.yml -n kube-system get secret | grep admin-user | awk '{print $1}') | grep ^token: | awk '{ print $2 }'eyJhbGciOiJSUzI1NiIs....<more_characters>
```

The string that is returned, is the token you need to login to the dashboard. Copy the whole string.

Set up the kubectl proxy as follows:

```
$ kubectl --kubeconfig kube_config_cluster.yml proxyStarting to serve on 127.0.0.1:8001
```

And open the following URL:

[http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

When prompted for login, choose **Token**, paste the token and click **Sign In**.

_Note: When you don’t get a login screen, open it manually by clicking Sign In on the top right._

## Run a demo application

Last step of this post, running a demo application and exposing it. For this example you will run a demo application `superseb/rancher-demo`, which is a web UI showing the scale of a deployment. It will be exposed using an Ingress, which is handled by the NGINX Ingress controller that is deployed by default. If you want to know more about Ingress, please see [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Start by deploying and exposing the demo application (which runs on port 8080):

```
$ kubectl --kubeconfig kube_config_cluster.yml run --image=superseb/rancher-demo rancher-demo --port 8080 --exposeservice/rancher-demo createddeployment.apps/rancher-demo created
```

Check the status of your deployment:

```
$ kubectl --kubeconfig kube_config_cluster.yml rollout status deployment/rancher-demo...deployment "rancher-demo" successfully rolled out
```

The command `kubectl run` is the easiest way to get a container running on your cluster. It takes an image parameter to specify the Docker image and a name at minimum. In this case, we also want to configure the port that this container exposes (internally), and expose it. What happened was that there was a Deployment created (and a ReplicaSet) with a scale of 1 (default), and a Service was created to abstract access to the pods (which can contain one or more containers, in this case 1). For more information on these subjects check the following links:

-   [https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/)
-   [https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)

RKE deploys the NGINX Ingress controller by default on every node. This opens op port 80 and port 443, and can serve as main entrypoint for any created Ingress. An Ingress can contain a single host or multiple, multiple paths, and you can configure SSL certificates. In this post you will configure a basic Ingress, making our demo application accessible on a certain hostname. In the example we will use `rancher-demo.domain.test` as hostname to access the demo application.

_Note: To access our test domain you have to add the domain name to /etc/hosts to visit the UI, as it’s not a valid DNS name. If you have access to your own domain, you can add a DNS A record pointing to each of the nodes._

![](https://miro.medium.com/max/401/1*1QWVZauO1CPPF9HxNrQx7w.png)

The only part that is not created, is the Ingress. Let’s create an Ingress called`rancher-demo-ingress`, having a host specification to match requests to our test domain (`rancher-demo.domain.test`), and pointing it to our Service called `rancher-demo` on port 8080. Save the following content to a file called `ingress.yml`:

```
apiVersion: extensions/v1beta1kind: Ingressmetadata:  name: rancher-demo-ingressspec:  rules:  - host: rancher-demo.domain.test    http:      paths:      - path: /        backend:          serviceName: rancher-demo          servicePort: 8080
```

Create this Ingress using kubectl:

```
$ kubectl --kubeconfig kube_config_cluster.yml apply -f ingress.ymlingress.extensions/rancher-demo-ingress created
```

Time to test accessing the demo application. You can try it on the command-line first, instruct `curl` to resolve the test domain to each of the nodes:

```
# Get node IP addresses$ kubectl --kubeconfig kube_config_cluster.yml get nodes                                                                                                                                                                                                                           NAME       STATUS    ROLES                      AGE       VERSION10.0.0.1   Ready     controlplane,etcd,worker   3h        v1.11.110.0.0.2   Ready     controlplane,etcd,worker   3h        v1.11.110.0.0.3   Ready     controlplane,etcd,worker   3h        v1.11.1# Test accessing the demo application$ curl --resolve rancher-demo.domain.test:80:10.0.0.1 http://rancher-demo.domain.test/ping                                                                                                                                                                                {"instance":"rancher-demo-5cbfb4b4-thmbh","version":"0.1"}$ curl --resolve rancher-demo.domain.test:80:10.0.0.2 http://rancher-demo.domain.test/ping                                                                                                                                                                                  {"instance":"rancher-demo-5cbfb4b4-thmbh","version":"0.1"}$ curl --resolve rancher-demo.domain.test:80:10.0.0.3http://rancher-demo.domain.test/ping                                                                                                                                                                          {"instance":"rancher-demo-5cbfb4b4-thmbh","version":"0.1"}
```

If you use the test domain, you will need to add it to your machine’s `/etc/hosts` file to be able to reach it properly.

```
echo "10.0.0.1  rancher-demo.domain.test" | sudo tee -a /etc/hosts
```

Now visit [http://rancher-demo.domain.test](http://rancher-demo.domain.test/) in your browser.

If this has all worked out, you can fill up the demo application a bit more by scaling up your Deployment:

```
$ kubectl --kubeconfig kube_config_cluster.yml scale deploy/rancher-demo --replicas=10deployment.extensions/rancher-demo scaled
```

_Note: Make sure to clean up the_ `_/etc/hosts_` _entry when you are done._

## Closing words

This started as a post how to create a Kubernetes cluster in under 10 minutes, but along the way I tried to add some useful information how certain parts work. To avoid having a post that takes a day to read (explaining every part), there will be other posts describing certain parts. For now, I’ve linked as much resources as possible to existing documentation where you can learn more.

-   [RKE documentation](https://rancher.com/docs/rke/v0.1.x/en/)
-   [NGINX Ingress controller](https://kubernetes.github.io/ingress-nginx/)
-   [Learn Kubernetes basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)