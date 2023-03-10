# 可在 Java 中流式传输

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterable-to-stream>

## 1。概述

在这个简短的教程中，让我们将一个 Java `Iterable`对象转换成一个`Stream`，并对其执行一些标准操作。

## 2。将`Iterable` 转换为`Stream`

`Iterable`接口的设计考虑到了通用性，它本身不提供任何`stream()` 方法。

简单地说，您可以将它传递给`StreamSupport.stream()`方法，并从给定的`Iterable` 实例中获得一个`Stream` 。

让我们考虑一下我们的`Iterable` 实例:

```java
Iterable<String> iterable 
  = Arrays.asList("Testing", "Iterable", "conversion", "to", "Stream");
```

下面是我们如何将这个`Iterable` 实例转换成一个`Stream:`

```java
StreamSupport.stream(iterable.spliterator(), false);
```

注意，`StreamSupport.stream()`中的第二个参数决定了结果`Stream`应该是并行的还是顺序的。你应该把它设定为真实的，因为平行的`Stream`。

现在让我们测试我们的实现:

```java
@Test
public void givenIterable_whenConvertedToStream_thenNotNull() {
    Iterable<String> iterable 
      = Arrays.asList("Testing", "Iterable", "conversion", "to", "Stream");

    Assert.assertNotNull(StreamSupport.stream(iterable.spliterator(), false));
}
```

另外，快速补充一下——流是不可重用的，而`Iterable`是；它还提供了一个`spliterator()` 方法，该方法通过给定的`Iterable`描述的元素返回一个`java.lang.Spliterator instance`。

## 3。执行`Stream`操作

让我们执行一个简单的流操作:

```java
@Test
public void whenConvertedToList_thenCorrect() {
    Iterable<String> iterable 
      = Arrays.asList("Testing", "Iterable", "conversion", "to", "Stream");

    List<String> result = StreamSupport.stream(iterable.spliterator(), false)
      .map(String::toUpperCase)
      .collect(Collectors.toList());

    assertThat(
      result, contains("TESTING", "ITERABLE", "CONVERSION", "TO", "STREAM"));
}
```

## 4。结论

这个简单的教程展示了如何将一个`Iterable` 实例转换成一个`Stream` 实例，并对其执行标准操作，就像您对任何其他`Collection` 实例所做的一样。

所有代码片段的实现可以在 [Github 项目](https://web.archive.org/web/20221205202900/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)中找到。