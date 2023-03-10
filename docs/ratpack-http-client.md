# Ratpack HTTP 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack-http-client>

## 1。简介

在过去的几年里，我们见证了用 Java 创建应用程序的函数式和反应式方法的兴起。Ratpack 提供了一种创建 HTTP 应用程序的方法。

由于它使用 Netty 来满足其网络需求，**它是完全异步和非阻塞的**。Ratpack 还通过提供配套的测试库来支持测试。

在本教程中，我们将回顾 Ratpack HTTP 客户端和相关组件的使用。

在这样做的时候，我们将尝试从我们在[介绍性 Ratpack 教程](/web/20220627182323/https://www.baeldung.com/ratpack)结束时离开的地方进一步理解。

## 2。Maven 依赖关系

首先，让我们添加所需的 [Ratpack 依赖关系](https://web.archive.org/web/20220627182323/https://search.maven.org/search?q=io.ratpack):

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.5.4</version>
</dependency>
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-test</artifactId>
    <version>1.5.4</version>
    <scope>test</scope>
</dependency>
```

有趣的是，我们只需要这么多来创建和测试我们的应用程序。

然而，我们总是可以选择使用其他 Ratpack 库来添加和扩展。

## 3。背景

在开始之前，让我们先了解一下 Ratpack 应用程序的工作原理。

### 3.1。基于处理程序的方法

Ratpack 使用基于处理程序的方法来处理请求。这个想法本身很简单。

最简单的形式是，我们可以让每个处理程序在每个特定的路径上处理请求:

```java
public class FooHandler implements Handler {
    @Override
    public void handle(Context ctx) throws Exception {
        ctx.getResponse().send("Hello Foo!");
    }
}
```

### 3.2。链、注册表和上下文

处理程序使用上下文对象与传入的请求进行交互。通过它，我们可以访问 HTTP 请求和响应，并能够委托给其他处理程序。

以下面的处理程序为例:

```java
Handler allHandler = context -> {
    Long id = Long.valueOf(context.getPathTokens().get("id"));
    Employee employee = new Employee(id, "Mr", "NY");
    context.next(Registry.single(Employee.class, employee));
};
```

这个处理程序负责做一些预处理，将结果放入 R `egistry`中，然后将请求委托给其他处理程序。

**通过使用`Registry`，我们可以实现处理器间的通信**。下面的`handler`使用对象类型从`Registry`查询先前计算的结果:

```java
Handler empNameHandler = ctx -> {
    Employee employee = ctx.get(Employee.class);
    ctx.getResponse()
      .send("Name of employee with ID " + employee.getId() + " is " + employee.getName());
};
```

我们应该记住，在生产应用程序中，我们将这些处理程序作为单独的类，以便更好地抽象、调试和开发复杂的业务逻辑。

**现在我们可以在`Chain `中使用这些处理程序来创建复杂的** **定制请求处理管道**。

例如:

```java
Action<Chain> chainAction = chain -> chain.prefix("employee/:id", empChain -> {
    empChain.all(allHandler)
      .get("name", empNameHandler)
      .get("title", empTitleHandler);
});
```

我们可以通过使用`Chain`中的`insert(..)`方法将多个链组合在一起，并让每个链负责不同的关注点，从而进一步发展这种方法。

以下测试案例展示了这些结构的使用:

```java
@Test
public void givenAnyUri_GetEmployeeFromSameRegistry() throws Exception {
    EmbeddedApp.fromHandlers(chainAction)
      .test(testHttpClient -> {
          assertEquals("Name of employee with ID 1 is NY", testHttpClient.get("employee/1/name")
            .getBody()
            .getText());
          assertEquals("Title of employee with ID 1 is Mr", testHttpClient.get("employee/1/title")
            .getBody()
            .getText());
      });
}
```

这里，我们使用 Ratpack 的测试库来单独测试我们的功能，并且不需要启动实际的服务器。

## 4。带 Ratpack 的 HTTP】

### 4.1。努力实现异步

HTTP 协议本质上是同步的。因此，web 应用程序通常是同步的，因此是阻塞的。这是一种非常耗费资源的方法，因为我们为每个传入的请求创建一个线程。

我们宁愿创建非阻塞的异步应用程序。这将确保我们只需要使用少量的线程池来处理请求。

### 4.2。回调函数

在处理异步 API 时，我们通常会向接收方提供一个回调函数，以便将数据返回给调用方。在 Java 中，这通常采用匿名内部类和 lambda 表达式的形式。但是当我们的应用程序扩展时，或者当有多个嵌套的异步调用时，这样的解决方案将很难维护和调试。

Ratpack 以`Promise`的形式提供了一个优雅的解决方案来处理这种复杂性。

### 4.3。Ratpack 承诺

Ratpack `Promise`可以被认为类似于 Java `Future`对象。**它本质上是一个以后可用的值的表示。**

我们可以指定一个操作管道，该值在变得可用时将通过该管道。每个操作都将返回一个新的 promise 对象，即前一个 promise 对象的转换版本。

正如我们所料，这导致线程间的上下文切换很少，并使我们的应用程序高效。

下面是一个利用了`Promise`的处理器实现:

```java
public class EmployeeHandler implements Handler {
    @Override
    public void handle(Context ctx) throws Exception {
        EmployeeRepository repository = ctx.get(EmployeeRepository.class);
        Long id = Long.valueOf(ctx.getPathTokens().get("id"));
        Promise<Employee> employeePromise = repository.findEmployeeById(id);
        employeePromise.map(employee -> employee.getName())
          .then(name -> ctx.getResponse()
          .send(name));
    }
}
```

我们需要记住，当我们定义如何处理最终值时， **`promise`特别有用。我们可以通过在其上调用终端操作`then(Action) `来实现。**

如果我们需要发回一个承诺，但是数据源是同步的，我们仍然可以这样做:

```java
@Test
public void givenSyncDataSource_GetDataFromPromise() throws Exception {
    String value = ExecHarness.yieldSingle(execution -> Promise.sync(() -> "Foo"))
      .getValueOrThrow();
    assertEquals("Foo", value);
}
```

### 4.4。HTTP 客户端

Ratpack 提供了一个异步 HTTP 客户端，可以从服务器注册表中检索到它的一个实例。然而，**我们鼓励创建和使用替代实例，因为默认实例不使用连接池，并且具有相当保守的默认值。**

我们可以使用`of(Action)`方法创建一个实例，该方法将类型为`HttpClientSpec.` 的`Action`作为参数

利用这一点，我们可以根据我们的偏好调整我们的客户端:

```java
HttpClient httpClient = HttpClient.of(httpClientSpec -> {
    httpClientSpec.poolSize(10)
      .connectTimeout(Duration.of(60, ChronoUnit.SECONDS))
      .maxContentLength(ServerConfig.DEFAULT_MAX_CONTENT_LENGTH)
      .responseMaxChunkSize(16384)
      .readTimeout(Duration.of(60, ChronoUnit.SECONDS))
      .byteBufAllocator(PooledByteBufAllocator.DEFAULT);
});
```

正如我们可能已经猜到的异步特性，`HttpClient`返回一个`Promise`对象。因此，我们可以以非阻塞的方式拥有一个复杂的操作管道。

举例来说，让一个客户端使用这个`HttpClient`调用我们的`EmployeeHandler`:

```java
public class RedirectHandler implements Handler {

    @Override
    public void handle(Context ctx) throws Exception {
        HttpClient client = ctx.get(HttpClient.class);
        URI uri = URI.create("http://localhost:5050/employee/1");
        Promise<ReceivedResponse> responsePromise = client.get(uri);
        responsePromise.map(response -> response.getBody()
          .getText()
          .toUpperCase())
          .then(responseText -> ctx.getResponse()
            .send(responseText));
    }
}
```

一个快速的 [cURL](/web/20220627182323/https://www.baeldung.com/curl-rest) 呼叫会确认我们得到了预期的响应:

```java
curl http://localhost:5050/redirect
JANE DOE
```

## 5。结论

在本文中，我们回顾了 Ratpack 中可用的主要库结构，这些库结构使我们能够开发非阻塞的异步 web 应用程序。

我们看了一下 Ratpack `HttpClient `和附带的`Promise `类，它代表了 Ratpack 中所有的异步事物。我们还看到了如何使用附带的`TestHttpClient`轻松测试我们的 HTTP 应用程序。

和往常一样，本教程的代码片段可以在我们的 [GitHub 库](https://web.archive.org/web/20220627182323/https://github.com/eugenp/tutorials/tree/master/ratpack)中找到。