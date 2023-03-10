# 在 Java 中迭代地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterate-map>

## 1。概述

在这个快速教程中，我们将看看 Java 中遍历`Map`条目的不同方式。

简单地说，我们可以使用`entrySet()`、`keySet()`或 `values()`来提取`Map`的内容。因为这些都是集合，相似的迭代原则适用于它们。

让我们仔细看看其中的几个。

## 延伸阅读:

## [Java 8 forEach 指南](/web/20221108150351/https://www.baeldung.com/foreach-java)

A quick and practical guide to Java 8 forEach[Read more](/web/20221108150351/https://www.baeldung.com/foreach-java) →

## [如何迭代带有索引的流](/web/20221108150351/https://www.baeldung.com/java-stream-indices)

Learn several ways of iterating over Java 8 Streams using indices[Read more](/web/20221108150351/https://www.baeldung.com/java-stream-indices) →

## [在 Java 映射中寻找最高值](/web/20221108150351/https://www.baeldung.com/java-find-map-max)

Take a look at ways to find the maximum value in a Java Map structure.[Read more](/web/20221108150351/https://www.baeldung.com/java-find-map-max) →

## 2.简要介绍`Map`的`entrySet(), keySet()`和`values()`方法

在我们使用这三种方法遍历地图之前，让我们了解一下这些方法的作用:

*   [`entrySet()`](https://web.archive.org/web/20221108150351/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#entrySet())–返回地图的集合视图，其元素来自`Map.Entry`类。`entry.getKey()`方法返回键，`entry.getValue()`返回相应的值
*   [`keySet()`](https://web.archive.org/web/20221108150351/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#keySet())–以`Set`的形式返回此地图中包含的所有键
*   [`values()`](https://web.archive.org/web/20221108150351/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#values())–返回此地图中包含的所有值作为`Set`

Next, let's see these methods in action.

## 3。使用`for`循环

### 3.1.使用`entrySet()`

首先，让我们看看如何使用`Entry` `**Set**`通过`Map`进行迭代:

```java
public void iterateUsingEntrySet(Map<String, Integer> map) {
    for (Map.Entry<String, Integer> entry : map.entrySet()) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

这里，我们从我们的`Map`中提取条目的`Set`，然后使用经典的 for-each 方法遍历它们。

### 3.2.使用`keySet()`

或者，我们可以首先使用`keySet`方法获取`Map`中的所有键，然后通过每个键遍历映射:

```java
public void iterateUsingKeySetAndForeach(Map<String, Integer> map) {
    for (String key : map.keySet()) {
        System.out.println(key + ":" + map.get(key));
    }
}
```

### 3.3.使用`values()`迭代值

有时，我们**只对地图中的值感兴趣，不管哪些键与它们相关联**。在这种情况下，`values()`是我们最好的选择:

```java
public void iterateValues(Map<String, Integer> map) {
    for (Integer value : map.values()) {
        System.out.println(value);
    }
} 
```

## 4。`Iterator`

执行迭代的另一种方法是使用 [`Iterator`](/web/20221108150351/https://www.baeldung.com/java-iterator) 。接下来，让我们看看这些方法如何处理一个`Iterator`对象。

### 4.1。`Iterator`和`entrySet()`

首先，让我们使用迭代器和`entrySet()`遍历地图:

```java
public void iterateUsingIteratorAndEntry(Map<String, Integer> map) {
    Iterator<Map.Entry<String, Integer>> iterator = map.entrySet().iterator();
    while (iterator.hasNext()) {
        Map.Entry<String, Integer> entry = iterator.next();
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

注意我们如何使用由`entrySet().` 返回的`Set`的`iterator()` API 来获得`Iterator`实例，然后像往常一样，我们用`iterator.next().`循环通过`Iterator`

### 4.2.`Iterator`和`keySet()`

类似地，我们可以使用`Iterator`和`keySet()`来迭代`Map`:

```java
public void iterateUsingIteratorAndKeySet(Map<String, Integer> map) {
    Iterator<String> iterator = map.keySet().iterator();
    while (iterator.hasNext()) {
        String key = iterator.next();
        System.out.println(key + ":" + map.get(key));
    }
} 
```

### 4.3.`Iterator`和`values()`

我们还可以使用`Iterator`和`values()`方法遍历地图的值:

```java
public void iterateUsingIteratorAndValues(Map<String, Integer> map) {
    Iterator<Integer> iterator = map.values().iterator();
    while (iterator.hasNext()) {
        Integer value = iterator.next();
        System.out.println("value :" + value);
    }
}
```

## 5.使用 Lambdas 和流 API

从版本 8 开始，Java 引入了[流 API](/web/20221108150351/https://www.baeldung.com/java-8-streams) 和 lambdas。接下来，让我们看看如何使用这些技术迭代地图。

### 5.1.使用`forEach()`和λ

像 Java 8 中的大多数其他东西一样，这比其他选择要简单得多。我们将利用`forEach()`方法:

```java
public void iterateUsingLambda(Map<String, Integer> map) {
    map.forEach((k, v) -> System.out.println((k + ":" + v)));
} 
```

在这种情况下，我们不需要将地图转换为一组条目。为了学习更多关于 lambda 表达式的知识，我们可以[从这里](/web/20221108150351/https://www.baeldung.com/java-8-lambda-expressions-tips)开始。

当然，我们可以从键开始迭代地图:

```java
public void iterateByKeysUsingLambda(Map<String, Integer> map) {
    map.keySet().foreach(k -> System.out.println((k + ":" + map.get(k))));
} 
```

类似地，我们可以对`values()`方法使用相同的技术:

```java
public void iterateValuesUsingLambda(Map<String, Integer> map) {
    map.values().forEach(v -> System.out.println(("value: " + v)));
} 
```

### 5.2。使用`Stream` API

API 是 Java 8 的一个重要特性。我们也可以使用这个特性来遍历一个`Map`。

**`Stream`当我们计划做一些额外的`Stream`处理时，应该使用 API 否则，这只是一个简单的 `forEach()`如前所述。**

让我们以`entrySet()`为例来看看`Stream` API 是如何工作的:

```java
public void iterateUsingStreamAPI(Map<String, Integer> map) {
    map.entrySet().stream()
      // ... some other Stream processings
      .forEach(e -> System.out.println(e.getKey() + ":" + e.getValue()));
} 
```

[`Stream` API](/web/20221108150351/https://www.baeldung.com/java-8-streams) 与`keySet()`和`values()`方法的用法与上面的例子非常相似。

## 6。结论

在本文中，我们关注一个关键但简单的操作:遍历一个`Map`的条目。

我们探索了一些只能用于 Java 8+的方法，即 Lambda 表达式和`Stream` API。

和往常一样，本文中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221108150351/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)