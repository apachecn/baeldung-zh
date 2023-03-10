# Java 9 可选 API 补充

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-optional>

## 1。概述

在本文中，我们将看看 Java 9 对`[Optional](https://web.archive.org/web/20221126224249/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)` API 的补充。

除了模块化，Java 9 还为`Optional`类增加了三个非常有用的方法。

## 2。`or()` 法

有时，当我们的`Optional`为空时，我们想要执行一些其他的动作，这些动作也返回一个 O `ptional.`

在 Java 9 之前，`Optional` 类只有`orElse()` 和`orElseGet()` 方法，但都需要返回未包装的值。

Java 9 引入了`or()` 方法，如果我们的`Optional`为空，该方法会延迟返回另一个`Optional`。如果我们的第一个`Optional` 有一个已定义的值，传递给`or()` 方法的 lambda 将不会被调用，值也不会被计算和返回:

```java
@Test
public void givenOptional_whenPresent_thenShouldTakeAValueFromIt() {
    //given
    String expected = "properValue";
    Optional<String> value = Optional.of(expected);
    Optional<String> defaultValue = Optional.of("default");

    //when
    Optional<String> result = value.or(() -> defaultValue);

    //then
    assertThat(result.get()).isEqualTo(expected);
}
```

在 `Optional`为空`ng`的情况下，返回的`result` 将与`defaultValue:`相同

```java
@Test
public void givenOptional_whenEmpty_thenShouldTakeAValueFromOr() {
    // given
    String defaultString = "default";
    Optional<String> value = Optional.empty();
    Optional<String> defaultValue = Optional.of(defaultString);

    // when
    Optional<String> result = value.or(() -> defaultValue);

    // then
    assertThat(result.get()).isEqualTo(defaultString);
}
```

## 3。`ifPresentOrElse()` 法

当我们有一个`Optional`实例时，我们经常想要对它的底层值执行一个特定的动作。另一方面，如果`Optional` 是`empty`，我们想要记录它或者通过增加一些度量来跟踪这个事实。

`ifPresentOrElse()` 方法正是为这样的场景而创建的。我们可以传递一个在定义了`Optional`的情况下将被调用的`Consumer` ，以及在`Optional`为空的情况下将被执行的`Runnable` 。

假设我们有一个已定义的`Optional` ，如果该值存在，我们希望递增一个特定的计数器:

```java
@Test
public void givenOptional_whenPresent_thenShouldExecuteProperCallback() {
    // given
    Optional<String> value = Optional.of("properValue");
    AtomicInteger successCounter = new AtomicInteger(0);
    AtomicInteger onEmptyOptionalCounter = new AtomicInteger(0);

    // when
    value.ifPresentOrElse(
      v -> successCounter.incrementAndGet(), 
      onEmptyOptionalCounter::incrementAndGet);

    // then
    assertThat(successCounter.get()).isEqualTo(1);
    assertThat(onEmptyOptionalCounter.get()).isEqualTo(0);
}
```

注意，作为第二个参数传递的回调没有被执行。

在空的`Optional,`的情况下，第二个回调被执行:

```java
@Test
public void givenOptional_whenNotPresent_thenShouldExecuteProperCallback() {
    // given
    Optional<String> value = Optional.empty();
    AtomicInteger successCounter = new AtomicInteger(0);
    AtomicInteger onEmptyOptionalCounter = new AtomicInteger(0);

    // when
    value.ifPresentOrElse(
      v -> successCounter.incrementAndGet(), 
      onEmptyOptionalCounter::incrementAndGet);

    // then
    assertThat(successCounter.get()).isEqualTo(0);
    assertThat(onEmptyOptionalCounter.get()).isEqualTo(1);
}
```

## 4。`stream()` 法

Java 9 中添加到`Optional`类的最后一个方法是`stream()` 方法。

Java 有一个非常流畅优雅的 API，可以在集合上操作，并利用了许多函数式编程概念。最新的 Java 版本在`Optional` 类上引入了`stream()` 方法，使得**允许我们将`Optional`实例视为`Stream.`**

假设我们有一个已定义的`Optional` ,我们正在调用它的`stream()` 方法。这将创建一个元素的`Stream` ,在这个元素上我们可以使用`Stream` API `:`中所有可用的方法

```java
@Test
public void givenOptionalOfSome_whenToStream_thenShouldTreatItAsOneElementStream() {
    // given
    Optional<String> value = Optional.of("a");

    // when
    List<String> collect = value.stream().map(String::toUpperCase).collect(Collectors.toList());

    // then
    assertThat(collect).hasSameElementsAs(List.of("A"));
}
```

另一方面，如果`Optional`不存在，调用它的`stream()` 方法将创建一个空的`Stream:`

```java
@Test
public void givenOptionalOfNone_whenToStream_thenShouldTreatItAsZeroElementStream() {
    // given
    Optional<String> value = Optional.empty();

    // when
    List<String> collect = value.stream()
      .map(String::toUpperCase)
      .collect(Collectors.toList());

    // then
    assertThat(collect).isEmpty();
}
```

我们现在可以快速过滤`Optionals.` 中的 [`Streams`](/web/20221126224249/https://www.baeldung.com/java-filter-stream-of-optional)

在空的`Stream`上操作不会有任何效果，但是由于有了`stream()`方法，我们现在可以将`Optional` API 与`Stream` API 链接起来。这允许我们创建更优雅和流畅的代码。

## 5。结论

在这篇简短的文章中，我们看了 Java 9 `Optional` API 的新增内容。

我们看到了如何使用`or()` 方法在源`Optional`为空的情况下返回一个可选的。如果值存在，我们使用`ifPresentOrElse()` 来执行`Consumer` ，否则，运行另一个回调。

最后，我们看到了如何使用 `stream()`方法将`Optional`与`Stream` API 链接起来。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221126224249/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。