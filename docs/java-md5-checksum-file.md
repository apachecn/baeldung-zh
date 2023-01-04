# 用 Java 为文件生成 MD5 校验和

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-md5-checksum-file>

## 1.概观

校验和是用来唯一标识文件的字符序列。它最常用于验证文件的副本是否与原件相同。

在这个简短的教程中，我们将看到如何用 Java 为文件生成 MD5 校验和。

## 2.使用`MessageDigest`类

我们可以很容易地使用`java.security`包中的`MessageDigest`类来生成文件的 MD5 校验和:

```java
byte[] data = Files.readAllBytes(Paths.get(filePath));
byte[] hash = MessageDigest.getInstance("MD5").digest(data);
String checksum = new BigInteger(1, hash).toString(16);
```

## 3.使用 Apache Commons 编解码器

我们也可以使用来自 [Apache Commons Codec](https://web.archive.org/web/20221102190719/https://search.maven.org/search?q=g:commons-codec) 库的`DigestUtils`类来实现同样的目标。

让我们给我们的`pom.xml`文件添加一个依赖项:

```java
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

现在，我们简单地使用`md5Hex()`方法来获得文件的 MD5 校验和:

```java
try (InputStream is = Files.newInputStream(Paths.get(filePath))) {
    String checksum = DigestUtils.md5Hex(is);
    // ....
}
```

让我们不要忘记使用 [try-with-resources](/web/20221102190719/https://www.baeldung.com/java-try-with-resources) ，这样我们就不用担心关闭流了。

## 4.用番石榴

最后，我们可以使用[番石榴](/web/20221102190719/https://www.baeldung.com/guava-guide)的`ByteSource`对象的`hash()`方法:

```java
File file = new File(filePath);
ByteSource byteSource = com.google.common.io.Files.asByteSource(file);
HashCode hc = byteSource.hash(Hashing.md5());
String checksum = hc.toString();
```

## 5.结论

在这个快速教程中，我们展示了用 Java 为文件生成 MD5 校验和的不同方法。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221102190719/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)