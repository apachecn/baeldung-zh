# Spring Cloud Gateway 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dapr-spring-cloud-gateway>

## 1.概观

在本文中，我们将从一个 [Spring Cloud Gateway](/web/20220926184938/https://www.baeldung.com/spring-cloud-gateway) 应用程序和一个 [Spring Boot](/web/20220926184938/https://www.baeldung.com/spring-boot) 应用程序开始。然后，我们将更新它，改为使用 [Dapr(分布式应用程序运行时)](https://web.archive.org/web/20220926184938/http://dapr.io/)。最后，我们将更新 Dapr 配置，以展示 Dapr 在与云原生组件集成时提供的**灵活性。**

## 2.Dapr 简介

借助 Dapr，我们可以管理云原生应用的部署，而不会对应用本身产生任何影响。Dapr 使用 [sidecar 模式](https://web.archive.org/web/20220926184938/https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar) 从应用程序中卸载部署问题，这允许我们**将其部署到其他环境**(例如内部部署、不同的专有云平台、Kubernetes 等)**中，而无需对应用程序本身进行任何更改**。有关更多详细信息，请查看 Dapr 网站上的此[概述](https://web.archive.org/web/20220926184938/https://docs.dapr.io/concepts/overview/)。

## 3。创建示例应用程序

我们将从创建一个示例 Spring 云网关和 Spring Boot 应用程序开始。在“Hello world”示例的伟大传统中，网关将把请求代理到后端 Spring Boot 应用程序，以获得标准的“Hello world”问候。

### 3.1。问候服务

首先，让我们为问候服务创建一个 Spring Boot 应用程序。这是一个标准的 Spring Boot 应用程序，其中`spring-boot-starter-web`是唯一的依赖项，是标准的主类，服务器端口配置为 3001。

让我们添加一个控制器来响应`hello`端点:

```java
@RestController
public class GreetingController {
    @GetMapping(value = "/hello")
    public String getHello() {
        return "Hello world!";
    }
} 
```

建立我们的问候服务应用程序后，我们将启动它:

```java
java -jar greeting/target/greeting-1.0-SNAPSHOT.jar
```

我们可以使用`curl`返回“Hello world！”消息:

```java
curl http://localhost:3001/hello
```

### 3.2。春云网关

现在，我们将在端口 3000 上创建一个 Spring Cloud Gateway，作为一个标准的 Spring Boot 应用程序，使用`spring-cloud-starter-gateway`作为唯一的依赖项和标准的主类。我们还将配置路由到 来访问问候服务:

```java
spring:
  cloud:
    gateway:
      routes:
        - id: greeting-service
          uri: http://localhost:3001/
          predicates:
            - Path=/**
          filters:
          - RewritePath=/?(?<segment>.*), /$\{segment} 
```

一旦我们构建了网关应用程序，我们就可以启动网关:

```java
java -Dspring.profiles.active=no-dapr -jar gateway/target/gateway-1.0-SNAPSHOT.jar
```

我们可以使用`curl`返回“Hello world！”来自问候服务的消息:

```java
curl http://localhost:3000/hello
```

## 4。添加 Dapr

现在我们已经有了一个基本的例子，让我们把 Dapr 也加入进来。

我们通过**配置网关与 Dapr 边车**进行通信，而不是直接与问候服务进行通信。然后，Dapr 将负责查找问候服务并将请求转发给它；现在，通信路径将从网关开始，通过 Dapr 边车，到达问候服务。

### 4.1。部署 Dapr 边车

首先，我们需要部署两个 Dapr sidecar 实例——一个用于网关，一个用于问候服务。我们使用 [Dapr CLI](https://web.archive.org/web/20220926184938/https://docs.dapr.io/reference/cli/) 来完成这项工作。

我们将使用标准的 Dapr 配置文件:

```java
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: daprConfig
spec: {} 
```

让我们使用`dapr`命令:为端口 4000 上的网关启动 Dapr sidecar

```java
dapr run --app-id gateway --dapr-http-port 4000 --app-port 3000 --config dapr-config/basic-config.yaml
```

接下来，让我们使用`dapr`命令为端口 4001 上的问候服务启动 Dapr sidecar:

```java
dapr run --app-id greeting --dapr-http-port 4001 --app-port 3001 --config dapr-config/basic-config.yaml
```

现在 sidecars 正在运行，我们可以看到它们是如何截取请求并将其转发给问候服务的。当我们使用`curl`测试它时，它应该返回“Hello world！”问候语:

```java
curl http://localhost:4001/v1.0/invoke/greeting/method/hello
```

让我们使用 gateway sidecar 尝试相同的测试，以确认它也返回“Hello world！”问候语:

```java
curl http://localhost:4000/v1.0/invoke/greeting/method/hello
```

幕后发生了什么？**用于网关的 Dapr 侧柜使用服务发现**(在本例中，本地环境的 mDNS)来发现用于问候服务的 Dapr 侧柜。然后，**使用[服务调用](https://web.archive.org/web/20220926184938/https://docs.dapr.io/developing-applications/building-blocks/service-invocation/service-invocation-overview/)来调用问候服务上的指定端点**。

### 4.2。更新网关配置

下一步是将网关路由配置为使用其 Dapr 边车:

```java
spring:
  cloud:
    gateway:
      routes:
        - id: greeting-service
          uri: http://localhost:4000/
          predicates:
            - Path=/**
          filters:
          - RewritePath=//?(?<segment>.*), /v1.0/invoke/greeting/method/$\{segment} 
```

然后，我们将使用更新后的路由重新启动网关:

```java
java -Dspring.profiles.active=with-dapr -jar gateway/target/gateway-1.0-SNAPSHOT.jar
```

我们可以使用`curl`命令测试一下，再次从问候服务:获得“Hello world”问候

```java
curl http://localhost:3000/hello
```

当我们使用 [Wireshark](https://web.archive.org/web/20220926184938/https://www.wireshark.org/) 查看网络上发生的事情时，我们可以看到**网关和服务之间的流量通过 Dapr sidecars** 。

恭喜你！我们现在已经成功地把 Dapr 带入了画面。让我们回顾一下这为我们带来了什么:网关不再需要配置来查找问候服务(也就是说，不再需要在路由配置中指定问候服务的端口号)，网关不再需要知道请求如何被转发到问候服务的细节。

## 5.更新 Dapr 配置

现在我们已经有了 Dapr，我们可以配置 Dapr 来使用其他云原生组件。

### 5.1。使用 Consul 进行服务发现

让我们用[领事](https://web.archive.org/web/20220926184938/https://www.consul.io/)来代替 mDNS 进行服务发现。

首先，我们需要 在默认端口 8500 上安装并启动 Consul，然后 u 更新 Dapr sidecar 配置以使用 Consul:

```java
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: daprConfig
spec:
  nameResolution:
    component: "consul"
    configuration:
      selfRegister: true
```

然后，我们将使用新配置重启两个 Dapr 边车:

```java
dapr run --app-id greeting --dapr-http-port 4001 --app-port 3001 --config dapr-config/consul-config.yaml
```

```java
dapr run --app-id gateway --dapr-http-port 4000 --app-port 3000 --config dapr-config/consul-config.yaml
```

一旦 sidecars 重新启动，我们可以访问 consul UI 中的服务页面，并看到列出的网关和问候应用程序。请注意，我们不需要重启应用程序本身。

看到这有多简单了吗？Dapr sidecars 的一个简单的配置改变现在给了我们对 Consul 和最重要的**的支持，而不影响底层应用**。这与使用[Spring Cloud consult](/web/20220926184938/https://www.baeldung.com/spring-cloud-consul)不同，后者需要更新应用程序本身。

### 5.2。使用 Zipkin 追踪

Dapr 还支持与 Zipkin(T2)的集成，用于跨应用程序追踪电话。

首先，在默认端口 9411 上安装并启动 Zipkin，然后更新 Dapr sidecar 的配置以添加 Zipkin:

```java
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: daprConfig
spec:
  nameResolution:
    component: "consul"
    configuration:
      selfRegister: true
  tracing:
    samplingRate: "1"
    zipkin:
      endpointAddress: "http://localhost:9411/api/v2/spans" 
```

我们需要重启两个 Dapr 边车，以获取新配置:

```java
dapr run --app-id greeting --dapr-http-port 4001 --app-port 3001 --config dapr-config/consul-zipkin-config.yaml
```

```java
dapr run --app-id gateway --dapr-http-port 4000 --app-port 3000 --config dapr-config/consul-zipkin-config.yaml
```

一旦 Dapr 重新启动，您就可以发出一个`curl`命令并检查 Zipkin UI 来查看调用跟踪。

再次重申，不需要重启网关和问候服务。它**只需要** **对 Dapr 配置**的简单更新。与使用 [Spring Cloud Zipkin](/web/20220926184938/https://www.baeldung.com/tracing-services-with-zipkin) 相比。

### 5.3。其他组件

Dapr 支持许多组件来解决其他问题，如安全、监控和报告。查看 Dapr 文档中的[完整列表](https://web.archive.org/web/20220926184938/https://docs.dapr.io/reference/components-reference/)。

## 6。结论

我们已经将 Dapr 添加到一个简单的 Spring 云网关与后端 Spring Boot 服务通信的例子中。我们已经展示了如何配置和启动 Dapr sidecar，以及它如何处理云本地的问题，比如服务发现、通信和跟踪。

虽然这是以部署和管理 sidecar 应用程序为代价的，但 Dapr 为部署到不同的云原生环境和云原生问题提供了**灵活性，一旦与 Dapr 集成到位，就无需更改应用程序。**

这种方法还意味着开发人员在编写代码时不需要受云原生问题的困扰，从而将他们解放出来专注于业务功能。一旦应用程序配置为使用 Dapr sidecar，就可以解决不同的部署问题，而不会对应用程序产生任何影响——无需重新编码、重新构建或重新部署应用程序。 **Dapr 在应用和部署问题之间提供了清晰的分离**。

一如既往，本文的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926184938/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-dapr)