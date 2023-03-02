## Harden Kubernetes cluster with Pod and container security contexts

![](https://miro.medium.com/max/770/1*UcX8jFTT6Tk8HrQNBFPJPA.png)

Kubernetes security context

When it comes to security in Kubernetes, It is very vital to secure the individual resources of the cluster. Pods and containers are considered the core resources running in the cluster and are the fundamental building block of Kubernetes workloads. That’s why applying security to that layer will have a huge impact on your overall security and configuration of your cluster and will minimize the risks and vulnerabilities in your cluster.

In this blog post, we’ll demonstrate how to harden Kubernetes cluster through security contexts and apply them to pods and containers.

**Security Context as Concept**

It is a mechanism or tool used by SELinux (Security Enhanced Linux) to enforce access controls and apply certain labels/contexts to the objects (files/directories) in the system, in other words, when an application/process try to access certain files, SELinux checks the security context of the process and the file and determine whether the process has the required permissions to access the file or not.

**Security Context as Implementation in Kubernetes**

Security contexts in Kubernetes are considered one of the most important features to harden and secure Kubernetes clusters. They allow you to control the behavior of the running pods and containers and how they interact with host server, OS and kernel. They authorize to bind certain resources in your cluster (pods and containers) to specific users or groups, restrict pods and containers from interacting with the host operating system processes, other pods and services and allow them to perform their intended tasks while staying secure at the same time.

**Samples of security control that can be handled through security contexts include:**

-   Limiting the privileges that a containerised application receives from the host OS (read-only vs read-write)
-   Specifying what network ports are allowed to communicate with the host network and other nodes in a cluster
-   Preventing applications from making direct calls to the kernel via the file system.
-   Blocking network traffic from a container to the host OS.
-   Restricting the number of processes that can be created in a container.
-   Which SELinux context to be applied to the container’s process while it is running.

**Now let’s jump to a demo which will consists of 3 parts:**

-   Risks behind running apps with default configuration
-   Applying security contexts on pod level
-   Applying security contexts on pod and container level

**Prerequisites:**

-   Up and running Kubernetes cluster. [minikube](https://minikube.sigs.k8s.io/docs/start/) or [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) can be used to start one.
-   Familiarity with kubectl commands.

1- Let’s start the demo by clarifying the **Risks behind running apps with default configuration:**

The best way to tackle this topic is to demonstrate it from a practical way. So let’s apply the below yaml manifest to run pod with default configuration.

```
echo "apiVersion: v1kind: Podmetadata:  name: security-context-demo-defaultspec:  volumes:  - name: sec-ctx-vol-default    emptyDir: {}  containers:  - name: sec-ctx-demo-default    image: busybox:1.28    command: [ "sh", "-c", "sleep 1h" ]    volumeMounts:    - name: sec-ctx-vol-default      mountPath: /data/demo" | kubectl apply -f -
```

Now let’s jump into the pod with sh terminal:

```
kubectl exec -it security-context-demo-default -- sh
```

Once you execute the above command, a terminal will be prompted as (/ #) which means you are root inside the pod, also we can try id command to show the exact user and group ids:

```
# iduid=0(root) gid=0(root) groups=10(wheel)
```

**The risks** behind it, is the root user here is as the same root user on your host and sharing the same kernel. Isolation is only provided by the Container Runtime Interface CRI isolation mechanism like docker.  
Also running as root means user will have access to all file system and can be easily edited, any packages can be installed and files can be overwritten.

Another level of administration permissions can be given to the root user if we add a security context as below:

```
containers:  - name: sec-ctx-demo    image: busybox:1.28    command: [ "sh", "-c", "sleep 1h" ]    volumeMounts:    - name: sec-ctx-vol      mountPath: /data/demo    securityContext:      privileged: true
```

Which can give the root user inside that container access to all the volumes mounted in the cluster and of course all sensitive data and credentials. If we try to redeploy the pod with the above security context, then list the available volumes, we will be able to see all system volumes which is a breach and could allow attackers to perform root-privileged actions to our system.

```
#ls /dev
```

![](https://miro.medium.com/max/770/1*IUkajZWzoe761LUrn_V1dA.png)

A command to show us the available volumes from inside the container

**2- Applying security contexts on pod level:**

Now let’s apply some security contexts to the pod level of a deployment and make sure they are applied correctly and as expected.  
securityContext is part of pod/spec or pod/spec/containers sections in the deployment file and can be explained through the below commands:

```
kubectl explain pod.spec.securityContext | morekubectl explain pod.spec.containers.securityContext | more
```

You will be able to see a detailed explanation of securityContext object:

![](https://miro.medium.com/max/770/1*mqZuS4AxtOtlnDvjZ_-gBA.png)

Sample from the expected output

Now let’s apply a YAML deployment to run a demo pod with security context defined and see how it will impact the behaviour of our app.

```
echo "apiVersion: v1kind: Podmetadata:  name: security-context-demospec:  securityContext:    runAsUser: 1000    runAsGroup: 3000    fsGroup: 2000  volumes:  - name: sec-ctx-vol    emptyDir: {}  containers:  - name: sec-ctx-demo    image: busybox:1.28    command: [ "sh", "-c", "sleep 1h" ]    volumeMounts:    - name: sec-ctx-vol      mountPath: /data/demo" | kubectl apply -f -
```

As you can see in securityContext section, we configured the pod to run any container process with user 1000, group 3000 and supplementary group 2000.

N**ote here** _whatever will be created under the mounted volume will take the value of runAsUser and fsGroup. And this security context will be applied to all the containers running in this pod._

Now let’s verify that through running a shell in the pod:

```
kubectl exec -it security-context-demo -- sh
```

Then checking the user running the container process through _id_ command:

```
$iduid=1000 gid=3000 groups=2000
```

Also let’s check the processes that are running in the container:

```
$ps -efPID   USER     TIME  COMMAND    1 1000      0:00 sleep 1h   17 1000      0:00 sh   24 1000      0:00 ps -ef
```

It shows that they are running as user 1000 which is what configured as runAsUser

Now let’s go to the mounted volume, create a file and check its permissions:

```
$cd /data/demo$touch testfile$ls -lt-rw-r--r--    1 1000 2000   6 Nov 12 09:49 testfile
```

As you can see that testfile is owned by user 1000 and group 2000 which are configured as runAsUser and fsGroup.

So now we can say that our configuration are configured and tested correctly.

**3- Applying security contexts on container and pod level:**

Now we will try to add security context to both pod and container specs, Let’s apply the below command:

N**ote here** that security context in the container section will have the upper hand and will overwrite the security context in pod/spec section

```
echo "apiVersion: v1kind: Podmetadata:  name: security-context-demospec:  securityContext:    runAsNonRoot: true    runAsUser: 1000    runAsGroup: 3000    fsGroup: 2000  volumes:  - name: sec-ctx-vol    emptyDir: {}  containers:  - name: sec-ctx-demo    image: busybox:1.28    command: [ "sh", "-c", "sleep 1h" ]    volumeMounts:    - name: sec-ctx-vol      mountPath: /data/demo    securityContext:      runAsUser: 10000      readOnlyRootFilesystem: true" | kubectl apply -f -
```

Here we modified the security context of the pod to force the containers running inside it to run as non-root, also we added a security context for the container to run as user 10000 and mount the root filesystem as readonly.

So what is expected here is to have a running container with the below specs:  
\- Non-root user  
\- User ID 10000  
\- Group ID 3000  
\- Supplementary group 2000  
\- Read-only root file system

Now let’s verify that through running a shell in the pod:

```
kubectl exec -it security-context-demo -- sh
```

After logging into the container, we can check which user running the container processes:

```
$ ps -efPID   USER     TIME  COMMAND    1 10000     0:00 sleep 1h    8 10000     0:00 sh   15 10000     0:00 ps -ef$ iduid=10000 gid=3000 groups=2000
```

So we can see that container is running with user 10000 so container configuration overrides pod configuration.

Now let’s check the permissions on the file system. when we change directory to any root filesystem and try to create a file for example, you will not be able to do it as it is mounted as read-only filesystem:

```
$ cd /etc$ touch newfiletouch: newfile: Read-only file system
```

Then we can check our permission on the mounted file system /data/demo:

```
$ cd /data/demo$ touch newfile$ ls -ltrtotal 0-rw-r--r--    1 10000  2000   0 Nov 15 11:20 newfile
```

As you can see that file has been created successfully with the configured UID and GID, which means that through these configurations, user running this container will only have write permission to the mounted file system /data/demo and will not be running as root and will run literally with 1000 user ID

**Conclusion**

In this article, we discussed the security context as concept and as implementation and we demonstrated the risks behind running application in Kubernetes with default configuration or mis-configuration.

Also we demonstrated how to apply security contexts on pod level and container level to guarantee running application with minimum needed permissions.