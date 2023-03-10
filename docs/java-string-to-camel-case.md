# 将字符串转换为骆驼大小写

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-camel-case>

## 1.概观

[骆驼大小写和标题大小写](/web/20220726144623/https://www.baeldung.com/java-camel-case-title-case-to-words)通常用作字段和类型的标识符。我们可能希望将文本转换成这种格式。

这可以通过编写自定义代码或利用第三方库来实现。

在本教程中，我们将看看如何编写一些自定义字符串转换为 camel cases，我们将探索一些第三方库功能，可以帮助我们完成这项任务。

## 2.Java 解决方案

**Camel 的 case 允许我们通过删除空格和使用大写字母显示单词边界来连接多个单词。**

有两种类型:

*   小写骆驼，其中第一个单词的第一个字符是小写的
*   大写 camel case，也称为 title case，其中第一个单词的第一个字符是大写的:

```java
thisIsLowerCamelCase
ThisIsLowerCamelCase
```

在本教程中，我们将重点放在转换成小写骆驼，虽然这些技术很容易适应。

### 2.1.正则表达式

我们可以使用[正则表达式](/web/20220726144623/https://www.baeldung.com/regular-expressions-java)将包含单词的字符串拆分成一个数组:

```java
String[] words = text.split("[\\W_]+");
```

这会在不属于单词的任何字符处拆分给定的字符串。下划线通常被视为正则表达式中的单词字符。Camel 的 case 不包括下划线，所以我们将它添加到分隔符表达式中。

当我们有单独的单词时，我们可以修改它们的大小写，然后将它们重组为骆驼大小写:

```java
StringBuilder builder = new StringBuilder();
for (int i = 0; i < words.length; i++) {
    String word = words[i];
    if (i == 0) {
        word = word.isEmpty() ? word : word.toLowerCase();
    } else {
        word = word.isEmpty() ? word : Character.toUpperCase(word.charAt(0)) + word.substring(1).toLowerCase();      
    }
    builder.append(word);
}
return builder.toString();
```

这里，我们将数组中的第一个字符串/单词转换成小写。对于数组中的每一个单词，我们将第一个字符转换为大写，其余的转换为小写。

让我们使用空白作为非单词字符来测试这个方法:

```java
assertThat(toCamelCaseByRegex("THIS STRING SHOULD BE IN CAMEL CASE"))
  .isEqualTo("thisStringShouldBeInCamelCase");
```

这个解决方案很简单，但是它需要一些原始文本的副本来计算答案。首先，它创建一个单词列表，然后以各种大写或小写格式创建这些单词的副本，以组成最终的字符串。**这可能会消耗大量内存，输入非常大**。

### 2.2.遍历字符串

我们可以用一个循环来代替上面的算法，这个循环在每个字符通过原始字符串时计算出每个字符的正确大小写。这会跳过任何分隔符，一次写入一个字符到`StringBuilder`。

首先，我们需要跟踪转换的状态:

```java
boolean shouldConvertNextCharToLower = true;
```

然后我们遍历源文本，跳过或适当地大写每个字符:

```java
for (int i = 0; i < text.length(); i++) {
    char currentChar = text.charAt(i);
    if (currentChar == delimiter) {
        shouldConvertNextCharToLower = false;
    } else if (shouldConvertNextCharToLower) {
        builder.append(Character.toLowerCase(currentChar));
    } else {
        builder.append(Character.toUpperCase(currentChar));
        shouldConvertNextCharToLower = true;
    }
}
return builder.toString();
```

这里的分隔符是一个`char`,它表示预期的非单词字符。

让我们尝试使用空格作为分隔符的解决方案:

```java
assertThat(toCamelCaseByIteration("THIS STRING SHOULD BE IN CAMEL CASE", ' '))
  .isEqualTo("thisStringShouldBeInCamelCase");
```

我们还可以尝试使用下划线分隔符:

```java
assertThat(toCamelCaseByIteration("THIS_STRING_SHOULD_BE_IN_CAMEL_CASE", '_'))
  .isEqualTo("thisStringShouldBeInCamelCase");
```

## 3.使用第三方库

我们可能更喜欢使用第三方库字符串函数，而不是自己编写。

### 3.1.Apache Commons 文本

为了使用 [Apache Commons 文本](/web/20220726144623/https://www.baeldung.com/java-apache-commons-text)，我们需要将它添加到我们的项目中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.9</version>
</dependency>
```

这个库在`CaseUtils`中提供了一个`toCamelCase`方法:

```java
String camelCase = CaseUtils.toCamelCase(text, false, delimiter);
```

让我们试一试:

```java
assertThat(CaseUtils.toCamelCase("THIS STRING SHOULD BE IN CAMEL CASE", false, ' '))
  .isEqualTo("thisStringShouldBeInCamelCase");
```

为了将字符串转换成标题大小写，我们需要将`true `传递给`toCamelCase `方法:

```java
String camelCase = CaseUtils.toCamelCase(text, true, delimiter);
```

让我们试一试:

```java
assertThat(CaseUtils.toCamelCase("THIS STRING SHOULD BE IN CAMEL CASE", true, ' '))
  .isEqualTo("ThisStringShouldBeInCamelCase");
```

### 3.2.番石榴

稍加预处理，我们可以通过[番石榴](/web/20220726144623/https://www.baeldung.com/guava-guide)将字符串转换成骆驼。

要使用 Guava，让我们将它的[依赖项](https://web.archive.org/web/20220726144623/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)添加到我们的项目中:

```java
<dependency>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
      <version>31.0.1-jre</version>
</dependency>
```

Guava 有一个实用程序类`CaseFormat`，用于格式转换:

```java
String camelCase = CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "THIS_STRING_SHOULD_BE_IN_CAMEL_CASE");
```

这将给定的由下划线分隔的大写字符串转换成小写字母。让我们看看:

```java
assertThat(CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "THIS_STRING_SHOULD_BE_IN_CAMEL_CASE"))
  .isEqualTo("thisStringShouldBeInCamelCase");
```

如果我们的字符串已经是这种格式，这没问题。但是，如果我们希望使用不同的分隔符并处理混合的大小写，我们需要预处理我们的输入:

```java
String toUpperUnderscore = "This string should Be in camel Case"
  .toUpperCase()
  .replaceAll(' ', "_");
```

首先，我们将给定的字符串转换成大写。然后，我们用下划线替换所有的分隔符。生成的格式相当于番石榴的`CaseFormat.UPPER_UNDERSCORE. `现在，我们可以使用番石榴制作驼色表壳版本:

```java
assertThat(toCamelCaseUsingGuava("THIS STRING SHOULD BE IN CAMEL CASE", " "))
  .isEqualTo("thisStringShouldBeInCamelCase");
```

## 4.结论

在本教程中，我们学习了如何将一个字符串转换成一个驼峰式大小写。

首先，我们构建了一个算法来将字符串拆分成单词。然后我们建立了一个迭代每个字符的算法。

最后，我们看了如何使用一些第三方库来实现这个结果。Apache Commons 文本非常匹配，经过一些预处理后，Guava 可以帮助我们。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220726144623/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)