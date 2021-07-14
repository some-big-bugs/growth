# Spring Boot Starters

## 官方



1、application starters
你还在手写配置引入框架？建议看看这个表格。

这一个表格是 Spring Boot 所有应用程序级的 Starters，一起来看都有哪些。

Starter 名称	Starter 描述
spring-boot-starter	核心 Starter，包括自动配置、日志及 YAML 支持等
spring-boot-starter-activemq	集成 Apache ActiveMQ，基于 JMS 的消息队列
spring-boot-starter-artemis	集成 Apache Artemis，基于 JMS 的消息队列
spring-boot-starter-amqp	集成 Spring AMQP 和 Rabbit MQ 的消息队列
spring-boot-starter-aop	集成 Spring AOP 和 AspectJ 面向切面编程
spring-boot-starter-batch	集成 Spring Batch（批处理）
spring-boot-starter-cache	集成 Spring Cache（缓存）
spring-boot-starter-data-cassandra	集成 Cassandra（分布式数据库） 和 Spring Data Cassandra
spring-boot-starter-data-cassandra-reactive	集成 Cassandra（分布式数据库） 和 Spring Data Cassandra Reactive
spring-boot-starter-data-couchbase	集成 Couchbase（文档型数据库） 和 Spring Data Couchbase
spring-boot-starter-data-couchbase-reactive	集成 Couchbase（文档型数据库） 和 Spring Data Couchbase Reactive
spring-boot-starter-data-elasticsearch	集成 Elasticsearch（搜索引擎）和 Spring Data Elasticsearch
spring-boot-starter-data-solr	集成 Apache Solr（搜索引擎）结合 Spring Data Solr
spring-boot-starter-data-jdbc	集成 Spring Data JDBC
spring-boot-starter-data-jpa	集成 Spring Data JPA 结合 Hibernate
spring-boot-starter-data-ldap	集成 Spring Data LDAP
spring-boot-starter-data-mongodb	集成 MongoDB（文档型数据库）和 Spring Data MongoDB
spring-boot-starter-data-mongodb-reactive	集成 MongoDB（文档型数据库）和 Spring Data MongoDB Reactive
spring-boot-starter-data-neo4j	集成 Neo4j（图形数据库）和 Spring Data Neo4j
spring-boot-starter-data-r2dbc	集成 Spring Data R2DBC
spring-boot-starter-data-redis	集成 Redis（内存数据库）结合 Spring Data Redis 和 Lettuce 客户端
spring-boot-starter-data-redis-reactive	集成 Redis（内存数据库）结合 Spring Data Redis reactive 和 Lettuce 客户端
spring-boot-starter-data-rest	集成 Spring Data REST 暴露 Spring Data repositories 输出 REST 资源
spring-boot-starter-thymeleaf	集成 Thymeleaf 视图构建 MVC web 应用
spring-boot-starter-freemarker	集成 FreeMarker 视图构建 MVC web 应用
spring-boot-starter-groovy-templates	集成 Groovy 模板视图构建 MVC web 应用
spring-boot-starter-hateoas	集成 Spring MVC 和 Spring HATEOAS 构建超媒体 RESTful Web 应用程序
spring-boot-starter-integration	集成 Spring Integration
spring-boot-starter-jdbc	集成 JDBC 结合 HikariCP 连接池
spring-boot-starter-jersey	集成 JAX-RS 和 Jersey 构建 RESTful web 应用，是 spring-boot-starter-web 的一个替代 Starter
spring-boot-starter-jooq	集成 jOOQ 访问 SQL 数据库，是 spring-boot-starter-data-jpa 或者 spring-boot-starter-jdbc 的替代 Starter
spring-boot-starter-json	用于读写 JSON
spring-boot-starter-jta-atomikos	集成 Atomikos 实现 JTA 事务
spring-boot-starter-jta-bitronix	集成 Bitronix 实现 JTA 事务（ 从 2.3.0 开始标识为 Deprecated）
spring-boot-starter-mail	集成 Java Mail 和 Spring 框架的邮件发送功能
spring-boot-starter-mustache	集成 Mustache 视图构建 web 应用
spring-boot-starter-security	集成 Spring Security
spring-boot-starter-oauth2-client	集成 Spring Security’s OAuth2/OpenID 连接客户端功能
spring-boot-starter-oauth2-resource-server	集成 Spring Security’s OAuth2 资源服务器功能
spring-boot-starter-quartz	集成 Quartz 任务调度
spring-boot-starter-rsocket	构建 RSocket 客户端和服务端
spring-boot-starter-test	集成 JUnit Jupiter, Hamcrest 和 Mockito 测试 Spring Boot 应用和类库
spring-boot-starter-validation	集成 Java Bean Validation 结合 Hibernate Validator
spring-boot-starter-web	集成 Spring MVC 构建 RESTful web 应用，使用 Tomcat 作为默认内嵌容器
spring-boot-starter-web-services	集成 Spring Web Services
spring-boot-starter-webflux	集成 Spring Reactive Web 构建 WebFlux 应用
spring-boot-starter-websocket	集成 Spring WebSocket 构建 WebSocket 应用
用到哪个技术就引用哪个技术的 Starter，Spring Boot 助你快速集成，别再手写配置了。

