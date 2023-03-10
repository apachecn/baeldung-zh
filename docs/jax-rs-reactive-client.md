# 反应式 JAX 遥感客户端 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jax-rs-reactive-client>

## 1。简介

在本教程中，我们将使用 Jersey API 研究 JAX RS 对反应式(Rx)编程的支持。本文假设读者已经了解 Jersey REST 客户端 API。

熟悉一下[反应式编程概念](/web/20220630141337/https://www.baeldung.com/rx-java)会有帮助，但不是必须的。

## 2。依赖性

首先，我们需要标准的 Jersey 客户端库依赖项:

```java
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>2.27</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.inject</groupId>
    <artifactId>jersey-hk2</artifactId>
    <version>2.27</version>
</dependency> 
```

这些依赖给了我们股票 JAX 遥感反应式编程支持。目前版本的[球衣-客户端](https://web.archive.org/web/20220630141337/https://search.maven.org/search?q=g:org.glassfish.jersey.core%20AND%20a:jersey-client&core=gav)和[球衣-hk2](https://web.archive.org/web/20220630141337/https://search.maven.org/search?q=g:org.glassfish.jersey.inject%20AND%20a:jersey-hk2&core=gav) 可以在 Maven Central 上获得。

对于第三方反应式框架支持，我们将使用这些扩展:

```java
<dependency>
    <groupId>org.glassfish.jersey.ext.rx</groupId>
    <artifactId>jersey-rx-client-rxjava</artifactId>
    <version>2.27</version>
</dependency> 
```

上面的依赖关系提供了对 RxJava 的`[Observable](https://web.archive.org/web/20220630141337/https://github.com/ReactiveX/RxJava/wiki/Observable);`的支持。对于较新的 RxJava2 的`[Flowable](/web/20220630141337/https://www.baeldung.com/rxjava-2-flowable), `,我们使用下面的扩展:

```java
<dependency>
    <groupId>org.glassfish.jersey.ext.rx</groupId>
    <artifactId>jersey-rx-client-rxjava2</artifactId>
    <version>2.27</version>
</dependency>
```

对 [rxjava](https://web.archive.org/web/20220630141337/https://search.maven.org/search?q=a:jersey-rx-client-rxjava%20AND%20g:org.glassfish.jersey.ext.rx) 和 [rxjava2](https://web.archive.org/web/20220630141337/https://search.maven.org/search?q=a:jersey-rx-client-rxjava2%20AND%20g:org.glassfish.jersey.ext.rx) 的依赖也可以在 Maven Central 上获得。

## 3。为什么我们需要反应式 JAX-遥感客户端

假设我们有三个 REST APIs 要使用:

*   `id-service`提供了一个长用户 id 列表
*   `name-service`为给定的用户 ID 提供用户名
*   `hash-service`将返回用户 ID 和用户名的散列

我们为每项服务创建一个客户端:

```java
Client client = ClientBuilder.newClient();
WebTarget userIdService = client.target("http://localhost:8080/id-service/ids");
WebTarget nameService 
  = client.target("http://localhost:8080/name-service/users/{userId}/name");
WebTarget hashService = client.target("http://localhost:8080/hash-service/{rawValue}");
```

这是一个人为的例子，但它适用于我们的说明。JAX-RS 规范支持至少三种同时使用这些服务的方法:

*   同步(阻塞)
*   异步(非阻塞)
*   反应式(功能性、非阻塞)

### 3.1。同步 Jersey 客户端调用的问题

使用这些服务的常规方法是使用`id-service`来获取用户 ID，然后为每个返回的 ID 依次调用`name-service`和`hash-service`API。

**使用这种方法，每个调用都会阻塞正在运行的线程，直到请求完成**，总共花费大量时间来完成合并的请求。在任何重要的用例中，这显然都不能令人满意。

### 3.2。异步 Jersey 客户端调用的问题

更复杂的方法是使用 JAX-RS 支持的`[InvocationCallback](https://web.archive.org/web/20220630141337/https://docs.oracle.com/javaee/7/api/javax/ws/rs/client/InvocationCallback.html) `机制。最基本的形式是，我们向`get`方法传递一个回调来定义当给定的 API 调用完成时会发生什么。

虽然我们现在获得了真正的异步执行([对线程效率有一些限制](https://web.archive.org/web/20220630141337/https://stackoverflow.com/questions/26150257/jersey-client-non-blocking))，但很容易看出**这种风格的代码在任何情况下都变得不可读和笨拙**。 [JAX-RS 规范](https://web.archive.org/web/20220630141337/https://github.com/jax-rs/spec)特别强调了这一场景为[末日金字塔](https://web.archive.org/web/20220630141337/https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming)):

```java
// used to keep track of the progress of the subsequent calls
CountDownLatch completionTracker = new CountDownLatch(expectedHashValues.size()); 

userIdService.request()
  .accept(MediaType.APPLICATION_JSON)
  .async()
  .get(new InvocationCallback<List<Long>>() {
    @Override
    public void completed(List<Long> employeeIds) {
        employeeIds.forEach((id) -> {
        // for each employee ID, get the name
        nameService.resolveTemplate("userId", id).request()
          .async()
          .get(new InvocationCallback<String>() {
              @Override
              public void completed(String response) {
                     hashService.resolveTemplate("rawValue", response + id).request()
                    .async()
                    .get(new InvocationCallback<String>() {
                        @Override
                        public void completed(String response) {
                            //complete the business logic
                        }
                        // ommitted implementation of the failed() method
                    });
              }
              // omitted implementation of the failed() method
          });
        });
    }
    // omitted implementation of the failed() method
});

// wait for inner requests to complete in 10 seconds
if (!completionTracker.await(10, TimeUnit.SECONDS)) {
    logger.warn("Some requests didn't complete within the timeout");
}
```

我们已经实现了异步、省时的代码，但是:

*   很难阅读
*   每次调用都会产生一个新线程

请注意，我们在所有代码示例中都使用了一个`CountDownLatch`来等待所有预期值被`hash-service.` 传递，这样我们就可以通过检查所有预期值是否都被传递来断言代码在单元测试中是有效的。

一个普通的客户端不会等待，而是在回调中做任何应该做的事情，以免阻塞线程。

### 3.3。功能性反应解决方案

功能性和反应性方法将为我们提供:

*   出色的代码可读性
*   流畅的编码风格
*   有效的线程管理

JAX 遥感中心在以下方面支持这些目标:

*   `CompletionStageRxInvoker `支持`CompletionStage`接口作为默认无功组件
*   `RxObservableInvokerProvider`支持 RxJava 的`Observable `
*   `RxFlowableInvokerProvider`支持 RxJava 的`Flowable`

还有一个 [API](https://web.archive.org/web/20220630141337/https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/rx-client.html#rx.client.spi) 用于添加对其他反应库的支持。

## 4。JAX-RS 无功组件支持

### 4.1。`CompletionStage `在 JAX-RS

利用 [`CompletionStage `](https://web.archive.org/web/20220630141337/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletionStage.html) 及其具体实现–`[CompletableFuture](https://web.archive.org/web/20220630141337/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletableFuture.html) – `我们可以编写一个优雅、无阻塞、流畅的服务调用流程。

让我们从检索用户 id 开始:

```java
CompletionStage<List<Long>> userIdStage = userIdService.request()
  .accept(MediaType.APPLICATION_JSON)
  .rx()
  .get(new GenericType<List<Long>>() {
}).exceptionally((throwable) -> {
    logger.warn("An error has occurred");
    return null;
});
```

**`rx() `方法调用是被动处理开始的地方。**我们使用`exceptionally`函数来流畅地定义我们的异常处理场景。

从这里，我们可以干净地编排调用，从名称服务中检索用户名，然后散列名称和用户 ID 的组合:

```java
List<String> expectedHashValues = ...;
List<String> receivedHashValues = new ArrayList<>(); 

// used to keep track of the progress of the subsequent calls 
CountDownLatch completionTracker = new CountDownLatch(expectedHashValues.size()); 

userIdStage.thenAcceptAsync(employeeIds -> {
  logger.info("id-service result: {}", employeeIds);
  employeeIds.forEach((Long id) -> {
    CompletableFuture completable = nameService.resolveTemplate("userId", id).request()
      .rx()
      .get(String.class)
      .toCompletableFuture();

    completable.thenAccept((String userName) -> {
        logger.info("name-service result: {}", userName);
        hashService.resolveTemplate("rawValue", userName + id).request()
          .rx()
          .get(String.class)
          .toCompletableFuture()
          .thenAcceptAsync(hashValue -> {
              logger.info("hash-service result: {}", hashValue);
              receivedHashValues.add(hashValue);
              completionTracker.countDown();
          }).exceptionally((throwable) -> {
              logger.warn("Hash computation failed for {}", id);
              return null;
         });
    });
  });
});

if (!completionTracker.await(10, TimeUnit.SECONDS)) {
    logger.warn("Some requests didn't complete within the timeout");
}

assertThat(receivedHashValues).containsAll(expectedHashValues); 
```

在上面的示例中，我们用流畅可读的代码组合了 3 个服务的执行。

方法`thenAcceptAsync`将执行提供的函数`after `，给定的`CompletionStage`已经完成执行(或者抛出异常)。

每个连续的调用都是非阻塞的，明智地利用了系统资源。

**`CompletionStage`接口提供了多种分段和编排方法，允许我们在多步骤编排(或单个服务调用)中组合、排序和异步执行任意数量的步骤**。

### 4.2。`Observable `在 JAX-RS

要使用`Observable ` RxJava 组件，我们必须首先在客户机上注册*rxbobservableinvokerprovider*提供者(而不是 Jersey 规范文档中提到的"`ObservableRxInvokerProvider” `):

```java
Client client = client.register(RxObservableInvokerProvider.class); 
```

然后我们覆盖默认调用者:

```java
Observable<List<Long>> userIdObservable = userIdService
  .request()
  .rx(RxObservableInvoker.class)
  .get(new GenericType<List<Long>>(){});
```

从这一点出发，我们**可以使用标准的`Observable `语义来编排处理流程**:

```java
userIdObservable.subscribe((List<Long> listOfIds)-> { 
  /** define processing flow for each ID */
});
```

### 4.3.`Flowable `在 JAX-RS

使用 RxJava `Flowable `的语义类似于`Observable. `我们注册适当的提供者:

```java
client.register(RxFlowableInvokerProvider.class);
```

然后我们供应`RxFlowableInvoker`:

```java
Flowable<List<Long>> userIdFlowable = userIdService
  .request()
  .rx(RxFlowableInvoker.class)
  .get(new GenericType<List<Long>>(){});
```

接下来，我们可以使用普通的`Flowable ` API。

## 5。结论

JAX-RS 规范提供了大量选项，可以干净、无阻塞地执行 REST 调用。

特别是,`CompletionStage `接口提供了一组健壮的方法，涵盖了各种服务编排场景，并提供了定制`Executors `的机会，以便对线程管理进行更细粒度的控制。

你可以在 Github 上查看这篇文章[的代码。](https://web.archive.org/web/20220630141337/https://github.com/eugenp/tutorials/tree/master/spring-jersey)