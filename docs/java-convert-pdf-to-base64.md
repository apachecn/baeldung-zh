# Java 将 PDF 转换为 Base64

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-pdf-to-base64>

## 1.概观

在这个简短的教程中，我们将看到**如何使用 Java 8 和 Apache Commons 编解码器**对 PDF 文件进行 Base64 编码和解码。

但首先，让我们快速浏览一下 Base64 的基础知识。

## 2.Base64 基础

当通过网络发送数据时，我们需要以二进制格式发送。但是如果我们只发送 0 和 1，不同的传输层协议可能会有不同的解释，我们的数据可能会在传输中被破坏。

因此，**为了在传输二进制数据时具有可移植性和通用标准，Base64 出现了**。

由于发送者和接收者都理解并同意使用该标准，我们的数据丢失或被误解的可能性大大降低。

现在让我们来看看将它应用到 PDF 的几种方法。

## 3.使用 Java 8 进行转换

从 Java 8 开始，我们有一个实用程序 [`java.util.Base64`](https://web.archive.org/web/20221129003820/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Base64.html) 为 Base64 编码方案提供编码器和解码器。它支持在 [RFC 4648](https://web.archive.org/web/20221129003820/https://www.ietf.org/rfc/rfc4648.txt) 和 [RFC 2045](https://web.archive.org/web/20221129003820/https://www.ietf.org/rfc/rfc2045.txt) 中指定的基本类型、URL 安全类型和 MIME 类型。

### 3.1.编码

要将 PDF 转换成 Base64，我们首先需要获取它的字节数，然后**通过`java.util.Base64.Encoder`的`encode`方法**传递它:

```java
byte[] inFileBytes = Files.readAllBytes(Paths.get(IN_FILE)); 
byte[] encoded = java.util.Base64.getEncoder().encode(inFileBytes);
```

这里，`IN_FILE`是我们输入 PDF 的路径。

### 3.2.流式编码

对于较大的文件或内存有限的系统，**使用流执行编码比读取内存中的所有数据更有效**。让我们看看如何实现这一点:

```java
try (OutputStream os = java.util.Base64.getEncoder().wrap(new FileOutputStream(OUT_FILE));
  FileInputStream fis = new FileInputStream(IN_FILE)) {
    byte[] bytes = new byte[1024];
    int read;
    while ((read = fis.read(bytes)) > -1) {
        os.write(bytes, 0, read);
    }
}
```

这里，`IN_FILE`是我们的输入 PDF 的路径，`OUT_FILE`是包含 Base64 编码文档的文件的路径。我们不是将整个 PDF 读入内存，然后在内存中对整个文档进行编码，而是一次读取 1Kb 的数据，并通过编码器将数据传递到`OutputStream`。

### 3.3.解码

在接收端，我们得到编码文件。

所以我们现在需要**解码它以获得我们的原始字节，并将它们写入`FileOutputStream`以获得解码的 PDF** :

```java
byte[] decoded = java.util.Base64.getDecoder().decode(encoded);

FileOutputStream fos = new FileOutputStream(OUT_FILE);
fos.write(decoded);
fos.flush();
fos.close();
```

这里，`OUT_FILE`是我们要创建的 PDF 的路径。

## 4.使用 Apache Commons 进行转换

接下来，我们将使用 Apache Commons 编解码器包来实现同样的功能。它基于 [RFC 2045](https://web.archive.org/web/20221129003820/https://www.ietf.org/rfc/rfc2045.txt) ，早于我们之前讨论的 Java 8 实现。因此，当我们需要支持多个 JDK 版本(包括遗留版本)或供应商时，这作为第三方 API 就派上了用场。

### 4.1.专家

为了能够使用 Apache 库，我们需要向我们的`pom.xml`添加一个依赖项:

```java
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.14</version>
</dependency> 
```

以上最新版本可以在 [Maven Central](https://web.archive.org/web/20221129003820/https://search.maven.org/search?q=g:commons-codec) 上找到。

### 4.2.编码

步骤与 Java 8 相同，只是这次我们将原始字节传递给 [`org.apache.commons.codec.binary.Base64`](https://web.archive.org/web/20221129003820/https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/Base64.html) 类的`encodeBase64`方法:

```java
byte[] inFileBytes = Files.readAllBytes(Paths.get(IN_FILE));
byte[] encoded = org.apache.commons.codec.binary.Base64.encodeBase64(inFileBytes); 
```

### 4.3.流式编码

此库不支持流式编码。

### 4.4.解码

同样，我们简单地调用`decodeBase64`方法并将结果写入文件:

```java
byte[] decoded = org.apache.commons.codec.binary.Base64.decodeBase64(encoded);

FileOutputStream fos = new FileOutputStream(OUT_FILE);
fos.write(decoded);
fos.flush();
fos.close(); 
```

## 5.测试

现在我们将使用一个简单的 JUnit 测试来测试我们的编码和解码:

```java
public class EncodeDecodeUnitTest {

    private static final String IN_FILE = // path to file to be encoded from;
    private static final String OUT_FILE = // path to file to be decoded into;
    private static byte[] inFileBytes;

    @BeforeClass
    public static void fileToByteArray() throws IOException {
        inFileBytes = Files.readAllBytes(Paths.get(IN_FILE));
    }

    @Test
    public void givenJavaBase64_whenEncoded_thenDecodedOK() throws IOException {
        byte[] encoded = java.util.Base64.getEncoder().encode(inFileBytes);
        byte[] decoded = java.util.Base64.getDecoder().decode(encoded);
        writeToFile(OUT_FILE, decoded);

        assertNotEquals(encoded.length, decoded.length);
        assertEquals(inFileBytes.length, decoded.length);
        assertArrayEquals(decoded, inFileBytes);
    }

    @Test
    public void givenJavaBase64_whenEncodedStream_thenDecodedStreamOK() throws IOException {
        try (OutputStream os = java.util.Base64.getEncoder().wrap(new FileOutputStream(OUT_FILE));
          FileInputStream fis = new FileInputStream(IN_FILE)) {
            byte[] bytes = new byte[1024];
            int read;
            while ((read = fis.read(bytes)) > -1) {
                os.write(bytes, 0, read);
            }
        }

        byte[] encoded = java.util.Base64.getEncoder().encode(inFileBytes);
        byte[] encodedOnDisk = Files.readAllBytes(Paths.get(OUT_FILE));
        assertArrayEquals(encoded, encodedOnDisk);

        byte[] decoded = java.util.Base64.getDecoder().decode(encoded);
        byte[] decodedOnDisk = java.util.Base64.getDecoder().decode(encodedOnDisk);
        assertArrayEquals(decoded, decodedOnDisk);
    }

    @Test
    public void givenApacheCommons_givenJavaBase64_whenEncoded_thenDecodedOK() throws IOException {
        byte[] encoded = org.apache.commons.codec.binary.Base64.encodeBase64(inFileBytes);
        byte[] decoded = org.apache.commons.codec.binary.Base64.decodeBase64(encoded);

        writeToFile(OUT_FILE, decoded);

        assertNotEquals(encoded.length, decoded.length);
        assertEquals(inFileBytes.length, decoded.length);

        assertArrayEquals(decoded, inFileBytes);
    }

    private void writeToFile(String fileName, byte[] bytes) throws IOException {
        FileOutputStream fos = new FileOutputStream(fileName);
        fos.write(bytes);
        fos.flush();
        fos.close();
    }
}
```

正如我们所看到的，我们首先在一个`@BeforeClass`方法中读取输入字节，并且在我们的两个`@Test`方法中，验证了:

*   `encoded`和`decoded`字节数组长度不同
*   `inFileBytes`和`decoded`字节数组长度相同，内容相同

当然，我们也可以打开我们创建的解码后的 PDF 文件，看到内容与我们输入的文件相同。

## 6.结论

在这个快速教程中，我们了解了更多关于 [Java 的 Base64 实用程序](/web/20221129003820/https://www.baeldung.com/java-base64-encode-and-decode)。

我们还看到了使用 Java 8 和 Apache Commons 编解码器将 PDF 转换为 Base64 的代码示例。有趣的是，JDK 的实现比 Apache 快得多。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129003820/https://github.com/eugenp/tutorials/tree/master/pdf)