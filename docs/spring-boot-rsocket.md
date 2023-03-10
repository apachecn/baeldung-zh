# 使用 Spring Boot 的 RSocket

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-rsocket>

## 1.概观

RSocket 是一个提供反应流语义的应用协议，例如，它可以作为 HTTP 的替代协议。

在本教程中，我们将看看使用 [Spring Boot](/web/20221126230003/https://www.baeldung.com/spring-boot-start) 的 [RSocket](/web/20221126230003/https://www.baeldung.com/rsocket) ，特别是它如何帮助抽象出低级别的 RSocket API。

## 2.属国

让我们从添加 [`spring-boot-starter-rsocket`](https://web.archive.org/web/20221126230003/https://mvnrepository.com/search?q=spring-boot-starter-rsocket) 依赖关系开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-rsocket</artifactId>
</dependency>
```

这将传递性地拉入 RSocket 相关的依赖项，如`[rsocket-core](https://web.archive.org/web/20221126230003/https://mvnrepository.com/search?q=rsocket-core)`和 [`rsocket-transport-netty`](https://web.archive.org/web/20221126230003/https://mvnrepository.com/search?q=rsocket-transport-netty) 。

## 3.示例应用程序

现在我们将继续我们的示例应用程序。为了突出 RSocket 提供的交互模型，我们将创建一个交易者应用程序。我们的 trader 应用程序将由一个客户端和一个服务器组成。

### 3.1.服务器设置

首先，让我们设置服务器，这将是一个引导 RSocket 服务器的 Spring Boot 应用程序。

**因为我们有了 [`spring-boot-starter-rsocket`](https://web.archive.org/web/20221126230003/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-rsocket/2.2.0.M2) 依赖项，Spring Boot 自动为我们配置了一个 RSocket 服务器。**和 Spring Boot 一样，我们可以以属性驱动的方式更改 RSocket 服务器的默认配置值。

例如，让我们通过向我们的`application.properties` 文件添加下面一行来更改我们的 RSocket 服务器的端口:

```java
spring.rsocket.server.port=7000
```

我们还可以改变其他属性来根据我们的需要进一步修改我们的服务器。

### 3.2.客户端设置

接下来，让我们设置客户端，它也将是一个 Spring Boot 应用程序。

虽然 Spring Boot 自动配置了大部分与 RSocket 相关的组件，但是我们还应该定义一些 beans 来完成设置:

```java
@Configuration
public class ClientConfiguration {

    @Bean
    public RSocketRequester getRSocketRequester(){
        RSocketRequester.Builder builder = RSocketRequester.builder();

        return builder
          .rsocketConnector(
             rSocketConnector ->
               rSocketConnector.reconnect(Retry.fixedDelay(2, Duration.ofSeconds(2)))
          )
          .dataMimeType(MimeTypeUtils.APPLICATION_JSON)
          .tcp("localhost", 7000);
    }
}
```

这里我们创建了`RSocket`客户端，并将其配置为在端口 7000 上使用 TCP 传输。请注意，这是我们之前配置的服务器端口。

定义了这个 bean 配置之后，我们就有了一个基本的结构。

接下来，**我们将探索不同的互动模式**，看看 Spring Boot 如何帮助我们。

## 4.RSocket 和 Spring Boot 的请求/响应

先说请求/响应。这可能是最常见和熟悉的交互模型，因为 HTTP 也采用这种类型的通信。

在这个交互模型中，客户端发起通信并发送请求。随后，服务器执行操作并向客户端返回响应——这样通信就完成了。

在我们的 trader 应用程序中，客户将请求给定股票的当前市场数据。作为回报，服务器将传递请求的数据。

### 4.1.计算机网络服务器

在服务器端，我们应该首先创建一个控制器来保存我们的处理程序方法。**但是不像 Spring MVC 中的 [`@RequestMapping`](/web/20221126230003/https://www.baeldung.com/spring-requestmapping) 或 [`@GetMapping`](/web/20221126230003/https://www.baeldung.com/spring-new-requestmapping-shortcuts) 注释，我们将使用`@MessageMapping`注释**:

```java
@Controller
public class MarketDataRSocketController {

    private final MarketDataRepository marketDataRepository;

    public MarketDataRSocketController(MarketDataRepository marketDataRepository) {
        this.marketDataRepository = marketDataRepository;
    }

    @MessageMapping("currentMarketData")
    public Mono<MarketData> currentMarketData(MarketDataRequest marketDataRequest) {
        return marketDataRepository.getOne(marketDataRequest.getStock());
    }
}
```

所以让我们调查一下我们的控制器。

我们使用@ `Controller`注释来定义一个处理程序，它应该处理传入的 RSocket 请求。此外，`@MessageMapping`注释让我们定义我们感兴趣的路由以及如何对请求做出反应。

在这种情况下，服务器监听`currentMarketData`路由，该路由**将单个结果作为** **`Mono<MarketData>`** 返回给客户端。

### 4.2.客户

接下来，我们的 RSocket 客户机应该请求一只股票的当前价格，并得到一个响应。

要发起请求，我们应该使用`RSocketRequester`类:

```java
@RestController
public class MarketDataRestController {

    private final RSocketRequester rSocketRequester;

    public MarketDataRestController(RSocketRequester rSocketRequester) {
        this.rSocketRequester = rSocketRequester;
    }

    @GetMapping(value = "/current/{stock}")
    public Publisher<MarketData> current(@PathVariable("stock") String stock) {
        return rSocketRequester
          .route("currentMarketData")
          .data(new MarketDataRequest(stock))
          .retrieveMono(MarketData.class);
    }
}
```

注意，在我们的例子中，RSocket 客户机也是一个 REST 控制器，我们从它调用 RSocket 服务器。因此，我们使用`@RestController`和`@GetMapping`来定义我们的请求/响应端点。

在端点方法中，我们使用`RSocketRequester`并指定路线。事实上，这是 RSocket 服务器所期望的路径。然后我们传递请求数据。最后，**当我们调用`retrieveMono()`方法时，Spring Boot 发起一个请求/响应交互**。

## 5.与`RSocket`和 Spring Boot 一刀两断

接下来，我们将看看“一发不可收拾”的交互模型。顾名思义，客户机向服务器发送一个请求，但不期望得到响应。

在我们的 trader 应用程序中，一些客户端将充当数据源，并将市场数据推送到服务器。

### 5.1.计算机网络服务器

让我们在服务器应用程序中创建另一个端点:

```java
@MessageMapping("collectMarketData")
public Mono<Void> collectMarketData(MarketData marketData) {
    marketDataRepository.add(marketData);
    return Mono.empty();
}
```

同样，我们用路由值`collectMarketData`定义一个新的`@MessageMapping`。此外，Spring Boot 自动将传入的有效载荷转换成一个`MarketData`实例。

然而，这里最大的不同是，**我们返回一个`Mono<Void>` ,因为客户端不需要我们的响应。**

### 5.2.客户

让我们看看如何发起一次性请求。

我们将创建另一个 REST 端点:

```java
@GetMapping(value = "/collect")
public Publisher<Void> collect() {
    return rSocketRequester
      .route("collectMarketData")
      .data(getMarketData())
      .send();
}
```

这里我们指定了我们的路线，我们的有效载荷将是一个`MarketData`实例。**因为我们使用`send()`方法而不是`retrieveMono()`来发起请求，所以交互模型变成了“一劳永逸”**。

## 6.带有 RSocket 和 Spring Boot 的请求流

请求流是一种更复杂的交互模型，在这种模型中，客户端发送一个请求，但随着时间的推移，会从服务器获得多个响应。

为了模拟这种交互模型，一个客户将要求一个给定股票的所有市场数据。

### 6.1.计算机网络服务器

让我们从我们的服务器开始。我们将添加另一个消息映射方法:

```java
@MessageMapping("feedMarketData")
public Flux<MarketData> feedMarketData(MarketDataRequest marketDataRequest) {
    return marketDataRepository.getAll(marketDataRequest.getStock());
}
```

正如我们所看到的，这个处理程序方法与其他方法非常相似。不同的部分是，**我们返回了一个`Flux<MarketData>`而不是一个`Mono<MarketData>`T3。最后，我们的 RSocket 服务器将向客户机发送多个响应。**

### 6.2.客户

在客户端，我们应该创建一个端点来启动我们的请求/流通信:

```java
@GetMapping(value = "/feed/{stock}", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Publisher<MarketData> feed(@PathVariable("stock") String stock) {
    return rSocketRequester
      .route("feedMarketData")
      .data(new MarketDataRequest(stock))
      .retrieveFlux(MarketData.class);
}
```

让我们调查一下我们的 RSocket 请求。

首先，我们定义路由并请求有效负载。**然后，我们用`retrieveFlux()`方法调用**来定义我们的响应期望。这是决定交互模型的部分。

还要注意，由于我们的客户机也是 REST 服务器，它将响应媒体类型定义为`MediaType.TEXT_EVENT_STREAM_VALUE.`

## 7.异常处理

现在让我们看看如何以声明的方式处理服务器应用程序中的异常。

当进行请求/响应时，我们可以简单地使用`@MessageExceptionHandler` 注释:

```java
@MessageExceptionHandler
public Mono<MarketData> handleException(Exception e) {
    return Mono.just(MarketData.fromException(e));
}
```

这里我们用`@MessageExceptionHandler`注释了我们的异常处理方法。因此，它将处理所有类型的异常，因为`Exception`类是所有其他类的超类。

我们可以更具体一些，为不同的异常类型创建不同的异常处理程序方法。

这当然是针对请求/响应模型的，所以我们返回一个`Mono<MarketData>.` **我们希望这里的返回类型与交互模型的返回类型相匹配。**

## 8.摘要

在本教程中，我们已经介绍了 Spring Boot 的 RSocket 支持，并详细介绍了 RSocket 提供的不同交互模型。

和往常一样，你可以在 GitHub 上查看所有的代码样本[。](https://web.archive.org/web/20221126230003/https://github.com/eugenp/tutorials/tree/master/spring-5-webflux)