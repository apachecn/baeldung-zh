# 用多个分隔符拆分 Java 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-split-multiple-delimiters>

## 1.介绍

我们都知道，拆分字符串是一项非常常见的任务。然而，我们经常只使用一个分隔符来分割。

在本教程中，我们将详细讨论通过多个分隔符分割字符串的不同选项**。**

## 2.用多个分隔符拆分 Java 字符串

为了展示下面的每个解决方案如何执行拆分，我们将使用相同的示例字符串:

```java
String example = "Mary;Thomas:Jane-Kate";
String[] expectedArray = new String[]{"Mary", "Thomas", "Jane", "Kate"};
```

### 2.1.正则表达式解决方案

程序员经常使用不同的[正则表达式](/web/20220524034154/https://www.baeldung.com/regular-expressions-java)来定义字符串的搜索模式。在拆分字符串时，它们也是一种非常流行的解决方案。那么，让我们看看如何在 Java 中使用正则表达式通过多个分隔符来拆分字符串。

首先，我们不需要添加新的依赖项，因为正则表达式在`java.util.regex` 包`. `中是可用的，我们只需要定义一个我们想要分割的输入字符串和一个模式。

下一步是应用模式。**一个模式可以匹配零次或多次。要通过不同的分隔符进行拆分，我们应该只设置模式中的所有字符。**

我们将编写一个简单的测试来演示这种方法:

```java
String[] names = example.split("[;:-]");
Assertions.assertEquals(4, names.length);
Assertions.assertArrayEquals(expectedArray, names);
```

我们已经定义了一个测试字符串，它的名字应该按照模式中的字符进行分割。模式本身包含一个分号、一个冒号和一个连字符。当应用于示例字符串时，我们将在数组中获得四个名称。

### 2.2.番石榴溶液

[Guava](/web/20220524034154/https://www.baeldung.com/guava-guide) 也提供了一个通过多重分隔符分割字符串的解决方案。它的解决方案基于一个`Splitter `类。这个类使用分隔符序列从输入字符串中提取子字符串。我们可以用多种方式定义这个序列:

*   作为单个字符
*   固定的字符串
*   正则表达式
*   一个实例

此外，`Splitter`类有两种方法来定义分隔符。所以，我们来测试一下他们两个。

首先，我们将添加[番石榴](https://web.archive.org/web/20220524034154/https://search.maven.org/artifact/com.google.guava/guava/31.0.1-jre/bundle)依赖关系:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

然后，我们将从`on`方法开始:`public static Splitter on(Pattern separatorPattern)`

它采用定义拆分分隔符的模式。首先，我们将定义分隔符的组合并编译模式。之后，我们就可以拆弦了。

在我们的示例中，我们将使用正则表达式来指定分隔符:

```java
Iterable<String> names = Splitter.on(Pattern.compile("[;:-]")).split(example);
Assertions.assertEquals(4, Iterators.size(names.iterator()));
Assertions.assertIterableEquals(Arrays.asList(expectedArray), names);
```

另一种方法是`onPattern`方法:`public static Splitter onPattern(String separatorPattern)`

这个方法和前面的方法的区别在于,`onPattern`方法将模式作为一个字符串。不需要像在`on`方法中那样编译它。我们将为测试`onPattern`方法定义相同的分隔符组合:

```java
Iterable<String> names = Splitter.onPattern("[;:-]").split(example);
Assertions.assertEquals(4, Iterators.size(names.iterator()));
Assertions.assertIterableEquals(Arrays.asList(expectedArray), names);
```

在这两个测试中，我们成功地分割了字符串，得到了有四个名字的数组。

因为我们用多个分隔符分割输入字符串，所以我们也可以在`[CharMatcher](/web/20220524034154/https://www.baeldung.com/guava-string-charmatcher) `类中使用 *anyOf* 方法:

```java
Iterable<String> names = Splitter.on(CharMatcher.anyOf(";:-")).split(example);
Assertions.assertEquals(4, Iterators.size(names.iterator()));
Assertions.assertIterableEquals(Arrays.asList(expectedArray), names);
```

这个选项只与`Splitter` 类中的`on`方法一起提供。结果与前两次测试相同。

### 2.3.Apache Commons 解决方案

我们将讨论的最后一个选项可以在 Apache Commons Lang 3 库中找到。

我们首先将 [Apache Commons Lang](https://web.archive.org/web/20220524034154/https://search.maven.org/artifact/org.apache.commons/commons-lang3/3.12.0/jar) 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

接下来，我们将使用来自`StringUtils`类的`split`方法:

```java
String[] names = StringUtils.split(example, ";:-");
Assertions.assertEquals(4, names.length);
Assertions.assertArrayEquals(expectedArray, names);
```

我们只需要定义所有用来分割字符串的字符。调用`split`方法会将`example `字符串分成四个名称。

## 3.结论

在本文中，我们看到了用多个分隔符分割输入字符串的不同选项。首先，我们讨论了一个基于正则表达式和普通 Java 的解决方案。后来，我们展示了番石榴的不同选择。最后，我们用一个基于 Apache Commons Lang 3 库的解决方案结束了我们的示例。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524034154/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)