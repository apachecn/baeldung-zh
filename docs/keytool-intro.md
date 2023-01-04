# keytool 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/keytool-intro>

## 1。概述

在这个简短的教程中，我们将介绍`[keytool](https://web.archive.org/web/20221113135035/https://docs.oracle.com/en/java/javase/11/tools/keytool.html)`命令。我们将学习如何使用`keytool`创建一个新的证书并检查该证书的信息。

## 2。`keytool?`什么是

Java 在其版本中包含了`keytool`实用程序。我们用它来**管理** **密钥和证书**，并将它们存储在密钥库中。`keytool`命令允许我们创建自签名证书并显示关于密钥库的信息。

在接下来的几节中，我们将介绍该实用程序的不同功能。

## 3。创建自签名证书

首先，让我们创建一个自签名证书，它可以用来在我们的开发环境中的项目之间建立安全的通信。

为了让**生成证书**，我们将打开一个命令行提示符并使用带有`-genkeypair`选项的`keytool`命令:

```
keytool -genkeypair -alias <alias> -keypass <keypass> -validity <validity> -storepass <storepass>
```

让我们进一步了解这些参数:

*   `alias`–我们证书的名称
*   `keypass`–证书的密码。我们需要这个密码来访问我们证书的私钥
*   我们证书的有效时间(天数)
*   `storepass`–密钥库的密码。如果存储不存在，这将是密钥库的密码

例如，让我们生成一个名为`“cert1”`的证书，它的私钥为`“pass123”`，有效期为一年。我们还将指定`“stpass123”`作为密钥库密码:

```
keytool -genkeypair -alias cert1 -keypass pass123 -validity 365 -storepass stpass123
```

执行该命令后，它会要求我们提供一些信息:

```
What is your first and last name?
  [Unknown]:  Name
What is the name of your organizational unit?
  [Unknown]:  Unit
What is the name of your organization?
  [Unknown]:  Company
What is the name of your City or Locality?
  [Unknown]:  City
What is the name of your State or Province?
  [Unknown]:  State
What is the two-letter country code for this unit?
  [Unknown]:  US
Is CN=Name, OU=Unit, O=Company, L=City, ST=State, C=US correct?
  [no]:  yes
```

如上所述，如果我们以前没有创建密钥库，那么创建这个证书将会自动创建它。

我们也可以不带参数地执行`-genkeypair`选项。如果我们不在命令行中提供它们，而它们是必需的，我们会被提示输入它们。

**注意，一般建议不要在生产环境**的命令行上提供密码(`-keypass`或 `-storepass`)。

## 4。列出密钥库中的证书

接下来，我们将学习如何**查看存储在我们的密钥库中的证书**。为此，我们将使用`-list`选项:

```
keytool -list -storepass <storepass> 
```

所执行命令的输出将显示我们创建的证书:

```
Keystore type: JKS
Keystore provider: SUN

Your keystore contains 1 entry

cert1, 02-ago-2020, PrivateKeyEntry, 
Certificate fingerprint (SHA1): 0B:3F:98:2E:A4:F7:33:6E:C4:2E:29:72:A7:17:E0:F5:22:45:08:2F
```

如果我们想要获得具体证书的**信息，我们只需要在命令中包含` -alias`选项。为了获得比默认提供的更多的信息，我们还将添加`-v` (verbose)选项:**

```
keytool -list -v -alias <alias> -storepass <storepass> 
```

这将为我们提供与请求的证书相关的所有信息:

```
Alias name: cert1
Creation date: 02-ago-2020
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=Name, OU=Unit, O=Company, L=City, ST=State, C=US
Issuer: CN=Name, OU=Unit, O=Company, L=City, ST=State, C=US
Serial number: 11d34890
Valid from: Sun Aug 02 20:25:14 CEST 2020 until: Mon Aug 02 20:25:14 CEST 2021
Certificate fingerprints:
	 MD5:  16:F8:9B:DF:2C:2F:31:F0:85:9C:70:C3:56:66:59:46
	 SHA1: 0B:3F:98:2E:A4:F7:33:6E:C4:2E:29:72:A7:17:E0:F5:22:45:08:2F
	 SHA256: 8C:B0:39:9F:A4:43:E2:D1:57:4A:6A:97:E9:B1:51:38:82:0F:07:F6:9E:CE:A9:AB:2E:92:52:7A:7E:98:2D:CA
Signature algorithm name: SHA256withDSA
Subject Public Key Algorithm: 2048-bit DSA key
Version: 3

Extensions: 

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: A1 3E DD 9A FB C0 9F 5D   B5 BE 2E EC E2 87 CD 45  .>.....].......E
0010: FE 0B D7 55                                        ...U
]
]
```

## 5。其他功能

除了我们已经看到的功能之外，这个工具中还有许多[附加功能](https://web.archive.org/web/20221113135035/https://docs.oracle.com/en/java/javase/11/tools/keytool.html)。

例如，我们可以**删除我们从密钥库中创建的证书**:

```
keytool -delete -alias <alias> -storepass <storepass>
```

另一个例子是，我们甚至能够**改变证书**的别名:

```
keytool -changealias -alias <alias> -destalias <new_alias> -keypass <keypass> -storepass <storepass>
```

最后，为了获得关于该工具的更多信息，我们可以**通过命令行请求帮助**:

```
keytool -help
```

## 6。结论

在这个快速教程中，我们学习了一些关于`keytool`实用程序的知识。我们还学会了使用该工具中包含的一些基本功能。