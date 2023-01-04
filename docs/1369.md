# Vert.x 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vertx>

## 1。概述

在本文中，我们将讨论 [Vert.x](https://web.archive.org/web/20221128103740/http://vertx.io/) ，涵盖其核心概念，并使用它创建一个简单的 RESTfull web 服务。

我们将从介绍关于工具包的基础概念开始，慢慢前进到 HTTP 服务器，然后构建 RESTfull 服务。

## 2。关于 Vert.x

Vert.x 是一个开源的、反应式的、多语言的软件开发工具包，由 Eclipse 的开发者开发。

反应式编程是一种编程范式，与异步流相关联，对任何变化或事件做出响应。

类似地，Vert.x 使用事件总线与应用程序的不同部分进行通信，并将事件异步传递给可用的处理程序。

我们称它为 polyglot，因为它支持多种 JVM 和非 JVM 语言，如 Java、Groovy、Ruby、Python 和 JavaScript。

## 3。设置

要使用 Vert.x，我们需要添加 Maven 依赖项:

```
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-core</artifactId>
    <version>3.4.1</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20221128103740/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.vertx%22%20a%3A%22vertx-core%22)找到。

## 3。垂直线

垂直是 Vert.x 引擎执行的代码片段。该工具包为我们提供了许多抽象的垂直类，可以根据我们的需要进行扩展和实现。

作为多语言用户，verticles 可以用任何支持的语言编写。一个应用程序通常由运行在同一个 Vert.x 实例中的多个 verticles 组成，并通过事件总线使用事件相互通信。

要在 JAVA 中创建一个 verticle，这个类必须实现`io.vertx.core.Verticle` 接口，或者它的任何一个子类。

## 4。事件总线

它是任何 Vert.x 应用程序的神经系统。

由于是反应性的，垂直设备保持休眠状态，直到它们接收到消息或事件。垂直设备通过事件总线相互通信。消息可以是从字符串到复杂对象的任何内容。

理想情况下，消息处理是异步的，消息排队进入事件总线，控制权返回给发送方。稍后，它会出队到垂直监听。使用`Future` 和`callback` 方法发送响应。

## 5。简单的 Vert.x 应用程序

**让我们创建一个简单的应用程序，使用一个`vertx`实例**来部署它。为了创建我们的垂直市场，我们将扩展

为了创建我们的 verticle，我们将扩展`io.vertx.core.AbstractVerticle` 类并覆盖`start()` 方法:

```
public class HelloVerticle extends AbstractVerticle {

    @Override
    public void start(Future<Void> future) {
        LOGGER.info("Welcome to Vertx");
    }
}
```

部署 verticle 时，`vertx` 实例将调用`start()` 方法。该方法将`io.vertx.core.Future`作为一个参数，可以用来发现垂直的异步部署的状态。

现在让我们部署垂直市场:

```
public static void main(String[] args) {
    Vertx vertx = Vertx.vertx();
    vertx.deployVerticle(new HelloVerticle());
}
```

类似地，我们可以覆盖来自`AbstractVerticle` 类的`stop()` 方法，该方法将在关闭 verticle 时被调用:

```
@Override
public void stop() {
    LOGGER.info("Shutting down application");
}
```

## 6。HTTP 服务器

现在，让我们使用一个垂直设备启动一个 HTTP 服务器:

```
@Override
public void start(Future<Void> future) {
    vertx.createHttpServer()
      .requestHandler(r -> r.response().end("Welcome to Vert.x Intro");
      })
      .listen(config().getInteger("http.port", 9090), 
        result -> {
          if (result.succeeded()) {
              future.complete();
          } else {
              future.fail(result.cause());
          }
      });
}
```

我们已经覆盖了`start()` 方法来创建一个 HTTP 服务器，并为它附加了一个请求处理程序。每当服务器收到请求时，就会调用`requestHandler()`方法。

最后，服务器被绑定到一个端口，在出现任何错误的情况下，无论连接或服务器启动是否成功，都使用`future.complete()` 或`future.fail()`将一个`AsyncResult<HttpServer>` 处理程序传递给`listen()`方法。

注意:`config.getInteger()` 方法正在读取从外部`conf.json`文件加载的 HTTP 端口配置值。

让我们测试我们的服务器:

```
@Test
public void whenReceivedResponse_thenSuccess(TestContext testContext) {
    Async async = testContext.async();

    vertx.createHttpClient()
      .getNow(port, "localhost", "/", response -> {
        response.handler(responseBody -> {
          testContext.assertTrue(responseBody.toString().contains("Hello"));
          async.complete();
        });
      });
}
```

对于测试，让我们将 vertx-unit 与 JUnit 一起使用。：

```
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-unit</artifactId>
    <version>3.4.1</version>
    <scope>test</scope>
