
# Quickstart to setup 3 nodes RKE cluster

## Downand and Setup RKE on Local environment

```
VERSION=v1.1.4 && \
curl -LO https://github.com/rancher/rke/releases/download/$VERSION/rke_linux-amd64 && \
chmod +x ./rke_linux-amd64 && \
sudo mv ./rke_linux-amd64 /usr/local/bin/rke
```
## Provision 3 VMS with the docker configuration

1. Install packages to allow apt to use repo over https

    ```
    sudo apt-get update && sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    ```

4. Add Docker official GPG Key

    ```
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

5. Setup Repository

    ```
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```
6. Update

    ```
    sudo apt-get update
    ```

7. Install Docker

    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
8. Create Docker Group

    ```
    sudo groupadd docker
    ```
9. Add User to the docker Group

    ```
    sudo usermod -aG docker $USER
    ```
10. Activate Changes

    ```
    newgrp docker
    ```

## Configure cluster
Use the example below to create a `cluster.yml`, and populate it with the IP and host name of your VM's.

cluster.yaml
```yaml
nodes:
  # Controlplane & Etcd nodes
  - address: NODE_IP
    user: HOST_USER
    role:
      - controlplane
      - etcd
      - worker
    hostname_override: master
  # Worker nodes
  - address: NODE_IP
    user: HOST_USER
    role:
      - controlplane
      - etcd
      - worker
    hostname_override: worker1
  - address: NODE_IP
    user: HOST_USER
    role:
      - controlplane
      - etcd
      - worker
    hostname_override: worker2
```

Note: Your local environment must have `ssh` access to all the nodes. Click [here](https://www.ssh.com/ssh/copy-id) to see more on how to setup public key authentication.

## Run RKE

```
rke up --config ./cluster.yaml
```

After provisioning the cluster, `rke` will create `kube_config_cluster.yml` to interact with the cluster,

## Intearct using kubectl

```
kubectl get nodes --kubeconfig kube_config_cluster.yml
```


## Setup Rancher

A quick way to setup a rancher server on a docker.

1. Create a dir on you local machine `/opt/rancher` to persist the rancher data.   

2. Run the docker command
    ```
    docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v /opt/rancher:/var/lib/rancher \
    rancher/rancher:latest
    ```
3. Goto: https://localhost and login.
4. GOTO Add Cluster > Import an existing cluster > Name the cluster and click create.
5. Follow the instructions on the rancher dashboard to import the cluster.