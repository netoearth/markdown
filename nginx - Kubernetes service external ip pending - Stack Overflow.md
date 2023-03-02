


I am trying to deploy nginx on kubernetes, kubernetes version is v1.5.2, I have deployed nginx with 3 replica, YAML file is below,

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: deployment-example
spec:
  replicas: 3
  revisionHistoryLimit: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.10
        ports:
        - containerPort: 80
```

and now I want to expose its port 80 on port 30062 of node, for that I created a service below,

```
kind: Service
apiVersion: v1
metadata:
  name: nginx-ils-service
spec:
  ports:
    - name: http
      port: 80
      nodePort: 30062
  selector:
    app: nginx
  type: LoadBalancer
```

this service is working good as it should be, but it is showing as pending not only on kubernetes dashboard also on terminal. [![Terminal output](https://i.stack.imgur.com/ix5v1.png)](https://i.stack.imgur.com/ix5v1.png)[![Dash board status](https://i.stack.imgur.com/TUMOB.png)](https://i.stack.imgur.com/TUMOB.png)


If it is your private k8s cluster, MetalLB would be a better fit. Below are the steps.

**Step 1: Install MetalLB in your cluster**

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

**Step 2: Configure it by using a configmap**

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.42.42.100-172.42.42.105 #Update this with your Nodes IP range 
```

**Step 3: Create your service to get an external IP (would be a private IP though).**

FYR:

