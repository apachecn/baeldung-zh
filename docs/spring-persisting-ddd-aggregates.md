# 持久性 DDD 聚集物

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-persisting-ddd-aggregates>

## 1。概述

在本教程中，我们将探索使用不同技术持久化 [DDD 聚集](https://web.archive.org/web/20220525132033/https://martinfowler.com/bliki/DDD_Aggregate.html)的可能性。

## 2。骨料介绍

**聚合是一组总是需要保持一致的业务对象**。因此，我们在一个事务中整体保存和更新聚合。

聚合是 DDD 的一个重要的战术模式，它有助于保持我们业务对象的一致性。然而，聚合的概念在 DDD 环境之外也是有用的。

这种模式在很多商业案例中都可以派上用场。**根据经验，当在同一个事务**中有多个对象发生变化时，我们应该考虑使用聚合。

让我们看看在对订单购买建模时如何应用这一点。

### 2.1.采购订单示例

因此，让我们假设我们想要对采购订单进行建模:

```java
class Order {
    private Collection<OrderLine> orderLines;
    private Money totalCost;
    // ...
}
```

```java
class OrderLine {
    private Product product;
    private int quantity;
    // ...
}
```

```java
class Product {
    private Money price;
    // ...
}
```

这些类形成了一个简单的集合。`Order`的`orderLines`和`totalCost`字段必须始终一致，即`totalCost`的值应该始终等于所有`orderLines`的总和。

**现在，我们可能都想把所有这些都变成成熟的 Java Beans。**但是，注意在`Order`中引入简单的 getters 和 setters 很容易破坏我们模型的封装并违反业务约束。

让我们看看会出什么问题。

### 2.2.简单总体设计

让我们想象一下，如果我们天真地决定在`Order`类的所有属性中添加 getters 和 setters，包括`setOrderTotal`，会发生什么。

没有任何东西禁止我们执行下面的代码:

```java
Order order = new Order();
order.setOrderLines(Arrays.asList(orderLine0, orderLine1));
order.setTotalCost(Money.zero(CurrencyUnit.USD)); // this doesn't look good...
```

在这段代码中，我们手动将`totalCost`属性设置为零，违反了一条重要的业务规则。肯定的，总成本不应该是零美元！

我们需要一种方法来保护我们的业务规则。让我们看看聚合根是如何帮助我们的。

### 2.3.聚集根

An `aggregate root`是一个类，它是我们聚合的入口点。**所有的业务操作都要经过根。**通过这种方式，聚合根可以保证聚合保持一致的状态。

**根负责我们所有的业务不变量**。

在我们的例子中，`Order`类是聚合根的合适候选。我们只需要做一些修改，以确保聚合始终一致:

```java
class Order {
    private final List<OrderLine> orderLines;
    private Money totalCost;

    Order(List<OrderLine> orderLines) {
        checkNotNull(orderLines);
        if (orderLines.isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one order line item");
        }
        this.orderLines = new ArrayList<>(orderLines);
        totalCost = calculateTotalCost();
    }

    void addLineItem(OrderLine orderLine) {
        checkNotNull(orderLine);
        orderLines.add(orderLine);
        totalCost = totalCost.plus(orderLine.cost());
    }

    void removeLineItem(int line) {
        OrderLine removedLine = orderLines.remove(line);
        totalCost = totalCost.minus(removedLine.cost());
    }

    Money totalCost() {
        return totalCost;
    }

    // ...
}
```

使用聚合根现在允许我们更容易地将`Product`和`OrderLine`变成不可变的对象，其中所有的属性都是最终的。

如我们所见，这是一个非常简单的集合。

而且，我们可以简单地计算每次的总成本，而不用使用字段。

然而，现在我们只是在谈论聚合持久性，而不是聚合设计。请继续关注，因为这个特定的领域很快就会派上用场。

这对持久性技术有多好？让我们来看看。最终，这将帮助我们为下一个项目选择正确的持久工具。

## 3.JPA 和 Hibernate

在本节中，让我们尝试使用 JPA 和 Hibernate 来持久化我们的`Order`聚合。我们将使用 Spring Boot 和 [JPA](https://web.archive.org/web/20220525132033/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa) 启动器:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

对于我们大多数人来说，这似乎是最自然的选择。毕竟，我们已经用关系系统工作了很多年，我们都知道流行的 ORM 框架。

**使用 ORM 框架时最大的问题可能是模型设计的简化**。它有时也被称为[对象关系阻抗不匹配](https://web.archive.org/web/20220525132033/https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)。让我们想一想，如果我们想要持久化我们的`Order`聚合，会发生什么:

```java
@DisplayName("given order with two line items, when persist, then order is saved")
@Test
public void test() throws Exception {
    // given
    JpaOrder order = prepareTestOrderWithTwoLineItems();

    // when
    JpaOrder savedOrder = repository.save(order);

    // then
    JpaOrder foundOrder = repository.findById(savedOrder.getId())
      .get();
    assertThat(foundOrder.getOrderLines()).hasSize(2);
}
```

此时，这个测试会抛出一个异常:`java.lang.IllegalArgumentException: Unknown entity: com.baeldung.ddd.order.Order`。**显然，我们遗漏了一些 JPA 要求:**

1.  添加映射注释
2.  `OrderLine`和`Product`类必须是实体或`@Embeddable`类，而不是简单的值对象
3.  为每个实体或`@Embeddable`类添加一个空构造函数
4.  用简单类型替换`Money`属性

**嗯，我们需要修改`Order` aggregate 的设计，以便能够使用 JPA。虽然添加注释没什么大不了的，但是其他需求会带来很多问题。**

### 3.1.对值对象的更改

试图将一个集合放入 JPA 的第一个问题是，我们需要打破我们的值对象的设计:它们的属性不再是最终的，我们需要打破封装。

**我们需要给`OrderLine`和`Product,`添加人工 id，即使这些类从未被设计成有标识符**。我们希望它们是简单的值对象。

可以使用`@Embedded`和`@ElementCollection`注释来代替，但是当使用复杂的对象图时，这种方法会使事情变得复杂很多(例如`@Embeddable`对象有另一个`@Embedded`属性等等)。).

使用`@Embedded`注释只是将平面属性添加到父表中。除此之外，基本属性(例如`String`类型)仍然需要一个 setter 方法，这违反了所需的值对象设计。

空的构造函数要求迫使值对象属性不再是最终的，打破了我们最初设计的一个重要方面。说实话，Hibernate 可以使用私有的无参数构造函数，这稍微缓解了这个问题，但是它还远远不够完美。

即使使用私有默认构造函数，我们也不能将属性标记为 final，或者需要在默认构造函数中用默认值(通常为 null)初始化它们。

然而，如果我们想要完全符合 JPA，我们必须至少为默认构造函数使用受保护的可见性，这意味着同一个包中的其他类可以创建值对象，而无需指定它们的属性值。

### 3.2.复杂类型

不幸的是，我们不能期望 JPA 自动将第三方复杂类型映射到表中。看看我们在上一节中引入了多少变化！

例如，当使用我们的`Order`聚合时，我们会在持久化`Joda Money`字段时遇到困难。

在这种情况下，我们可能会以编写 JPA 2.1 中可用的自定义类型`@Converter`而告终。不过，这可能需要一些额外的工作。

或者，我们也可以将`Money`属性拆分成两个基本属性。例如，`String`表示货币单位，`BigDecimal`表示实际值。

虽然我们可以隐藏实现细节，并且仍然通过公共方法 API 使用`Money`类，但是实践表明大多数开发人员无法证明额外的工作是合理的，并且会简单地简化模型以符合 JPA 规范。

### 3.3.结论

虽然 JPA 是世界上被采用最多的规范之一，但它可能不是持久化我们的`Order`聚合的最佳选择。

如果我们希望我们的模型反映真实的业务规则，我们应该把它设计成不是底层表的简单的 1:1 表示。

基本上，我们有三个选择:

1.  创建一组简单的数据类，并使用它们来持久化和重新创建丰富的业务模型。不幸的是，这可能需要很多额外的工作。
2.  接受 JPA 的局限性，选择正确的折中方案。
3.  考虑另一种技术。

第一种选择潜力最大。实际上，大多数项目都是使用第二种方法开发的。

现在，让我们考虑另一种持久化聚合的技术。

## 4.文档存储

文档存储是存储数据的另一种方式。我们不使用关系和表格，而是保存整个对象。**这使得文档存储成为持久化聚集的潜在完美候选者**。

出于本教程的需要，我们将关注类似 JSON 的文档。

让我们仔细看看我们的订单持久性问题在像 MongoDB 这样的文档存储中是怎样的。

### 4.1.使用 MongoDB 持久化聚合

现在，有相当多的数据库可以存储 JSON 数据，其中一个流行的是 MongoDB。 MongoDB 实际上以二进制形式存储 BSON，或者 JSON。

**多亏了 MongoDB，我们可以存储`Order`示例集合 `as-is`。**

在我们继续之前，让我们添加 Spring Boot [MongoDB](https://web.archive.org/web/20220525132033/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-mongodb) 启动器:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

现在我们可以运行一个类似于 JPA 示例中的测试用例，但是这次使用 MongoDB:

```java
@DisplayName("given order with two line items, when persist using mongo repository, then order is saved")
@Test
void test() throws Exception {
    // given
    Order order = prepareTestOrderWithTwoLineItems();

    // when
    repo.save(order);

    // then
    List<Order> foundOrders = repo.findAll();
    assertThat(foundOrders).hasSize(1);
    List<OrderLine> foundOrderLines = foundOrders.iterator()
      .next()
      .getOrderLines();
    assertThat(foundOrderLines).hasSize(2);
    assertThat(foundOrderLines).containsOnlyElementsOf(order.getOrderLines());
}
```

**重要的是——我们根本没有改变最初的`Order`聚合类；**不需要为`Money`类创建默认的构造器、设置器或自定义转换器。

下面是我们的`Order`汇总在商店中显示的内容:

```java
{
  "_id": ObjectId("5bd8535c81c04529f54acd14"),
  "orderLines": [
    {
      "product": {
        "price": {
          "money": {
            "currency": {
              "code": "USD",
              "numericCode": 840,
              "decimalPlaces": 2
            },
            "amount": "10.00"
          }
        }
      },
      "quantity": 2
    },
    {
      "product": {
        "price": {
          "money": {
            "currency": {
              "code": "USD",
              "numericCode": 840,
              "decimalPlaces": 2
            },
            "amount": "5.00"
          }
        }
      },
      "quantity": 10
    }
  ],
  "totalCost": {
    "money": {
      "currency": {
        "code": "USD",
        "numericCode": 840,
        "decimalPlaces": 2
      },
      "amount": "70.00"
    }
  },
  "_class": "com.baeldung.ddd.order.mongo.Order"
}
```

这个简单的 BSON 文档包含了整个`Order`集合，非常符合我们最初的概念，即所有这些应该是共同一致的。

注意，BSON 文档中的复杂对象只是被序列化为一组常规的 JSON 属性。由于这一点，即使是第三方类(如`Joda Money`)也可以很容易地序列化，而不需要简化模型。

### 4.2.结论

使用 MongoDB 持久化聚合比使用 JPA 简单。

这绝对不意味着 MongoDB 优于传统数据库。在很多合理的情况下，我们甚至不应该尝试将我们的类建模为聚合，而是使用 SQL 数据库。

尽管如此，当我们根据复杂的需求确定了一组应该始终保持一致的对象时，使用文档存储可能是一个非常有吸引力的选择。

## 5。结论

在 DDD，集合通常包含系统中最复杂的对象。与大多数 CRUD 应用程序相比，使用它们需要非常不同的方法。

使用流行的 ORM 解决方案可能会导致过于简单或过度暴露的领域模型，这通常无法表达或执行复杂的业务规则。

文档存储可以在不牺牲模型复杂性的情况下，使聚合更容易持久化。

GitHub 上的[提供了所有示例的完整源代码。](https://web.archive.org/web/20220525132033/https://github.com/eugenp/tutorials/tree/master/ddd)