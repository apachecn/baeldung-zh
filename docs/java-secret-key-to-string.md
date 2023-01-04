# Java 中的密钥和字符串转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-secret-key-to-string>

## 1.概观

在现实生活中，我们会遇到出于安全目的需要加密和解密的几种情况。我们可以使用秘密密钥轻松实现这一点。因此，为了加密和解密密钥，我们必须知道如何将密钥转换为字符串，反之亦然。在本教程中，我们将看到 Java 中的密钥和`String`转换。此外，我们将通过例子来介绍在 Java 中创建密钥的不同方法。

## 2.秘密钥匙

密钥是用于加密和解密消息的信息或参数。在 Java 中，我们有一个接口将它定义为一个秘密(对称)密钥。此接口的目的是对所有密钥接口进行分组(并为其提供类型安全)。

在 Java 中有两种生成密钥的方法:从随机数生成或者从给定的密码导出。

**在第一种方法中，密钥是从类似于`SecureRandom `类的加密安全(伪)随机数生成器中生成的。**

为了生成密钥，我们可以使用`KeyGenerator`类。让我们定义一个生成`SecretKey`的方法——参数`n`以位为单位指定密钥的长度(128、192 或 256):

```
public static SecretKey generateKey(int n) throws NoSuchAlgorithmException {
    KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
    keyGenerator.init(n);
    SecretKey originalKey = keyGenerator.generateKey();
    return originalKey;
}
```

**在第二种方法中，使用基于密码的密钥导出函数(如 PBKDF2** )从给定的密码中导出密钥。我们还需要一个 salt 值来将密码转换成密钥。盐也是一个随机值。

我们可以使用`SecretKeyFactory`类和`PBKDF2WithHmacSHA256`算法从给定的密码中生成一个密钥。

让我们定义一种从给定密码生成`SecretKey`的方法，迭代 65，536 次，密钥长度为 256 位:

```
public static SecretKey getKeyFromPassword(String password, String salt)
  throws NoSuchAlgorithmException, InvalidKeySpecException {
    SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    KeySpec spec = new PBEKeySpec(password.toCharArray(), salt.getBytes(), 65536, 256);
    SecretKey originalKey = new SecretKeySpec(factory.generateSecret(spec).getEncoded(), "AES");
    return originalKey;
}
```

## 3.`SecretKey`和字符串转换

### 3.1.`SecretKey`至`String`

我们将把`SecretKey`转换成一个`byte`数组。然后，我们将使用`Base64`编码将`byte`数组转换成`String`:

```
public static String convertSecretKeyToString(SecretKey secretKey) throws NoSuchAlgorithmException {
    byte[] rawData = secretKey.getEncoded();
    String encodedKey = Base64.getEncoder().encodeToString(rawData);
    return encodedKey;
}
```

### 3.2.`String`至`SecretKey`

我们将使用`Base64 `解码将编码的`String`键转换成一个`byte`数组。然后，使用`SecretKeySpecs`，我们将把`byte`数组转换成`SecretKey`:

```
public static SecretKey convertStringToSecretKeyto(String encodedKey) {
    byte[] decodedKey = Base64.getDecoder().decode(encodedKey);
    SecretKey originalKey = new SecretKeySpec(decodedKey, 0, decodedKey.length, "AES");
    return originalKey;
}
```

让我们快速验证一下转换:

```
SecretKey encodedKey = ConversionClassUtil.getKeyFromPassword("[[email protected]](/web/20220630020741/https://www.baeldung.com/cdn-cgi/l/email-protection)", "@$#[[email protected]](/web/20220630020741/https://www.baeldung.com/cdn-cgi/l/email-protection)#^$*");
String encodedString = ConversionClassUtil.convertSecretKeyToString(encodedKey);
SecretKey decodeKey = ConversionClassUtil.convertStringToSecretKeyto(encodedString);
Assertions.assertEquals(encodedKey, decodeKey);
```

## 4.结论

总之，我们已经学会了如何在 Java 中将一个`SecretKey`转换成`String`，反之亦然。此外，我们已经讨论了创建`SecretKey in` Java 的各种方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220630020741/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-3)