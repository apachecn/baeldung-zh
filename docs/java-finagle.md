# 终曲介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-finagle>

## 1.概观

在本教程中，我们将快速浏览一下 Twitter 的 RPC 库 Finagle。

我们将用它来构建一个简单的客户机和服务器。

## 2.积木

在我们深入研究实现之前，我们需要了解我们将用来构建应用程序的基本概念。它们广为人知，但在 Finagle 的世界里可能有稍微不同的含义。

### 2.1.服务

服务是由接受请求并返回包含操作最终结果或失败信息的`Future`的类表示的函数。

### 2.2.过滤

过滤器也是功能。他们接受一个请求和一个服务，对请求进行一些操作，将请求传递给服务，对结果`Future`进行一些操作，最后返回最终的`Future`。我们可以将它们视为[方面](/web/20220627083557/https://www.baeldung.com/spring-aop)，因为它们可以实现在函数执行过程中发生的逻辑，并改变其输入和输出。

### 2.3.期货

期货代表异步操作的最终结果。它们可能处于以下三种状态之一:挂起、成功或失败。

## 3.服务

首先，我们将实现一个简单的 HTTP 问候服务。它将从请求中获取 name 参数并做出响应，然后添加惯用的“Hello”消息。

为此，我们需要创建一个类，该类将扩展来自 Finagle 库的抽象类`Service`，**实现其`apply`方法。**

我们所做的看起来类似于实现一个[功能接口](/web/20220627083557/https://www.baeldung.com/java-8-functional-interfaces)。不过有趣的是，我们实际上不能使用那个特定的特性，因为 Finagle 是用 Scala 编写的，我们正在利用 Java-Scala 的互操作性:

```java
public class GreetingService extends Service<Request, Response> {
    @Override
    public Future<Response> apply(Request request) {
        String greeting = "Hello " + request.getParam("name");
        Reader<Buf> reader = Reader.fromBuf(new Buf.ByteArray(greeting.getBytes(), 0, greeting.length()));
        return Future.value(Response.apply(request.version(), Status.Ok(), reader));
    }
}
```

## 4.过滤器

接下来，我们将编写一个过滤器，记录一些关于控制台请求的数据。类似于`Service`，我们将需要实现`Filter`的`apply`方法，该方法将接受请求并返回一个`Future`响应，但这次它也将服务作为第二个参数。

基本的`[Filter](https://web.archive.org/web/20220627083557/https://twitter.github.io/finagle/docs/com/twitter/finagle/Filter.html)` 类有四个类型参数，但是通常我们不需要改变过滤器中请求和响应的类型。

为此，我们将使用`[SimpleFilter](https://web.archive.org/web/20220627083557/https://twitter.github.io/finagle/docs/com/twitter/finagle/SimpleFilter.html)`将四个类型参数合并成两个。我们将打印请求中的一些信息，然后简单地从提供的服务中调用`apply`方法:

```java
public class LogFilter extends SimpleFilter<Request, Response> {
    @Override
    public Future apply(Request request, Service<Request, Response> service) {
        logger.info("Request host:" + request.host().getOrElse(() -> ""));
        logger.info("Request params:");
        request.getParams().forEach(entry -> logger.info("\t" + entry.getKey() + " : " + entry.getValue()));
        return service.apply(request);
    }
} 
```

## 5.计算机网络服务器

现在，我们可以使用服务和过滤器来构建一个服务器，该服务器将实际监听请求并处理它们。

我们将为该服务器提供一个服务，该服务包含我们的过滤器和使用`andThen`方法链接在一起的服务:

```java
Service serverService = new LogFilter().andThen(new GreetingService()); 
Http.serve(":8080", serverService);
```

## 6.客户

最后，我们需要一个客户端向我们的服务器发送请求。

为此，我们将使用 Finagle 的`[Http](https://web.archive.org/web/20220627083557/https://twitter.github.io/finagle/docs/com/twitter/finagle/Http$.html)`类中方便的`newService`方法创建一个 HTTP 服务。它将直接负责发送请求。

此外，我们将使用之前实现的相同日志过滤器，并将其与 HTTP 服务链接起来。然后，我们只需要调用`apply`方法。

**最后一个操作是异步的，其最终结果存储在`Future`实例中。**我们可以等待这个`Future`成功或失败，但这将是一个阻塞操作，我们可能希望避免它。相反，我们可以实现一个在`Future`成功时触发的回调:

```java
Service<Request, Response> clientService = new LogFilter().andThen(Http.newService(":8080"));
Request request = Request.apply(Method.Get(), "/?name=John");
request.host("localhost");
Future<Response> response = clientService.apply(request);

Await.result(response
        .onSuccess(r -> {
            assertEquals("Hello John", r.getContentString());
            return BoxedUnit.UNIT;
        })
        .onFailure(r -> {
            throw new RuntimeException(r);
        })
);
```

注意，我们返回`BoxedUnit.UNIT.`返回`[Unit](https://web.archive.org/web/20220627083557/https://www.scala-lang.org/api/current/scala/Unit.html)` 是 Scala 处理`void`方法的方式，所以我们在这里这么做是为了保持互操作性。

## 7.摘要

在本教程中，我们学习了如何使用 Finagle 构建一个简单的 HTTP 服务器和一个客户端，以及如何在它们之间建立通信和交换消息。

和往常一样，带有所有示例的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627083557/https://github.com/eugenp/tutorials/tree/master/libraries-rpc)