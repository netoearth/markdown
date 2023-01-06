_作者：十眠_

## 背景

微服务的稳定性一直是开发者非常关注的话题。随着业务从单体架构向分布式架构演进以及部署方式的变化，服务之间的依赖关系变得越来越复杂，业务系统也面临着巨大的高可用挑战。疫情期间，大家可能都经历过以下的场景：

-   线上预约购买口罩时瞬间洪峰流量导致系统超出最大负载，load 飙高，用户无法下单；
-   在线选课时同一时刻提交选课的请求过多，系统无法响应；
-   在线办公 / 教学时同时在线会议的用户过多，会议比较卡； 

这些可用性下降的场景会严重影响用户体验，所以我们需要预先通过一些手段来提前对不稳定的因素进行防护，同时在突发流量的情况下我们也要具备快速止损的能力。

### 流控降级 - 保障微服务稳定性重要的一环

影响微服务可用性的因素有非常多，而这些不稳定的场景可能会导致严重后果。我们从微服务流量的视角来看，可以粗略分为两类常见的场景：

1.  服务自身流量超过承载能力导致不可用。比如激增流量、批量任务投递导致服务负载飙高，无法正常处理请求。

流量是非常随机性的、不可预测的。前一秒可能还风平浪静，后一秒可能就出现流量洪峰了（例如双十一零点的场景）。然而我们系统的容量总是有限的，如果突然而来的流量超过了系统的承受能力，就可能会导致请求处理不过来，堆积的请求处理缓慢，CPU/Load 飙高，最后导致系统崩溃。因此，我们需要针对这种突发的流量来进行限制，在尽可能处理请求的同时来保障服务不被打垮。

![1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c937e0c3bfa04a9ab9955d9de967ebc2~tplv-k3u1fbpfcp-zoom-1.image "1.png")

2.  服务因依赖其他不可用服务，导致自身连环不可用。比如我们的服务可能依赖好几个第三方服务，假设某个支付服务出现异常，调用非常慢，而调用端又没有有效地进行预防与处理，则调用端的线程池会被占满，影响服务自身正常运转。在分布式系统中，调用关系是网状的、错综复杂的，某个服务出现故障可能会导致级联反应，导致整个链路不可用。

一个服务常常会调用别的模块，可能是另外的一个远程服务、数据库，或者第三方 API 等。例如，支付的时候，可能需要远程调用银联提供的 API；查询某个商品的价格，可能需要进行数据库查询。然而，这个被依赖服务的稳定性是不能保证的。如果依赖的服务出现了不稳定的情况，请求的响应时间变长，那么调用服务的方法的响应时间也会变长，线程会产生堆积，最终可能耗尽业务自身的线程池，服务本身也变得不可用。现代微服务架构都是分布式的，由非常多的服务组成。不同服务之间相互调用，组成复杂的调用链路。以上的问题在链路调用中会产生放大的效果。复杂链路上的某一环不稳定，就可能会层层级联，最终导致整个链路都不可用。因此我们需要对不稳定的服务进行熔断降级，暂时切断不稳定调用，避免局部不稳定因素导致整体的雪崩。

![2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aa97e4fe4d1464395fa7f84e141d1a8~tplv-k3u1fbpfcp-zoom-1.image "2.png")

MSE 服务治理基于阿里限流降级组件 Sentinel 的稳定性防护能力，以流量为切入点，从流量控制、并发控制、熔断降级、热点防护、系统自适应保护等多个维度来帮助保障服务的稳定性，覆盖微服务、云原生网关、Service Mesh 等几大场景。

介绍完流控降级的场景与能力之后，下面讲请出我们今天要重点介绍的主人公：运行时动态 Enhance 能力。我们将介绍如何通过 MSE 服务治理一键实现任意点位的流控降级，任意点位包含但不限于 Web、Rpc、SQL、Redis 等访问接口、任意编写的业务方法、框架的接口等等。

## 运行时 Enhance 能力 - 一键实现任意点位的流控降级

如何在运行时，给任意指定的方法增加一个流控降级能力呢？下面我将以一个 Demo 为例简单介绍。我们编写了如下一个业务代码，我们编写了一个简单的 Spring Boot 应用，其中 a 方法是一个随意编写的内部方法。

```
@SpringBootApplication
public class AApplication {

    public static void main(String[] args) {
        SpringApplication.run(AApplication.class, args);
    }

    @Api(value = "/", tags = {"入口应用"})
    @RestController
    class AController {
        ...
    @ApiOperation(value = "HTTP 全链路灰度入口", tags = {"入口应用"})
        @GetMapping("/a")
        public String restA(HttpServletRequest request) {
            return a(request);
        }
        
        private String a(HttpServletRequest request) {
            StringBuilder headerSb = new StringBuilder();
            Enumeration<String> enumeration = request.getHeaderNames();
            while (enumeration.hasMoreElements()) {
                String headerName = enumeration.nextElement();
                Enumeration<String> val = request.getHeaders(headerName);
                while (val.hasMoreElements()) {
                    String headerVal = val.nextElement();
                    headerSb.append(headerName + ":" + headerVal + ",");
                }
            }
            return "A"+SERVICE_TAG+"[" + inetUtils.findFirstNonLoopbackAddress().getHostAddress() + "]" + " -> " +
                    restTemplate.getForObject("http://sc-B/b", String.class);
        }
        ...
    }
}
```

到目前为止监控是看不到 a 方法的，我们只能看到 restA 的接口或者说是 GET:/a 的监控数据，并且可以对其配置限流降级规则。

![3.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f58fbdd82551416097236cc412660e4a~tplv-k3u1fbpfcp-zoom-1.image "3.png")

