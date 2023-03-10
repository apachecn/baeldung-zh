# Java 8 流跳过()与限制()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-skip-vs-limit>

## 1。简介

在这篇短文中，我们将讨论 [Java 流 API](/web/20221020160506/https://www.baeldung.com/java-8-streams) 的`skip()`和`limit()`方法，并强调它们的异同。

尽管这两个操作乍看起来非常相似，但实际上它们的行为非常不同，并且不可互换。实际上，它们是互补的，在一起使用时会很方便。请继续阅读，了解更多关于它们的信息。

## 2。`skip()`法

`skip(n)`方法是一个**中间操作，它丢弃流**的第一个`n`元素。`n`参数不能为负，如果大于流的大小，`skip()`返回一个空流。

让我们看一个例子:

```java
Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .filter(i -> i % 2 == 0)
    .skip(2)
    .forEach(i -> System.out.print(i + " "));
```

在这个流中，我们选择流中的偶数，但是我们跳过前两个。因此，我们的结果是:

```java
6 8 10
```

当这个流被执行时，`forEach`开始请求项目。当到达`skip()`时，这个操作知道前两项必须被丢弃，所以它不会将它们添加到结果流中。之后，它创建并返回一个包含剩余项目的流。

为了做到这一点，`skip()`操作必须保持元素在每个时刻的状态。由于这个原因，**我们说`skip()`是有状态操作**。

## 3。`limit()`法

`limit(n)`方法是另一个**中间操作，它返回一个不长于请求大小**的流。和以前一样，`n`参数不能为负。

让我们在一个例子中使用它:

```java
Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .filter(i -> i % 2 == 0)
    .limit(2)
    .forEach(i -> System.out.print(i + " "));
```

在这种情况下，我们只从我们的`int`流中选取两个偶数:

```java
2 4
```

与`skip()`操作一样，`limit()`也是一个有状态操作，因为它必须保持被拾取的项目的状态。

但是与消耗整个流的 `skip()`不同，一旦`limit()`达到最大项数，它就不再消耗更多的项，而只是返回结果流。因此，**我们说`limit()`是一个短路操作**。

当处理无限流时，`limit()`对于将流截断成有限流非常有用:

```java
Stream.iterate(0, i -> i + 1)
    .filter(i -> i % 2 == 0)
    .limit(10)
    .forEach(System.out::println);
```

在这个例子中，我们将一个无限的数字流截断成一个只有十个偶数的流。

## 4。组合`skip()`和`limit()`和

正如我们前面提到的，`skip`和`limit`操作是互补的，如果我们将它们结合起来，它们在某些情况下会非常有帮助。

假设我们想修改前面的例子，使它以十为一批得到偶数。我们可以简单地通过在同一个流中使用`skip()`和`limit()`来实现:

```java
private static List<Integer> getEvenNumbers(int offset, int limit) {
    return Stream.iterate(0, i -> i + 1)
        .filter(i -> i % 2 == 0)
        .skip(offset)
        .limit(limit)
        .collect(Collectors.toList());
}
```

正如我们所看到的，使用这种方法，我们可以很容易地对流进行分页。尽管这是一个非常简单的分页，但我们可以看到在对流进行切片时，这是多么强大。

## 5。结论

在这篇简短的文章中，我们展示了 Java Stream API 的`skip()`和`limit()`方法的相似之处和不同之处。我们还实现了一些简单的例子来展示如何使用这些方法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221020160506/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-2)