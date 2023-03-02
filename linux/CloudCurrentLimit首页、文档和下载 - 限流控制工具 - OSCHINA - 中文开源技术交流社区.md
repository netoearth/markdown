CloudCurrentLimit 基于 redis 的 Lua 脚本实现对高并发下的限流控制，可极大降低系统的接入成本，对高并发系统进行限流达到保护系统的目的。

### 开箱即用

引入 maven，配置文件，注解一打，1 分钟集成好限流，真正的做到了开箱即用。

**使用场景:**

**无需独立启动第三方服务，以 jar 包方式和项目一起启动，不存在第三方服务挂掉的风险，对整个集群限流**

如果我们的系统有高并发限流场景，自己临时写可能会有 bug，引入现在市面上的限流工具，要么需要单独启动服务，要么配置繁琐，有没有低成本高效接入的限流组件呢？

显而易见 CloudCurrentLimit 正是很好符合该需求的选择，只需要简单的配置即可达到限流效果.

### springboot 集成版本支持 2.2.2.RELEASE<=Version<3.x

### 实现了集群状态下的限流，限流对整个集群生效

 **1\. 引入 maven 依赖**

```
<dependency>
    <groupId>net.oschina.likaixuan</groupId>
    <artifactId>cloud-limit-spring-boot-starter</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

**2\. application.yml 中开启限流配置**

```
spring:
  redis:
    host: localhost
    database: 0
    lettuce:
      pool:
        max-active: 10
        max-idle: 10
        max-wait: -1ms
      shutdown-timeout: 100ms
    password:
    port: 6379
cloud:
  limit:
    scan-package: com/example/demoplus/controller  #项目中的controller包
    type: TOKEN #目前算法只支持令牌桶算法，写死TOKEN就行
```

**3.Controller 类上打上注解**

```
@CloudLimitAnnottion(
limitCount = 2, #每秒限流请求次数，默认50
limitMsg = "你被限流了" #支持配置自定义限流提示语句，可不配置，不配置默认提示限流【触发限流了】
)
```

举个栗子

![输入图片说明](https://foruda.gitee.com/images/1673338293953394761/15d3a0a6_1382315.png)

**4\. 该步骤一般项目中都有自己的全局捕获异常，只需要捕获 RuntimeException 即可，当然也可以捕获限流的异常类 CloudLimitException**

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = CloudLimitException.class)
    public R handler(CloudLimitException e){
        return R.error(e.getMessage());
    }
}
```

**如果项目启动有限流相关日志输出，则说明限流生效，如下图**

![输入图片说明](https://foruda.gitee.com/images/1673338463183781958/2f4ead7d_1382315.png)