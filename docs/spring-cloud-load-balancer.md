# Spring 云负载平衡器简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-load-balancer>

## 1.介绍

随着微服务架构变得越来越流行，运行分布在不同服务器上的多个服务变得越来越常见。在这个快速教程中，**我们将看看如何使用 [Spring Cloud 负载平衡器](https://web.archive.org/web/20220727020730/https://spring.io/guides/gs/spring-cloud-loadbalancer/)来创建更多容错应用**。

## 2.什么是负载平衡？

负载平衡是在同一应用程序的不同实例之间分配流量的过程。

为了创建一个容错系统，通常每个应用程序运行多个实例。因此，每当一个服务需要与另一个服务通信时，它需要选择一个特定的实例来发送它的请求。

谈到负载平衡，有许多算法:

*   随机选择:随机选择一个实例
*   循环法:每次以相同的顺序选择一个实例
*   最少连接数:选择当前连接数最少的实例
*   加权度量:使用加权度量选择最佳实例(例如，CPU 或内存使用情况)
*   IP 哈希:使用客户端 IP 的哈希映射到实例

这些只是负载平衡算法的几个例子，**并且每一个都有它的优点和缺点**。

随机选择和循环调度易于实现，但可能无法优化服务的使用。相反，最少连接数和加权指标更复杂，但通常会产生更优的服务利用率。当服务器粘性很重要时，IP hash 很有用，但是它不是很容错。

## 3.Spring 云负载平衡器简介

Spring Cloud 负载平衡器库**允许我们创建以负载平衡方式与其他应用程序通信的应用程序**。使用我们想要的任何算法，我们可以在进行远程服务调用时轻松实现负载平衡。

为了说明这一点，我们来看一些示例代码。我们将从简单的服务器应用程序开始。服务器将有一个 HTTP 端点，可以作为多个实例运行。

然后，我们将创建一个客户机应用程序，它使用 Spring Cloud 负载平衡器在服务器的不同实例之间交替请求。

### 3.1.示例服务器

对于我们的示例服务器，我们从一个简单的 Spring Boot 应用程序开始:

```java
@SpringBootApplication
@RestController
public class ServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServerApplication.class, args);
    }

    @Value("${server.instance.id}")
    String instanceId;

    @GetMapping("/hello")
    public String hello() {
        return String.format("Hello from instance %s", instanceId);
    }
}
```

我们首先注入一个名为`instanceId.`的可配置变量，这允许我们区分多个正在运行的实例。接下来，我们添加一个 HTTP GET 端点，它回显一条消息和实例 ID。

默认实例将在 ID 为 1 的端口 8080 上运行。为了运行第二个实例，我们只需要添加几个程序参数:

```java
--server.instance.id=2 --server.port=8081
```

### 3.2.示例客户端

现在，让我们看看客户端代码。**这就是我们使用 Spring Cloud 负载平衡器**的地方，所以让我们从将它包含在我们的应用程序中开始:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

接下来，我们创建一个`ServiceInstanceListSupplier`的实现。**这是 Spring Cloud 负载均衡器**中的关键接口之一。它定义了我们如何找到可用的服务实例。

对于我们的示例应用程序，我们将对示例服务器的两个不同实例进行硬编码。它们运行在同一台机器上，但使用不同的端口:

```java
class DemoInstanceSupplier implements ServiceInstanceListSupplier {
    private final String serviceId;

    public DemoInstanceSupplier(String serviceId) {
        this.serviceId = serviceId;
    }

    @Override
    public String getServiceId() {
        return serviceId;
    }

    @Override
        public Flux<List<ServiceInstance>> get() {
          return Flux.just(Arrays
            .asList(new DefaultServiceInstance(serviceId + "1", serviceId, "localhost", 8080, false),
              new DefaultServiceInstance(serviceId + "2", serviceId, "localhost", 8081, false)));
    }
}
```

在现实世界的系统中，我们希望使用不硬编码服务地址的实现。稍后我们将对此进行更深入的探讨。

现在，让我们创建一个`LoadBalancerConfiguration`类:

```java
@Configuration
@LoadBalancerClient(name = "example-service", configuration = DemoServerInstanceConfiguration.class)
class WebClientConfig {
    @LoadBalanced
    @Bean
    WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}
```

这个类有一个作用:创建一个负载平衡的`WebClient`生成器来发出远程请求。**注意，我们的注释为服务使用了一个伪名**。

这是因为我们可能不会提前知道运行实例的实际主机名和端口。因此，我们使用一个伪名称作为占位符，当框架选择一个正在运行的实例时，它将替换真实值。

接下来，让我们创建一个`Configuration`类来实例化我们的服务实例供应商。请注意，我们使用了与上面相同的伪名称:

```java
@Configuration
class DemoServerInstanceConfiguration {
    @Bean
    ServiceInstanceListSupplier serviceInstanceListSupplier() {
        return new DemoInstanceSupplier("example-service");
    }
}
```

现在，我们可以创建实际的客户端应用程序。让我们使用上面的`WebClient` bean 向示例服务器发送十个请求:

```java
@SpringBootApplication
public class ClientApplication {

    public static void main(String[] args) {

        ConfigurableApplicationContext ctx = new SpringApplicationBuilder(ClientApplication.class)
          .web(WebApplicationType.NONE)
          .run(args);

        WebClient loadBalancedClient = ctx.getBean(WebClient.Builder.class).build();

        for(int i = 1; i <= 10; i++) {
            String response =
              loadBalancedClient.get().uri("http://example-service/hello")
                .retrieve().toEntity(String.class)
                .block().getBody();
            System.out.println(response);
        }
    }
}
```

查看输出，我们可以确认我们正在两个不同的实例之间进行负载平衡:

```java
Hello from instance 2
Hello from instance 1
Hello from instance 2
Hello from instance 1
Hello from instance 2
Hello from instance 1
Hello from instance 2
Hello from instance 1
Hello from instance 2
Hello from instance 1
```

## 4.其他功能

**示例服务器和客户端展示了 Spring Cloud 负载平衡器**的一个非常简单的用法。但是其他库特性也值得一提。

首先，示例客户机使用默认的`RoundRobinLoadBalancer`策略。该库还提供了一个`RandomLoadBalancer`类。我们也可以用任何我们想要的算法创建我们自己的`ReactorServiceInstanceLoadBalancer`实现。

此外，这个库提供了一种方式来让**动态地**发现服务实例。我们使用`DiscoveryClientServiceInstanceListSupplier`接口来完成这项工作。这对于集成服务发现系统很有用，比如[尤里卡](/web/20220727020730/https://www.baeldung.com/spring-cloud-netflix-eureka)或[动物园管理员](/web/20220727020730/https://www.baeldung.com/java-zookeeper)。

除了不同的负载平衡和服务发现功能，该库还提供了基本的重试功能。在引擎盖下，它最终依赖于 [Spring Retry 库](/web/20220727020730/https://www.baeldung.com/spring-retry)。**这允许我们重试失败的请求**，可能在等待一段时间后使用相同的实例。

另一个内置特性是 metrics，它建立在[微米](/web/20220727020730/https://www.baeldung.com/micrometer)库之上。开箱即用，我们可以获得每个实例的基本服务级别指标，但我们也可以添加自己的指标。

最后，Spring Cloud 负载平衡器库提供了一种使用`LoadBalancerCacheManager`接口缓存服务实例的方法。这很重要，因为在现实中，**查找可用的服务实例可能涉及到一个远程调用**。这意味着查找不经常更改的数据可能会很昂贵，而且这也代表了应用程序中可能的故障点。**通过使用服务实例缓存，我们的应用程序可以解决这些缺点。**

## 5.结论

负载平衡是构建现代容错系统的重要组成部分。使用 Spring Cloud 负载平衡器，**我们可以轻松地创建应用程序，使用各种负载平衡技术将请求分发到不同的服务实例**。

当然，这里的所有示例代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220727020730/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-loadbalancer)