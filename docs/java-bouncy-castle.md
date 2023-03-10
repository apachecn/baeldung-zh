# Java bouncy castle 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bouncy-castle>

## 1。概述

[BouncyCastle](https://web.archive.org/web/20220911184840/https://www.bouncycastle.org/) 是一个 Java 库，补充了默认的 Java 加密扩展(JCE)。

在这篇介绍性文章中，我们将展示如何使用 BouncyCastle 来执行加密操作，如加密和签名。

## 2。Maven 配置

在我们开始使用这个库之前，我们需要将所需的依赖项添加到我们的`pom.xml` 文件中:

```java
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcpkix-jdk15on</artifactId>
    <version>1.58</version>
</dependency>
```

注意，我们总是可以在 [Maven 中央存储库](https://web.archive.org/web/20220911184840/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.bouncycastle%22%20)中查找最新的依赖版本。

## 3。设置无限强度管辖策略文件

标准 Java 安装在加密函数的强度方面受到限制，这是由于政策禁止使用超过特定值(例如 AES 的 128)的密钥。

为了克服这个限制，我们需要**配置无限强度管辖策略文件**。

为了做到这一点，我们首先需要通过这个[链接](https://web.archive.org/web/20220911184840/http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)下载这个包。然后，我们需要将压缩文件解压到我们选择的目录中，其中包含两个 jar 文件:

*   `local_policy.jar`
*   `US_export_policy.jar`

最后，我们需要查找`{JAVA_HOME}/lib/security`文件夹，并用我们在这里提取的文件替换现有的策略文件。

注意在 Java 9 中的**，我们不再需要下载策略文件包**，将`crypto.policy`属性设置为`unlimited`就足够了:

```java
Security.setProperty("crypto.policy", "unlimited");
```

完成后，我们需要检查配置是否正常工作:

```java
int maxKeySize = javax.crypto.Cipher.getMaxAllowedKeyLength("AES");
System.out.println("Max Key Size for AES : " + maxKeySize);
```

因此:

```java
Max Key Size for AES : 2147483647
```

基于由`getMaxAllowedKeyLength()`方法返回的最大密钥大小，我们可以有把握地说无限强度策略文件已经被正确安装。

如果返回值等于 128，我们需要确保已经将文件安装到运行代码的 JVM 中。

## 4。密码操作

### 4.1。准备证书和私钥

在我们开始实现加密函数之前，我们首先需要创建一个证书和一个私钥。

出于测试目的，我们可以使用这些资源:

*   [Baeldung.cer](https://web.archive.org/web/20220911184840/https://github.com/eugenp/tutorials/tree/master/libraries/src/main/resources)
*   [bael dung . p12(password = " password ")](https://web.archive.org/web/20220911184840/https://github.com/eugenp/tutorials/tree/master/libraries/src/main/resources)

`Baeldung.cer`是使用国际 X.509 公钥基础设施标准的数字证书，而`Baeldung.p12`是包含私钥的受密码保护的 [PKCS12](https://web.archive.org/web/20220911184840/https://tools.ietf.org/html/rfc7292) 密钥库。

让我们看看如何在 Java 中加载这些:

```java
Security.addProvider(new BouncyCastleProvider());
CertificateFactory certFactory= CertificateFactory
  .getInstance("X.509", "BC");

X509Certificate certificate = (X509Certificate) certFactory
  .generateCertificate(new FileInputStream("Baeldung.cer"));

char[] keystorePassword = "password".toCharArray();
char[] keyPassword = "password".toCharArray();

KeyStore keystore = KeyStore.getInstance("PKCS12");
keystore.load(new FileInputStream("Baeldung.p12"), keystorePassword);
PrivateKey key = (PrivateKey) keystore.getKey("baeldung", keyPassword);
```

首先，我们使用`addProvider()`方法动态添加了`BouncyCastleProvider`作为安全提供者。

这也可以通过编辑`{JAVA_HOME}/jre/lib/security/java.security`文件来静态完成，并添加以下行:

```java
security.provider.N = org.bouncycastle.jce.provider.BouncyCastleProvider
```

一旦正确安装了提供者，我们就使用`getInstance()`方法创建了一个`CertificateFactory`对象。

`getInstance()`方法有两个参数；证书类型“X.509”和安全提供者“BC”。

通过`generateCertificate()`方法，`certFactory`实例随后被用来生成一个`X509Certificate`对象。

同样，我们已经创建了一个 PKCS12 Keystore 对象，在其上调用了`load()`方法。

`getKey()`方法返回与给定别名相关联的私钥。

请注意，PKCS12 密钥库包含一组私钥，每个私钥可以有一个特定的密码，这就是为什么我们需要一个全局密码来打开密钥库，并需要一个特定的密码来检索私钥。

证书和私钥对主要用于非对称加密操作:

*   加密
*   [通信]解密
*   签名
*   确认

### 4.2。CMS/PKCS7 加密和解密

在非对称加密技术中，每次通信都需要一个公共证书和一个私钥。

收件人绑定到一个证书，该证书在所有发件人之间公开共享。

简单地说，发送方需要接收方的证书来加密消息，而接收方需要相关的私钥来解密消息。

让我们看看如何使用加密证书实现一个`encryptData()`函数:

```java
public static byte[] encryptData(byte[] data,
  X509Certificate encryptionCertificate)
  throws CertificateEncodingException, CMSException, IOException {

    byte[] encryptedData = null;
    if (null != data && null != encryptionCertificate) {
        CMSEnvelopedDataGenerator cmsEnvelopedDataGenerator
          = new CMSEnvelopedDataGenerator();

        JceKeyTransRecipientInfoGenerator jceKey 
          = new JceKeyTransRecipientInfoGenerator(encryptionCertificate);
        cmsEnvelopedDataGenerator.addRecipientInfoGenerator(transKeyGen);
        CMSTypedData msg = new CMSProcessableByteArray(data);
        OutputEncryptor encryptor
          = new JceCMSContentEncryptorBuilder(CMSAlgorithm.AES128_CBC)
          .setProvider("BC").build();
        CMSEnvelopedData cmsEnvelopedData = cmsEnvelopedDataGenerator
          .generate(msg,encryptor);
        encryptedData = cmsEnvelopedData.getEncoded();
    }
    return encryptedData;
}
```

我们已经使用接收者的证书创建了一个`JceKeyTransRecipientInfoGenerator`对象。

然后，我们创建了一个新的`CMSEnvelopedDataGenerator`对象，并在其中添加了收件人信息生成器。

之后，我们使用 AES CBC 算法，使用`JceCMSContentEncryptorBuilder`类创建了一个`OutputEncrytor`对象。

加密器稍后用于生成封装加密消息的`CMSEnvelopedData`对象。

最后，信封的编码表示作为字节数组返回。

现在，让我们看看`decryptData()`方法的实现是什么样子的:

```java
public static byte[] decryptData(
  byte[] encryptedData, 
  PrivateKey decryptionKey) 
  throws CMSException {

    byte[] decryptedData = null;
    if (null != encryptedData && null != decryptionKey) {
        CMSEnvelopedData envelopedData = new CMSEnvelopedData(encryptedData);

        Collection<RecipientInformation> recipients
          = envelopedData.getRecipientInfos().getRecipients();
        KeyTransRecipientInformation recipientInfo 
          = (KeyTransRecipientInformation) recipients.iterator().next();
        JceKeyTransRecipient recipient
          = new JceKeyTransEnvelopedRecipient(decryptionKey);

        return recipientInfo.getContent(recipient);
    }
    return decryptedData;
}
```

首先，我们使用加密数据字节数组初始化了一个`CMSEnvelopedData`对象，然后我们使用`getRecipients()`方法检索了消息的所有预期接收者。

一旦完成，我们就创建了一个与接收者的私钥相关联的新的`JceKeyTransRecipient`对象。

`recipientInfo`实例包含解密/封装的消息，但是我们不能检索它，除非我们有相应的接收者的密钥。

**最后，给定接收者键作为参数，`getContent()`方法返回从与该接收者相关联的`EnvelopedData`** 中提取的原始字节数组。

让我们编写一个简单的测试来确保一切正常运行:

```java
String secretMessage = "My password is 123456Seven";
System.out.println("Original Message : " + secretMessage);
byte[] stringToEncrypt = secretMessage.getBytes();
byte[] encryptedData = encryptData(stringToEncrypt, certificate);
System.out.println("Encrypted Message : " + new String(encryptedData));
byte[] rawData = decryptData(encryptedData, privateKey);
String decryptedMessage = new String(rawData);
System.out.println("Decrypted Message : " + decryptedMessage);
```

因此:

```java
Original Message : My password is 123456Seven
Encrypted Message : 0�*�H��...
Decrypted Message : My password is 123456Seven
```

### 4.3。CMS/PKCS7 签名和验证

签名和验证是验证数据真实性的加密操作。

让我们看看如何使用数字证书签署秘密消息:

```java
public static byte[] signData(
  byte[] data, 
  X509Certificate signingCertificate,
  PrivateKey signingKey) throws Exception {

    byte[] signedMessage = null;
    List<X509Certificate> certList = new ArrayList<X509Certificate>();
    CMSTypedData cmsData= new CMSProcessableByteArray(data);
    certList.add(signingCertificate);
    Store certs = new JcaCertStore(certList);

    CMSSignedDataGenerator cmsGenerator = new CMSSignedDataGenerator();
    ContentSigner contentSigner 
      = new JcaContentSignerBuilder("SHA256withRSA").build(signingKey);
    cmsGenerator.addSignerInfoGenerator(new JcaSignerInfoGeneratorBuilder(
      new JcaDigestCalculatorProviderBuilder().setProvider("BC")
      .build()).build(contentSigner, signingCertificate));
    cmsGenerator.addCertificates(certs);

    CMSSignedData cms = cmsGenerator.generate(cmsData, true);
    signedMessage = cms.getEncoded();
    return signedMessage;
} 
```

首先，我们将输入嵌入到一个`CMSTypedData`中，然后，我们创建了一个新的`CMSSignedDataGenerator`对象。

我们使用了`SHA256withRSA`作为签名算法，并用我们的签名密钥创建了一个新的`ContentSigner`对象。

随后使用`contentSigner`实例和签名证书来创建一个`SigningInfoGenerator` 对象。

在将`SignerInfoGenerator`和签名证书添加到`CMSSignedDataGenerator`实例之后，我们最后使用`generate()`方法创建一个 CMS 签名数据对象，它也带有 CMS 签名。

现在我们已经了解了如何对数据进行签名，让我们来看看如何验证已签名的数据:

```java
public static boolean verifSignedData(byte[] signedData)
  throws Exception {

    X509Certificate signCert = null;
    ByteArrayInputStream inputStream
     = new ByteArrayInputStream(signedData);
    ASN1InputStream asnInputStream = new ASN1InputStream(inputStream);
    CMSSignedData cmsSignedData = new CMSSignedData(
      ContentInfo.getInstance(asnInputStream.readObject()));

    SignerInformationStore signers 
      = cmsSignedData.getCertificates().getSignerInfos();
    SignerInformation signer = signers.getSigners().iterator().next();
    Collection<X509CertificateHolder> certCollection 
      = certs.getMatches(signer.getSID());
    X509CertificateHolder certHolder = certCollection.iterator().next();

    return signer
      .verify(new JcaSimpleSignerInfoVerifierBuilder()
      .build(certHolder));
}
```

同样，我们已经基于我们的签名数据字节数组创建了一个`CMSSignedData`对象，然后，我们已经使用`getSignerInfos()`方法检索了与签名相关联的所有签名者。

在这个例子中，我们只验证了一个签名者，但是对于一般用途，必须遍历由`getSigners()`方法返回的签名者集合，并分别检查每个签名者。

最后，我们使用`build()`方法创建了一个`SignerInformationVerifier`对象，并将其传递给`verify()`方法。

如果给定的对象可以成功验证 signer 对象上的签名，verify()方法返回`true` 。

这里有一个简单的例子:

```java
byte[] signedData = signData(rawData, certificate, privateKey);
Boolean check = verifSignData(signedData);
System.out.println(check);
```

因此:

```java
true
```

## 5。结论

在本文中，我们发现了如何使用 BouncyCastle 库来执行基本的加密操作，比如加密和签名。

在现实世界中，我们通常希望签署然后加密我们的数据，这样，只有收件人能够使用私钥解密它，并根据数字签名检查其真实性。

代码片段一如既往地可以在 GitHub 的[上找到。](https://web.archive.org/web/20220911184840/https://github.com/eugenp/tutorials/tree/master/libraries-security)