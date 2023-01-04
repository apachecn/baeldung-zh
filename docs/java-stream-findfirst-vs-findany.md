# Java 8 流 findFirst()与 findAny()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-findfirst-vs-findany>

## 1。概述

Java 8 `Stream` API 引入了两个经常被误解的方法:`findAny()`和`findFirst()`。

在这个快速教程中，我们将看看这两种方法之间的区别以及何时使用它们。

## 延伸阅读:

## [过滤 Java 中的可选流](/web/20221121201205/https://www.baeldung.com/java-filter-stream-of-optional)

A quick and practical guide to filtering Streams of Optionals in Java 8 and Java 9[Read more](/web/20221121201205/https://www.baeldung.com/java-filter-stream-of-optional) →

## [Java 8 中的原始类型流](/web/20221121201205/https://www.baeldung.com/java-8-primitive-streams)

A quick and practical guide to using Java 8 Streams with primitive types.[Read more](/web/20221121201205/https://www.baeldung.com/java-8-primitive-streams) →

## [可在 Java 中流式传输](/web/20221121201205/https://www.baeldung.com/java-iterable-to-stream)

The article explains how to convert an Iterable to Stream and why the Iterable interface doesn't support it directly.[Read more](/web/20221121201205/https://www.baeldung.com/java-iterable-to-stream) →

## 2。使用` Stream.findAny()`

顾名思义，`findAny()`方法允许我们从一个`Stream`中找到任何元素。当我们寻找一个元素而不注意遇到顺序时，我们使用它:

该方法返回一个`Optional`实例，如果`Stream`为空，则该实例为空:

```java
@Test
public void createStream_whenFindAnyResultIsPresent_thenCorrect() {
    List<String> list = Arrays.asList("A","B","C","D");

    Optional<String> result = list.stream().findAny();

    assertTrue(result.isPresent());
    assertThat(result.get(), anyOf(is("A"), is("B"), is("C"), is("D")));
}
```

在非并行操作中，**最有可能返回`Stream`中的第一个元素，但对此没有保证。**

为了在处理并行操作时获得最佳性能，无法可靠地确定结果:

```java
@Test
public void createParallelStream_whenFindAnyResultIsPresent_thenCorrect()() {
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    Optional<Integer> result = list
      .stream().parallel()
      .filter(num -> num < 4).findAny();

    assertTrue(result.isPresent());
    assertThat(result.get(), anyOf(is(1), is(2), is(3)));
}
```

## 3。使用` Stream.findFirst()`

`findFirst()`方法查找`Stream`中的第一个元素。所以，当我们特别想要序列中的第一个元素时，我们使用这个方法。

当没有遇到顺序时，它返回来自`Stream`的任何元素。根据 `java.util.streams`包文档，“流可能有也可能没有定义的相遇顺序。这取决于来源和中间操作。”

返回类型也是一个`Optional`实例，如果`Stream`也是空的，那么它也是空的:

```java
@Test
public void createStream_whenFindFirstResultIsPresent_thenCorrect() {

    List<String> list = Arrays.asList("A", "B", "C", "D");

    Optional<String> result = list.stream().findFirst();

    assertTrue(result.isPresent());
    assertThat(result.get(), is("A"));
}
```

在并行场景中，`findFirst` 方法的行为不会改变。如果遭遇顺序存在，它将总是表现出确定性。

## 4。结论

在本文中，我们查看了 Java 8 Streams API 的`findAny()` 和 `findFirst()`方法。

`findAny()`方法从`Stream`中返回任何元素，而`findFirst()`方法返回`Stream`中的第一个元素。

本文的完整源代码和所有代码片段都在 GitHub 上的[。](https://web.archive.org/web/20221121201205/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)