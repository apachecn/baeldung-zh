# 加密的初始化向量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-encryption-iv>

## 1。概述

在本教程中，我们将讨论如何在加密算法中使用一个[初始化向量(IV)](https://web.archive.org/web/20220821143847/https://en.wikipedia.org/wiki/Initialization_vector) 。我们还将讨论使用 IV 时的最佳实践。

本文假设读者对密码学有基本的了解。

对于我们所有的例子，我们将在不同的模式下使用 [AES 算法。](/web/20220821143847/https://www.baeldung.com/java-aes-encryption-decryption)

## 2。加密算法

任何加密算法都需要一些数据或明文和一个密钥来生成加密的文本或密文。此外，它还使用生成的密文和相同的密钥来生成解密的数据或原始明文。

例如，块密码算法通过加密和解密固定长度的块来提供安全性。我们使用不同的加密模式对整个数据重复应用算法，并指定要使用的 IV 类型。

在分组密码的情况下，我们使用相同大小的块。如果明文的大小小于块的大小，我们使用填充。一些模式不使用填充，因为它们使用分组密码作为流密码。

## 3。初始化向量(IV)

我们在加密算法中使用 IV 作为起始状态，将其添加到密码中以隐藏加密数据中的模式。这有助于避免每次调用后重新发布新密钥的需要。

### 3.1。IV 的属性

对于大多数加密模式，我们使用独特的序列或 IV。而且，我们永远不应该用同一个密钥重复使用同一个 IV。这确保了相同明文加密的不同密文，即使我们用相同的密钥多次加密它。

让我们根据加密模式来看看 IV 的一些**特征:**

*   必须是不重复的
*   基于加密模式，它也需要是随机的
*   它不必是秘密的
*   它需要是一个加密随机数
*   无论密钥长度如何，AES 的 IV 始终是 128 位

### 3.2。生成 IV

我们可以直接从`Cipher`类获得一个 IV:

```java
byte[] iv = cipher.getIV();
```

如果我们不确定默认的实现，我们总是可以编写我们的方法来生成 IV。如果我们不提供显式的 IV，那么使用`Cipher` `.getIV()`隐式地获取 IV。我们可以使用任何方法来生成 IV，只要它符合上面讨论的属性。

首先，让我们使用`SecureRandom`创建一个随机的 IV:

```java
public static IvParameterSpec getIVSecureRandom(String algorithm) throws NoSuchAlgorithmException, NoSuchPaddingException {
    SecureRandom random = SecureRandom.getInstanceStrong();
    byte[] iv = new byte[Cipher.getInstance(algorithm).getBlockSize()];
    random.nextBytes(iv);
    return new IvParameterSpec(iv);
}
```

接下来，我们将通过从`Cipher`类获取参数来创建 IV:

```java
public static IvParameterSpec getIVInternal(Cipher cipher) throws InvalidParameterSpecException {
    AlgorithmParameters params = cipher.getParameters();
    byte[] iv = params.getParameterSpec(IvParameterSpec.class).getIV();
    return new IvParameterSpec(iv);
}
```

我们可以使用上述任何一种方法来生成随机的、不可预测的 IV。然而，**对于像 GCM 这样的一些模式，我们将 IV 与计数器**一起使用。在这种情况下，我们使用前几个字节，主要是 12 个字节用于 IV，接下来的 4 个字节用于计数器:

```java
public static byte[] getRandomIVWithSize(int size) {
    byte[] nonce = new byte[size];
    new SecureRandom().nextBytes(nonce);
    return nonce;
}
```

在这种情况下，我们需要确保不重复计数器，并且 IV 也是唯一的。

最后，**尽管不推荐，我们也可以使用硬编码的 IV** 。

## 4。在不同模式下使用静脉注射

众所周知，加密的主要功能是屏蔽明文，使攻击者无法猜测。因此，我们使用不同的密码模式来掩盖密文中的模式。

像 ECB、CBC、OFB、CFB、CTR、CTS 和 XTS 这样的模式提供了保密性。但是这些模式不能防止篡改和修改。我们可以添加消息认证码(MAC)或数字签名进行检测。我们使用各种实现来提供认证加密(AE)下的组合模式。CCM、GCM、CWC、EAX、IAPM 和 OCB 就是几个例子。

### 4.1。电子码本(ECB)模式

[电码本模式](https://web.archive.org/web/20220821143847/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB))用密钥分别加密每个块。这总是将相同的明文加密成相同的密文块，因此不能很好地隐藏模式。因此，我们不把它用于加密协议。解密同样容易受到重放攻击。

为了在 ECB 模式下加密数据，我们使用:

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);
ciphertext = cipher.doFinal(data);
```

为了在 ECB 模式下解密数据，我们写:

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, key);
plaintext = cipher.doFinal(cipherText);
```

我们没有使用任何 IV，所以同样的明文会产生同样的密文，从而容易受到攻击。虽然 ECB 模式最容易受到攻击，但它仍然是许多提供商的默认加密模式。因此，我们需要对显式设置加密模式更加警惕。

### 4.2。网络区块链(CBC)模式

[赛博区块链模式](https://web.archive.org/web/20220821143847/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_block_chaining_(CBC))使用 IV 来防止相同的明文产生相同的密文。**我们需要注意 IV 是可靠随机的或者唯一的。否则，我们将面临与欧洲央行模式**相同的脆弱性。

让我们使用`getIVSecureRandom`得到随机 IV:

```java
IvParameterSpec iv = CryptoUtils.getIVSecureRandom("AES"); 
```

首先，我们将使用 IV 以 CBC 模式加密数据:

```java
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key, iv);
```

接下来，让我们使用`IvParameterSpec`对象传递相同的 IV 进行解密:

```java
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, key, new IvParameterSpec(iv));
```

### 4.3。网络反馈(CFB)模式

[赛博回馈模式](https://web.archive.org/web/20220821143847/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_feedback_(CFB))是最基本的串流模式。这就像一个自同步流密码。与 CBC 模式不同，我们不需要任何填充。在 CFB 模式中，我们使用 IV 作为密码生成的流的源。**同样，如果我们对不同的加密使用相同的 IV，相似之处可能会出现在密文中。这里也和 CBC 模式一样，IV 应该是随机的。** **在第四种情况下是可预测的，那么我们在保密性上就吃亏了。**

让我们为 CFB 模式生成一个随机 IV:

```java
IvParameterSpec iv = CryptoUtils.getIVSecureRandom("AES/CFB/NoPadding");
```

另一种极端情况是，如果我们使用全零 IV，那么在 CFB-8 模式下，一些密钥可能生成全零 IV 和全零明文。在这种情况下，1/256 密钥不会产生加密。这将导致明文作为密文返回。

**对于 CBC 和 CFB 模式，重用 IV 揭示了关于由两个消息共享的公共块的信息。**

### 4.4。计数器(CTR) **和输出反馈(OFB)模式**

[计数器模式](https://web.archive.org/web/20220821143847/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_(CTR))和[输出反馈模式](https://web.archive.org/web/20220821143847/https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Output_feedback_(OFB))将分组密码变成同步流密码。每种模式都生成密钥流块。在这种情况下，我们用一个特定的 IV 初始化密码。我们这样做主要是为了给 IV 分配 12 个字节，给计数器分配 4 个字节。这样，我们可以加密长度为 2^32 块的消息。

这里，让我们创建一个 IV:

```java
IvParameterSpec ivSpec = CryptoUtils.getIVSecureRandom("AES");
```

对于 CTR 模式，**初始比特流取决于 IV 和键。这里，重用 IV 也会导致关键比特流重用。这反过来会导致破坏安全性**。

如果 IV 不是唯一的，则计数器可能无法为对应于重复计数器块的块提供预期的机密性。但是，其他数据块不受影响。

### 4.5。伽罗瓦/计数器(GCM)模式

[伽罗瓦/计数器模式](https://web.archive.org/web/20220821143847/https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf)是一种 AEAD 加密模式。它将计数器模式加密与身份验证机制相结合。而且，它保护明文和附加认证数据(AAD)。

然而，GCM 中的认证依赖于 IVs 的唯一性。我们使用随机数作为 IV。如果我们重复甚至一个 IV，那么我们的实现可能容易受到攻击。

由于 GCM 使用 AES 进行加密，因此 IV 或计数器是 16 个字节。因此，我们使用前 12 个字节作为 IV，后 4 个字节作为计数器。

为了在 GCM 模式下创建一个 IV，我们需要设置`GCMParameterSpec`。让我们创建一个 IV:

```java
byte[] iv = CryptoUtils.getRandomIVWithSize(12); 
```

首先，让我们获取一个`Cipher`的实例，并使用 IV:

```java
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(128, iv));
```

现在，我们将用 IV 创建并初始化`Cipher`以进行解密:

```java
cipher.init(Cipher.DECRYPT_MODE, key, new GCMParameterSpec(128, iv)); 
```

这里，我们也需要一个唯一的 IV，否则就可以破译明文。

### 4.6。iv 总结

下表总结了不同模式所需的静脉注射类型:

[![](img/fbfa86d9af27e7e880e9b6001752928f.png)](/web/20220821143847/https://www.baeldung.com/wp-content/uploads/2021/11/Screenshot-2021-10-22-at-10.42.59-AM.png)

正如我们所看到的，重用具有相同密钥的 IV 会导致安全性的损失。如果可能，我们应该使用更高级的模式，如 GCM。此外，一些模式，如 CCM，在标准 JCE 分布中不可用。在这种情况下，我们可以使用[Bouncy Castle](https://web.archive.org/web/20220821143847/https://www.bouncycastle.org/fips-java/BCFipsIn100.pdf)API 来实现。

## 5。结论

在本文中，我们展示了如何在不同的加密模式下使用 IV。我们还讨论了使用 IV 时的问题和最佳实践。

和往常一样，我们可以在 GitHub 上找到源代码[。](https://web.archive.org/web/20220821143847/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-3)