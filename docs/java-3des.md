# Java 中的 3DES

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-3des>

## 1。简介

3DES 或[三重数据加密算法](https://web.archive.org/web/20220628120017/https://en.wikipedia.org/wiki/Triple_DES)是一种对称密钥块密码，它对每个数据块应用三次 DES 密码算法。

在本教程中，我们将学习如何在 Java 中**创建 3DES 密钥并使用它们来加密和解密`String`和文件**。

## 2。生成密钥

生成 3DES 密钥需要几个步骤。首先，我们需要生成一个用于加密-解密过程的密钥。在我们的例子中，我们将使用由随机数和字母构成的 24 字节密钥:

```java
byte[] secretKey = "9mng65v8jf4lxn93nabf981m".getBytes();
```

注意**秘密密钥不应该公开共享**。

现在，我们将把我们的密钥包装在`SecretKeySpec`中，并结合一个选择的算法:

```java
SecretKeySpec secretKeySpec = new SecretKeySpec(secretKey, "TripleDES");
```

在我们的例子中，我们使用的是`TripleDES`，它是 [Java 安全标准算法](https://web.archive.org/web/20220628120017/https://docs.oracle.com/en/java/javase/11/docs/specs/security/standard-names.html#algorithmparameters-algorithms)中的一种。

另一个我们应该提前生成的项目是键的[初始化向量](https://web.archive.org/web/20220628120017/https://en.wikipedia.org/wiki/Initialization_vector)。我们将使用由随机数和字母组成的 8 字节数组:

```java
byte[] iv = "a76nb5h9".getBytes();
```

然后，我们将它包装在`IvParameterSpec`类中:

```java
IvParameterSpec ivSpec = new IvParameterSpec(iv);
```

## 3.加密`String` s

我们现在准备加密简单的`String`值。让我们首先定义一个我们将使用的`String`:

```java
String secretMessage = "Baeldung secret message";
```

接下来，**我们需要一个 [`Cipher`](/web/20220628120017/https://www.baeldung.com/java-cipher-class) 对象，用加密模式、密钥和我们之前生成的初始化向量**进行初始化:

```java
Cipher encryptCipher = Cipher.getInstance("TripleDES/CBC/PKCS5Padding");
encryptCipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, ivSpec);
```

注意，我们使用的是带有 [CBC](https://web.archive.org/web/20220628120017/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation) 和 [PKCS#5 填充方案](https://web.archive.org/web/20220628120017/https://en.wikipedia.org/wiki/Padding_(cryptography))的`TripleDES`算法。

**使用`Cipher`，我们可以运行`doFinal`方法来加密我们的消息**。注意，它只适用于`byte`数组，所以我们需要首先转换我们的`String`:

```java
byte[] secretMessagesBytes = secretMessage.getBytes(StandardCharsets.UTF_8);
byte[] encryptedMessageBytes = encryptCipher.doFinal(secretMessagesBytes);
```

现在，我们的消息成功加密了。如果我们想将它存储在数据库中，或者通过一个 REST API 发送，用 Base64 字母表对它进行编码会更方便:

```java
String encodedMessage = Base64.getEncoder().encodeToString(encryptedMessageBytes);
```

Base64 编码使邮件可读性更强，也更容易处理。

## 4.解密`Strings`

现在，让我们看看如何逆转加密过程，将消息解密为原始形式。为此，**我们将需要一个新的`Cipher`实例，但是这一次，我们将在解密模式**中初始化它:

```java
Cipher decryptCipher = Cipher.getInstance("TripleDES/CBC/PKCS5Padding");
decryptCipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivSpec);
```

接下来，我们将运行`doFinal`方法:

```java
byte[] decryptedMessageBytes = decryptCipher.doFinal(encryptedMessageBytes);
```

现在，我们将结果解码成一个`String`变量:

```java
String decryptedMessage = new String(decryptedMessageBytes, StandardCharsets.UTF_8);
```

最后，我们可以通过将结果与初始值进行比较来验证结果，以确保解密过程正确执行:

```java
Assertions.assertEquals(secretMessage, decryptedMessage);
```

## 5.使用文件

我们也可以加密整个文件。例如，让我们创建一个包含一些文本内容的临时文件:

```java
String originalContent = "Secret Baeldung message";
Path tempFile = Files.createTempFile("temp", "txt");
writeString(tempFile, originalContent);
```

接下来，让我们将其内容转换成一个单字节数组:

```java
byte[] fileBytes = Files.readAllBytes(tempFile);
```

现在，我们可以像使用`String`一样使用加密密码:

```java
Cipher encryptCipher = Cipher.getInstance("TripleDES/CBC/PKCS5Padding");
encryptCipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, ivSpec);
byte[] encryptedFileBytes = encryptCipher.doFinal(fileBytes);
```

最后，让我们用新的加密数据覆盖文件内容:

```java
try (FileOutputStream stream = new FileOutputStream(tempFile.toFile())) {
    stream.write(encryptedFileBytes);
}
```

解密过程看起来非常相似。唯一的区别是在解密模式下初始化的密码:

```java
encryptedFileBytes = Files.readAllBytes(tempFile);
Cipher decryptCipher = Cipher.getInstance("TripleDES/CBC/PKCS5Padding");
decryptCipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivSpec);
byte[] decryptedFileBytes = decryptCipher.doFinal(encryptedFileBytes);
```

让我们再次覆盖文件内容，这一次是用解密的数据:

```java
try (FileOutputStream stream = new FileOutputStream(tempFile.toFile())) {
    stream.write(decryptedFileBytes);
}
```

作为最后一步，我们可以验证文件内容是否与原始值匹配:

```java
String fileContent = readString(tempFile);
Assertions.assertEquals(originalContent, fileContent);
```

## 6。总结

在本文中，我们学习了如何用 Java 创建 3DES 密钥，以及如何用它来加密和解密文件。

和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20220628120017/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-algorithms)