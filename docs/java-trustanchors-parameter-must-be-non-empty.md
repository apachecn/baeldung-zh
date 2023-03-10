# 错误:“trustAnchors 参数必须非空”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-trustanchors-parameter-must-be-non-empty>

## 1.概观

在本教程中，我们将解释什么是信任锚。此外，我们将展示一个`TrustStore`的默认位置和预期的文件格式。最后，我们将阐明错误的原因:"`java.security.InvalidAlgorithmParameterException`:信任锚参数必须非空"。

## 2.信任锚定义

先解释一下[信任锚](https://web.archive.org/web/20220907104613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/security/cert/TrustAnchor.html)是什么？**在密码系统中，信任锚定义了信任被假设和导出的根实体**。在像 X.509 这样的架构中，根证书是一个信任锚。此外，根证书保证信任链中的所有其他证书。

## 3.`TrustStore`位置和格式

现在让我们看看 Java 中的`[TrustStore](/web/20220907104613/https://www.baeldung.com/java-keystore-truststore-difference#java-truststore)`位置和格式。首先，Java 在两个位置寻找`TrustStore`(按顺序):

*   `$JAVA_HOME/lib/security/jssecacerts`
*   `$JAVA_HOME/lib/security/cacerts`

我们可以用参数`-Djavax.net.ssl.trustStore.`覆盖默认位置

此外，参数`-Djavax.net.ssl.trustStorePassword `允许我们向`TrustStore`提供一个密码。最后，该命令如下所示:

```java
java -Djavax.net.ssl.trustStore=/some/loc/on/server/ our_truststore.jks -Djavax.net.ssl.trustStorePassword=our_password -jar application.jar
```

而且， [JKS](/web/20220907104613/https://www.baeldung.com/convert-pem-to-jks#file-formats) 是默认的`TrustStore`格式。参数`-Djavax.net.ssl.trustStoreType`允许覆盖默认的`TrustStore`类型。

让我们看看为`$JAVA_HOME/lib/security/cacerts`执行的 Java 16 中的`keytool`实用程序的输出:

```java
$ keytool -list -cacerts
Enter keystore password:
Keystore type: JKS
Keystore provider: SUN

Your keystore contains 90 entries
....
```

不出所料，`KeyStore`型是 JKS。此外，我们在文件中存储了所有 90 个证书。

## 4.例外的原因

现在让我们来看看异常“`java.security.InvalidAlgorithmParameterException` : trustAnchors 参数必须非空”。

首先，Java 运行时只在`[PKIXParameters](/web/20220907104613/https://www.baeldung.com/java-list-trusted-certificates#reading-certificates-from-a-specified-keystore)`类中创建 [`InvalidAlgorithmParameterException`](https://web.archive.org/web/20220907104613/https://cr.openjdk.java.net/~iris/se/11/latestSpec/api/java.base/java/security/InvalidAlgorithmParameterException.html) ，用于从`KeyStore`中读取证书。**`PKIXParameters` 的构造函数从作为参数给出的 K `eyStore`** 中收集`trustAnchors` 。

**当提供的`KeyStore` 没有** `**trustAnchors**`时抛出异常:

```java
...
if (trustAnchors.isEmpty()) {
    throw new InvalidAlgorithmParameterException("the trustAnchors " +
        "parameter must be non-empty");
}
...
```

**我们来试着重现一下案例。首先，让我们创建一个空的 K `eyStore`** :

```java
private KeyStore getKeyStore() throws CertificateException, NoSuchAlgorithmException, IOException, KeyStoreException {
    KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
    ks.load(null, "changeIt".toCharArray());
    return ks;
}
```

现在让我们测试一下`PKIXParameters` 类的实例化:

```java
@Test
public void whenOpeningTrustStore_thenExceptionIsThrown() throws Exception {
    KeyStore keyStore = getKeyStore();
    InvalidAlgorithmParameterException invalidAlgorithmParameterException =
      Assertions.assertThrows(InvalidAlgorithmParameterException.class, () -> new PKIXParameters(keyStore));
    Assertions.assertEquals("the trustAnchors parameter must be non-empty", invalidAlgorithmParameterException.getMessage());
}
```

也就是说，构造函数按照预期抛出了异常。换句话说，当给定的`KeyStore`中没有可信证书时，不可能创建`PKIXParameters` 类的实例。

## 5.结论

在这篇短文中，我们描述了什么是信任锚。然后，我们展示了默认的`TrustStore`位置和文件格式。最后，我们展示了“信任锚参数必须非空”错误的原因。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220907104613/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-3)