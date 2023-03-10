# 如何读取 PEM 文件以获取公钥和私钥

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-pem-file-keys>

## 1.概观

在公钥密码中，也称为[非对称密码](/web/20221117030337/https://www.baeldung.com/cs/symmetric-vs-asymmetric-cryptography)，加密机制依赖于两个相关的密钥 一个公钥和一个私钥。公钥用于加密消息，而只有私钥的所有者才能解密消息。

在本教程中，我们将学习如何从 PEM 文件中读取公钥和私钥。

首先，我们将研究公钥加密的一些重要概念。然后我们将学习如何使用纯 Java 读取 PEM 文件。

最后，我们将探索作为替代方法的 [BouncyCastle](/web/20221117030337/https://www.baeldung.com/java-bouncy-castle) 库。

## 2.概念

开始之前，我们先讨论一些关键概念。

X.509 是定义公钥证书格式的标准。 所以这种格式描述了一个公钥，以及其他信息。

**DER** **是存储数据最流行的编码格式，如文件中的 X.509 证书和 PKCS8 私钥。**这是一种二进制编码，生成的内容无法用文本编辑器查看。

**PKCS8** **是存储私钥信息的标准语法。**可以选择使用对称算法对私钥进行加密。

这个标准不仅可以处理 RSA 私钥，还可以处理其他算法。pkcs 8 私钥通常通过 PEM 编码格式进行交换。

**PEM** **是一种 DER 证书的 base-64 编码机制。** PEM 还可以对其他类型的数据进行编码，例如公钥/私钥和证书请求。

PEM 文件还包含描述编码数据类型的页眉和页脚:

```java
-----BEGIN PUBLIC KEY-----
...Base64 encoding of the DER encoded certificate...
-----END PUBLIC KEY-----
```

## 3.使用纯 Java

### 3.1.从文件中读取 PEM 数据

让我们从读取 PEM 文件开始，并将它的内容存储到一个字符串:

```java
String key = new String(Files.readAllBytes(file.toPath()), Charset.defaultCharset());
```

### 3.2.从 PEM 字符串获取公钥

现在我们将构建一个实用方法，从 PEM 编码的字符串中获取公钥:

```java
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsjtGIk8SxD+OEiBpP2/T
JUAF0upwuKGMk6wH8Rwov88VvzJrVm2NCticTk5FUg+UG5r8JArrV4tJPRHQyvqK
wF4NiksuvOjv3HyIf4oaOhZjT8hDne1Bfv+cFqZJ61Gk0MjANh/T5q9vxER/7TdU
NHKpoRV+NVlKN5bEU/NQ5FQjVXicfswxh6Y6fl2PIFqT2CfjD+FkBPU1iT9qyJYH
A38IRvwNtcitFgCeZwdGPoxiPPh1WHY8VxpUVBv/2JsUtrB/rAIbGqZoxAIWvijJ
Pe9o1TY3VlOzk9ASZ1AeatvOir+iDVJ5OpKmLnzc46QgGPUsjIyo6Sje9dxpGtoG
QQIDAQAB
-----END PUBLIC KEY-----
```

假设我们收到一个`File`作为参数:

```java
public static RSAPublicKey readPublicKey(File file) throws Exception {
    String key = new String(Files.readAllBytes(file.toPath()), Charset.defaultCharset());

    String publicKeyPEM = key
      .replace("-----BEGIN PUBLIC KEY-----", "")
      .replaceAll(System.lineSeparator(), "")
      .replace("-----END PUBLIC KEY-----", "");

    byte[] encoded = Base64.decodeBase64(publicKeyPEM);

    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    X509EncodedKeySpec keySpec = new X509EncodedKeySpec(encoded);
    return (RSAPublicKey) keyFactory.generatePublic(keySpec);
}
```

如我们所见，首先我们需要移除页眉、页脚和新行。然后我们需要将 Base64 编码的字符串解码成相应的二进制格式。

接下来，我们需要将结果加载到一个能够处理公钥材料的密钥规范类中。在本例中，我们将使用*x 509 encodedkeyspec*类。

最后，我们可以使用*key factory*类从规范中生成一个公钥对象。

### 3.3.从 PEM 字符串获取私钥

现在我们知道了如何读取公钥，读取私钥的算法也非常相似。

我们将使用 PKCS8 格式的 PEM 编码私钥。让我们看看页眉和页脚是什么样子的:

```java
-----BEGIN PRIVATE KEY-----
...Base64 encoded key...
-----END PRIVATE KEY-----
```

正如我们之前了解到的，我们需要一个能够处理 PKCS8 密钥材料的类。*pkcs 8 encodedkeyspec*类填补了这个角色。

那么我们来看看算法:

```java
public RSAPrivateKey readPrivateKey(File file) throws Exception {
    String key = new String(Files.readAllBytes(file.toPath()), Charset.defaultCharset());

    String privateKeyPEM = key
      .replace("-----BEGIN PRIVATE KEY-----", "")
      .replaceAll(System.lineSeparator(), "")
      .replace("-----END PRIVATE KEY-----", "");

    byte[] encoded = Base64.decodeBase64(privateKeyPEM);

    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(encoded);
    return (RSAPrivateKey) keyFactory.generatePrivate(keySpec);
}
```

## 4。使用 BouncyCastle 库

### 4.1。读取公钥

我们将探索 BouncyCastle 库，看看如何用它来替代纯 Java 实现。

让我们得到公钥:

```java
public RSAPublicKey readPublicKey(File file) throws Exception {
    KeyFactory factory = KeyFactory.getInstance("RSA");

    try (FileReader keyReader = new FileReader(file);
      PemReader pemReader = new PemReader(keyReader)) {

        PemObject pemObject = pemReader.readPemObject();
        byte[] content = pemObject.getContent();
        X509EncodedKeySpec pubKeySpec = new X509EncodedKeySpec(content);
        return (RSAPublicKey) factory.generatePublic(pubKeySpec);
    }
}
```

使用 BouncyCastle 时，我们需要注意几个重要的类:

*   *PemReader*–将一个`Reader`作为参数并解析其内容。**它删除不必要的头**并且**将底层 Base64 PEM 数据解码成二进制格式。**
*   *PEM object*–**存储`PemReader`** 生成的结果

让我们看看另一种方法，将 Java 的类(`X509EncodedKeySpec, KeyFactory`)包装到 BouncyCastle 自己的类(`JcaPEMKeyConverter`)中:

```java
public RSAPublicKey readPublicKeySecondApproach(File file) throws IOException {
    try (FileReader keyReader = new FileReader(file)) {
        PEMParser pemParser = new PEMParser(keyReader);
        JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        SubjectPublicKeyInfo publicKeyInfo = SubjectPublicKeyInfo.getInstance(pemParser.readObject());
        return (RSAPublicKey) converter.getPublicKey(publicKeyInfo);
    }
}
```

### 4.2.读取私钥

现在我们将看到两个与上面显示的例子非常相似的例子。

在第一个例子中，我们只需要用`PKCS8EncodedKeySpec`类替换`X509EncodedKeySpec`类，并返回一个`RSAPrivateKey`对象而不是一个`RSAPublicKey`:

```java
public RSAPrivateKey readPrivateKey(File file) throws Exception {
    KeyFactory factory = KeyFactory.getInstance("RSA");

    try (FileReader keyReader = new FileReader(file);
      PemReader pemReader = new PemReader(keyReader)) {

        PemObject pemObject = pemReader.readPemObject();
        byte[] content = pemObject.getContent();
        PKCS8EncodedKeySpec privKeySpec = new PKCS8EncodedKeySpec(content);
        return (RSAPrivateKey) factory.generatePrivate(privKeySpec);
    }
}
```

现在，让我们稍微修改一下上一节中的第二种方法，以便读取私钥:

```java
public RSAPrivateKey readPrivateKeySecondApproach(File file) throws IOException {
    try (FileReader keyReader = new FileReader(file)) {

        PEMParser pemParser = new PEMParser(keyReader);
        JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        PrivateKeyInfo privateKeyInfo = PrivateKeyInfo.getInstance(pemParser.readObject());

        return (RSAPrivateKey) converter.getPrivateKey(privateKeyInfo);
    }
}
```

正如我们所看到的，我们只是将`SubjectPublicKeyInfo`替换为`PrivateKeyInfo`，将`RSAPublicKey`替换为`RSAPrivateKey`。

### 4.3.优势

BouncyCastle 库有几个优点。

一个好处是**我们不需要手动跳过或删除页眉和页脚。**另一个是**我们不负责 Base64 解码，**也不负责。因此，我们可以用 BouncyCastle 编写不太容易出错的代码。

此外， **BouncyCastle 库也支持 PKCS1 格式**。尽管 PKCS1 也是一种用于存储加密密钥(只有 RSA 密钥)的流行格式，但 Java 本身并不支持它。

## 5.结论

在本文中，我们学习了如何从 PEM 文件中读取公钥和私钥。

首先，我们研究了公钥加密的几个关键概念。然后我们看到了如何使用纯 Java 读取公钥和私钥。

最后，我们探索了 BouncyCastle 库，发现它是一个很好的选择，因为与纯 Java 实现相比，它提供了一些优势。

GitHub 上有关于 Java 和 T2 的完整源代码。