开源的方式我们需要在代码中增加 Sentinel 的依赖，并且对 com.alibabacloud.mse.demo.AApplication.AController#a 方法配置注解或者编码方式增加 Sentinel 能力

```
// 注解方式进行埋点，注解方式受 AOP 代理的诸多限制
@SentinelResource("com.alibabacloud.mse.demo.AApplication.AController:a")
private String a(HttpServletRequest request) {
    StringBuilder headerSb = new StringBuilder();
    Enumeration<String> enumeration = request.getHeaderNames();
    while (enumeration.hasMoreElements()) {
        String headerName = enumeration.nextElement();
        Enumeration<String> val = request.getHeaders(headerName);
        while (val.hasMoreElements()) {
            String headerVal = val.nextElement();
            headerSb.append(headerName + ":" + headerVal + ",");
        }
    }
    return "A"+SERVICE_TAG+"[" + inetUtils.findFirstNonLoopbackAddress().getHostAddress() + "]" + " -> " +
            restTemplate.getForObject("http://sc-B/b", String.class);
}

// SDK 方式增加流控降级能力，需要侵入业务代码
private String a(HttpServletRequest request) {
    Entry entry = null;
    try {
        entry = SphU.entry("HelloWorld");
        
        StringBuilder headerSb = new StringBuilder();
        Enumeration<String> enumeration = request.getHeaderNames();
        while (enumeration.hasMoreElements()) {
            String headerName = enumeration.nextElement();
            Enumeration<String> val = request.getHeaders(headerName);
            while (val.hasMoreElements()) {
                String headerVal = val.nextElement();
                headerSb.append(headerName + ":" + headerVal + ",");
            }
        }
        return "A"+SERVICE_TAG+"[" + inetUtils.findFirstNonLoopbackAddress().getHostAddress() + "]" + " -> " +
                restTemplate.getForObject("http://sc-B/b", String.class);
    } catch (BlockException ex) {
      System.err.println("blocked!");
    } finally {
        if (entry != null) {
            entry.exit();
        }
    }
}
```

需要编码那就自然会有许多的弊端，要增加依赖要改代码，要重新发布，难以做到即上即下... 到处都是成本。

那么我们如何可以不编写一行代码，就可以做到对 com.alibabacloud.mse.demo.AApplication.AController#a 的限流降级能力呢？

### 配置运行时白屏化规则

配置运行时白屏化规则，并选择当前应用的自定义埋点类型的接口，并填入类与方法。

![4.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff6708fb11e64f8fa583022c784d0462~tplv-k3u1fbpfcp-zoom-1.image "4.png")

当然可以看到，我们白屏化规则能力不仅仅支持动态限流降级，还支持任意点位的访问日志以及请求上下文的收集

![5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82a979c9bd1a402dbf5ad45189559424~tplv-k3u1fbpfcp-zoom-1.image "5.png")

### 观察到指定方法的监控数据

我们在应用治理找到目标应用，在接口监控 > 自定义埋点中看到指定方法 com.alibabacloud.mse.demo.AApplication.AController#a 的监控数据

![6.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa2129e8f56941e48cf74f205193ad0d~tplv-k3u1fbpfcp-zoom-1.image "6.png")

### 配置流控规则

我们可以点击接口概览右上角的 “新增防护规则” 按钮，添加一条流控规则：

![7.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5597cb99a45a4f4dbea8b51682787059~tplv-k3u1fbpfcp-zoom-1.image "7.png")

我们可以配置最简单的 QPS 模式的流控规则，比如上面的例子即限制该接口每秒单机调用量不超过 1 次。

配置规则后，稍等片刻即可在监控页面看到限流效果：

![8.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/359ddb34828e42c6abbb8f40809eea7a~tplv-k3u1fbpfcp-zoom-1.image "8.png")

被拒绝的流量也会返回错误信息。MSE 自带的框架埋点都有默认的流控处理逻辑，如 Web 接口被限流后返回 429 Too Many Requests，DAO 层、java 方法被限流后抛出异常等。

## 总结

我们将运行时白屏化能力抽象为如下规则：WhiteScreenRule = Taget + Action\*\*\*\*

![9.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cc46e737f574168bf18faaeb540f13d~tplv-k3u1fbpfcp-zoom-1.image "9.png")

**Target:**

-   ResourceTarget: 目标接口，支持 Web、Rpc、SQL 以及任意的自定义方法
-   WorkloadTarget: 目标实例，可以选择所有机器或指定机器 IP
-   TrafficCondition: 是否仅针对异常、慢调用、全链路灰度标签 

**Action:**

-   相关上下文诊断信息的收集，参数、返回值、线程上下文、Target 对象、类加载器信息等
-   后续链路是否日志打印
-   进行限流降级
-   指定流量进行打标染色（规划中） 

近期 MSE 将推出基于上述规则的模型结合动态 Enhance 能力的日志治理，我们不仅仅有基于动态 Enhance 能力的任意点位的限流降级，还可以帮助我们洞察全链路流量运行的行为，并做出实时的治理与保护。

![10.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf3d450b9b7247ffbc04f238141b6d81~tplv-k3u1fbpfcp-zoom-1.image "10.png")

MSE Sentinel 不仅在阿里内部淘宝、天猫等电商领域有着广泛的应用，在互联网金融、在线教育、游戏、直播行业和其他大型政央企行业也有着大量的实践。有了针对任何方法都可以做到限流降级的能力后，我们可以快速赋予任意一个微服务系统具备流量防护的能力，让我们有更多的时间专注于业务的快速发展，关于系统的稳定性就放心地交给 MSE ，让专业的团队做专业的事情。

MSE 云原生网关预付费、MSE 注册配置预付费首购 8 折，首购 1 年及以上 7 折。点击[此处](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.aliyun.com%2Fproduct%2Faliware%2Fmse)，查看更多详情～