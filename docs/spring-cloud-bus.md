# 春天云巴士

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-bus>

## 1。概述

在本文中，我们将了解新的 Spring Cloud Bus 项目。Spring Cloud Bus 使用轻量级消息代理链接分布式系统节点。主要用途是广播配置更改或其他管理信息。我们可以把它想象成一个分布式的[执行器](/web/20220630142054/https://www.baeldung.com/spring-boot-actuators)。

该项目使用 AMQP 代理作为传输工具，但是可以使用 Apache Kafka 或 Redis 来代替 RabbitMQ。尚不支持其他传输方式。

在本教程的整个过程中，我们将使用 RabbitMQ 作为我们的主要传输工具——我们自然已经运行了它。

## 2。先决条件

在我们开始之前，建议您已经完成了“[Spring 云配置快速介绍](/web/20220630142054/https://www.baeldung.com/spring-cloud-configuration)”。我们将利用现有的云配置服务器和客户端来扩展它们，并添加关于配置更改的自动通知。

### 2.1。RabbitMQ

让我们从 RabbitMQ 开始，我们推荐它作为 docker 映像作为 [RabbitMQ 运行。这很容易设置——要让 RabbitMQ 在本地运行，我们需要](https://web.archive.org/web/20220630142054/https://hub.docker.com/_/rabbitmq/)[安装 Docker](https://web.archive.org/web/20220630142054/https://www.docker.com/get-docker) 并在 Docker 安装成功后运行以下命令:

```java
docker pull rabbitmq:3-management
```

该命令将 RabbitMQ docker 映像与默认安装和启用的管理插件一起拉出来。

接下来，我们可以运行 RabbitMQ:

```java
docker run -d --hostname my-rabbit --name some-rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3-management
```

一旦我们执行了这个命令，我们就可以转到 web 浏览器并打开 [http://localhost:15672](https://web.archive.org/web/20220630142054/http://localhost:15672/) ，这将显示管理控制台登录表单。默认用户名为:`‘guest'`；密码:`‘guest'`。RabbitMQ 也将监听端口 5672。

## 3。向云配置客户端添加执行器

我们应该同时运行云配置服务器和云配置客户端。要刷新配置更改，每次都需要重启客户端，这并不理想。

让我们停止配置客户端并用`@RefreshScope`注释`ConfigClient`控制器类:

```java
@SpringBootApplication
@RestController
@RefreshScope
public class SpringCloudConfigClientApplication {
    // Code here...
}
```

最后，让我们更新`pom.xml`文件并添加致动器:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220630142054/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-actuator%22)

默认情况下，执行器添加的所有敏感端点都是安全的。这包括`‘/refresh'`端点。为简单起见，我们将通过更新`application.yml`来关闭安全性:

```java
management:
  security:
    enabled: false
```

此外，从 Spring Boot 2 开始，[致动器端点默认不暴露](https://web.archive.org/web/20220630142054/https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints)。为了使它们可供访问，我们需要将它添加到一个`application.yml`中:

```java
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

让我们首先启动客户端，在 GitHub 的属性文件中将用户角色从现有的`‘Developer'`更新为`‘Programmer'`。配置服务器将立即显示更新的值；但是，客户端不会。为了让客户端看到新文件，我们只需要向`‘/refresh'`端点发送一个空的 POST 请求，这是由 actuator 添加的:

```java
$> curl -X POST http://localhost:8080/actuator/refresh
```

我们将获得显示更新属性的 JSON 文件:

```java
[
  "user.role"
]
```

最后，我们可以检查用户角色是否已更新:

```java
$> curl http://localhost:8080/whoami/Mr_Pink
Hello Mr_Pink! You are a(n) Programmer and your password is 'd3v3L'.
```

用户角色已通过调用`‘/refresh'`端点成功更新。客户端更新了配置，但没有重新启动。

## 4。春云巴士

通过使用 Actuator，我们可以刷新客户端。然而，在云环境中，我们需要访问每一个客户端，并通过访问 actuator 端点来重新加载配置。

要解决这个问题，可以用春云总线。

### 4.1。客户端

我们需要更新云配置客户端，以便它可以订阅 RabbitMQ exchange:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220630142054/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-bus-amqp%22)

要完成配置客户端更改，我们需要在一个`application.yml`文件中添加 RabbitMQ 细节并启用云总线:

```java
---
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
  cloud:
    bus:
      enabled: true
      refresh:
        enabled: true 
```

请注意，我们使用默认的用户名和密码。这需要针对现实生活和生产应用进行更新。对于本教程来说，这很好。

现在，客户端将拥有另一个端点`‘/bus-refresh'`。调用此端点将导致:

*   从配置服务器获取最新配置，并更新其由`@RefreshScope`标注的配置
*   向 AMQP 交易所发送消息，通知刷新事件
*   所有订阅的节点也将更新它们的配置

这样，我们就不需要去单个节点并触发配置更新。

### 4.2。服务器

最后，让我们向配置服务器添加两个依赖项，以完全自动化配置更改。

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-monitor</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220630142054/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-config-monitor%22)

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    <version>3.0.4.RELEASE</version>
</dependency>
```

最新版本可以在[这里](https://web.archive.org/web/20220630142054/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-stream-rabbit%22)找到。

我们将使用`spring-cloud-config-monitor`来监控配置更改，并使用 RabbitMQ 作为传输工具来广播事件。

我们只需要更新`application.properties`并给出 RabbitMQ 细节:

```java
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

### 4.3。GitHub Webhook

现在一切都准备好了。一旦服务器得到配置更改的通知，它将把这作为消息广播给 RabbitMQ。当发送配置更改事件时，客户端将监听消息并更新其配置。但是，现在一个服务器将如何修改呢？

我们需要配置一个 GitHub Webhook。让我们去 GitHub 并打开我们保存配置属性的存储库。现在，让我们选择`Settings`和`Webhook`。让我们点击`Add webhook`按钮。

有效负载 URL 是我们的配置服务器`‘/monitor'`端点的 URL。在我们的例子中，URL 应该是这样的:

http://root: [【邮件保护】](/web/20220630142054/https://www.baeldung.com/cdn-cgi/l/email-protection) _IP:8888/monitor

我们只需将下拉菜单中的`Content type`更改为`application/json.` 。接下来，请将`Secret`留空并点击`Add webhook`按钮——之后，我们就都设置好了。

## 5。测试

让我们确保所有应用程序都在运行。如果我们返回并检查客户端，它会将`user.role`显示为`‘Programmer'`并将`user.password`显示为“`d3v3L`”:

```java
$> curl http://localhost:8080/whoami/Mr_Pink
Hello Mr_Pink! You are a(n) Programmer and your password is 'd3v3L'.
```

以前，我们必须使用`‘/refresh'`端点来重新加载配置更改。让我们打开属性文件，将`user.role`改回`Developer`，并推送更改:

```java
user.role=Programmer
```

如果我们现在检查客户端，我们会看到:

```java
$> curl http://localhost:8080/whoami/Mr_Pink
Hello Mr_Pink! You are a(n) Developer and your password is 'd3v3L'.
```

几乎同时，Config 客户端在没有重新启动和没有显式刷新的情况下更新其配置。我们可以回到 GitHub，打开最近创建的 Webhook。在最底部，有最近的交货。我们可以选择列表顶部的一个(假设这是第一个更改——反正只有一个——并检查已经发送到配置服务器的 JSON。

我们还可以检查配置和服务器日志，我们将看到以下条目:

```java
o.s.cloud.bus.event.RefreshListener: Received remote refresh request. Keys refreshed []
```

## 6。结论

在本文中，我们采用了现有的 spring cloud 配置服务器和客户机，并添加了 actuator 端点来刷新客户机配置。接下来，我们使用 Spring Cloud Bus 来广播配置更改和自动化客户端更新。我们还配置了 GitHub Webhook，并测试了整个设置。

和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630142054/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-bus)