# 仅在分隔符第一次出现时拆分字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-split-string-first-delimiter>

## 1.概观

在本教程中，我们将学习如何使用两种方法，仅在分隔符第一次出现时[分割](/web/20220525133501/https://www.baeldung.com/java-split-string)Java`String`。

## 2.问题陈述

假设我们有一个文本文件，其中每一行都是由两部分组成的字符串，左边部分表示一个人的名字，右边部分表示他们的问候:

```java
Roberto "I wish you a bug-free day!"
Daniele "Have a great day!"
Jonas "Good bye!"
```

随后，我们想从每一行中得到这个人的名字。

我们可以看到两个部分都被一个" "(空格)隔开，就像右边部分的其他单词一样。因此，我们的分隔符将是空格字符。

## 3.使用`split()` 方法

来自`String`类的 [`split()`](https://web.archive.org/web/20220525133501/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#split(java.lang.String)) 实例方法根据提供的正则表达式分割字符串。此外，我们可以使用它的一个重载变量来获得所需的第一次出现。

我们可以提供一个 [`limit`](https://web.archive.org/web/20220525133501/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#split(java.lang.String,int)) 作为`split()`方法的参数，来指定我们想要应用模式的次数，从而指定结果数组中令牌的最大数量。例如，如果我们将`limit`设为`n` ( `n` > 0)，则意味着该模式最多应用`n-1 `次。

这里，我们将使用 space(" ")作为正则表达式，在第一次出现空格时拆分`String`。

因此，我们可以使用重载的`split()`方法将每一行标记为两部分:

```java
public String getFirstWordUsingSplit(String input) {
    String[] tokens = input.split(" ", 2);
    return tokens[0];
}
```

因此，如果我们将示例中的第一行作为输入传递给这个方法，它将返回“Roberto”。

**然而，如果输入的`String`只有一个单词或者其中没有空格，上述方法将简单地返回相同的`String`。**

让我们来测试一下:

```java
assertEquals("Roberto", getFirstWordUsingSplit("Roberto \"I wish you a bug-free day\""));
assertEquals("StringWithNoSpace", getFirstWordUsingSplit("StringWithNoSpace"));
```

## 4.使用 `substring()`方法

`String `类的`[substring()](https://web.archive.org/web/20220525133501/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#substring(int,int))`方法返回`String.`的子串。这是一个重载方法，其中一个重载版本接受`index` 并返回字符串中的所有字符，直到给定的索引。

让我们把`substring()`和 [`indexOf()`](https://web.archive.org/web/20220525133501/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#indexOf(java.lang.String)) 结合起来解决同一个问题`.`

首先，我们将得到第一个空格字符的索引。然后，我们将得到子串，直到这个索引，这将是我们的结果，人的名字:

```java
public String getFirstWordUsingSubString(String input) {
    return input.substring(0, input.indexOf(" "));
}
```

如果我们像以前一样传递相同的输入`String`，我们的方法将返回`String` “Roberto”。

**然而，如果输入`String`不包含任何空格，那么这个方法将抛出`StringIndexOutOfBoundsException`。**如果没有找到匹配，`indexOf() `方法返回-1。

为了避免这种异常，我们可以修改上面的方法:

```java
public String getFirstWordUsingSubString(String input) {
    int index = input.contains(" ") ? input.indexOf(" ") : 0;
    return input.substring(0, index);
}
```

现在，如果我们传递一个没有空格的`String`给这个方法，我们将得到一个空的`String`作为回报。

让我们来测试一下:

```java
assertEquals("Roberto", getFirstWordUsingSubString("Roberto \"I wish you a bug-free day\""));
assertEquals("", getFirstWordUsingSubString("StringWithNoSpace"));
```

## 5.结论

在本文中，我们已经看到了两种仅在 Java 中第一次出现分隔符时分割`String`的方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220525133501/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)