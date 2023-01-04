# map()和 flatMap()的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-difference-map-and-flatmap>

## 1。概述

`map()`和`flatMap()`API 源于函数式语言。在 Java 8 中，我们可以在`Optional`、 `Stream`和`CompletableFuture`中找到它们(虽然名字略有不同)。

代表一系列的对象，而选项是代表一个可以存在或不存在的值的类。在其他聚合操作中，我们有`map()`和`flatMap()`方法。

尽管两者具有相同的返回类型，但它们却有很大的不同。让我们通过分析一些流和选项的例子来解释这些差异。

## 延伸阅读:

## [在 Java 中迭代地图](/web/20220811145441/https://www.baeldung.com/java-iterate-map)

Learn different ways of iterating through the entries of a Map in Java.[Read more](/web/20220811145441/https://www.baeldung.com/java-iterate-map) →

## [用 Jackson 映射序列化和反序列化](/web/20220811145441/https://www.baeldung.com/jackson-map)

A quick and practical guide to serializing and deserializing Java Maps using Jackson.[Read more](/web/20220811145441/https://www.baeldung.com/jackson-map) →

## [如何在 Java 中存储一个 Map 中的重复键？](/web/20220811145441/https://www.baeldung.com/java-map-duplicate-keys)

A quick and practical guide to handling duplicate keys by using multimaps in Java.[Read more](/web/20220811145441/https://www.baeldung.com/java-map-duplicate-keys) →

## 2。选项中的地图和平面图

`map()`方法与`Optional`配合得很好——如果函数返回我们需要的确切类型:

```
Optional<String> s = Optional.of("test");
assertEquals(Optional.of("TEST"), s.map(String::toUpperCase));
```

然而，在更复杂的情况下，我们可能会得到一个返回`Optional`的函数。在这种情况下，使用`map()`会导致嵌套结构，因为`map()`实现在内部做了额外的包装。

让我们看另一个例子来更好地理解这种情况:

```
assertEquals(Optional.of(Optional.of("STRING")), 
  Optional
  .of("string")
  .map(s -> Optional.of("STRING")));
```

正如我们所见，我们以嵌套结构`Optional<Optional<String>>`结束。虽然它可以工作，但使用起来相当麻烦，并且不提供任何额外的空安全，所以最好保持扁平的结构。

这正是`flatMap()` 帮助我们做的:

```
assertEquals(Optional.of("STRING"), Optional
  .of("string")
  .flatMap(s -> Optional.of("STRING")));
```

## 3。流中的地图和平面地图

对于`Optional`，两种方法的工作方式相似。

`map()` 方法将底层序列包装在一个`Stream`实例中，而`flatMap()` 方法允许避免嵌套的`Stream<Stream<R>>`结构。

这里，`map()`产生一个`Stream`，它由将`toUpperCase()`方法应用于输入`Stream`的元素的结果组成:

```
List<String> myList = Stream.of("a", "b")
  .map(String::toUpperCase)
  .collect(Collectors.toList());
assertEquals(asList("A", "B"), myList);
```

在这种简单的情况下，效果相当好。但是如果我们有一些更复杂的东西，比如作为输入的列表列表呢？

让我们看看它是如何工作的:

```
List<List<String>> list = Arrays.asList(
  Arrays.asList("a"),
  Arrays.asList("b"));
System.out.println(list);
```

这个代码片段打印了一个列表列表`[[a], [b]]`。

现在让我们用一个`flatMap()`:

```
System.out.println(list
  .stream()
  .flatMap(Collection::stream)
  .collect(Collectors.toList()));
```

**这样一个片段的结果会被展平为`[a, b]`。**

`T`他的`flatMap()`方法首先将 `Streams`的输入`Stream` 展平为`Strings`的一个`Stream`(关于展平的更多内容，参见本文[文章](/web/20220811145441/https://www.baeldung.com/java-flatten-nested-collections))。此后，其工作方式类似于`map()`方法。

## 4。结论

Java 8 让我们有机会使用最初在函数式语言中使用的`map()`和`flatMap()`方法。

我们可以在`Streams`和选项上调用它们。这些方法帮助我们通过应用提供的映射函数来获得映射的对象。

和往常一样，本文中的例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220811145441/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)