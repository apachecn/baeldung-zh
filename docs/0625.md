# 同时遍历两个数组列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/iterate-through-two-arraylists-simultaneously>

## 1.概观

有时，我们需要将多个列表中的数据连接在一起，将第一个列表中的第一项与第二个列表中的对应项连接起来，依此类推。

在本教程中，我们将学习几种同时遍历两个 [`ArrayList`](/web/20221116222735/https://www.baeldung.com/java-arraylist) 集合的方法。我们将研究循环、迭代器、流和第三方工具来解决这个问题。

## 2.问题陈述和用例

让我们举一个例子，我们有两个列表，一个包含国家名称，另一个包含国家的电话代码。假设在第二个列表中，任何给定索引处的电话代码对应于第一个列表中相同索引处的国家名称。

我们希望将第二个列表中正确的国家代码与第一个列表中相应的国家名称相关联。

**在 Java** 中，没有现成的解决方案适用于这种用例。然而，有几种方法可以实现这一点。

让我们首先创建两个用于处理的`List`对象:

```
List<String> countryName = List.of("USA", "UK", "Germany", "India");
List<String> countryCode = List.of("+1", "+44", "+49", "+91");
```

这里，我们有两个相同大小的列表，因为每个国家都应该有一个代码。如果列表大小不同，我们的解决方案可能不起作用。

我们希望处理这两个列表，并获得每个国家的正确代码。我们将测试每个解决方案的预期输出:

```
assertThat(processedList)
  .containsExactly("USA: +1", "UK: +44", "Germany: +49", "India: +91");
```

## 3.使用`for`循环进行迭代

让我们从使用一个`for`循环迭代两个列表的最简单的方法开始:

```
for (int i = 0; i < countryName.size(); i++) {
    String processedData = String.format("%s: %s", countryName.get(i), countryCode.get(i));
    processedList.add(processedData);
}
```

在这里，我们在两个具有相同索引的列表上使用了`get()`——来配对我们的项目。在循环结束时，`processedList`将包含正确的结果。

## 4.使用`Collection` `Iterator`进行迭代

我们也可以使用 [`Collection`](/web/20221116222735/https://www.baeldung.com/java-collections) 接口的`iterator()`方法得到一个`[Iterator](/web/20221116222735/https://www.baeldung.com/java-iterator)`实例。我们将首先获得两个列表的`Iterator`实例:

```
Iterator<String> nameIterator = countryName.iterator();
Iterator<String> codeIterator = countryCode.iterator();
```

我们将使用一个`while`循环来管理两个迭代器:

```
while (nameIterator.hasNext() && codeIterator.hasNext()) {
    String processedData = String.format("%s: %s", nameIterator.next(), codeIterator.next());
    processedList.add(processedData);
}
```

`while`循环调用`hasNext()`来确保两个迭代器仍然有值，在循环中，我们使用`next()`挑选下一对值。

## 5.使用`StreamUtils`方法处理`zip()`

我们可以说，我们的目标是将每个列表中的数据相互关联，或者一起处理这一对数据。这也被称为拉链一对集合。各种库提供了这个过程的实现，我们可以开箱即用。

**让我们使用 Spring 的`[StreamUtils](/web/20221116222735/https://www.baeldung.com/spring-stream-utils)`库的`zip()`方法，并提供一个`[Lambda](/web/20221116222735/https://www.baeldung.com/java-8-lambda-expressions-tips)`来创建我们的组合值。**

### 5.1.属国

首先，我们应该在`pom.xml`文件中添加 [Spring 数据依赖关系](https://web.archive.org/web/20221116222735/https://search.maven.org/artifact/org.springframework.data/spring-data-commons):

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
    <version>2.7.5</version>
</dependency>
```

### 5.2.履行

我们将把列表流和一个 lambda 函数传递给`zip()`方法。lambda 将拥有处理逻辑，我们将使用`collect()`方法在一个列表中获取所有处理过的数据。

```
List<String> processedList = StreamUtils.zip(
  countryName.stream(), 
  countryCode.stream(),
  (name, code) -> String.format("%s: %s", name, code))
  .collect(Collectors.toList());
```

`zip()`的输出是我们收集的一个`Stream`。在输入列表的两个`Stream`之后，我们提供的`BiFunction`作为第三个参数，用于创建`Stream`的元素，然后我们可以在最后将这些元素收集到一个包含正确配对的列表中。

我们应该注意，这个方法具有 Java 流的所有优点，包括过滤和映射输入数据、过滤输出数据，以及尽可能少地占用内存。

## 6.结论

在本教程中，我们学习了同时迭代两个`ArrayList`的不同方法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221116222735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)