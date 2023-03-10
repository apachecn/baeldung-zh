# 带 Hystrix 的鼠夹

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-hystrix>

## 1。简介

之前，我们已经向展示了如何使用 Ratpack 构建高性能的反应式应用。

在本文中，我们将了解如何将网飞·海斯特里克斯与 Ratpack 应用程序集成在一起。

网飞海斯特里克斯通过隔离接入点来阻止级联故障，并为容错提供回退选项，从而帮助控制分布式服务之间的交互。它可以帮助我们构建一个更有弹性的应用程序。查看我们对 Hystrix 的[介绍，进行快速回顾。](/web/20221128104429/https://www.baeldung.com/introduction-to-hystrix)

这就是我们将如何使用它——我们将使用 Hystrix 提供的这些有用功能来增强我们的 Ratpack 应用程序。

## 2。Maven 依赖关系

要将 Hystrix 与 Ratpack 一起使用，我们需要项目`pom.xml`中的 ratpack-hystrix 依赖关系:

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-hystrix</artifactId>
    <version>1.4.6</version>
</dependency>
```

ratpack-hystrix 的最新版本可以在[这里](https://web.archive.org/web/20221128104429/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22ratpack-hystrix%22%20AND%20g%3A%22io.ratpack%22)找到。棘轮套包括[棘轮套芯](https://web.archive.org/web/20221128104429/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-core%22)和[棘轮套芯](https://web.archive.org/web/20221128104429/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.netflix.hystrix%22%20AND%20a%3A%22hystrix-core%22)。

为了利用 Ratpack 的反应特性，我们还需要 ratpack-rx:

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-rx</artifactId>
    <version>1.4.6</version>
</dependency>
```

ratpack-rx 的最新版本可以在[这里](https://web.archive.org/web/20221128104429/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-rx%22)找到。

## 3。使用 Hystrix 命令服务

使用 Hystrix 时，底层服务通常被包装在 [`HystrixCommand`](https://web.archive.org/web/20221128104429/https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html) 或 [`HystrixObservableCommand`](https://web.archive.org/web/20221128104429/https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixObservableCommand.html) 中。Hystrix 支持以同步、异步和反应的方式执行这些命令。其中，只有 reactive 是非阻塞的，是官方推荐的。

在下面的例子中，我们将构建一些从 [Github REST API](https://web.archive.org/web/20221128104429/https://docs.github.com/en/rest) 获取概要文件的端点。

### 3.1。无功命令执行

首先，让我们用 Hystrix 构建一个反应式后端服务:

```java
public class HystrixReactiveHttpCommand extends HystrixObservableCommand<String> {

    //...

    @Override
    protected Observable<String> construct() {
        return RxRatpack.observe(httpClient
          .get(uri, r -> r.headers(h -> h.add("User-Agent", "Baeldung HttpClient")))
          .map(res -> res.getBody().getText()));
    }

    @Override
    protected Observable<String> resumeWithFallback() {
        return Observable.just("eugenp's reactive fallback profile");
    }
}
```

这里用一个 Ratpack reactive [`HttpClient`](https://web.archive.org/web/20221128104429/https://hc.apache.org/httpcomponents-client-5.1.x/index.html) 来做一个 GET 请求。`HystrixReactiveHttpCommand`可以作为一个反应处理器:

```java
chain.get("rx", ctx -> 
  new HystrixReactiveHttpCommand(
    ctx.get(HttpClient.class), eugenGithubProfileUri, timeout)
    .toObservable()
    .subscribe(ctx::render));
```

可通过以下测试验证终点:

```java
@Test
public void whenFetchReactive_thenGotEugenProfile() {
    assertThat(appUnderTest.getHttpClient().getText("rx"), 
      containsString("www.baeldung.com"));
}
```

### 3.2。异步命令执行

`HystrixCommand`的异步执行将命令在线程池上排队，并返回一个`Future`:

```java
chain.get("async", ctx -> ctx.render(
  new HystrixAsyncHttpCommand(eugenGithubProfileUri, timeout)
    .queue()
    .get()));
```

`HystrixAsyncHttpCommand`看起来像是:

```java
public class HystrixAsyncHttpCommand extends HystrixCommand<String> {

    //...

    @Override
    protected String run() throws Exception {
        return EntityUtils.toString(HttpClientBuilder.create()
          .setDefaultRequestConfig(requestConfig)
          .setDefaultHeaders(Collections.singleton(
            new BasicHeader("User-Agent", "Baeldung Blocking HttpClient")))
          .build().execute(new HttpGet(uri)).getEntity());
    }

    @Override
    protected String getFallback() {
        return "eugenp's async fallback profile";
    }

}
```

这里我们使用阻塞的 [`HttpClient`](https://web.archive.org/web/20221128104429/https://hc.apache.org/httpcomponents-client-5.1.x/examples.html) 而不是非阻塞的，因为我们希望 Hystrix 控制实际命令的执行超时，这样我们在从`Future`获得响应时就不需要自己处理了。这也允许 Hystrix 回退或缓存我们的请求。

异步执行也会产生预期的结果:

```java
@Test
public void whenFetchAsync_thenGotEugenProfile() {
    assertThat(appUnderTest.getHttpClient().getText("async"),
      containsString("www.baeldung.com"));
}
```

### 3.3。同步命令执行

同步执行直接在当前线程中执行命令:

```java
chain.get("sync", ctx -> ctx.render(
  new HystrixSyncHttpCommand(eugenGithubProfileUri, timeout).execute()));
```

除了我们给它一个不同的回退结果外，`HystrixSyncHttpCommand`的实现与`HystrixAsyncHttpCommand`几乎相同。当不回退时，它的行为与被动和异步执行完全相同:

```java
@Test
public void whenFetchSync_thenGotEugenProfile() {
    assertThat(appUnderTest.getHttpClient().getText("sync"),
      containsString("www.baeldung.com"));
}
```

## 4.韵律学

通过将 [Guice 模块](/web/20221128104429/https://www.baeldung.com/ratpack-google-guice)–[`HystrixModule`](https://web.archive.org/web/20221128104429/https://ratpack.io/manual/current/api/ratpack/hystrix/HystrixModule.html)注册到 Ratpack registry 中，我们可以流式传输请求范围的度量，并通过`GET`端点公开事件流:

```java
serverSpec.registry(
  Guice.registry(spec -> spec.module(new HystrixModule().sse())))
  .handlers(c -> c.get("hystrix", new HystrixMetricsEventStreamHandler()));
```

`[HystrixMetricsEventStreamHandler](https://web.archive.org/web/20221128104429/https://ratpack.io/manual/current/api/ratpack/hystrix/HystrixMetricsEventStreamHandler.html)`帮助以`text/event-stream`格式传输 Hystrix 度量，这样我们可以在 [`Hystrix Dashboard`](https://web.archive.org/web/20221128104429/https://github.com/Netflix-Skunkworks/hystrix-dashboard) 中监控度量。

我们可以设置一个[独立的 Hystrix 仪表板](https://web.archive.org/web/20221128104429/https://github.com/kennedyoliveira/standalone-hystrix-dashboard)，并将我们的 Hystrix 事件流添加到监视器列表中，以查看我们的 Ratpack 应用程序的执行情况:

[![Snip20170815 1](img/0a085625787ac6f797d66a037ef7f2a3.png)](/web/20221128104429/https://www.baeldung.com/wp-content/uploads/2017/08/Snip20170815_1.png)

在对我们的 Ratpack 应用程序进行了几次请求之后，我们可以在仪表板中看到与 Hystrix 相关的命令。

### 4.1。引擎盖下

在`HystrixModule`中， [Hystrix 并发策略](https://web.archive.org/web/20221128104429/https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/strategy/concurrency/HystrixConcurrencyStrategy.html)通过 [HystrixPlugin](https://web.archive.org/web/20221128104429/https://github.com/Netflix/Hystrix/wiki/Plugins) 向 Hystrix 注册，以管理 Ratpack 注册表的请求上下文。这消除了在每个请求开始之前初始化 Hystrix 请求上下文的必要性。

```java
public class HystrixModule extends ConfigurableModule<HystrixModule.Config> {

    //...

    @Override
    protected void configure() {
      try {
        HystrixPlugins.getInstance().registerConcurrencyStrategy(
          new HystrixRegistryBackedConcurrencyStrategy());
      } catch (IllegalStateException e) {
        //...
      }
    }

    //...

}
```

## 5。结论

在这篇简短的文章中，我们展示了如何将 Hystrix 集成到 Ratpack 中，以及如何将我们的 Ratpack 应用程序的指标推送到 Hystrix Dashboard，以便更好地查看应用程序性能。

和往常一样，完整的实现可以在 Github 项目中找到。