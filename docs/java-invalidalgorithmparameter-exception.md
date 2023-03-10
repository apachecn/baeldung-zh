# InvalidAlgorithmParameterException:错误的 IV 长度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-invalidalgorithmparameter-exception>

## 1.概观

[高级加密标准](/web/20220628112139/https://www.baeldung.com/java-aes-encryption-decryption) (AES)是一种广泛使用的对称分组密码算法。[初始化向量](/web/20220628112139/https://www.baeldung.com/java-aes-encryption-decryption#3-initialization-vector-iv) (IV)在 AES 算法中起着重要的作用。

在本教程中，我们将解释如何在 Java 中生成 IV。此外，我们将描述当我们生成 IV 并在密码算法中使用它时**如何避免`InvalidAlgorithmParameterException`。**

## 2.初始化向量

AES 算法通常有三个输入:明文、密钥和 IV。它支持 128、192 和 256 位的密钥来加密和解密 128 位数据块中的数据。下图显示了 AES 输入:

[![](img/0344be2bdce5ad9a7cb6312be3c187ee.png)](/web/20220628112139/https://www.baeldung.com/wp-content/uploads/2020/12/Figures-Page-2.png)

IV 的目标是增强加密过程。在某些 [AES 操作模式](/web/20220628112139/https://www.baeldung.com/java-aes-encryption-decryption#aes-variations)中，IV 与密钥结合使用。例如，密码块链接(CBC)模式在其算法中使用 IV。

通常，IV 是由发送者选择的伪随机值。解密信息时，加密的 IV 必须相同。

它与加密的块大小相同。因此，IV 的大小是 16 字节或 128 位。

## 3.生成静脉注射

建议使用`java.security.SecureRandom`类代替`java.util.Random`生成随机 IV。此外，最佳实践是静脉注射不可预测。此外，我们不应该在源代码中硬编码 IV。

为了在密码中使用 IV，我们使用了`IvParameterSpec`类。让我们创建一个生成 IV 的方法:

```java
public static IvParameterSpec generateIv() {
    byte[] iv = new byte[16];
    new SecureRandom().nextBytes(iv);
    return new IvParameterSpec(iv);
}
```

## 4.例外

AES 算法要求 IV 大小必须为 16 字节(128 位)。所以，**如果我们提供一个大小不等于 16 字节的 IV，就会抛出一个`InvalidAlgorithmParameterException`**。

为了解决这个问题，我们必须使用大小为 16 字节的 IV。关于在 AES CBC 模式下使用 IV 的示例代码片段可在本文的[中找到。](/web/20220628112139/https://www.baeldung.com/java-aes-encryption-decryption#encryption-and-decryption)

## 5.结论

总之，我们已经学习了如何在 Java 中生成一个初始化向量(IV)。此外，我们已经描述了与 IV 代相关的异常。本教程中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220628112139/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-algorithms)