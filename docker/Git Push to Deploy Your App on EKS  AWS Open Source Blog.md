![Gitkube + EKS logos](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2018/08/30/gitkube-plus-eks.jpg)

It’s a joyful experience for a developer when you can focus just on building an application, and rely on a tool to do the grunt work of pushing that application to a Kubernetes cluster. When the tool makes use of your existing knowledge of git, it only adds to the pleasure! In this post, Tirumarai Selvan of [Hasura](https://hasura.io/) explains an open source project, Gitkube, that provides a seamless experience for deploying your applications to an [Amazon Elastic Container Service for Kubernetes](https://aws.amazon.com/eks/) (EKS) cluster using git semantics. Installation steps and a deployment pipeline for a simple example are explained in detail.

– [Arun](http://twitter.com/arungupta)

___

[Gitkube](https://github.com/hasura/gitkube) is a tool used to deploy applications on a Kubernetes cluster by simply running a `git push`. In other words, you can go from your source code to a deployed application on a Kubernetes cluster in 60 seconds. This is the experience of `git push to deploy`, brought to your Kubernetes cluster.

Gitkube’s benefits include:

1.  Very thin clients: The only thing you need to have installed on your machines is git.
2.  Hardened security controls: Gitkube runs the build and deploy processes in the cluster, which can be managed according to your organizational security policies using RBAC and IAM.
3.  Kubernetes native: It builds on the Kubernetes CRD and operator pattern to provide a reference implementation for git-based automation, aka GitOps. It’s easy to extend for your custom needs.

With over 2k stars on Github, Gitkube has proven to be a powerful tool that developers use for deploying their apps on Kubernetes.

## Gitkube and EKS

You can get started with Gitkube on your EKS cluster in minutes. Gitkube runs on EKS as-is; no additional steps or external configurations are required to get started.

Here is a short video showing how to deploy your code on a bare EKS cluster using Gitkube:

<iframe loading="lazy" title="Gitkube on Amazon EKS" width="500" height="375" src="https://www.youtube-nocookie.com/embed/hBOgxW-MNDQ?feature=oembed" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen="" sandbox="allow-scripts allow-same-origin"></iframe>

Now let’s look at the steps in detail.

## Getting Started

### Set Up an EKS Cluster

Set up an EKS cluster by following the instructions in [Getting Started with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html).

Or [use `eksctl` to get started very quickly with a basic cluster](https://aws.amazon.com/blogs/opensource/eksctl-eks-cluster-one-command/).

## Configure kubectl

**1\. Install kubectl**

Make sure you have `kubectl` installed with version at least 1.10:

If not, [install the latest kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

**2\. Install heptio-authenticator**

EKS uses IAM to manage identities on your cluster. You need to install the `heptio-authenticator` so that kubectl can authenticate to your cluster via IAM.

Your `ekskubeconfig` is already configured to use `heptio-authenticator`, so you can start using kubectl right away.  
**3\. Verify kubectl access**

## Install Gitkube

Assuming you have set your kubeconfig to point to your EKS cluster, follow these instructions to install Gitkube on it.

**1\. Install gitkube-cli**

Gitkube comes with a cli which can be used to automate many steps into a few commands. You can also use Gitkube without the cli, but in this example we will use the cli.

**2\. Install Gitkube**

The setup is now complete, you should see gitkube pods running in the `kube-system` namespace. You will also see a `gitkubed` service of type LoadBalancer in the `kube-system` namespace.

## Push Your Application

We will now describe a typical workflow for Gitkube in action.

**1\. Start with a git repo**

Your application must live in a git repo and include `Dockerfile(s)` to build the container image(s). Let’s start with an example repo which includes a simple nginx application:

This repo has examples with different styles of code organization. For example, you can have your application in a single repo including the config files, or you can have a multi-repo organization with code and config kept separately. In the following demo, we will use the mono-repo example.

**2\. Generate a Remote spec**

Gitkube is configured using the concept of `Remotes`. A `Remote` is a Custom Resource in Kubernetes. Each `Remote` specifies the parameters used to manage the application. Parameters include authentication users for git push access, build definitions, image registry, and runtime arguments. [The complete spec for a Remote is documented in the Gitkube repo](https://github.com/hasura/gitkube/blob/master/docs/remote.md).

You can quickly generate a Remote spec interactively using the gitkube-cli. Here's a sample run:

This outputs the Remote spec into a `remote.yaml` file in the working directory. You may edit the the file to further configure the remote, for instance to [add a Docker registry of your choice](https://github.com/hasura/gitkube/blob/master/docs/registry.md#elastic-container-registry).

**3\. Create Remote**

Use the generated `remote.yaml` file to create the Remote.

This outputs the git remote url to which you will push your branches.

**4\. Add git remote**

Add the git remote from the previous step.

Now, you are ready to deploy your app.

**5\. Commit and push**

Make any changes to the code, commit, and push it.

On push, the Gitkube pipeline kicks in and does a build and deploy of your application to the cluster.

You can see your application live using kubectl proxy at [http://localhost:8080/api/v1/namespaces/default/services/www/proxy/](http://localhost:8080/api/v1/namespaces/default/services/www/proxy/)

![Hello Gitkube on EKS screen](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2018/08/31/hello-gitkube.jpg)

**6\. What Next?**

Just keep pushing your branch, and see your application go live everytime!

## Final Remarks

This is just an example workflow for using Gitkube to deploy your app on EKS. You can use similar workflows with multiple repository configurations, such as:

-   Single repo including the config (K8s yamls or helm chart)
-   Multi repo with code and config separate

Check out [the example repo](https://github.com/hasura/gitkube-example) for more walkthroughs.

To conclude: start git push-ing your apps to EKS using Gitkube today!

![Tirumarai Selvan](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2018/08/30/Tirumarai-150x150.jpg)

### Tirumarai Selvan

Tirumarai Selvan is a senior engineer at Hasura and co-author of Gitkube. At Hasura, he works on all things Kubernetes. When not actively coding, he spends his time figuring out new paradigms of computing like Serverless and GraphQL.

_The content and opinions in this post are those of the third-party author and AWS is not responsible for the content or accuracy of this post._