# 列出可用的密码算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-cipher-algorithms>

## 1.概观

在这个快速教程中，我们将学习 Java 中的 [`Cipher`](/web/20220628062613/https://www.baeldung.com/java-cipher-class) 类。然后，我们将看到如何列出可用的密码算法及其提供商。

## 2.密码课

位于`javax.crypto`包中的`Cipher`类是 Java 加密扩展(JCE)框架的核心。这个框架提供了一组用于数据加密、解密和散列的加密密码。

## 3.列出密码算法

**我们可以通过调用`Cipher.getInstance()`静态方法来实例化一个密码对象，使用请求的转换的名称作为参数:**

```java
Cipher cipher = Cipher.getInstance("AES");
```

有些情况下，我们需要获得可用密码算法及其提供商的列表。例如，我们希望根据类路径中存在的库来检查特定算法是否可用。

首先，我们需要**使用 [`Security.getProviders()`](https://web.archive.org/web/20220628062613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/Security.html#getProviders()) 方法**获得注册提供者的列表。**然后，调用 `Provider` 对象上的`[getServices()](https://web.archive.org/web/20220628062613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/Provider.html#getServices())` 方法将返回一组不可修改的由这个** `**Provider**` **:** 支持的所有服务

```java
for (Provider provider : Security.getProviders()) {
    for (Provider.Service service : provider.getServices()) {
        String algorithm = service.getAlgorithm();
        // ...
    }
}
```

可用算法列表:

```java
SHA3-224
NONEwithDSA
DSA
JavaLoginConfig
DSA
SHA3-384
SHA3-256
SHA1withDSA
...
```

然而，**并不是所有列出的算法都被`[Cipher.getInstance()](https://web.archive.org/web/20220628062613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html#getInstance(java.lang.String))`静态方法作为转换来支持** **。比如用`SHA3-224`实例化一个密码对象，这是一个哈希算法，会抛出一个`NoSuchAlgorithmException:`**

```java
Cipher cipher = Cipher.getInstance("SHA3-224");
```

让我们来看看运行时异常消息:

```java
java.security.NoSuchAlgorithmException: Cannot find any provider supporting SHA3-224
```

**因此，我们需要过滤列表并保留带有`Cipher`类型**的服务。我们可以使用 Java 流来过滤和收集兼容算法的名称列表:

```java
List<String> algorithms = Arrays.stream(Security.getProviders())
  .flatMap(provider -> provider.getServices().stream())
  .filter(service -> "Cipher".equals(service.getType()))
  .map(Provider.Service::getAlgorithm)
  .collect(Collectors.toList());
// ...
```

结果将类似于:

```java
AES_192/CBC/NoPadding
AES_192/OFB/NoPadding
AES_192/CFB/NoPadding
AESWrap_192
PBEWithHmacSHA224AndAES_256
AES_192/ECB/NoPadding
AES_192/GCM/NoPadding
ChaCha20-Poly1305
PBEWithHmacSHA384AndAES_128
AES_128/ECB/NoPadding
AES_128/OFB/NoPadding
AES_128/CBC/NoPadding
...
```

## 4.结论

在本教程中，我们首先了解了`Cipher`类。然后，我们学习了如何列出可用的密码算法。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628062613/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-algorithms)