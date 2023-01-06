企业主流框架用什么，也不能一概而论。需要有所划分，市场需求可以分为两种。其一是**传统企业开发**，其二是**互联网企业开发**。

常用技术有：[struts1](https://www.zhihu.com/search?q=struts1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A868382281%7D)/2,Spring/mvc/boot/cloud,Hibernate/MyBatis。在此论述的是JavaEE，JavaME另有他论。技术核心原理、理论知识、优点缺点、功能作用等就不再展开描述。

一、传统企业开发要求安全、稳定，易于维护。常用于政府机关、企业OA、ERP等管理系统，对于此类型系统常用框架是Struts1/2+Spring+Hibernate/MyBatis/Ibatis/Servlet，至今统计很多政府系统多用老框架，不易更新扩展。为便于维护与新系统融合，所以常采用稳定的系统架构技术。也有部分采用Spring mvc，迭代旧的系统。

常有组合有：Struts + Spring+ Mybatis，Spring MVC + Hibernate，Spring mvc+MyBatis,Struts1/2Hibernate

二、互联网企业开发要求高并发、高用性、易扩展。常用于科技企业，交互系统、支付系统、购物系统等。而针对这些系统老技术框架不易于开发实现分布式、版本更新、扩展，近几年出现不少分布式技术。

常使用spring boot,spring cloud的套件组装，拆分各子业务系统，对核心业务服务进行解耦划分，可实现灵活组装，极大提升业务可复用性、拆分性。

另外，采用权限分级管理，异步调用及服务降级等方式，有利保证系统的稳定性。

Spring cloud的组件有，服务发现(Netflix Eureka)，客服端负载均衡(Netflix Ribbon)，[断路器](https://www.zhihu.com/search?q=%E6%96%AD%E8%B7%AF%E5%99%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A868382281%7D)(Netflix Hystrix)，分布式配置(Spring Cloud Config)。

结合[k8s](https://www.zhihu.com/search?q=k8s&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A868382281%7D),docker编排，让spring cloud得到更有效的发挥其作用。

举例架构如下：

系统采用一系列稳定的技术框架，实现数据的读写分析、数据清洗、整合、汇总、统计分析、搜索引擎、推荐分析，得出可信度、高精度的结果。

基于nodejs、vue的混合前端开发体系实现前后分离，Spring mvc，Spring boot,Spring Cloud应用开发框架以及SOA的理念，Java、NLP提供语义分析；通过CDN，业务路由、多重负载均衡以及分布式缓存、数据库存储等技术，提供一个高可靠、高并发、可扩展的大型分布式系统。

技术细分

核心框架：Spring Boot，Spring cloud

安全框架：Apache Shiro

视图框架：Spring MVC

服务端验证：Hibernate Validator

任务调度：Quartz

持久层框架：Mybatis、Mybatis plus

数据库连接池：Alibaba Druid

缓存框架：Ehcache

日志管理：SLF4J、Log4j

工具类：Apache Commons、Jackson、Xstream、

后端渲染模板引擎: Thymeleaf

JAVA入门还是稍稍要有点技术知识的，要不好多东西也只能死记硬背，如果你想初步的先了解下，可以点下面的链接找管理领免费的资料先学习下，链接放这了有需求自取