# 在 Java 中通过索引从字符串中获取字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-character-at-position>

## 1.介绍

`[String](/web/20220816192509/https://www.baeldung.com/java-string)` 类的`charAt()`方法返回`String`给定位置的字符。这是一个有用的方法，从 Java 语言的 1.0 版本开始就有了。

在本教程中，我们将通过一些例子来探索这种方法的用法。我们还将学习如何让角色处于一个`String.`的位置

## 2.`charAt()`法

让我们看看来自`String`类的方法签名:

```java
public char charAt(int index) {...}
```

**该方法返回输入参数中指定的索引处的`char`。索引范围从 0(第一个字符)到字符串的总长度–1(最后一个字符)。**

现在，让我们看一个例子:

```java
String sample = "abcdefg";
Assert.assertEquals('d', sample.charAt(3)); 
```

在这种情况下，结果是字符串的第四个字符，即字符“d”。

## 3.预期异常

**如果参数`index`为负或者等于或大于字符串**的长度，则抛出运行时异常`IndexOutOfBoundsException`

```java
String sample = "abcdefg";
assertThrows(IndexOutOfBoundsException.class, () -> sample.charAt(-1));
assertThrows(IndexOutOfBoundsException.class, () -> sample.charAt(sample.length())); 
```

## 4.得到`Character`作为`String`

正如我们前面提到的，`charAt()`方法返回一个`char`。通常，我们需要一个`String`来代替。

有不同的方法将结果转换成一个`String`。让我们假设下面的`String`文字适用于所有的例子:

```java
String sample = "abcdefg";
```

### 4.1.使用`Character.toString()` 方法

我们可以用`Character.toString()` 方法包装`charAt()`的结果:

```java
assertEquals("a", Character.toString(sample.charAt(0))); 
```

### 4.2.使用`String.valueOf()` 方法

最后，我们可以使用静态方法`String`。`valueOf()`:

```java
assertEquals("a", String.valueOf(sample.charAt(0))); 
```

## 5.结论

在本文中，我们学习了如何使用`charAt()`方法在`String`的给定位置获取字符。我们也看到了在使用它时会发生什么异常，以及一些不同的方式来获得作为`String`的角色。

和往常一样，所有的片段都可以在 Github 上找到[。](https://web.archive.org/web/20220816192509/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)