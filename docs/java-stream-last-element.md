# Java 中如何获取一个流的最后一个元素？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-last-element>

## 1.概观

Java API 是 Java 8 版本的主要特性。`Streams`表示延迟求值的对象序列，并提供丰富、流畅、类似单子的 API。

在这篇文章中，我们将快速查看获取最后一个元素`Stream.` 的方法。请记住**由于流的性质，它不是一个自然的操作**。始终确保您没有使用无限`Streams.`

## 2.使用`Reduce` API

`Reduce`，简单来说就是将一个`Stream`中的元素集合缩减为单个元素。

在这种情况下，我们将减少元素集，以获得最后一个元素 fn。请记住，**该方法将只为顺序的`Streams.`** 返回确定性结果

让我们使用`String`值中的一个`List`，从`List`中得到`Stream`，然后减少:

```java
List<String> valueList = new ArrayList<>();
valueList.add("Joe");
valueList.add("John");
valueList.add("Sean");

Stream<String> stream = valueList.stream();
stream.reduce((first, second) -> second)
  .orElse(null); 
```

这里，流被减少到只剩下最后一个元素的级别。如果流是空的，它将返回一个`null` 值。

## 2.使用跳过功能

获取流的最后一个元素的另一种方法是**，跳过它前面的所有元素**。这可以通过使用`Stream`类的`Skip`函数来实现。请记住，在这种情况下，我们消耗了两次`Stream`，因此有一些明显的性能影响**。**

让我们创建一个字符串值的`List`，并使用它的`size`函数来确定要跳过多少个元素才能到达最后一个元素。

下面是使用`skip`获取最后一个元素的示例代码:

```java
List<String> valueList = new ArrayList<String>();
valueList.add("Joe");
valueList.add("John");
valueList.add("Sean");

long count = valueList.stream().count();
Stream<String> stream = valueList.stream();

stream.skip(count - 1).findFirst().get(); 
```

最后成为最后一个元素。

## 4.获取无限流的最后一个元素

试图获取无限流的最后一个元素将导致对无限元素执行无限序列的求值。除非我们使用`limit`运算将无限流限制为特定数量的元素，否则`skip`和`reduce`都不会从求值的执行中返回。

下面是我们获取无限流并尝试获取最后一个元素的示例代码:

```java
Stream<Integer> stream = Stream.iterate(0, i -> i + 1);
stream.reduce((first, second) -> second).orElse(null);
```

因此，流将不会从评估中返回，并且它将终止**暂停程序**的执行。

## 5.结论

我们看到了使用`reduce`和`Skip`API 获得`Stream`的最后一个元素的不同方法。我们还研究了为什么这在无限流中是不可能的。

我们看到，与从其他数据结构中获取最后一个元素相比，从一个`Stream`中获取它并不容易。这是因为`Streams`的懒惰特性，除非调用终端函数，否则不会对其求值，并且**我们永远不知道当前求值的元素是否是最后一个。**

代码片段可以在 GitHub 上找到。