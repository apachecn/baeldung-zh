# Java 中两个列表的交集

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lists-intersection>

## 1。概述

在本教程中，我们将学习如何检索两个`List`的交集。

像许多其他事情一样，由于在 Java 8 中引入了[流，这变得容易多了。](/web/20220926180850/https://www.baeldung.com/java-streams)

## 2。两个字符串列表的交集

让我们创建两个有交集的`String`的`List`——两个都有一些重复的元素:

```java
List<String> list = Arrays.asList("red", "blue", "blue", "green", "red");
List<String> otherList = Arrays.asList("red", "green", "green", "yellow");
```

现在**我们将在流方法**的帮助下确定列表的交集:

```java
Set<String> result = list.stream()
  .distinct()
  .filter(otherList::contains)
  .collect(Collectors.toSet());

Set<String> commonElements = new HashSet(Arrays.asList("red", "green"));

Assert.assertEquals(commonElements, result);
```

首先，我们用`distinct`删除重复的元素。然后，我们使用 `filter`来选择也包含在`otherList`中的元素。

最后，我们用一个`Collector`转换我们的输出。交集应该只包含每个公共元素一次。顺序不重要，因此`toSet`是最直接的选择，但是我们也可以使用`toList`或其他收集器方法。

要了解更多细节，请查看我们的 Java 8 收集器指南。

## 3。自定义类别列表的交集

如果我们的`List`不包含`String`而是包含我们创建的自定义类的实例，那会怎么样？好吧，只要我们遵循 Java 的约定，流方法的解决方案对我们的自定义类来说就很好。

**方法`contains`如何决定一个特定的对象是否出现在列表中？基于`equals`的方法。因此，我们必须覆盖`equals`方法，并确保它根据相关属性的值来比较两个对象。**

例如，如果两个矩形的宽度和高度相等，则它们相等。

如果我们不覆盖`equals`方法，我们的类使用父类的`equals`实现。在一天结束时，或者更确切地说，在继承链中，`Object`类的`equals`方法被执行。那么只有当两个实例引用堆上完全相同的对象时，它们才是相等的。

有关`equals`方法的更多信息，请参见我们关于 [Java `equals()`和`hashCode()`契约](/web/20220926180850/https://www.baeldung.com/java-equals-hashcode-contracts)的文章。

## 4。结论

在这篇简短的文章中，我们看到了如何使用流来计算两个列表的交集。有许多其他的操作过去是相当乏味的，但是如果我们了解 Java Stream API 的话，它们是非常简单的。在这里看看我们关于 Java 流的[进一步教程。](/web/20220926180850/https://www.baeldung.com/java-streams)

GitHub 上的[提供了代码示例。](https://web.archive.org/web/20220926180850/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)