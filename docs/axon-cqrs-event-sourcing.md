# 轴突框架指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/axon-cqrs-event-sourcing>

## 1.**概述**

在本文中，我们将着眼于`[Axon](https://web.archive.org/web/20220926201149/https://axoniq.io/product-overview/axon-framework)`以及它如何帮助我们实现考虑了 [`CQRS`](https://web.archive.org/web/20220926201149/https://martinfowler.com/bliki/CQRS.html) (命令查询责任分离)和 [`Event Sourcing`](https://web.archive.org/web/20220926201149/https://martinfowler.com/eaaDev/EventSourcing.html) 的应用程序。

在本指南中，将同时使用 Axon 框架和 Axon 服务器。前者将包含我们的实现，后者将是我们专用的事件存储和消息路由解决方案。

我们将要构建的示例应用程序关注于一个`Order`领域。为此，**我们将利用 Axon 为我们提供的 CQRS 和事件采购构建模块**。

请注意，许多共享的概念都来自于`[DDD](https://web.archive.org/web/20220926201149/https://en.wikipedia.org/wiki/Domain-driven_design),`，这超出了本文的范围。

## 2。Maven 依赖关系

我们将创建一个 Axon / Spring Boot 应用程序。因此，我们需要将最新的`[axon-spring-boot-starter](https://web.archive.org/web/20220926201149/https://search.maven.org/search?q=a:axon-spring-boot-starter)`依赖项添加到我们的`pom.xml`中，以及用于测试的`[axon-test](https://web.archive.org/web/20220926201149/https://search.maven.org/search?q=a:axon-test)`依赖项。
为了使用匹配的版本，我们将在依赖管理部分使用 [axon-bom](https://web.archive.org/web/20220926201149/https://search.maven.org/search?q=a:axon-bom) :

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.axonframework</groupId>
            <artifactId>axon-bom</artifactId>
            <version>4.5.13</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.axonframework</groupId>
        <artifactId>axon-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 3。Axon 服务器

我们将使用 [Axon 服务器](https://web.archive.org/web/20220926201149/https://axoniq.io/product-overview/axon-server)作为我们的[事件存储库](https://web.archive.org/web/20220926201149/https://en.wikipedia.org/wiki/Event_store)和我们专用的命令、事件和查询路由解决方案。

作为一个事件存储，它为我们提供了存储事件时所需的理想特征。[这篇](https://web.archive.org/web/20220926201149/https://axoniq.io/blog-overview/eventstore)文章提供了为什么这是可取的背景。

作为一个消息路由解决方案，它为我们提供了将几个实例连接在一起的选项，而无需关注于配置 RabbitMQ 或 Kafka 主题来共享和分发消息。

Axon Server 可以在这里下载[。由于它是一个简单的 JAR 文件，下面的操作足以启动它:](https://web.archive.org/web/20220926201149/https://download.axoniq.io/axonserver/AxonServer.zip)

```java
java -jar axonserver.jar
```

这将启动一个 Axon 服务器实例，可通过 [`localhost:8024`](https://web.archive.org/web/20220926201149/http://localhost:8024/) 访问。端点提供了连接的应用程序和它们可以处理的消息的概述，以及对 Axon Server 中包含的事件存储的查询机制。

Axon 服务器的默认配置和`axon-spring-boot-starter`依赖关系将确保我们的订单服务自动连接到它。

## 4。订单服务 API–命令

我们将考虑 CQRS 来建立我们的订单服务。因此，我们将强调流经应用程序的消息。

首先，我们将定义命令，意思是意图的表达。订单服务能够处理三种不同类型的操作:

1.  创建新秩序
2.  确认订单
3.  运送订单

自然，我们的域可以处理三个命令消息— `CreateOrderCommand`、` ConfirmOrderCommand`和`ShipOrderCommand`:

```java
public class CreateOrderCommand {

    @TargetAggregateIdentifier
    private final String orderId;
    private final String productId;

    // constructor, getters, equals/hashCode and toString 
}
public class ConfirmOrderCommand {

    @TargetAggregateIdentifier
    private final String orderId;

    // constructor, getters, equals/hashCode and toString
}
public class ShipOrderCommand {

    @TargetAggregateIdentifier
    private final String orderId;

    // constructor, getters, equals/hashCode and toString
}
```

**`[TargetAggregateIdentifier](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.0/org/axonframework/modelling/command/TargetAggregateIdentifier.html)` 注释告诉 Axon，带注释的字段是命令应该指向的给定集合的 id。**我们将在本文的后面简要介绍一下聚合。

另外，请注意，我们将命令中的字段标记为`final.`这是有意的，因为**对于不可变的**消息实现来说，这是一个最佳实践。

## 5。订单服务 API–事件

**我们的聚合将处理命令**，因为它负责决定订单是否可以创建、确认或发货。

它将通过发布一个事件来通知应用程序的其余部分它的决定。我们将有三种类型的事件— `OrderCreatedEvent, OrderConfirmedEvent`和`OrderShippedEvent`:

```java
public class OrderCreatedEvent {

    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}
public class OrderConfirmedEvent {

    private final String orderId;

    // default constructor, getters, equals/hashCode and toString
}
public class OrderShippedEvent { 

    private final String orderId; 

    // default constructor, getters, equals/hashCode and toString 
}
```

## 6。命令模型-订单集合

现在，我们已经针对命令和事件对核心 API 进行了建模，我们可以开始创建命令模型了。

[`Aggregate`](https://web.archive.org/web/20220926201149/https://www.martinfowler.com/bliki/DDD_Aggregate.html) 是命令模型中的常规组件，源于 DDD。其他框架也使用这个概念，例如在[这篇关于用 Spring 持久化 DDD 聚集的](/web/20220926201149/https://www.baeldung.com/spring-persisting-ddd-aggregates#introduction-to-aggregates)文章中可以看到。

由于我们的领域侧重于处理命令，**我们将创建一个`OrderAggregate`作为我们的命令模型的中心。**

### 6.1。聚合类

因此，让我们创建基本的聚合类:

```java
@Aggregate
public class OrderAggregate {

    @AggregateIdentifier
    private String orderId;
    private boolean orderConfirmed;

    @CommandHandler
    public OrderAggregate(CreateOrderCommand command) {
        AggregateLifecycle.apply(new OrderCreatedEvent(command.getOrderId(), command.getProductId()));
    }

    @EventSourcingHandler
    public void on(OrderCreatedEvent event) {
        this.orderId = event.getOrderId();
        orderConfirmed = false;
    }

    protected OrderAggregate() { }
}
```

**[`Aggregate`](https://web.archive.org/web/20220926201149/https://docs.axoniq.io/reference-guide/v/4.0/implementing-domain-logic/command-handling/aggregate)注释是 Axon Spring 特有的注释，将这个类标记为一个聚合。**它将通知框架需要为这个`OrderAggregate`实例化所需的 CQRS 和事件源特定构建块。

**由于一个聚合将处理指向特定聚合实例的命令，我们需要用 [`AggregateIdentifier`](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.0/org/axonframework/modelling/command/AggregateIdentifier.html) 注释来指定标识符。**

我们的聚合将在处理`OrderAggregate`‘命令处理构造函数’中的`CreateOrderCommand`时开始其生命周期。**为了告诉框架给定的函数能够处理命令，我们将添加 [`CommandHandler`](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/3.1/org/axonframework/commandhandling/CommandHandler.html) 注释。**

**当处理`CreateOrderCommand`时，它将通知应用程序的其余部分，通过发布`OrderCreatedEvent.`** 创建了一个订单。要从聚合中发布一个事件，我们将使用 [`AggregateLifecycle#apply(Object…)`](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.0/org/axonframework/modelling/command/AggregateLifecycle.html) 。

从这一点出发，我们实际上可以开始将事件源作为驱动力，从事件流中重新创建一个聚合实例。

我们从“聚合创建事件”`OrderCreatedEvent`开始，该事件在一个 [`EventSourcingHandler`](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.0/org/axonframework/eventsourcing/EventSourcingHandler.html) 注释函数中处理，以设置订单聚合的`orderId`和`orderConfirmed`状态。

还要注意，为了能够基于事件获得聚合，Axon 需要一个默认的构造函数。

### 6.2。聚合命令处理程序

现在我们有了基本的集合，我们可以开始实现剩下的命令处理程序了:

```java
@CommandHandler 
public void handle(ConfirmOrderCommand command) { 
    if (orderConfirmed) {
        return;
    }
    apply(new OrderConfirmedEvent(orderId)); 
} 

@CommandHandler 
public void handle(ShipOrderCommand command) { 
    if (!orderConfirmed) { 
        throw new UnconfirmedOrderException(); 
    } 
    apply(new OrderShippedEvent(orderId)); 
} 

@EventSourcingHandler 
public void on(OrderConfirmedEvent event) { 
    orderConfirmed = true; 
}
```

我们的命令和事件源处理程序的签名简单地声明了`handle({the-command})` 和`on({the-event})` 以保持简洁的格式。

此外，我们还定义了订单只能确认一次，并在确认后发货。因此，我们将忽略前一种情况下的命令，如果后一种情况不是这样，就抛出一个`UnconfirmedOrderException`。

这说明了`OrderConfirmedEvent`采购处理程序需要将订单集合的`orderConfirmed`状态更新为`true` 。

## 7。测试命令模型

首先，我们需要通过为`OrderAggregate`创建一个`[FixtureConfiguration](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/3.3/org/axonframework/test/aggregate/FixtureConfiguration.html)` 来设置我们的测试:

```java
private FixtureConfiguration<OrderAggregate> fixture;

@Before
public void setUp() {
    fixture = new AggregateTestFixture<>(OrderAggregate.class);
}
```

第一个测试用例应该涵盖最简单的情况。当聚合处理`CreateOrderCommand`时，它应该产生一个`OrderCreatedEvent`:

```java
String orderId = UUID.randomUUID().toString();
String productId = "Deluxe Chair";
fixture.givenNoPriorActivity()
  .when(new CreateOrderCommand(orderId, productId))
  .expectEvents(new OrderCreatedEvent(orderId, productId));
```

接下来，我们可以测试只有订单被确认后才能发货的决策逻辑。因此，我们有两种场景——一种是我们预期的异常，另一种是我们预期的`OrderShippedEvent`。

让我们看一下第一个场景，在这里我们会遇到一个异常:

```java
String orderId = UUID.randomUUID().toString();
String productId = "Deluxe Chair";
fixture.given(new OrderCreatedEvent(orderId, productId))
  .when(new ShipOrderCommand(orderId))
  .expectException(UnconfirmedOrderException.class); 
```

现在是第二个场景，我们期待一个`OrderShippedEvent`:

```java
String orderId = UUID.randomUUID().toString();
String productId = "Deluxe Chair";
fixture.given(new OrderCreatedEvent(orderId, productId), new OrderConfirmedEvent(orderId))
  .when(new ShipOrderCommand(orderId))
  .expectEvents(new OrderShippedEvent(orderId));
```

## 8.查询模型–事件处理程序

到目前为止，我们已经建立了包含命令和事件的核心 API，并且我们已经有了 CQRS 订单服务的命令模型`OrderAggregate`。

接下来，**我们可以开始考虑我们的应用程序应该服务的查询模型之一**。

其中一种型号是`Order`:

```java
public class Order {

    private final String orderId;
    private final String productId;
    private OrderStatus orderStatus;

    public Order(String orderId, String productId) {
        this.orderId = orderId;
        this.productId = productId;
        orderStatus = OrderStatus.CREATED;
    }

    public void setOrderConfirmed() {
        this.orderStatus = OrderStatus.CONFIRMED;
    }

    public void setOrderShipped() {
        this.orderStatus = OrderStatus.SHIPPED;
    }

    // getters, equals/hashCode and toString functions
}
public enum OrderStatus {
    CREATED, CONFIRMED, SHIPPED
}
```

**我们将根据系统中传播的事件更新该模型。**一个春天`Service`比恩更新我们的模型就行了:

```java
@Service
public class OrdersEventHandler {

    private final Map<String, Order> orders = new HashMap<>();

    @EventHandler
    public void on(OrderCreatedEvent event) {
        String orderId = event.getOrderId();
        orders.put(orderId, new Order(orderId, event.getProductId()));
    }

    // Event Handlers for OrderConfirmedEvent and OrderShippedEvent...
}
```

因为我们已经使用了`axon-spring-boot-starter`依赖项来启动 Axon 应用程序，所以框架会自动扫描所有的 beans 来查找现有的消息处理功能。

由于`OrdersEventHandler`有`[EventHandler](https://web.archive.org/web/20220926201149/https://docs.axoniq.io/reference-guide/axon-framework/events/event-handlers)`带注释的函数来存储一个`Order`并更新它，这个 bean 将被框架注册为一个类，它应该接收事件，而不需要我们进行任何配置。

## 9.查询模型–查询处理程序

接下来，为了查询这个模型，例如检索所有订单，我们应该首先向我们的核心 API 引入一个查询消息:

```java
public class FindAllOrderedProductsQuery { }
```

其次，我们必须更新`OrdersEventHandler` 以便能够处理`FindAllOrderedProductsQuery`:

```java
@QueryHandler
public List<Order> handle(FindAllOrderedProductsQuery query) {
    return new ArrayList<>(orders.values());
}
```

带注释的函数`[QueryHandler](https://web.archive.org/web/20220926201149/https://docs.axoniq.io/reference-guide/axon-framework/queries/query-handlers)`将处理`FindAllOrderedProductsQuery`，并被设置为无论如何都返回一个`List<Order>`，类似于任何“查找全部”查询。

## 10。将所有东西放在一起

我们已经用命令、事件和查询充实了我们的核心 API，并通过使用`OrderAggregate`和`Order`模型建立了我们的命令和查询模型。

下一步是解决我们基础设施的遗留问题。当我们使用`axon-spring-boot-starter`时，这将自动设置许多所需的配置。

首先，**由于我们希望利用事件源为我们的聚合服务，我们需要一个`[EventStore](https://web.archive.org/web/20220926201149/https://docs.axoniq.io/reference-guide/axon-framework/events/event-bus-and-event-store).`** Axon 服务器，我们在第三步中启动的服务器将填补这个漏洞。****

其次，我们需要一种机制来存储我们的`Order`查询模型。对于这个例子，我们可以添加`[h2](https://web.archive.org/web/20220926201149/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2)`作为内存数据库，添加`[spring-boot-starter-data-jpa](https://web.archive.org/web/20220926201149/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa%20AND%20g:org.springframework.boot)`以便于使用:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 10.1。设置休息端点

接下来，我们需要能够访问我们的应用程序，为此我们将通过添加`[spring-boot-starter-web](https://web.archive.org/web/20220926201149/https://search.maven.org/search?q=a:spring-boot-starter-web%20AND%20g:org.springframework.boot)`依赖项来利用 REST 端点:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

从我们的 REST 端点，我们可以开始分派命令和查询:

```java
@RestController
public class OrderRestEndpoint {

    private final CommandGateway commandGateway;
    private final QueryGateway queryGateway;

    // Autowiring constructor and POST/GET endpoints
}
```

**`[CommandGateway](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.0/org/axonframework/commandhandling/gateway/CommandGateway.html)`用作发送命令消息的机制，而`[QueryGateway](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.1/org/axonframework/queryhandling/QueryGateway.html)`则发送查询消息。**与它们连接的`[CommandBus](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.3/org/axonframework/commandhandling/CommandBus.html)`和`[QueryBus](https://web.archive.org/web/20220926201149/https://docs.axoniq.io/reference-guide/v/3.3/part-iii-infrastructure-components/query-processing)`相比，网关提供了一个更简单、更直接的 API。

从现在开始，**我们的`OrderRestEndpoint`应该有一个 POST 端点来创建、确认和发送订单**:

```java
@PostMapping("/ship-order")
public CompletableFuture<Void> shipOrder() {
    String orderId = UUID.randomUUID().toString();
    return commandGateway.send(new CreateOrderCommand(orderId, "Deluxe Chair"))
                         .thenCompose(result -> commandGateway.send(new ConfirmOrderCommand(orderId)))
                         .thenCompose(result -> commandGateway.send(new ShipOrderCommand(orderId)));
}
```

这就完成了我们的 CQRS 应用程序的命令端。注意，网关返回一个 CompletableFuture，支持异步。

现在，剩下的就是一个 GET 端点来查询所有的`Order:`

```java
@GetMapping("/all-orders")
public CompletableFuture<List<Order>> findAllOrders() {
    return queryGateway.query(new FindAllOrderedProductsQuery(), ResponseTypes.multipleInstancesOf(Order.class));
}
```

**在 GET 端点中，我们利用`QueryGateway`来调度点对点查询。**在这样做的时候，我们创建了一个默认的`FindAllOrderedProductsQuery`，但是我们还需要指定预期的返回类型。

因为我们期望返回多个`Order`实例，所以我们利用了静态的 [`ResponseTypes#multipleInstancesOf(Class)`](https://web.archive.org/web/20220926201149/https://apidocs.axoniq.io/4.1/org/axonframework/messaging/responsetypes/ResponseTypes.html) 函数。这样，我们就为订单服务的查询端提供了一个基本入口。

我们完成了设置，所以一旦启动了`OrderApplication.`，现在我们就可以通过 REST 控制器发送一些命令和查询

POST-ing 到端点`/ship-order` 将实例化一个`OrderAggregate`，它将发布事件，反过来，它将保存/更新我们的`Orders.`。从`/all-orders`端点获取的消息将发布一个查询消息，该消息将由`OrdersEventHandler`处理，它将返回所有现有的`Orders.`

## 11。结论

在本文中，我们介绍了 Axon 框架，它是构建利用 CQRS 和事件源优势的应用程序的强大基础。

我们使用该框架实现了一个简单的订单服务，以展示这样一个应用程序在实践中应该如何构造。

最后，Axon Server 充当了我们的事件存储和消息路由机制，极大地简化了基础设施。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220926201149/https://github.com/eugenp/tutorials/tree/master/axon)

关于这个主题的任何其他问题，也可以查看[讨论 AxonIQ](https://web.archive.org/web/20220926201149/https://discuss.axoniq.io/) 。