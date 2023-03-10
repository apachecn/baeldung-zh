# 如何迭代带有索引的流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-indices>

## 1。概述

Java 8 `Streams`不是集合，元素不能使用它们的索引来访问，但是仍然有一些技巧可以做到这一点。

在这篇短文中，我们将看看如何使用`IntStream,` `StreamUtils, EntryStream,` 和`Vavr`的`Stream`来迭代`Stream`。

## 2。使用普通 Java

我们可以使用一个`Integer`范围在一个`Stream`中导航，并且受益于这样一个事实，即原始元素在一个数组或一个可通过索引访问的集合中。

让我们实现一个使用索引进行迭代的方法，并演示这种方法。

简单地说，我们想要得到一个数组`Strings`，并且只选择偶数索引元素:

```java
public List<String> getEvenIndexedStrings(String[] names) {
    List<String> evenIndexedNames = IntStream
      .range(0, names.length)
      .filter(i -> i % 2 == 0)
      .mapToObj(i -> names[i])
      .collect(Collectors.toList());

    return evenIndexedNames;
}
```

现在让我们测试一下实现:

```java
@Test
public void whenCalled_thenReturnListOfEvenIndexedStrings() {
    String[] names 
      = {"Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim"};
    List<String> expectedResult 
      = Arrays.asList("Afrim", "Besim", "Durim");
    List<String> actualResult 
      = StreamIndices.getEvenIndexedStrings(names);

    assertEquals(expectedResult, actualResult);
}
```

## 3。使用`StreamUtils`

另一种迭代索引的方法是使用来自`proton-pack`库的`StreamUtils`的`zipWithIndex()`方法(最新版本可以在找到[)。](https://web.archive.org/web/20221108150357/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.codepoetics%22%20AND%20a%3A%22protonpack%22)

首先，您需要将它添加到您的 `pom.xml`:

```java
<dependency>
    <groupId>com.codepoetics</groupId>
    <artifactId>protonpack</artifactId>
    <version>1.13</version>
</dependency> 
```

现在，让我们看看代码:

```java
public List<Indexed<String>> getEvenIndexedStrings(List<String> names) {
    List<Indexed<String>> list = StreamUtils
      .zipWithIndex(names.stream())
      .filter(i -> i.getIndex() % 2 == 0)
      .collect(Collectors.toList());

    return list;
}
```

以下测试了此方法并成功通过:

```java
@Test
public void whenCalled_thenReturnListOfEvenIndexedStrings() {
    List<String> names = Arrays.asList(
      "Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim");
    List<Indexed<String>> expectedResult = Arrays.asList(
      Indexed.index(0, "Afrim"), 
      Indexed.index(2, "Besim"), 
      Indexed.index(4, "Durim"));
    List<Indexed<String>> actualResult 
      = StreamIndices.getEvenIndexedStrings(names);

    assertEquals(expectedResult, actualResult);
}
```

## 4。使用`StreamEx`

我们还可以使用来自 *StreamEx* 库的`EntryStream`类的`filterKeyValue()`来迭代索引(最新版本可以在[这里](https://web.archive.org/web/20221108150357/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22one.util%22%20AND%20a%3A%22streamex%22)找到)。首先，我们需要将它添加到我们的`pom.xml:`

```java
<dependency>
    <groupId>one.util</groupId>
    <artifactId>streamex</artifactId>
    <version>0.6.5</version>
</dependency>
```

让我们用前面的例子来看看这个方法的一个简单应用:

```java
public List<String> getEvenIndexedStringsVersionTwo(List<String> names) {
    return EntryStream.of(names)
      .filterKeyValue((index, name) -> index % 2 == 0)
      .values()
      .toList();
}
```

我们将使用类似的测试对此进行测试:

```java
@Test
public void whenCalled_thenReturnListOfEvenIndexedStringsVersionTwo() {
    String[] names 
      = {"Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim"};
    List<String> expectedResult 
      = Arrays.asList("Afrim", "Besim", "Durim");
    List<String> actualResult 
      = StreamIndices.getEvenIndexedStrings(names);

   assertEquals(expectedResult, actualResult);
}
```

## 5。迭代使用**`Vavre``Stream`**

另一种可行的迭代方式是使用`Vavr`(以前称为`Javaslang`)的`Stream`实现的`zipWithIndex()`方法:

```java
public List<String> getOddIndexedStringsVersionTwo(String[] names) {
    return Stream
      .of(names)
      .zipWithIndex()
      .filter(tuple -> tuple._2 % 2 == 1)
      .map(tuple -> tuple._1)
      .toJavaList();
}
```

我们可以用下面的方法测试这个例子:

```java
@Test
public void whenCalled_thenReturnListOfOddStringsVersionTwo() {
    String[] names 
      = {"Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim"};
    List<String> expectedResult 
      = Arrays.asList("Bashkim", "Lulzim", "Shpetim");
    List<String> actualResult 
      = StreamIndices.getOddIndexedStringsVersionTwo(names);

    assertEquals(expectedResult, actualResult);
}
```

如果你想了解更多关于 Vavr 的内容，请查看这篇文章。

## 6。结论

在这个快速教程中，我们看到了如何使用索引遍历流的四种方法。流已经得到了很多关注，能够用索引遍历它们会很有帮助。

Java 8 流中包含了很多特性，其中一些已经在 [Baeldung](/web/20221108150357/https://www.baeldung.com/java-8-streams-introduction) 中介绍过了。

这里解释的所有例子的代码，以及更多可以在 GitHub 上找到的[。](https://web.archive.org/web/20221108150357/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)