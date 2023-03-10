# Spring Boot 的 Web 服务器正常关闭

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-web-server-shutdown>

## 1.概观

在这个快速教程中，我们将了解如何配置 Spring Boot 应用程序来更好地处理关机。

## 2.正常关机

从 [Spring Boot 2.3](https://web.archive.org/web/20220728225617/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes#graceful-shutdown) 开始，Spring Boot 现在支持 servlet 和反应平台上所有四个嵌入式 web 服务器(Tomcat、Jetty、Undertow 和 Netty)的正常关机特性。

**为了实现正常关机，我们所要做的就是在我们的`application.properties`文件中将`server.shutdown`属性设置为`graceful`** :

```java
server.shutdown=graceful
```

然后，Tomcat、Netty 和 Jetty 将停止在网络层接受新的请求。另一方面，Undertow 将继续接受新的请求，但会立即向客户端发送 503 服务不可用响应。

**默认情况下，该属性的值等于`immediate,` ，这意味着服务器会立即关闭。**

在正常关机阶段开始之前，有些请求可能会被接受。在这种情况下， **t** **服务器将等待那些活动的请求在指定的时间内完成它们的工作**。我们可以使用`spring.lifecycle.timeout-per-shutdown-phase` 配置属性来配置这个宽限期:

```java
spring.lifecycle.timeout-per-shutdown-phase=1m
```

如果我们添加此项，服务器将等待一分钟来完成活动请求。该属性的默认值是 30 秒。

## 3.结论

在这个简短的教程中，我们看到了如何利用 Spring Boot 2.3 中新的正常关机功能。