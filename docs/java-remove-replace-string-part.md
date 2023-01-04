# 在 Java 中删除或替换字符串的一部分

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-replace-string-part>

## 1。概述

在本教程中，我们将会看到在 Java 中移除或替换部分`String`的各种方法。

我们将探索使用`String` API 移除和/或替换子串，然后使用`StringBuilder` API，最后使用 Apache Commons 库的`StringUtils`类。

作为奖励，**我们还将研究使用 String API 和 Apache Commons `RegExUtils`类**替换一个`exact`单词。

## 2。`String` API

替换子串的最简单直接的方法之一是使用`String class.`的`replace, replaceAll `或`replaceFirst`

`replace()`方法有两个参数——目标和替换文本:

```java
String master = "Hello World Baeldung!";
String target = "Baeldung";
String replacement = "Java";
String processed = master.replace(target, replacement);
assertTrue(processed.contains(replacement));
assertFalse(processed.contains(target));
```

上面的代码片段将产生以下输出:

```java
Hello World Java!
```

如果在选择目标时需要正则表达式，那么应该选择`replaceAll()`或`replaceFirst()`方法。顾名思义，`replaceAll() `将替换所有匹配的事件，而`replaceFirst()`将替换第一个匹配的事件:

```java
String master2 = "Welcome to Baeldung, Hello World Baeldung";
String regexTarget = "(Baeldung)$";
String processed2 = master2.replaceAll(regexTarget, replacement);
assertTrue(processed2.endsWith("Java"));
```

`processed2`的值将为:

```java
Welcome to Baeldung, Hello World Java
```

这是因为在上面给出的所有例子中，作为`regexTarget`提供的正则表达式将只匹配最后出现的`Baeldung. ` **，我们可以使用一个空的替换，它将有效地从`master`** 中删除一个`target`。

## 3。`StringBuilder` API

我们也可以使用`StringBuilder` 类`. `在 Java 中操作文本，这里的两个方法是`delete()`和`replace()`。

我们可以从现有的`String`构造一个`StringBuilder `的实例，然后使用上面提到的方法根据需要执行`String`操作:

```java
String master = "Hello World Baeldung!";
String target = "Baeldung";
String replacement = "Java";

int startIndex = master.indexOf(target);
int stopIndex = startIndex + target.length();

StringBuilder builder = new StringBuilder(master);
```

现在我们可以用`delete()`删除`target`:

```java
builder.delete(startIndex, stopIndex);
assertFalse(builder.toString().contains(target));
```

我们也可以使用`replace()`来更新主文件:

```java
builder.replace(startIndex, stopIndex, replacement);
assertTrue(builder.toString().contains(replacement));
```

使用`StringBuilder `和`String` API 的一个明显区别是，我们必须自己获取`target`T3 的开始和停止索引。

## 4。`StringUtils`阶级

我们将考虑的另一种方法是 Apache Commons 库。

首先，让我们将所需的依赖项添加到项目中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

这个库的最新版本可以在这里找到。

`StringUtils `类有替换`String`子串的方法:

```java
String master = "Hello World Baeldung!";
String target = "Baeldung";
String replacement = "Java";

String processed = StringUtils.replace(master, target, replacement);
assertTrue(processed.contains(replacement));
```

有一个重载的`replace()`变量，它接受一个整数`max`参数，该参数决定要替换的出现次数。**如果不考虑大小写，我们也可以使用`replaceIgnoreCase()`**:

```java
String master2 = "Hello World Baeldung!";
String target2 = "baeldung";
String processed2 = StringUtils.replaceIgnoreCase(master2, target2, replacement);
assertFalse(processed2.contains(target));
```

## 5。替换精确单词

在最后这个例子中，**我们将学习如何替换`String`** 中的一个精确单词。

执行这种替换的直接方法是使用带有单词边界的正则表达式。

**字界正则表达式是`\b`** 。将所需的单词包含在这个正则表达式中只会匹配精确的匹配项。

首先，让我们看看如何将这个正则表达式与字符串 API 一起使用:

```java
String sentence = "A car is not the same as a carriage, and some planes can carry cars inside them!";
String regexTarget = "\\bcar\\b";
String exactWordReplaced = sentence.replaceAll(regexTarget, "truck");
```

`exactWordReplaced`字符串包含:

```java
"A truck is not the same as a carriage, and some planes can carry cars inside them!"
```

只有精确的单词会被替换。注意[在 Java 中使用正则表达式时，反斜杠总是需要被转义](/web/20220709111401/https://www.baeldung.com/java-regexp-escape-char)。

进行这种替换的另一种方法是使用 Apache Commons 库中的`RegExUtils`类，正如我们在上一节中看到的，可以将它作为一个依赖项添加:

```java
String regexTarget = "\\bcar\\b";
String exactWordReplaced = RegExUtils.replaceAll(sentence, regexTarget, "truck"); 
```

虽然这两种方法会产生相同的结果，但决定应该使用哪一种将取决于我们的特定场景。

## 6。结论

总之，我们已经探索了多种在 Java 中删除和替换子串的方法。应用的最佳方法仍然很大程度上取决于当前的情况和背景。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220709111401/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)