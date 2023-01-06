## SpringBoot 教程

   ![Spring Boot教程](https://www.cainiaojc.com/static/upload/210423/1629260.jpg)

Spring Boot教程提供了Spring Framework的基本和高级概念。我们的Spring Boot教程面向初学者和专业人士。

Spring Boot是一个Spring模块，为Spring框架提供RAD(快速应用程序开发)功能。

我们的Spring Boot教程涵盖了Spring Boot的所有主题，例如功能，项目，maven项目，启动程序项目向导，Spring Initializr，CLI，应用程序，注释，依赖项管理，属性，启动程序，执行器，JPA，JDBC等。

## 什么是Spring Boot

Spring Boot是一个构建在Spring框架顶部的项目。它提供了一种简便，快捷的方式来设置，配置和运行基于Web的简单应用程序。

它是一个Spring模块，提供了 **RAD(快速应用程序开发)**功能。它用于创建独立的基于Spring的应用程序，因为它需要最少的Spring配置，因此可以运行。

![什么是Spring Boot](https://www.cainiaojc.com/static/upload/210423/1629261.png)

简而言之，Spring Boot是 **Spring Framework** 和 **嵌入式服务器**的组合。

在Spring Boot不需要XML配置(部署描述符)。它使用约定优于配置软件设计范例，这意味着可以减少开发人员的工作量。

我们可以使用Spring **STS IDE** 或 **Spring Initializr** 进行开发Spring Boot Java应用程序。

**为什么要使用Spring Boot Framework？**

我们应该使用Spring Boot Framework，因为:

Spring Boot中使用了依赖项注入方法。 它包含强大的数据库事务管理功能。 它简化了与其他Java框架(如JPA/Hibernate ORM，Struts等)的集成。 它减少了应用程序的成本和开发时间。

与Spring Boot框架一起，其他许多Spring姐妹项目也有助于构建满足现代业务需求的应用程序。 Spring姐妹项目如下:

**Spring Data:** 它简化了来自关系数据库和 **NoSQL** 数据库的数据访问。 **Spring Batch:** 它提供了强大的**批处理**处理。 **Spring Security:** 这是一个安全框架，可为应用程序提供强大的**安全性**。 **Spring Social:** 它支持与LinkedIn等**社交网络**集成。 **Spring Integration:** 它是企业集成模式的实现。使用轻量级消息传递和声明性适配器，它有助于与其他**企业应用程序**集成。

## Spring Boot的优点

它创建**独立** Spring应用程序，这些应用程序可以使用Java **\-jar** 启动。 借助不同的**嵌入式** HTTP服务器(例如 **Tomcat，Jetty** 等)，它可以轻松测试Web应用程序。我们不需要部署WAR文件。 它提供了有用的' **starter** 'POM，以简化我们的Maven配置。 它提供了**production-ready**功能，例如**metrics, health checks**和**externalized configuration.**。 不需要 **XML** 配置。 它提供了一个用于开发和测试Spring Boot应用程序的 **CLI** 工具。 它提供了许多**插件**。 它还最大限度地减少了编写多个**样板代码**(必须在几乎没有任何改动的情况下将其包含在许多地方)，XML配置和注释的情况。 它**提高生产力**并减少开发时间。

## Spring Boot的限制

Spring Boot可以使用应用程序中不会使用的依赖项。这些依赖性增加了应用程序的大小。

## Spring Boot的目标

Spring Boot的主要目标是减少 **开发，单元测试**和 **集成测试**时间。

提供有目的的开发方法 避免定义更多的注释配置 避免编写大量导入语句 避免XML配置。

通过提供或避免上述几点，Spring Boot Framework减少了 **开发时间，开发人员工作量**并 **提高了生产力**。

## Spring Boot的先决条件

要创建Spring Boot应用程序，必须满足以下先决条件。在本教程中，我们将使用 **Spring Tool Suite** (STS)IDE。

Java 1.8 Maven 3.0 + Spring Framework 5.0.0.BUILD-SNAPSHOT 建议使用IDE(Spring工具套件)。

## Spring Boot功能

Web开发 SpringApplication 应用程序事件和侦听器 应用管理 外部配置 属性文件 YAML支持 类型安全配置 日志 安全性

**Web开发**

这是用于Web应用程序开发的非常适合的Spring模块。我们可以轻松创建一个独立的HTTP应用程序，该应用程序使用 **Tomcat，Jetty** 或Undertow等嵌入式服务器。我们可以使用 **spring-boot-starter-web** 模块快速启动和运行应用程序。

**SpringApplication**

SpringApplication是一个类，提供了一种方便的方式来引导Spring应用程序。可以从main方法开始。我们可以仅通过调用静态run()方法来调用应用程序。

示例

```
public static void main(String[] args)
{  
    SpringApplication.run(ClassName.class, args);  
}
```

**应用程序事件和侦听器**

Spring Boot使用事件来处理各种任务。它允许我们创建用于添加侦听器的工厂文件。我们可以使用 **ApplicationListener键**来引用它。

总是在META-INF文件夹中创建工厂文件，例如 **META-INF/spring.factories** 。

**应用管理**

Spring Boot提供了为应用程序启用与管理员相关的功能的功能。它用于远程访问和管理应用程序。我们可以使用 **spring.application.admin.enabled** 属性在Spring Boot应用程序中启用它。

**外部配置**

Spring Boot允许我们外部化我们的配置，以便我们可以在不同环境中使用同一应用程序。该应用程序使用YAML文件来外部化配置。

**属性文件**

Spring Boot提供了一组丰富的 **应用程序属性**。因此，我们可以在项目的属性文件中使用它。该属性文件用于设置诸如 **server-port = 8082** 等属性。它有助于组织应用程序属性。

**YAML支持**

它提供了一种方便的方法来指定层次结构。它是JSON的超集。 SpringApplication类自动支持YAML。它是属性文件的代替方法。

**类型安全配置**

强大的类型安全配置用于管理和验证应用程序的配置。应用程序配置始终是至关重要的任务，应该是类型安全的。我们还可以使用此库提供的注释。

**日志**

Spring Boot对所有内部记录都使用通用记录。默认情况下管理日志记录依赖项。如果不需要自定义，我们不应更改日志记录依赖项。

**安全性**

Spring Boot应用程序是spring的Web应用程序。因此，默认情况下，通过所有HTTP端点上的基本身份验证，它是安全的。可以使用一组丰富的端点来开发安全的Spring Boot应用程序。

___