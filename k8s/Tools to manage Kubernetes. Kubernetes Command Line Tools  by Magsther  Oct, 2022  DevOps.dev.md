## Kubernetes Command Line Tools

In this post, we will look at different Kubernetes Command Line tools that can help us to manage our Kubernetes cluster.

## [Kubectl](https://kubernetes.io/docs/reference/kubectl/)

We all know the `kubectl` tool. It’s the command-line tool provided by Kubernetes that allows you to run commands against Kubernetes clusters.

_You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. kubectl is installable on a variety of Linux platforms, macOS and Windows._

## [K9s](https://github.com/derailed/k9s)

_K9s provides a terminal UI to interact with your Kubernetes clusters. The aim of this project is to make it easier to navigate, observe and manage your applications in the wild. K9s continually watches Kubernetes for changes and offers subsequent commands to interact with your observed resources._

![](https://miro.medium.com/max/842/1*OM6XjTBYmNm9hDREdyBRWA.png)

Install with `brew install k9s` (on MacOS) for other platforms see this [page](https://github.com/derailed/k9s#installation).

![](https://miro.medium.com/max/842/1*fXbRNKUkbCLHo_cWWlw1Ww.png)

Check the Key bindings in the [docs](https://k9scli.io/topics/commands/) to learn more how to manage the cluster.

## [kube-shell](https://github.com/cloudnativelabs/kube-shell)

_An integrated shell for working with the Kubernetes CLI_

Install it using `pip install kube-shell`

Then run `kube-shell`to bring up shell.

![](https://miro.medium.com/max/842/1*cQZbFh6AmkoybeDhWv11HQ.png)

## [kubens](https://github.com/ahmetb/kubectx)

**_kubens_** _is a tool to switch between Kubernetes namespaces._

Install with `brew install kubectx`(on MacOS) for other platforms see this [page](https://github.com/ahmetb/kubectx#installation).

The tools is super simple and super useful!

![](https://miro.medium.com/max/842/1*IownL9sejUHJOMFaCsqEgA.png)

## [kubespy](https://github.com/pulumi/kubespy)

`**_kubespy_**` **_is a small tool that makes it easy to observe how Kubernetes resources change in real time._** _Run_ `_kubespy_` _at any point in time, and it will watch and report information about a Kubernetes resource continuously until you kill it._

Install with `brew install kubespy`(on MacOS) for other platforms see this [page](https://github.com/pulumi/kubespy#installation).

`_kubespy trace deployment nginx_` _will "trace" the complex changes a complex Kubernetes resource makes in the cluster (in this case, a_ `_Deployment_` _called_ `_nginx_`_), and aggregate them into a high-level summary, which is updated in real time._

![](https://miro.medium.com/max/842/1*maQAbhiGY951KyH3OhWvXQ.png)

## [kube-prompt](https://github.com/c-bata/kube-prompt)

_An interactive Kubernetes client featuring auto-complete._ A similar tool is kube-shell (see above).

Install with `brew install c-bata/kube-prompt/kube-prompt` (on MacOS) for for other platforms see this [page](https://github.com/c-bata/kube-prompt#installation).

kube-prompt accepts the same commands as the kubectl, except you don’t need to provide the `kubectl` prefix.

```
get podsNAME                    READY   STATUS    RESTARTS   AGEnginx-8f458dc5b-shth5   1/1     Running   0          2m54snginx-8f458dc5b-tp6zw   1/1     Running   0          2m54snginx-8f458dc5b-xnh72   1/1     Running   0          2m54s
```

## [kube-ops-view](https://codeberg.org/hjacobs/kube-ops-view)

_Kubernetes Operational View — read-only system dashboard for multiple K8s clusters_

**Installation on Kubernetes**

```
git clone https://codeberg.org/hjacobs/kube-ops-view.gitcd kube-ops-viewkubectl apply -k deploy
```

After a little while the pods are up and running.

![](https://miro.medium.com/max/842/1*m-6daEXl___h8zd_fnAKhQ.png)

Open **kube-ops-view** via kubectl port-forward:

```
kubectl port-forward service/kube-ops-view 8080:80
```

Now direct your browser to [http://localhost:8080/](http://localhost:8080/)

![](https://miro.medium.com/max/842/1*jgSClGqCtuXbhGkR9uw2WQ.png)

Read more about this amazing tool [here](https://codeberg.org/hjacobs/kube-ops-view).

## [Kubetail](https://github.com/johanhaleby/kubetail)

_Kubetail enables you to aggregate (tail/follow) logs from multiple pods into one stream. This is the same as running “kubectl logs -f “ but for multiple pods._

Install with `brew tap johanhaleby/kubetail && brew install kubetail` (on MacOS) for for other platforms see this [page](https://github.com/johanhaleby/kubetail#installation).

First find the names of all your pods:

`kubectl get pods`

```
NAME                    READY   STATUS    RESTARTS   AGEnginx-8f458dc5b-shth5   1/1     Running   0          27mnginx-8f458dc5b-tp6zw   1/1     Running   0          27mnginx-8f458dc5b-xnh72   1/1     Running   0          27m
```

To tail the logs of the three **nginx** pods in one go simply do:

`kubetail nginx`

![](https://miro.medium.com/max/842/1*zB_wc1PeFy2_R8fXikoEnw.png)

For more examples, see this [page](https://github.com/johanhaleby/kubetail#usage).

## [Stern](https://github.com/wercker/stern)

_Stern allows you to_ `_tail_` _multiple pods on Kubernetes and multiple containers within the pod. Each result is color coded for quicker debugging._

Install with `brew install stern` (on MacOS) for for other platforms see this [page](https://github.com/wercker/stern).

Let’s say that you have three pods running like this:

`kubectl get pods`

```
NAME                    READY   STATUS    RESTARTS   AGEnginx-8f458dc5b-shth5   1/1     Running   0          27mnginx-8f458dc5b-tp6zw   1/1     Running   0          27mnginx-8f458dc5b-xnh72   1/1     Running   0          27m
```

To follow the logs of the three **nginx** pods in one go simply do:

`stern nginx`

![](https://miro.medium.com/max/842/1*qZkogvP9Lx4gIOxXXQzrOw.png)

For more examples, see this [page](https://github.com/wercker/stern#examples).

To find more Kubernetes tools, check the [Awesome Kubernetes Resources](https://github.com/tomhuang12/awesome-k8s-resources) page, which is a curated list of awesome Kubernetes tools and resources.

## Conclusion

In this post we tried out different Kubernetes command-line tools that can help us to manage our clusters.