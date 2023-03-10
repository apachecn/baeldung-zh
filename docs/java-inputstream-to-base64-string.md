# 如何将 InputStream 转换为 Base64 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-inputstream-to-base64-string>

## 1.概观

Base64 是一种文本编码方案，为应用程序和平台之间的二进制数据提供可移植性。Base64 可用于在数据库字符串列中存储二进制数据，从而避免混乱的文件操作。当与[数据 URI 方案](https://web.archive.org/web/20220810181340/https://en.wikipedia.org/wiki/Data_URI_scheme)结合使用时，Base64 可用于在网页和电子邮件中嵌入图像，符合 HTML 和多用途互联网邮件扩展(MIME)标准。

在这个简短的教程中，我们将演示 Java 流 IO 函数和内置的 Java `Base64`类来**加载二进制数据作为`InputStream`，然后将其转换为`String`** 。

## 2.设置

让我们看看代码所需的依赖关系和测试数据。

### 2.1.属国

我们将使用 [Apache IOUtils](https://web.archive.org/web/20220810181340/https://mvnrepository.com/artifact/commons-io/commons-io/2.11.0) 库，通过将它的依赖项添加到我们的`pom.xml`来方便地访问测试数据文件:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency> 
```

### 2.2.测试数据

这里需要一个二进制测试数据文件。所以我们将把一个`logo.png`图像文件添加到我们的标准`src/test/resources`文件夹中。

## 3.将`InputStream`转换为 Base64 字符串

**Java 在`java.util.Base64 `类中内置了对 Base64 编码和解码的支持。因此，我们将使用那里的`static`方法来完成繁重的工作。**

`Base64.encode()`方法需要一个`byte`数组，我们的图像在一个文件中。因此，我们需要首先将文件转换成一个`InputStream`，然后将流一个字节一个字节地读入一个数组。

我们使用 Apache `commons-io`包中的`IOUtils.toByteArray()`方法作为冗长的纯 Java 方法的一种方便的替代方法。

首先，我们将编写一个简单的方法来生成“穷人的”校验和:

```java
int calculateChecksum(byte[] bytes) {
    int checksum = 0; 
    for (int index = 0; index < bytes.length; index++) {
        checksum += bytes[index]; 
    }
    return checksum; 
} 
```

我们将使用它来比较两个数组，验证它们是否匹配。

接下来的几行打开文件，将其转换成一个字节数组，然后 Base64 编码成一个`String`:

```java
InputStream sourceStream  = getClass().getClassLoader().getResourceAsStream("logo.png");
byte[] sourceBytes = IOUtils.toByteArray(sourceStream);

String encodedString = Base64.getEncoder().encodeToString(sourceBytes); 
assertNotNull(encodedString); 
```

该字符串看起来像一组随机字符。事实上，它不是随机的，正如我们在验证步骤中看到的:

```java
byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
assertNotNull(decodedBytes);
assertTrue(decodedBytes.length == sourceBytes.length);
assertTrue(calculateChecksum(decodedBytes) == calculateChecksum(sourceBytes)); 
```

## 4.结论

在本文中，我们演示了将`InputStream`编码为 Base64 字符串，并成功地将该字符串解码回二进制数组。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220810181340/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2)