# Java 17 中的 HexFormat 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hexformat>

## 1.介绍

在 Java 中，我们通常编写自己的方法来处理字节和十六进制字符串之间的转换。然而，Java 17 引入了`java.util.HexFormat`，一个实用程序类**支持原始类型、字节数组或字符数组到十六进制字符串的转换，反之亦然**。

在本教程中，我们将探索如何使用`HexFormat`并演示它提供的功能。

## 2.Java 17 之前处理十六进制字符串

[十六进制计数系统](https://web.archive.org/web/20220525121925/https://en.wikipedia.org/wiki/Hexadecimal)使用 16 为基数来表示数字。这意味着它由 16 个符号组成，通常符号 0-9 代表从 0 到 9 的值，A-F 代表从 10 到 15 的值。

这是表示长二进制值的流行选择，因为与 1 和 0 的二进制字符串相比，它更容易推理。

当我们需要在十六进制字符串和字节数组之间进行转换时，开发人员通常会使用`String.format()`编写他们自己的方法来完成这项工作。

这是一种简单且易于理解的实现方式，但往往效率低下:

```java
public static String byteArrayToHex(byte[] a) {
    StringBuilder sb = new StringBuilder(a.length * 2);
    for (byte b: a) {
       sb.append(String.format("%02x", b));
    }
    return sb.toString();
}
```

另一个流行的解决方案是使用 [Apache Commons Codec](https://web.archive.org/web/20220525121925/https://commons.apache.org/proper/commons-codec/) 库，其中包含一个`Hex`实用程序类:

```java
String foo = "I am a string";
byte[] bytes = foo.getBytes();
Hex.encodeHexString(bytes);
```

我们的另一个教程解释了手动执行这个转换的不同方法。

## 3.`HexFormat`Java 17 中的用法

`HexFormat`可以在 Java 17 标准库中找到，可以**处理** **字节和十六进制字符串**之间的转换。它还支持几个格式化选项。

### 3.1.创建一个`HexFormat`

我们如何创建一个新的`HexFormat` **实例取决于我们是否需要分隔符支持**。`HexFormat`是线程安全的，所以一个实例可以在多个线程中使用。

`HexFormat.of() `是最常见的用例，当我们不关心分隔符支持时使用:

```java
HexFormat hexFormat = HexFormat.of();
```

`HexFormat.ofDelimiter(“:”)` 可用于分隔符支持，本例使用冒号作为分隔符:

```java
HexFormat hexFormat = HexFormat.ofDelimiter(":");
```

### 3.2.字符串格式

`HexFormat`允许我们**向现有的`HexFormat`对象添加前缀、后缀和分隔符格式选项。**我们可以使用这些来控制正在被解析或生成的`String`的格式。

下面是三者结合使用的一个例子:

```java
HexFormat hexFormat = HexFormat.of().withPrefix("[").withSuffix("]").withDelimiter(", ");
assertEquals("[48], [0c], [11]", hexFormat.formatHex(new byte[] {72, 12, 17}));
```

在本例中，我们使用简单的`of()`方法创建对象，然后使用`withDelimiter().`添加分隔符

### 3.3.字节和十六进制字符串转换

现在我们已经看到了如何创建一个`HexFormat`实例，让我们回顾一下如何执行转换。

我们将使用创建实例的简单方法:

```java
HexFormat hexFormat = HexFormat.of();
```

接下来，让我们用它来把一个`String`转换成`byte[]`:

```java
byte[] hexBytes = hexFormat.parseHex("ABCDEF0123456789");
assertArrayEquals(new byte[] { -85, -51, -17, 1, 35, 69, 103, -119 }, hexBytes);
```

然后再回来:

```java
String bytesAsString = hexFormat.formatHex(new byte[] { -85, -51, -17, 1, 35, 69, 103, -119});
assertEquals("ABCDEF0123456789", bytesAsString);
```

### 3.4.原始类型到十六进制字符串的转换

`HexFormat`还支持原语类型到十六进制字符串的转换:

```java
String fromByte = hexFormat.toHexDigits((byte) 64);
assertEquals("40", fromByte);

String fromLong = hexFormat.toHexDigits(1234_5678_9012_3456L);
assertEquals("000462d53c8abac0", fromLong);
```

### 3.5.大写和小写输出

如示例所示，`HexFormat`的默认行为是产生一个小写的十六进制值。**我们可以通过在创建我们的`HexFormat`实例**时调用`withUpperCase()`来改变这种行为:

```java
upperCaseHexFormat = HexFormat.of().withUpperCase();
```

尽管小写是默认行为，但也存在一个`withLowerCase()` 方法。这有助于使我们的代码自文档化，并对其他开发人员显而易见。

## 4.结论

Java 17 中`HexFormat`的引入解决了我们传统上在执行字节和十六进制字符串之间的转换时面临的许多问题。

在本文中，我们已经讨论了最常见的用例，但是`HexFormat`还支持更多的特殊功能。例如，有更多的转换方法和管理一个完整字节的上半部分和下半部分的能力。

在 [Java 17 文档](https://web.archive.org/web/20220525121925/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HexFormat.html)中可以找到`HexFormat`的官方文档。

像往常一样，我们在本文中展示的例子是在 GitHub 上的[。](https://web.archive.org/web/20220525121925/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-17)