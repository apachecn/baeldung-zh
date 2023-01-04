# 为 Spring REST API 设置请求超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-timeout>

## 1.概观

在本教程中，我们将探索几种可能的方法来实现 Spring REST API 的请求超时。

然后我们将讨论每种方法的优点和缺点。请求超时对于防止糟糕的用户体验很有用，尤其是当资源占用时间太长时，我们可以默认使用其他方法。这种设计模式被称为[断路器模式](/web/20220727020632/https://www.baeldung.com/spring-cloud-circuit-breaker)，但我们不会在这里详细说明。

## 2.`@Transactional`超时

我们可以在数据库调用上实现请求超时的一种方法是利用 Spring 的`@Transactional`注释。它有一个我们可以设置的`timeout`属性。该属性的默认值为-1，相当于没有任何超时。对于外部配置的超时值，我们必须使用不同的属性，`timeoutString,`来代替。

例如，假设我们将这个超时设置为 30。如果带注释的方法的执行时间超过这个秒数，就会抛出异常。这对于回滚长时间运行的数据库查询可能很有用。

为了看到这一点，我们将编写一个非常简单的 JPA 存储库层，它将表示一个需要很长时间才能完成并导致超时的外部服务。这个 JpaRepository 扩展有一个耗时的方法:

```
public interface BookRepository extends JpaRepository<Book, String> {

    default int wasteTime() {
        Stopwatch watch = Stopwatch.createStarted();

        // delay for 2 seconds
        while (watch.elapsed(SECONDS) < 2) {
          int i = Integer.MIN_VALUE;
          while (i < Integer.MAX_VALUE) {
              i++;
          }
        }
    }
}
```

如果我们在一个超时 1 秒的事务中调用我们的`wasteTime()`方法，超时将在方法完成执行之前过去:

```
@GetMapping("/author/transactional")
@Transactional(timeout = 1)
public String getWithTransactionTimeout(@RequestParam String title) {
    bookRepository.wasteTime();
    return bookRepository.findById(title)
      .map(Book::getAuthor)
      .orElse("No book found for this title.");
}
```

调用这个端点会导致一个 500 HTTP 错误，我们可以将它转换成一个更有意义的响应。它还需要很少的设置来实现。

然而，这种超时解决方案有一些缺点。

首先，它依赖于拥有一个包含 Spring 管理的事务的数据库。第二，它不是全局适用于一个项目，因为注释必须出现在每个需要它的方法或类上。它也不允许亚秒精度。最后，当超时到达时，它不会中断请求，因此请求实体仍然必须等待全部时间。

让我们考虑一些备选方案。

## 3.弹性 4j `TimeLimiter`

Resilience4j 是一个库，**主要管理远程通信的容错。**它的 [`TimeLimiter`模块](https://web.archive.org/web/20220727020632/https://resilience4j.readme.io/docs/timeout)才是我们这里感兴趣的。

首先，我们必须将 [`resilience4j-timelimiter`依赖项](https://web.archive.org/web/20220727020632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.resilience4j%22%20AND%20a%3A%22resilience4j-timelimiter%22)包含在我们的项目中:

```
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-timelimiter</artifactId>
    <version>1.6.1</version>
</dependency>
```

接下来，我们将定义一个简单的`TimeLimiter`，它的超时持续时间为 500 毫秒:

```
private TimeLimiter ourTimeLimiter = TimeLimiter.of(TimeLimiterConfig.custom()
  .timeoutDuration(Duration.ofMillis(500)).build());
```

我们可以很容易地从外部进行配置。

我们可以使用我们的`TimeLimiter`来包装我们的`@Transactional`示例所使用的相同逻辑:

```
@GetMapping("/author/resilience4j")
public Callable<String> getWithResilience4jTimeLimiter(@RequestParam String title) {
    return TimeLimiter.decorateFutureSupplier(ourTimeLimiter, () ->
      CompletableFuture.supplyAsync(() -> {
        bookRepository.wasteTime();
        return bookRepository.findById(title)
          .map(Book::getAuthor)
          .orElse("No book found for this title.");
    }));
}
```

与`@Transactional`解决方案相比，`TimeLimiter`提供了几个优势。也就是说，它支持亚秒精度和超时响应的即时通知。但是，我们仍然必须手动将它包含在所有需要超时的端点中。它还需要一些冗长的包装代码，并且它产生的错误仍然是一般的 500 HTTP 错误。最后，它要求返回一个`Callable<String>`而不是一个原始的`String.`

`TimeLimiter`仅包括来自 Resilience4j 的[特征的子集，并且与断路器模式很好地接口。](/web/20220727020632/https://www.baeldung.com/resilience4j)

## 4.Spring MVC `request-timeout`

Spring 为我们提供了一个名为`spring.mvc.async.request-timeout`的属性。这个属性允许我们定义毫秒级精度的请求超时。

让我们用 750 毫秒的超时来定义该属性:

```
spring.mvc.async.request-timeout=750
```

这个属性是全局的，可以在外部配置，但是像`TimeLimiter`解决方案一样，它只适用于返回`Callable`的端点。让我们定义一个类似于`TimeLimiter`例子的端点，但是不需要在`Futures,`中包装逻辑或者提供一个`TimeLimiter`:

```
@GetMapping("/author/mvc-request-timeout")
public Callable<String> getWithMvcRequestTimeout(@RequestParam String title) {
    return () -> {
        bookRepository.wasteTime();
        return bookRepository.findById(title)
          .map(Book::getAuthor)
          .orElse("No book found for this title.");
    };
}
```

我们可以看到代码不那么冗长，并且当我们定义应用程序属性时，Spring 会自动实现配置。一旦超时，响应会立即返回，甚至会返回更具描述性的 503 HTTP 错误，而不是一般的 500。我们项目中的每个端点都将自动继承这个超时配置。

现在让我们考虑另一个选项，它允许我们以更细粒度的方式定义超时。

## 5.`WebClient` 超时

与其为整个端点设置超时，**我们可能只想为单个外部调用设置超时。** [`WebClient`是 Spring 的反应式 web 客户端](/web/20220727020632/https://www.baeldung.com/spring-5-webclient)，它允许我们配置响应超时。

也可以在 [Spring 的老`RestTemplate`对象](/web/20220727020632/https://www.baeldung.com/rest-template)上配置超时；然而，大多数开发者现在更喜欢`WebClient`而不是`RestTemplate`。

要使用 WebClient，我们必须首先将 [Spring 的 WebFlux 依赖关系](https://web.archive.org/web/20220727020632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-webflux%22)添加到我们的项目中:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
    <version>2.4.2</version>
</dependency>
```

让我们定义一个响应超时为 250 毫秒的`WebClient`,我们可以使用它通过其基本 URL 中的 localhost 调用自己:

```
@Bean
public WebClient webClient() {
    return WebClient.builder()
      .baseUrl("http://localhost:8080")
      .clientConnector(new ReactorClientHttpConnector(
        HttpClient.create().responseTimeout(Duration.ofMillis(250))
      ))
      .build();
}
```

显然，我们可以很容易地从外部配置这个超时值。我们还可以在外部配置基本 URL，以及其他几个可选属性。

现在我们可以将我们的`WebClient`注入到我们的控制器中，并使用它来调用我们自己的`/transactional`端点，它仍然有 1 秒的超时。由于我们将`WebClient`配置为在 250 毫秒内超时，我们应该会看到它比 1 秒更快地失败。

这是我们的新端点:

```
@GetMapping("/author/webclient")
public String getWithWebClient(@RequestParam String title) {
    return webClient.get()
      .uri(uriBuilder -> uriBuilder
        .path("/author/transactional")
        .queryParam("title", title)
        .build())
      .retrieve()
      .bodyToMono(String.class)
      .block();
}
```

在调用这个端点之后，我们可以看到我们确实收到了 500 HTTP 错误响应形式的`WebClient`的超时。我们还可以检查日志来查看下游的`@Transactional`超时，但是如果我们调用外部服务而不是本地主机，它的超时将被远程打印出来。

为不同的后端服务配置不同的请求超时可能是必要的，并且在这个解决方案中是可能的。另外，`WebClient`返回的发布者的`Mono`或`Flux`响应中包含了大量的[错误处理方法](/web/20220727020632/https://www.baeldung.com/spring-webflux-errors)，用于处理一般的超时错误响应。

## 6.结论

在本文中，我们探讨了实现请求超时的几种不同的解决方案。在决定使用哪一种时，有几个因素需要考虑。

如果我们想对数据库请求设置一个超时，我们可能想使用 Spring 的`@Transactional`方法和它的`timeout`属性。如果我们试图与更广泛的断路器模式集成，使用 Resilience4j 的`TimeLimiter`将是有意义的。使用 Spring MVC `request-timeout`属性最适合为所有请求设置全局超时，但是我们也可以使用`WebClient`为每个资源定义更细粒度的超时。

作为所有这些解决方案的一个工作示例，代码已经准备好，可以在 GitHub 上开箱运行。