2、production starters
除了上面的应用程序级 starters，还有下面的生产级 Starters 能被用于线上/生产功能：

Starter 名称	Starter 描述
spring-boot-starter-actuator	集成 Spring Boot Actuator，提供生产功能以帮助监控和管理应用程序
这个意味着和任何技术、任何业务没关系，只要用了 Spring Boot 框架，上了生产环境就能使用，也不是只有生产才能使用，只是在生产环境使用更能体验它的意义。

3、technical starters
除了应用程序和生产 Starters，Spring Boot 还包括下面的技术类 Starters，用于帮助你排除或者替换指定的框架或技术：

Starter 名称	Starter 描述
spring-boot-starter-jetty	集成 Jetty 作为内嵌的 servlet 容器，可用于替代 spring-boot-starter-tomcat
spring-boot-starter-log4j2	集成 Log4j2 日志框架，可用于替代 spring-boot-starter-logging
spring-boot-starter-logging	集成 Logback 日志框架，这个也是默认的日志 Starter
spring-boot-starter-reactor-netty	集成 Netty 作为内嵌的响应式 HTTP 服务器
spring-boot-starter-tomcat	集成 Tomcat 作为内嵌的 servlet 容器，这也是默认的 servlet 容器 starter 被集成 spring-boot-starter-web 里面
spring-boot-starter-undertow	集成 Undertow 作为内嵌的 servlet 容器，可用于替代 spring-boot-starter-tomcat
这个表格的技术也很熟悉了，Spring Boot 默认内嵌 Servlet 容器为 Tomcat，如果你想换成 Jetty、Undertow 或者其他容器，又或者你想换成其他的日志框架，都在这个表格里，怎么换？点击这里参考我之前写的这篇教程。

最新请参考：

https://docs.spring.io/spring-boot/docs/

# 结语

本文一共收集了 54 个 Spring Boot 官方的 Starter，参考来源于 Spring Boot 2.4.0，不限于这 54 个，随着 Spring Boot 版本的不断升级，后续可能会增加更多的 Starter，当然也有少数 Starter 可能会得到删除。

官方自带的可以直接拿来用，大家看看，就没有必要重复造轮子了。

如果 Spring Boot 官方没有自带的 Starter，一般第三方的框架也都会提供自制的 Spring Boot Starter，如：Dubbo、Zookeeper 等，这样只要几个依赖，几行配置参数就能轻松实现集成。后面栈长再整理一篇常用的第三方的 Starters，关注公众号Java技术栈第一时间推送。

当然，除了第三方的 Starter，使用 Spring Boot 的公司一般也会有私有定制的 Starter，可以用于在公司内部各业务部门快速集成使用，而不用各自造轮子。

除了会使用 Spring Boot Starter，了解它的原理也非常有必要，因为你的上司随时都会让你写一个！

好了，今天的分享就到这了，后续有大版本更新，官方 Starters 调整比较大的话，后续栈长再继续更新本文，关注公众号Java技术栈第一时间推送。

如果有帮助，点个在看鼓励一下哦！也欢迎分享转发给更多有需要的朋友～

版权申明：本文系公众号 "Java技术栈" 原创，原创实属不易，转载、引用本文内容请注明出处，禁止抄袭、洗稿，请自重，尊重他人劳动成果和知识产权。

————————————————
版权声明：本文为CSDN博主「Java技术栈」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/youanyyou/article/details/111579104