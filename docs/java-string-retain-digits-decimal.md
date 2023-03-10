# 在字符串中只保留数字和小数点分隔符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-retain-digits-decimal>

## 1.概观

假设我们需要从包含字母数字和特殊字符的`String`中删除所有非数字字符，同时保留小数点分隔符。例如，我们希望从“这个包的价格是 100.5 美元”中提取文本的数字和小数部分，以得到“100.5”，这是价格部分。

在本教程中，我们将探索用 Java 实现这一目的的四种不同方法。

## 2.使用正则表达式和`String`的`replaceAll()`方法

最简单的方法是使用`String`类的内置`[replaceAll()](/web/20221113102735/https://www.baeldung.com/string/replace-all) `方法。它用指定的替换项替换该文本中与提供的正则表达式匹配的每个部分。

`replaceAll()`方法有两个参数:正则表达式和替换。

因此，如果**我们将一个相关的正则表达式和一个空字符串作为替换参数传递给方法**，就可以达到我们的目的。

为了简单起见，我们将定义一个单元测试来验证预期的结果:

```java
String s = "Testing abc123.555abc";
s = s.replaceAll("[^\\d.]", "");
assertEquals("123.555", s);
```

在上面的测试案例中，我们将正则表达式定义为 **`[^\\d.]` 来表示一个求反的集合，该集合匹配不在包含任何数字字符(0-9)和“.”的集合中的任何字符人物**。

上面的测试成功执行，从而验证了最终结果只包含数字字符和一个小数分隔符。

## 3.使用 Java 8 `Stream`

使用 [Java 8 Streams](/web/20221113102735/https://www.baeldung.com/java-streams) ，我们有能力在不同的小步骤中定义一系列数据操作:

```java
String s = "Testing abc123.555abc"; 
StringBuilder sb = new StringBuilder(); 
s.chars() 
  .mapToObj(c -> (char) c) 
  .filter(c -> Character.isDigit(c) || c == '.') 
  .forEach(sb::append); 
assertEquals("123.555", sb.toString());
```

首先，我们创建了一个`StringBuilder`实例来保存最终结果。然后，我们使用`chars()`方法迭代`String`中的单个字符，这将返回`int`的流，它本质上是字符代码。为了处理这种情况，我们使用了一个映射函数`mapToObj()`，它返回一个`Character`的`Stream`。

最后，**我们使用`filter()`方法只选择那些数字或者小数点**的字符。

## 4.使用外部库

我们也可以通过将一些外部库如 [Guava](/web/20221113102735/https://www.baeldung.com/guava-guide) 和 Apache Commons 集成到我们的代码库中来解决我们的问题。我们可以利用这些库中预定义的实用程序类。

### 4.1.番石榴

为了使用番石榴、**删除所有非数字字符，但保留 Java 中的小数点分隔符，我们将使用 [`CharMatcher`](/web/20221113102735/https://www.baeldung.com/guava-string-charmatcher) 实用程序类**中的方法。

为了包含 [`Guava`](https://web.archive.org/web/20221113102735/https://mvnrepository.com/artifact/com.google.guava/guava/31.1-jre) ，我们首先需要更新我们的`pom.xml`文件:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

接下来，让我们使用来自`CharMatcher`类的方法重写单元测试:

```java
String s = "Testing abc123.555abc";
String result = CharMatcher.inRange('0', '9')
  .or(CharMatcher.is('.'))
  .retainFrom(s);
assertEquals("123.555", result);
```

如果我们运行测试，它会成功执行并返回预期的结果。为了清楚起见，让我们回顾一下我们用过的方法:

*   `inRange()`方法接受两个`char`参数 `startInclusive` 和 `endInclusive`，并匹配给定范围内定义的字符。
*   `or() `方法接受一个`CharMatcher` 类型的参数。它通过匹配这个匹配器或调用它的那个匹配器来返回一个匹配器。
*   `is()`方法采用单个参数， `char match.`它只匹配一个指定的字符。
*   `retainFrom()`方法采用单个参数，`CharSequence sequence.` **It** **从满足指定匹配标准**的字符序列中返回字符。

### 4.2 .Apache common(Apache 公共)

在 [Apache Commons](/web/20221113102735/https://www.baeldung.com/java-commons-lang-3) 中， **`RegExUtils`类** **提供了一个简单的方法 [`removeAll(String text, String regex)`](https://web.archive.org/web/20221113102735/https://commons.apache.org/proper/commons-lang/javadocs/api-3.9/org/apache/commons/lang3/RegExUtils.html#removeAll-java.lang.String-java.lang.String-) 来删除所有符合 regex** 中指定标准的字符。

为了包含 [`Apache Commons Lang`](https://web.archive.org/web/20221113102735/https://mvnrepository.com/artifact/org.apache.commons/commons-lang3/3.12.0) ，我们需要更新我们的`pom.xml`文件:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

如果我们看一下`RegExUtils` 类，我们会发现它的`removeAll() `方法可以帮助我们解决问题:

```java
String s = "Testing abc123.555abc";
String result = RegExUtils.removeAll(s, "[^\\d.]");
assertEquals("123.555", result);
```

`RegExUtils.removeAll()`需要两个`String`参数，`text`和`regex`。这里，我们用与上面的`String.replaceAll`例子相同的方式定义了`regex`。

## 5.结论

在本文中，我们探索了四种不同的方法来删除 Java `String`中的所有非数字字符，同时保留小数点分隔符。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20221113102735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis-2)