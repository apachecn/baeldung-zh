# Java 中的 RSA

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rsa>

## 1.介绍

RSA，或者说[Rivest–sha mir–ad leman](https://web.archive.org/web/20221021020206/https://en.wikipedia.org/wiki/RSA_(cryptosystem))，是一种非对称加密算法。它与对称算法如 [DES](https://web.archive.org/web/20221021020206/https://en.wikipedia.org/wiki/Data_Encryption_Standard) 或 [AES](/web/20221021020206/https://www.baeldung.com/java-aes-encryption-decryption) 不同，它有两个密钥。我们可以与任何人共享的公钥用于加密数据。另一个是私人的，我们只为自己保留，用于解密数据

在本教程中，我们将学习如何在 Java 中生成、存储和使用 RSA 密钥。

## 2.生成 RSA 密钥对

在开始真正的加密之前，我们需要生成 RSA 密钥对。我们可以通过使用`java.security`包中的`KeyPairGenerator`很容易地做到这一点:

```
KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA");
generator.initialize(2048);
KeyPair pair = generator.generateKeyPair();
```

生成的密钥将具有 2048 位的大小。

接下来，我们可以提取私钥和公钥:

```
PrivateKey privateKey = pair.getPrivate();
PublicKey publicKey = pair.getPublic();
```

我们将使用公钥加密数据，使用私钥解密数据。

## 3.将密钥存储在文件中

将密钥对存储在内存中并不总是一个好的选择。大多数情况下，密钥会保持很长时间不变。在这种情况下，将它们存储在文件中更方便。

要在文件中保存一个密钥，我们可以使用`getEncoded`方法，该方法以其主要编码格式返回密钥内容:

```
try (FileOutputStream fos = new FileOutputStream("public.key")) {
    fos.write(publicKey.getEncoded());
}
```

要从文件中读取密钥，我们首先需要以字节数组的形式加载内容:

```
File publicKeyFile = new File("public.key");
byte[] publicKeyBytes = Files.readAllBytes(publicKeyFile.toPath());
```

然后使用`KeyFactory`来重新创建实际的实例:

```
KeyFactory keyFactory = KeyFactory.getInstance("RSA");
EncodedKeySpec publicKeySpec = new X509EncodedKeySpec(publicKeyBytes);
keyFactory.generatePublic(publicKeySpec);
```

关键字节内容需要用一个`EncodedKeySpec`类包装。在这里，我们使用的是代表默认算法的`X509EncodedKeySpec,`，用于保存文件的`Key::getEncoded`方法。

在这个例子中，我们只保存和读取了公钥文件。相同的步骤可用于处理私钥。

记住，尽可能安全地保存带有私钥的文件，并尽可能限制访问。未经授权的访问可能会带来安全问题。

## 4.使用字符串

现在，让我们看看如何加密和解密简单的字符串。首先，我们需要一些数据来处理:

```
String secretMessage = "Baeldung secret message";
```

其次，我们需要一个 [`Cipher`](/web/20221021020206/https://www.baeldung.com/java-cipher-class) 对象，用我们之前生成的公钥初始化加密:

```
Cipher encryptCipher = Cipher.getInstance("RSA");
encryptCipher.init(Cipher.ENCRYPT_MODE, publicKey);
```

准备就绪后，我们可以调用`doFinal`方法来加密我们的消息。注意，它只接受字节数组参数，所以我们需要在:

```
byte[] secretMessageBytes = secretMessage.getBytes(StandardCharsets.UTF_8);)
byte[] encryptedMessageBytes = encryptCipher.doFinal(secretMessageBytes);
```

现在，我们的信息被成功编码。如果我们想将它存储在数据库中或者通过 [REST API](/web/20221021020206/https://www.baeldung.com/rest-with-spring-series) 发送，那么用 Base64 字母表对其进行[编码会更方便:](/web/20221021020206/https://www.baeldung.com/java-base64-encode-and-decode)

```
String encodedMessage = Base64.getEncoder().encodeToString(encryptedMessageBytes);
```

这样，信息将更具可读性，也更容易处理。

现在，让我们看看如何将消息解密为其原始形式。为此，我们需要另一个`Cipher`实例。这一次，我们将使用解密模式和私钥对其进行初始化:

```
Cipher decryptCipher = Cipher.getInstance("RSA");
decryptCipher.init(Cipher.DECRYPT_MODE, privateKey);
```

我们将像前面一样用`doFinal`方法调用密码:

```
byte[] decryptedMessageBytes = decryptCipher.doFinal(encryptedMessageBytes);
String decryptedMessage = new String(decryptedMessageBytes, StandardCharsets.UTF_8);
```

最后，让我们验证加密-解密过程是否正确:

```
assertEquals(secretMessage, decryptedMessage);
```

## 5.使用文件

也可以加密整个文件。例如，让我们创建一个包含一些文本内容的临时文件:

```
Path tempFile = Files.createTempFile("temp", "txt");
Files.writeString(tempFile, "some secret message");
```

在开始加密之前，我们需要将其内容转换成一个字节数组:

```
byte[] fileBytes = Files.readAllBytes(tempFile);
```

现在，我们可以使用加密密码:

```
Cipher encryptCipher = Cipher.getInstance("RSA");
encryptCipher.init(Cipher.ENCRYPT_MODE, publicKey);
byte[] encryptedFileBytes = encryptCipher.doFinal(fileBytes);
```

最后，我们可以用新的加密内容覆盖它:

```
try (FileOutputStream stream = new FileOutputStream(tempFile.toFile())) {
    stream.write(encryptedFileBytes);
}
```

解密过程看起来非常相似。唯一的区别是在解密模式下用私钥初始化的密码:

```
byte[] encryptedFileBytes = Files.readAllBytes(tempFile);
Cipher decryptCipher = Cipher.getInstance("RSA");
decryptCipher.init(Cipher.DECRYPT_MODE, privateKey);
byte[] decryptedFileBytes = decryptCipher.doFinal(encryptedFileBytes);
try (FileOutputStream stream = new FileOutputStream(tempFile.toFile())) {
    stream.write(decryptedFileBytes);
}
```

作为最后一步，我们可以验证文件内容是否与原始值匹配:

```
String fileContent = Files.readString(tempFile);
Assertions.assertEquals("some secret message", fileContent);
```

## 6.摘要

在本文中，我们学习了如何用 Java 创建 RSA 密钥，以及如何使用它们来加密和解密消息和文件。和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20221021020206/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-algorithms)