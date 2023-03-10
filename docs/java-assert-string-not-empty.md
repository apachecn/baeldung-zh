# Java 中的字符串非空测试断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-assert-string-not-empty>

## 1。概述

在某些场景中，我们需要断言给定的`String`是否为空。在 Java 中有相当多的方法来做这样的断言。

在这个快速教程中，让我们探索一些测试断言技术。

## 2。Maven 依赖关系

我们需要先获取一些依赖项。在 Maven 项目中，我们可以向`pom.xml`添加以下依赖项:

**[朱尼](https://web.archive.org/web/20221205110525/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22junit%22%20AND%20a%3A%22junit%22) :**

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

**:**

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-core</artifactId>
    <version>1.3</version>
</dependency>
```

**[【Apache common lang】](https://web.archive.org/web/20221205110525/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22):**

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

**评估**

```java
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.11.1</version>
</dependency>
```

**[谷歌番石榴](https://web.archive.org/web/20221205110525/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) :**

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

## 3。使用 JUnit

**我们将使用来自`[String](https://web.archive.org/web/20221205110525/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html)`类的 [`isEmpty`](/web/20221205110525/https://www.baeldung.com/string/is-empty) 方法以及来自 JUnit 的 [`Assert`](https://web.archive.org/web/20221205110525/http://junit.sourceforge.net/javadoc/org/junit/Assert.html) 类来验证给定的`String`是否为空。**因为如果输入`String`为空，则`isEmpty`方法返回 true，所以我们可以将它与`assertFalse` 方法一起使用:

```java
assertFalse(text.isEmpty());
```

或者，我们也可以使用:

```java
assertTrue(!text.isEmpty());
```

**认为既然`text`可能为空，**另一种方法是使用`assertNotEquals`方法进行等式检查:

```java
assertNotEquals("", text);
```

或者:

```java
assertNotSame("", text);
```

点击查看我们关于 JUnit 断言[的深入指南。](/web/20221205110525/https://www.baeldung.com/junit-assertions)

所有这些断言，当失败时，将返回一个`AssertionError.`

## 4。使用 Hamcrest 核心

Hamcrest 是一个众所周知的框架，它提供了在 Java 生态系统中常用于单元测试的匹配器。

**我们可以利用 Hamcrest `CoreMatchers `类进行空字符串检查**:

```java
assertThat(text, CoreMatchers.not(isEmptyString()));
```

`isEmptyString`方法在 [`IsEmptyString`](https://web.archive.org/web/20221205110525/http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/text/IsEmptyString.html) 类中可用。

失败时，这也会返回 AssertionError，但输出更有帮助:

```java
java.lang.AssertionError: 
Expected: not an empty string
     but: was ""
```

如果需要，为了验证一个字符串既不是空的也不是 null，我们可以使用`isEmptyOrNullString`:

```java
assertThat(text, CoreMatchers.not(isEmptyOrNullString()));
```

要了解`CoreMatchers`类的其他方法，请阅读[这篇](/web/20221205110525/https://www.baeldung.com/hamcrest-core-matchers)以前发表的文章。

## 5。使用 Apache Commons Lang

[Apache Commons Lang](https://web.archive.org/web/20221205110525/https://commons.apache.org/proper/commons-lang/) 库为`java.lang` API 提供了许多辅助工具。

**`StringUtils`类提供了一个方法，我们可以用它来检查空字符串**:

```java
assertTrue(StringUtils.isNotBlank(text));
```

当失败时，这将返回一个简单的`AssertionError.`

要了解关于 Apache Commons 字符串处理的更多信息，Lang 阅读了这篇文章。

## 6。使用资产 J

AssertJ 是一个开源的、社区驱动的库，用于在 Java 测试中编写流畅而丰富的断言。

方法`AbstractCharSequenceAssert.isNotEmpty()` 验证实际的`CharSequence`不为空，或者**换句话说，它不为空并且长度为 1 或更多**:

```java
Assertions.assertThat(text).isNotEmpty()
```

如果失败，将打印输出:

```java
java.lang.AssertionError: 
Expecting actual not to be empty
```

我们有一篇关于 AssertJ [的很好的介绍文章。](/web/20221205110525/https://www.baeldung.com/introduction-to-assertj)

## 7。使用谷歌番石榴

[Guava](https://web.archive.org/web/20221205110525/https://github.com/google/guava) 是 Google 提供的一套核心库。

**Guava`Strings`类的方法`isNullOrEmpty`可以用来验证一个字符串是否为空**(或 null):

```java
assertFalse(Strings.isNullOrEmpty(text));
```

当失败且没有其他输出消息时，这也会返回一个`AssertionError`。

要浏览我们关于番石榴 API 的其他文章，请点击这里的链接。

## 8。结论

在这个快速教程中，我们发现了如何断言给定的`String`是否为空。

和往常一样，完整的代码片段可以在 GitHub 的[中找到。](https://web.archive.org/web/20221205110525/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)