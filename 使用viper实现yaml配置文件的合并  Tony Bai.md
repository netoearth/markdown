![](https://tonybai.com/wp-content/uploads/use-viper-to-do-merge-of-yml-configuration-files-1.png)

[本文永久链接](https://tonybai.com/2022/09/20/use-viper-to-do-merge-of-yml-configuration-files) – https://tonybai.com/2022/09/20/use-viper-to-do-merge-of-yml-configuration-files

作为小厂，我们的基础设施还不够完备，项目经理中秋节通知我们的系统近期要上second-to-last stage环境和生产环境，于是从运维人员部署效率方面考量，我们紧急开发了一个一键安装脚本生成工具，这样运维人员便可以利用该工具结合实际目标环境生成一键安装脚本。这个工具的原理十分简单，如下示意图所示：

![](https://tonybai.com/wp-content/uploads/use-viper-to-do-merge-of-yml-configuration-files-2.png)

从上图可以知道，我们的工具是基于模板定制最终的配置与安装脚本的，其中：

-   templates/conf下面是服务配置；
-   templates/manifests下面是服务的[k8s yaml脚本](https://tonybai.com/2019/02/25/introduction-to-yaml-creating-a-kubernetes-deployment/);
-   custom/configure文件存储的是针对templates/conf下面服务配置的定制化配置数据；
-   custom/manifests文件存储的是针对templates/manifests下面k8s yaml的定制化配置数据；
-   templates/install.sh则是安装脚本。

custom目录下的两个存储定制化配置的文件是**与目标环境紧密相关的**。

提到template，Gopher们首先想到的是[Go text/template技术](https://pkg.go.dev/text/template)，利用模板语法编写上面templates目录下的模板配置文件。不过基于text/template就需要我们事先将所有需要定制化的变量都一一识别出来，这个量有些大，且不够灵活。

那我们还可以采用什么技术方案呢？我最终选择了**yaml文件合并(包括覆盖与追加)的方案**，该方案示意图如下：

![](https://tonybai.com/wp-content/uploads/use-viper-to-do-merge-of-yml-configuration-files-3.png)

这个示例包含了覆盖和(追加)合并两种情况，我们首先看一下覆盖。

-   custom/manifests.yml中配置覆盖templates/manifests/\*.yaml的配置

以templates/manifests/a.yml为例，该模板中metadata.name的默认值为default，但运维人员根据目标环境定制了(customizing)custom/manifests.yml文件。在该文件中，a.yml文件名作为key值，然后将要覆盖的配置项的全路径配置到该文件中(这里的全路径为metadata.name)：

```
a.yml：
  metadata:
    name: foo
```

custom/manifests.yml文件中对namespace name的修改值foo将会覆盖原模板中的default，这在最终的xx\_install/manifests/a.yml中会体现出来：

```
// a.yml
apiVersion: v1
kind: Namespace
metadata:
  name: foo
```

-   custom/manifests.yml中配置追加到templates/manifests/\*.yaml配置中

对于原模板文件中没有而custom中新增的配置，会追加到最终生成的配置文件中，以b.yml为例。原模板目录下的b.yml内容如下：

```
// templates/manifests/b.yml
log:
  type: file
  level: 0
  compress: true
```

这里log下仅有三个子配置项：type、level和compress。

而运维在custom/manifests.yml为log增加了其他若干种配置，比如access\_log、error\_log等：

```
// custom/manifests.yml
b.yml:
  log:
    level: 1
    compress: false
    access_log: "access.log"
    error_log: "error.log"
    max_age: 3
    maxbackups: 7
    maxsize: 100
```

这样，除了level、compress会覆盖原模板中的值之外，其余新增的配置都会追加到生成的xx\_install/manifests/b.yml中会体现出来：

```
// b.yml
log:
  type: file
  level: 1
  compress: false
  access_log: "access.log"
  error_log: "error.log"
  max_age: 3
  maxbackups: 7
  maxsize: 100
```

好了！方案确定了，那**如何实现yaml文件的合并呢**？Go社区的yaml包要数https://github.com/go-yaml/yaml(Canonical import paths为gopkg.in/yaml.v2或gopkg.in/yaml.v3)最为知名，这个包实现了[YAML 1.2规范](https://yaml.org/spec/1.2.2/)，可以方便实现Yaml与go struct之间的marshal与unmarshal。不过，yaml包提供的接口都比较初级，要想实现yaml文件的合并，还需要自己做较多额外工作，时间上可能不允许了。那有没有现成可用的工具呢？答案是有的，它就是在Go社区大名鼎鼎的[viper](https://github.com/spf13/viper)！

viper是由gohugo作者、前Go语言项目组产品经理[Steve Francia](https://github.com/spf13/)开发的开源Go应用配置框架。viper不仅支持命令行参数传入配置，还支持从各种类型配置文件、环境变量、远程配置系统(etcd等)等获取配置。除此之外，viper还支持配置文件的merge和对配置文件的写入操作。

我们是否可以直接使用viper的Merge系列操作呢？答案是不能！为什么呢？因为这与我们上面的设计有关。我们将与环境有关的配置都放入了custom/manifests.yml这一个文件中了，这与一merge就会导致custom/manifests.yml中的配置数据出现在每一个最终生成的templates/xx.yml配置文件中。

那我们就自行来实现一套merge(覆盖和追加)操作！

我们先来看驱动merge的main函数:

```
// github.com/bigwhite/experiments/tree/master/yml-merge-using-viper/main.go

var (
    sourceDir string
    dstDir    string
)

func init() {
    flag.StringVar(&sourceDir, "s", "./", "template directory path")
    flag.StringVar(&dstDir, "d", "./k8s-install", "the target directory path in which the generated files are put")
}

func main() {
    var err error
    flag.Parse()

    // create target directory if not exist
    err = os.MkdirAll(dstDir+"/conf", 0775)
    if err != nil {
        fmt.Printf("create %s error: %s\n", dstDir+"/conf", err)
        return
    }

    err = os.MkdirAll(dstDir+"/manifests", 0775)
    if err != nil {
        fmt.Printf("create %s error: %s\n", dstDir+"/manifests", err)
        return
    }

    // override manifests files with same config item in custom/manifests.yml,
    // store the final result to the target directory
    err = mergeManifestsFiles()
    if err != nil {
        fmt.Printf("override and generate manifests files error: %s\n", err)
        return
    }
    fmt.Printf("override and generate manifests files ok\n")
}
```

我们看到main包利用标准库flag包创建了两个命令行参数-s和-d，分别代表存放templates/custom的源路径和存储生成文件的目标路径。进入main函数后，我们首先在目标路径下建立manifests和conf目录用于分别存储相关配置文件（本例中，conf目录下不生成任何文件），然后main函数调用mergeManifestsFiles对源路径下的templates/manifests中的yml文件与custom/manifests.yml进行合并：

```
// github.com/bigwhite/experiments/tree/master/yml-merge-using-viper/main.go

var (
    manifestFiles = []string{
        "a.yml",
        "b.yml",
    }
)

func mergeManifestsFiles() error {
    for _, file := range manifestFiles {
        // check whether the file exist
        srcFile := sourceDir + "/templates/manifests/" + file
        _, err := os.Stat(srcFile)
        if os.IsNotExist(err) {
            fmt.Printf("%s not exist, ignore it\n", srcFile)
            continue
        }

        err = mergeConfig("yml", sourceDir+"/templates/manifests", strings.TrimSuffix(file, ".yml"),
            sourceDir+"/custom", "manifests", dstDir+"/manifests/"+file)
        if err != nil {
            fmt.Println("mergeConfig error: ", err)
            return err
        }
        fmt.Printf("mergeConfig %s ok\n", file)

    }
    return nil
}
```

我们看到mergeManifestsFiles遍历模板文件，并针对每个文件调用一次真正进行yml文件merge的函数mergeConfig：

```
// github.com/bigwhite/experiments/tree/master/yml-merge-using-viper/main.go

func mergeConfig(configType, srcPath, srcFile, overridePath, overrideFile, target string) error {
    v1 := viper.New()
    v1.SetConfigType(configType) // e.g. "yml"
    v1.AddConfigPath(srcPath)    // file directory
    v1.SetConfigName(srcFile)    // filename(without postfix)
    err := v1.ReadInConfig()
    if err != nil {
        return err
    }

    v2 := viper.New()
    v2.SetConfigType(configType)
    v2.AddConfigPath(overridePath)
    v2.SetConfigName(overrideFile)
    err = v2.ReadInConfig()
    if err != nil {
        return err
    }

    overrideKeys := v2.AllKeys()

    // override special keys
    prefixKey := srcFile + "." + configType + "." // e.g "a.yml."
    for _, key := range overrideKeys {
        if !strings.HasPrefix(key, prefixKey) {
            continue
        }

        stripKey := strings.TrimPrefix(key, prefixKey)
        val := v2.Get(key)
        v1.Set(stripKey, val)
    }

    // write the final result after overriding
    return v1.WriteConfigAs(target)
}
```

我们看到：mergeConfig函数针对templates/manifests下的文件和custom下的manifests.yml文件创建了两个viper实例(viper.New())并分别加载各自的配置数据。然后遍历custom下manifests.yml中的key，将符合要求的配置项的值set到代表对templates/manifests下文件的viper实例中，最后我们将merge后的viper实例数据写到目标文件中。

编译运行该生成工具：

```
$make
go build
$./generator
mergeConfig a.yml ok
mergeConfig b.yml ok
override and generate manifests files ok
```

在默认命令行参数的情况下，文件被生成在k8s-install路径下，我们查看一下生成的文件：

```
$cat k8s-install/manifests/a.yml
apiversion: v1
kind: Namespace
metadata:
    name: foo

$cat k8s-install/manifests/b.yml
log:
    access_log: access.log
    compress: false
    error_log: error.log
    level: 1
    max_age: 3
    maxbackups: 7
    maxsize: 100
    type: file
```

我们看到merge的结果与我们预期的一致(字段顺序不一致没关系，这与viper内部存储key-value时使用go map有关，go map的遍历顺序是随机的)。

不过细心的朋友可能会发现一处问题：那就是a.yml中原先的apiVersion在结果文件中变成了小写的apiversion，这会a.yml在提交给k8s时校验失败！

为什么会这样呢？[viper官方给出的说明](https://github.com/spf13/viper#does-viper-support-case-sensitive-keys)如下(机翻)：

```
Viper合并了来自不同来源的配置，其中许多配置是不区分大小写的，或者使用与其他来源不同的大小写（例如，env vars）。为了在使用多个资源时提供最佳体验，我们决定让所有按键不区分大小写。

已经有一些人试图实现大小写敏感，但不幸的是，这不是那么简单的事情。我们可能会在Viper v2中试着实现它。。。。
```

好吧，既然官方说在v2可能支持，但v2又遥遥无期，我们就用viper的fork版本来解决这个问题吧！[开发者lnashier曾因这个大小写问题fork过一份viper代码](https://github.com/spf13/viper/issues/373)并fix了这个问题，虽然比较old(且可能改的不全面)，但能满足我们的要求就行！我们来试试将spf13/viper换为[lnashier/viper](https://github.com/lnashier/viper)，并重新构建和执行generator：

```
$go mod tidy
go: finding module for package github.com/lnashier/viper
go: found github.com/lnashier/viper in github.com/lnashier/viper v0.0.0-20180730210402-cc7336125d12

$make clean
rm -fr generator k8s-install

$make
go build 

$./generator
mergeConfig a.yml ok
mergeConfig b.yml ok
override and generate manifests files ok

$cat k8s-install/manifests/a.yml
apiVersion: v1
kind: Namespace
metadata:
  name: foo

$cat k8s-install/manifests/b.yml
log:
  access_log: access.log
  compress: false
  error_log: error.log
  level: 1
  max_age: 3
  maxbackups: 7
  maxsize: 100
  type: file
```

我们看到更换为lnashier/viper后，a.yml中的apiVersion这个key没有再被改为小写。

这个工具基本可以使用了。但是这个工具是否没有问题了呢？很遗憾不是的！当generator面对下面的两种形式的配置文件时就会生成错误的文件：

```
//c.yml

apiVersion: v1
data:
  .dockerconfigjson: xxxxyyyyyzzz==
kind: Secret
type: kubernetes.io/dockerconfigjson
metadata:
  name: mysecret
  namespace: foo
```

和

```
//d.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
  namespace: foo
data:
  my-nginx.conf: |
    server {
          listen 80;
          client_body_timeout 60000;
          client_max_body_size 1024m;
          send_timeout 60000;
          proxy_headers_hash_bucket_size 1024;
          proxy_headers_hash_max_size 4096;
          proxy_read_timeout 60000;
          location /dashboard {
             proxy_pass http://localhost:8081;
          }
    }
```

这两个问题就比较棘手了，lnashier/viper也无法解决。我也只能fork lnashier/viper到[bigwhite/viper](https://github.com/bigwhite/viper)自己解决这个问题，并且像d.yml这样的配置形式十分特化，不具有通用性，因此bigwhite/viper并不具有通用性，这里就不细说了，有兴趣的朋友可以自行阅读代码(commit diff)来查看解决上述问题的方法。

本文涉及的代码可以从[这里](https://github.com/bigwhite/experiments/tree/master/yml-merge-using-viper)下载。

___

后记：

-   [kustomize](https://github.com/kubernetes-sigs/kustomize)

kustomize是k8s官方工具，它可以让你基于k8s resource模板YAML文件(类似本文的templates/manifests目录下的文件)并结合kustomization.yaml(类似custom/manifests.yaml)为多种目的定制YAML文件，原始的YAML不会进行任何改动。

不过它的目标仅仅是k8s相关的yaml文件，对于我们的业务服务配置可能无能为力。

-   [CUE数据配置语言](https://cuelang.org/)

CUE是这两年流行起来的一种强大的声明性配置语言，它由前Go核心团队成员[Marcel van Lohuizen](https://github.com/mpvl)创建，他曾与人合作创建了Borg配置语言（BCL）–在谷歌用于部署所有应用程序的语言。CUE是谷歌多年编写配置语言经验的结晶，旨在改善开发者的体验，同时避免一些陷阱。它是JSON的超集且还具有额外的功能特性。Docker之父Solomon Hykes的新创业项目[dagger](https://dagger.io/)大量使用CUE，阿里力推的企业云原生应用管理平台[kubevela](https://kubevela.io/)也是CUE的重度用户。

关于如何使用CUE来替代我上述的方案，还待后续深入研究。

___

[“Gopher部落”知识星球](https://wx.zsxq.com/dweb2/index/group/51284458844544)旨在打造一个精品Go学习和进阶社群！高品质首发Go技术文章，“三天”首发阅读权，每年两期Go语言发展现状分析，每天提前1小时阅读到新鲜的Gopher日报，网课、技术专栏、图书内容前瞻，六小时内必答保证等满足你关于Go语言生态的所有需求！2022年，Gopher部落全面改版，将持续分享Go语言与Go应用领域的知识、技巧与实践，并增加诸多互动形式。欢迎大家加入！

![img{512x368}](http://image.tonybai.com/img/tonybai/gopher-tribe-zsxq-small-card.png)  
![img{512x368}](http://image.tonybai.com/img/tonybai/go-programming-from-beginner-to-master-qr.png)

![img{512x368}](http://image.tonybai.com/img/tonybai/go-first-course-banner.png)  
![img{512x368}](http://image.tonybai.com/img/tonybai/imooc-go-column-pgo-with-qr.jpg)

[我爱发短信](https://51smspush.com/)：企业级短信平台定制开发专家 https://51smspush.com/。smspush : 可部署在企业内部的定制化短信平台，三网覆盖，不惧大并发接入，可定制扩展； 短信内容你来定，不再受约束, 接口丰富，支持长短信，签名可选。2020年4月8日，中国三大电信运营商联合发布《5G消息白皮书》，51短信平台也会全新升级到“51商用消息平台”，全面支持5G RCS消息。

著名云主机服务厂商DigitalOcean发布最新的主机计划，入门级Droplet配置升级为：1 core CPU、1G内存、25G高速SSD，价格5$/月。有使用DigitalOcean需求的朋友，可以打开这个[链接地址](https://m.do.co/c/bff6eed92687)：https://m.do.co/c/bff6eed92687 开启你的DO主机之路。

Gopher Daily(Gopher每日新闻)归档仓库 – https://github.com/bigwhite/gopherdaily

我的联系方式：

-   微博：https://weibo.com/bigwhite20xx
-   博客：tonybai.com
-   github: https://github.com/bigwhite

![](http://image.tonybai.com/img/tonybai/iamtonybai-wechat-qr.png)

商务合作方式：撰稿、出书、培训、在线课程、合伙创业、咨询、广告合作。

© 2022, [bigwhite](https://tonybai.com/). 版权所有.

Related posts:

1.  [Rust vs. Go：为什么强强联合会更好](https://tonybai.com/2021/03/15/rust-vs-go-why-they-are-better-together/ "Rust vs. Go：为什么强强联合会更好")
2.  [使用Go开发Kubernetes Operator：基本结构](https://tonybai.com/2022/08/15/developing-kubernetes-operators-in-go-part1/ "使用Go开发Kubernetes Operator：基本结构")
3.  [使用section.key的形式读取ini配置项](https://tonybai.com/2021/07/10/read-ini-config-item-by-passing-section-key/ "使用section.key的形式读取ini配置项")
4.  [Golang程序配置方案小结](https://tonybai.com/2015/07/01/config-solutions-for-golang-app/ "Golang程序配置方案小结")
5.  [一篇文章带你了解Kubernetes安装](https://tonybai.com/2016/10/18/learn-how-to-install-kubernetes-on-ubuntu/ "一篇文章带你了解Kubernetes安装")