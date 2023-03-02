![](https://miro.medium.com/max/700/1*8lV6ti2Ub4Zmn71k4fb-8w.png)

R**ancher** has developed multiple tools which can make your life easier in handling K8s infrastructure. There is a nice UI where you can Monitor/Add/Remove Nodes. You can also create new deployments, ingress controllers, Volumes etc using Rancher UI.

Follow this documentation to set up a highly available Kubernetes cluster on **Ubuntu 20.04 LTS** machines using Rancher’s RKE.

This documentation guides you in setting up a cluster with six nodes all of which play master, etcd and worker role.

## Environment:

```
Minimum HW for VMs|Role|IP|OS|RAM|CPU||Master, etcd|172.17.1.100|Ubuntu 20.04|2G|2||Master, etcd|172.17.1.101|Ubuntu 20.04|2G|2||Master, etcd|172.17.1.102|Ubuntu 20.04|2G|2||worker|172.17.1.103|Ubuntu 20.04|2G|2||worker|172.17.1.104|Ubuntu 20.04|2G|2||worker|172.17.1.105|Ubuntu 20.04|2G|2|
```

**Pre-requisites:** If you want to try this in a virtualized environment on your workstation

-   Host machine has at least 8 cores
-   Host machine has at least 16G memory

## Download RKE Binary

Download the latest release from the GitHub releases [page](https://github.com/rancher/rke/releases)

```
$ wget https://github.com/rancher/rke/releases/download/v1.2.9/rke_linux-amd64$ chmod +x rke_linux-amd64$ cp rke_linux-amd64 /usr/local/bin/rke $ which rke$ rke --help
```

## Set up passwordless SSH Logins on all nodes

We will be using SSH Keys to login to root account on all the kubernetes nodes. I am not going to set a passphrase for this ssh keypair.

## Create an ssh keypair on the host machine

```
$ ssh-keygen -t rsa -b 2048
```

## Copy SSH Keys to all the kubernetes nodes

```
$ ssh-copy-id root@172.17.1.100$ ssh-copy-id root@172.17.1.101$ ssh-copy-id root@172.17.1.102$ ssh-copy-id root@172.17.1.103$ ssh-copy-id root@172.17.1.104$ ssh-copy-id root@172.17.1.105
```

## Prepare the kubernetes nodes

-   **Disable Firewall**

```
$ ufw disable
```

-   **Disable swap**

```
$ swapoff -a; sed -i '/swap/d' /etc/fstab
```

-   **Update sysctl settings for Kubernetes networking**

```
$ cat >>/etc/sysctl.d/kubernetes.conf<<EOFnet.bridge.bridge-nf-call-ip6tables = 1net.bridge.bridge-nf-call-iptables = 1EOFsysctl --system
```

-   **Install docker engine**

```
$ apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add-add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"$ apt update && apt install -y docker-ce containerd.io$ systemctl start docker && systemctl enable docker$ usermod -aG docker ${USER}
```

## Configure RKE Kubernetes cluster

**Create cluster configuration**

```
$ rke config
```

> _Once gone through this interactive cluster configuration, you will end up with cluster.yml file in the current directory._

**cluster.yml sample below**

```
# If you intened to deploy Kubernetes in an air-gapped environment,# please consult the documentation on how to configure custom RKE images.nodes:- address: 172.17.1.100  port: "22"  internal_address: 172.17.1.100  role:  - controlplane  - worker  - etcd  hostname_override: ""  user: rke  docker_socket: /var/run/docker.sock  ssh_key: ""  ssh_key_path: ~/.ssh/id_rsa  ssh_cert: ""  ssh_cert_path: ""  labels:    app: ingress  taints: []- address: 172.17.1.101  port: "22"  internal_address: 172.17.1.101  role:  - controlplane  - worker  - etcd  hostname_override: ""  user: rke  docker_socket: /var/run/docker.sock  ssh_key: ""  ssh_key_path: ~/.ssh/id_rsa  ssh_cert: ""  ssh_cert_path: ""  labels:    app: ingress  taints: []- address: 172.17.1.102  port: "22"  internal_address: 172.17.1.102  role:  - controlplane  - worker  - etcd  hostname_override: ""  user: rke  docker_socket: /var/run/docker.sock  ssh_key: ""  ssh_key_path: ~/.ssh/id_rsa  ssh_cert: ""  ssh_cert_path: ""  labels:    app: ingress  taints: []  - address: 172.17.1.103  port: "22"  internal_address: 172.17.1.103  role:  - worker  hostname_override: ""  user: rke  docker_socket: /var/run/docker.sock  ssh_key: ""  ssh_key_path: ~/.ssh/id_rsa  ssh_cert: ""  ssh_cert_path: ""  labels: {}  taints: []- address: 172.17.1.104  port: "22"  internal_address: 172.17.1.104  role:  - worker  hostname_override: ""  user: rke  docker_socket: /var/run/docker.sock  ssh_key: ""  ssh_key_path: ~/.ssh/id_rsa  ssh_cert: ""  ssh_cert_path: ""  labels: {}  taints: []- address: 172.17.1.105  port: "22"  internal_address: 172.17.1.105  role:  - worker  hostname_override: ""  user: rke  docker_socket: /var/run/docker.sock  ssh_key: ""  ssh_key_path: ~/.ssh/id_rsa  ssh_cert: ""  ssh_cert_path: ""  labels: {}  taints: []services:  etcd:    image: ""    extra_args: {}    extra_binds: []    extra_env: []    win_extra_args: {}    win_extra_binds: []    win_extra_env: []    external_urls: []    ca_cert: ""    cert: ""    key: ""    path: ""    uid: 0    gid: 0    snapshot: null    retention: ""    creation: ""    backup_config: null  kube-api:    image: ""    extra_args: {}    extra_binds: []    extra_env: []    win_extra_args: {}    win_extra_binds: []    win_extra_env: []    service_cluster_ip_range: 10.43.0.0/16    service_node_port_range: ""    pod_security_policy: false    always_pull_images: false    secrets_encryption_config: null    audit_log: null    admission_configuration: null    event_rate_limit: null  kube-controller:    image: ""    extra_args: {}    extra_binds: []    extra_env: []    win_extra_args: {}    win_extra_binds: []    win_extra_env: []    cluster_cidr: 10.42.0.0/16    service_cluster_ip_range: 10.43.0.0/16  scheduler:    image: ""    extra_args: {}    extra_binds: []    extra_env: []    win_extra_args: {}    win_extra_binds: []    win_extra_env: []  kubelet:    image: ""    extra_args: {}    extra_binds:    - /var/openebs/local:/var/openebs/local    extra_env: []    win_extra_args: {}    win_extra_binds: []    win_extra_env: []    cluster_domain: cluster.local    infra_container_image: ""    cluster_dns_server: 10.43.0.10    fail_swap_on: false    generate_serving_certificate: false  kubeproxy:    image: ""    extra_args: {}    extra_binds: []    extra_env: []    win_extra_args: {}    win_extra_binds: []    win_extra_env: []network:  plugin: calico  options: {}  mtu: 0  node_selector: {}  update_strategy: null  tolerations: []authentication:  strategy: x509  sans: []  webhook: nullsystem_images:  etcd: rancher/mirrored-coreos-etcd:v3.4.15-rancher1  alpine: rancher/rke-tools:v0.1.74  nginx_proxy: rancher/rke-tools:v0.1.74  cert_downloader: rancher/rke-tools:v0.1.74  kubernetes_services_sidecar: rancher/rke-tools:v0.1.74  kubedns: rancher/mirrored-k8s-dns-kube-dns:1.15.10  dnsmasq: rancher/mirrored-k8s-dns-dnsmasq-nanny:1.15.10  kubedns_sidecar: rancher/mirrored-k8s-dns-sidecar:1.15.10  kubedns_autoscaler: rancher/mirrored-cluster-proportional-autoscaler:1.8.1  coredns: rancher/mirrored-coredns-coredns:1.8.0  coredns_autoscaler: rancher/mirrored-cluster-proportional-autoscaler:1.8.1  nodelocal: rancher/mirrored-k8s-dns-node-cache:1.15.13  kubernetes: rancher/hyperkube:v1.20.6-rancher1  flannel: rancher/coreos-flannel:v0.13.0-rancher1  flannel_cni: rancher/flannel-cni:v0.3.0-rancher6  calico_node: rancher/mirrored-calico-node:v3.17.2  calico_cni: rancher/mirrored-calico-cni:v3.17.2  calico_controllers: rancher/mirrored-calico-kube-controllers:v3.17.2  calico_ctl: rancher/mirrored-calico-ctl:v3.17.2  calico_flexvol: rancher/mirrored-calico-pod2daemon-flexvol:v3.17.2  canal_node: rancher/mirrored-calico-node:v3.17.2  canal_cni: rancher/mirrored-calico-cni:v3.17.2  canal_controllers: rancher/mirrored-calico-kube-controllers:v3.17.2  canal_flannel: rancher/coreos-flannel:v0.13.0-rancher1  canal_flexvol: rancher/mirrored-calico-pod2daemon-flexvol:v3.17.2  weave_node: weaveworks/weave-kube:2.8.1  weave_cni: weaveworks/weave-npc:2.8.1  pod_infra_container: rancher/mirrored-pause:3.2  ingress: rancher/nginx-ingress-controller:nginx-0.43.0-rancher3  ingress_backend: rancher/mirrored-nginx-ingress-controller-defaultbackend:1.5-rancher1  metrics_server: rancher/mirrored-metrics-server:v0.4.1  windows_pod_infra_container: rancher/kubelet-pause:v0.1.6  aci_cni_deploy_container: noiro/cnideploy:5.1.1.0.1ae238a  aci_host_container: noiro/aci-containers-host:5.1.1.0.1ae238a  aci_opflex_container: noiro/opflex:5.1.1.0.1ae238a  aci_mcast_container: noiro/opflex:5.1.1.0.1ae238a  aci_ovs_container: noiro/openvswitch:5.1.1.0.1ae238a  aci_controller_container: noiro/aci-containers-controller:5.1.1.0.1ae238a  aci_gbp_server_container: noiro/gbp-server:5.1.1.0.1ae238a  aci_opflex_server_container: noiro/opflex-server:5.1.1.0.1ae238assh_key_path: ~/.ssh/id_rsassh_cert_path: ""ssh_agent_auth: falseauthorization:  mode: rbac  options: {}ignore_docker_version: nullkubernetes_version: ""private_registries: []#ingress:#  provider: nginx#  update_strategy: #    strategy: RollingUpdate#    rollingUpdate:#      maxUnavailable: 5#  node_selector:#    app: ingress#  ingress_priority_class_name: system-cluster-critical#  default_backend: falsecluster_name: "k8s-dev"cloud_provider:  name: ""prefix_path: ""win_prefix_path: ""addon_job_timeout: 0bastion_host:  address: ""  port: ""  user: ""  ssh_key: ""  ssh_key_path: ""  ssh_cert: ""  ssh_cert_path: ""monitoring:  provider: "metrics-server"  options: {}  node_selector: {}  update_strategy:    strategy: RollingUpdate    rollingUpdate:      maxUnavailable: 8  replicas: 1  tolerations: []  metrics_server_priority_class_name: ""restore:  restore: false  snapshot_name: ""rotate_encryption_key: falsedns:  provider: coredns  update_strategy:    strategy: RollingUpdate    rollingUpdate:      maxUnavailable: 20%      maxSurge: 15%  linear_autoscaler_params:    cores_per_replica: 0.34    nodes_per_replica: 4    prevent_single_point_failure: true    min: 2    max: 3
```

**Configure ETCD backup**

K8s is a distributed system. It uses ETCD to store the cluster data like configurations, metadata etc. So it is important to backup the ETCD. Rancher RKE can snapshot the ETCD and upload it to S3. It can also save it in your workstation. RKE can create snapshots automatically in a scheduled period.

To create snapshot manually

```
$ rke etcd snapshot-save --name backup-19Jul2021 --config cluster.yml
```

For automated snapshots add following configuration to cluster.yaml

```
services:  etcd:    backup_config:      enabled: true  # enables recurring etcd snapshots      interval_hours: 6 # time increment between snapshots      retention: 60  # time in days before snapshot purge
```

**Provision the cluster**

```
$ rke up
```

> _Once this command completed provisioning the cluster, you will have cluster state file (cluster.rkestate) and kube config file (kube\_config\_cluster.yml) in the current directory._

## Downloading kube config to your local machine

On your host machine

```
$ mkdir ~/.kube$ cp kube_config_cluster.yml ~/.kube/config
```

## Verifying the cluster

```
$ kubectl cluster-info$ kubectl get nodes
```

## Install Rancher UI by Helm

run the following commands from your workstation.

```
$ helm repo add rancher-latest https://releases.rancher.com/server-charts/latest$ kubectl create namespace cattle-system$ helm install rancher rancher-latest/rancher \  --namespace cattle-system \  --set hostname=rancher.yourdomain.com \  --set ingress.tls.source=letsEncrypt \  --set letsEncrypt.email=name@example.com$ kubectl -n cattle-system rollout status deploy/rancher
```

Map the DNS to one of your Worker node IPs and access the URL in your browser. [https://rancher.yourdomain.com](https://rancher.yourdomain.com/)

Finally, I hope this tutorial being useful to you, for any questions feel free to comment, or contact me directly on Mohamed.ElEmam.Hussin@gmail.com.