# 获取 Java 中可信证书的列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-trusted-certificates>

## 1.概观

在这个快速教程中，我们将通过快速实用的例子来学习如何阅读 Java 中的可信证书列表。

## 2.加载`KeyStore`

Java 将可信证书存储在一个名为`cacerts`的特殊文件中，该文件位于我们的 Java 安装文件夹中。

让我们从读取这个文件并将其加载到 [`KeyStore`](/web/20221206113225/https://www.baeldung.com/java-keystore#what-is-a-keystore) 开始:

```java
private KeyStore loadKeyStore() {
    String relativeCacertsPath = "/lib/security/cacerts".replace("/", File.separator);
    String filename = System.getProperty("java.home") + relativeCacertsPath;
    FileInputStream is = new FileInputStream(filename);

    KeyStore keystore = KeyStore.getInstance(KeyStore.getDefaultType());
    String password = "changeit";
    keystore.load(is, password.toCharArray());

    return keystore;
}
```

**这个`KeyStore`的默认密码是`“changeit”`，但是如果它之前在我们的系统中被改变，它可能是不同的。**

一旦加载完毕，`KeyStore`将保存我们的可信证书，接下来，我们将看到如何读取它们。

## 3.从指定的`KeyStore`读取证书

我们将使用`[PKIXParameters](https://web.archive.org/web/20221206113225/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/cert/PKIXParameters.html)`类，它接受一个`KeyStore`作为构造函数参数:

```java
@Test
public void whenLoadingCacertsKeyStore_thenCertificatesArePresent() {
    KeyStore keyStore = loadKeyStore();
    PKIXParameters params = new PKIXParameters(keyStore);

    Set<TrustAnchor> trustAnchors = params.getTrustAnchors();
    List<Certificate> certificates = trustAnchors.stream()
      .map(TrustAnchor::getTrustedCert)
      .collect(Collectors.toList());

    assertFalse(certificates.isEmpty());
}
```

`PKIXParameters`类通常用于验证证书，但是在我们的例子中，我们只是用它从我们的`KeyStore`中获取证书。

当创建一个`PKIXParametrs`的实例时，它会构建一个`[TrustAnchor](https://web.archive.org/web/20221206113225/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/cert/TrustAnchor.html)`列表，该列表将包含我们的`KeyStore`中存在的可信证书。

一个`TrustAnchor`实例仅仅代表一个可信的证书。

## 4.从默认位置读取证书`KeyStore`

我们还可以通过**使用`TrustManagerFactory`类并在没有`KeyStore`** 的情况下初始化它来获得系统中存在的可信证书的列表，这将使用默认的`KeyStore`。

如果我们没有明确提供一个`KeyStore`,默认情况下将使用前一章中的同一个:

```java
@Test
public void whenLoadingDefaultKeyStore_thenCertificatesArePresent() {
    TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
    trustManagerFactory.init((KeyStore) null);

    List<TrustManager> trustManagers = Arrays.asList(trustManagerFactory.getTrustManagers());
    List<X509Certificate> certificates = trustManagers.stream()
      .filter(X509TrustManager.class::isInstance)
      .map(X509TrustManager.class::cast)
      .map(trustManager -> Arrays.asList(trustManager.getAcceptedIssuers()))
      .flatMap(Collection::stream)
      .collect(Collectors.toList());

    assertFalse(certificates.isEmpty());
}
```

在上面的例子中，我们使用了`X509TrustManager`，它是一个专门的`TrustManager`，用于[认证 SSL 连接](/web/20221206113225/https://www.baeldung.com/x-509-authentication-in-spring-security)的远程部分。

注意，这种行为可能取决于特定的 JDK 实现，因为[规范](https://web.archive.org/web/20221206113225/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/net/ssl/TrustManagerFactory.html#init(java.security.KeyStore))没有定义在`init()` `KeyStore`参数为`null`的情况下应该发生什么。

## 5.证书别名

证书别名就是唯一标识证书的`String`。

**在 Java 导入的默认证书中，还有一个由公共互联网域名注册商 GoDaddy 发布的著名证书，我们将在测试中使用它:**

```java
String GODADDY_CA_ALIAS = "godaddyrootg2ca [jdk]";
```

让我们看看如何读取`KeyStore`中的所有证书别名:

```java
@Test
public void whenLoadingKeyStore_thenGoDaddyCALabelIsPresent() {
    KeyStore keyStore = loadKeyStore();

    Enumeration<String> aliasEnumeration = keyStore.aliases();
    List<String> aliases = Collections.list(aliasEnumeration);
    assertTrue(aliases.contains(GODADDY_CA_ALIAS));
}
```

在下一个示例中，我们将了解如何通过别名检索证书:

```java
@Test
public void whenLoadingKeyStore_thenGoDaddyCertificateIsPresent() {
    KeyStore keyStore = loadKeyStore();

    Certificate goDaddyCertificate = keyStore.getCertificate(GODADDY_CA_ALIAS);
    assertNotNull(goDaddyCertificate);
}
```

## 6.结论

在这篇简短的文章中，我们通过快速实用的例子，研究了在 Java 中列出可信证书的不同方式。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221206113225/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2)