# 密码课程指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cipher-class>

## 1。概述

简而言之，加密是对消息进行编码的过程，只有授权用户才能理解或访问它。

该消息被称为`plaintext`，使用加密算法进行加密——a`cipher`——生成只能由授权用户通过解密读取的`ciphertext`。

在本文中，我们详细描述了核心类`Cipher`的**，它在 Java 中提供了加密和解密功能**。

## 2。密码类

Java Cryptography Extension (JCE)是 Java Cryptography Architecture(JCA)的**部分，它为应用程序提供用于数据加密和解密以及私有数据散列的加密密码。**

位于`javax.crypto`包中的 [`Cipher`](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html) 类构成了 JCE 框架的核心，提供了加密和解密的功能。

### 2.1。密码实例化

为了实例化一个`Cipher`对象，我们**调用静态`getInstance`方法，传递所请求的转换的名称**。可选地，可以指定提供者的名称。

让我们编写一个示例类来说明`Cipher`的实例化:

```
public class Encryptor {

    public byte[] encryptMessage(byte[] message, byte[] keyBytes) 
      throws InvalidKeyException, NoSuchPaddingException, NoSuchAlgorithmException {
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        //...
    }
}
```

转换`AES/ECB/PKCS5Padding`告诉 *getInstance* 方法将`Cipher`对象实例化为具有 ECB [操作模式](https://web.archive.org/web/20220525141502/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation)和 PKCS5 [填充方案](https://web.archive.org/web/20220525141502/https://en.wikipedia.org/wiki/Padding_(cryptography))的 [AES](https://web.archive.org/web/20220525141502/https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 密码。

我们还可以通过在转换中只指定算法来实例化`Cipher`对象:

```
Cipher cipher = Cipher.getInstance("AES");
```

在这种情况下，Java 将为模式和填充方案使用特定于提供者的默认值。

注意，如果转换是`null`、空的或者格式无效，或者提供者不支持，那么`getInstance` 将抛出一个`NoSuchAlgorithmException`。

如果转换包含不支持的填充方案，它将抛出一个`NoSuchPaddingException`。

### 2.2.线程安全

`Cipher `类是有状态类，没有任何形式的内部同步。事实上，像`[init()](https://web.archive.org/web/20220525141502/https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/java.base/share/classes/javax/crypto/Cipher.java#L1235) `或`[update()](https://web.archive.org/web/20220525141502/https://github.com/openjdk/jdk/blob/1aa653957619acfdb5f08ce0f3a1ad1a17cfa127/src/java.base/share/classes/javax/crypto/Cipher.java#L1820) `这样的方法会改变特定`Cipher `实例的内部状态。

**因此，`Cipher `类不是线程安全的。**所以我们应该为每个加密/解密需求创建一个`Cipher `实例。

### 2.3。按键

`[Key](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/Key.html)` 接口代表加密操作的密钥。密钥是不透明的容器，包含编码的密钥、密钥的编码格式及其加密算法。

密钥一般通过[密钥生成器](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/KeyGenerator.html)，证书，或者[密钥规范](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/spec/KeySpec.html)使用[密钥工厂](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/KeyFactory.html)获得。

让我们从提供的密钥字节创建一个对称的`Key`:

```
SecretKey secretKey = new SecretKeySpec(keyBytes, "AES");
```

### 2.4。密码初始化

**我们调用`init()`方法，用一个 [`Key`](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/Key.html) 或 [`Certificate`](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/cert/Certificate.html) 和一个`opmode` 来初始化 C `ipher`对象**，表示密码的运行模式。

可选地，**我们可以传入一个随机源**。默认情况下，使用最高优先级安装提供程序的 [`SecureRandom`](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/security/SecureRandom.html) 实现。否则，它将使用系统提供的资源。

**我们可以随意指定一组算法特定的参数。**例如，我们可以通过一个 [`IvParameterSpec`](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/spec/IvParameterSpec.html) 到**指定一个[初始化向量](https://web.archive.org/web/20220525141502/https://en.wikipedia.org/wiki/Initialization_vector)** 。

以下是可用的密码操作模式:

*   `ENCRYPT_MODE`:将`cipher`对象初始化为加密模式
*   `DECRYPT_MODE`:将`cipher`对象初始化为解密模式
*   `WRAP_MODE`:将`cipher`对象初始化为[绕键](https://web.archive.org/web/20220525141502/https://en.wikipedia.org/wiki/Key_Wrap)模式
*   `UNWRAP_MODE`:将`cipher`对象初始化为[解键](https://web.archive.org/web/20220525141502/https://en.wikipedia.org/wiki/Key_Wrap)模式

让我们初始化`Cipher`对象:

```
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
SecretKey secretKey = new SecretKeySpec(keyBytes, "AES");
cipher.init(Cipher.ENCRYPT_MODE, secretKey);
// ...
```

现在，如果提供的密钥不适合初始化密码，比如当密钥长度/编码无效时，`init` 方法会抛出一个`InvalidKeyException`。

当密码需要某些无法从密钥中确定的算法参数时，或者如果密钥的密钥大小超过了最大允许密钥大小(由[配置的 JCE 辖区](https://web.archive.org/web/20220525141502/https://docs.oracle.com/javase/9/security/java-cryptography-architecture-jca-reference-guide.htm#JSSEC-GUID-EFA5AC2D-644E-4CD9-8523-C6D3936D5FB1)策略文件确定)，也会抛出该错误。

让我们看一个使用`Certificate`的例子:

```
public byte[] encryptMessage(byte[] message, Certificate certificate) 
  throws InvalidKeyException, NoSuchPaddingException, NoSuchAlgorithmException {

    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
    cipher.init(Cipher.ENCRYPT_MODE, certificate);
    // ...
}
```

`Cipher`对象通过调用`getPublicKey`方法从证书中获取数据加密的公钥。

### 2.5。加密和解密

初始化`Cipher`对象后，我们调用`doFinal()`方法来执行加密或解密操作。此方法返回包含加密或解密消息的字节数组。

`doFinal()`方法还通过调用`init()`方法将`Cipher`对象重置为之前初始化时的状态，使`Cipher`对象可用于加密或解密其他消息。

让我们在我们的`encryptMessage`方法中调用`doFinal`:

```
public byte[] encryptMessage(byte[] message, byte[] keyBytes)
  throws InvalidKeyException, NoSuchPaddingException, NoSuchAlgorithmException, 
    BadPaddingException, IllegalBlockSizeException {

    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
    SecretKey secretKey = new SecretKeySpec(keyBytes, "AES");
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    return cipher.doFinal(message);
}
```

为了执行解密操作，我们将`opmode`改为`DECRYPT_MODE`:

```
public byte[] decryptMessage(byte[] encryptedMessage, byte[] keyBytes) 
  throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, 
    BadPaddingException, IllegalBlockSizeException {

    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
    SecretKey secretKey = new SecretKeySpec(keyBytes, "AES");
    cipher.init(Cipher.DECRYPT_MODE, secretKey);
    return cipher.doFinal(encryptedMessage);
}
```

### 2.6。供应商

设计使用基于提供者的架构,**JCE 允许合格的加密库，如 [BouncyCastle](https://web.archive.org/web/20220525141502/https://www.bouncycastle.org/) 作为安全提供者插入，新算法无缝添加**。

现在让我们添加 BouncyCastle 作为安全提供者。我们可以静态或动态地添加安全提供者。

**为了静态添加 BouncyCastle，我们修改了位于`<JAVA_HOME>/jre/lib/security`文件夹中的`java.security`文件**。

我们在列表的末尾添加一行:

```
...
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=org.bouncycastle.jce.provider.BouncyCastleProvider
```

添加提供者属性时，属性键的格式为`security.provider.N`，其中数字`N`比列表中的最后一个数字多 1。

**我们还可以动态添加 BouncyCastle 安全提供者**,而无需修改安全文件:

```
Security.addProvider(new BouncyCastleProvider());
```

我们现在可以在密码初始化期间指定提供者:

```
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding", "BC");
```

`BC` 指定 BouncyCastle 作为提供者。我们可以通过`Security.getProviders()`方法获得注册提供者的列表。

## 3。测试加密和解密

让我们编写一个示例测试来说明消息加密和解密。

在此测试中，我们使用 AES 加密算法和 128 位密钥，并断言解密结果等于原始消息文本:

```
@Test
public void whenIsEncryptedAndDecrypted_thenDecryptedEqualsOriginal() 
  throws Exception {

    String encryptionKeyString =  "thisisa128bitkey";
    String originalMessage = "This is a secret message";
    byte[] encryptionKeyBytes = encryptionKeyString.getBytes();

    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
    SecretKey secretKey = new SecretKeySpec(encryptionKeyBytes, "AES");
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);

    byte[] encryptedMessageBytes = cipher.doFinal(message.getBytes());

    cipher.init(Cipher.DECRYPT_MODE, secretKey);

    byte[] decryptedMessageBytes = cipher.doFinal(encryptedMessageBytes);
    assertThat(originalMessage).isEqualTo(new String(decryptedMessageBytes));
}
```

## 4。结论

在本文中，我们讨论了`Cipher`类并给出了使用示例。关于`Cipher`类和 JCE 框架的更多细节可以在[类文档](https://web.archive.org/web/20220525141502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html)和 [Java 加密体系结构(JCA)参考指南](https://web.archive.org/web/20220525141502/https://docs.oracle.com/javase/9/security/java-cryptography-architecture-jca-reference-guide.htm)中找到。

实现所有这些例子和代码片段**可以在 GitHub** 上的 [**中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。**](https://web.archive.org/web/20220525141502/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)