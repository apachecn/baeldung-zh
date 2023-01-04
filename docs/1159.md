# 弹簧云断路器快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-circuit-breaker>

## 1.概观

在本教程中，**我们将介绍 Spring Cloud 断路器项目**，并学习如何利用它。

首先，我们将看看除了现有的断路器实现之外，Spring Cloud 断路器还提供了什么。接下来，我们将学习如何使用 [Spring Boot 自动配置](/web/20221208143837/https://www.baeldung.com/spring-boot-custom-auto-configuration)机制将一个或多个断路器集成到我们的应用程序中。

请注意，我们在[Hystrix 简介](/web/20221208143837/https://www.baeldung.com/introduction-to-hystrix)、 [Spring Cloud 网飞 Hystrix](/web/20221208143837/https://www.baeldung.com/spring-cloud-netflix-hystrix) 和[resilience 4j 指南](/web/20221208143837/https://www.baeldung.com/resilience4j)中获得了更多关于断路器及其工作原理的信息。

## 2.弹簧云断路器

直到最近，Spring Cloud 只为我们提供了一种在应用程序中添加断路器的方法。这是通过使用网飞海斯特里克斯作为春天云网飞项目的一部分。

Spring Cloud 网飞项目实际上只是一个基于注释的包装器库。因此，这两个库是紧密耦合的。这意味着我们不能在不改变应用的情况下切换到另一种断路器实现。

春云断路器项目解决了这个问题。它提供了一个跨越不同断路器实现的抽象层。这是一个可插拔的架构。因此，我们可以根据提供的抽象/接口进行编码，并根据我们的需要切换到另一个实现。

对于我们的例子，**我们将只关注 Resilience4J 的实现。然而，这些技术可以用于其他插件。**

## 3.自动配置

**为了在我们的应用中使用特定的断路器实现，我们需要添加合适的弹簧启动器。**在我们的例子中，让我们用 [`spring-cloud-starter-circuitbreaker-resilience4j`](https://web.archive.org/web/20221208143837/https://search.maven.org/search?q=spring-cloud-starter-circuitbreaker-resilience4j) :

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    <version>1.0.2.RELEASE</version>
</dependency>
```

**如果自动配置机制在类路径中发现一个启动器，它会配置必要的断路器 bean**。

如果我们想要禁用 Resilience4J 自动配置，我们可以将`spring.cloud.circuitbreaker.resilience4j.enabled`属性设置为`false`。

## 4.一个简单的断路器示例

让我们使用 Spring Boot 创建一个 web 应用程序，来探索 Spring Cloud 断路器库是如何工作的。

我们将构建一个简单的 web 服务来返回相册列表。假设原始列表是由第三方服务提供的。为了简单起见，我们将使用由 [Jsonplaceholder](https://web.archive.org/web/20221208143837/https://jsonplaceholder.typicode.com/) 提供的外部伪 API 来检索列表:

```
https://jsonplaceholder.typicode.com/albums
```

### 4.1.创建一个断路器

让我们创造我们的第一个断路器。我们将从注入一个`CircuitBreakerFactory` bean 的实例开始:

```
@Service
public class AlbumService {

    @Autowired
    private CircuitBreakerFactory circuitBreakerFactory;

    //... 

}
```

现在，我们可以使用`CircuitBreakerFactory#create`方法轻松创建一个断路器。它将断路器标识符作为参数:

```
CircuitBreaker circuitBreaker = circuitBreakerFactory.create("circuitbreaker");
```

### 4.2.将任务包装在断路器中

为了包装和运行受断路器保护的任务，我们需要调用 r `un`方法，该方法将一个`Supplier`作为参数。

```
public String getAlbumList() {
    CircuitBreaker circuitBreaker = circuitBreakerFactory.create("circuitbreaker");
    String url = "https://jsonplaceholder.typicode.com/albums";

    return circuitBreaker.run(() -> restTemplate.getForObject(url, String.class));
}
```

**断路器为我们运行我们的方法，并提供容错。**

有时，我们的外部服务可能需要很长时间来响应，抛出一个意外的异常，或者外部服务或主机不存在。在这种情况下，**我们可以提供一个回退**作为`run`方法的第二个参数:

```
public String getAlbumList() {
    CircuitBreaker circuitBreaker = circuitBreakerFactory.create("circuitbreaker");
    String url = "http://localhost:1234/not-real";

    return circuitBreaker.run(() -> restTemplate.getForObject(url, String.class), 
      throwable -> getDefaultAlbumList());
}
```

回退的 lambda 接收描述错误的`Throwable`作为输入。这意味着**我们可以根据触发回退的异常**的类型，向调用者提供不同的回退结果。

在这种情况下，我们不会考虑异常。我们将返回一个缓存的专辑列表。

如果外部调用以异常结束，并且没有提供回退，Spring 就会抛出一个`NoFallbackAvailableException`。

### 4.3.构建控制器

现在，让我们完成我们的示例并创建一个简单的控制器，该控制器调用服务方法并通过浏览器呈现结果:

```
@RestController
public class Controller {

    @Autowired
    private Service service;

    @GetMapping("/albums")
    public String albums() {
        return service.getAlbumList();
    }

}
```

最后，让我们调用 REST 服务并查看结果:

```
[GET] http://localhost:8080/albums
```

## 5.全局自定义配置

通常情况下，默认配置是不够的。出于这个原因，我们需要根据我们的用例创建带有定制配置的断路器。

为了覆盖默认配置，我们需要在一个`@Configuration`类中指定我们自己的 beans 和属性。

这里，我们将为所有断路器定义一个全局配置。由于这个原因，**我们需要定义一个`Customizer<CircuitBreakerFactory>` bean** 。所以让我们使用`Resilience4JCircuitBreakerFactory`实现。

首先，我们将根据 [Resilience4j 教程](/web/20221208143837/https://www.baeldung.com/resilience4j)定义断路器和时间限制器配置类:

```
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
  .failureRateThreshold(50)
  .waitDurationInOpenState(Duration.ofMillis(1000))
  .slidingWindowSize(2)
  .build();
TimeLimiterConfig timeLimiterConfig = TimeLimiterConfig.custom()
  .timeoutDuration(Duration.ofSeconds(4))
  .build();
```

接下来，让我们通过使用`Resilience4JCircuitBreakerFactory.configureDefault`方法将配置嵌入到`Customizer` bean 中:

```
@Configuration
public class Resilience4JConfiguration {
    @Bean
    public Customizer<Resilience4JCircuitBreakerFactory> globalCustomConfiguration() {

        // the circuitBreakerConfig and timeLimiterConfig objects

        return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
          .timeLimiterConfig(timeLimiterConfig)
          .circuitBreakerConfig(circuitBreakerConfig)
          .build());
    } 
}
```

## 6.特定自定义配置

当然，在我们的应用中可以有多个断路器。因此，在某些情况下，我们需要对每个断路器进行特定配置。

类似地，我们可以定义一个或多个`Customizer`bean。然后，我们可以通过使用`Resilience4JCircuitBreakerFactory.configure`方法为每一个提供不同的配置:

```
@Bean
public Customizer<Resilience4JCircuitBreakerFactory> specificCustomConfiguration1() {

    // the circuitBreakerConfig and timeLimiterConfig objects

    return factory -> factory.configure(builder -> builder.circuitBreakerConfig(circuitBreakerConfig)
      .timeLimiterConfig(timeLimiterConfig).build(), "circuitBreaker");
}
```

这里我们提供第二个参数，即我们正在配置的断路器的 id。

我们还可以通过向相同的方法提供断路器 id 列表来设置多个具有相同配置的断路器:

```
@Bean
public Customizer<Resilience4JCircuitBreakerFactory> specificCustomConfiguration2() {

    // the circuitBreakerConfig and timeLimiterConfig objects

    return factory -> factory.configure(builder -> builder.circuitBreakerConfig(circuitBreakerConfig)
      .timeLimiterConfig(timeLimiterConfig).build(),
        "circuitBreaker1", "circuitBreaker2", "circuitBreaker3");
}
```

## 7.替代实现

我们已经看到了如何使用`Resilience4j`实现创建一个或多个带有 Spring Cloud 断路器的断路器。

然而，Spring Cloud 断路器还支持其他实现，我们可以在我们的应用中加以利用:

*   [Hystrix](/web/20221208143837/https://www.baeldung.com/spring-cloud-netflix-hystrix)
*   [哨兵](https://web.archive.org/web/20221208143837/https://github.com/alibaba/Sentinel)
*   [弹簧重试](/web/20221208143837/https://www.baeldung.com/spring-retry)

**值得一提的是，我们可以在应用中混合搭配不同的断路器实施**。我们不仅仅局限于一个图书馆。

上述库比我们在这里探索的功能更多。然而，Spring Cloud 断路器只是断路器部分的抽象。例如，除了本文使用的`CircuitBreaker`和`TimeLimiter`模块之外，Resilience4j 还提供了其他类似`RateLimiter`、`Bulkhead`、`Retry`的模块。

## 8.结论

在本文中，我们发现了 Spring Cloud 断路器项目。

首先，我们了解了什么是 Spring Cloud 断路器，以及它如何允许我们在应用程序中添加断路器。

接下来，我们利用 Spring Boot 自动配置机制来展示如何定义和集成断路器。此外，我们通过一个简单的 REST 服务演示了 Spring Cloud 断路器是如何工作的。

最后，我们学会了一起配置所有断路器，以及单独配置。

和往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-circuit-breaker)