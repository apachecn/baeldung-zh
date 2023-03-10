# Java 中 LowerCase 和 toUpperCase 方法的字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-convert-case>

## 1.概观

在本教程中，我们将介绍 Java [`String`](/web/20221206034204/https://www.baeldung.com/java-string) 类中包含的`toUpperCase`和`toLowerCase`方法。

我们将从创建一个名为`name`的`String`开始:

```java
String name = "John Doe";
```

## 2.转换成大写

为了基于`name`创建一个新的大写字母`String`，我们调用`toUpperCase`方法:

```java
String uppercaseName = name.toUpperCase();
```

这导致`uppercaseName`具有值`“JOHN DOE”`:

```java
assertEquals("JOHN DOE", uppercaseName);
```

注意在 Java 中`Strings`是**不可变的**，调用`toUpperCase`会创建一个新的`String`。换句话说，`name`在调用`toUpperCase`时是不变的。

## 3.转换成小写

类似地，我们通过调用`toLowerCase`基于`name`创建一个新的小写字母`String`:

```java
String lowercaseName = name.toLowerCase();
```

这导致`lowercaseName`具有值`“john doe”`:

```java
assertEquals("john doe", lowercaseName);
```

正如`toUpperCase`，`toLowerCase`不会改变`name`的值。

## 4.使用区域设置更改大小写

此外，通过向`toUpperCase`和`toLowerCase`方法提供一个 [`Locale`](/web/20221206034204/https://www.baeldung.com/java-8-localization#localization) ，我们可以使用特定于地区的规则来改变`String`的大小写。

例如，我们可以提供一个`Locale`来大写一个土耳其语`i` (Unicode `0069` ) `:`

```java
Locale TURKISH = new Locale("tr");
System.out.println("\u0069".toUpperCase());
System.out.println("\u0069".toUpperCase(TURKISH));
```

因此，这导致大写`I`和带点的大写`I`:

```java
I
İ
```

我们可以使用以下断言来验证这一点:

```java
assertEquals("\u0049", "\u0069".toUpperCase());
assertEquals("\u0130", "\u0069".toUpperCase(TURKISH));
```

同样，我们可以使用土耳其语`I` (Unicode `0049`)对`toLowerCase`做同样的事情:

```java
System.out.println("\u0049".toLowerCase());
System.out.println("\u0049".toLowerCase(TURKISH));
```

因此，这导致小写`i`和小写无点`i`:

```java
i
ı
```

我们可以使用以下断言来验证这一点:

```java
assertEquals("\u0069", "\u0049".toLowerCase());
assertEquals("\u0131", "\u0049".toLowerCase(TURKISH));
```

## 5.结论

总之，Java `String `类包含了用于改变`String`大小写的`toUpperCase`和`toLowerCase`方法。如果需要，在改变`String.`的大小写时，可以提供一个`Locale`来提供特定于地区的规则

本文的源代码，包括示例，可以在 GitHub 的[中找到。](https://web.archive.org/web/20221206034204/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)