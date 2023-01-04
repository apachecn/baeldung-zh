# Jasypt 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jasypt>

 ![](img/b808e9503a721fd8002f00c4cc71c674.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524032115/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在本文中，我们将关注`[Jasypt](https://web.archive.org/web/20220524032115/http://www.jasypt.org/index.html)` (Java 简化加密)库。

Jasypt 是一个 Java 库，它允许开发人员以最少的努力向项目添加基本的加密功能，并且不需要深入了解加密协议的实现细节。

## 2。使用简单加密

假设我们正在构建一个 web 应用程序，用户在其中提交一个帐户私有数据。我们需要将数据存储在数据库中，但是存储纯文本是不安全的。

处理它的一种方法是在数据库中存储一个加密的数据，并在特定用户检索该数据时对其进行解密。

要使用非常简单的算法执行加密和解密，我们可以使用来自 Jasypt 库的`[BasicTextEncryptor](https://web.archive.org/web/20220524032115/http://www.jasypt.org/api/jasypt/1.8/org/jasypt/util/text/BasicTextEncryptor.html)` 类:

```java
BasicTextEncryptor textEncryptor = new BasicTextEncryptor();
String privateData = "secret-data";
textEncryptor.setPasswordCharArray("some-random-data".toCharArray());
```

然后我们可以使用一个`encrypt()` 方法来加密纯文本:

```java
String myEncryptedText = textEncryptor.encrypt(privateData);
assertNotSame(privateData, myEncryptedText);
```

如果我们想在数据库中存储给定用户的私有数据，我们可以存储一个`myEncryptedText` 而不违反任何安全限制。如果我们想将数据解密回纯文本，我们可以使用一种`decrypt()` 方法:

```java
String plainText = textEncryptor.decrypt(myEncryptedText);

assertEquals(plainText, privateData);
```

我们看到解密的数据等于之前加密的纯文本数据。

## 3。单向加密

前面的例子不是执行身份验证的理想方式，也就是当我们想要存储用户密码时。理想情况下，我们希望加密密码，但没有办法解密。当用户试图登录我们的服务时，我们会加密他的密码，并将其与存储在数据库中的加密密码进行比较。这样我们就不需要操作明文密码了。

我们可以使用一个`BasicPasswordEncryptor` 类来执行单向加密:

```java
String password = "secret-pass";
BasicPasswordEncryptor passwordEncryptor = new BasicPasswordEncryptor();
String encryptedPassword = passwordEncryptor.encryptPassword(password); 
```

然后，我们可以将已经加密的密码与执行登录过程的用户的密码进行比较，而无需解密已经存储在数据库中的密码:

```java
boolean result = passwordEncryptor.checkPassword("secret-pass", encryptedPassword);

assertTrue(result);
```

## 4。配置加密算法

我们可以使用更强的加密算法，但是我们需要记住为我们的 JVM 安装[Java Cryptography Extension(JCE)无限强度管辖策略文件](https://web.archive.org/web/20220524032115/http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)(安装说明包含在下载中)。

在 Jasypt 中，我们可以通过使用一个`StandardPBEStringEncryptor` 类来使用强加密，并使用一个`setAlgorithm()` 方法来定制它:

```java
StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
String privateData = "secret-data";
encryptor.setPassword("some-random-passwprd");
encryptor.setAlgorithm("PBEWithMD5AndTripleDES");
```

让我们将加密算法设置为`PBEWithMD5AndTripleDES.`

接下来，加密和解密的过程看起来与前面使用`BasicTextEncryptor` 类的过程相同:

```java
String encryptedText = encryptor.encrypt(privateData);
assertNotSame(privateData, encryptedText);

String plainText = encryptor.decrypt(encryptedText);
assertEquals(plainText, privateData);
```

## 5。使用多线程解密

当我们在多核机器上运行时，我们希望并行处理解密。为了获得良好的性能，我们可以使用一个`[PooledPBEStringEncryptor](https://web.archive.org/web/20220524032115/http://www.jasypt.org/api/jasypt/1.9.0/org/jasypt/encryption/pbe/PooledPBEStringEncryptor.html)` 和`setPoolSize()` API 来创建一个摘要池。它们中的每一个都可以被不同的线程并行使用:

```java
PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
encryptor.setPoolSize(4);
encryptor.setPassword("some-random-data");
encryptor.setAlgorithm("PBEWithMD5AndTripleDES");
```

将池大小设置为等于机器的核心数量是一个很好的做法。加密和解密的代码与之前的代码相同。

## 6。在其他框架中的使用

最后一点需要注意的是，`Jasypt` 库可以与许多其他库集成，当然包括`Spring`框架。

我们只需要创建一个配置，将加密支持添加到我们的 Spring 应用程序中。如果我们想要将敏感数据存储到数据库中，并且我们使用`Hibernate` 作为数据访问框架，我们也可以将`Jasypt`与它集成。

关于这些集成以及其他一些框架的说明可以在[jassypt 主页](https://web.archive.org/web/20220524032115/http://www.jasypt.org/)的`Guides`部分找到。

## 7。结论

在本文中，我们看到了`Jasypt` 库，它通过使用一个已经众所周知且经过测试的加密算法，帮助我们创建更安全的应用程序。它包含了易于使用的简单 API。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220524032115/https://github.com/eugenp/tutorials/tree/master/libraries-security)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。