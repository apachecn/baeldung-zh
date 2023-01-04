# 在 Java 中替换字符串中特定索引处的字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-replace-character-at-index>

## 1.介绍

在这个快速教程中，我们将演示如何在 Java 中替换`String`中特定索引处的字符。

我们将给出四个简单方法的实现，这些方法采用原始的`String,` a 字符，以及我们需要替换它的索引。

## 2.使用字符数组

让我们从一个简单的方法开始，使用一个数组`char.`

这里的想法是将`String`转换为`char[]`，然后在给定的索引处分配新的`char`。最后，我们从该数组中构造所需的`String`。

```java
public String replaceCharUsingCharArray(String str, char ch, int index) {
    char[] chars = str.toCharArray();
    chars[index] = ch;
    return String.valueOf(chars);
}
```

这是一种低级的设计方法，给了我们很大的灵活性。

## 3.使用`substring`方法

更高级的方法是使用`String`类的 `substring()`方法。

它将创建一个新的`String`，将索引前的原始`String`的子串与索引后的原始`String`的新字符和子串连接起来:

```java
public String replaceChar(String str, char ch, int index) {
    return str.substring(0, index) + ch + str.substring(index+1);
} 
```

## 4.使用`StringBuilder`

我们可以用 [`StringBuilder`](/web/20220628063643/https://www.baeldung.com/java-string-builder-string-buffer) 得到同样的效果。我们可以使用方法`setCharAt():`替换特定索引处的字符

```java
public String replaceChar(String str, char ch, int index) {
    StringBuilder myString = new StringBuilder(str);
    myString.setCharAt(index, ch);
    return myString.toString();
}
```

## 5.结论

在本文中，我们关注了几种使用 Java `.`替换`String`中特定索引处的字符的方法

`String`实例是不可变的，所以我们需要创建一个新的字符串或者使用`StringBuilder `来给我们一些可变性。

像往常一样，上述教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628063643/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)