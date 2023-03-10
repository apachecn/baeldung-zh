# Java 中的数字签名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-digital-signature>

## 1。概述

在本教程中，我们将学习**数字签名机制，以及如何使用 [Java 加密体系结构(JCA)](https://web.archive.org/web/20220628143444/https://docs.oracle.com/en/java/javase/11/security/java-cryptography-architecture-jca-reference-guide.html)** 来实现它。我们将探索`KeyPair, MessageDigest, Cipher, KeyStore, Certificate,`和`Signature`JCA API。

我们将首先了解什么是数字签名，如何生成密钥对，以及如何从证书颁发机构(CA)认证公钥。之后，我们将看到如何使用低级和高级 JCA API 实现数字签名。

## 2。什么是数字签名？

### 2.1.数字签名定义

数字签名是一种技术，用于确保:

*   完整性:消息在传输过程中没有被更改
*   真实性:消息的作者确实是他们声称的那个人
*   不可否认性:消息的作者不能否认他们是消息的来源

### 2.2.发送带有数字签名的邮件

从技术上讲，****数字签名是消息**的加密散列(摘要、校验和)。这意味着我们从消息中生成一个散列，并根据选择的算法用私钥加密它。**

 **然后，消息、加密的散列、相应的公钥和算法都被发送出去。这被归类为带有数字签名的消息。

### 2.3.接收和检查数字签名

为了检查数字签名，消息接收方从接收到的消息中生成新的散列，使用公钥解密接收到的加密散列，并比较它们。如果它们匹配，数字签名就被认为是经过验证的。

我们应该注意，我们只加密消息散列，而不是消息本身。换句话说，数字签名并不试图对消息保密。我们的数字签名只能证明消息在传输过程中没有被修改。

**当签名被验证时，我们确信只有私钥的拥有者可能是消息**的作者。

## 3。数字证书和公钥标识

**证书是将身份与给定公钥相关联的文档。**证书由称为认证机构(CA)的第三方实体签名。

我们知道，如果我们用公开密钥解密的散列与实际的散列相匹配，那么消息就被签名了。然而，我们如何知道公钥真的来自正确的实体呢？这可以通过使用数字证书来解决。

数字证书包含一个公钥，它本身由另一个实体签名。该实体的签名本身可以被另一个实体验证，依此类推。我们最终拥有了我们所谓的证书链。每个顶级实体证明下一个实体的公钥。最顶级的实体是自签名的，也就是说他的公钥是用他自己的私钥签名的。

X.509 是最常用的证书格式，它以二进制格式(DER)或文本格式(PEM)提供。JCA 已经通过`X509Certificate`类为此提供了一个实现。

## 4.密钥对管理

由于数字签名使用一个私钥和一个公钥，我们将使用 JCA 类`PrivateKey`和`PublicKey`分别对消息进行签名和检查。

### 4.1.得到一对钥匙

为了创建一个私钥和公钥的密钥对`,`，我们将使用 Java [`keytool`](https://web.archive.org/web/20220628143444/https://docs.oracle.com/en/java/javase/11/tools/keytool.html) 。

让我们使用`genkeypair`命令生成一个密钥对:

```java
keytool -genkeypair -alias senderKeyPair -keyalg RSA -keysize 2048 \
  -dname "CN=Baeldung" -validity 365 -storetype PKCS12 \
  -keystore sender_keystore.p12 -storepass changeit
```

这为我们创建了一个私钥及其对应的公钥。公钥被包装到 X.509 自签名证书中，该证书又被包装到单元素证书链中。我们将证书链和私有密钥存储在密钥库文件`sender_keystore.p12`中，我们可以使用[密钥库 API](/web/20220628143444/https://www.baeldung.com/java-keystore) 对其进行处理。

这里，我们使用了 PKCS12 密钥存储格式，因为它是标准的，并且推荐使用 Java 专有的 JKS 格式。此外，我们应该记住密码和别名，因为我们将在下一小节加载密钥库文件时使用它们。

### 4.2.正在加载用于签名的私钥

为了给消息签名，我们需要一个`PrivateKey.`的实例

使用`KeyStore` API 和之前的密钥库文件`sender_keystore.p12,`，我们可以得到一个`PrivateKey`对象:

```java
KeyStore keyStore = KeyStore.getInstance("PKCS12");
keyStore.load(new FileInputStream("sender_keystore.p12"), "changeit");
PrivateKey privateKey = 
  (PrivateKey) keyStore.getKey("senderKeyPair", "changeit");
```

### 4.3.发布公钥

在发布公钥之前，我们必须首先决定是使用自签名证书还是 CA 签名证书。

**当使用自签名证书时，我们只需要从密钥库文件中导出它。**我们可以用`exportcert`命令来完成:

```java
keytool -exportcert -alias senderKeyPair -storetype PKCS12 \
  -keystore sender_keystore.p12 -file \
  sender_certificate.cer -rfc -storepass changeit
```

否则，**如果我们要使用 CA 签名的证书，那么我们需要创建一个证书签名请求(CSR)** 。我们用`certreq`命令来做这件事:

```java
keytool -certreq -alias senderKeyPair -storetype PKCS12 \
  -keystore sender_keystore.p12 -file -rfc \
  -storepass changeit > sender_certificate.csr
```

CSR 文件`sender_certificate.csr,`随后被发送到认证机构以进行签名。完成后，我们将收到一个包装在 X.509 证书中的签名公钥，可以是二进制(DER)或文本(PEM)格式。这里，我们对 PEM 格式使用了`rfc`选项。

我们从 CA`sender_certificate.cer,`收到的公钥现在已经由 CA 签名，可以供客户端使用。

### 4.4.加载用于验证的公钥

有了对公钥的访问权，接收者可以使用`importcert`命令将其加载到他们的密钥库中:

```java
keytool -importcert -alias receiverKeyPair -storetype PKCS12 \
  -keystore receiver_keystore.p12 -file \
  sender_certificate.cer -rfc -storepass changeit
```

像以前一样使用`KeyStore` API，我们可以得到一个`PublicKey`实例:

```java
KeyStore keyStore = KeyStore.getInstance("PKCS12");
keyStore.load(new FileInputStream("receiver_keytore.p12"), "changeit");
Certificate certificate = keyStore.getCertificate("receiverKeyPair");
PublicKey publicKey = certificate.getPublicKey();
```

现在我们在发送方有了一个`PrivateKey`实例，在接收方有了一个`PublicKey`实例，我们可以开始签名和验证的过程了。

## 5。具有`MessageDigest`和`Cipher`类的数字签名

正如我们已经看到的，数字签名是基于散列和加密的。

通常我们用`MessageDigest`类配合 [SHA](/web/20220628143444/https://www.baeldung.com/sha-256-hashing-java) 或者 [MD5](/web/20220628143444/https://www.baeldung.com/java-md5) 进行哈希，用 [`Cipher`](/web/20220628143444/https://www.baeldung.com/java-cipher-class) 类进行加密。

现在，让我们开始实现数字签名机制。

### 5.1。生成消息哈希

消息可以是字符串、文件或任何其他数据。让我们来看一个简单文件的内容:

```java
byte[] messageBytes = Files.readAllBytes(Paths.get("message.txt"));
```

现在，使用`MessageDigest,`让我们使用`digest`方法生成一个散列:

```java
MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] messageHash = md.digest(messageBytes);
```

这里，我们使用了最常用的 SHA-256 算法。其他替代方案有 MD5、SHA-384 和 SHA-512。

### 5.2。加密生成的哈希

要加密消息，我们需要一种算法和一个私钥。这里我们将使用 RSA 算法。DSA 算法是另一种选择。

让我们创建一个`Cipher`实例，并初始化它以进行加密。然后我们将调用`doFinal()`方法来加密之前散列的消息:

```java
Cipher cipher = Cipher.getInstance("RSA");
cipher.init(Cipher.ENCRYPT_MODE, privateKey);
byte[] digitalSignature = cipher.doFinal(messageHash);
```

签名可以保存到文件中，以便以后发送:

```java
Files.write(Paths.get("digital_signature_1"), digitalSignature);
```

此时，消息、数字签名、公钥和算法都被发送，接收者可以使用这些信息来验证消息的完整性。

### 5.3。验证签名

当我们收到消息时，我们必须验证它的签名。为此，我们解密接收到的加密哈希，并将其与我们对接收到的消息制作的哈希进行比较。

让我们阅读收到的数字签名:

```java
byte[] encryptedMessageHash = 
  Files.readAllBytes(Paths.get("digital_signature_1"));
```

为了解密，我们创建了一个`Cipher`实例。然后我们调用`doFinal`方法:

```java
Cipher cipher = Cipher.getInstance("RSA");
cipher.init(Cipher.DECRYPT_MODE, publicKey);
byte[] decryptedMessageHash = cipher.doFinal(encryptedMessageHash);
```

接下来，我们从收到的消息中生成一个新的消息哈希:

```java
byte[] messageBytes = Files.readAllBytes(Paths.get("message.txt"));

MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] newMessageHash = md.digest(messageBytes);
```

最后，我们检查新生成的消息哈希是否与解密的消息哈希匹配:

```java
boolean isCorrect = Arrays.equals(decryptedMessageHash, newMessageHash);
```

在这个例子中，我们使用了文本文件`message.txt`来模拟我们想要发送的消息，或者我们已经收到的消息的正文的位置。通常情况下，我们希望在收到签名的同时收到邮件。

## 6。使用*签名的数字签名*类

到目前为止，我们已经使用低级 API 构建了我们自己的数字签名验证过程。这有助于我们理解它是如何工作的，并允许我们定制它。

然而，JCA 已经以`Signature`类的形式提供了一个专用的 API。

### 6.1。签署消息

为了开始签名过程，我们首先创建一个`Signature`类的实例。为此，我们需要一个签名算法。然后我们用我们的私钥初始化`Signature`:

```java
Signature signature = Signature.getInstance("SHA256withRSA");
signature.initSign(privateKey);
```

我们选择的签名算法，本例中的`SHA256withRSA` 是哈希算法和加密算法的组合。其他备选方案包括`SHA1withRSA`、`SHA1withDSA`、`MD5withRSA`等。

接下来，我们对消息的字节数组进行签名:

```java
byte[] messageBytes = Files.readAllBytes(Paths.get("message.txt"));

signature.update(messageBytes);
byte[] digitalSignature = signature.sign();
```

我们可以将签名保存到文件中，以便以后传输:

```java
Files.write(Paths.get("digital_signature_2"), digitalSignature);
```

### 6.2。验证签名

为了验证收到的签名，我们再次创建一个`Signature`实例:

```java
Signature signature = Signature.getInstance("SHA256withRSA");
```

接下来，我们通过调用`initVerify`方法初始化用于验证的`Signature`对象，该方法采用一个公钥:

```java
signature.initVerify(publicKey);
```

然后，我们需要通过调用`update`方法将接收到的消息字节添加到签名对象中:

```java
byte[] messageBytes = Files.readAllBytes(Paths.get("message.txt"));

signature.update(messageBytes);
```

最后，我们可以通过调用`verify`方法来检查签名:

```java
boolean isCorrect = signature.verify(receivedSignature);
```

## 7。结论

在本文中，我们首先了解了数字签名是如何工作的，以及如何为数字证书建立信任。然后，我们使用 Java 加密体系结构中的`MessageDigest,` `Cipher,`和`Signature`类实现了一个数字签名。

我们详细了解了如何使用私钥签署数据，以及如何使用公钥验证签名。

和往常一样，这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628143444/https://github.com/eugenp/tutorials/tree/master/libraries-security)**