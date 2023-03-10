# 比较字符串，同时忽略 Java 中的空白

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compare-string-whitespace>

## 1.概观

在这个简短的教程中，我们将看到如何**比较字符串，同时忽略 Java** 中的空白。

## 2.使用`replaceAll()`方法

假设我们有两个字符串，一个包含空格，另一个只包含非空格字符:

```java
String normalString = "ABCDEF";
String stringWithSpaces = " AB  CD EF ";
```

我们可以简单地比较它们，同时使用`String`类的内置`replaceAll()`方法忽略空格:

```java
assertEquals(normalString.replaceAll("\\s+",""), stringWithSpaces.replaceAll("\\s+",""));
```

使用上面的`replaceAll()`方法将**移除我们字符串中的所有空格，包括不可见的字符**，如 tab、\n 等。

[除了\s+，我们还可以用\s](/web/20221208143815/https://www.baeldung.com/java-regex-s-splus) 。

## 3.使用 Apache Commons Lang

接下来，我们可以使用来自 [Apache Commons Lang](/web/20221208143815/https://www.baeldung.com/java-commons-lang-3) 库的`StringUtils`类来实现相同的目标。

这个类有一个方法`deleteWhitespace()`，用来删除一个`String`中的所有空格:

```java
assertEquals(StringUtils.deleteWhitespace(normalString), StringUtils.deleteWhitespace(stringWithSpaces));
```

## 4.使用 Spring 框架的`StringUtils`类

最后，如果我们的项目已经在使用 Spring 框架，我们可以使用`org.springframework.util`包中的`StringUtils`类。

使用这个时间的方法是`trimAllWhitespace()`:

```java
assertEquals(StringUtils.trimAllWhitespace(normalString), StringUtils.trimAllWhitespace(stringWithSpaces));
```

我们应该记住，如果我们想要比较空格有意义的字符串，比如人名，我们不应该使用本文中的方法。例如，以下两个字符串将被视为相等:“成龙”和“JACKIE 成龙”，这可能不是我们实际想要的。

## 5.结论

在本文中，我们看到了不同的方法来比较字符串，同时忽略 Java 中的空格。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143815/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)