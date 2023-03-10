# 什么是 Hi/Lo 算法？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hi-lo-algorithm-hibernate>

## 1.介绍

在本教程中，我们将解释高/低算法。它主要用作数据库标识符生成策略。

我们将从算法概述开始。然后，我们将展示一个基于 Hibernate 框架的实际例子。最后，我们将讨论该算法的用例，它的优点和缺点。

## 2.高/低算法概述

### 2.1。定义

Hi/Lo 算法的主要目的是**创建一系列可以安全用作数据库标识符的数字**。为了做到这一点，它使用了三个数字变量，通常称为`high, low`和`incrementSize`。

`incrementSize`变量保存一批中可以生成的最大数量的标识符。它应该被视为在算法开始时定义的常数值。在多个客户机使用相同的 Hi/Lo 配置来保存条目的环境中，任何运行时修改都可能导致严重的问题。

`high`变量通常从数据库序列中赋值。在这种情况下，我们确信没有人会两次得到相同的号码。

`low`变量保存当前分配的数字，范围为[0 `, incrementSize`。

给定这些点，Hi/Lo 算法生成范围[(`hi` –1) `* incrementSize` +1`,` (`hi * incrementSize`)内的值。

### 2.2。伪代码

让我们看看使用 Hi/Lo 算法生成新值的步骤:

*   如果`low`大于或等于`incrementSize`，给`high`分配一个新值，并将`low`重置为 0
*   使用公式生成新值:(`high`–1)*`incrementSize`+`low`
*   将`low`增加 1
*   返回生成的值

## 3.实际例子

让我们看看高/低算法的实际应用。为此，我们将使用 Hibernate 框架及其 Hi/Lo 实现。

首先，让我们定义一个要使用的数据库实体:

```java
@Entity
public class RestaurantOrder {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "hilo_sequence_generator")
    @GenericGenerator(
      name = "hilo_sequence_generator",
      strategy = "sequence",
      parameters = {
        @Parameter(name = "sequence_name", value = "hilo_seqeunce"),
        @Parameter(name = "initial_value", value = "1"),
        @Parameter(name = "increment_size", value = "3"),
        @Parameter(name = "optimizer", value = "hilo")
      }
    )
    private Long id;
}
```

这是一个简单的餐馆订单，只有一个`id`字段。为了在 Hibernate 中正确定义 Hi/Lo 算法，在`id`字段的定义中，我们必须选择一个`sequence`策略—`hilo`优化器—并指定`increment_size`参数。

为了展示 Hi/Lo 算法的实际应用，我们将在一个循环中保存九个餐馆订单:

```java
public void persist() {
    Transaction transaction = session.beginTransaction();

    for (int i = 0; i < 9; i++) {
        session.persist(new RestaurantOrder());
        session.flush();
    }

    transaction.commit();
}
```

根据实体中指定的增量大小，对于下一个`high`值，我们应该只有三次对数据库的调用。假设数据库序列从 1 开始，第一批生成的标识符将在范围[1，3]内。

当 Hi/Lo 算法返回 3，Hibernate 请求下一个标识符的值时，`low`变量的值等于`incrementSize`常量。在这种情况下，必须对数据库进行下一次调用，以获取新的`high`值。将 2 作为新的`high`值，该算法生成范围为[4，6]的值。

最后，最后一次调用数据库获取下一个`high`值，范围【7，9】内的值被分配给实体。

在执行 `persist()`方法期间捕获的休眠日志确认了这些值:

```java
Hibernate: call next value for hilo_seqeunce
org.hibernate.id.enhanced.SequenceStructure  - Sequence value obtained: 1
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 1, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 2, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 3, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
Hibernate: call next value for hilo_seqeunce
org.hibernate.id.enhanced.SequenceStructure  - Sequence value obtained: 2
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 4, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 5, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 6, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
Hibernate: call next value for hilo_seqeunce
org.hibernate.id.enhanced.SequenceStructure  - Sequence value obtained: 3
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 7, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 8, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
org.hibernate.event.internal.AbstractSaveEventListener  - Generated identifier: 9, using strategy: org.hibernate.id.enhanced.SequenceStyleGenerator
```

## 4.算法的优点和缺点

Hi/Lo 算法的主要优点是**减少了下一个序列值的数据库调用**次数。增加`incrementSize`的值会减少到数据库的往返次数。显然，这意味着我们的应用程序的性能提高了。除此之外，Hi/Lo 算法是互联网连接弱的环境中的首选**。**

 **另一方面，Hi/Lo 算法**在多个不同客户端将数据保存到数据库**的同一个表中的环境中并不是最佳选择。第三方应用程序可能不知道我们用来生成标识符的 Hi/Lo 策略。因此，他们可能会使用我们的应用程序中当前使用的生成的数字范围中的实体 id。在这种情况下，当持久化数据时，我们可能会遇到难以修复的错误。

## 5.结论

在本教程中，我们讨论了高/低算法。

首先，我们解释了它是如何工作的，并讨论了它的伪代码实现。然后，我们展示了一个使用 Hibernate 算法实现的实际例子。最后，我们列出了 Hi/Lo 的优点和缺点。

和往常一样，本文中显示的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628091437/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)**