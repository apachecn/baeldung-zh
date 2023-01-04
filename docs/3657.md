# 如何将. cer 证书导入到 Java 密钥库中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-import-cer-certificate-into-keystore>

## 1。概述

顾名思义，密钥库基本上是证书、公钥和私钥的存储库。此外， **JDK 发行版附带了一个可执行文件来帮助管理它们， [`keytool`](/web/20221103142804/https://www.baeldung.com/keytool-intro) 。**

另一方面，证书可以有许多扩展，但是我们需要记住一个**。cer 文件包含公共 X.509 密钥，因此只能用于身份验证。**

在这篇短文中，我们将看看如何将一个`.cer`文件导入到 Java 密钥库中。

## 2。导入证书

事不宜迟，现在让我们将 Baeldung 公共证书文件导入到一个示例密钥库中。

**`keytool`有许多选项，但我们感兴趣的是`importcert`，它就像它的名字一样简单明了。**由于在一个密钥库中通常有不同的条目，我们将不得不使用`alias`参数为它分配一个惟一的名称:

```
> keytool -importcert -alias baeldung_public_cert -file baeldung.cer -keystore sample_keystore
> Enter keystore password:
...
> Trust this certificate? [no]:  y
> Certificate was added to keystore 
```

**尽管命令提示输入密码和确认信息，但我们可以通过添加`storepass`和`noprompt`参数**来绕过它们。这在从脚本运行`keytool`时尤其方便:

```
> keytool -importcert -alias baeldung_public_cert -file baeldung.cer -keystore sample_keystore -storepass pass123 -noprompt
> Certificate was added to keystore
```

此外，如果 KeyStore 不存在，它将自动生成。在这种情况下，**我们可以通过`storetype`参数来设置格式。如果没有指定，如果我们使用的是 Java 8 或更早版本，那么 KeyStore 格式默认为`JKS`。从 Java 9 开始，它默认为 PKCS12** :

```
> keytool -importcert -alias baeldung_public_cert -file baeldung.cer -keystore sample_keystore -storetype PKCS12
> Enter keystore password:
> Re-enter new password:
...
> Trust this certificate? [no]: y
> Certificate was added to keystore 
```

在这里，我们创建了一个 PKCS12 密钥库。JKS 和 PKCS12 的主要区别在于，JKS 是一种特定于 Java 的格式，而 PKCS12 是一种存储密钥和证书的标准化方式

如果我们需要，我们也可以通过编程来执行这些操作[。](/web/20221103142804/https://www.baeldung.com/java-keystore)

## 3。结论

在本教程中，我们介绍了如何在密钥库中导入. cer 文件。为了做到这一点，我们使用了`keytool's importcert`选项。