Before MetalLB installation: [![enter image description here](https://i.stack.imgur.com/Unq7M.png)](https://i.stack.imgur.com/Unq7M.png)

After MetalLB installation: [![enter image description here](https://i.stack.imgur.com/2gPaq.png)](https://i.stack.imgur.com/2gPaq.png)

[![enter image description here](https://i.stack.imgur.com/K1M24.png)](https://i.stack.imgur.com/K1M24.png)

answered Jun 18, 2020 at 2:34



[Mohsin](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4979693/mohsin)Mohsin

911 silver badge4 bronze badges

Following @Javier's answer. I have decided to go with "patching up the external IP" for my load balancer.

```
 $ kubectl patch service my-loadbalancer-service-name \
-n lb-service-namespace \
-p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.39.25"]}}'
```

This will replace that 'pending' with a new patched up IP address you can use for your cluster.

For more on this. Please see **karthik's** post on [LoadBalancer support with Minikube for Kubernetes](https://blog.codonomics.com/2019/02/loadbalancer-support-with-minikube-for-k8s.html)

Not the cleanest way to do it. I needed a temporary solution. Hope this helps somebody.

answered Aug 18, 2019 at 15:37

[

![Godfrey's user avatar](https://www.gravatar.com/avatar/2d8c1c0c9cb723cd9175854835384b4e?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4869748/godfrey)

[Godfrey](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4869748/godfrey)Godfrey

91313 silver badges17 bronze badges

1

If you are using minikube then run commands below from terminal,

```
$ minikube ip
$ 172.17.0.2 // then 
$ curl http://172.17.0.2:31245
or simply
$ curl http://$(minikube ip):31245
```

answered Mar 7, 2020 at 22:45

1

In case someone is using MicroK8s: You need a network load balancer.

MicroK8s comes with metallb, you can enable it like this:

```
microk8s enable metallb
```

`<pending>` should turn into an actual IP address then.

answered Sep 21, 2020 at 14:42

[

![Willi Mentzel's user avatar](https://www.gravatar.com/avatar/0ca4c6bc08e59c34de7fc39e09de8dbe?s=64&d=identicon&r=PG)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/1788806/willi-mentzel)

[Willi Mentzel](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/1788806/willi-mentzel)Willi Mentzel

26.7k19 gold badges112 silver badges117 bronze badges

1

A general way to expose an application running on a set of Pods as a network service is called service in Kubernetes. There are four types of service in Kubernetes.

ClusterIP The Service is only reachable from within the cluster.

NodePort You'll be able to communicate the Service from outside the cluster using `NodeIP`:`NodePort`.default node port range is `30000-32767`, and this range can be changed by define `--service-node-port-range` in the time of cluster creation.

LoadBalancer Exposes the Service externally using a cloud provider's load balancer.

ExternalName Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.

Only the LoadBalancer gives value for the External-IP Colum. and it only works if the Kubernetes cluster is able to assign an IP address for that particular service. you can use [metalLB](https://metallb.universe.tf/) load balancer for provision IPs to your load balancer services.

I hope your doubt may go away.

answered Aug 27, 2021 at 7:31

[

![Damith Udayanga's user avatar](https://lh6.googleusercontent.com/-c57x8nYylhg/AAAAAAAAAAI/AAAAAAAAB14/FoS_aUr1J3Q/photo.jpg?sz=64)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/7037949/damith-udayanga)

You can patch the IP of Node where pods are hosted ( Private IP of Node ) , this is the easy workaround .

Taking reference with above posts , Following worked for me :

kubectl patch service my-loadbalancer-service-name \\ -n lb-service-namespace \\ -p '{"spec": {"type": "LoadBalancer", "externalIPs":\["xxx.xxx.xxx.xxx Private IP of Physical Server - Node - where deployment is done "\]}}'

answered Apr 8, 2020 at 5:34

[

![Dheeraj Sharma's user avatar](https://lh3.googleusercontent.com/a-/AOh14Gga0NcqUm0K1NtDFVaVz4HomLYRuKddvc4mh5ea=k-s64)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/13255575/dheeraj-sharma)

Adding a solution for those who encountered this error while running on [amazon-eks](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/questions/tagged/amazon-eks "show questions tagged 'amazon-eks'").

First of all run:

```
kubectl describe svc <service-name>
```

And then review the `events` field in the example output below:

```
Name:                     some-service
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"some-service","namespace":"default"},"spec":{"ports":[{"port":80,...
Selector:                 app=some
Type:                     LoadBalancer
IP:                       10.100.91.19
Port:                     <unset>  80/TCP
TargetPort:               5000/TCP
NodePort:                 <unset>  31022/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason                  Age        From                Message
  ----     ------                  ----       ----                -------
  Normal   EnsuringLoadBalancer    68s  service-controller  Ensuring load balancer
  Warning  SyncLoadBalancerFailed  67s  service-controller  Error syncing load balancer: failed to ensure load balancer: could not find any suitable subnets for creating the ELB
```

Review the error message:

```
Failed to ensure load balancer: could not find any suitable subnets for creating the ELB
```

In my case, the reason that no suitable subnets were provided for creating the ELB were:

**1:** The EKS cluster was deployed on the wrong subnets group - internal subnets instead of public facing.  
(\*) By default, services of type `LoadBalancer` create public-facing load balancers if no `service.beta.kubernetes.io/aws-load-balancer-internal: "true"` annotation was provided).  

**2:** The Subnets weren't tagged according to the requirements mentioned [here](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html#vpc-subnet-tagging).

Tagging VPC with:

```
Key: kubernetes.io/cluster/yourEKSClusterName
Value: shared
```

Tagging public subnets with:

```
Key: kubernetes.io/role/elb
Value: 1
```

answered Jul 26, 2020 at 22:21

[

![RtmY's user avatar](https://www.gravatar.com/avatar/107522343c9ed3b01cdc0d3eece92241?s=64&d=identicon&r=PG)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/1103953/rtmy)

[RtmY](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/1103953/rtmy)RtmY

15.9k10 gold badges105 silver badges111 bronze badges

If you are using a bare metal you need the NodePort type [https://kubernetes.github.io/ingress-nginx/deploy/baremetal/](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)

LoadBalancer works by default in other cloud providers like Digital Ocean, Aws, etc

```
k edit service ingress-nginx-controller
type: NodePort

spec:
   externalIPs:
   - xxx.xxx.xxx.xx 
```

using the public IP

answered Oct 8, 2021 at 12:14

[

![Math's user avatar](https://www.gravatar.com/avatar/132938e1f98b5e92547a901167e705d7?s=64&d=identicon&r=PG)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/992630/math)

[Math](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/992630/math)Math

7101 gold badge7 silver badges17 bronze badges

**Use NodePort:**

```
$ kubectl run user-login --replicas=2 --labels="run=user-login" --image=kingslayerr/teamproject:version2  --port=5000

$ kubectl expose deployment user-login --type=NodePort --name=user-login-service

$ kubectl describe services user-login-service
```

**(Note down the port)**

```
$ kubectl cluster-info
```

**(IP-> Get The IP where master is running)**

Your service is accessible at **(IP):(port)**

[

![PjoterS's user avatar](https://i.stack.imgur.com/KrI3V.png?s=64&g=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/11148139/pjoters)

[PjoterS](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/11148139/pjoters)

12.1k1 gold badge20 silver badges49 bronze badges

answered Dec 7, 2018 at 9:58

[

![Shubham Sawant's user avatar](https://graph.facebook.com/100001531576287/picture?type=large)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4158990/shubham-sawant)

The LoadBalancer ServiceType will only work if the underlying infrastructure supports the automatic creation of Load Balancers and have the respective support in Kubernetes, as is the case with the Google Cloud Platform and AWS. If no such feature is configured, the LoadBalancer IP address field is not populated and still in pending status , and the Service will work the same way as a NodePort type Service

answered Mar 23, 2020 at 21:46

[

![Medone's user avatar](https://www.gravatar.com/avatar/117e7dfc4c78542591bf46f210152b47?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/9503747/medone)

[Medone](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/9503747/medone)Medone

1171 gold badge3 silver badges11 bronze badges

## minikube tunnel

The below solution works in my case.

First of all, try this command:

```
minikube tunnel
```

If it's not working for you. follow the below:

I restart `minikube` container.

```
docker minikube stop 
```

then

```
docker minikube start
```

After that re-run `kubernetes`

```
minikube dashboard
```

After finish execute :

```
 minikube tunnel
```

answered Mar 25, 2022 at 22:15

[

![Abd Abughazaleh's user avatar](https://i.stack.imgur.com/VYp8v.jpg?s=64&g=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/8370334/abd-abughazaleh)

[Abd Abughazaleh](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/8370334/abd-abughazaleh)Abd Abughazaleh

4,0382 gold badges38 silver badges49 bronze badges

I have the same problem. Windows 10 Desktop + Docker Desktop 4.7.1 (77678) + Minikube v1.25.2

Following the [official docs](https://minikube.sigs.k8s.io/docs/start/) on my side, I resolve with:

```
PS C:\WINDOWS\system32> kubectl expose deployment sito-php --type=LoadBalancer --port=8080 --name=servizio-php
service/servizio-php exposed
PS C:\WINDOWS\system32> minikube tunnel
 * Tunnel successfully started

 * NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

 * Starting tunnel for service servizio-php.


PS E:\docker\apache-php> kubectl get service
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes     ClusterIP      10.96.0.1      <none>        443/TCP          33h
servizio-php   LoadBalancer   10.98.218.86   127.0.0.1     8080:30270/TCP   4m39s

```

The open browser on [http://127.0.0.1:8080/](http://127.0.0.1:8080/)

answered May 7, 2022 at 16:50

[

![Max Monterumisi's user avatar](https://lh3.googleusercontent.com/-xKijKstI_A8/AAAAAAAAAAI/AAAAAAAAABg/P4SBvrQuuPE/photo.jpg?sz=64)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/9492903/max-monterumisi)

same issue:

> os>kubectl get svc right-sabertooth-wordpress
> 
> NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S)  
> **right-sabertooth-wordpress** LoadBalancer 10.97.130.7 "pending" 80:30454/TCP,443:30427/TCP
> 
> os>minikube service list
> 
> |-------------|----------------------------|--------------------------------|
> 
> | NAMESPACE | NAME | URL |
> 
> |-------------|----------------------------|--------------------------------|
> 
> | default | kubernetes | No node port |
> 
> | default | right-sabertooth-mariadb | No node port |
> 
> | default | **right-sabertooth-wordpress** | **[http://192.168.99.100:30454](http://192.168.99.100:30454/)** |
> 
> | | | [http://192.168.99.100:30427](http://192.168.99.100:30427/) |
> 
> | kube-system | kube-dns | No node port |
> 
> | kube-system | tiller-deploy | No node port |
> 
> |-------------|----------------------------|--------------------------------|

It is, however,accesible via that [http://192.168.99.100:30454](http://192.168.99.100:30454/).

answered Feb 12, 2019 at 8:28

[

![system programmer's user avatar](https://www.gravatar.com/avatar/d4c8d1fa3442930cf35de7344c08954b?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/10119950/system-programmer)

1

There are three types of exposing your service Nodeport ClusterIP LoadBalancer

When we use a loadbalancer we basically ask our cloud provider to give us a dns which can be accessed online Note not a domain name but a dns.

So loadbalancer type does not work in our local minikube env.

answered Aug 18, 2020 at 5:25

[

![Sunjay Jeffrish's user avatar](https://lh3.googleusercontent.com/a-/AOh14GhPoK1HgEEu-vXZFdOBVhe6GZsuyffQ84Sc4mw2=k-s64)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/13927402/sunjay-jeffrish)

Those who are using **minikube** and trying to access the service of kind `NodePort` or `LoadBalancer`.

> We don’t get the external IP to access the service on the local system. So a good option is to use minikube IP

Use the below command to get the `minikube` IP once your service is exposed.

```
minikube service service-name --url
```

Now use that URL to serve your purpose.

answered Sep 28, 2022 at 22:41

[

![Maheshvirus's user avatar](https://www.gravatar.com/avatar/870c52156790f84bcf6b39d81a27b953?s=64&d=identicon&r=PG)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4615540/maheshvirus)

[Maheshvirus](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4615540/maheshvirus)Maheshvirus

6,0791 gold badge37 silver badges38 bronze badges

Check kube-controller logs. I was able to solve this issue by setting the clusterID tags to the ec2 instance I deployed the cluster on.

answered Jul 30, 2019 at 9:49

[

![Rakesh Pelluri's user avatar](https://www.gravatar.com/avatar/cccf8a7f65eff9221c18a4e8cd578238?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4232045/rakesh-pelluri)

If you are not on a supported cloud (aws, azure, gcloud etc..) you can't use LoadBalancer without MetalLB [https://metallb.universe.tf/](https://metallb.universe.tf/) but it's in beta yet..

answered Nov 7, 2019 at 0:04

[

![Pierluigi Di Lorenzo's user avatar](https://i.stack.imgur.com/ivSpI.png?s=64&g=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/5619290/pierluigi-di-lorenzo)

1

Delete existing service and create a same new service solved my problems. My problems is that the loading balancing IP I defines is used so that external endpoint is pending. When I changed a new load balancing IP it still couldn't work.

Finally, delete existing service and create a new one solved my problem.

[

![pampasman's user avatar](https://www.gravatar.com/avatar/?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/3276944/pampasman)

answered Jul 18, 2019 at 3:18

[

![Shakira Sun's user avatar](https://www.gravatar.com/avatar/51495101791b628c3995391a523a5e61?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/9093635/shakira-sun)

1

For your use case best option is to use NordPort service instead of loadbalancer type because loadbalancer is not available.

answered Sep 26, 2021 at 3:20

[

![deepu kumar singh's user avatar](https://www.gravatar.com/avatar/de80bb5b706cffef9a91058558efadf4?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4604182/deepu-kumar-singh)

I was getting this error on the Docker-desktop. I just exit and turn it on again(Docker-desktop). It took few seconds, then It worked fine.

answered Oct 7, 2021 at 6:34

[

![Dayan's user avatar](https://www.gravatar.com/avatar/b6f07ab672518fb1e67db558352c3b8d?s=64&d=identicon&r=PG&f=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4395420/dayan)

[Dayan](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/4395420/dayan)Dayan

3691 gold badge5 silver badges12 bronze badges

Deleting all older services and creating new resolved my issue. IP was bound to older service. just try "`$kubectl get svc`" and then delete all svc's one by one "`$kubectl delete svc 'svc name'` "

[

![Procrastinator's user avatar](https://i.stack.imgur.com/h0CyY.jpg?s=64&g=1)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/2963422/procrastinator)

answered Jul 25, 2022 at 17:11

[

![Nitin Jagadhane's user avatar](https://lh5.googleusercontent.com/-dbrYdYntizM/AAAAAAAAAAI/AAAAAAAAAAA/AMZuuckZXAZ8f-ufPzKjOWYEPjlToJbUug/photo.jpg?sz=64)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/13830791/nitin-jagadhane)

May be the subnet in which you are deploying your service, have not enough ip's

answered Apr 12, 2021 at 17:07

[

![Hemant yadav's user avatar](https://lh3.googleusercontent.com/a-/AOh14GhpD9Ffnv1hFw3WKrGdek4QdkOQK-ljsj5Bi8jXHA=k-s64)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/15613574/hemant-yadav)

If you are trying to do this in your on-prem cloud, you need an L4LB service to create the LB instances.

Otherwise you end up with the endless "pending" message you described. It is visible in a video here: [https://www.youtube.com/watch?v=p6FYtNpsT1M](https://www.youtube.com/watch?v=p6FYtNpsT1M)

You can use open source tools to solve this problem, the video provides some guidance on how the automation process should work.

answered Nov 16, 2021 at 12:18

[

![Joshua Saul's user avatar](https://graph.facebook.com/592043844/picture?type=large)

](chrome-extension://pcmpcfapbekmbjjkdalcgopdkipoggdi/users/2112808/joshua-saul)

1