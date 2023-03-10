# Java 中加密 API 的常见异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-crypto-apis-exceptions>

## 1.介绍

[`Cipher`](/web/20221231082749/https://www.baeldung.com/java-cipher-class) 对象是一个重要的 Java 类，它帮助我们提供加密和解密功能。

在本文中，我们将看看在使用它加密和解密文本时可能出现的一些常见异常。

## 2.`NoSuchAlgorithmException`:找不到任何支持 X 的提供程序

如果我们使用一个虚构的算法运行下面的代码来获得一个`Cipher` 的实例:

```java
Cipher.getInstance("ABC");
```

我们将看到一个堆栈跟踪开始:

```java
java.security.NoSuchAlgorithmException: Cannot find any provider supporting ABC
    at javax.crypto.Cipher.getInstance(Cipher.java:543)
```

这到底是怎么回事？

嗯，要使用`Cipher.getInstance`，**，我们需要将一个算法转换作为`String,`传入，这必须是在[文档](https://web.archive.org/web/20221231082749/https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#Cipher)中列出的允许值。如果不是，我们会得到一个`NoSuchAlgorithmException`。**

如果我们已经检查了文档并且我们仍然看到这个，我们最好确保我们检查了转换的错误。

我们还可以在转换中指定算法模式和填充。

让我们确保这些字段的值也与给定的文档相匹配。否则，我们会看到一个异常:

```java
Cipher.getInstance("AES/ABC"); // invalid, causes exception

Cipher.getInstance("AES/CBC/ABC"); // invalid, causes exception

Cipher.getInstance("AES/CBC/PKCS5Padding"); // valid, no exception
```

请记住，如果我们不指定这些额外的字段，那么将使用默认值。

算法模式的默认值为 [ECB](https://web.archive.org/web/20221231082749/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB)) ，填充的默认值为`“NoPadding”.`

由于欧洲央行被认为是软弱的，我们将希望指定一种模式，以确保我们不会最终使用它。

总的来说，当解析一个`NoSuchAlgorithmException`时，我们将想要检查我们选择的转换的每个部分是否存在于文档的允许列表中，注意检查我们拼写中的任何拼写错误。

## 3.`IllegalBlockSizeException`:输入长度不是 X 字节的倍数

我们可能会看到这个异常有几个原因。

首先，让我们看看当我们试图解密时抛出的异常，然后让我们看看当我们试图加密时抛出的异常。

### 3.1.`IllegalBlockSizeException`解密期间

让我们写一个非常简单的解密方法:

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, key);

return cipher.doFinal(cipherTextBytes);
```

这段代码的行为会根据传递给我们的方法的密文而改变，这可能是我们无法控制的。

有时，我们可能会看到一个`IllegalBlockSizeException:`

```java
javax.crypto.IllegalBlockSizeException: Input length not multiple of 16 bytes
    at com.sun.crypto.provider.CipherCore.finalNoPadding(CipherCore.java:1109)
    at com.sun.crypto.provider.CipherCore.fillOutputBuffer(CipherCore.java:1053)
    at com.sun.crypto.provider.CipherCore.doFinal(CipherCore.java:853)
    at com.sun.crypto.provider.AESCipher.engineDoFinal(AESCipher.java:446)
    at javax.crypto.Cipher.doFinal(Cipher.java:2168)
```

那么“区块大小”到底是什么意思，是什么让它“非法”呢？

为了理解这一点，让我们记住 [AES](/web/20221231082749/https://www.baeldung.com/java-aes-encryption-decryption) 是[分组密码](https://web.archive.org/web/20221231082749/https://en.wikipedia.org/wiki/Block_cipher)的一个例子。

分组密码的工作原理是采用称为分组的固定长度的比特组。

对于我们的算法，要找出一个块中有多少字节，我们可以使用:

```java
Cipher.getInstance("AES/ECB/PKCS5Padding").getBlockSize();
```

由此我们可以看出 **AES 使用的是 16 字节的块。**

这意味着它将获取一个 16 字节的块，执行相关的算法步骤，然后移动到下一个 16 字节的块。

简单地说，**非法块是指不包含正确字节数的块。**

通常，当文本长度不是 16 字节的倍数时，这发生在最后一个块上。

这通常意味着要解密的文本一开始就没有正确加密，因此无法解密。

请记住，我们不能控制给我们的代码解密的输入，所以我们必须准备好处理这个异常。

因此，像`cipher.doFinal` 这样的方法抛出一个`IllegalBlockSizeException` 来强迫我们处理这个场景，要么通过[抛出](/web/20221231082749/https://www.baeldung.com/java-exceptions#1throws)它，要么在`[try-catch](/web/20221231082749/https://www.baeldung.com/java-exceptions#2-try-catch)` 语句*中。否则*，代码不会编译。

但是，请记住，大约每 16 次中有一次，一些错误的密文恰好是正确的长度，以避免 AES 出现这种例外。

在这种情况下，我们很可能会遇到本文中提到的其他异常。

### 3.2.`IllegalBlockSizeException`加密期间

现在，让我们在尝试加密文本“`https://www.baeldung.com/`”时看看这个异常:

```java
String plainText = "https://www.baeldung.com/";
byte[] plainTextBytes = plainText.getBytes();

Cipher cipher = Cipher.getInstance("AES/ECB/NoPadding");
cipher.init(Cipher.ENCRYPT_MODE, key);

return cipher.doFinal(plainTextBytes);
```

正如我们在上面看到的，为了让 AES 算法工作，字节数必须是 16 的倍数，而我们的文本不是这样。因此，运行这段代码会产生与上面相同的异常。

那么我们只能用 AES 加密 16、32、48…字节的文本吗？

如果我们想要加密的东西没有正确的字节数呢？

嗯，这就是我们需要填充数据的地方。

**填充数据仅仅意味着我们将在文本的开始、中间或结尾添加额外的字节，**从而确保数据现在具有正确的字节数。

与算法名称和模式一样，我们可以使用一系列允许的填充操作。

幸运的是，Java 为我们解决了这个问题，所以我们不打算在这里详细讨论它是如何工作的。

我们所要做的就是在我们的`Cipher`实例上**设置一个填充操作，就像 [PKCS #5](https://web.archive.org/web/20221231082749/https://www.rfc-editor.org/rfc/rfc8018) ，而不是指定“no padding”:**

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
```

当然，解密文本的代码也必须使用相同的填充操作。

### 3.3.其他故障排除提示

如果我们开始得到一个`NoSuchAlgorithmException`或者一个 `NoSuchPaddingException,`，我们将需要检查[Java 文档](https://web.archive.org/web/20221231082749/https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#Cipher)以确保我们使用了有效的填充——并且我们的拼写没有错别字。

如果我们做到了这一点，那么就有必要检查我们正在查看的文档是否与我们正在使用的 Java 版本相匹配，因为允许的填充操作可能会在不同版本之间发生变化。本文提供的链接是针对 Java 8 的。

## 4.`BadPaddingException`:给定的最后一个模块未正确填充

如果在处理填充时遇到问题，代码将抛出一个`BadPaddingException`，表明我们使用的填充有问题。

然而，实际上可能有几个不同的问题导致我们看到这个异常。

### 4.1.`BadPaddingException `由不正确的填充引起

假设我们的文本“`https://www.baeldung.com/`”是使用填充的 ISO 10126 加密的:

```java
Cipher cipher = Cipher.getInstance("AES/ECB/ISO10126Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);
byte[] cipherTextBytes = cipher.doFinal(plainTextBytes);
```

然后，如果我们尝试使用不同的填充来解密它，比如 PKCS #5:

```java
cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, encryptionKey);

return cipher.doFinal(cipherTextBytes);
```

我们的代码会抛出一个异常:

```java
javax.crypto.BadPaddingException: Given final block not properly padded. Such issues can arise if a bad key is used during decryption.
  at com.sun.crypto.provider.CipherCore.unpad(CipherCore.java:975)
  at com.sun.crypto.provider.CipherCore.fillOutputBuffer(CipherCore.java:1056)
  at com.sun.crypto.provider.CipherCore.doFinal(CipherCore.java:853)
  at com.sun.crypto.provider.AESCipher.engineDoFinal(AESCipher.java:446)
  at javax.crypto.Cipher.doFinal(Cipher.java:2168)
```

但是，当我们看到这个异常时，填充往往不是根本原因。

上面异常中的一行暗示了这一点，“如果在解密过程中使用了错误的密钥，就会出现这样的问题。”

那么让我们看看还有什么能引起一个`BadPaddingException.`

### 4.2.`BadPaddingException `由不正确的键引起

正如堆栈跟踪所示，当我们没有使用正确的加密密钥进行解密时，我们可能会看到这个异常:

```java
SecretKey encryptionKey = CryptoUtils.getKeyForText("BaeldungIsASuperCoolSite");
SecretKey differentKey = CryptoUtils.getKeyForText("ThisGivesUsAnAlternative");

Cipher cipher = Cipher.getInstance("AES/ECB/ISO10126Padding");

cipher.init(Cipher.ENCRYPT_MODE, encryptionKey);
byte[] cipherTextBytes = cipher.doFinal(plainTextBytes);

cipher.init(Cipher.DECRYPT_MODE, differentKey);

return cipher.doFinal(cipherTextBytes);
```

上面的代码抛出了一个`BadPaddingException` 而不是一个`InvalidKeyException`,因为这是代码遇到问题并且无法继续的地方。

这可能是该异常最常见的原因。

如果我们看到这个异常，那么我们必须确保我们使用了正确的密钥。

这意味着我们必须使用相同的密钥进行加密和解密。

### 4.3.`BadPaddingException `由不正确的算法引起

鉴于上述情况，下一个应该是显而易见的，但它总是值得检查。

如果我们尝试使用与数据加密方式不同的算法或算法模式进行解密，我们可能会看到类似的症状:

```java
Cipher cipher = Cipher.getInstance("AES/ECB/ISO10126Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);
byte[] cipherTextBytes = cipher.doFinal(plainTextBytes);

cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, key);

return cipher.doFinal(cipherTextBytes);
```

在上面的例子中，数据使用模式 [CBC](https://web.archive.org/web/20221231082749/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_block_chaining_(CBC)) 加密，但是使用模式 ECB 解密，这是行不通的(在大多数情况下)。

一般来说，我们解决这个异常的方法是**验证我们的解密机制的每个组件与数据的加密方式相匹配。**

## 5.`InvalidKeyException`

**`InvalidKeyException` 通常表示我们错误地设置了`Cipher`对象。**

让我们来看看最常见的原因。

### 5.1.`InvalidKeyException`:参数缺失

我们使用的一些算法需要一个[初始化向量](https://web.archive.org/web/20221231082749/https://en.wikipedia.org/wiki/Initialization_vector) (IV)。

IV 防止加密文本的重复，因此是某些加密模式(如 CBC)所必需的。

让我们尝试初始化一个没有 IV 集的`Cipher`的实例:

```java
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, encryptionKey);
cipher.doFinal(cipherTextBytes);
```

如果我们运行上面的代码，我们会看到下面的 stacktrace:

```java
java.security.InvalidKeyException: Parameters missing
  at com.sun.crypto.provider.CipherCore.init(CipherCore.java:469)
  at com.sun.crypto.provider.AESCipher.engineInit(AESCipher.java:313)
  at javax.crypto.Cipher.implInit(Cipher.java:805)
  at javax.crypto.Cipher.chooseProvider(Cipher.java:867)
  at javax.crypto.Cipher.init(Cipher.java:1252)
  at javax.crypto.Cipher.init(Cipher.java:1189)
```

幸运的是，这个问题很容易解决，因为**我们只需要用 IV 初始化`Cipher `:**

```java
byte[] ivBytes = new byte[]{'B', 'a', 'e', 'l', 'd', 'u', 'n', 'g', 'I', 's', 'G', 'r', 'e', 'a', 't', '!'};
IvParameterSpec ivParameterSpec = new IvParameterSpec(ivBytes);

cipher = Cipher.getInstance("AES/CBC/PKCS5Padding"); 
cipher.init(Cipher.DECRYPT_MODE, encryptionKey, ivParameterSpec);
byte[] decryptedBytes = cipher.doFinal(cipherTextBytes);
```

请注意，给定的 IV 必须与用于加密文本的 IV 相同。

关于 IV 的最后一点是它必须有一定的长度。

如果我们使用 CBC，我们的 IV 必须正好是 16 字节长。

如果我们尝试使用不同的字节数，我们会得到一个非常清晰的`InvalidAlgorithmParameterException:`

```java
java.security.InvalidAlgorithmParameterException: Wrong IV length: must be 16 bytes long
  at com.sun.crypto.provider.CipherCore.init(CipherCore.java:525)
  at com.sun.crypto.provider.AESCipher.engineInit(AESCipher.java:346)
  at javax.crypto.Cipher.implInit(Cipher.java:809)
  at javax.crypto.Cipher.chooseProvider(Cipher.java:867)
```

修复只是为了确保我们的 IV 是 16 字节长。

### 5.2.`InvalidKeyException`:无效的 AES 密钥长度:X 字节

我们将很快介绍这一点，因为它与上述情况非常相似。

如果我们试图使用一个长度不正确的键，那么我们会看到一个简单的异常:

```java
java.security.InvalidKeyException: Invalid AES key length: X bytes
  at com.sun.crypto.provider.AESCrypt.init(AESCrypt.java:87)
  at com.sun.crypto.provider.CipherBlockChaining.init(CipherBlockChaining.java:93)
  at com.sun.crypto.provider.CipherCore.init(CipherCore.java:591)
```

我们的密钥也必须是 16 字节。

这是因为 Java 通常默认只支持 128 位(16 字节)加密。

## 6.结论

在本文中，我们看到了加密和解密文本时可能发生的各种异常。

特别是，我们看到异常可能提到一件事，如填充，但根本原因实际上是其他事情，如无效的键。

与往常一样，GitHub 上的示例项目[可用。](https://web.archive.org/web/20221231082749/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-3)