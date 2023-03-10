# 将字符串转换为字符流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-stream>

## 1。概述

Java 8 引入了`Stream` API，用类似函数的操作来处理序列。如果你想了解更多，请看这篇文章。

在这篇简短的文章中，我们将看到如何将单个字符的`String`转换为`Stream`。

## 2。转换使用`chars()`

`String` API 有一个新方法——`chars()`——用它我们可以从一个`String`对象中获得一个`S`T3 的实例。这个简单的 API 从输入`String`返回一个`Int` `Stream`的实例。

简单地说，`IntStream`包含来自`String`对象的字符的整数表示:

```java
String testString = "String";
IntStream intStream = testString.chars();
```

可以使用字符的整数表示，而不需要将它们转换成对应的`Character`。这可以带来一些小的性能提升，因为不需要将每个整数打包到一个`Character`对象中。

然而，如果我们要显示用于阅读的字符，我们需要将整数转换成人类友好的`Character`形式:

```java
Stream<Character> characterStream = testString.chars()
  .mapToObj(c -> (char) c);
```

## 3。转换使用`codePoints()`

或者，我们可以使用`codePoints()`方法从一个`String.`中获得一个`IntStream`的实例。使用这个 API 的好处是可以有效地处理 Unicode 补充字符。

补充字符由 Unicode 代理项对表示，并将合并成一个代码点。这样我们可以正确地处理(和显示)任何 Unicode 符号:

```java
IntStream intStream1 = testString.codePoints();
```

我们需要将返回的`IntStream`映射到`Stream<Character>`向用户显示:

```java
Stream<Character> characterStream2 
  = testString.codePoints().mapToObj(c -> (char) c); 
```

## 4。转换为单字符`Strings` 的`Stream`

到目前为止，我们已经能够得到一个`Stream`字符；如果我们想要一个单字符`String` s 的`Stream`呢？

正如本文前面所述，我们将使用`codePoints()` 或`chars()`方法来获取`IntStream`的一个实例，我们现在可以将它映射到`Stream<String>`。

映射过程包括首先将整数值转换成它们各自的等价字符。

然后我们可以使用`String.valueOf()`或`Character.toString()`将字符转换成一个`String`对象:

```java
Stream<String> stringStream = testString.codePoints()
  .mapToObj(c -> String.valueOf((char) c));
```

## 5。结论

在这个快速教程中，我们学习通过调用`codePoints()`或`chars()`方法从`String`对象获得一个`Character`流。

这允许我们充分利用`Stream`API——方便有效地操纵角色。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221206051324/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)