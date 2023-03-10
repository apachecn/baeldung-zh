# Java 中的 MD5 哈希算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-md5>

## 1。概述

MD5 是广泛使用的加密散列函数，它产生 128 位的散列。

在本文中，我们将看到使用各种 Java 库创建 MD5 散列的不同方法。

## 2。MD5 使用`MessageDigest`类

在`java.security.MessageDigest`类中有一个[散列](/web/20221010092922/https://www.baeldung.com/cs/hashing)功能。想法是首先用您想用作参数的算法类型实例化`MessageDigest`:

```java
MessageDigest.getInstance(String Algorithm)
```

然后继续使用`update()` 函数更新消息摘要:

```java
public void update(byte [] input)
```

当你在读一个长文件时，上面的函数可以被多次调用。然后最后我们需要使用`digest()`函数来生成一个散列码:

```java
public byte[] digest()
```

下面是一个示例，它为密码生成一个哈希，然后对其进行验证:

```java
@Test
public void givenPassword_whenHashing_thenVerifying() 
  throws NoSuchAlgorithmException {
    String hash = "35454B055CC325EA1AF2126E27707052";
    String password = "ILoveJava";

    MessageDigest md = MessageDigest.getInstance("MD5");
    md.update(password.getBytes());
    byte[] digest = md.digest();
    String myHash = DatatypeConverter
      .printHexBinary(digest).toUpperCase();

    assertThat(myHash.equals(hash)).isTrue();
}
```

同样，我们也可以验证文件的校验和:

```java
@Test
public void givenFile_generatingChecksum_thenVerifying() 
  throws NoSuchAlgorithmException, IOException {
    String filename = "src/test/resources/test_md5.txt";
    String checksum = "5EB63BBBE01EEED093CB22BB8F5ACDC3";

    MessageDigest md = MessageDigest.getInstance("MD5");
    md.update(Files.readAllBytes(Paths.get(filename)));
    byte[] digest = md.digest();
    String myChecksum = DatatypeConverter
      .printHexBinary(digest).toUpperCase();

    assertThat(myChecksum.equals(checksum)).isTrue();
}
```

我们需要注意的是， **MessageDigest 不是线程安全的**。因此，我们应该为每个线程使用一个新的实例。

## 3。使用 Apache Commons 的 MD5

类使事情变得简单多了。

让我们看一个散列和验证密码的例子:

```java
@Test
public void givenPassword_whenHashingUsingCommons_thenVerifying()  {
    String hash = "35454B055CC325EA1AF2126E27707052";
    String password = "ILoveJava";

    String md5Hex = DigestUtils
      .md5Hex(password).toUpperCase();

    assertThat(md5Hex.equals(hash)).isTrue();
}
```

## 4。使用番石榴的 MD5

下面是使用`com.google.common.io.Files.hash`生成 MD5 校验和的另一种方法:

```java
@Test
public void givenFile_whenChecksumUsingGuava_thenVerifying() 
  throws IOException {
    String filename = "src/test/resources/test_md5.txt";
    String checksum = "5EB63BBBE01EEED093CB22BB8F5ACDC3";

    HashCode hash = com.google.common.io.Files
      .hash(new File(filename), Hashing.md5());
    String myChecksum = hash.toString()
      .toUpperCase();

    assertThat(myChecksum.equals(checksum)).isTrue();
}
```

请注意，`Hashing.md5`已被弃用。然而，正如官方文件所指出的，原因是建议出于安全考虑一般不要使用 MD5。这意味着我们仍然可以使用这种方法，例如，如果我们需要与需要 MD5 的遗留系统集成。否则，我们最好考虑更安全的选择，比如 SHA-256 和 T4。

## 5。结论

Java API 和其他第三方 API(如 Apache commons 和 Guava)有不同的方法来生成 MD5 散列。根据项目的需求和项目需要遵循的依赖关系做出明智的选择。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20221010092922/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2)