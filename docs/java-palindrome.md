# 在 Java 中检查一个字符串是否是回文

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-palindrome>

## 1。简介

在本文中，我们将看到如何使用 Java 来检查给定的`String`是否是回文。

**回文是一个单词、短语、数字或其他字符序列，向后读与向前读相同**，如“夫人”或“赛车”。

## 2。解决方案

在接下来的几节中，我们将看看检查给定的`String`是否是回文的各种方法。

### 2.1。一种简单的方法

我们可以同时开始向前和向后迭代给定的`string`，一次一个字符。如果存在匹配，则循环继续；否则，循环退出:

```java
public boolean isPalindrome(String text) {
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    int length = clean.length();
    int forward = 0;
    int backward = length - 1;
    while (backward > forward) {
        char forwardChar = clean.charAt(forward++);
        char backwardChar = clean.charAt(backward--);
        if (forwardChar != backwardChar)
            return false;
    }
    return true;
}
```

### 2.2。反转琴弦

有几个不同的实现适合这个用例:我们可以在检查回文时使用来自`StringBuilder` 和`StringBuffer` 类的 API 方法，或者我们可以在没有这些类的情况下反转`String`。

让我们先看看没有助手 API 的代码实现:

```java
public boolean isPalindromeReverseTheString(String text) {
    StringBuilder reverse = new StringBuilder();
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    char[] plain = clean.toCharArray();
    for (int i = plain.length - 1; i >= 0; i--) {
        reverse.append(plain[i]);
    }
    return (reverse.toString()).equals(clean);
}
```

在上面的代码片段中，我们简单地从最后一个字符开始迭代给定的`String` ，并将每个字符追加到下一个字符，一直到第一个字符，从而反转给定的`String.`

最后，我们测试给定的`String` 和反转的`String.`之间的相等性

使用 API 方法可以实现相同的行为。

让我们来看一个快速演示:

```java
public boolean isPalindromeUsingStringBuilder(String text) {
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    StringBuilder plain = new StringBuilder(clean);
    StringBuilder reverse = plain.reverse();
    return (reverse.toString()).equals(clean);
}

public boolean isPalindromeUsingStringBuffer(String text) {
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    StringBuffer plain = new StringBuffer(clean);
    StringBuffer reverse = plain.reverse();
    return (reverse.toString()).equals(clean);
}
```

在代码片段中，我们从`StringBuilder`和`StringBuffer` API 调用`reverse()` 方法来反转给定的`String`并测试相等性。

### 2.3。使用`Stream` API

我们还可以使用一个`IntStream`来提供一个解决方案:

```java
public boolean isPalindromeUsingIntStream(String text) {
    String temp  = text.replaceAll("\\s+", "").toLowerCase();
    return IntStream.range(0, temp.length() / 2)
      .noneMatch(i -> temp.charAt(i) != temp.charAt(temp.length() - i - 1));
}
```

在上面的代码片段中，我们验证了来自`String`两端的字符对都不满足`Predicate`条件。

### 2.4。使用递归

递归是解决这类问题的一种非常流行的方法。在演示的例子中，我们递归地迭代给定的`String`，并测试它是否是一个回文:

```java
public boolean isPalindromeRecursive(String text){
    String clean = text.replaceAll("\\s+", "").toLowerCase();
    return recursivePalindrome(clean,0,clean.length()-1);
}

private boolean recursivePalindrome(String text, int forward, int backward) {
    if (forward == backward) {
        return true;
    }
    if ((text.charAt(forward)) != (text.charAt(backward))) {
        return false;
    }
    if (forward < backward + 1) {
        return recursivePalindrome(text, forward + 1, backward - 1);
    }

    return true;
}
```

## 3。结论

在这个快速教程中，我们看到了如何确定给定的`String`是否是回文。

和往常一样，本文的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626081044/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms)