# 番石榴多集指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-multiset>

## 1.概观

在本教程中，我们将探索[番石榴](/web/20220524122138/https://www.baeldung.com/category/guava/)系列之一—`Multiset`。像`java.util.Set`一样，它允许在没有保证顺序的情况下高效地存储和检索项目。

然而，与`Set`不同，它允许**多次出现相同的元素**，通过跟踪它包含的每个唯一元素的计数。

## 2.Maven 依赖性

首先，让我们添加[的`guava` 依赖](https://web.archive.org/web/20220524122138/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22):

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

## 3.使用`Multiset`

让我们考虑一个书店，它有不同书籍的多个副本。我们可能希望执行一些操作，比如添加一个副本，获取副本的数量，以及在售出后删除一个副本。**因为`Set`不允许同一元素多次出现，所以它不能处理这个需求。**

**让我们从添加书名的**副本开始。`Multiset`应该返回标题存在，并向我们提供正确的计数`:`

```java
Multiset<String> bookStore = HashMultiset.create();
bookStore.add("Potter");
bookStore.add("Potter");
bookStore.add("Potter");

assertThat(bookStore.contains("Potter")).isTrue();
assertThat(bookStore.count("Potter")).isEqualTo(3);
```

现在让我们删除一个副本。我们希望相应地更新计数:

```java
bookStore.remove("Potter");
assertThat(bookStore.contains("Potter")).isTrue();
assertThat(bookStore.count("Potter")).isEqualTo(2);
```

实际上，**我们可以只设置计数**而不是执行各种加法运算:

```java
bookStore.setCount("Potter", 50); 
assertThat(bookStore.count("Potter")).isEqualTo(50);
```

`Multiset`验证`count`值。如果我们把它设为负数，就会抛出一个`IllegalArgumentException`:

```java
assertThatThrownBy(() -> bookStore.setCount("Potter", -1))
  .isInstanceOf(IllegalArgumentException.class);
```

## 4.与`Map`的比较

没有对`Multiset`的访问，我们可以通过使用`java.util.Map:`实现我们自己的逻辑来实现上面所有的操作

```java
Map<String, Integer> bookStore = new HashMap<>();
// adding 3 copies
bookStore.put("Potter", 3);

assertThat(bookStore.containsKey("Potter")).isTrue();
assertThat(bookStore.get("Potter")).isEqualTo(3);

// removing 1 copy
bookStore.put("Potter", 2);
assertThat(bookStore.get("Potter")).isEqualTo(2);
```

当我们想要使用`Map`添加或删除副本时，我们需要记住当前的计数并相应地进行调整。我们还需要每次在调用代码中实现这个逻辑，或者为此构建我们自己的库。我们的代码还需要控制`value`参数。如果我们不小心，我们很容易将值设置为`null`或负值，即使这两个值都是无效的:

```java
bookStore.put("Potter", null);
assertThat(bookStore.containsKey("Potter")).isTrue();

bookStore.put("Potter", -1);
assertThat(bookStore.containsKey("Potter")).isTrue(); 
```

我们可以看到，用`Multiset`代替`Map`要方便很多。

## 5.并发

当我们想在并发环境中使用`Multiset`时，我们可以使用`ConcurrentHashMultiset`，这是一个线程安全的`Multiset`实现。

但是，我们应该注意，线程安全并不能保证一致性。使用`add`或`remove`方法将在多线程环境中运行良好，**但是如果几个线程调用了 `setCount` 方法呢？**

如果我们使用`setCount` 、**方法，最终结果将取决于线程间的执行顺序**，这是不可预测的。`add `和`remove`方法是增量的，`ConcurrentHashMultiset`能够保护它们的行为。**直接设置计数不是递增的，因此在同时使用时会导致意外的结果。**

然而，还有另一种风格的 `setCount` 方法，它只在当前值与传递的参数匹配时更新计数。如果操作成功，该方法返回 true，这是一种乐观锁定:

```java
Multiset<String> bookStore = HashMultiset.create();
// updates the count to 2 if current count is 0
assertThat(bookStore.setCount("Potter", 0, 2)).isTrue();
// updates the count to 5 if the current value is 50
assertThat(bookStore.setCount("Potter", 50, 5)).isFalse();
```

如果要在并发代码中使用`setCount`方法，应该使用上面的版本来保证一致性。如果更改计数失败，多线程客户端可以执行重试。

## 6.结论

在这个简短的教程中，我们讨论了何时以及如何使用一个`Multiset,` 与一个标准的`Map` 进行比较，并查看如何在并发应用程序中最好地使用它。

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524122138/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-set)