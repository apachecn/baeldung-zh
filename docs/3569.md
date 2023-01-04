# 数字证书:如何导入？cer 文件到信任库文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/import-cer-file-into-truststore>

## 1.概观

每当应用程序需要通过网络与客户端通信时， [SSL 协议](/web/20220628090540/https://www.baeldung.com/java-ssl)通常是首选。除了数据加密，SSL 还强制浏览器等应用程序在握手期间交换非对称密钥，以建立安全连接。

通常，应用程序共享 [X.509 证书](/web/20220628090540/https://www.baeldung.com/java-digital-signature)格式的非对称密钥。**因此，在 SSL 握手之前，客户端必须将这样的证书导入到它们的信任库文件中。**

在本文中，我们将讨论一些可以用来在中导入证书的工具。cer 格式加载到客户端的信任库中。

## 2.`keytool `命令

JDK 发行版提供了一个 [`keytool `实用程序](/web/20220628090540/https://www.baeldung.com/keytool-intro)，我们可以用它来管理 [Java 密钥库(JKS)](/web/20220628090540/https://www.baeldung.com/java-keystore) 。该命令最重要的目的是生成自签名的 X.509 证书，用于测试客户端和服务器之间的 SSL 通信。

**我们还可以将自签名或 CA 签名的证书导入到 JKS 文件中，并将其用作信任库**:

```
keytool -importcert -alias trustme -file baeldung.cer -keystore cacerts

Enter keystore password:

Trust this certificate? [no]:  yes
Certificate was added to keystore
```

这里，我们使用`keytool `命令导入了一个自签名的`baeldung.cer `证书。我们可以将这个证书导入到任何 Java 密钥库中。例如，**这里显示的是将证书添加到 JDK** 的`cacerts `密钥库中。

如果我们现在列出密钥库中的证书，我们将看到一个别名`trustme`:

```
keytool -list -keystore cacerts

trustme, Oct 31, 2020, trustedCertEntry,
Certificate fingerprint (SHA1): 04:40:6C:B0:06:65:EE:80:9A:90:A5:E9:DA:19:05:4A:AA:F2:CF:A4
```

## 3.`openssl` 命令

到目前为止，我们只讨论了将证书导入到 JKS 文件中。这样的密钥库只能用于 [Java 应用程序](/web/20220628090540/https://www.baeldung.com/java-ssl)。**如果我们必须用其他语言实现一个 SSL 库或者在多种语言平台上使用同一个证书，我们更有可能使用 PKCS12 密钥库**。

要将证书导入到 PKCS12 密钥库中，我们也可以使用`[openssl](https://web.archive.org/web/20220628090540/https://www.openssl.org/)` :

```
openssl pkcs12 -export -in baeldung.cer -inkey baeldung.key -out baeldung.keystore -name trustme
```

这个命令将名为`baeldung.cer `的证书导入到一个密钥库`baeldung.keystore `中，别名为`trustme. `

我们可以在密钥库中看到导入的证书:

```
openssl pkcs12 -info -in baeldung.keystore
Enter Import Password:
MAC: sha1, Iteration 2048
MAC length: 20, salt length: 8
PKCS7 Encrypted data: pbeWithSHA1And40BitRC2-CBC, Iteration 2048
Certificate bag
Bag Attributes
    friendlyName: trustme
    localKeyID: F4 36 4E 19 E4 E4 E7 65 74 56 FB 50 40 02 68 8B EC F0 4D B3
subject=C = IN, ST = DE, L = DC, O = BA, OU = AU, CN = baeldung.com

issuer=C = IN, ST = DE, L = DC, O = BA, OU = AU, CN = baeldung.com

-----BEGIN CERTIFICATE-----
MIIFkTCCA3mgAwIBAgIUL/OjGExnppeZkiNNh0i2+TPHaCQwDQYJKoZIhvcNAQEL
BQAwWDELMAkGA1UEBhMCSU4xCzAJBgNVBAgMAkRFMQswCQYDVQQHDAJEQzELMAkG
A1UECgwCQkExCzAJBgNVBAsMAkFVMRUwEwYDVQQDDAxiYWVsZHVuZy5jb20wHhcN
MjAxMTAzMTIwMjI5WhcNMjExMTAzMTIwMjI5WjBYMQswCQYDVQQGEwJJTjELMAkG
A1UECAwCREUxCzAJBgNVBAcMAkRDMQswCQYDVQQKDAJCQTELMAkGA1UECwwCQVUx
FTATBgNVBAMMDGJhZWxkdW5nLmNvbTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCC
AgoCggIBAK/XF/xmqQRJlTx2Vtq70x1KFwkHJEcZOyFbQP7O9RgicvMTAnbZtKpS
BSVjwroklIr4OVK4wmwdaTnlIm22CsFrbn+iBVL00tVs+sBYEcgO5nphVWGFbvHl
Q3PO4vTedSyH1qIyYrrhAn8wYvzdmr2g6tRwBX8K5H948Zb32Xbp5r9aR5M2i8Qz
fc0QasJUM5b71TNt8Qcsru3pFKj5hUMBTNrGCQrr6vrADTcG0YHuVSMeJId7f67h
l0vEY0BmRPnWNwGe+Sg/jqOWH9WWvkk/umkEQNWCQZaXZNZZ8jl5WMKFnmA7sPQ+
UjZPabNOTxhz6fJv5nJu7aMS/6tUWO0SdQ+ctO3HgR42wtBPoEOOuFMP6OqHI4hf
CXFTYg6aLwxFJP7LngfRvETgzVlsb9L/m++JBeoWRqpWaQUEgxDYJGFGA5dwQJaf
f24d042i44X0WqBBoWLjSQd/JFVH5MF17waiYpxFBOgpz3XEM/1j+juJPVut2k96
3ecgR54iKILbibizPUojn7t3AFT1Ug8exdefHdf+QsL8/L5+8/xOYkp/pnglQJJl
W0Lq4Sh9LWiux9XVdY6n2UYf/crgLSHatVkPa26cysdXhiiEPn4yYr2AdYVf0Xr5
W5PULufdi0HW2Eja/TfeXoBQtkdidqP8SMW+DwqafX80s37bZZBvAgMBAAGjUzBR
MB0GA1UdDgQWBBQPHIpCFhAy3kGAbzHpXMjXCMVQRzAfBgNVHSMEGDAWgBQPHIpC
FhAy3kGAbzHpXMjXCMVQRzAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUA
A4ICAQBzOK52I7lDX+7CHy6cQ8PnLZjAD4S5qC1P9dMy50E9N6Tmw2TiPbWl9CnU
7a/kVO6xDJDmNMnqRbHmlZclJaTFv6naXSX27PdIWjhwAfLjNa+FO9JNwMgiP25I
ISVjyrA3HwbhFnMs5FyBW9hbxfQ+X2Q2ooa+J3TKU7FImuDRKF3Sdb63+/j0go8S
5/TsoYpQxg86xbWf6IYGYwegd2SPSWUZ0HQSobZ7fRA7Y0EyPKgyqsBbmDtJ+X1g
P8Kep4N1oocc7ZkkX4pNfXTgXib9fUkKMAfRJz8w62z8I1OM61bciW7V2VSp/Y5p
iTihyuwO0aHG+YTnsr3qFrSFQLQUjCeBvx+euQelsGm8W9xM9YfASXiaEwCmb9PO
i/umD70J1e0HFDay9FW6mMoCCEBTZIF9ARqzhHgg9fi9iH2ctrsxadFAlOTFp5+/
p+nxrencfvc4CP6aHoqkE45HpMBoNDAxRMVd/FRzIG2as0q5At873MNFXP6WxmQV
4KGIhteNLyrXk82yHdHfm1EENTI7OEst/Fc8O3fFy9wn0OvoHIuCv1FVCq4Gnjmq
vNnBnGldrYruCNvj5KT6U14FFdK/5Zng0nSky4oMTs49zt392bdYi31SHWeATdw/
PscFRdig2stoI8ku6R+K7XvmseGzPmUW4K2IWU0zdRP2a4YyvA==
-----END CERTIFICATE-----
PKCS7 Data
Shrouded Keybag: pbeWithSHA1And3-KeyTripleDES-CBC, Iteration 2048
Bag Attributes
    friendlyName: trustme
    localKeyID: F4 36 4E 19 E4 E4 E7 65 74 56 FB 50 40 02 68 8B EC F0 4D B3
Key Attributes: <No Attributes>
```

因此，我们已经成功地将我们的证书导入到 PKCS12 密钥库中。因此，**这个密钥库现在可以在 SSL 客户端应用程序(如 HTTP 客户端库**)中用作信任库文件。同样，这个文件也可以用作 SSL 服务器应用程序(如 [Tomcat](/web/20220628090540/https://www.baeldung.com/tomcat) )中的密钥库。

## 4.结论

在本文中，我们讨论了两种流行的管理数字证书的 SSL 工具——OpenSSL 和 Java Keytool。**我们进一步使用了`keytool `和`openssl `命令来导入一个证书。cer 格式分别转换成 JKS 和 PKCS12 文件**。