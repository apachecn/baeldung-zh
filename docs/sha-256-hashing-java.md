# Java 中的 SHA-256 和 SHA3-256 哈希算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sha-256-hashing-java>

## 1。概述

SHA(安全哈希算法)是一种流行的加密哈希函数。加密哈希可用于为文本或数据文件制作签名。

在本教程中，让我们看看如何使用各种 Java 库执行 SHA-256 和 SHA3-256 哈希运算。

[SHA-256](https://web.archive.org/web/20220730222905/https://en.wikipedia.org/wiki/SHA-2) 算法生成一个几乎唯一的、固定大小的 256 位(32 字节)散列。这是一个单向函数，因此结果无法解密回原始值。

目前，SHA-2 散列法被广泛使用，因为它被认为是加密领域中最安全的散列算法。

SHA-3 是继 SHA-2 之后最新的安全哈希标准。与 SHA-2 相比，SHA-3 提供了一种不同的方法来生成唯一的单向散列，并且在某些硬件实现上可以快得多。与 SHA-256 类似，SHA3-256 是 SHA-3 中的 256 位固定长度算法。

[NIST](https://web.archive.org/web/20220730222905/https://csrc.nist.gov/projects/hash-functions)2015 年发布了 SHA-3，所以暂时还没有 SHA-2 那么多的 SHA-3 库。直到 JDK 9，SHA-3 算法才在内置的默认提供程序中可用。

现在让我们从 SHA-256 开始。

## 延伸阅读:

## [使用 Java-LSH 的 Java 中的位置敏感散列法](/web/20220730222905/https://www.baeldung.com/locality-sensitive-hashing)

A quick and practical guide to applying the Locality-Sensitive Hashing algorithm in Java using the java-lsh library.[Read more](/web/20220730222905/https://www.baeldung.com/locality-sensitive-hashing) →

## [Java 中的 MD5 散列法](/web/20220730222905/https://www.baeldung.com/java-md5)

A quick writeup show you how to deal with MD5 hashing in Java.[Read more](/web/20220730222905/https://www.baeldung.com/java-md5) →

## [Java HashSet 指南](/web/20220730222905/https://www.baeldung.com/java-hashset)

A quick but comprehensive introduction to HashSet in Java.[Read more](/web/20220730222905/https://www.baeldung.com/java-hashset) →

## 2。`MessageDigest`Java 中的类

Java 为 SHA-256 哈希提供了内置的`MessageDigest`类:

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] encodedhash = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
```

但是，这里我们必须使用一个自定义的字节到十六进制的转换器来获得十六进制的哈希值:

```java
private static String bytesToHex(byte[] hash) {
    StringBuilder hexString = new StringBuilder(2 * hash.length);
    for (int i = 0; i < hash.length; i++) {
        String hex = Integer.toHexString(0xff & hash[i]);
        if(hex.length() == 1) {
            hexString.append('0');
        }
        hexString.append(hex);
    }
    return hexString.toString();
}
```

我们需要知道 **MessageDigest 不是线程安全的。因此，我们应该为每个线程使用一个新的实例。**

## 3。番石榴库

Google Guava 库还提供了一个用于散列的实用程序类。

首先，让我们定义依赖关系:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

接下来，我们可以使用番石榴散列字符串:

```java
String sha256hex = Hashing.sha256()
  .hashString(originalString, StandardCharsets.UTF_8)
  .toString();
```

## 4 .apache commons 编解码器〔t1〕

同样，我们也可以使用 Apache Commons 编解码器:

```java
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.11</version>
</dependency>
```

下面是支持 SHA-256 散列的实用程序类，名为`DigestUtils`:

```java
String sha256hex = DigestUtils.sha256Hex(originalString);
```

## 5。充气城堡图书馆

### 5.1。Maven 依赖关系

```java
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.60</version>
</dependency>
```

### 5.2。使用弹力城堡库进行哈希运算

Bouncy Castle API 提供了一个实用程序类，用于将十六进制数据转换为字节，以及将字节转换为十六进制数据。

然而，我们需要首先使用内置的 Java API 来填充一个摘要:

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
String sha256hex = new String(Hex.encode(hash));
```

## 6。SHA3-256

现在让我们继续讨论 SHA3-256。Java 中的 SHA3-256 散列法与 SHA-256 并没有太大的不同。

### 6.1。`MessageDigest`Java 中的类

[从 JDK 9](https://web.archive.org/web/20220730222905/https://docs.oracle.com/javase/9/security/oracleproviders.htm#JSSEC-GUID-3A80CC46-91E1-4E47-AC51-CB7B782CEA7D) 开始，我们可以简单使用内置的 SHA3-256 算法:

```java
final MessageDigest digest = MessageDigest.getInstance("SHA3-256");
final byte[] hashbytes = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = bytesToHex(hashbytes);
```

### 6.2 .apache commons 编解码器〔t1〕

Apache Commons 编解码器为`MessageDigest`类提供了一个方便的`DigestUtils`包装器。

这个库从版本 [1.11](https://web.archive.org/web/20220730222905/https://search.maven.org/artifact/commons-codec/commons-codec/1.11/jar) 开始支持 SHA3-256，它[也需要 JDK 9+](https://web.archive.org/web/20220730222905/https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/digest/MessageDigestAlgorithms.html#SHA3_256) :

```java
String sha3Hex = new DigestUtils("SHA3-256").digestAsHex(originalString);
```

### 6.3。Keccak-256

Keccak-256 是另一种流行的 SHA3-256 哈希算法。目前，它是标准 SHA3-256 的替代产品。Keccak-256 提供了与标准 SHA3-256 相同的安全级别，它与 SHA3-256 的区别仅在于填充规则。它已经在几个区块链项目中使用，如 [Monero](https://web.archive.org/web/20220730222905/https://monerodocs.org/cryptography/keccak-256/) 。

同样，我们需要导入 Bouncy Castle 库来使用 Keccak-256 哈希:

```java
Security.addProvider(new BouncyCastleProvider());
final MessageDigest digest = MessageDigest.getInstance("Keccak-256");
final byte[] encodedhash = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = bytesToHex(encodedhash);
```

我们还可以利用 Bouncy Castle API 来进行哈希运算:

```java
Keccak.Digest256 digest256 = new Keccak.Digest256();
byte[] hashbytes = digest256.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = new String(Hex.encode(hashbytes));
```

## 7。结论

在这篇简短的文章中，我们了解了使用内置库和第三方库在 Java 中实现 SHA-256 和 SHA3-256 哈希的几种方法。

示例的源代码可以在 [GitHub 项目](https://web.archive.org/web/20220730222905/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2)中找到。