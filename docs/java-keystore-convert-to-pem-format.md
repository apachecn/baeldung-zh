# 将 Java 密钥库转换为 PEM 格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-keystore-convert-to-pem-format>

 ![announcement - icon](img/01b326ce62f014f5f9d6db6f3b8a0e49.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220525133944/https://www.baeldung.com/lightrun-n-security)

## 1.介绍

Java 密钥库是一个安全证书的容器，我们可以在编写 Java 代码时使用它。Java 密钥库持有一个或多个证书及其匹配的私钥，并使用 JDK 附带的 [`keytool`](/web/20220525133944/https://www.baeldung.com/keytool-intro) 创建。

**在本教程中，我们将使用`keytool`和`openssl.`** 的组合将 Java 密钥库转换为 PEM(隐私增强邮件)格式，步骤包括使用`keytool`将 JKS 转换为 PKCS#12 密钥库，然后使用 [`openssl`](/web/20220525133944/https://www.baeldung.com/linux/ssl-certificates#using-openssl) 将 PKCS#12 密钥库转换为 PEM 文件。

`keytool`可从 JDK 获得，我们可以从 OpenSSL 网站[下载`openssl`](https://web.archive.org/web/20220525133944/https://www.openssl.org/source/) 。

## 2.文件格式

Java 密钥库以 JKS 文件格式存储。这是一种专用于 Java 程序的专有格式。PKCS#12 密钥库是非专有的，并且越来越受欢迎——从 Java 9 开始，PKCS#12 被用作 JKS 上的默认密钥库格式。

PEM 文件也是证书容器，它们使用 Base64 编码二进制数据，这使得内容可以更容易地通过不同的系统传输。一个 PEM 文件可能包含多个实例，每个实例遵循两条规则:

*   `-----BEGIN <label>-----`的单行标题
*   `-----END <label>-----`的单行页脚

`<label>`指定编码信息的类型，常用值为`CERTIFICATE` 和`PRIVATE KEY`。

## 3.将整个 JKS 转换为 PEM 格式

现在让我们来看一下将所有证书和私钥从 JKS 转换成 PEM 格式的步骤。

### 3.1.创建 Java 密钥库

我们首先用一个 RSA 密钥对创建一个 JKS:

```java
keytool -genkey -keyalg RSA -v -keystore keystore.jks -alias first-key-pair
```

我们将在提示符下输入一个密钥库密码，并输入关于密钥对的信息。

对于本例，我们还将创建第二个密钥对:

```java
keytool -genkey -keyalg RSA -v -keystore keystore.jks -alias second-key-pair
```

### 3.2.JKS 至 PKCS#12

转换过程的第一步是使用`keytool`将 JKS 转换成 PKCS#12:

```java
keytool -importkeystore -srckeystore keystore.jks \
   -destkeystore keystore.p12 \
   -srcstoretype jks \
   -deststoretype pkcs12
```

同样，我们将回答密码提示—一个将要求输入原始 JKS 的密码，另一个将要求我们为生成的 PKCS#12 密钥库创建密码。

让我们检查运行该命令的输出:

```java
Entry for alias first-key-pair successfully imported.
Entry for alias second-key-pair successfully imported.
Import command completed:  2 entries successfully imported, 0 entries failed or cancelled
```

结果是一个以 PKCS#12 格式存储的`keystore.p12`密钥库。

### 3.3.PKCS#12 至 PEM

从这里开始，我们将使用`openssl`将`keystore.p12`编码成一个 PEM 文件:

```java
openssl pkcs12 -in keystore.p12 -out keystore.pem
```

该工具将提示我们为每个别名输入 PKCS#12 密钥库密码和 PEM 密码短语。**PEM 密码用于加密生成的私钥。**

如果我们不想加密生成的私钥，我们应该使用:

```java
openssl pkcs12 -nodes -in keystore.p12 -out keystore.pem
```

`keystore.pem`将包含密钥库中的所有密钥和证书。在这个例子中，它包含了一个私钥和一个针对`first-key-pair`和`second-key-pair`别名的证书。

## 4.将单个证书从 JKS 转换为 PEM

**我们可以单独使用`keytool`将单个公钥证书从 JKS 导出到 PEM 格式**:

```java
keytool -exportcert -alias first-key-pair -keystore keystore.jks -rfc -file first-key-pair-cert.pem
```

在提示符下输入 JKS 密码后，我们将看到该命令的输出:

```java
Certificate stored in file <first-key-pair-cert.pem>
```

## 5.结论

我们已经使用`keytool`、`openssl`和 PKCS#12 格式的中间阶段成功地将整个 JKS 转换成 PEM 格式。我们还讨论了单独使用`keytool`来转换单个公钥证书。