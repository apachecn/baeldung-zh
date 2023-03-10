# 用网飞丝带介绍 Spring 云 Rest 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-rest-client-with-netflix-ribbon>

## 1。简介

网飞 [Ribbon](https://web.archive.org/web/20220122060633/https://github.com/Netflix/ribbon) 是一个进程间通信(IPC)云库。Ribbon 主要提供客户端负载平衡算法。

除了客户端负载平衡算法，Ribbon 还提供了其他功能:

*   **服务发现集成**–带状负载平衡器在像云一样的动态环境中提供服务发现。功能区库中包括与尤里卡和网飞服务发现组件的集成
*   **容错**–Ribbon API 可以动态地确定服务器是否在实时环境中正常运行，并且可以检测那些停机的服务器
*   **可配置的负载均衡规则**–Ribbon 支持开箱即用的`RoundRobinRule`、`AvailabilityFilteringRule`、`WeightedResponseTimeRule`，还支持定义自定义规则

Ribbon API 基于名为“命名客户端”的概念工作。在应用程序配置文件中配置 Ribbon 时，我们为负载平衡所包含的服务器列表提供了一个名称。

让我们兜一圈。

## 2。依赖性管理

通过将下面的依赖项添加到我们的`pom.xml:`中，可以将网飞功能区 API 添加到我们的项目中

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

最新的库可以在这里找到。

## 3。应用示例

为了了解 Ribbon API 的工作原理，我们使用 Spring `RestTemplate`构建了一个示例微服务应用程序，并使用网飞 Ribbon API 和 Spring Cloud 网飞 API 对其进行了增强。

我们将使用 Ribbon 的负载平衡策略之一`WeightedResponseTimeRule`，在我们的应用程序中，在配置文件中的一个命名客户端下定义的 2 个服务器之间启用客户端负载平衡。

## 4。色带配置

Ribbon API 使我们能够配置负载平衡器的以下组件:

*   `Rule`–指定我们在应用程序中使用的负载平衡规则的逻辑组件
*   `Ping`–指定我们用来实时确定服务器可用性的机制的组件
*   `ServerList`–可以是动态的或静态的。在我们的例子中，我们使用服务器的静态列表，因此我们直接在应用程序配置文件中定义它们

让我们为该库编写一个简单的配置:

```java
public class RibbonConfiguration {

    @Autowired
    IClientConfig ribbonClientConfig;

    @Bean
    public IPing ribbonPing(IClientConfig config) {
        return new PingUrl();
    }

    @Bean
    public IRule ribbonRule(IClientConfig config) {
        return new WeightedResponseTimeRule();
    }
}
```

注意我们如何使用`WeightedResponseTimeRule`规则来确定服务器，以及如何使用`PingUrl`机制来实时确定服务器的可用性。

根据这个规则，每个服务器根据其平均响应时间被赋予一个权重，响应时间越短，权重越小。这个规则随机选择一个服务器，其可能性由服务器的权重决定。

并且`PingUrl`将 ping 每个 URL 以确定服务器的可用性。

## 5。`application.yml`

下面是我们为这个示例应用程序创建的`application.yml`配置文件:

```java
spring:
  application:
    name: spring-cloud-ribbon

server:
  port: 8888

ping-server:
  ribbon:
    eureka:
      enabled: false
    listOfServers: localhost:9092,localhost:9999
    ServerListRefreshInterval: 15000
```

在上面的文件中，我们指定:

*   应用程序名称
*   应用程序的端口号
*   服务器列表的命名客户端:“ping-server”
*   通过将 eureka: `enabled`设置为`false`，禁用了 Eureka 服务发现组件
*   定义了可用于负载平衡的服务器列表，在本例中是 2 台服务器
*   用`ServerListRefreshInterval`配置服务器刷新率

## 6。`RibbonClient`

现在让我们设置主应用程序组件片段——在这里我们使用`RibbonClient`而不是普通的`RestTemplate`来实现负载平衡:

```java
@SpringBootApplication
@RestController
@RibbonClient(
  name = "ping-a-server",
  configuration = RibbonConfiguration.class)
public class ServerLocationApp {

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/server-location")
    public String serverLocation() {
        return this.restTemplate.getForObject(
          "http://ping-server/locaus", String.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(ServerLocationApp.class, args);
    }
}
```

这里是`RestTemplate`的配置:

```java
@Configuration
public class RestTemplateConfiguration{
    @LoadBalanced
    @Bean
    RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

我们用注释`@RestController`定义了一个控制器类；我们还用名称和配置类用`@RibbonClient`注释了这个类。

我们在这里定义的配置类与之前定义的类相同，在之前的类中，我们为该应用程序提供了所需的功能区 API 配置。

注意，我们用`@LoadBalanced`注释了`RestTemplate`,这表明我们希望这是负载平衡的，在本例中是用 Ribbon。

## 7。功能区中的故障恢复能力

正如我们在本文前面所讨论的，Ribbon API 不仅提供了客户端负载平衡算法，还内置了故障恢复能力。

如前所述，Ribbon API 可以通过定期对服务器进行持续的 ping 操作来确定服务器的可用性，并且能够跳过不活动的服务器。

除此之外，它还实现了断路器模式，根据指定的标准过滤掉服务器。

断路器模式通过快速拒绝对出现故障的服务器的请求，而不等待超时，从而最大限度地降低了服务器故障对性能的影响。我们可以通过将属性`niws.loadbalancer.availabilityFilteringRule.filterCircuitTripped`设置为`false`来禁用这个断路器特性。

当所有服务器都关闭时，因此没有服务器可用于服务请求，`pingUrl()`将失败，我们将收到一个异常`java.lang.IllegalStateException`和一条消息`“No instances are available to serve the request”`。

## 8。结论

在本文中，我们讨论了网飞 Ribbon API 及其在一个简单示例应用程序中的实现。

上述示例的完整源代码可以在 [GitHub 库](https://web.archive.org/web/20220122060633/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-ribbon-client)中找到。