> ## 今天叶秋学长带领大家学习云原生系列三：10大K8s应用安全加固技术~
> 
> 本文译自 Top 10 Kubernetes Application Security Hardening Techniques\[1\]。作者：Rory McCune

![](https://img-blog.csdnimg.cn/img_convert/d0ae34ba9a09e98262cce0771093a531.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/27f40ef290f640868635712a33b90008.gif "image.gif")编辑

将应用部署到K8s集群时，开发者面临的主要挑战是如何管理安全风险。快速解决此问题的一个好方法是在开发过程中对应用清单进行安全加固。本文，将介绍10种开发者可以对应用程序应用加固的方法。

以下技术允许在开发过程中测试强化版本，从而降低在生产环境中应用的控件对运行工作负载造成不利影响的风险。此外，没有强制性控制的集群中，比如Pod安全策略，自愿加固可以帮助降低容器突破攻击的风险。

## 一般方法

在编写K8s工作负载清单时，无论是pod对象还是部署daemonset之类的更高级别的东西，清单中都有一个名为securityContext的部分，允许您指定应该应用于工作负载的安全参数。

例如，下面的代码显示了一个更改其功能

![](https://img-blog.csdnimg.cn/img_convert/76f1a7c64c07ba10dc81459c0ba2ce8e.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/06c03a12b4f34914bb1f43d1aee31be4.gif "image.gif")编辑

下面将详细介绍这些不同部分的工作原理，但从这里你可以看到使用的一般结构。

## runAsUser, runAsGroup

默认情况下，Docker容器以root用户的身份运行，从安全角度看这并不理想。虽然对容器内部的访问权限仍有限制，但在过去一年中，出现了多个容器漏洞，只有在容器以root用户身份运行时才能利用这些漏洞，确保所有容器以非root用户身份运行是一个很好的加固步骤。

在基本层面上，在pod清单中配置这个是相当简单的。最好的方法是将security Context中的runAsUser和runAsGroup字段设置为非0值。

![](https://img-blog.csdnimg.cn/img_convert/a1f04d8af88bccfe3cce74220624acd9.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/6179cbbfb01c42759160d0c17c60efcc.gif "image.gif")编辑

然而，在执行此操作时，重要的是要确保容器在以非root用户身份运行时能够正常工作。如果原始容器镜像被设计为以root身份运行，并且有限制性的文件权限，可能会导致应用程序的运行出现问题。

最好的办法是确保在容器的Dockerfile中设置相同的UID/GID组合，以便在整个开发和测试过程中使用该组合运行。可以通过在Dockerfile中设置USER指令来做到这一点。按照上面的示例，此行将设置相同的 UID和GID组合：

![](https://img-blog.csdnimg.cn/img_convert/c3c1ca86f3b2e75b3f080c10202d15d9.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/9e99ba1d22364f2794a2726728b6005f.gif "image.gif")编辑

## Privileged

Docker和类似的容器运行时提供Privileged标志，作为从容器中移除安全隔离的便捷方法。这不应该在应用工作负载中使用，而应该只在完全必要的情况下使用。

一般来说，Linux容器有相当灵活的安全模型，因此如果容器的运行需要特定的权限，则可以添加该权限，而无需使用总括Privileged设置。

在设计容器清单时，关键是在每个清单的 securityContext 中默认将 privileged 设置为 false，这样就可以清楚地看到它应该在没有这些权限的情况下运行。

![](https://img-blog.csdnimg.cn/img_convert/348996ab40df81227bf47f4984bb6c95.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/4ffc34afbda64e7284d6132912a14fd8.gif "image.gif")编辑

## Capabilities

Linux能力是用于为进程提供传统上为root用户保留的一个或多个方面的权限。默认情况下，Docker和其他容器运行时将为容器提供可用能力的子集。

一个好的加固步骤是仅允许应用程序特别需要的能力。如果你的应用程序设计为以非root用户身份运行，那么它根本不需要任何能力。

一般来说，对能力的处理方法应该是首先删除所有的能力，如果你的应用需要这些能力，再把特定的能力加回来。举个例子，如果你需要CHOWN能力，你会有这样的securityContext：

![](https://img-blog.csdnimg.cn/img_convert/4846836bc82d2c849875ed9a6da1f69b.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/27f53b2672344a6599fe09d24572876e.gif "image.gif")编辑

## Read Only Root File system

你可以使用这个设置来利用容器的短暂性。通常，运行中的容器不应该在容器文件系统中存储有关应用程序的任何状态。这是因为它们可能随时被降速并在集群的其他地方创建新版本。

在这种情况下，你可以在工作负载清单中设置readOnlyRootFilesystem标志，这将使容器的根文件系统成为只读。这可能会让那些在发现应用漏洞后试图在容器中安装工具的攻击者感到沮丧。

与此设置有关的一个常见问题是如何处理应用程序进程运行时需要的临时文件。处理这些的最佳方法是在容器中挂载一个 emptyDir 卷，允许文件被写入某个位置，然后在容器被销毁时自动删除。

设置 readOnlyRootFilesystem 是 securityContext 中一个简单的布尔值。

![](https://img-blog.csdnimg.cn/img_convert/179538a05dd15bd5eec1be7ec4615722.png)

 ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/0829aaa2ad9649f1a79009e094a4fe0d.gif "image.gif")编辑

## AllowPrivilegeEscalation

Linux内核公开的另一个安全设置，这通常是一个很好的、低影响的加固选项。此标志控制子进程是否可以获得比其父进程更多的权限，对于在容器中运行的应用进程来说，它们的运行很少需要这样做。

  ![](https://img-blog.csdnimg.cn/ca02ced0e26e4cfcb1fa676f6c117e46.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/f7c130e4feac432d975f4f23fc2377fd.gif "image.gif")编辑

## Seccomp

清单最后一个值得关注的安全层是Seccomp。Seccomp配置文件可以阻止访问可能导致安全风险的特定Linux系统调用。默认情况下，Docker等容器运行时提供了一个系统调用过滤器，可以阻止对一些特定调用的访问。但是，在K8s下运行时，该过滤器在默认情况下是禁用的。

因此，确保重新启用过滤器是对工作负载清单的重要补充。你可以使用运行时的默认配置文件，或者（如AppArmor和SELinux）提供一个自定义的配置文件。

  ![](https://img-blog.csdnimg.cn/6daaf731534c4ebea9dfda5a33baefec.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/c7d2549df98a478db19df36c662697de.gif "image.gif")编辑

在1.19及更高版本中，seccomp过滤器已被集成到securityContext字段中，因此要设置一个pod使用默认的seccomp过滤器，你可以使用下面的方法：

  ![](https://img-blog.csdnimg.cn/1e8213b10ad6427aab3fa0c656968725.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/e5fc515f96874a74b44e25b9f0084d9a.gif "image.gif")编辑

## Resource limits

由于K8s工作负载共享底层节点，因此确保单个容器不能使用节点上的所有资源非常重要，这可能会导致集群中运行的其他容器出现性能问题。在容器层面，可以设置资源限制，指定容器所需的资源数量以及允许的资源数量限制。

一个容器资源请求的示例如下。这不是在 securityContext 中设置的，而是在通用容器规范中设置的。

  ![](https://img-blog.csdnimg.cn/1fa7312790ce43f4862e8e9b5d487012.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/ddc3e5da334f4fc185fd24bddb13779f.gif "image.gif")编辑虽然内存请求和限制是相当直截了当的，但CPU的限制可能不那么明显。它们实际上是以 "毫秒 "为单位的，其中1,000等于一个CPU核心或超线程。因此，在上面的例子中，请求是针对一个核心的25%，而限制是针对一个内核的50%。设计资源限制时要注意的另一件事是，容器运行时超出限制将如何反应。对于CPU来说，进程将受到限制，有效地降低了它的性能。然而，如果超过了内存限制，容器可能会终止进程，所以确保限制符合应用程序在正常操作中可能合理的请求内容非常重要。

## imageTag

Docker风格的容器通常是通过提供镜像名称和标签名称来指定。Docker有一个特殊情况，就是如果没有指定标签，就会使用 "latest "标签。然而，随着镜像注册表的更新，使用的确切的镜像可能会改变。例如，如果一个操作系统有了新的版本，最新的标签可能会改变为新版本。

这种缺乏固定目标的情况下使得指定要在pod中使用的容器镜像时，使用未指定的标签或特别是 "latest "标签是个坏主意。相反，使用一个明确的标签，你可以使用注册表中存在的命名标签，或者使用唯一标识它的SHA-256哈希值来指定一个镜像，来做到这一点。

使用第一个选项，为每个容器指定镜像和标签。这种方法，仍然依赖于维护者不以损害部署的方式修改镜像，因为标签通常是可变的指针，可以被重定向到另一个镜像。

  ![](https://img-blog.csdnimg.cn/9bf4ee0dc4b94abab6796a21fbf29f14.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/48279e7036dd44f1855907573defd8dc.gif "image.gif")编辑如果你指定了SHA-256哈希值，则仅使用与该哈希值特别对应的镜像。但这是一个高维护选项，因为每次修补镜像时都必须更新清单以反映新的哈希值。

在撰写本文时，与上述指定镜像等效的是：

  ![](https://img-blog.csdnimg.cn/e04fd11f8674400bbcff45dac48b575e.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/c41a1a8de72f40a1b25f468d67a7b319.gif "image.gif")编辑

## AppArmor

该选项适用于使用AppArmor的Linux发行版（主要是Debian衍生的发行版本）。AppArmor可以增加一个强制性的安全执行级别，即使其他隔离层失败或被攻击者绕过，也能提供保护。

如果你不指定AppArmor策略，容器运行时的默认值将适用，因此在许多情况下，无需向应用程序清单添加明确的声明。但是，如果你确实想添加自定义的AppArmor配置文件来进一步加固你的容器，则需要注意的是，与大多数其他加固设置不同，它不在securityContext字段中设置。相反，它是通过清单元数据中的自定义注解来完成的（在K8s的未来版本中有一个更改此行为的提案）。

指定的配置文件必须提前放在集群节点上，然后在下面的例子中代替指定。

  ![](https://img-blog.csdnimg.cn/bf8fc7b2bc7a426e9e99142bfeec67bd.png) ![image.gif](https://ucc.alicdn.com/pic/developer-ecology/374255b1b4b24db3b77963e9ed098a46.gif "image.gif")编辑

## SELinux

这个选项适用于使用SELinux的Linux发行版（主要是Red Hat系列）。SELinux的工作方式类似于AppArmor，为进程增加了额外的安全层。

但是，配置策略稍微复些，而且它将取决于容器运行时和主机操作系统的组合是否启用。

与AppArmor一样，创建自定义SELinux策略在安全性更高的环境中可能很有用，但在大多数情况下，使用默认策略将提供有用的额外安全层。

## 总结

创建一个安全的K8s环境有很多方面，从控制平面到集群上运行的应用程序。主动加固用于部署工作负载的K8s清单是这一过程的重要组成部分，如果在开发生命周期的早期完成，可以显著提高安全性并降低漏洞风险。

> ### 引用链接
> 
> \[1\]
> 
> 原文链接: _[https://blog.aquasec.com/kubernetes-hardening-techniques](https://blog.aquasec.com/kubernetes-hardening-techniques)_

## 本期分享到期为止，关注博主不迷路，叶秋学长带你上高速~~