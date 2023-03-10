# 轴突中的多实体聚集

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-axon-multi-entity-aggregates>

## 1。概述

在本文中，**我们将探讨 [Axon](/web/20220628060423/https://www.baeldung.com/axon-cqrs-event-sourcing) 如何支持具有多个实体的聚合**。

我们认为这篇文章是我们关于 [Axon](/web/20220628060423/https://www.baeldung.com/axon-cqrs-event-sourcing) 的主要指南的扩展。因此，我们将再次利用 [Axon 框架](https://web.archive.org/web/20220628060423/https://axoniq.io/product-overview/axon-framework)和 [Axon 服务器](https://web.archive.org/web/20220628060423/https://axoniq.io/product-overview/axon-server)。我们将在本文的代码中使用前者，后者是事件存储和消息路由器。

由于这是一个扩展，让我们详细阐述一下我们在基础文章中提出的`Order`域。

## 2.集合和实体

Axon 支持的聚合和实体源于[领域驱动设计](https://web.archive.org/web/20220628060423/https://en.wikipedia.org/wiki/Domain-driven_design)。在深入研究代码之前，**让我们首先在这个上下文中确定什么是**实体:

*   **一个基本上由其属性定义的物体，而是由一系列连续性和同一性定义的物体**

因此，一个实体是可识别的，但不是通过它所包含的属性。此外，变化发生在实体上，因为它保持了一种连续性。

了解了这一点，我们可以采取以下步骤，分享聚合在这种情况下的含义(摘自[领域驱动设计:解决软件核心的复杂性](https://web.archive.org/web/20220628060423/https://www.dddcommunity.org/book/evans_2003/)):

*   聚合是一组相关联的对象，作为数据更改的单个单元
*   关于聚合的引用仅限于单个成员，即聚合根
*   一组一致性规则适用于聚合边界内

正如第一点所说，**一个集合体不是一个单一的东西，而是一群`objects`** 。`Objects` 可以是[值对象](https://web.archive.org/web/20220628060423/https://martinfowler.com/bliki/ValueObject.html) **，但更重要的是，它们也可以是实体**。Axon 支持将聚合建模为一组相关联的对象，而不是单个对象，我们将在后面看到。

## 3.订单服务 API:命令和事件

当我们处理一个消息驱动的应用程序时，我们从定义新的命令开始，当扩展聚合来包含多个实体时。

我们的`Order`域目前包含一个`OrderAggregate`。这个集合中包含的一个逻辑概念是`OrderLine` 实体。订单行是指正在订购的特定产品，包括产品条目的总数。

了解了这一点，我们可以扩展命令 API——由一个`PlaceOrderCommand`、`ConfirmOrderCommand,`和`ShipOrderCommand`组成——增加三个操作:

*   添加产品
*   增加订单行的产品数量
*   减少订单行的产品数量

这些操作转化为类`AddProductCommand`、`IncrementProductCountCommand`和`DecrementProductCountCommand`:

```java
public class AddProductCommand {

    @TargetAggregateIdentifier
    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}

public class IncrementProductCountCommand {

    @TargetAggregateIdentifier
    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}

public class DecrementProductCountCommand {

    @TargetAggregateIdentifier
    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}
```

`[TargetAggregateIdentifier](https://web.archive.org/web/20220628060423/https://apidocs.axoniq.io/4.4/org/axonframework/modelling/command/TargetAggregateIdentifier.html)`仍然存在于`orderId`上，因为`OrderAggregate` 仍然是系统内的集合。

**从定义上记得，实体也有`an identity`。**这就是为什么`productId`是命令的一部分。在本文的后面，我们将展示这些字段如何引用一个精确的实体。

事件将作为命令处理的结果发布，通知相关的事情已经发生。因此，作为新命令 API 的结果，事件 API 也应该扩展。

让我们来看看反映增强的**连续性线索** — `ProductAddedEvent`、`ProductCountIncrementedEvent`、`ProductCountDecrementedEvent`和`ProductRemovedEvent`的 POJOs:

```java
public class ProductAddedEvent {

    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}

public class ProductCountIncrementedEvent {

    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}

public class ProductCountDecrementedEvent {

    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}

public class ProductRemovedEvent {

    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
}
```

## 4.总量和实体:实施

新的 API 规定我们可以添加一个产品并增加或减少它的计数。由于添加到`Order`的每个产品都会发生这种情况，我们需要定义不同的`order lines`来允许这些操作。**这表明需要添加一个属于`OrderAggregate`的`OrderLine`实体。**

在没有指导的情况下，Axon 不知道一个对象是否是一个集合中的实体。**我们应该将`[AggregateMember](https://web.archive.org/web/20220628060423/https://apidocs.axoniq.io/4.4/org/axonframework/modelling/command/AggregateMember.html)`注释放在一个公开实体的字段或方法上，以此来标记它。**

我们可以将这种注释用于单个对象、对象集合和地图。在`Order`域中，我们最好在`OrderAggregate. `上使用`OrderLine`实体的地图

### 4.1.总计调整

了解了这些，我们再来增强一下`OrderAggregate`:

```java
@Aggregate
public class OrderAggregate {

    @AggregateIdentifier
    private String orderId;
    private boolean orderConfirmed;

    @AggregateMember
    private Map<String, OrderLine> orderLines;

    @CommandHandler
    public void handle(AddProductCommand command) {
        if (orderConfirmed) {
            throw new OrderAlreadyConfirmedException(orderId);
        }

        String productId = command.getProductId();
        if (orderLines.containsKey(productId)) {
            throw new DuplicateOrderLineException(productId);
        }

        AggregateLifecycle.apply(new ProductAddedEvent(orderId, productId));
    }

    // previous command- and event sourcing handlers left out for conciseness

    @EventSourcingHandler
    public void on(OrderPlacedEvent event) {
        this.orderId = event.getOrderId();
        this.orderConfirmed = false;
        this.orderLines = new HashMap<>();
    }

    @EventSourcingHandler
    public void on(ProductAddedEvent event) {
        String productId = event.getProductId();
        this.orderLines.put(productId, new OrderLine(productId));
    }

    @EventSourcingHandler
    public void on(ProductRemovedEvent event) {
        this.orderLines.remove(event.getProductId());
    }
}
```

用`AggregateMember`注释标记`orderLines` 字段告诉 Axon 它是域模型的一部分。**这样做允许我们在`OrderLine`对象中添加 [`CommandHandler`](https://web.archive.org/web/20220628060423/https://apidocs.axoniq.io/4.4/org/axonframework/commandhandling/CommandHandler.html) 和 [`EventSourcingHandler`](https://web.archive.org/web/20220628060423/https://apidocs.axoniq.io/4.4/org/axonframework/eventsourcing/EventSourcingHandler.html) 带注释的方法，就像在集合中一样。**

由于`OrderAggregate`持有`OrderLine`实体，**负责添加和删除产品，因此也负责添加和删除相应的`OrderLines`。**应用程序使用[事件源](https://web.archive.org/web/20220628060423/https://martinfowler.com/eaaDev/EventSourcing.html)，所以有一个`ProductAddedEvent`和`ProductRemovedEvent` [`EventSourcingHandler`](https://web.archive.org/web/20220628060423/https://apidocs.axoniq.io/4.4/org/axonframework/eventsourcing/EventSourcingHandler.html) 分别添加和删除一个`OrderLine`。

`OrderAggregate`决定何时添加产品或拒绝添加，因为它持有`OrderLines.` ，这种所有权决定了`AddProductCommand`命令处理程序位于`OrderAggregate`内。

通过发布`ProductAddedEvent`通知添加成功。如果产品已经存在，则抛出`DuplicateOrderLineException`，如果`OrderAggregate`已经被确认，则抛出`OrderAlreadyConfirmedException`，添加失败。

最后，我们在`OrderPlacedEvent`处理程序**中设置了`orderLines`地图，因为它是`OrderAggregate`的事件流**中的第一个事件。我们可以在`OrderAggregate`或私有构造函数中全局设置该字段，但这意味着状态改变不再是事件源处理程序的唯一领域。

### 4.2.实体介绍

有了更新的`OrderAggregate`，我们可以开始看看`OrderLine`:

```java
public class OrderLine {

    @EntityId
    private final String productId;
    private Integer count;
    private boolean orderConfirmed;

    public OrderLine(String productId) {
        this.productId = productId;
        this.count = 1;
    }

    @CommandHandler
    public void handle(IncrementProductCountCommand command) {
        if (orderConfirmed) {
            throw new OrderAlreadyConfirmedException(orderId);
        }

        apply(new ProductCountIncrementedEvent(command.getOrderId(), productId));
    }

    @CommandHandler
    public void handle(DecrementProductCountCommand command) {
        if (orderConfirmed) {
            throw new OrderAlreadyConfirmedException(orderId);
        }

        if (count <= 1) {
            apply(new ProductRemovedEvent(command.getOrderId(), productId));
        } else {
            apply(new ProductCountDecrementedEvent(command.getOrderId(), productId));
        }
    }

    @EventSourcingHandler
    public void on(ProductCountIncrementedEvent event) {
        this.count++;
    }

    @EventSourcingHandler
    public void on(ProductCountDecrementedEvent event) {
        this.count--;
    }

    @EventSourcingHandler
    public void on(OrderConfirmedEvent event) {
        this.orderConfirmed = true;
    }
}
```

**`OrderLine`应可识别，如第 2 节**中所定义。实体可以通过`productId` 字段来识别，我们用`[EntityId](https://web.archive.org/web/20220628060423/https://apidocs.axoniq.io/4.4/org/axonframework/modelling/command/EntityId.html)`注释对其进行了标记。

**用`EntityId`注释标记一个字段告诉 Axon 哪个字段标识一个集合中的实体实例。**

由于`OrderLine`反映的是被订购的产品，它负责处理`IncrementProductCountCommand`和`DecrementProductCountCommand`。我们可以在实体内部使用`CommandHandler`注释，将这些命令直接路由到适当的实体。

当使用事件源时，`OrderLine`的状态需要根据事件来设置。与`OrderAggregate`类似，`OrderLine`可以简单地包含设置状态所需的事件的`EventSourcingHandler`注释。

通过使用`EntityId`注释字段将命令路由到正确的`OrderLine`实例。为了正确路由，**注释字段的名称应该与命令**中包含的字段之一相同。在这个示例中，这反映在命令和实体中的`productId`字段上。

每当实体存储在集合或映射中时，正确的命令路由使得`EntityId`成为一个硬性要求。如果只定义了聚合成员的单个实例，则此要求不严格。

每当命令中的名称不同于注释字段时，我们应该调整`EntityId`注释的`routingKey`值。`routingKey`值应该反映命令上的现有字段，以允许命令路由成功。

我们通过一个例子来解释一下:

```java
public class IncrementProductCountCommand {

    @TargetAggregateIdentifier
    private final String orderId;
    private final String productId;

    // default constructor, getters, equals/hashCode and toString
} ...
public class OrderLine {

    @EntityId(routingKey = "productId")
    private final String orderLineId;
    private Integer count;
    private boolean orderConfirmed;

    // constructor, command and event sourcing handlers
}
```

`IncrementProductCountCommand`保持不变，包含`orderId`集合标识符和`productId`实体标识符。在`OrderLine`实体中，标识符现在被称为`orderLineId`。

因为在`IncrementProductCountCommand,`中没有名为`orderLineId`的字段，这会破坏基于字段名`.` 的自动命令路由

**因此，`EntityId`注释上的`routingKey`字段应该反映命令中的字段名称，以保持这种路由能力。**

## 5.结论

在本文中，我们已经了解了包含多个实体的聚合意味着什么，以及 Axon Framework 如何支持这一概念。

我们增强了订单应用程序，允许订单行作为单独的实体属于`OrderAggregate`。

Axon 的聚合建模支持提供了`AggregateMember`注释，使用户能够将对象标记为给定聚合的实体。这样做允许命令直接路由到实体，并保持事件源支持。

所有这些例子的实现和代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220628060423/https://github.com/eugenp/tutorials/tree/master/axon)

关于这个主题的任何其他问题，也可以查看[讨论 AxonIQ](https://web.archive.org/web/20220628060423/https://discuss.axoniq.io/) 。