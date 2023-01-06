[![trackgit-views](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAAAAACIM/FCAAACh0lEQVR4Ae3ch5W0OgyG4dt/mQJ2xgQPzJoM1m3AbALrxzrf28FzsoP0HykJEEAAAUQTBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEkKK0789+GK/I2ezfQB522PnS1qc8pGgXvr4tE4aY0XOUWlGImThWgyCk6DleixzE7qwBkg/MGiDPlVVAyp1VQGrPKiACDhFI6VkF5LmzCki+sg7IwDoglnVAil0IMkeG9CyUiwsxLFUVFzJJOQaKCjFCDN9RXMjIX7W6ztZXZDKKCyn8sWJvH+nca7WHDN9lROlAliPH9iRKCPI4cswFJQWxB46toLQgQ9jhn5QYZA9DOkoMUoQde5YapAxDWkoNYsOQR3KQd9CxUnIQF4S49CB9ENKlBxmDEKsFUgMCCCCAAHIrSF61f6153Ajy8nyiPr8L5MXnmm4CyT2fzN4DUvHZ+ntA2tOQBRBAAAEEEEAAAQQQ7ZBaC6TwSiDUaYHQ2yuB0MN+ft+43whyrs4rgVCjBUKTFshLC6TUAjGA3AxSaYFYLZBOC2RUAsk8h5qTg9QcbEoOsoQhQ2qQhsO5xCD5dgB5JQaZ+KBKGtKecvR81Ic0ZDjByKdDx0rSEDZ/djQbH+bkIdvfJFm98BfV8hD2zprfVdlu9PxVeyYAkciREohRAplJCaRSAplJCcQogTjSAdlyHRBvSAekJR0QRzogA+mADJkOiCPSAPEtqYBshlRAXC43hxix2QiOuEZkVERykGyNo9idIZKE0HO7XrG6OiMShlDWjstVzdPgXtUH9v0CEidAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQP4HgjZxTpdEii0AAAAASUVORK5CYII=)](https://gitee.com/link?target=https%3A%2F%2Ftrackgit.com)

___

[Docs](https://gitee.com/link?target=https%3A%2F%2Fwww.sealos.io%2Fdocs%2FIntro) | [简体中文](https://gitee.com/link?target=https%3A%2F%2Fwww.sealos.io%2Fzh-Hans%2Fdocs%2FIntro)

## [](https://gitee.com/mirrors/sealos#run-a-kubernetes-cluster)Run a Kubernetes cluster

[![asciicast](https://asciinema.org/a/519263.svg)](https://gitee.com/link?target=https%3A%2F%2Fasciinema.org%2Fa%2F519263%3Fspeed%3D3)

## [](https://gitee.com/mirrors/sealos#what-is-sealos)What is sealos

**sealos is a cloud operating system distribution based on Kubernetes.**

![](https://user-images.githubusercontent.com/8912557/173866494-379ba0dd-05af-4095-b63d-08f594581c52.png)

-   From now on, think of all your machines as an abstract supercomputer whose operating system is sealos, where Kubernetes serves as the OS kernel.
-   Instead of IaaS, PaaS and SaaS, there will only be cloud OS drivers(CSI, CNI and CRI implementations), cloud OS kernel(Kubernetes) and distributed applications.

## [](https://gitee.com/mirrors/sealos#this-is-not-win11-but-sealos-desktop)This is not win11 but sealos desktop

Use the cloud like a PC desktop, Freely run and uninstall any distributed applications:

![](https://user-images.githubusercontent.com/8912557/191533678-6ab8915e-23c7-456e-b0c0-506682c001fb.png)

## [](https://gitee.com/mirrors/sealos#core-features)Core features

-   Manage clusters lifecycle
    -   Quickly install HA Kubernetes clusters
    -   Add / remove nodes
    -   Clean the cluster, backup and auto recovering, etc.
-   Download and use OCI-compatible distributed applications
    -   OpenEBS, MinIO, Ingress, PostgreSQL, MySQL, Redis, etc.
-   Customize your own distributed applications
    -   Using Dockerfile to build distributed applications images, saving all dependencies.
    -   Push distributed applications images to Docker Hub.
    -   Combine multiple applications to build your own cloud platform.
-   Sealos cloud
    -   Run any distributed applications
    -   Have a full public cloud capability, and run it anywhere

## [](https://gitee.com/mirrors/sealos#quickstart)Quickstart

> Installing an HA Kubernetes cluster with calico as CNI

Here `kubernetes:v1.24.0` and `calico:v3.24.1` are the cluster images in the registry which are fully compatible with OCI standard. Wonder if we can use flannel instead? Of course!

```
# Download and install sealos. sealos is a golang binary so you can just download and copy to bin. You may also download it from release page.
$ wget  https://github.com/labring/sealos/releases/download/v4.1.3/sealos_4.1.3_linux_amd64.tar.gz  && \
    tar -zxvf sealos_4.1.3_linux_amd64.tar.gz sealos &&  chmod +x sealos && mv sealos /usr/bin 
# Create a cluster
$ sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 \
     --masters 192.168.64.2,192.168.64.22,192.168.64.20 \
     --nodes 192.168.64.21,192.168.64.19 -p [your-ssh-passwd]
```

-   Supported Kubernetes versions: [240+ Kubernetes versions](https://gitee.com/link?target=https%3A%2F%2Fhub.docker.com%2Fr%2Flabring%2Fkubernetes%2Ftags) [Kubernetes use cri-docker runtime](https://gitee.com/link?target=https%3A%2F%2Fhub.docker.com%2Fr%2Flabring%2Fkubernetes-docker%2Ftags)
-   Other distributed [applications images](https://gitee.com/link?target=https%3A%2F%2Fhub.docker.com%2Fu%2Flabring)

> Single host

```
$ sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 --single
# remove taint
$ kubectl taint node --all node-role.kubernetes.io/control-plane-
```

> Building a custom cluster image

See [Building an Example CloudImage](https://gitee.com/link?target=https%3A%2F%2Fwww.sealos.io%2Fdocs%2Fexamples%2Fbuild-example-cloudimage).

> Storage, message queue, database, etc.

Don't be shocked by the following:

```
sealos run labring/helm:v3.8.2 # install helm
sealos run labring/openebs:v1.9.0 # install openebs
sealos run labring/minio-operator:v4.4.16 labring/ingress-nginx:4.1.0 \
   labring/mysql-operator:8.0.23-14.1 labring/redis-operator:3.1.4 # oneliner
```

And now everything is ready.

## [](https://gitee.com/mirrors/sealos#use-cri-docker-image)Use cri-docker image

```
sealos run labring/kubernetes-docker:v1.20.5-4.1.3 labring/calico:v3.24.1 \
     --masters 192.168.64.2,192.168.64.22,192.168.64.20 \
     --nodes 192.168.64.21,192.168.64.19 -p [your-ssh-passwd]
```

## [](https://gitee.com/mirrors/sealos#links)Links

-   [Contribution Guidelines](https://gitee.com/mirrors/sealos/blob/main/CONTRIBUTING.md)
-   [Development Guide](https://gitee.com/mirrors/sealos/blob/main/DEVELOPGUIDE.md)
-   [sealos 3.0(older version)](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Flabring%2Fsealos%2Ftree%2Frelease-v3.3.9%23readme) For older version users. Note that sealos 4.0 includes significant improvements, so please upgrade ASAP.
-   [buildah](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fcontainers%2Fbuildah) Capabilities of buildah is widely used in sealos 4.0 to make cluster images compatible with container images and docker registry.
-   [sealer](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fsealerio%2Fsealer) Capabilities of sealer is widely used in sealos 4.0 to make Clusterfile compatible with sealer, some module forked sealer source code.

**Join us: [Telegram](https://gitee.com/link?target=https%3A%2F%2Ft.me%2Fcloudnativer), QQ Group(98488045), Wechat：fangnux**

## [](https://gitee.com/mirrors/sealos#license)License

[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Flabring%2Fsealos.svg?type=large)](https://gitee.com/link?target=https%3A%2F%2Fapp.fossa.com%2Fprojects%2Fgit%252Bgithub.com%252Flabring%252Fsealos%3Fref%3Dbadge_large)