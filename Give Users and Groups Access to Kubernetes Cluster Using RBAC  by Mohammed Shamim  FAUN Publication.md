## **RBAC Overview**

Role-based access control (RBAC) is a way of granting users granular access to Kubernetes API resources. RBAC is a security design that limits access to Kubernetes resources based on the role the user holds.

![](https://miro.medium.com/max/700/1*XYd7d6p3lA14sK_Tqt_J0w.png)

RBAC Overview

## API objects

The RBAC API declares four kinds of Kubernetes objects: _Role_, _ClusterRole_, _RoleBinding,_ and _ClusterRoleBinding_.

## Role and ClusterRole

An RBAC _Role_ or _ClusterRole_ contains rules that represent a set of permissions. Permissions are purely additive (there are no “deny” rules).

**Role —** Role always sets permissions within a particular _namespace.  
_**ClusterRole —** In contrast, ClusterRole always sets permissions for non-namespaced resources.

## RoleBinding and ClusterRoleBinding

A Rolebinding or ClusterRoleBinding grants the permissions defined in a Role or ClusterRole to a user or groups.

In this article, our goal is to give users or a group of users access to our Kubernetes cluster using RBAC.

Let's assume, we have an existing Kubernetes cluster. We need to give users access to a specific _namespace_ within the cluster. See the following image for a better understanding —

![](https://miro.medium.com/max/529/1*mvnczYDqvrSQqY_pujofCQ.png)

User access to a namespace

For achieving this goal, we can primarily divide our task into two halves.

In the first half, we have to create a user (david), who belongs to the _developer_ group. If we want to give the user (david) access to the Kubernetes cluster, he must authenticate with Kubernetes API first. For that, the user must have a **_kubeconfig_** file containing all the credentials and necessary information required. We will call this process as the **authentication** process.

And in the second half, after the user (david) successfully authenticated with the Kubernetes cluster, we have to define what the user (david) can perform within the cluster. We will call this process as the **authorization** process. There are several ways to authorize the user/group to the Kubernetes cluster, but in this article, we will see the most popular and widely used one. Which is called Role-based access control **(RBAC)**.

> Before proceeding further, you can download all the manifest files and scripts related to this article from [**here**](https://github.com/shamimice03/Kubernetes_RBAC)**.**

## _Authentication_

There are several authentication mechanisms, but in this article, we will see authentication using client certificates.

When authenticating through client certificates, the client must first obtain a valid x509 client certificate which the Kubernetes API server will accept as authentication. This usually means that the client certificate must be signed by the cluster CA certificate. Additionally, for generating a _kubeconfig_ file, the user (david) must have a client private key and access to the cluster-CA certificate.

See the following image to understand how we are going to have a cluster CA-signed client certificate and how we will generate the _kubeconfig_ file —

![](https://miro.medium.com/max/700/1*q5VZMXEhy5Ndv3BHMRCJYg.png)

Authentication workflow

● Generate a private key for the user (david) :

```
> openssl genrsa -out <keyname>.key 2048--------------------------------------------------------------------openssl genrsa -out david.key 2048
```

● Generate CSR using the private key with the appropriate username (david), and the group (developer) :

```
> openssl req -new -key <keyname>.key \ -subj "/CN=<common-name>/O=<group-name>" -out <NAME>.csr--------------------------------------------------------------------openssl req -new -key david.key -subj "/CN=david/O=developer" -out david.csr
```

> In real life, a new user might create his key and CSR file by himself and provide you(admin) with the CSR for further process.

● Configure the **_CertificateSigningRequest_** object using `**certificates.k8s.io**` API**_._** The **_CertificateSigningRequest_** allows a client to ask for an X.509 certificate to be issued.

**Template :**

```
apiVersion: certificates.k8s.io/v1kind: CertificateSigningRequestmetadata:  name: <name>spec:  request: <csr-base64>  signerName: kubernetes.io/kube-apiserver-client  expirationSeconds: 8640000   # minimum - 600sec/10min  usages:  - client auth
```

`**spec.request :**` needs to be populated with the previously generated `.csr` file data, but in _base64_ format.

●`**spec.signerName :**` `kubernetes.io/kube-apiserver-client` — Signs certificates that will be honored as client certificates by the API server.

●`**spec.expirationSeconds :**` to request a particular lifetime for the issued certificate. The minimum valid value for this field is `600` seconds.

● `**usages :**` must include `**["client auth"]**`

Now, let's convert previously generated `.csr` files into _base64_ format and save it as an environment variable for future use.

```
> ENV_VARIABLE=$(cat <NAME>.csr | base64 -w 0)--------------------------------------------------------------------CSR_david=$(cat david.csr | base64 -w 0)
```

● Use the _“sed”_ command to replace the placeholder value with the required ones in the above template.

```
> curl <template-link> | sed "s/<name>/<user-name>/ ; s/<csr-base64>/$ENV_VARIABLE/" > <filename>.yaml--------------------------------------------------------------------curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/CertificateSigningRequest-Template.yaml | sed "s/<name>/david/ ; s/<csr-base64>/$CSR_david/" > david_csr.yaml
```

And then, create a **_CertificateSigningRequest_** object using the following command :

```
> kubectl create -f <csr-object-file.yaml>--------------------------------------------------------------------kubectl create -f david_csr.yaml
```

With that, we have asked for an X.509 certificate to be issued by Cluster CA.  
See the following commands for viewing, inspecting, and approving CSR objects created earlier.

```
#view the csrkubectl get csr#inspect csrkubectl describe csr <csr-name> kubectl describe csr david#approve csrkubectl certificate approve <csr-name>kubectl certificate approve david
```

● After the CSR object is approved. We need to extract the approved and signed client certificate from the Kubernetes API.

```
> kubectl get csr <CSR-NAME> -o jsonpath='{.status.certificate}' | base64 --decode > <client-name>.crt--------------------------------------------------------------------kubectl get csr david -o jsonpath='{.status.certificate}' | base64 --decode > david.crt
```

For generating a kubeconfig file, the Cluster-CA certificate is also required. Let’s extract the Cluster-CA certificate from the existing config file :

```
kubectl config view --raw -o jsonpath='{..cluster.certificate-authority-data}' | base64 --decode > ca.crt
```

View the details of the certificates using the following command :

```
openssl x509 -in <certificate-path.crt> -text -noout--------------------------------------------------------------------openssl x509 -in david.crt -text -nooutopenssl x509 -in ca.crt -text -noout
```

> In real life, you (admin) might have to provide client certificate and cluster-CA certificate to the client (user) for generating the kubeconfig file by themselves.

● And finally, Generate the kubeconfig file for the user (david) who belongs to the _developer_ group.

**Template :**

```
#kubeconfig file templateapiVersion: v1kind: Configcurrent-context: <context>clusters:- name: <cluster-name>  cluster:    certificate-authority-data: <ca.crt>    server: <cluster-endpoint>contexts:- name: <context>  context:    cluster: <cluster-name>    user: <user-name>    namespace: <namespace>users:- name: <user-name>  user:    client-certificate-data: <user.crt>    client-key-data: <user.key>
```

We have to convert all keys and certificates to _base64_ format. After then we can add them to the _kubeconfig_ file.

Let's convert the keys and _base64_ format and save them as an environmental variable for future use.

```
CA_CRT=$(cat ca.crt | base64 -w 0)CONTEXT=$(kubectl config current-context)CLUSTER_ENDPOINT=$(kubectl config view -o jsonpath='{.clusters[?(@.name=="'"$CONTEXT"'")].cluster.server}')USER=davidNAMESPACE=productionDAVID_CRT=$(cat david.crt | base64 -w 0)DAVID_KEY=$(cat david.key | base64 -w 0)
```

And finally, use “sed” magic to replace placeholder data with the required ones in the template file shown above.

```
curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/kubeconfig-template.yaml | sed "s#<context>#$CONTEXT# ;s#<cluster-name>#$CONTEXT# ;s#<ca.crt>#$CA_CRT# ;s#<cluster-endpoint>#$CLUSTER_ENDPOINT# ;s#<user-name>#$USER# ;s#<namespace>#$NAMESPACE# ;s#<user.crt>#$DAVID_CRT# ; s#<user.key>#$DAVID_KEY#" > config
```

With that, we are at the end of the authentication process. Provide the _kubeconfig_ file to the corresponding user, so that the user can authenticate with the Kubernetes API server.

**Alternatively**, use the following [script](https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/generate-kubeconfig.sh) for generating a _kubeconfig_ file.

**Script :**

**Execution procedure :**

```
#Download the above script> curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/generate-kubeconfig.sh > generate_kubeconfig.sh#Make the file executable > chmod +x generate_kubeconfig.sh#Execture the script, terminal will ask for the "Username","Group Name" and "Namespace Name". Finally, type "yes" for execution.> bash generate_kubeconfig.sh
```

## Authorization

As discussed earlier, we want to give users of the _developer_ group access to the specific namespace within the Kubernetes cluster. For achieving this goal, we will create a namespace (you can use an existing namespace too). And then we will create a role. Finally, bind the role with the _developer_ group to which the user (david) belongs.

● Create a namespace

```
kubectl create namespace production
```

● Configure a Role

```
#RoleapiVersion: rbac.authorization.k8s.io/v1kind: Rolemetadata:  name: developer-group-role  namespace: productionrules:- apiGroups: [""]  resources: ["*"]  verbs: ["*"]- apiGroups: ["apps"]  resources: ["*"]  verbs: ["*"]
```

The above role grants permission for performing any resources associated with _the core_ and _apps_ API-Group in the _production_ namespace.

● Bind the Role with the _developer_ group which has the user (david):

```
#Bind Role with a GroupapiVersion: rbac.authorization.k8s.io/v1kind: RoleBindingmetadata:  name: developer-group-binding  namespace: productionsubjects:- kind: Group  name: developer  apiGroup: rbac.authorization.k8s.ioroleRef:  kind: Role  name: developer-group-role  apiGroup: rbac.authorization.k8s.io
```

`**subjects:**` Can be a user, group, or service account.  
`**roleRef:**` Attach the role to the subjects.

● Create/inspect the Role and Rolebinings using the following commands:

```
#create the role and rolebindingskubectl create -f https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/Developer-group-RoleAndBinding.yaml#get role and role bindingskubectl get roles -n productionkubectl get rolebindings -n production#inspect the policy of the role kubectl describe role -n production | grep -A20 PolicyRule#inspect the rolebindingskubectl describe rolebindings -n production
```

With that, we are at the end of the authorization part. The new user (david) from the _developer_ group should be able to access the _production_ namespace of the Kubernetes cluster.

## TEST

● Install [**_kubectl_**](https://kubernetes.io/docs/tasks/tools/)  
● Place the earlier created “_config_” file under `**~/.kube/**`directory  
● Check the current user and also check whether; we can access resources in the authorized namespace.

```
host~$ kubectl config get-usersNAMEdavid--------------------------------------------------------------------host~$ kubectl get podsNo resources found in production namespace.--------------------------------------------------------------------host~$  kubectl run nginx --image=nginxpod/nginx created--------------------------------------------------------------------host~$  kubectl get podsNAME    READY   STATUS    RESTARTS   AGEnginx   1/1     Running   0          6m38s
```

● Now check what will happen, if we want to access other resources rather than the authorized resources:

```
host~$ kubectl get nodesError from server (Forbidden): nodes is forbidden: User "david" cannot list resource "nodes" in API group "" at the cluster scope.
```

Let’s assume that a new senior engineer (harry) is employed for the existing _developer_ group. As he is a senior engineer, he should have some extra responsibilities. The organization wants to give him some cluster-level resource permissions. Such as, he will be able to access PV or storage classes resources if required. For granting permissions for cluster-level resources, we have to use _ClusterRole_ and _ClusterRoleBindings_. See the following image for a better understanding —

![](https://miro.medium.com/max/700/1*GqM1egqzXkgKH_uRCBxQ8A.png)

User with role and clusterRole

As the new user (harry) belongs to the existing _developer_ group, so he will inherit all the permissions the _developer_ group has. In addition to that, he will also get permission for accessing two cluster-level resources (PV and Storage Class).

● [Similarly](https://faun.pub/give-users-and-groups-access-to-kubernetes-cluster-using-rbac-b614b6c0b383#9144), generate a _kubeconfig_ file for the new user (harry). With that, the new user (harry) can access the _production_ namespace of the cluster. Because [previously](https://faun.pub/give-users-and-groups-access-to-kubernetes-cluster-using-rbac-b614b6c0b383#714d) we bound the _developer_ group with a _role_, so if we create a new user under the _developer_ group. The user will inherit group permissions.

Now, the next step is to configure ClusterRole and ClusterRoleBindings for the new user (harry) so that he can access cluster-level resources.

● Configure a ClusterRole

```
apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRolemetadata:  name: pv-sc-accessrules:- apiGroups: [""]  resources: ["persistentvolumes"]  verbs: ["*"]- apiGroups: ["storage.k8s.io"]  resources: ["storageclasses"]  verbs: ["*"]
```

● Bind the ClusterRole to the new senior engineer (harry)

```
apiVersion: rbac.authorization.k8s.io/v1kind: ClusterRoleBindingmetadata:  name: pv-sc-bindingsubjects:- kind: User  name: harry    #name is case sensitive              apiGroup: rbac.authorization.k8s.ioroleRef:  kind: ClusterRole  name: pv-sc-access  apiGroup: rbac.authorization.k8s.io
```

● Create/inspect the ClusterRole and ClusterRoleBinings using the following commands:

```
#create the ClusterRole and CluserRoleBindingskubectl create -f https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/ClusterRoleAndBindingsToUser.yaml#get clusterRole and cluserRoleBindingskubectl get clusterrole pv-sc-accesskubectl get clusterrolebindings pv-sc-binding#inspect the policy of the cluster role  kubectl describe clusterrole pv-sc-access | grep -A20 PolicyRule#inspect the cluster role bindingskubectl describe clusterrolebindings pv-sc-binding
```

With that, the new user (harry) will be able to access the _production_ namespace along with two cluster-level resources (_persistent volume_ and _storage classes)_.

## Checking API Access

`kubectl` provides the `auth can-i` subcommand for quickly querying the API authorization layer. The command uses the `SelfSubjectAccessReview` API to determine if the current user can perform a given action, and works regardless of the authorization mode used.

● Check what action can a current user perform :

```
kubectl auth can-i create pod --namespace production
```

> Output can be **“yes or no”** based on what permission the user has.

● Admin can use **_impersonation_** to determine what action other users can perform :

```
> kubectl auth can-i get pod --as david# yes--------------------------------------------------------------------   > kubectl auth can-i create pod --as harry# yes--------------------------------------------------------------------> kubectl auth can-i get pv --as david# no--------------------------------------------------------------------> kubectl auth can-i get pv --as harry# yes 
```

> _If you found this article helpful, please_ **_don't forget_** _to hit the_ **_Clap_** _and_ **_Follow_** _buttons to help me write more articles like this.  
> Thank You 🖤_

## References

![](https://miro.medium.com/max/700/0*wDF73WYLOIF8xwDK.png)

## If this post was helpful, please click the clap 👏 button below a few times to show your support for the author 👇

## 🚀Developers: Learn and grow by keeping up with what matters, [JOIN FAUN.](https://faun.to/8zxxd)