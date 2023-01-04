# 如何在 Java 中反转数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-invert-array>

## 1。 **概述**

在这篇简短的文章中，我们将展示如何在 Java 中反转一个数组。

我们将看到使用纯基于 Java 8 的解决方案来实现这一点的几种不同方法——一些方法改变了现有的数组，一些方法创建了新的数组。

接下来，我们将看两个使用外部库的解决方案——一个使用`Apache Commons Lang`另一个使用`Google Guava`。

## 2。定义问题

基本思想是颠倒数组中元素的顺序。因此，如果给定数组:

```
fruits = {"apples", "tomatoes", "bananas", "guavas", "pineapples"}
```

我们希望获得:

```
invertedFruits = {"pineapples", "guavas", "bananas", "tomatoes",  "apples"}
```

让我们看看我们能做的一些方法。

## 3。使用传统的`for`循环

我们可能想到的第一种反转数组的方法是使用一个`for`循环:

```
void invertUsingFor(Object[] array) {
    for (int i = 0; i < array.length / 2; i++) {
        Object temp = array[i];
        array[i] = array[array.length - 1 - i];
        array[array.length - 1 - i] = temp;
    }
}
```

正如我们所看到的，代码遍历了数组的一半，改变了对称位置的元素。

我们使用一个临时变量，这样我们就不会在迭代过程中丢失数组当前位置的值。

## 4。使用 Java 8 `Stream` API

我们也可以通过使用 Stream API 来反转数组:

```
Object[] invertUsingStreams(Object[] array) {
    return IntStream.rangeClosed(1, array.length)
      .mapToObj(i -> array[array.length - i])
      .toArray();
}
```

这里我们使用方法`IntStream.range` 来生成一个连续的数字流。然后我们将这个序列按照降序映射到数组索引中。

## 5。使用`Collections.reverse()`

让我们看看如何使用`Collections.reverse()`方法反转一个数组:

```
public void invertUsingCollectionsReverse(Object[] array) {
    List<Object> list = Arrays.asList(array);
    Collections.reverse(list);
}
```

与前面的例子相比，这是一种可读性更强的方法。

## 6。使用 Apache Commons Lang

反转数组的另一个选择是使用`Apache Commons Lang`库。要使用它，我们必须首先将库作为一个依赖项包括进来:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

最新版本的`Commons Lang`可以在 [Maven Central](https://web.archive.org/web/20220629004027/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 找到。

让我们使用`ArrayUtils`类来反转数组:

```
public void invertUsingCommonsLang(Object[] array) {
    ArrayUtils.reverse(array);
}
```

正如我们所看到的，这个解决方案非常简单。

## 7。使用谷歌番石榴

另一个选择是使用`Google Guava`库。正如我们对`Commons Lang`所做的一样，我们将把这个库作为一个依赖项包括进来:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220629004027/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 找到。

然后，我们可以使用`Guava's` `Lists`类中的`reverse`方法来反转数组:

```
public Object[] invertUsingGuava(Object[] array) {
    List<Object> list = Arrays.asList(array);
    List<Object> reversed = Lists.reverse(list);
    return reversed.toArray();
}
```

## 8。结论

在本文中，我们研究了在 Java 中反转数组的几种不同方法。我们展示了几个仅使用核心 Java 的解决方案和另外两个使用第三方库的解决方案— `Commons Lang`和`Guava`。

这里显示的所有代码示例都可以在 GitHub 上找到[——这是一个 Maven 项目，因此它应该很容易导入和运行。](https://web.archive.org/web/20220629004027/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-sorting)