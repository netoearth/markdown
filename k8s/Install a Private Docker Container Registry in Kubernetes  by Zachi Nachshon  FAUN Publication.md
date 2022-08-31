## Preparation

In this section we will mount an external storage device into a dedicated Kubernetes node, make it available as a static local storage for the Docker registry pod which is intended to run on that node.

## External Volume

External storage devices can be a USB stick, external Hard Disk Drive (HDD) and other such devices. To connect an external storage device please follow the post [Mount Storage Volumes onto Linux Operating Systems](https://levelup.gitconnected.com/mount-storage-volumes-onto-linux-operating-systems-24283d9a676e).

> **Tip:** It is advised to store Docker registry content on an external storage device rather than on the Raspberry Pi SD card. Reason is that in case of SD card failure, the images you have built, tagged and used would still be available within the external storage device.

## Persistent Storage

Once an external storage is ready and connected we will want to use it as a persistent volume and write data to it from pods running on the Kubernetes node it is connected to. To achieve this behavior we will have to tell Kubernetes that this volume is available by using the `PersistentVolume` and `PersistentVolumeClaim` resources.

**PersistentVolume (PV)**

`_PV_` _is a storage definition provisioned manually by the user or_ provisioned _dynamically using a storage class_

When creating the registry as a Kubernetes resource, we will have to tell the registry that we want it to save its own data on the external storage device rather than on the Raspberry Pi SD card. In order to do so, we will assign a statically provisioned local persistent storage that we have mounted manually.

We will use Kubernetes `PersistentVolume (PV)` which is a resource describing an independent volume detached from any pod/node lifecycle.

1.  Create a `PersistentVolume` using a `local` storage-class, mounted on the `kmaster` node

```
cat <<EOF | kubectl apply -f -apiVersion: v1kind: PersistentVolumemetadata:  name: docker-registry-pvspec:  capacity:    storage: 60Gi  volumeMode: Filesystem  accessModes:  - ReadWriteOnce  persistentVolumeReclaimPolicy: Delete  storageClassName: docker-registry-local-storage  local:    path: /mnt/container-registry  nodeAffinity:    required:      nodeSelectorTerms:      - matchExpressions:        - key: kubernetes.io/hostname          operator: In          values:          - kmasterEOF
```

> **Note:** Make sure to use the same mount point as you have used when mounting the external storage device onto the Raspberry Pi `kmaster` node i.e `/mnt/container-registry`.

2\. Verify the persistent volume was properly created

```
kubectl get pv container-registry-pv
```

**PersistentVolumeClaim (PVC)**

`_PVC_` _is a claim to use a durable_ **_pre-defined_** _abstract_ `_PV_` _storage with size and access-mode consideration without exposing the user of how those volumes are implemented._

1.  Create a `container-registry` namespace

```
kubectl create namespace container-registry
```

2\. Create a `PersistentVolumeClaim` using a `local` storage-class, mounted on the `kmaster` node

```
cat <<EOF | kubectl apply -f -apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: docker-registry-pv-claim  namespace: container-registryspec:  accessModes:    - ReadWriteOnce  volumeMode: Filesystem  resources:    requests:      storage: 60Gi  storageClassName: docker-registry-local-storageEOF
```

3\. Verify the persistent volume claim was properly created and it is bounded to `docker-registry-pv`

```
kubectl get pvc docker-registry-pv-claim --namespace container-registry   # NAME                       STATUS   VOLUME# docker-registry-pv-claim   Bound    docker-registry-pv
```

## Registry

The following steps instruct you how to install a Docker registry using a persistent external storage device (a.k.a non-volatile storage) and making the registry available across the entire cluster nodes to interact with.

## Authentication

1.  Generate a user & password using `htpasswd`

```
bash <<'EOF'   # Change these credentials to your ownexport REGISTRY_USER=adminexport REGISTRY_PASS=registryexport DESTINATION_FOLDER=${HOME}/temp/registry-creds   # Backup credentials to local files (in case you'll forget them later on)mkdir -p ${DESTINATION_FOLDER}echo ${REGISTRY_USER} >> ${DESTINATION_FOLDER}/registry-user.txtecho ${REGISTRY_PASS} >> ${DESTINATION_FOLDER}/registry-pass.txt   docker run --entrypoint htpasswd registry:2.7.0 \    -Bbn ${REGISTRY_USER} ${REGISTRY_PASS} \    > ${DESTINATION_FOLDER}/htpasswd      unset REGISTRY_USER REGISTRY_PASS DESTINATION_FOLDER   EOF
```

> **Note:** Since `htpasswd` was removed from latest docker images we will be using version `2.7.0`, for additional information read [here](https://stackoverflow.com/questions/62531462/docker-local-registry-exec-htpasswd-executable-file-not-found-in-path).

## Prepare

We want Kubernetes to create the `docker-registry` pod on the `kmaster` node. In order to do that, we’ll have to label that node and use `nodeSelector` attribute when installing `docker-registry` Helm chart.

> **Tip:** This preparation step is relevant only if you have used a persistent external storage device. It will ensure that the docker-registry pod is being created at the Kubernetes node that has the local storage connected to, otherwise please continue to the next Install step.

1.  Get all nodes names and labels

```
kubectl get nodes --show-labels
```

2\. Label `kmaster` node with `node-type=master`

```
kubectl label nodes kmaster node-type=master
```

> **Note:** I’ll refer to the master node named `kmaster` as specified in the [RPi cluster installation post](https://levelup.gitconnected.com/setting-up-a-raspberry-pi-cluster-b0fda1ee44ba), replace the name with the one you have assigned to the master node on your Kubernetes cluster.

3\. Verify that label had been created successfully

```
kubectl get nodes --show-labels | grep node-type
```

## Install

1.  Create a `container-registry` namespace, if it doesn’t exists yet

```
kubectl create namespace container-registry
```

2\. Add the `twuni/docker-registry` Helm repository successor of previous `stable/docker-registry`

```
helm repo add twuni https://helm.twun.io
```

3\. Update local Helm chart repository cache

```
helm repo update
```

4\. Search for latest `twuni/docker-registry` Helm chart version

```
helm search repo docker-registry   # NAME                 CHART VERSIONAPP VERSION# twuni/docker-registry1.10.1       2.7.1
```

5\. Install the `docker-registry` Helm chart using the version from previous step:

_With persistence and node affinity:_

```
helm upgrade --install docker-registry \    --namespace container-registry \    --set replicaCount=1 \    --set persistence.enabled=true \    --set persistence.size=60Gi \    --set persistence.deleteEnabled=true \    --set persistence.storageClass=docker-registry-local-storage \    --set persistence.existingClaim=docker-registry-pv-claim \    --set secrets.htpasswd=$(cat $HOME/temp/registry-creds/htpasswd) \    --set nodeSelector.node-type=master \    twuni/docker-registry \    --version 1.10.1
```

_Without persistence or node affinity:_

```
helm upgrade --install docker-registry \    --namespace container-registry \    --set replicaCount=1 \    --set secrets.htpasswd=$(cat $HOME/temp/registry-creds/htpasswd) \    twuni/docker-registry \    --version 1.10.1
```

6\. Verify installation

```
# Make sure docker-registry pod is runningkubectl get pods --namespace container-registry | grep docker-registry
```

## Uninstall

1.  Remove `docker-registry` from the cluster

```
helm uninstall docker-registry --namespace container-registry
```

2\. Clear the `container-registry` namespace

```
kubectl delete namespaces container-registry
```

## Registry Ingress

We will expose the Docker registry using `traefik` ingress controller, it will allow access to the the registry via HTTPS with proper TLS/Cert.

## Certificate

We will create a certificate using `cert-manager` to allow accessing the `docker-registry` using the hosted name `registry.MY_DOMAIN.com` within our home network. Create a self signed certificate as described in [here](https://faun.pub/install-certificate-manager-controller-in-kubernetes-ba435aedf2e8#142c) under `container-registry` namespace.

Verify that a TLS secret had been created for the certificate:

```
kubectl get secret MY_DOMAIN-com-cert-secret --namespace container-registry
```

## Configure

1.  Create an `IngressRoute` for accessing the Docker registry, make sure to replace `MY_DOMAIN` with your domain name

```
cat <<EOF | kubectl apply -f -apiVersion: traefik.containo.us/v1alpha1kind: IngressRoutemetadata:  name: docker-registry  namespace: container-registryspec:  entryPoints:    - websecure  routes:    - kind: Rule      match: Host(\`registry.MY_DOMAIN.com\`)      services:        - kind: Service          port: 5000          name: docker-registry          namespace: container-registry      # No basic auth middleware is required since secrets are set up during registry creation      middlewares:        - name: docker-registry-cors          namespace: container-registry  tls:    secretName: MY_DOMAIN-com-cert-secret---apiVersion: traefik.containo.us/v1alpha1kind: Middlewaremetadata:  name: docker-registry-cors  namespace: container-registryspec:  headers:    accessControlAllowMethods:      - GET      - OPTIONS      - PUT      - POST      - DELETE    accessControlAllowOriginList:      - https://registry-ui.MY_DOMAIN.com    accessControlAllowCredentials: true    accessControlMaxAge: 100    addVaryHeader: trueEOF
```

> **Note:** A `Middleware` resource is being created to allow [CORS (Cross Origin Resource Sharing)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) requests from the registry UI service to the registry domain.

2\. Check the resource was created successfully

```
kubectl describe ingressroute docker-registry --namespace container-registry
```

3\. Define the hosted name `registry.MY_DOMAIN.com` on the client machine (laptop/desktop)

```
# Edit the file using `sudo /etc/hosts`# Append manually to the existing k3s master hosted name 111.222.333.444 kmaster, registry.MY_DOMAIN.com# Alternatively, add a new hosted name entry with a one-linerecho -e "111.222.333.444\tregistry.MY_DOMAIN.com" | sudo tee -a /etc/hosts
```

> **Note:** Replace `111.222.333.444` with the node IP address the registry was deployed on, which in the example above is the `kmaster` node. Replace `MY_DOMAIN` with your domain name.

4\. Verify access to the registry by replacing `MY_DOMAIN` with yours

```
export USER=$(cat $HOME/temp/registry-creds/registry-user.txt)export PASSWORD=$(cat $HOME/temp/registry-creds/registry-pass.txt)curl -kiv -H \  "Authorization: Basic $(echo -n "${USER}:${PASSWORD}" | base64)" \  https://registry.MY_DOMAIN.com/v2/_catalog    wget --no-check-certificate --header \  "Authorization: Basic $(echo -n "${USER}:${PASSWORD}" | base64)" \  https://registry.MY_DOMAIN.com/v2/_catalog     unset USER PASSWORD
```

5\. Open browser at [https://registry.MY\_DOMAIN.com/v2/\_catalog](https://registry.my_domain.com/v2/_catalog)

> **Note:** If you encounter an untrusted certificate warning, please follow the cert-manager post [Trust section](https://faun.pub/install-certificate-manager-controller-in-kubernetes-ba435aedf2e8#df0d) for instructions on how to trust the `registry` certificate.

## Dashboard

We will install a web user interface to simplify interactions with the private Docker registry. The dashboard we will use is based on [Joxit Docker Registry UI](https://github.com/Joxit/docker-registry-ui) which is an excellent lightweight and simple solution for Docker registry web UI ([see example](https://joxit.dev/docker-registry-ui/demo/)).

The Docker Registry UI repository has a helm chart but it is missing a chart `index.yaml` metadata, as a result we will have to clone the repository and perform a Helm installation from disk manually.

We will rely on previously installed docker registry, thus we will indicate in the dashboard Helm chart that an external repository should be used instead of creating its own.

## Install

1.  Clone `docker-registry-ui` repository

```
mkdir -p $HOME/temp/docker-registry-uigit clone https://github.com/Joxit/docker-registry-ui.git $HOME/temp/docker-registry-ui
```

2\. Install the `docker-registry-ui` Helm chart from local path, replace `MY_DOMAIN` with yours

```
helm upgrade --install docker-registry-ui \    --namespace container-registry \    --set registry.external=true \    --set registry.url=https://registry.MY_DOMAIN.com \    --set ui.title="Docker Registry UI" \    --set ui.replicaCount=1 \    --set ui.nodeSelector.node-type=master \    --set ui.image.tag=main \    --set ui.delete_images=true \    --set ui.ingress.enabled=false \    --set ui.proxy=false \    $HOME/temp/docker-registry-ui/examples/helm/docker-registry-ui
```

> **Important:** Make sure that the Docker image `main` tag in use by `ui.image.tag` attribute has a Raspberry Pi ARMv7 supported OS architecture. Check [here](https://hub.docker.com/r/joxit/docker-registry-ui/tags?page=1&ordering=last_updated) for available `docker-registry-ui` image tags.
> 
> **Note:** Notice that we rely on a `nodeSelector` label that we have created before so the registry UI would get deployed on the same node of the registry itself which is on the master node.

3\. Verify installation was successful

```
# Make sure docker-registry-ui pod is runningkubectl get pods --namespace container-registry | grep docker-registry-ui
```

## Uninstall

1.  Remove `docker-registry-ui` from the cluster

```
helm uninstall docker-registry-ui --namespace container-registry
```

## Known Issues

The registry web UI has a few known issues, in here you will find ways to mitigate them.

**Image Deletion**

When deleting an image from the registry web UI, the image name remains and still shown in the UI, follow [this GitHub issue](https://github.com/Joxit/docker-registry-ui/issues/77) for additional information.

If the empty image names bothers you, the easiest way to clear them out is to delete the image folder from the mounted storage that holds the private registry images.

1.  SSH to the node that is connect to the registry storage (`kmaster` as as described in this post)

```
ssh pi@kmaster
```

2\. Delete the directory of the image you removed from the web UI (which still remains as a list entry)

```
sudo rm -rf \  /mnt/container-registry/docker/registry/v2/repositories/<IMAGE-NAME-TO-DELETE>
```

## Dashboard Ingress

We will expose the Docker registry UI using `traefik` ingress controller, it will allow access to the the registry UI via HTTPS with proper TLS/Cert.

## Authentication

We will create basic authentication for accessing the Docker registry Web UI.

1.  Generate a user & password using `htpasswd`

```
bash <<'EOF'   # Change these credentials to your ownexport REGISTRY_UI_USER=adminexport REGISTRY_UI_PASS=dashboardexport DESTINATION_FOLDER=${HOME}/temp/registry-ui-creds# Backup credentials to local files (in case you'll forget them later on)mkdir -p ${DESTINATION_FOLDER}echo ${REGISTRY_UI_USER} >> ${DESTINATION_FOLDER}/registry-ui-user.txtecho ${REGISTRY_UI_PASS} >> ${DESTINATION_FOLDER}/registry-ui-pass.txthtpasswd -Bbn ${REGISTRY_UI_USER} ${REGISTRY_UI_PASS} \    > ${DESTINATION_FOLDER}/htpasswd   unset REGISTRY_UI_USER REGISTRY_UI_PASS DESTINATION_FOLDER   EOF
```

2\. Create a Kubernetes secret to be used for dashboard access

```
kubectl create secret generic docker-registry-ui-secret \    --from-file=$HOME/temp/registry-ui-creds/htpasswd \    --namespace container-dashboard
```

3\. Verify secret created successfully

```
kubectl get secret docker-registry-ui-secret --namespace container-dashboard
```

## Certificate

We will create a certificate using `cert-manager` to allow accessing the `docker-registry-ui` using the hosted name `registry-ui.MY_DOMAIN.com` within our home network. Create a self signed certificate as described in [here](https://blog.zachinachshon.com/cert-manager/#self-signed-certificate) under `container-registry` namespace.

> **Note:** Please skip to the `Configure` instructions in case you have already created a certificate for the Docker registry in the previous steps since they both share the same wildcard domain `*.MY_DOMAIN.com` under the same namespace.

Verify that a TLS secret had been created for the certificate:

```
kubectl get secret MY_DOMAIN-com-cert-secret --namespace container-registry
```

## Configure

1.  Create an `IngressRoute` for accessing the Docker registry web UI, make sure to replace `MY_DOMAIN` with your domain name

```
cat <<EOF | kubectl apply -f -apiVersion: traefik.containo.us/v1alpha1kind: IngressRoutemetadata:  name: docker-registry-ui  namespace: container-registryspec:  entryPoints:    - websecure  routes:    - kind: Rule      match: Host(\`registry-ui.MY_DOMAIN.com\`)      services:        - kind: Service          port: 80          name: docker-registry-ui-ui # Suffix added by the Helm chart           namespace: container-registry      middlewares:        - name: docker-registry-ui-auth          namespace: container-registry  tls:    secretName: MY_DOMAIN-com-cert-secret---apiVersion: traefik.containo.us/v1alpha1kind: Middlewaremetadata:  name: docker-registry-ui-auth  namespace: container-registryspec:  basicAuth:    secret: docker-registry-ui-secretEOF
```

2\. Check the resource was created successfully

```
kubectl describe ingressroute docker-registry-ui --namespace container-registry
```

3\. Define the hosted name `registry-ui.MY_DOMAIN.com` on the client machine (laptop/desktop)

```
# Edit the file using `sudo /etc/hosts`# Append manualy to the existing k3s master hosted name 111.222.333.444 kmaster, registry.MY_DOMAIN.com, registry-ui.MY_DOMAIN.com# Alternatively, add a new hosted name entry with a one-linerecho -e "111.222.333.444\tregistry-ui.MY_DOMAIN.com" | sudo tee -a /etc/hosts
```

> **Note:** Replace `111.222.333.444` with the node IP address the registry web UI was deployed on, which in the example above is the `kmaster` node. Replace `MY_DOMAIN` with your domain name.

4\. Open browser at [https://registry-ui.MY\_DOMAIN.com](https://registry-ui.my_domain.com/) and add the registry URL as a new server

> **Note:** If you encounter an untrusted certificate warning, please follow the cert-manager post [Trust section](https://faun.pub/install-certificate-manager-controller-in-kubernetes-ba435aedf2e8#df0d) for instructions on how to trust the `registry-ui` certificate.

5\. Register the docker registry URL as a new server

**Fresh Registry UI:**

![](https://miro.medium.com/max/1400/1*0KtC7YUHuF47FVo-tqlRtQ.png)

**Add a New Server:**

![](https://miro.medium.com/max/1400/1*fK5M8wDgTR7f8537nG1UBA.png)

> **Note:** Make sure to supply basic auth credentials for both registry and registry-ui when asked via the user/pass pop-up.

## Docker Registry Mirror

Now that we have a running private Docker registry, we would like to interact with it from within the Kubernetes cluster (`k3s` in our case) and allow nodes to pull private images. In order to so that we should tell Kubernetes that `registry.MY_DOMAIN.com` is another mirror for pulling docker images.

> **Note:** These instructions are relevant for the Rancher Labs Kubernetes distribution `k3s`. For other distributions, please refer to the official documentation for adding another Docker registry mirror.

`k3s` uses a container runtime called _containerd_ directly (no docker). We will create a configuration file containing our private registry domain name, auth secrets and certificate secrets.

_These instructions_ **_MUST be applied_** _on all the Kubernetes nodes on the Raspberry Pi cluster, in this example we will focus on the_ `_kmaster_` _node._

1.  Copy the certificate secrets (_cert\_file_, _key\_file_, _ca\_file_) to **each of the cluster nodes**

```
# Create a dedicated folder on remote serverssh pi@kmaster "sudo mkdir -p /tmp/container-registry"   # Change permissions on the destination folderssh pi@kmaster "sudo chmod 777 /tmp/container-registry"   # Copy client certificate, client key, CA certificatescp -r $HOME/temp/container-registry/cert-secrets pi@kmaster:/tmp/container-registry
```

> **Note:** This step assume that the certificate secrets were exported locally to `$HOME/temp/container-registry/cert-secrets` as instructed in the [cert-manager post](https://faun.pub/install-certificate-manager-controller-in-kubernetes-ba435aedf2e8#ca75).

2\. Create a `/etc/rancher/k3s/registries.yaml` file on the remote server, replace `MY_DOMAIN`, user & pass where necessary

```
ssh pi@kmaster "sudo tee /etc/rancher/k3s/registries.yaml > /dev/null <<EOTmirrors:  \"registry.MY_DOMAIN.com\":    endpoint:      - \"https://registry.MY_DOMAIN.com\"configs:  \"registry.MY_DOMAIN.com\":    auth:      username: <insert-registry-user-here>      password: <insert-registry-pass-here>    tls:      cert_file: /tmp/container-registry/cert-secrets/cert_file.crt      key_file: /tmp/container-registry/cert-secrets/key_file.key      ca_file: /tmp/container-registry/cert-secrets/ca_file.crtEOT"
```

3\. Verify the file created successfully

```
ssh pi@kmaster "cat /etc/rancher/k3s/registries.yaml"
```

4\. Add the host name `registry.MY_DOMAIN.com` to the node’s hosts file (only if we are using a non-registered self domain)

**On the registry node**

```
# Append to kmaster node hosts filessh pi@kmaster "echo -e "127.0.1.1\tregistry.MY_DOMAIN.com" | sudo tee -a /etc/hosts"
```

**On other cluster nodes**

```
# Append to worker node hosts file (Replace X with node number)ssh pi@knodeX "echo -e "REPLACE_WITH_REGISTRY_NODE_IP\tregistry.MY_DOMAIN.com" | sudo tee -a /etc/hosts"
```

5\. Restart `k3s` service on master or agent nodes, according to which node you are adding the docker mirror

```
# Restart master nodessh pi@kmastersudo systemctl restart k3s   # Restart worker nodeX (replace X with node number)ssh pi@knodeXsudo systemctl restart k3s-agent
```

> **Note:** We must restart the `k3s-server` / `k3s-agent` for registry changes to take effect since `k3s` will check to see if a `registries.yaml` file exists upon startup and instruct containerd to use any registries defined in the file as Docker mirrors.

6\. Verify k3s server/agent started in **_active (running)_** mode

```
ssh pi@kmaster systemctl status k3s.service    ssh pi@knodeXsystemctl status k3s-agent.service
```

> **Note:** Perform these steps again for every Kubernetes node which is a part of the Raspberry Pi cluster i.e. `kmaster`, `knode1`, `knode2` … `knodeN`.

## Usage

In this section we will verify that we can interact with the private Docker registry from:

-   Client machine (laptop / desktop)
-   Raspberry Pi cluster node(s)
-   Kubernetes cluster by deploying a pod that is using a private image

## Docker Client

Docker client has two approaches of interacting with a private docker registry — _Trusted_ vs. _Insecure_.

**Trusted Registry**

When interacting with the private Docker registry using a Docker client, it is advised to have a valid certificate for `registry.MY_DOMAIN.com`. If we’re using a self signed one we can register it as specified in the [Trust section](https://faun.pub/install-certificate-manager-controller-in-kubernetes-ba435aedf2e8#df0d) of the cert-manager post.

**Insecure Registry**

If you want the fast-and-dirty approach (which isn’t recommended) to push images from a client machine to the private Docker registry, you can tell Docker to treat the private registry as insecure.

1.  Edit the `deamon.json` (on macOS via `Docker` -> `Preferences` -> `Docker Engine`)
2.  Add the following:

```
{   "insecure-registries" : ["registry.MY_DOMAIN.com"]}
```

3\. Restart Docker

**Pull/Tag/Push**

We will use a Docker client to pull a `busybox` image for ARM architecture, tag it and push it to the private Docker registry.

> **Important:** It is crucial to understand that the architecture of our client machines (usually not ARM based) is different from the architecture of the Raspberry Pi boards (ARM based).

The fact that we are working on different architectures prevent us from pulling images on client machine (would pull according to the local machine arch), pushing them to the private registry and then pulling them from within the cluster (unsupported arch).

Doing so would result on the `exec format error` error, for example:

```
standard_init_linux.go:219: exec user process caused: exec format errorpod "busybox-blog-hello" deletedpod playground/busybox-blog-hello terminated (Error)
```

In order to solve that limitation, we’ll have to specify the exact build SHA of the image when pulling from a non-ARM based machine.

1.  Login to the remote private registry

```
docker login \   -u $(cat $HOME/temp/registry-creds/registry-user.txt) \   -p $(cat $HOME/temp/registry-creds/registry-pass.txt) \   https://registry.MY_DOMAIN.com
```

2\. Get the `busybox:latest` SHA of the tag which is suitable to the `linux/arm/v7` architecture, [click here for available tags](https://hub.docker.com/_/busybox?tab=tags&page=1&ordering=last_updated&name=latest)

3\. Pull the `busybox` image suitable for `arm/v7` architecture

```
docker pull \ busybox@sha256:fd659a6f4786d18666586ab4935f8e846d7cf1ff1b2709671f3ff0fcd15519b9
```

4\. Tag image with a custom name and append the private registry domain name prefix

```
docker tag \ busybox@sha256:fd659a6f4786d18666586ab4935f8e846d7cf1ff1b2709671f3ff0fcd15519b9 \ registry.MY_DOMAIN.com/busybox-blog-armv7
```

5\. Push image to the private registry

```
docker push registry.MY_DOMAIN.com/busybox-blog-armv7
```

6\. Verify that you can read the image manifest

```
curl -ksS \   -u $(cat $HOME/temp/registry-creds/registry-user.txt):$(cat $HOME/temp/registry-creds/registry-pass.txt) \   https://registry.MY_DOMAIN.com/v2/busybox-blog-armv7/manifests/latest
```

7\. **(Optional)**: verify the image information on the Docker registry web UI at `registry-ui.MY_DOMAIN.com`

![](https://miro.medium.com/max/1400/1*2DHiz3bw8w9aPrcHJlWkfQ.png)

## crictl

**What is** `**crictl**` **?**

It is a CLI of the container runtime which we use from within a Raspberry Pi server to interact with the private registry.

**How to install** `**crictl**` **?**

The `k3s` Kubernetes distribution we are using on top of the Raspberry Pi cluster installed `crictl` for us. The RPi boards architecture is `ARMv7`, it means that if we will pull images from within a RPi server using `crictl` it will pull the appropriate `arm/v7` image built for the ARM architecture, rather than relying on a specific SHA (as described on the `Docker Client` section).

> **Tip:** It is advised to verify that the image you pull contains a tag for the architecture that you require. For `busybox` image check the available tags [in here](https://hub.docker.com/_/busybox?tab=tags&page=1&ordering=last_updated&name=latest).
> 
> **Important:** Unfortunately, `crictl` doesn’t support tag & push operations, you are welcome to follow [this GitHub feature request](https://github.com/kubernetes-sigs/cri-tools/issues/438) if anything should change in the future.

**Pull**

1.  SSH into a remote server, either master or worker

```
# Connect to master nodessh pi@kmaster   # Connect to worker node (replace X with node number)ssh pi@knodeX
```

2\. Pull the `busybox-blog-armv7` image we have previously pushed using Docker client, replace to the appropriate user & pass

```
sudo crictl pull \   --creds <REGISTRY_USER>:<REGISTRY_PASS> \   registry.MY_DOMAIN.com/busybox-blog-armv7
```

## Kubernetes Pod

We will create a running Kubernetes pod on the `playground` namespace using an image pulled from the private registry. Make sure to follow the `Docker Client` step to properly push the `busybox-blog-armv7` image to the private registry.

1.  Create an ephemeral pod that prints the node architecture, a hello greeting and terminates itself

```
kubectl run --rm -t -i busybox-blog-hello \   --namespace playground \   --restart Never \   --image registry.MY_DOMAIN.com/busybox-blog-armv7 \   --overrides='{"spec": { "nodeSelector": {"node-type": "master"}}}' \   -- sh -c 'sleep 3; \             echo "Architecture: $(uname -m)"; \             echo "Message: Hello from Kubernetes!"'
```

> **Note:** Replace the node selector according to the node type the pod should be assigned to i.e. for checking private registry on worker node #1 we should use `"node-type": "node1"`.

2\. Verify the pod is created successfully

```
kubectl describe pod busybox-blog-hello --namespace playground
```

Pod Pull Image Events:

![](https://miro.medium.com/max/1400/1*emQJ_Jmv2aHwQky9YcmG9A.png)

## Summary

**Wow !** This post is indeed a long one with many pieces of information required to be glued together to grasp the big picture.

I hope that you have successfully created a running private registry and by doing that, you feel confident enough to host freely any image you can think about on any storage device as simple as any USB stick lying around to use within your Kubernetes cluster 😍

Don’t limit yourself to a pricing/subscription model, just replace/add a new storage device to suite your needs without suffering from any data transfer cap for your homelab.

**What now?** Build any custom images you wish and store them within the new private Docker registry you have created. Stay tuned for blog posts describing real use cases of custom images hosted on the private Docker registry.

Please leave your comment, suggestion or any other input you think is relevant to this post in the discussion below.

**Like this post?**  
You can find more by:

Checking out my blog: [https://blog.zachinachshon.com](https://blog.zachinachshon.com/)  
Following me on Twitter: [@zachinachshon](https://twitter.com/zachinachshon)

Thanks for reading! ❤️

[

![](https://miro.medium.com/max/1400/1*BCiLLad3dvZLwBa-B5cAVQ.png)

](https://faun.to/bP1m5)

Join FAUN: [**Website**](https://faun.to/i9Pt9) 💻**|**[**Podcast**](https://faun.dev/podcast) 🎙️**|**[**Twitter**](https://twitter.com/joinfaun) 🐦**|**[**Facebook**](https://www.facebook.com/faun.dev/) 👥**|**[**Instagram**](https://instagram.com/fauncommunity/) 📷|[**Facebook Group**](https://www.facebook.com/groups/364904580892967/) 🗣️**|**[**Linkedin Group**](https://www.linkedin.com/company/faundev) 💬**|** [**Slack**](https://faun.dev/chat) 📱**|**[**Cloud Native** **News**](https://thechief.io/) 📰**|**[**More**](https://linktr.ee/faun.dev/)**.**

**If this post was helpful, please click the clap 👏 button below a few times to show your support for the author 👇**