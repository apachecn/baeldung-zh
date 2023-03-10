# 在 Java 中过滤可选流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-filter-stream-of-optional>

## 1。简介

在本文中，我们将讨论如何从`Optionals`的`Stream`中过滤出非空值。

我们将看到三种不同的方法——两种使用 Java 8，一种使用 Java 9 中的新支持。

我们将在所有示例中使用相同的列表:

```java
List<Optional<String>> listOfOptionals = Arrays.asList(
  Optional.empty(), Optional.of("foo"), Optional.empty(), Optional.of("bar"));
```

## 2。使用`filter()`

Java 8 中的一个选项是用`Optional::isPresent`过滤掉值，然后用`Optional::get`函数执行映射来提取值:

```java
List<String> filteredList = listOfOptionals.stream()
  .filter(Optional::isPresent)
  .map(Optional::get)
  .collect(Collectors.toList());
```

## 3。使用`flatMap()`

另一种选择是使用带有 lambda 表达式的`flatMap`，将空的`Optional`转换成空的`Stream`实例，将非空的`Optional`转换成只包含一个元素的`Stream`实例:

```java
List<String> filteredList = listOfOptionals.stream()
  .flatMap(o -> o.isPresent() ? Stream.of(o.get()) : Stream.empty())
  .collect(Collectors.toList());
```

或者，您可以应用相同的方法，使用不同的方式将`Optional`转换为`Stream`:

```java
List<String> filteredList = listOfOptionals.stream()
  .flatMap(o -> o.map(Stream::of).orElseGet(Stream::empty))
  .collect(Collectors.toList());
```

## 4.Java 9 的可选::stream

随着 Java 9 的到来，所有这些都会变得非常简单，Java 9 为 `Optional`增加了一个`stream()`方法。

这种方法类似于第 3 节中展示的方法，但这次我们使用预定义的方法将`Optional`实例转换成`Stream`实例:

无论`Optional`值是否存在，它都将返回一个或零个元素的流:

```java
List<String> filteredList = listOfOptionals.stream()
  .flatMap(Optional::stream)
  .collect(Collectors.toList());
```

## 5。结论

这样，我们很快就看到了从`Optionals`的`Stream`中过滤当前值的三种方法。

代码示例的完整实现可以在 [Github 项目](https://web.archive.org/web/20220626202740/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional)中找到。