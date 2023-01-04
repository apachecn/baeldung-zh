# 如何在 Java 中截断字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-truncating-strings>

## 1.概观

在本教程中，我们将学习在 Java 中将`String`截断成所需字符数的多种方法。

我们将首先探索使用 JDK 本身来实现这一点的方法。然后我们将看看如何使用一些流行的第三方库来实现这一点。

## 2.**使用 JDK** 截断一个`String`

Java 提供了许多方便的方法来截断`String`。让我们来看看。

### 2.1.使用 **`String'` s `substring()`** 的方法

`String`类附带了一个叫做`[substring](/web/20221020160428/https://www.baeldung.com/java-substring). `的简便方法，顾名思义`,` **` substring()`返回指定索引**之间给定`String`的部分。

让我们来看看它的实际应用:

```java
static String usingSubstringMethod(String text, int length) {
    if (text.length() <= length) {
        return text;
    } else {
        return text.substring(0, length);
    }
}
```

在上面的例子中，如果指定的`length`大于`text`的长度，我们返回`text`本身。这是因为**传递给`substring()`的一个`length` 大于`String`中的字符数导致一个`IndexOutOfBoundsException`** 。

否则，我们返回从索引零开始延伸到——但不包括——索引处的字符`length.`的子字符串

让我们用一个测试案例来证实这一点:

```java
static final String TEXT = "Welcome to baeldung.com";

@Test
public void givenStringAndLength_whenUsingSubstringMethod_thenTrim() {

    assertEquals(TrimStringOnLength.usingSubstringMethod(TEXT, 10), "Welcome to");
}
```

我们可以看到，**开始索引是包含性的，结束索引是排他性的**。因此，索引`length`处的字符不会包含在返回的子字符串中。

### 2.2.使用`String'`s `split()`的方法

**截断`String`的另一种方法是使用`split()`方法，该方法使用正则表达式将`String`分割成多个部分。**

这里我们将使用一个名为[正向后视](/web/20221020160428/https://www.baeldung.com/java-regex-lookahead-lookbehind#positive-lookbehind)的正则表达式特性来匹配从`String`开头开始的指定数量的字符:

```java
static String usingSplitMethod(String text, int length) {

    String[] results = text.split("(?<=\\G.{" + length + "})");

    return results[0];
}
```

`results`的第一个元素要么是我们截断的`String`，要么是原始的`String`，如果`length`比`text`长的话。

让我们测试一下我们的方法:

```java
@Test
public void givenStringAndLength_whenUsingSplitMethod_thenTrim() {

    assertEquals(TrimStringOnLength.usingSplitMethod(TEXT, 13), "Welcome to ba");
}
```

### 2.3.使用`Pattern `类

类似地，**我们可以使用`Pattern `类来编译一个[正则表达式](/web/20221020160428/https://www.baeldung.com/regular-expressions-java)，它匹配`String`的开始到指定数量的字符**。

例如，让我们使用 `{1,” + length + “}.` 这个正则表达式匹配至少一个最多`length`个字符:

```java
static String usingPattern(String text, int length) {

    Optional<String> result = Pattern.compile(".{1," + length + "}")
      .matcher(text)
      .results()
      .map(MatchResult::group)
      .findFirst();

    return result.isPresent() ? result.get() : EMPTY;

}
```

如上所述，在将我们的正则表达式编译成一个`Pattern`之后，我们可以使用`Pattern's` `matcher()`方法根据那个正则表达式来解释我们的`String`。然后，我们能够对结果进行分组，并返回第一个结果，这是我们截断的`String`。

现在让我们添加一个测试用例来验证我们的代码是否如预期的那样工作:

```java
@Test
public void givenStringAndLength_whenUsingPattern_thenTrim() {

    assertEquals(TrimStringOnLength.usingPattern(TEXT, 19), "Welcome to baeldung");
}
```

### 2.4.使用`CharSequence'` s c `odePoints()`方法

Java 9 提供了一个`codePoints()`方法来将一个`String`转换成一个由[代码点值](https://web.archive.org/web/20221020160428/https://developer.mozilla.org/en-US/docs/Glossary/Code_point)组成的流。

让我们看看如何使用这个方法和[流 API](/web/20221020160428/https://www.baeldung.com/java-8-streams) 来截断一个字符串:

```java
static String usingCodePointsMethod(String text, int length) {

    return text.codePoints()
      .limit(length)
      .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
      .toString();
}
```

这里，我们使用了 [`limit()`](/web/20221020160428/https://www.baeldung.com/java-stream-skip-vs-limit#limit) 的方法将`Stream`限制为给定的`length`。然后，我们使用`StringBuilder`来构建我们的截断字符串。

接下来，让我们验证我们的方法是否可行:

```java
@Test
public void givenStringAndLength_whenUsingCodePointsMethod_thenTrim() {

    assertEquals(TrimStringOnLength.usingCodePointsMethod(TEXT, 6), "Welcom");
}
```

## 3.阿帕奇公共图书馆

[Apache Commons Lang](/web/20221020160428/https://www.baeldung.com/java-commons-lang-3) 库包括一个用于操作`String`的`StringUtils`类

首先，让我们将 Apache Commons [依赖项](https://web.archive.org/web/20221020160428/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

### 3.1.使用`StringUtils's left()` 方法

`StringUtils`有一个有用的`static`方法叫做`left()`。 **`StringUtils.left()`以空安全的方式返回指定数量的`String`最左边的字符:**

```java
static String usingLeftMethod(String text, int length) {

    return StringUtils.left(text, length);
}
```

### 3.2.使用`StringUtils'` s `truncate()` 方法

或者，我们可以使用`StringUtils.truncate()`来完成同样的目标:

```java
public static String usingTruncateMethod(String text, int length) {

    return StringUtils.truncate(text, length);
}
```

## 4.番石榴图书馆

除了使用核心 Java 方法和 Apache Commons 库来截断一个`String`，我们还可以使用[番石榴](/web/20221020160428/https://www.baeldung.com/guava-guide)。让我们首先将番石榴[依赖关系](https://web.archive.org/web/20221020160428/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava)添加到我们的`pom.xml` 文件中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在我们可以使用 Guava 的`Splitter`类来截断我们的`String`:

```java
static String usingSplitter(String text, int length) {

    Iterable<String> parts = Splitter.fixedLength(length)
      .split(text);

    return parts.iterator()
      .next();
}
```

**我们使用`Splitter.fixedLength()` 将我们的`String`分割成给定`length`的多个片段。**然后，我们返回结果的第一个元素。

## 5.结论

在本文中，我们学习了在 Java 中将`String`截断成特定数量字符的各种方法。

我们研究了一些使用 JDK 的方法。然后我们使用几个第三方库截断了`String` s。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221020160428/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)