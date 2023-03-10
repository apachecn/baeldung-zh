# 计算 Java 字符串中的空格数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-count-spaces>

## 1.概观

当我们使用 Java 字符串时，有时我们会想计算一个字符串中有多少个空格。

有各种方法可以得到这个结果。在这个快速教程中，我们将通过例子来看看如何做到这一点。

## 2.示例输入字符串

首先，让我们准备一个输入字符串作为例子:

```java
String INPUT_STRING = "  This string has nine spaces and a Tab:'	'";
```

上面的字符串包含九个空格和一个用单引号括起来的制表符。**我们的目标是只计算给定输入字符串**中的空格字符。

因此，我们的预期结果是:

```java
int EXPECTED_COUNT = 9;
```

接下来，让我们探索各种解决方案来获得正确的结果。

我们将首先使用 Java 标准库来解决这个问题，然后我们将使用一些流行的外部库来解决它。

最后，在本教程中，我们将解决单元测试方法中的所有解决方案。

## 3.使用 Java 标准库

### 3.1.经典解决方案:循环和计数

这大概是解决问题最直白的思路了。

我们遍历输入字符串中的所有字符。此外，我们维护一个计数器变量，并在看到空格字符时递增计数器。

最后，我们将得到字符串中空格的数量:

```java
@Test
void givenString_whenCountSpaceByLooping_thenReturnsExpectedCount() {
    int spaceCount = 0;
    for (char c : INPUT_STRING.toCharArray()) {
        if (c == ' ') {
            spaceCount++;
        }
    }
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

### 3.2.使用 Java 8 的流 API

[Stream API](/web/20221208143917/https://www.baeldung.com/java-8-streams) 从 Java 8 开始就有了。

另外，**从 Java 9 开始，`String`类中增加了一个新的 [`chars()`](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#chars()) 方法，将`String`中的`char`值转换成`IntStream`实例**。

如果我们使用的是 Java 9 或更高版本，我们可以将这两个特性结合起来，用一行代码解决这个问题:

```java
@Test
void givenString_whenCountSpaceByJava8StreamFilter_thenReturnsExpectedCount() {
    long spaceCount = INPUT_STRING.chars().filter(c -> c == (int) ' ').count();
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

### 3.3.使用 Regex 的`Matcher.find()` 方法

到目前为止，我们已经看到了通过搜索给定字符串中的空格字符进行计数的解决方案。我们已经使用了`character == ‘ ‘ `来检查一个字符是否是空格字符。

**正则表达式(Regex)是搜索字符串的另一个有力武器，Java 对 Regex 有很好的支持。**

因此，我们可以将单个空格定义为一个模式，并使用`[Matcher.find()](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Matcher.html#find())`方法来检查该模式是否在输入字符串中找到。

此外，为了得到空间的计数，我们在每次发现模式时增加一个计数器:

```java
@Test
void givenString_whenCountSpaceByRegexMatcher_thenReturnsExpectedCount() {
    Pattern pattern = Pattern.compile(" ");
    Matcher matcher = pattern.matcher(INPUT_STRING);
    int spaceCount = 0;
    while (matcher.find()) {
        spaceCount++;
    }
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

### 3.4.使用`String.replaceAll()`方法

使用`Matcher.find()`方法搜索和查找空格非常简单。然而，既然我们讨论的是正则表达式，还有其他快速计算空格的方法。

我们知道我们可以使用`[String.replaceAll()](/web/20221208143917/https://www.baeldung.com/string/replace-all)`方法进行“搜索和替换”。

因此，**如果我们用空字符串替换输入字符串中的所有非空格字符，那么输入中的所有空格都将是结果**。

所以，如果我们想得到计数，结果字符串的长度就是答案。接下来，让我们试试这个想法:

```java
@Test
void givenString_whenCountSpaceByReplaceAll_thenReturnsExpectedCount() {
    int spaceCount = INPUT_STRING.replaceAll("[^ ]", "").length();
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

如上面的代码所示，我们只有一行代码来获取计数。

值得一提的是，在`String.replaceAll()`调用中，我们使用了模式`“[^ ]”`而不是`“\\S”. `，这是因为我们希望替换非空格字符，而不仅仅是非空白字符。

### 3.5.使用`String.split()`方法

我们已经看到了使用`String.replaceAll()`方法的解决方案简洁而紧凑。现在，让我们看看解决问题的另一种思路:使用 [`String.split()`](/web/20221208143917/https://www.baeldung.com/string/split) 方法。

正如我们所知，我们可以将一个模式传递给`String.split()`方法，并获得一个由该模式拆分的字符串数组。

所以，我们可以用一个空格分割输入字符串。然后，原始字符串中的空格数将比字符串数组长度小 1。

现在，让我们看看这个想法是否可行:

```java
@Test
void givenString_whenCountSpaceBySplit_thenReturnsExpectedCount() {
    int spaceCount = INPUT_STRING.split(" ").length - 1;
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

## 4.使用外部库

[Apache Commons Lang 3](/web/20221208143917/https://www.baeldung.com/java-commons-lang-3) 库在 Java 项目中被广泛使用。另外， [Spring](/web/20221208143917/https://www.baeldung.com/spring-tutorial) 是 Java 爱好者中流行的框架。

两个库都提供了一个方便的字符串实用程序类。

现在，让我们看看如何使用这些库计算输入字符串中的空格。

### 4.1.使用 Apache Commons Lang 3 库

Apache Commons Lang 3 库提供了一个包含许多方便的字符串相关方法的`StringUtil`类。

**要统计一个字符串中的空格，我们可以使用这个类中的 [`countMatches()`](https://web.archive.org/web/20221208143917/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#countMatches-java.lang.CharSequence-char-) 方法。**

在我们开始使用`StringUtil`类之前，我们应该检查这个库是否在类路径中。我们可以在我们的`pom.xml`中添加与[最新版本](https://web.archive.org/web/20221208143917/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)的依赖关系:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

现在，让我们创建一个单元测试来展示如何使用这种方法:

```java
@Test
void givenString_whenCountSpaceUsingApacheCommons_thenReturnsExpectedCount() {
    int spaceCount = StringUtils.countMatches(INPUT_STRING, " ");
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

### 4.2.使用弹簧

今天，很多 Java 项目都是基于 Spring 框架的。所以，如果我们正在使用 Spring，Spring 提供的一个很好的字符串工具已经准备好可以使用了: [`StringUtils`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/StringUtils.html) 。

是的，它和 Apache Commons Lang 3 中的类同名。而且，它提供了一个 [`countOccurrencesOf()`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/StringUtils.html#countOccurrencesOf-java.lang.String-java.lang.String-) 方法来统计一个字符在字符串中的出现次数。

这正是我们想要的:

```java
@Test
void givenString_whenCountSpaceUsingSpring_thenReturnsExpectedCount() {
    int spaceCount = StringUtils.countOccurrencesOf(INPUT_STRING, " ");
    assertThat(spaceCount).isEqualTo(EXPECTED_COUNT);
} 
```

## 5.结论

在本文中，我们讨论了计算输入字符串中空格字符的不同方法。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)