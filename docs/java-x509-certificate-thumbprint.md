# 用 Java 计算 X509 证书的指纹

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-x509-certificate-thumbprint>

## 1.概观

证书的**拇指纹(或指纹)是** **证书的唯一标识**。不是证书的一部分，但是是从中计算出来的。

在这个简短的教程中，我们将看到如何用 Java 计算 X509 证书的指纹。

## 2.使用普通 Java

首先，让我们从证书文件中获取一个`X509Certificate`对象:

```
public static X509Certificate getCertObject(String filePath) 
  throws IOException, CertificateException {
     try (FileInputStream is = new FileInputStream(filePath)) {
        CertificateFactory certificateFactory = CertificateFactory
          .getInstance("X.509");
        return (X509Certificate) certificateFactory.generateCertificate(is);
    }
}
```

接下来，让我们从这个对象获取指纹:

```
private static String getThumbprint(X509Certificate cert) 
  throws NoSuchAlgorithmException, CertificateEncodingException {
    MessageDigest md = MessageDigest.getInstance("SHA-1");
    md.update(cert.getEncoded());
    return DatatypeConverter.printHexBinary(md.digest()).toLowerCase();
}
```

例如，如果我们有一个名为`baeldung.pem`的 X509 证书文件，我们可以使用上面的方法轻松地打印它的指纹:

```
X509Certificate certObject = getCertObject("baeldung.pem");
System.out.println(getThumbprint(certObject));
```

结果将类似于:

```
c9fa9f008655c8401ad27e213b985804854d928c
```

## 3.使用 Apache Commons 编解码器

我们也可以使用来自 [Apache Commons Codec](https://web.archive.org/web/20221209160638/https://search.maven.org/search?q=g:commons-codec) 库的`DigestUtils`类来实现同样的目标。

让我们给我们的`pom.xml`文件添加一个依赖项:

```
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

现在，我们简单地使用`sha1Hex()`方法从我们的`X509Certificate`对象中获取指纹:

```
DigestUtils.sha1Hex(certObject.getEncoded());
```

## 4.结论

在这个快速教程中，我们学习了用 Java 计算 X509 证书指纹的两种方法。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221209160638/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-3)