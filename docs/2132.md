# 使用 Spring Cloud 网飞功能区重试失败的请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-netflix-ribbon-retry>

## 1.概观

Spring Cloud 通过使用[网飞功能区](/web/20220627170527/https://www.baeldung.com/spring-cloud-rest-client-with-netflix-ribbon)提供客户端负载平衡。Ribbon 的负载平衡机制可以通过重试来补充。

在本教程中，我们将探索这种重试机制。

首先，我们将了解为什么在构建我们的应用程序时需要考虑这个特性是如此重要。然后，我们将使用 Spring Cloud 网飞功能区构建和配置一个应用程序来演示这种机制。

## 2.动机

在基于云的应用程序中，服务向其他服务发出请求是一种常见的做法。但是在这样一个动态多变的环境中，网络可能会出现故障，服务可能会暂时不可用。

我们希望以优雅的方式处理故障，并快速恢复。在很多情况下，这些问题都是短暂的。如果我们在失败发生后不久重复同样的请求，也许它会成功。

**这一实践有助于我们提高应用的弹性**，这是可靠的云应用的关键方面之一。

然而，我们需要关注重试，因为它们也可能导致不好的情况。例如，它们会增加延迟，这可能是不希望的。

## 3.设置

为了试验重试机制，我们需要两个 Spring Boot 服务。首先，我们将创建一个通过 REST 端点显示今天天气信息的`weather-service`。

其次，我们将定义一个使用`weather`端点的客户端服务。

### 3.1.气象局

让我们构建一个非常简单的**天气服务，它有时会失败，带有 503 HTTP 状态代码**(服务不可用)。我们将通过选择在调用次数是可配置的`successful.call.divisor`属性的倍数时失败来模拟这种间歇性故障:

```
@Value("${successful.call.divisor}")
private int divisor;
private int nrOfCalls = 0;

@GetMapping("/weather")
public ResponseEntity<String> weather() {
    LOGGER.info("Providing today's weather information");
    if (isServiceUnavailable()) {
        return new ResponseEntity<>(HttpStatus.SERVICE_UNAVAILABLE);
    }
    LOGGER.info("Today's a sunny day");
    return new ResponseEntity<>("Today's a sunny day", HttpStatus.OK);
}

private boolean isServiceUnavailable() {
    return ++nrOfCalls % divisor != 0;
}
```

此外，为了帮助我们观察对服务的重试次数，我们在处理程序中有一个消息记录器。

稍后，我们将配置客户端服务，以便在天气服务暂时不可用时触发重试机制。

### 3.2.客户服务

我们的第二项服务将使用春云网飞丝带。

首先，让我们定义[功能区客户端配置](/web/20220627170527/https://www.baeldung.com/spring-cloud-rest-client-with-netflix-ribbon):

```
@Configuration
@RibbonClient(name = "weather-service", configuration = RibbonConfiguration.class)
public class WeatherClientRibbonConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

}
```

我们的 HTTP 客户端用`@LoadBalanced `标注，这意味着我们希望它用 Ribbon 实现负载平衡。

我们现在将添加一个 ping 机制来确定服务的可用性，以及一个循环负载平衡策略，方法是定义上面的`@RibbonClient`注释中包含的`RibbonConfiguration`类:

```
public class RibbonConfiguration {

    @Bean
    public IPing ribbonPing() {
        return new PingUrl();
    }

    @Bean
    public IRule ribbonRule() {
        return new RoundRobinRule();
    }
}
```

接下来，我们需要从功能区客户端关闭[尤里卡](/web/20220627170527/https://www.baeldung.com/spring-cloud-netflix-eureka)，因为**我们没有使用服务发现**。相反，我们使用手动定义的可用于负载平衡的`weather-service`实例列表。

因此，让我们也将这些全部添加到`application.yml`文件中:

```
weather-service:
    ribbon:
        eureka:
            enabled: false
        listOfServers: http://localhost:8021, http://localhost:8022
```

最后，让我们构建一个控制器，并让它调用后端服务:

```
@RestController
public class MyRestController {

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/client/weather")
    public String weather() {
        String result = this.restTemplate.getForObject("http://weather-service/weather", String.class);
        return "Weather Service Response: " + result;
    }
}
```

## 4.启用重试机制

### 4.1.配置`application.yml`属性

我们需要将天气服务属性放在客户端应用程序的`application.yml`文件中:

```
weather-service:
  ribbon:
    MaxAutoRetries: 3
    MaxAutoRetriesNextServer: 1
    retryableStatusCodes: 503, 408
    OkToRetryOnAllOperations: true
```

上述配置使用我们需要定义的标准功能区属性来启用重试:

*   `MaxAutoRetries` **`–`** 同一服务器上重试失败请求的次数(默认为 0)
*   `MaxAutoRetriesNextServer` **–**尝试排除第一台服务器的服务器数量(默认为 0)
*   `retryableStatusCodes` **–**要重试的 HTTP 状态代码列表
*   `OkToRetryOnAllOperations` **–**当该属性设置为 true 时，将重试所有类型的 HTTP 请求，而不仅仅是 GET 请求(默认)

当客户端服务收到 503(服务不可用)或 408(请求超时)响应代码时，我们将重试失败的请求。

### 4.2.必需的依赖关系

**Spring Cloud 网飞功能区利用 [Spring Retry](https://web.archive.org/web/20220627170527/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.retry%22%20AND%20a%3A%22spring-retry%22) 来重试失败的请求。**

我们必须确保依赖关系在类路径上。否则，将不会重试失败的请求。我们可以省略这个版本，因为它是由 Spring Boot 管理的:

```
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

### 4.3.实践中的重试逻辑

最后，让我们看看实践中的重试逻辑。

因此，我们需要两个天气服务实例，我们将在 8021 和 8022 端口上运行它们。当然，这些实例应该与上一节定义的`listOfServers`列表相匹配。

此外，我们需要在每个实例上配置`successful.call.divisor`属性，以确保我们模拟的服务在不同的时间失败:

```
successful.call.divisor = 5 // instance 1
successful.call.divisor = 2 // instance 2
```

接下来，让我们在端口 8080 上运行客户端服务，并调用:

```
http://localhost:8080/client/weather
```

让我们来看看`weather-service`的控制台:

```
weather service instance 1:
    Providing today's weather information
    Providing today's weather information
    Providing today's weather information
    Providing today's weather information

weather service instance 2:
    Providing today's weather information
    Today's a sunny day
```

因此，经过几次尝试(实例 1 上 4 次，实例 2 上 2 次)，我们得到了有效的响应。

## 5.回退策略配置

当网络遇到超过其处理能力的数据量时，就会发生拥塞。为了缓解这种情况，我们可以建立一个退避策略。

**默认情况下，重试之间没有延迟。**下面，春云丝带使用[春重试](/web/20220627170527/https://www.baeldung.com/spring-retry)的`NoBackOffPolicy`对象，它什么也不做。

然而，**我们可以通过扩展`RibbonLoadBalancedRetryFactory`类来覆盖默认行为**:

```
@Component
private class CustomRibbonLoadBalancedRetryFactory 
  extends RibbonLoadBalancedRetryFactory {

    public CustomRibbonLoadBalancedRetryFactory(
      SpringClientFactory clientFactory) {
        super(clientFactory);
    }

    @Override
    public BackOffPolicy createBackOffPolicy(String service) {
        FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(2000);
        return fixedBackOffPolicy;
    }
}
```

**`FixedBackOffPolicy`类提供重试尝试之间的固定延迟。**如果我们不设置退避周期，默认是 1 秒。

或者，**我们可以设置一个`ExponentialBackOffPolicy` 或者一个`ExponentialRandomBackOffPolicy`T3:**

```
@Override
public BackOffPolicy createBackOffPolicy(String service) {
    ExponentialBackOffPolicy exponentialBackOffPolicy = 
      new ExponentialBackOffPolicy();
    exponentialBackOffPolicy.setInitialInterval(1000);
    exponentialBackOffPolicy.setMultiplier(2); 
    exponentialBackOffPolicy.setMaxInterval(10000);
    return exponentialBackOffPolicy;
}
```

这里，尝试之间的初始延迟是 1 秒。然后，每次不超过 10 秒的后续尝试的延迟会加倍:1000 毫秒、2000 毫秒、4000 毫秒、8000 毫秒、10000 毫秒、10000 毫秒…

此外，`ExponentialRandomBackOffPolicy`在不超过下一个值的情况下，给每个休眠周期添加一个随机值。因此，它可能产生 1500 毫秒、3400 毫秒、6200 毫秒、9800 毫秒、10000 毫秒、10000 毫秒…

选择一个或另一个取决于我们有多少流量和多少不同的客户服务。从固定到随机，这些策略帮助我们更好地传播流量峰值，也意味着更少的重试。例如，对于许多客户端，随机因子有助于避免几个客户端在重试时同时点击服务。

## 6.结论

在本文中，我们学习了如何使用 Spring Cloud 网飞功能区在 Spring Cloud 应用程序中重试失败的请求。我们还讨论了这种机制提供的好处。

接下来，我们演示了重试逻辑如何通过由两个 Spring Boot 服务支持的 REST 应用程序工作。Spring Cloud 网飞 Ribbon 通过利用 Spring Retry 库使之成为可能。

最后，我们看到了如何在重试尝试之间配置不同类型的延迟。

和往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627170527/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-ribbon-retry)