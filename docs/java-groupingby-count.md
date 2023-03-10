# 使用 Java 分组按收集器计数出现次数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-groupingby-count>

## 1。概述

在这个简短的教程中，我们将看到如何在 Java 中对相等的对象进行分组并计算它们的出现次数。我们将在 Java 中使用 [`groupingBy()`收集器](/web/20221208143813/https://www.baeldung.com/java-groupingby-collector)。

## 2。使用`Collectors.groupingBy()` 统计出现次数

**`Collectors.groupingBy()`提供了类似于 SQL 中 GROUP BY 子句的功能。**我们可以用它按照任何属性对对象进行分组，并将结果存储在`Map`中。

例如，让我们考虑一个场景，我们需要在一个流中分组相等的`String`并计算它们的出现次数:

```java
List<String> list = new ArrayList<>(Arrays.asList("Foo", "Bar", "Bar", "Bar", "Foo"));
```

我们可以将相等的字符串分组，在本例中是“Foo”和“Bar”。结果`Map`将把这些字符串存储为键。这些键的值将是出现的次数。“Foo”的值将是 2，“Bar”的值将是 3:

```java
Map<String, Long> result = list.stream()
  .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
Assert.assertEquals(new Long(2), result.get("Foo"));
Assert.assertEquals(new Long(3), result.get("Bar"));
```

让我们来解码上面的代码片段:

*   `Map<String, Long> result`–这是输出结果`Map`，它将分组元素存储为键，并将它们的出现次数计为值
*   将列表元素转换成 Java 流，以声明的方式处理集合
*   `Collectors.groupingBy()`–这是`Collectors`类的方法，通过一些属性对对象进行分组，并将结果存储在`Map`实例中
*   `Function.identity() –` 是 Java 里的一个[函数接口](/web/20221208143813/https://www.baeldung.com/java-8-functional-interfaces)；`identity `方法返回一个总是返回其输入参数的`Function`
*   `Collectors.counting()`–这个`Collectors`类方法将流中传递的元素数量作为一个参数进行计数

我们可以用`Collectors.groupingByConcurrent()` 代替`Collectors.groupingBy().` ，它也对输入流元素执行 group by 操作。该方法将结果收集在`ConcurrentMap`中，提高了效率。

例如，对于输入列表:

```java
List<String> list = new ArrayList<>(Arrays.asList("Adam", "Bill", "Jack", "Joe", "Ian"));
```

我们可以使用`Collectors.groupingByConcurrent()`将等长的字符串分组:

```java
Map<Integer, Long> result = list.stream()
  .collect(Collectors.groupingByConcurrent(String::length, Collectors.counting()));
Assert.assertEquals(new Long(2), result.get(3));
Assert.assertEquals(new Long(3), result.get(4));
```

## 3。结论

在本文中，我们介绍了如何使用 `Collector.groupingBy()`对相等的对象进行分组。

最后，你可以在 GitHub 上找到这篇文章[的源代码。](https://web.archive.org/web/20221208143813/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-4)