# 春季推迟结果指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-deferred-result>

## 1。概述

在本教程中，我们将看看**如何在 Spring MVC 中使用`DeferredResult`类来执行异步请求处理**。

Servlet 3.0 中引入了异步支持，简单地说，它允许在请求接收线程之外的另一个线程中处理 HTTP 请求。

从 Spring 3.2 开始可用，帮助将一个长时间运行的计算从一个 http-worker 线程卸载到一个单独的线程。

尽管另一个线程将占用一些计算资源，但工作线程在此期间不会被阻塞，可以处理传入的客户端请求。

异步请求处理模型非常有用，因为它有助于在高负载期间很好地扩展应用程序，尤其是对于 IO 密集型操作。

## 2。设置

对于我们的示例，我们将使用一个 Spring Boot 应用程序。关于如何引导应用程序的更多细节，请参考我们之前的[文章](/web/20220626211007/https://www.baeldung.com/spring-boot-start)。

接下来，我们将使用`DeferredResult `演示同步和异步通信，并比较异步通信如何更好地适应高负载和 IO 密集型用例。

## 3。阻塞休息服务

让我们从开发一个标准的阻塞 REST 服务开始:

```java
@GetMapping("/process-blocking")
public ResponseEntity<?> handleReqSync(Model model) { 
    // ...
    return ResponseEntity.ok("ok");
}
```

这里的问题是**请求处理线程被阻塞，直到整个请求被处理**并且结果被返回。在长时间运行计算的情况下，这是一个次优的解决方案。

为了解决这个问题，我们可以更好地利用容器线程来处理客户端请求，我们将在下一节看到这一点。

## 4。无阻塞休息使用`DeferredResult`

为了避免阻塞，我们将使用基于回调的编程模型，而不是实际的结果，我们将返回一个`DeferredResult`到 servlet 容器。

```java
@GetMapping("/async-deferredresult")
public DeferredResult<ResponseEntity<?>> handleReqDefResult(Model model) {
    LOG.info("Received async-deferredresult request");
    DeferredResult<ResponseEntity<?>> output = new DeferredResult<>();

    ForkJoinPool.commonPool().submit(() -> {
        LOG.info("Processing in separate thread");
        try {
            Thread.sleep(6000);
        } catch (InterruptedException e) {
        }
        output.setResult(ResponseEntity.ok("ok"));
    });

    LOG.info("servlet thread freed");
    return output;
}
```

**请求处理在一个单独的线程中完成，一旦完成，我们就调用`DeferredResult`对象上的`setResult`操作。**

让我们看看日志输出，检查我们的线程是否如预期的那样运行:

```java
[nio-8080-exec-6] com.baeldung.controller.AsyncDeferredResultController: 
Received async-deferredresult request
[nio-8080-exec-6] com.baeldung.controller.AsyncDeferredResultController: 
Servlet thread freed
[nio-8080-exec-6] java.lang.Thread : Processing in separate thread
```

在内部，通知容器线程，并将 HTTP 响应传递给客户端。容器(servlet 3.0 或更高版本)将保持连接打开，直到响应到达或超时。

## 5。`DeferredResult`试镜

我们可以用 DeferredResult 注册 3 种类型的回调:完成、超时和错误回调。

让我们使用`onCompletion()`方法来定义一个异步请求完成时执行的代码块:

```java
deferredResult.onCompletion(() -> LOG.info("Processing complete"));
```

类似地，我们可以使用`onTimeout()`来注册自定义代码，以便在超时发生时调用。为了限制请求处理时间，我们可以在`DeferredResult`对象创建期间传递一个超时值:

```java
DeferredResult<ResponseEntity<?>> deferredResult = new DeferredResult<>(500l);

deferredResult.onTimeout(() -> 
  deferredResult.setErrorResult(
    ResponseEntity.status(HttpStatus.REQUEST_TIMEOUT)
      .body("Request timeout occurred.")));
```

如果超时，我们将通过向`DeferredResult`注册的超时处理程序设置不同的响应状态。

让我们通过处理一个超过定义的 5 秒超时值的请求来触发超时错误:

```java
ForkJoinPool.commonPool().submit(() -> {
    LOG.info("Processing in separate thread");
    try {
        Thread.sleep(6000);
    } catch (InterruptedException e) {
        ...
    }
    deferredResult.setResult(ResponseEntity.ok("OK")));
});
```

让我们看看日志:

```java
[nio-8080-exec-6] com.baeldung.controller.DeferredResultController: 
servlet thread freed
[nio-8080-exec-6] java.lang.Thread: Processing in separate thread
[nio-8080-exec-6] com.baeldung.controller.DeferredResultController: 
Request timeout occurred
```

将会出现由于某些错误或异常导致长时间运行的计算失败的情况。在这种情况下，我们也可以注册一个`onError()`回调:

```java
deferredResult.onError((Throwable t) -> {
    deferredResult.setErrorResult(
      ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body("An error occurred."));
});
```

如果出现错误，在计算响应时，我们将通过这个错误处理程序设置不同的响应状态和消息体。

## 6。结论

在这篇简短的文章中，我们看到了 Spring MVC `DeferredResult`如何简化异步端点的创建。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220626211007/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)