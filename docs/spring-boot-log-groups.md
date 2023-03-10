# Spring Boot 2.1 中的日志组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-log-groups>

## 1.概观

Spring Boot 提供了许多自动配置来帮助编写企业应用程序。然而，将相同的[日志](/web/20221208143832/https://www.baeldung.com/spring-boot-logging)配置应用到一组日志记录器上总是有点麻烦。

在这个快速教程中，我们将看到新的[日志组](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-log-groups)特性将如何解决这个问题。

## 2.日志组

**从 [Spring Boot 2.1](https://web.archive.org/web/20221208143832/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.1-Release-Notes#logging-groups) 开始，可以将多个记录器组合在一起，然后同时配置它们。**

为了使用这个特性，首先，我们应该通过`logging.group`配置属性声明一个组:

```java
logging.group.rest=com.baeldung.web,org.springframework.web,org.springframework.http
```

这里我们创建一个名为`rest `的组，包含三个不同的日志名称。**将记录器分组就像用逗号分隔它们各自的记录器名称一样简单。**

然后，我们可以一次将配置应用于一个组中的所有记录器。例如，这里我们将该组的日志级别更改为`debug:`

```java
logging.level.rest=DEBUG 
```

因此，Spring Boot 对所有三个组成员应用相同的日志级别。

### 2.1.内置组

默认情况下，Spring Boot 有两个内置组:`sql `和`web. `

目前，`web `组由以下记录器组成:

*   `org.springframework.core.codec`
*   `org.springframework.http`
*   `org.springframework.web`
*   `org.springframework.boot.actuate.endpoint.web`
*   `org.springframework.boot.web.servlet.ServletContextInitializerBeans`

类似地，`sql `组包含以下记录器:

*   `org.springframework.jdbc.core`
*   `org.hibernate.SQL`
*   `org.jooq.tools.LoggerListener`

为这些组中的任何一个组配置日志级别都会自动应用到所有组成员。

## 3.结论

在这篇短文中，我们熟悉了 Spring Boot 的日志组。此功能使我们能够一次将日志配置应用于一组记录器。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-logging-log4j2)