# 用 Apache Commons Lang 3 进行字符串处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/string-processing-commons-lang>

## 1。概述

Apache Commons Lang 3 库提供了对 Java APIs 核心类的操作支持。这种支持包括处理字符串、数字、日期、并发性、对象反射等的方法。

除了提供库的一般介绍，本教程还演示了用于操纵`String`实例的`StringUtils`类的方法。

## 2。Maven 依赖关系

为了使用 Commons Lang 3 库，只需使用以下依赖项从中央 Maven 存储库中取出它:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

你可以在这里找到这个库的最新版本。

## 3。`StringUtils`

`StringUtils`类提供了对字符串进行`null`安全操作的方法。

这个类的很多方法都在类 [`java.lang.String`](https://web.archive.org/web/20221129003903/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) 中定义了相应的方法，这些方法不是`null`安全的。然而，本节将关注几个在`String`类中没有对等物的方法。

## 4。`containsAny`法

`containsAny`方法检查给定的`String`是否包含给定字符集中的任何字符。这组字符可以以`String`或`char` varargs 的形式传递。

下面的代码片段演示了此方法的两种重载形式在结果验证中的使用:

```java
String string = "baeldung.com";
boolean contained1 = StringUtils.containsAny(string, 'a', 'b', 'c');
boolean contained2 = StringUtils.containsAny(string, 'x', 'y', 'z');
boolean contained3 = StringUtils.containsAny(string, "abc");
boolean contained4 = StringUtils.containsAny(string, "xyz");

assertTrue(contained1);
assertFalse(contained2);
assertTrue(contained3);
assertFalse(contained4);
```

## 5。`containsIgnoreCase`法

`containsIgnoreCase`方法以不区分大小写的方式检查给定的`String`是否包含另一个`String`。

以下代码片段验证了当忽略大小写时`String “baeldung.com”`包含`“BAELDUNG”`:

```java
String string = "baeldung.com";
boolean contained = StringUtils.containsIgnoreCase(string, "BAELDUNG");

assertTrue(contained);
```

## 6。`countMatches`法

`counterMatches`方法计算一个字符或子串在给定的`String.`中出现的次数

下面是这个方法的演示，确认在`String` `“welcome to www.baeldung.com”`中`‘w'`出现四次，`“com”`出现两次:

```java
String string = "welcome to www.baeldung.com";
int charNum = StringUtils.countMatches(string, 'w');
int stringNum = StringUtils.countMatches(string, "com");

assertEquals(4, charNum);
assertEquals(2, stringNum);
```

## 7。追加和前置方法

如果给定的`String`还没有以任何传入的后缀结尾，那么`appendIfMissing`和`appendIfMissingIgnoreCase`方法分别以区分大小写和不区分大小写的方式将后缀附加到给定的【】的末尾。

类似地，`prependIfMissing`和`prependIfMissingIgnoreCase`方法在给定的`String`的开头加上前缀，如果它不是以任何传入的前缀开头的话。

在下面的例子中，`appendIfMissing`和`prependIfMissing`方法用于为`String “baeldung.com”`添加后缀和前缀，而这些词缀不会重复:

```java
String string = "baeldung.com";
String stringWithSuffix = StringUtils.appendIfMissing(string, ".com");
String stringWithPrefix = StringUtils.prependIfMissing(string, "www.");

assertEquals("baeldung.com", stringWithSuffix);
assertEquals("www.baeldung.com", stringWithPrefix);
```

## 8。案例改变方法

`String`类已经定义了将`String`的所有字符转换成大写或小写的方法。本小节仅说明以其他方式改变`String`大小写的方法的使用，包括`swapCase`、`capitalize`和`uncapitalize`。

`swapCase`方法将`String,`的大小写从大写转换为小写，从小写转换为大写:

```java
String originalString = "baeldung.COM";
String swappedString = StringUtils.swapCase(originalString);

assertEquals("BAELDUNG.com", swappedString);
```

`capitalize`方法将给定的`String`的第一个字符转换成大写，其余的字符保持不变:

```java
String originalString = "baeldung";
String capitalizedString = StringUtils.capitalize(originalString);

assertEquals("Baeldung", capitalizedString);
```

`uncapitalize`方法将给定的`String`的第一个字符转换成小写，其余的字符保持不变:

```java
String originalString = "Baeldung";
String uncapitalizedString = StringUtils.uncapitalize(originalString);

assertEquals("baeldung", uncapitalizedString);
```

## 9。反转方法

`StringUtils`类定义了两种反转字符串的方法:`reverse`和`reverseDelimited`。`reverse`方法以相反的顺序重新排列一个`String`的所有字符，而`reverseDelimited`方法重新排列由指定分隔符分隔的字符组。

以下代码片段反转字符串`“baeldung”`并验证结果:

```java
String originalString = "baeldung";
String reversedString = StringUtils.reverse(originalString);

assertEquals("gnudleab", reversedString);
```

使用`reverseDelimited`方法，字符成组反转，而不是单个反转:

```java
String originalString = "www.baeldung.com";
String reversedString = StringUtils.reverseDelimited(originalString, '.');

assertEquals("com.baeldung.www", reversedString);
```

## 10。`rotate()`法

`rotate()`方法将`String`的字符循环移位若干个位置。下面的代码片段将`String “baeldung”`的所有字符向右移动四个位置，并验证结果:

```java
String originalString = "baeldung";
String rotatedString = StringUtils.rotate(originalString, 4);

assertEquals("dungbael", rotatedString);
```

## 11。`difference`法

`difference`方法比较两个字符串，从与第一个不同的位置开始返回第二个`String,`的剩余部分。以下代码片段在两个方向上比较两个`Strings: “Baeldung Tutorials”`和`“Baeldung Courses”`，并验证结果:

```java
String tutorials = "Baeldung Tutorials";
String courses = "Baeldung Courses";
String diff1 = StringUtils.difference(tutorials, courses);
String diff2 = StringUtils.difference(courses, tutorials);

assertEquals("Courses", diff1);
assertEquals("Tutorials", diff2);
```

## 12。结论

本教程介绍了 Apache Commons Lang 3 中的字符串处理，并介绍了我们可以在`StringUtils`库类之外使用的主要 API。

与往常一样，上面给出的所有示例和代码片段的实现都可以在 GitHub 项目中找到。