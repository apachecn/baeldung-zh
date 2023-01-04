# 将 PEM 文件转换为 Java 密钥库格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-pem-to-jks>

## 1.概观

在之前的教程中，我们展示了如何[将 Java 密钥库(JKS)转换成 PEM 格式](/web/20220628123539/https://www.baeldung.com/java-keystore-convert-to-pem-format)。在本教程中，我们将把 PEM 格式转换成标准的 Java 密钥库(JKS)格式。Java 密钥库是一个存储证书及其匹配私钥的容器。

我们将使用`keytool`和`openssl`命令的组合来从 PEM 转换到 JKS。`keytool`命令来自 JDK (Java 开发工具包),用于从 PEM 转换到 PKCS12。第二个命令`openssl`，需要[下载](https://web.archive.org/web/20220628123539/https://www.openssl.org/source/)，作用是从 PKCS12 转换到 JKS。

## 2.文件格式

JKS 是一种特定于 Java 的文件格式，在 Java 8 之前是密匙库的默认格式。从 Java 9 开始， **PKCS#12 是默认的密钥库格式**。除了 JKS，PKCS#12 是一种存储加密数据的标准化和语言中立的格式。PKCS#12 格式也称为 PKCS12 或 PFX。

**PEM(隐私增强邮件)也是一种证书容器格式。**PEM 文件以 Base64 编码。这确保了数据在不同系统之间的转换过程中保持完整。

此外，PEM 文件可以包含一个或多个实例，每个实例由纯文本页眉和页脚分隔:

```java
-----BEGIN CERTIFICATE-----

// base64 encoded

-----END CERTIFICATE-----
```

## 3.将 PEM 转换为 JKS 格式

我们现在将完成将所有证书和私钥从 PEM 格式转换为 JKS 格式的步骤。

出于示例的目的，我们将创建一个自签名证书。

### 3.1.创建 PEM 文件

**我们首先使用`openssl`** 生成两个文件`key.pem`和`cert.pem`:

```java
openssl req -newkey rsa:2048 -x509 -keyout key.pem -out cert.pem -days 365 
```

该工具将提示我们输入 PEM 密码和其他信息。

一旦我们回答了所有的提示，`openssl`工具输出两个文件:

*   `key.pem`(私钥)
*   `cert.pem`(公开证书)

**我们将使用这些文件生成我们的自签名证书**。

### 3.2.正在生成 PKCS12 证书

在大多数情况下，证书采用公钥加密标准#12 (PKCS12)格式。我们不经常使用 Java 密钥库(JKS)格式。

让我们**将 PEM 转换成 PKCS12 格式**:

```java
openssl pkcs12 -export -in cert.pem -inkey key.pem -out certificate.p12 -name "certificate"
```

命令运行时，系统会提示我们输入之前为 `key.pem`创建的密码:

```java
Enter pass phrase for key.pem:
```

然后我们会看到提示要求输入`certificate.p12`的新密码:

```java
Enter Export Password:
```

之后，我们将拥有一个以 PCKS12 格式存储的`certificate.p12`密钥库。

### 3.3.PKCS 12 号到 JKS

最后一步是将 PKCS12 转换为 JKS 格式:

```java
keytool -importkeystore -srckeystore certificate.p12 -srcstoretype pkcs12 -destkeystore cert.jks
```

当命令执行时，它会提示为` cert.jks`文件输入一个新密码:

```java
Enter destination keystore password:
```

它会提示我们输入之前创建的`certificate.p12`密码:

```java
Enter source keystore password:
```

然后，我们应该会看到最终的输出:

```java
Entry for alias certificate successfully imported.
Import command completed: 1 entries successfully imported, 0 entries failed or cancelled
```

结果是一个以 JKS 格式存储的`cert.jks`密钥库。

## 4.结论

在本文中，我们描述了借助 PKCS12 中间格式将 PEM 文件转换为 JKS 格式的步骤。

作为帮助工具，我们使用了`keytool`和`openssl`命令。