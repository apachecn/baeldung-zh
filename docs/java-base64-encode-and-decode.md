# Java Base64 编码和解码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-base64-encode-and-decode>

## 1。概述

在本教程中，我们将探索在 Java 中提供 Base64 编码和解码功能的各种实用程序。

我们将主要展示新的 Java 8 APIs。此外，我们使用 Apache Commons 的实用程序 API。

## 延伸阅读:

## [Java URL 编码/解码指南](/web/20221017115849/https://www.baeldung.com/java-url-encoding-decoding)

The article discusses URL encoding in Java, some pitfalls, and how to avoid them.[Read more](/web/20221017115849/https://www.baeldung.com/java-url-encoding-decoding) →

## [Java 中的 SHA-256 和 SHA3-256 散列法](/web/20221017115849/https://www.baeldung.com/sha-256-hashing-java)

A quick and practical guide to SHA-256 hashing in Java[Read more](/web/20221017115849/https://www.baeldung.com/sha-256-hashing-java) →

## [Spring Security 5 中的新密码存储](/web/20221017115849/https://www.baeldung.com/spring-security-5-password-storage)

A quick guide to understanding password encryption in Spring Security 5 and migrating to better encryption algorithms.[Read more](/web/20221017115849/https://www.baeldung.com/spring-security-5-password-storage) →

## 2。Java 8 for Base 64

**Java 8 终于通过`java.util.Base64`实用程序类为标准 API 添加了 Base64 功能**。

让我们先来看一个基本的编码器流程。

### 2.1。Java 8 Basic Base64

基本编码器保持简单，按原样编码输入，没有任何行分隔。

编码器将输入映射到`A-Za-z0-9+/`字符集中的一组字符。

先把**编码成一个简单的`String`** :

```java
String originalInput = "test input";
String encodedString = Base64.getEncoder().encodeToString(originalInput.getBytes()); 
```

请注意我们如何通过简单的`getEncoder()`实用程序方法检索完整的编码器 API。

现在让我们将`String`解码回原始形式:

```java
byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
String decodedString = new String(decodedBytes);
```

### 2.2。无填充的 Java 8 Base64 编码

在 Base64 编码中，输出编码的长度`String`必须是 3 的倍数。编码器根据需要在输出的末尾添加一个或两个填充字符(`=`)，以满足这一要求。

解码时，解码器会丢弃这些额外的填充字符。要深入探究 Base64 中的填充，请查看[这篇关于堆栈溢出](https://web.archive.org/web/20221017115849/https://stackoverflow.com/a/18518605/370481)的详细回答。

有时候，我们需要**跳过输出**的填充。例如，产生的`String`将永远不会被解码回来。所以，我们可以简单地选择**编码而不填充**:

```java
String encodedString = 
  Base64.getEncoder().withoutPadding().encodeToString(originalInput.getBytes());
```

### 2.3。Java 8 URL 编码

URL 编码与基本编码器非常相似。此外，它使用安全的 Base64 字母表的 URL 和文件名。此外，它不添加任何分隔线:

```java
String originalUrl = "https://www.google.co.nz/?gfe_rd=cr&ei;=dzbFV&gws;_rd=ssl#q=java";
String encodedUrl = Base64.getUrlEncoder().encodeToString(originalURL.getBytes()); 
```

解码也是如此。`getUrlDecoder()`实用程序方法返回一个`java.util.Base64.Decoder`。所以，我们用它来解码 URL:

```java
byte[] decodedBytes = Base64.getUrlDecoder().decode(encodedUrl);
String decodedUrl = new String(decodedBytes); 
```

### 2.4。Java 8 MIME 编码

让我们从生成一些要编码的基本 MIME 输入开始:

```java
private static StringBuilder getMimeBuffer() {
    StringBuilder buffer = new StringBuilder();
    for (int count = 0; count < 10; ++count) {
        buffer.append(UUID.randomUUID().toString());
    }
    return buffer;
}
```

MIME 编码器使用基本字母表生成 Base64 编码的输出。但是，这种格式是 MIME 友好的。

输出的每行不超过 76 个字符。此外，它以回车结束，后跟换行符(`\r\n`):

```java
StringBuilder buffer = getMimeBuffer();
byte[] encodedAsBytes = buffer.toString().getBytes();
String encodedMime = Base64.getMimeEncoder().encodeToString(encodedAsBytes);
```

在解码过程中，我们可以使用返回一个`java.util.Base64.Decoder`的`getMimeDecoder()`方法:

```java
byte[] decodedBytes = Base64.getMimeDecoder().decode(encodedMime);
String decodedMime = new String(decodedBytes); 
```

## 3。使用 Apache Commons 代码进行编码/解码

首先，我们需要定义`pom.xml`中的`[commons-codec](https://web.archive.org/web/20221017115849/https://search.maven.org/classic/#search|gav|1|g%3A%22commons-codec%22%20AND%20a%3A%22commons-codec%22)`依赖关系:

```java
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

主要的 API 是`org.apache.commons.codec.binary.Base64`类。我们可以用不同的构造函数初始化它:

*   `Base64(boolean urlSafe)`通过控制 URL 安全模式(开或关)创建 Base64 API。
*   `Base64(int lineLength)`在 URL 不安全模式下创建 Base64 API，并控制行的长度(默认为 76)。
*   `Base64(int lineLength, byte[] lineSeparator)`通过接受额外的行分隔符来创建 Base64 API，默认情况下，该分隔符是 CRLF ("\r\n ")。

一旦创建了 Base64 API，编码和解码都非常简单:

```java
String originalInput = "test input";
Base64 base64 = new Base64();
String encodedString = new String(base64.encode(originalInput.getBytes())); 
```

此外，`Base64` 类的`decode()`方法返回解码后的字符串:

```java
String decodedString = new String(base64.decode(encodedString.getBytes())); 
```

另一个选择是**使用`Base64`** 的静态 API，而不是创建一个实例:

```java
String originalInput = "test input";
String encodedString = new String(Base64.encodeBase64(originalInput.getBytes()));
String decodedString = new String(Base64.decodeBase64(encodedString.getBytes()));
```

## 4。将`String`转换为`byte`数组

有时候，我们需要将一个`String`转换成一个`byte[]`。最简单的办法就是使用`String`的`getBytes()`方法:

```java
String originalInput = "test input";
byte[] result = originalInput.getBytes();

assertEquals(originalInput.length(), result.length);
```

我们也可以提供编码，不依赖默认编码。因此，它依赖于系统:

```java
String originalInput = "test input";
byte[] result = originalInput.getBytes(StandardCharsets.UTF_16);

assertTrue(originalInput.length() < result.length);
```

**如果我们的`String`是`Base64`编码的，我们可以使用`the Base64`解码器**:

```java
String originalInput = "dGVzdCBpbnB1dA==";
byte[] result = Base64.getDecoder().decode(originalInput);

assertEquals("test input", new String(result));
```

**我们也可以使用`DatatypeConverter parseBase64Binary()`方法**:

```java
String originalInput = "dGVzdCBpbnB1dA==";
byte[] result = DatatypeConverter.parseBase64Binary(originalInput);

assertEquals("test input", new String(result));
```

**最后，我们可以使用`DatatypeConverter.parseHexBinary` 方法**将十六进制的`String`转换为`byte[]`:

```java
String originalInput = "7465737420696E707574";
byte[] result = DatatypeConverter.parseHexBinary(originalInput);

assertEquals("test input", new String(result));
```

## 5。结论

本文解释了如何用 Java 进行 Base64 编码和解码的基础知识。我们使用了 Java 8 和 Apache Commons 中引入的新 API。

最后，还有一些其他的 API 提供类似的功能:`java.xml.bind.DataTypeConverter`与`printHexBinary`和`parseBase64Binary`。

代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221017115849/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)