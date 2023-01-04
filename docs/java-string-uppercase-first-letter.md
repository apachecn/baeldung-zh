# Java 中字符串的首字母大写

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-uppercase-first-letter>

## 1.概观

Java 标准库提供了 [`String.toUpperCase()`](/web/20221208143917/https://www.baeldung.com/string/to-upper-case) 方法，允许我们将字符串中的所有字母转换成大写。

在本教程中，我们将学习如何转换一个给定的字符串的第一个字符只大写。

## 2.问题简介

举个例子可以很快说明这个问题。假设我们有一个输入字符串:

```java
String INPUT = "hi there, Nice to Meet You!";
```

给定这个`INPUT`字符串，这是我们预期的结果:

```java
String EXPECTED = "Hi there, Nice to Meet You!";
```

如我们所见，我们只想将第一个字符“`h`”改为“`H`”。然而，**剩余的字符不应该被修改**。

当然，如果输入字符串为空，结果也应该是空字符串:

```java
String EMPTY_INPUT = "";
String EMPTY_EXPECTED = "";
```

在本教程中，我们将提出该问题的几种解决方案。为了简单起见，我们将使用单元测试断言来验证我们的解决方案是否如预期的那样工作。

## 3.使用`substring()`方法

解决这个问题的第一个想法是将输入字符串分成两个子字符串。例如，我们可以将`INPUT`字符串拆分为“`h`和“`i there, Nice ….`”。换句话说，第一个子字符串只包含第一个字符，另一个子字符串包含字符串的剩余字符。

然后，**我们可以对第一个子串应用`toUpperCase()`方法，并连接第二个子串来解决问题**。

Java 的`String`类的 [`substring()`](/web/20221208143917/https://www.baeldung.com/string/substring) 方法可以帮助我们得到这两个子字符串:

*   `INPUT.substring(0, 1)`–包含第一个字符的子串 1
*   `INPUT.substring(1)`–保存剩余字符的子串 2

接下来，让我们编写一个测试来看看这个解决方案是否有效:

```java
String output = INPUT.substring(0, 1).toUpperCase() + INPUT.substring(1);
assertEquals(EXPECTED, output);
```

如果我们进行测试，就会通过。然而，**如果我们的输入是一个空字符串，这种方法会引发`IndexOutOfBoundsException`** 。这是因为当我们调用`INPUT.substring(1)`时，end-index ( `1`)大于空字符串的长度(`0`):

```java
assertThrows(IndexOutOfBoundsException.class, () -> EMPTY_INPUT.substring(1));
```

进一步，我们要注意，如果输入字符串是`null`，这个方法会抛出`NullPointerException`。

**因此，在使用子串方法之前，我们需要检查并确保输入字符串不是`null`或空的**。

## 4.使用`Matcher.replaceAll()`方法

解决问题的另一个思路是使用[regex](/web/20221208143917/https://www.baeldung.com/regular-expressions-java)(“`^.`”)匹配第一个字符，并将匹配的组转换为大写。

在 Java 9 之前这并不是一件容易的事情。这是因为 [`Matcher`的替换方法](/web/20221208143917/https://www.baeldung.com/regular-expressions-java#123-replacement-methods)，比如`replaceAll()``replaceFirst(),`不支持`Function`对象或者 lambda 表达式替换器。然而，这在 Java 9 中已经改变了。

**从 Java 9 开始，`Matcher`的替换方法支持一个`Function`对象作为替换对象。**也就是说，我们可以用一个函数来处理匹配的字符序列并完成替换。当然，要解决我们的问题，我们只需要在匹配的字符上调用`toUpperCase()`方法:

```java
String output = Pattern.compile("^.").matcher(INPUT).replaceFirst(m -> m.group().toUpperCase());
assertEquals(EXPECTED, output);
```

如果我们试一试，测试就会通过。

如果正则表达式什么都不匹配，替换就不会发生。因此，**这个解决方案也适用于空输入字符串**:

```java
String emptyOutput = Pattern.compile("^.").matcher(EMPTY_INPUT).replaceFirst(m -> m.group().toUpperCase());
assertEquals(EMPTY_EXPECTED, emptyOutput);
```

值得一提的是，如果输入字符串是`null`，这个解也会抛出`NullPointerException`。所以，**我们在使用它之前还是需要做一个`null`检查。**

## 5.使用 Apache Commons Lang 3 中的`StringUtils`

Apache Commons Lang3 是一个很受欢迎的图书馆。它附带了许多方便的实用程序类，并扩展了标准 Java 库的功能。

**它的 [`StringUtils`](/web/20221208143917/https://www.baeldung.com/java-commons-lang-3#the-stringutils-class) 类提供了`capitalize()`方法，直接解决了我们的问题。**

要使用这个库，让我们首先添加 [Maven 依赖项](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav):

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

然后，像往常一样，让我们创建一个测试来看看它是如何工作的:

```java
String output = StringUtils.capitalize(INPUT);
assertEquals(EXPECTED, output);
```

如果我们执行它，测试就通过了。正如我们所看到的，我们简单地调用`StringUtils.capitalize(INPUT).` 然后库为我们做了这项工作。

值得一提的是**`StringUtils.capitalize()`方法是空安全的，也适用于空输入字符串**:

```java
String emptyOutput = StringUtils.capitalize(EMPTY_INPUT);
assertEquals(EMPTY_EXPECTED, emptyOutput);
String nullOutput = StringUtils.capitalize(null);
assertNull(nullOutput);
```

## 6.结论

在本文中，我们学习了如何将给定字符串的第一个字符转换成大写。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)