# Spring 配置引导程序与应用程序属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-bootstrap-properties>

## 1.概观

Spring Boot 是一个固执己见的框架。尽管如此，我们通常会在应用程序配置文件中覆盖自动配置的属性，比如`application.properties`。

然而，在 Spring Cloud 应用程序中，我们经常使用另一个名为`bootstrap.properties`的配置文件。

在这个快速教程中，我们将解释`bootstrap.properties`和`application.properties` 之间的**差异。**

## 2.什么时候使用应用程序配置文件？

我们使用`application.yml`或`application.properties` **来配置应用上下文**。

当 Spring Boot 应用程序启动时，它创建一个不需要显式配置的应用程序上下文——它已经自动配置好了。然而， **Spring Boot 提供了不同的方法来覆盖这些属性**。

我们可以在代码、命令行参数、`ServletConfig` init 参数、`ServletContext` init 参数、Java 系统属性、操作系统变量和应用程序属性文件中覆盖这些参数。

要记住的一件重要事情是，与其他形式的覆盖应用程序上下文属性相比，这些应用程序属性文件**具有最低的优先级**。

我们倾向于对可以在应用程序上下文中覆盖的属性进行分组:

*   核心属性(日志属性、线程属性)
*   集成属性(`RabbitMQ`属性，`ActiveMQ`属性)
*   网页属性(`HTTP`属性，`MVC`属性)
*   安全属性(`LDAP`属性，`OAuth2`属性)

## 3.什么时候使用引导配置文件？

我们使用`bootstrap.yml`或`bootstrap.properties`T2 来配置引导上下文。这样，我们可以很好地将引导和主上下文的外部配置分开。

**引导上下文负责从外部源**加载配置属性，并负责解密本地外部配置文件中的属性。

当 Spring Cloud 应用程序启动时，它会创建一个`bootstrap context`。首先要记住的是，`bootstrap context`是主应用程序的父上下文。

需要记住的另一个要点是，这两个上下文共享 **`Environment`，这是任何 Spring 应用程序**的外部属性的来源。与应用程序上下文相比，引导上下文使用不同的约定来定位外部配置。

例如，配置文件的来源可以是文件系统，甚至是 git 存储库。服务使用它们的`spring-cloud-config-client`依赖关系来访问配置服务器。

简而言之，**配置服务器是我们访问应用上下文配置文件**的地方。

## 4.快速示例

在本例中，引导上下文配置文件配置了`spring-cloud-config-client`依赖项，以加载正确的应用程序属性文件。

让我们看一个`bootstrap.properties`文件的例子:

```java
spring.application.name=config-client
spring.profiles.active=development
spring.cloud.config.uri=http://localhost:8888
spring.cloud.config.username=root
spring.cloud.config.password=s3cr3t
spring.cloud.config.fail-fast=true
management.security.enabled=false
```

## 5.结论

与 Spring Boot 应用程序相比，Spring Cloud 应用程序具有一个引导上下文，它是应用程序上下文的父上下文。虽然它们共享相同的`Environment`，但是它们有不同的定位外部配置文件的约定。

引导上下文正在搜索`bootstrap.properties`或`bootstrap.yaml file,`，而应用上下文正在搜索`application.properties`或`application.yaml file`。

当然，引导上下文的配置属性在应用上下文的配置属性之前加载。