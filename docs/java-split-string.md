# 在 Java 中拆分字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-split-string>

## 1。简介

拆分`Strings`是一个非常频繁的操作；这个快速教程集中在一些 API 上，我们可以用 Java 简单地做到这一点。

## 2。`String.split()`

让我们从核心库开始—— `String`类本身提供了一个`split()`方法——这对于大多数情况来说非常方便和足够了。它只是根据分隔符分割给定的`String`，返回一个数组`Strings`。

让我们看一些例子。我们将从逗号分隔开始:

```
String[] splitted = "peter,james,thomas".split(",");
```

让我们用空格分开:

```
String[] splitted = "car jeep scooter".split(" ");
```

让我们也用一个点来分割:

```
String[] splitted = "192.168.1.178".split("\\.")
```

现在让我们用多个字符来分割——逗号、空格和连字符，通过正则表达式:

```
String[] splitted = "b a, e, l.d u, n g".split("\\s+|,\\s*|\\.\\s*"));
```

## 3。`StringUtils.split()`

Apache 的公共 lang 包提供了一个`StringUtils`类——它包含一个空安全的`split()`方法，该方法使用空白作为默认分隔符进行拆分:

```
String[] splitted = StringUtils.split("car jeep scooter");
```

此外，它忽略了额外的空格:

```
String[] splitted = StringUtils.split("car   jeep  scooter");
```

## 4。`Splitter.split()`

最后，番石榴还有一个很好的流畅的 API:

```
List<String> resultList = Splitter.on(',')
  .trimResults()
  .omitEmptyStrings()
  .splitToList("car,jeep,, scooter"); 
```

## 5。分割和修剪

有时给定的`String`在分隔符周围包含一些前导、尾随或额外的空格。让我们看看如何一次性处理输入拆分和结果修整。

假设我们有这样一个输入:

```
String input = " car , jeep, scooter ";
```

要删除分隔符前后多余的空格，我们可以使用 regex:

```
String[] splitted = input.trim().split("\\s*,\\s*");
```

这里，`trim()`方法删除输入字符串中的前导和尾随空格，regex 本身处理分隔符周围的多余空格。

我们可以通过使用 Java 8 `Stream`的特性达到同样的效果:

```
String[] splitted = Arrays.stream(input.split(","))
  .map(String::trim)
  .toArray(String[]::new);
```

## 6。结论

`String.split()`一般就够了。然而，对于更复杂的情况，我们可以利用 Apache 的基于 commons-lang 的`StringUtils`类，或者干净灵活的 Guava APIs。

和往常一样，这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206065811/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)