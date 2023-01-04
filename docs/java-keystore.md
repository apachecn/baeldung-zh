# Java 密钥库 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-keystore>

## 1。概述

在本教程中，我们将学习使用`KeyStore ` API 在 Java 中管理加密密钥和证书。

## 2。密钥库

如果我们需要在 Java 中管理密钥和证书，我们需要一个 **`keystore`，它只是密钥和证书的别名`entries`的安全集合。**

我们通常将密钥库保存到文件系统中，并且可以用密码来保护它。

默认情况下，Java 有一个位于`JAVA_HOME/` jre `/lib/security/cacerts`的密钥库文件。我们可以使用默认的密钥库密码`changeit`来访问这个密钥库。

现在，有了这些背景，让我们开始创建我们的第一个。

## 3。创建密钥库

### 3.1。施工

我们可以使用 [keytool](https://web.archive.org/web/20220625174447/https://docs.oracle.com/en/java/javase/11/tools/keytool.html) 轻松地创建一个密钥库，或者我们可以使用`KeyStore` API 以编程方式完成:

```java
KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
```

这里，我们使用默认类型，尽管有一些密钥库类型可用，如`jceks`或`pkcs12`。

我们可以使用一个`-Dkeystore.type`参数覆盖默认的“JKS”(Oracle 专有的密钥库协议)类型:

```java
-Dkeystore.type=pkcs12
```

或者，我们当然可以在 `getInstance`中列出支持的格式之一:

```java
KeyStore ks = KeyStore.getInstance("pkcs12"); 
```

### 3.2。初始化

最初，我们需要`load`密钥库:

```java
char[] pwdArray = "password".toCharArray();
ks.load(null, pwdArray); 
```

无论是创建新的密钥库还是打开现有的密钥库，我们都使用`load`。

并且，我们通过将`null`作为第一个参数来告诉`KeyStore`创建一个新的。

我们还提供了一个密码，用于将来访问密钥库。我们也可以将这个设置为`null`，尽管这会使我们的秘密公开。

### 3.3。存储

最后，我们将新的密钥库保存到文件系统:

```java
try (FileOutputStream fos = new FileOutputStream("newKeyStoreFileName.jks")) {
    ks.store(fos, pwdArray);
} 
```

注意，上面没有显示的是`getInstance`、`load, `和`store`各自抛出的几个检查过的异常。

## 4。加载密钥库

要加载密钥库，我们首先需要创建一个`KeyStore` 实例，就像前面一样。

不过，这一次，让我们指定格式，因为我们正在加载一个现有的格式:

```java
KeyStore ks = KeyStore.getInstance("JKS");
ks.load(new FileInputStream("newKeyStoreFileName.jks"), pwdArray);
```

如果我们的 JVM 不支持我们传递的 keystore 类型，或者如果它与我们正在打开的文件系统上的 keystore 类型不匹配，我们将得到一个`KeyStoreException`:

```java
java.security.KeyStoreException: KEYSTORE_TYPE not found
```

同样，如果密码是错误的，我们将得到一个`UnrecoverableKeyException:`

```java
java.security.UnrecoverableKeyException: Password verification failed
```

## 5。存储条目

在密钥库中，我们可以存储三种不同的条目，每个条目都有自己的别名:

*   对称密钥(在 JCE 中称为秘密密钥)，
*   非对称密钥(在 JCE 称为公钥和私钥)，以及
*   可信证书

让我们来看看每一个。

### 5.1。保存对称密钥

我们可以在密钥库中存储的最简单的东西是对称密钥。

要保存对称密钥，我们需要三样东西:

1.  **别名**–这只是我们将来用来指代条目的名字
2.  **一把钥匙**——被包裹在`KeyStore.SecretKeyEntry`里。
3.  **一个密码**——它被包在一个叫做`ProtectionParam`的东西里。

```java
KeyStore.SecretKeyEntry secret
 = new KeyStore.SecretKeyEntry(secretKey);
KeyStore.ProtectionParameter password
 = new KeyStore.PasswordProtection(pwdArray);
ks.setEntry("db-encryption-secret", secret, password);
```

**请记住，密码不能是`null, `然而，它可以是空的** `**String.** `如果我们将密码`null `留给一个条目，我们将得到一个`KeyStoreException:`

```java
java.security.KeyStoreException: non-null password required to create SecretKeyEntry
```

我们需要将密钥和密码包装在包装类中，这似乎有点奇怪。

我们包装这个键是因为`setEntry`是一个通用方法，也可以用于其他条目类型。条目的类型允许`KeyStore` API 对其进行不同的处理。

我们包装密码是因为`KeyStore ` API 支持对 GUI 和 CLI 的回调，以便从最终用户那里收集密码。查看[`KeyStore``.CallbackHandlerProtection`Javadoc](https://web.archive.org/web/20220625174447/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/KeyStore.CallbackHandlerProtection.html)了解更多详情。

我们也可以使用这个方法来更新一个现有的键。我们只需要使用相同的别名和密码以及新的`secret.`再次调用它

### 5.2。保存私钥

存储非对称密钥有点复杂，因为我们需要处理证书链。

另外，`KeyStore ` API 给了我们一个叫做`setKeyEntry `的专用方法，它比通用的`setEntry `方法更方便。

因此，要保存非对称密钥，我们需要四样东西:

1.  **别名**，和以前一样
2.  **一个私钥**。因为我们没有使用泛型方法，所以键不会被包装。同样，对于我们的例子，它应该是`PrivateKey`的一个实例
3.  用于访问条目的密码。这一次，密码是强制性的
4.  **认证相应公钥的证书链**

```java
X509Certificate[] certificateChain = new X509Certificate[2];
chain[0] = clientCert;
chain[1] = caCert;
ks.setKeyEntry("sso-signing-key", privateKey, pwdArray, certificateChain);
```

当然，这里很多地方会出错，比如说`pwdArray`是`null`:

```java
java.security.KeyStoreException: password can't be null
```

但是，有一个非常奇怪的例外需要注意，那就是如果`pwdArray` 是一个空数组:

```java
java.security.UnrecoverableKeyException: Given final block not properly padded
```

为了更新，我们可以简单地使用相同的别名和新的`privateKey` 和`certificateChain.`再次调用该方法

此外，快速复习一下`[how to generate a certificate chain](https://web.archive.org/web/20220625174447/https://support.sas.com/documentation/cdl/en/secref/69831/HTML/default/viewer.htm#p0gy97oedcx0fin1n83srxchqpzk.htm).`可能很有价值

### 5.3。保存可信证书

**存储可信证书非常简单。** **它只需要别名和证书本身**，证书类型为`Certificate`:

```java
ks.setCertificateEntry("google.com", trustedCertificate);
```

通常，证书不是我们生成的，而是来自第三方。

因此，这里需要注意的是`KeyStore`实际上并不验证这个证书。存放之前我们应该自己验证一下。

为了更新，我们可以简单地使用相同的别名和新的`trustedCertificate`再次调用该方法。

## 6。阅读条目

既然我们已经写了一些条目，我们当然想读它们。

### 6.1。读取单个条目

首先，我们可以通过别名提取密钥和证书:

```java
Key ssoSigningKey = ks.getKey("sso-signing-key", pwdArray);
Certificate google = ks.getCertificate("google.com");
```

如果没有那个名称的条目或者它是不同的类型，那么`getKey `简单地返回`null`:

```java
public void whenEntryIsMissingOrOfIncorrectType_thenReturnsNull() {
    // ... initialize keystore
    // ... add an entry called "widget-api-secret"

   Assert.assertNull(ks.getKey("some-other-api-secret"));
   Assert.assertNotNull(ks.getKey("widget-api-secret"));
   Assert.assertNull(ks.getCertificate("widget-api-secret")); 
}
```

但是，如果密匙的密码是错误的，**我们将会得到我们之前讨论过的那个奇怪的错误:**

```java
java.security.UnrecoverableKeyException: Given final block not properly padded
```

### 6.2。检查密钥库是否包含别名

因为`KeyStore`只是使用一个`Map`来存储条目，所以它公开了检查条目是否存在的能力，而无需检索条目:

```java
public void whenAddingAlias_thenCanQueryWithoutSaving() {
    // ... initialize keystore
    // ... add an entry called "widget-api-secret"
```

```java
 assertTrue(ks.containsAlias("widget-api-secret"));
    assertFalse(ks.containsAlias("some-other-api-secret"));
}
```

### 6.3。检查条目的种类

或者，`KeyStore` `#entryInstanceOf`更厉害一点。

类似于`containsAlias`，除了它还检查条目类型:

```java
public void whenAddingAlias_thenCanQueryByType() {
    // ... initialize keystore
    // ... add a secret entry called "widget-api-secret"
```

```java
 assertTrue(ks.containsAlias("widget-api-secret"));
    assertFalse(ks.entryInstanceOf(
      "widget-api-secret",
      KeyType.PrivateKeyEntry.class));
}
```

## 7。删除条目

`KeyStore`当然，支持删除我们已经添加的条目:

```java
public void whenDeletingAnAlias_thenIdempotent() {
    // ... initialize a keystore
    // ... add an entry called "widget-api-secret"
```

```java
 assertEquals(ks.size(), 1);
```

```java
 ks.deleteEntry("widget-api-secret");
    ks.deleteEntry("some-other-api-secret");
```

```java
 assertFalse(ks.size(), 0);
}
```

幸运的是，`deleteEntry `是幂等的，所以无论条目存在与否，方法的反应都是一样的。

## 8。删除密钥库

如果我们想删除我们的密钥库，API 对我们没有帮助，但是我们仍然可以使用 Java 来完成:

```java
Files.delete(Paths.get(keystorePath));
```

或者，作为替代，我们可以保留密钥库，只删除条目:

```java
Enumeration<String> aliases = keyStore.aliases();
while (aliases.hasMoreElements()) {
    String alias = aliases.nextElement();
    keyStore.deleteEntry(alias);
}
```

## 9。结论

在本文中，我们讨论了使用`KeyStore API`管理证书和密钥。我们讨论了什么是密钥库，如何创建、加载和删除密钥库，如何在密钥库中存储密钥或证书，以及如何用新值加载和更新现有条目。

这个例子的完整实现可以在 Github 的[中找到。](https://web.archive.org/web/20220625174447/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)