# 在 Java 映射中寻找最大值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-find-map-max>

## 1.概观

在这个快速教程中，**我们将探索在 Java `Map`** 中寻找最大值的各种方法。我们还将看到`Java 8`中的新特性是如何简化这一操作的。

在我们开始之前，让我们简要回顾一下在 Java 中如何比较[对象](/web/20220525124559/https://www.baeldung.com/java-comparator-comparable)。

通常，对象可以通过从`Comparable`接口实现方法`compareTo()`来表达自然排序。然而，通过`Comparator`对象可以采用除自然排序之外的排序。我们将在后面看到更多的细节。

## 2.Java 8 之前

我们先来探讨一下，在没有 Java 8 特性的情况下，如何才能找到最高的价值。

### 2.1.使用简单迭代

使用迭代，我们可以简单地遍历一个`Map`的所有条目来挑选最高值，将当前最高值存储在一个变量中:

```java
public <K, V extends Comparable<V>> V maxUsingIteration(Map<K, V> map) {
    Map.Entry<K, V> maxEntry = null;
    for (Map.Entry<K, V> entry : map.entrySet()) {
        if (maxEntry == null || entry.getValue()
            .compareTo(maxEntry.getValue()) > 0) {
            maxEntry = entry;
        }
    }
    return maxEntry.getValue();
}
```

这里，我们还利用 Java 泛型来构建一个可以应用于不同类型的方法。

### 2.2.使用`Collections.max()`

现在让我们看看`Collections`类中的实用方法`max()`如何让我们不用自己编写大量的代码:

```java
public <K, V extends Comparable<V>> V maxUsingCollectionsMax(Map<K, V> map) {
    Entry<K, V> maxEntry = Collections.max(map.entrySet(), new Comparator<Entry<K, V>>() {
        public int compare(Entry<K, V> e1, Entry<K, V> e2) {
            return e1.getValue()
                .compareTo(e2.getValue());
        }
    });
    return maxEntry.getValue();
}
```

在这个例子中，**我们将一个`Comparator`对象传递给`max()`** ，它可以通过`compareTo()`利用`Entry`值的自然排序或者实现一个完全不同的排序。

## 3.Java 8 之后

Java 8 的特性可以简化我们以上从一个`Map`中获得最大值的尝试。

### 3.1.使用带有 Lambda 表达式的`Collections.max()`

让我们从探究 lambda 表达式如何简化对`Collections.max()`的调用开始:

```java
public <K, V extends Comparable<V>> V maxUsingCollectionsMaxAndLambda(Map<K, V> map) {
    Entry<K, V> maxEntry = Collections.max(map.entrySet(), (Entry<K, V> e1, Entry<K, V> e2) -> e1.getValue()
        .compareTo(e2.getValue()));
    return maxEntry.getValue();
}
```

正如我们在这里看到的， **lambda 表达式让我们不用定义完整的函数接口**，并且提供了一种定义逻辑的简洁方法。要阅读更多关于 lambda 表达式的内容，也请查看我们之前的文章[。](/web/20220525124559/https://www.baeldung.com/java-8-lambda-expressions-tips)

### 3.2.使用`Stream`

`Stream` API 是`Java 8`的另一个补充，它极大地简化了集合的工作:

```java
public <K, V extends Comparable<V>> V maxUsingStreamAndLambda(Map<K, V> map) {
    Optional<Entry<K, V>> maxEntry = map.entrySet()
        .stream()
        .max((Entry<K, V> e1, Entry<K, V> e2) -> e1.getValue()
            .compareTo(e2.getValue())
        );

    return maxEntry.get().getValue();
}
```

这个 API 提供了很多数据处理查询，比如集合上的`map-reduce`转换。**这里，我们在`Map Entry`流中使用了`max()`，这是归约操作的一个特例。关于`Stream` API 的更多细节可以从[这里](/web/20220525124559/https://www.baeldung.com/java-8-streams-introduction)获得。**

这里我们还使用了`Optional` API，它是 Java 8 中添加的一个容器对象，可能包含也可能不包含非空值。更多关于`Optional`的细节可以从[这里](/web/20220525124559/https://www.baeldung.com/java-optional)获得。

### 3.3.通过方法引用使用`Stream`

最后，让我们看看方法引用如何进一步简化 lambda 表达式的使用:

```java
public <K, V extends Comparable<V>> V maxUsingStreamAndMethodReference(Map<K, V> map) {
    Optional<Entry<K, V>> maxEntry = map.entrySet()
        .stream()
        .max(Comparator.comparing(Map.Entry::getValue));
    return maxEntry.get()
        .getValue();
}
```

在 lambda 表达式仅仅调用现有方法的情况下，方法引用允许我们直接使用方法名来做这件事。关于 m `ethod references`的更多细节，请看一下[上一篇文章](/web/20220525124559/https://www.baeldung.com/java-8-double-colon-operator)。

## 4.结论

在本文中，我们已经看到了寻找 a `Java Map`中最高值的多种方法，其中一些使用了 Java 8 中添加的特性。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220525124559/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)