# Axon 框架中的调度查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/axon-query-dispatching>

## 1.概观

Axon Framework 帮助我们构建事件驱动的微服务系统。在[Axon 框架指南](/web/20220909152418/https://www.baeldung.com/axon-cqrs-event-sourcing)中，我们通过一个简单的 Axon [Spring Boot](/web/20220909152418/https://www.baeldung.com/spring-boot-start) 应用了解了 Axon，其中包括构建一个示例`Order`模型供我们更新和查询。那篇文章使用了一个简单的点对点查询。

在本教程中，我们将基于上面的例子来研究我们在 Axon 中分派查询的所有方法。除了更深入地了解点对点查询，我们还将了解分散-聚集查询和订阅查询。

## 2.查询调度

当我们向 Axon 提交查询时，框架会将该查询发送给所有能够回答我们的查询的注册查询处理程序。在分布式系统中，多个节点可能支持同一种查询，单个节点也可能有多个支持该查询的查询处理程序。

那么，Axon 如何决定将哪些结果包含在它的响应中呢？答案取决于我们如何分派查询。Axon 给了我们三个选择:

*   **点对点查询从支持我们的查询的任何节点获得完整的答案**
*   **分散-聚集查询从支持我们的查询的所有节点获得完整的答案**
*   **订阅查询获得迄今为止的答案，然后继续监听任何更新**

在接下来的几节中，我们将学习如何支持和调度各种查询。

## 3.点对点查询

通过点对点查询，Axon 将查询发送到支持该查询的每个节点。Axon 假设任何节点都能够对点对点查询给出完整的答案，并且它将简单地返回从第一个响应的节点获得的结果。

在本节中，我们将使用点对点查询来获取系统中所有当前的 `Order`。

### 3.1.定义查询

Axon 使用强类型类来表示查询类型并封装查询参数。在本例中，由于我们正在查询所有订单，因此不需要任何查询参数。因此，我们可以用一个空类来表示我们的查询:

```java
public class FindAllOrderedProductsQuery {}
```

### 3.2.定义查询处理程序

**我们可以使用`@QueryHandler`注释注册查询处理程序。**

让我们为查询处理程序创建一个类，并添加一个可以支持`FindAllOrderedProductsQuery`查询的处理程序:

```java
@Service
public class InMemoryOrdersEventHandler implements OrdersEventHandler {
    private final Map<String, Order> orders = new HashMap<>();

    @QueryHandler
    public List<Order> handle(FindAllOrderedProductsQuery query) {
        return new ArrayList<>(orders.values());
    }
}
```

在上面的例子中，我们将`handle()`注册为 Axon 查询处理程序，它:

1.  能够响应`FindAllOrderedProductsQuery `查询
2.  返回一个`Orders`的`List`。正如我们稍后将看到的， **Axon 在决定哪个查询处理程序可以响应给定的查询时，会考虑返回类型。**这使得逐渐迁移到新的 API 变得更加容易。

我们使用上面的`OrdersEventHandler`接口，这样我们以后就可以换成使用持久数据存储的实现，比如 MongoDB。在本教程中，我们将简单地将`Order`对象存储在内存`Map`中。因此，我们的查询处理程序只需要将`Order`对象作为`List`返回。

### 3.3.分派点对点查询

现在我们已经定义了一个查询类型和一个查询处理程序，我们准备向 Axon 发送一个`FindAllOrderedProductsQuery`。让我们用发出点对点`FindAllOrderedProductsQuery`的方法创建一个服务类:

```java
@Service
public class OrderQueryService {
    private final QueryGateway queryGateway;

    public OrderQueryService(QueryGateway queryGateway) {
        this.queryGateway = queryGateway;
    }

    public CompletableFuture<List<OrderResponse>> findAllOrders() {
        return queryGateway.query(new FindAllOrderedProductsQuery(),
            ResponseTypes.multipleInstancesOf(Order.class))
          .thenApply(r -> r.stream()
            .map(OrderResponse::new)
            .collect(Collectors.toList()));
    }
}
```

在上面的例子中，我们使用 Axon 的`QueryGateway`来调度`FindAllOrderedProductsQuery`的一个实例。我们使用网关的`query()`方法来发布点对点查询。因为我们指定了`ResponseTypes.multipleInstancesOf(Order.class)`，Axon 知道我们只想与返回类型是`Order`对象集合的查询处理程序对话。

最后，为了在我们的`Order`模型类和我们的外部客户端之间添加一个间接层，我们将结果包装在`OrderResponse`对象中。

### 3.4.测试我们的点对点查询

我们将使用 [`@SpringBootTest`](/web/20220909152418/https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest) 来测试我们使用 Axon 集成的查询。让我们从将[弹簧测试](https://web.archive.org/web/20220909152418/https://search.maven.org/search?q=g:org.springframework%20a:spring-test)依赖项添加到我们的`pom.xml`文件开始:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <scope>test</scope>
</dependency> 
```

接下来，让我们添加一个调用我们的服务方法来检索`Order`的测试:

```java
@SpringBootTest(classes = OrderApplication.class)
class OrderQueryServiceIntegrationTest {

    @Autowired
    OrderQueryService queryService;

    @Autowired
    OrdersEventHandler handler;

    private String orderId;

    @BeforeEach
    void setUp() {
        orderId = UUID.randomUUID().toString();
        Order order = new Order(orderId);
        handler.reset(Collections.singletonList(order));
    }

    @Test
    void givenOrderCreatedEventSend_whenCallingAllOrders_thenOneCreatedOrderIsReturned()
            throws ExecutionException, InterruptedException {
        List<OrderResponse> result = queryService.findAllOrders().get();
        assertEquals(1, result.size());
        OrderResponse response = result.get(0);
        assertEquals(orderId, response.getOrderId());
        assertEquals(OrderStatusResponse.CREATED, response.getOrderStatus());
        assertTrue(response.getProducts().isEmpty());
    }
}
```

在上面的`@BeforeEach`方法中，我们调用了`reset()`，这是`OrdersEventHandler`中的一个方便的方法，用于从遗留系统预加载`Order` 对象，或者帮助简化迁移。在这里，我们使用它将一个`Order`预加载到我们的内存存储中进行测试。

然后，我们调用我们的服务方法，并验证在将我们的查询分派给我们之前设置的查询处理程序之后，它已经检索了我们的测试订单。

## 4.分散-收集查询

分散-聚集查询被分派给支持该查询的所有节点中的所有查询处理器。对于这些查询，来自每个查询处理器的结果被组合成单个响应。如果两个节点有相同的 Spring 应用程序名称，Axon 认为它们是等价的，并且将只使用第一个响应的节点的结果。

在本节中，我们将创建一个查询来检索与给定产品 ID 匹配的已发货产品的总数。我们将模拟查询一个实时系统和一个遗留系统，以显示 Axon 将组合来自两个系统的响应。

### 4.1.定义查询

与我们的点对点查询不同，这次我们需要提供一个参数:产品 ID。**我们将使用我们的产品 ID 参数**创建一个 [POJO](/web/20220909152418/https://www.baeldung.com/java-pojo-class) ，而不是一个空类

```java
public class TotalProductsShippedQuery {
    private final String productId;

    public TotalProductsShippedQuery(String productId) {
        this.productId = productId;
    }

    // getter
}
```

### 4.2.定义查询处理程序

首先，我们将查询基于事件的系统，我们会记得，该系统使用内存中的数据存储。让我们向现有的`InMemoryOrdersEventHandler` 添加一个查询处理程序，以获得发货产品的总数:

```java
@QueryHandler
public Integer handle(TotalProductsShippedQuery query) {
    return orders.values().stream()
      .filter(o -> o.getOrderStatus() == OrderStatus.SHIPPED)
      .map(o -> Optional.ofNullable(o.getProducts().get(query.getProductId())).orElse(0))
      .reduce(0, Integer::sum);
}
```

上图中，我们检索了所有内存中的`Order`对象，并删除了所有尚未发布的对象。然后，我们调用每个`Order`上的`getProducts()`来获取产品 ID 与我们的查询参数相匹配的已发货产品的数量。然后，我们将这些数字相加，得到发货产品的总数。

因为我们希望将这些结果与我们假设的遗留系统中的数字结合起来，所以让我们用一个单独的类和查询处理程序来模拟遗留数据:

```java
@Service
public class LegacyQueryHandler {
    @QueryHandler
    public Integer handle(TotalProductsShippedQuery query) {
        switch (query.getProductId()) {
        case "Deluxe Chair":
            return 234;
        case "a6aa01eb-4e38-4dfb-b53b-b5b82961fbf3":
            return 10;
        default:
            return 0;
        }
    }
}
```

出于本教程的考虑，这个查询处理程序与我们的`InMemoryOrdersEventHandler`处理程序存在于同一个 Spring 应用程序中。在真实的场景中，我们可能不会在同一个应用程序中为同一个查询类型使用多个查询处理程序。**分散-聚集查询通常组合来自多个 Spring 应用程序的结果，每个应用程序都有一个单独的处理程序。**

### 4.3.分派分散-收集查询

让我们为我们的`OrderQueryService`添加一个新方法来分派一个分散-聚集查询:

```java
public Integer totalShipped(String productId) {
    return queryGateway.scatterGather(new TotalProductsShippedQuery(productId),
        ResponseTypes.instanceOf(Integer.class), 10L, TimeUnit.SECONDS)
      .reduce(0, Integer::sum);
}
```

这一次，我们用参数`productId`构造查询对象。我们还为我们的`scatterGather()`调用设置了 10 秒的超时。 **Axon 将只对它在该时间窗口内检索到的结果做出响应**。如果一个或多个处理程序在该窗口内没有响应，他们的结果将不会包含在`queryGateway`的响应中。

### 4.4.测试我们的分散-聚集查询

让我们为我们的`OrderQueryServiceIntegrationTest`添加一个测试:

```java
void givenThreeDeluxeChairsShipped_whenCallingAllShippedChairs_then234PlusTreeIsReturned() {
    Order order = new Order(orderId);
    order.getProducts().put("Deluxe Chair", 3);
    order.setOrderShipped();
    handler.reset(Collections.singletonList(order));

    assertEquals(237, queryService.totalShipped("Deluxe Chair"));
}
```

上面，我们使用我们的`reset()`方法来模拟我们的事件驱动系统中的三个订单。之前，在我们的`LegacyQueryHandler`中，我们在遗留系统中硬编码了 234 把豪华座椅。因此，我们的测试应该会产生总共 237 把豪华座椅。

## 5.订阅查询

**通过订阅查询，我们得到一个初始结果，然后是一系列更新。**在本节中，我们将在系统中查询当前状态的`Order`,然后保持与 Axon 的连接，以便在`Order`出现新的更新时获取这些更新。

### 5.1.定义查询

因为我们想要检索一个特定的订单，所以让我们创建一个查询类，其中包含一个订单 ID 作为它的唯一参数:

```java
public class OrderUpdatesQuery {
    private final String orderId;

    public OrderUpdatesQuery(String orderId) {
        this.orderId = orderId;
    }

    // getter
}
```

### 5.2.定义查询处理程序

从内存映射中检索`Order`的查询处理程序非常简单。让我们将它添加到我们的`InMemoryOrdersEventHandler`类中:

```java
@QueryHandler
public Order handle(OrderUpdatesQuery query) {
    return orders.get(query.getOrderId());
}
```

### 5.3.发出查询更新

订阅查询只有在有更新时才有意义。 **Axon Framework 提供了一个`QueryUpdateEmitter`类，我们可以用它来通知 Axon 应该如何以及何时更新订阅。**让我们将发射器注入到我们的`InMemoryOrdersEventHandler`类中，并以一种方便的方法使用它:

```java
@Service
public class InMemoryOrdersEventHandler implements OrdersEventHandler {

    private final QueryUpdateEmitter emitter;

    public InMemoryOrdersEventHandler(QueryUpdateEmitter emitter) {
        this.emitter = emitter;
    }

    private void emitUpdate(Order order) {
        emitter.emit(OrderUpdatesQuery.class, q -> order.getOrderId()
          .equals(q.getOrderId()), order);
    }

    // our event and query handlers
}
```

**我们的`emitter.emit()`调用告诉 Axon，任何订阅了`OrderUpdatesQuery`的客户端可能需要接收更新。**第二个参数是一个过滤器，告诉 Axon 只有与所提供的订单 ID 相匹配的订阅才能获得更新。

我们现在可以在任何修改订单的事件处理程序中使用我们的`emitUpdate()`方法。例如，如果订单已发货，则应该通知该订单的任何活动更新订阅。让我们为[前一篇文章](/web/20220909152418/https://www.baeldung.com/axon-cqrs-event-sourcing)中涉及的`OrderShippedEvent`创建一个事件处理程序，并让它发出对已发货订单的更新:

```java
@Service
public class InMemoryOrdersEventHandler implements OrdersEventHandler {
    @EventHandler
    public void on(OrderShippedEvent event) {
        orders.computeIfPresent(event.getOrderId(), (orderId, order) -> {
            order.setOrderShipped();
            emitUpdate(order);
            return order;
        });
    }

    // fields, query handlers, other event handlers, and our emitUpdate() method
}
```

我们可以对我们的`ProductAddedEvent`、`ProductCountIncrementedEvent`、`ProductCountDecrementedEvent`和`OrderConfirmedEvent`事件做同样的事情。

### 5.4.订阅查询

接下来，我们将构建一个订阅查询的服务方法。我们将使用来自[反应堆核心](/web/20220909152418/https://www.baeldung.com/reactor-core)的`Flux`，以便将更新流式传输到客户端代码。

让我们将[依赖项](https://web.archive.org/web/20220909152418/https://search.maven.org/search?q=g:io.projectreactor%20a:reactor-core)添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
```

现在，让我们将我们的服务方法实现添加到`OrderQueryService`:

```java
public class OrderQueryService {
    public Flux<OrderResponse> orderUpdates(String orderId) {
        return subscriptionQuery(new OrderUpdatesQuery(orderId), ResponseTypes.instanceOf(Order.class))
                .map(OrderResponse::new);
    }

    private <Q, R> Flux<R> subscriptionQuery(Q query, ResponseType<R> resultType) {
        SubscriptionQueryResult<R, R> result = queryGateway.subscriptionQuery(query,
          resultType, resultType);
        return result.initialResult()
          .concatWith(result.updates())
          .doFinally(signal -> result.close());
    }

    // our other service methods
}
```

上面的公共`orderUpdates()`方法将其大部分工作委托给我们的私有便利方法`subscriptionQuery()`，尽管我们再次将我们的响应打包为`OrderResponse`对象，所以我们没有公开我们的内部`Order`对象。

我们的广义`subscriptionQuery()`便利方法是我们将从 Axon 获得的初始结果与任何未来更新相结合。

首先，我们调用 Axon 的`queryGateway.subscriptionQuery()`来获得一个`SubscriptionQueryResult`对象。我们将`resultType`提供给`queryGateway.` `subscriptionQuery()` 两次，因为我们总是期待一个`Order` 对象，但是如果我们愿意，我们可以使用不同的类型进行更新。

接下来，我们使用`result.getInitialResult()`和`result.getUpdates()`来获取完成订阅所需的所有信息。

最后，我们关闭流。

虽然我们在这里不使用它，但 Axon Framework 还有一个[反应式扩展](https://web.archive.org/web/20220909152418/https://github.com/AxonFramework/extension-reactor)，它提供了一个替代的查询网关，可以更容易地处理订阅查询。

### 5.5.测试我们的订阅查询

为了帮助我们测试返回`Flux`的服务方法，我们将使用从 [reactor-test](https://web.archive.org/web/20220909152418/https://search.maven.org/search?q=g:io.projectreactor%20a:reactor-test) 依赖关系中获得的`StepVerifier` 类:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
</dependency>
```

让我们添加我们的测试:

```java
class OrderQueryServiceIntegrationTest {
    @Test
    void givenOrdersAreUpdated_whenCallingOrderUpdates_thenUpdatesReturned() {
        ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
        executor.schedule(this::addIncrementDecrementConfirmAndShip, 100L, TimeUnit.MILLISECONDS);
        try {
            StepVerifier.create(queryService.orderUpdates(orderId))
              .assertNext(order -> assertTrue(order.getProducts().isEmpty()))
              .assertNext(order -> assertEquals(1, order.getProducts().get(productId)))
              .assertNext(order -> assertEquals(2, order.getProducts().get(productId)))
              .assertNext(order -> assertEquals(1, order.getProducts().get(productId)))
              .assertNext(order -> assertEquals(OrderStatusResponse.CONFIRMED, order.getOrderStatus()))
              .assertNext(order -> assertEquals(OrderStatusResponse.SHIPPED, order.getOrderStatus()))
              .thenCancel()
              .verify();
        } finally {
            executor.shutdown();
        }
    }

    private void addIncrementDecrementConfirmAndShip() {
        sendProductAddedEvent();
        sendProductCountIncrementEvent();
        sendProductCountDecrement();
        sendOrderConfirmedEvent();
        sendOrderShippedEvent();
    }

    private void sendProductAddedEvent() {
        ProductAddedEvent event = new ProductAddedEvent(orderId, productId);
        eventGateway.publish(event);
    }

    private void sendProductCountIncrementEvent() {
        ProductCountIncrementedEvent event = new ProductCountIncrementedEvent(orderId, productId);
        eventGateway.publish(event);
    }

    private void sendProductCountDecrement() {
        ProductCountDecrementedEvent event = new ProductCountDecrementedEvent(orderId, productId);
        eventGateway.publish(event);
    }

    private void sendOrderConfirmedEvent() {
        OrderConfirmedEvent event = new OrderConfirmedEvent(orderId);
        eventGateway.publish(event);
    }

    private void sendOrderShippedEvent() {
        OrderShippedEvent event = new OrderShippedEvent(orderId);
        eventGateway.publish(event);
    }

    // our other tests
}
```

上面，我们有一个私有的`addIncrementDecrementConfirmAndShip()`方法，向 Axon 发布五个`Order`相关的事件。在测试开始后 100 毫秒，我们通过 [`ScheduledExecutorService`](/web/20220909152418/https://www.baeldung.com/java-executor-service-tutorial#ScheduledExecutorService) 在一个单独的线程中调用它，以便模拟在我们开始`OrderUpdatesQuery`订阅后出现的事件。

在我们的主线程中，我们调用正在测试的`orderUpdates()`查询，使用`StepVerifier `允许我们对从订阅接收到的每个离散更新做出断言。

## 6.结论

在本文中，我们探讨了在 Axon 框架中调度查询的三种方法:点对点查询、分散-聚集查询和订阅查询。

与往常一样，GitHub 上的[提供了本文的完整代码示例。](https://web.archive.org/web/20220909152418/https://github.com/eugenp/tutorials/tree/master/axon)