</dependency>
```

我们可以在这里获得最新版本[。](https://web.archive.org/web/20221128103740/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.vertx%22%20a%3A%22vertx-unit%22)

verticle 被部署在单元测试的`setup()`方法的`vertx`实例中:

```
@Before
public void setup(TestContext testContext) {
    vertx = Vertx.vertx();

    vertx.deployVerticle(SimpleServerVerticle.class.getName(), 
      testContext.asyncAssertSuccess());
}
```

类似地，`vertx` 实例在`@AfterClass tearDown()` 方法中关闭:

```
@After
public void tearDown(TestContext testContext) {
    vertx.close(testContext.asyncAssertSuccess());
}
```

注意，`@BeforeClass setup()` 方法带有一个`TestContext` 参数。这有助于控制和测试测试的异步行为。例如，垂直部署是异步的，所以基本上我们不能测试任何东西，除非它被正确部署。

我们有第二个参数给`deployVerticle()` 方法，`testContext.asyncAssertSuccess().` `T`这个参数用于知道服务器是否被正确部署或者是否发生了任何故障。它等待服务器中的`future.complete() or future.fail()` 被调用。在失败的情况下，它没有通过测试。

## 7。RESTful 服务

我们已经创建了一个 HTTP 服务器，现在让我们用它来托管一个 RESTfull WebService。为此，我们将需要另一个名为 **vertx-web** 的 Vert.x 模块。这为基于 **vertx-core** 的 web 开发提供了许多额外的特性。

让我们将依赖项添加到我们的`pom.xml:`

```
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web</artifactId>
    <version>3.4.1</version>
</dependency>
```

我们可以在这里找到最新版本[。](https://web.archive.org/web/20221128103740/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.vertx%22%20a%3A%22vertx-web%22)

### 7.1。`Router`和`Routes`

让我们为我们的 web 服务创建一个`router` 。这个路由器将采用 GET 方法和 handler 方法`getArtilces()`的简单路线:

```
Router router = Router.router(vertx);
router.get("/api/baeldung/articles/article/:id")
  .handler(this::getArticles);
```

`getArticle()` 方法是一个简单的方法，返回新的`Article`对象:

```
private void getArticles(RoutingContext routingContext) {
    String articleId = routingContext.request()
      .getParam("id");
    Article article = new Article(articleId, 
      "This is an intro to vertx", "baeldung", "01-02-2017", 1578);

    routingContext.response()
      .putHeader("content-type", "application/json")
      .setStatusCode(200)
      .end(Json.encodePrettily(article));
}
```

A `Router,` 在收到请求时，寻找匹配的路由，并进一步传递请求。`routes` 有一个与之关联的处理程序方法来处理请求。

在我们的例子中，处理程序调用`getArticle()` 方法。它接收`routingContext` 对象作为参数。导出路径参数`id,` ，并用它创建一个`Article` 对象。

在该方法的最后一部分，让我们调用`routingContext`对象上的`response()` 方法，放置头部，设置 HTTP 响应代码，并使用 JSON 编码的`article` 对象结束响应。

### 7.2。将`Router`添加到服务器

现在让我们将上一节中创建的`router,` 添加到 HTTP 服务器:

```
vertx.createHttpServer()
  .requestHandler(router::accept)
  .listen(config().getInteger("http.port", 8080), 
    result -> {
      if (result.succeeded()) {
          future.complete();
      } else {
          future.fail(result.cause());
      }
});
```

请注意，我们已经向服务器添加了`requestHandler(router::accept)` 。这指示服务器在收到任何请求时调用`router`对象的`accept()` 。

现在让我们测试我们的 web 服务:

```
@Test
public void givenId_whenReceivedArticle_thenSuccess(TestContext testContext) {
    Async async = testContext.async();

    vertx.createHttpClient()
      .getNow(8080, "localhost", "/api/baeldung/articles/article/12345", 
        response -> {
            response.handler(responseBody -> {
            testContext.assertTrue(
              responseBody.toString().contains("\"id\" : \"12345\""));
            async.complete();
        });
      });
}
```

## 8。打包 Vert.x 应用程序

将应用程序打包成可部署的 Java 档案文件(。jar)让我们使用 Maven Shade 插件和`execution`标签中的配置:

```
<configuration>
    <transformers>
        <transformer 
          implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <manifestEntries>
                <Main-Class>io.vertx.core.Starter</Main-Class>
                <Main-Verticle>com.baeldung.SimpleServerVerticle</Main-Verticle>
            </manifestEntries>
        </transformer>
    </transformers>
    <artifactSet />
    <outputFile>
        ${project.build.directory}/${project.artifactId}-${project.version}-app.jar
    </outputFile>
</configuration>
```

在`manifestEntries,` `Main-Verticle`中表示应用程序的起点，`Main-Class`是一个 Vert.x 类，它创建`vertx` 实例并部署`Main-Verticle.`

## 9。结论

在这篇介绍性文章中，我们讨论了 Vert.x 工具包及其基本概念。看到了如何使用 Vert.x 和 RESTFull WebService 创建 HTTP 服务器，并展示了如何使用`vertx-unit`测试它们。

最后将应用程序打包成一个可执行的 jar。

GitHub 上的[提供了代码片段的完整实现。](https://web.archive.org/web/20221128103740/https://github.com/eugenp/tutorials/tree/master/vertx-modules/vertx)