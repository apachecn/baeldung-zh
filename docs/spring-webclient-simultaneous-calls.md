# 并发 Spring WebClient 调用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webclient-simultaneous-calls>

## 1.概观

通常，当在我们的应用程序中发出 HTTP 请求时，我们按顺序执行这些调用。但是，有时我们可能希望同时执行这些请求。

例如，当从多个来源检索数据时，或者当我们只是想提高应用程序的性能时，我们可能希望这样做。

在这个快速教程中，**我们将看看几种方法，看看如何通过使用**[Spring reactive`WebClient`](/web/20220630133110/https://www.baeldung.com/spring-5-webclient)**进行并行服务调用来实现这一点。**

## 2.回顾反应式编程

快速回顾一下 *WebClient* 是在 Spring 5 中引入的，是 Spring Web 反应模块的一部分。**它为发送 HTTP 请求提供了一个被动的、非阻塞的接口**。

关于 WebFlux 反应式编程的深入指南，请查看我们优秀的 Spring 5 WebFlux 指南。

## 3.简单的用户服务

在我们的例子中，我们将使用一个简单的`User` API。**这个 API 有一个 GET 方法，它公开了一个使用 id 作为参数**来检索用户的方法`getUser`。

让我们来看看如何发出一个调用来检索给定 id 的用户:

```java
WebClient webClient = WebClient.create("http://localhost:8080");
```

```java
public Mono<User> getUser(int id) {
    LOG.info(String.format("Calling getUser(%d)", id));

    return webClient.get()
        .uri("/user/{id}", id)
        .retrieve()
        .bodyToMono(User.class);
}
```

在下一节中，我们将学习如何同时调用这个方法。

## 4.同时拨打`WebClient`个电话

**在这一节，我们将看到几个同时调用我们的`getUser`方法**的例子。我们还将看看示例中的发布者实现`[Flux](https://web.archive.org/web/20220630133110/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)` 和`[Mono](https://web.archive.org/web/20220630133110/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)` 。

### 4.1.对同一服务的多次调用

**现在让我们想象一下，我们想要同时获取关于五个用户的数据，并将结果作为用户列表返回**:

```java
public Flux fetchUsers(List userIds) {
    return Flux.fromIterable(userIds)
        .flatMap(this::getUser);
}
```

让我们分解这些步骤来理解我们所做的事情:

我们首先使用静态的`fromIterable`方法从我们的`userIds`列表中创建一个通量。

接下来，我们调用`flatMap`来运行我们之前创建的 getUser 方法。默认情况下，这个反应式操作符的并发级别为 256，这意味着它最多同时执行 256 个`getUser`调用。这个数字可以通过使用`flatMap`重载版本的方法参数进行配置。

值得注意的是，由于操作是并行发生的，我们不知道结果的顺序。如果需要维护输入顺序，可以用`flatMapSequential`运算符代替。

由于 Spring WebClient 使用了一个非阻塞的 HTTP 客户端，所以用户不需要定义任何调度程序。负责调度调用并在内部适当的线程上发布它们的结果，没有阻塞。

### 4.2.对返回相同类型的不同服务的多次调用

**现在让我们看看如何同时调用多个服务**。

在本例中，我们将创建另一个端点，它返回相同的`User`类型:

```java
public Mono<User> getOtherUser(int id) {
    return webClient.get()
        .uri("/otheruser/{id}", id)
        .retrieve()
        .bodyToMono(User.class);
}
```

现在，并行执行两个或更多调用的方法变成了:

```java
public Flux fetchUserAndOtherUser(int id) {
    return Flux.merge(getUser(id), getOtherUser(id));
}
```

**这个例子中的主要区别是我们使用了静态方法`merge`而不是`fromIterable`方法**。使用合并方法，我们可以将两个或更多的*通量*合并成一个结果。

### 4.3.对不同服务不同类型的多次调用

两个服务返回相同内容的概率相当低。更典型的是，我们会有另一个服务提供不同的响应类型，我们的目标是合并两个(或更多)响应。

`Mono`类提供了静态 zip 方法，让我们可以组合两个或多个结果:

```java
public Mono fetchUserAndItem(int userId, int itemId) {
    Mono user = getUser(userId);
    Mono item = getItem(itemId);

    return Mono.zip(user, item, UserWithItem::new);
}
```

**`zip`方法将给定的`user`和`item`和`Mono`组合成一个新的`Mono`，其类型为`UserWithItem`** 。这是一个简单的 POJO 对象，它包装了用户和项目。

## 5.测试

在这一节中，我们将看到如何测试我们已经看到的代码，特别是验证服务调用是并行发生的。

为此，我们将使用 [Wiremock](/web/20220630133110/https://www.baeldung.com/introduction-to-wiremock) 来创建一个模拟服务器，并且我们将测试`fetchUsers`方法:

```java
@Test
public void givenClient_whenFetchingUsers_thenExecutionTimeIsLessThanDouble() {

    int requestsNumber = 5;
    int singleRequestTime = 1000;

    for (int i = 1; i <= requestsNumber; i++) {
        stubFor(get(urlEqualTo("/user/" + i)).willReturn(aResponse().withFixedDelay(singleRequestTime)
            .withStatus(200)
            .withHeader("Content-Type", "application/json")
            .withBody(String.format("{ \"id\": %d }", i))));
    }

    List<Integer> userIds = IntStream.rangeClosed(1, requestsNumber)
        .boxed()
        .collect(Collectors.toList());

    Client client = new Client("http://localhost:8089");

    long start = System.currentTimeMillis();
    List<User> users = client.fetchUsers(userIds).collectList().block();
    long end = System.currentTimeMillis();

    long totalExecutionTime = end - start;

    assertEquals("Unexpected number of users", requestsNumber, users.size());
    assertTrue("Execution time is too big", 2 * singleRequestTime > totalExecutionTime);
}
```

在这个例子中，我们采用的方法是模拟用户服务，让它在一秒钟内响应任何请求。**现在，如果我们用我们的`WebClient`打五个电话，我们可以假设不会超过两秒钟，因为这些电话是同时发生的**。

要了解测试`WebClient`的其他技术，请查看我们的指南[在春天](/web/20220630133110/https://www.baeldung.com/spring-mocking-webclient)模仿网络客户端。

## 6.结论

在本教程中，我们探索了几种使用 Spring 5 Reactive WebClient 同时进行 HTTP 服务调用的方法。

首先，我们展示了如何并行调用同一个服务。稍后，我们看到了如何调用返回不同类型的两个服务的示例。然后，我们展示了如何使用模拟服务器测试这段代码。

和往常一样，这篇文章的源代码可以在 GitHub 上找到。