# 操作流中的项目并从流中移除项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-use-remove-item-stream>

## 1.概观

在这个快速教程中，我们将学习在 Java 8 流中操作一个项目的各种方法，然后在操作完成后删除它。

## 2.设置

让我们首先定义我们的`Item`对象。这是一个只有一个`int`字段的简单对象。

它有一个根据内部值确定对象是否符合操作条件的方法:

```java
class Item {
    private int value;

    // constructors

    public boolean isQualified() {
        return value % 2 == 0;
    }

    public void operate() {
        System.out.println("Even Number");
    }
}
```

现在我们可以为我们的`Stream`创建一个源，它可以是`Items:`的集合

```java
List<Item> itemList = new ArrayList<>();
for (int i = 0; i < 10; i++) {
    itemList.add(new Item(i));
}
```

## 3.过滤项目

**在很多情况下，如果我们想对项目的子集做一些事情，我们可以使用`Stream#filter`方法，我们不需要先删除任何东西:**

```java
itemList.stream()
  .filter(item -> item.isQualified())
  ...
```

## 4.操作和移除项目

### 4.1.`Collection.removeIf`

我们可以使用`Streams`对`Items`的集合进行迭代和操作。

使用`Streams`，我们可以应用称为`Predicates`的 lambda 函数。为了阅读更多关于`Streams`和`Predicates`的内容，我们在这里还有另一篇文章。

我们可以创建一个`Predicate`来确定一个`Item`是否有资格接受手术:

```java
Predicate<Item> isQualified = item -> item.isQualified();
```

我们的`Predicate`将过滤我们想要操作的`Items`:

```java
itemList.stream()
  .filter(isQualified)
  .forEach(item -> item.operate());
```

一旦我们完成了对流中项目的操作，我们就可以使用之前用于过滤的相同的`Predicate`来删除它们:

```java
itemList.removeIf(isQualified);
```

**在内部，`removeIf`使用一个`Iterator`遍历列表，并使用谓词匹配元素。**我们现在可以从列表中删除任何匹配的元素。

### 4.2.`Collection.removeAll`

我们还可以使用另一个列表来保存已经被操作过的元素，然后将它们从原始列表中删除:

```java
List<Item> operatedList = new ArrayList<>();
itemList.stream()
  .filter(item -> item.isQualified())
  .forEach(item -> {
    item.operate();
    operatedList.add(item);
});
itemList.removeAll(operatedList);
```

与使用迭代器的`removeIf`不同， **`removeAll`使用简单的`for-loop`来删除列表中所有匹配的元素。**

## 5.结论

在本文中，我们研究了一种在 Java 8 流中操作一个项目然后删除它的方法。我们还看到了一种方法，即使用一个附加列表并删除所有匹配的元素。

本教程的源代码和相关的测试用例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221129012400/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)