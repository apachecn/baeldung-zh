# 用 Java 从字符串中获取子串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-substring>

## 1。概述

在这个快速教程中，我们将关注 Java 中字符串的子串功能。

我们将主要使用来自 [`String`](https://web.archive.org/web/20220829143229/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) 类的方法，很少使用来自 Apache Commons 的 [`StringUtils`](https://web.archive.org/web/20220829143229/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html) 类的方法。

在以下所有示例中，我们将使用这个简单的字符串:

```java
String text = "Julia Evans was born on 25-09-1984\. "
  + "She is currently living in the USA (United States of America).";
```

## 2。`substring` 基础知识

让我们从一个非常简单的例子开始——提取带有起始索引的子字符串:

```java
assertEquals("USA (United States of America).", 
  text.substring(67));
```

请注意，在我们的示例中，我们是如何提取 Julia 的居住国的。

**还有一个选项来指定结束索引**，但是如果没有它的话—`substring`将一直到`String. `的结尾

让我们这样做，并去掉最后多余的点，在上面的例子中:

```java
assertEquals("USA (United States of America)", 
  text.substring(67, text.length() - 1));
```

在上面的例子中，我们使用了精确的位置来提取子串。

### 2.1。获取从特定字符开始的子字符串

**如果需要根据一个角色或者`String`动态计算位置，我们可以使用`indexOf`的方法:**

```java
assertEquals("United States of America", 
  text.substring(text.indexOf('(') + 1, text.indexOf(')')));
```

一个类似的可以帮助我们定位子串的方法是`lastIndexOf`。我们用`lastIndexOf`提取年份“1984”。它是最后一个破折号和第一个点之间的文本部分:

```java
assertEquals("1984",
  text.substring(text.lastIndexOf('-') + 1, text.indexOf('.')));
```

`indexOf`和`lastIndexOf`都可以将一个字符或一个`String`作为参数。让我们提取文本“USA”和括号中的其余文本:

```java
assertEquals("USA (United States of America)",
  text.substring(text.indexOf("USA"), text.indexOf(')') + 1));
```

## 3。使用`subSequence`

`String`类提供了另一个名为`subSequence`的方法，其行为类似于`substring`方法。

**唯一的区别是它返回一个 [`CharSequence`](https://web.archive.org/web/20220829143229/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/CharSequence.html) 而不是一个`String` ，并且它只能与一个特定的开始和结束索引一起使用:**

```java
assertEquals("USA (United States of America)", 
  text.subSequence(67, text.length() - 1));
```

## 4。使用正则表达式

如果我们必须提取与特定模式匹配的子串，正则表达式将会帮助我们。

在示例`String,`中，Julia 的出生日期格式为“dd-mm-yyyy”。我们可以使用 Java 正则表达式 API 来匹配这个模式。

首先，我们需要为“dd-mm-yyyy”创建一个模式:

```java
Pattern pattern = Pattern.compile("\\d{2}-\\d{2}-\\d{4}");
```

然后，我们将应用该模式从给定文本中查找匹配:

```java
Matcher matcher = pattern.matcher(text);
```

匹配成功后，我们可以提取匹配的`String:`

```java
if (matcher.find()) {                                  
    Assert.assertEquals("25-09-1984", matcher.group());
}
```

关于 Java 正则表达式的更多细节，请查看本教程。

## 5。使用`split`

我们可以使用来自`String`类的`split`方法来提取一个子串。假设我们想从示例`String.`中提取第一句话，使用`split`很容易做到:

```java
String[] sentences = text.split("\\.");
```

因为 split 方法接受正则表达式，所以我们必须对句点字符进行转义。现在结果是一个包含两个句子的数组。

我们可以使用第一句话(或者遍历整个数组):

```java
assertEquals("Julia Evans was born on 25-09-1984", sentences[0]);
```

请注意，使用 Apache OpenNLP 有更好的句子检测和标记化方法。查看[这篇](/web/20220829143229/https://www.baeldung.com/apache-open-nlp)教程，了解更多关于 OpenNLP API 的信息。

## 6。使用`Scanner`

我们一般用 [`Scanner`](https://web.archive.org/web/20220829143229/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html) 解析原语类型，用`Strings`解析正则表达式。**`Scanner`使用一个分隔符模式**将其输入分解成标记，默认情况下匹配空白。

让我们看看如何使用它来获得示例文本中的第一个句子:

```java
try (Scanner scanner = new Scanner(text)) {
    scanner.useDelimiter("\\.");           
    assertEquals("Julia Evans was born on 25-09-1984", scanner.next());    
}
```

在上面的例子中，我们已经将示例`String`设置为扫描仪要使用的源。

然后，我们将句点字符设置为分隔符(需要对其进行转义，否则它将被视为上下文中的特殊正则表达式字符)。

最后，我们断言这个分隔输出中的第一个令牌。

如果需要，我们可以使用一个`while`循环遍历整个令牌集合。

```java
while (scanner.hasNext()) {
   // do something with the tokens returned by scanner.next()
}
```

## 7。Maven 依赖关系

我们可以更进一步，使用一个有用的实用程序——`StringUtils`类——[Apache Commons Lang](https://web.archive.org/web/20220829143229/https://commons.apache.org/proper/commons-lang/)库的一部分:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

你可以在这里找到这个库的最新版本。

## 8。使用`StringUtils`

Apache Commons 库增加了一些操作核心 Java 类型的有用方法。Apache Commons Lang 为 java.lang API 提供了许多辅助工具，最著名的是`String`操作方法。

在这个例子中，我们将看到**如何提取嵌套在两个`Strings:`** 之间的子串

```java
assertEquals("United States of America", 
  StringUtils.substringBetween(text, "(", ")"));
```

如果子字符串嵌套在同一个`String:`的两个实例之间，这种方法有一个简化版本

```java
substringBetween(String str, String tag)
```

来自同一个类的`substringAfter`方法获取第一次出现分隔符后的子字符串。

分隔符不返回:

```java
assertEquals("the USA (United States of America).", 
  StringUtils.substringAfter(text, "living in "));
```

类似地，`substringBefore`方法获取第一次出现分隔符之前的子字符串。

分隔符不返回:

```java
assertEquals("Julia Evans", 
  StringUtils.substringBefore(text, " was born"));
```

您可以查看本教程，了解更多关于使用 Apache Commons Lang API 进行处理的信息。

## 9。结论

在这篇简短的文章中，我们发现了在 Java 中从`String`中提取子串的各种方法。你可以探索我们关于 Java 中的`String`操作的[其他教程](/web/20220829143229/https://www.baeldung.com/java-string)。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220829143229/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)