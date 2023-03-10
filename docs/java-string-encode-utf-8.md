# 用 Java 将字符串编码成 UTF-8

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-encode-utf-8>

## 1.概观

在 Java 中处理`String`时，我们有时需要将它们编码成特定的字符集。

## 延伸阅读:

## [字符编码指南](/web/20221109093624/https://www.baeldung.com/java-char-encoding)

Explore character encoding in Java and learn about common pitfalls.[Read more](/web/20221109093624/https://www.baeldung.com/java-char-encoding) →

## [Java URL 编码/解码指南](/web/20221109093624/https://www.baeldung.com/java-url-encoding-decoding)

The article discusses URL encoding in Java, some pitfalls, and how to avoid them.[Read more](/web/20221109093624/https://www.baeldung.com/java-url-encoding-decoding) →

## [Java Base64 编解码](/web/20221109093624/https://www.baeldung.com/java-base64-encode-and-decode)

How to do Base64 encoding and decoding in Java, using the new APIs introduced in Java 8 as well as Apache Commons.[Read more](/web/20221109093624/https://www.baeldung.com/java-base64-encode-and-decode) →

本教程是一个实践指南，展示了用不同的方式**将`String`编码成 UTF-8 字符集。**

要更深入地了解技术，请参阅我们的[字符编码指南](/web/20221109093624/https://www.baeldung.com/java-char-encoding)。

## 2.定义问题

为了展示 Java 编码，我们将使用德语`String`“Entwickeln Sie MIT vergnügen”:

```java
String germanString = "Entwickeln Sie mit Vergnügen";
byte[] germanBytes = germanString.getBytes();

String asciiEncodedString = new String(germanBytes, StandardCharsets.US_ASCII);

assertNotEquals(asciiEncodedString, germanString);
```

这个使用 US_ASCII 编码的`String`给了我们一个值“Entwickeln Sie mit Vergn？因为**不理解非 ASCII ü字符。**

但是当我们将一个使用所有英文字符的 ASCII 编码的`String`转换为 UTF-8 时，我们会得到相同的字符串:

```java
String englishString = "Develop with pleasure";
byte[] englishBytes = englishString.getBytes();

String asciiEncondedEnglishString = new String(englishBytes, StandardCharsets.US_ASCII);

assertEquals(asciiEncondedEnglishString, englishString);
```

让我们看看当我们使用 UTF 8 编码时会发生什么。

## 3.用核心 Java 编码

让我们从核心库开始。

在 Java 中,`String`是不可变的，这意味着我们不能改变一个`String`字符编码。为了实现我们想要的，**我们需要复制`String`的字节，然后用期望的编码创建一个新的。**

首先，我们获取`String`字节，然后使用检索到的字节和所需的字符集创建一个新的字节:

```java
String rawString = "Entwickeln Sie mit Vergnügen";
byte[] bytes = rawString.getBytes(StandardCharsets.UTF_8);

String utf8EncodedString = new String(bytes, StandardCharsets.UTF_8);

assertEquals(rawString, utf8EncodedString);
```

## 4.用 Java 7 编码`StandardCharsets`

或者，我们可以使用 **Java 7** 中引入的`StandardCharsets`类**对`String`进行编码。**

首先，我们将把`String`解码成字节，其次，我们将把`String`编码成 UTF-8:

```java
String rawString = "Entwickeln Sie mit Vergnügen";
ByteBuffer buffer = StandardCharsets.UTF_8.encode(rawString); 

String utf8EncodedString = StandardCharsets.UTF_8.decode(buffer).toString();

assertEquals(rawString, utf8EncodedString);
```

## 5.使用 Commons-Codec 编码

除了使用核心 Java，我们还可以使用 [Apache Commons Codec](https://web.archive.org/web/20221109093624/https://commons.apache.org/proper/commons-codec/) 来获得相同的结果。

Apache Commons Codec 是一个方便的包，包含各种格式的简单编码器和解码器。

首先，我们从项目配置开始。

使用 Maven 时，我们必须将[的`commons-codec`依赖项](https://web.archive.org/web/20221109093624/https://search.maven.org/search?q=g:commons-codec%20AND%20a:commons-codec)添加到我们的 *pom.xml* 中:

```java
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.14</version>
</dependency>
```

那么，在我们的例子中，最有趣的类是 [`StringUtils`](https://web.archive.org/web/20221109093624/https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/StringUtils.html) ，它提供了编码`String` s 的方法

使用这个类，获得 UTF-8 编码的`String`非常简单:

```java
String rawString = "Entwickeln Sie mit Vergnügen"; 
byte[] bytes = StringUtils.getBytesUtf8(rawString);

String utf8EncodedString = StringUtils.newStringUtf8(bytes);

assertEquals(rawString, utf8EncodedString);
```

## 6.结论

将一个*字符串*编码成 UTF-8 并不困难，但是并不那么直观。本文介绍了三种方法，使用核心 Java 或 Apache Commons 编解码器。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221109093624/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)