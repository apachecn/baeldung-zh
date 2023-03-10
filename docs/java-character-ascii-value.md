# 获取 Java 中字符的 ASCII 值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-character-ascii-value>

## 1.概观

在这个简短的教程中，我们将看到如何在 Java 中获取字符的 ASCII 值。

## 2.使用铸造

要获得字符的 [ASCII](/web/20221115042544/https://www.baeldung.com/cs/ascii-code) 值，我们可以简单地将我们的`char`转换为`int`:

```java
char c = 'a';
System.out.println((int) c);
```

这是输出结果:

```java
97
```

记住 Java 中的`char`可以是 Unicode 字符。所以**我们的字符必须是一个 ASCII 字符**才能得到正确的 ASCII 数值。

## 3.字符串中的字符

如果我们的`char`在一个`String`中，我们可以使用`charAt()`方法来检索它:

```java
String str = "abc";
char c = str.charAt(0);
System.out.println((int) c);
```

## 4.结论

总之，我们已经学习了如何在 Java 中获取字符的 ASCII 值。