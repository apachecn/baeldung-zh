# 使用 OpenSSL 创建自签名证书

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/openssl-self-signed-cert>

## 1.概观

OpenSSL 是一个开源命令行工具，允许用户执行各种与 SSL 相关的任务。

在本教程中，我们将学习**如何使用 OpenSSL** 创建自签名证书。

## 延伸阅读:

## [信任 OkHttp 中的自签名证书](/web/20220928195219/https://www.baeldung.com/okhttp-self-signed-cert)

Learn how to configure an OkHttpClient to trust self-signed certificates[Read more](/web/20220928195219/https://www.baeldung.com/okhttp-self-signed-cert) →

## [将 PEM 文件转换为 Java 密钥库格式](/web/20220928195219/https://www.baeldung.com/convert-pem-to-jks)

Learn how to convert certificates from a PEM (Privacy Enhanced Email) file to JKS (Java KeyStore) format using the `openssl` and `keytool` command-line utilities.[Read more](/web/20220928195219/https://www.baeldung.com/convert-pem-to-jks) →

## [HTTPS 在 Spring Boot 使用自签名证书](/web/20220928195219/https://www.baeldung.com/spring-boot-https-self-signed-certificate)

Explore how to generate a self-signed certificate to enable HTTPS in a Spring Boot application.[Read more](/web/20220928195219/https://www.baeldung.com/spring-boot-https-self-signed-certificate) →

## 2.创建私钥

首先，我们将创建一个私钥。私钥有助于实现加密，是我们证书中最重要的组成部分。

让我们用`openssl`命令创建一个受密码保护的 2048 位 RSA 私钥(`domain.key`):

```
openssl genrsa -des3 -out domain.key 2048 
```

出现提示时，我们将输入密码。输出将类似于:

```
Generating RSA private key, 2048 bit long modulus (2 primes)
.....................+++++
.........+++++
e is 65537 (0x010001)
Enter pass phrase for domain.key:
Verifying - Enter pass phrase for domain.key:
```

如果我们希望我们的私有密钥不加密，我们可以简单地从命令中删除`-des3`选项。

## 3.创建证书签名请求

**如果我们想要我们的证书被签署，我们需要一个证书签署请求(CSR)** 。CSR 包括公钥和一些附加信息(如组织和国家)。

让我们从现有的私钥创建一个 CSR ( `domain.csr`):

```
openssl req -key domain.key -new -out domain.csr
```

我们将输入我们的私钥密码和一些 CSR 信息来完成该过程。输出将类似于:

```
Enter pass phrase for domain.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:AU
State or Province Name (full name) [Some-State]:stateA                        
Locality Name (eg, city) []:cityA
Organization Name (eg, company) [Internet Widgits Pty Ltd]:companyA
Organizational Unit Name (eg, section) []:sectionA
Common Name (e.g. server FQDN or YOUR name) []:domain
Email Address []:[[email protected]](/web/20220928195219/https://www.baeldung.com/cdn-cgi/l/email-protection)

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []: 
```

一个重要的字段是“`Common Name,”`，它应该是我们的域的完全合格的域名(FQDN)。

“`A challenge password`”和“`An optional company name`”可以留空。

**我们还可以用一个命令**创建私钥和 CSR:

```
openssl req -newkey rsa:2048 -keyout domain.key -out domain.csr
```

如果我们希望我们的私钥不加密，我们可以添加`-nodes`选项:

```
openssl req -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr
```

## 4.创建自签名证书

自签名证书是用自己的私钥签名的证书。它可以像 CA 签名的证书一样用来加密数据，但是我们的用户会看到一个警告，说这个证书不可信。

让我们用现有的私钥和 CSR 创建一个自签名证书(`domain.crt`):

```
openssl x509 -signkey domain.key -in domain.csr -req -days 365 -out domain.crt
```

`-days`选项指定证书有效的天数。

我们可以只用一个私钥创建一个自签名证书:

```
openssl req -key domain.key -new -x509 -days 365 -out domain.crt
```

该命令将创建一个临时 CSR 。当然，我们还有 CSR 信息提示。

我们甚至可以只使用一条命令来创建私钥和自签名证书:

```
openssl req -newkey rsa:2048 -keyout domain.key -x509 -days 365 -out domain.crt
```

## 5.使用我们自己的 CA 创建 CA 签名的证书

我们可以通过创建自签名的根 CA 证书，然后将其作为可信证书安装在本地浏览器中，成为我们自己的证书颁发机构(CA)。

### 5.1.创建自签名根 CA

让我们从命令行创建一个私钥(`rootCA.key`)和一个自签名的根 CA 证书(`rootCA.crt`):

```
openssl req -x509 -sha256 -days 1825 -newkey rsa:2048 -keyout rootCA.key -out rootCA.crt
```

### 5.2.用根 CA 签署我们的 CSR

首先，我们将创建一个包含以下内容的配置文本文件(`domain.ext`):

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
DNS.1 = domain
```

“`DNS.1`”字段应该是我们网站的域名。

然后，我们可以用根 CA 证书及其私钥签署我们的 CSR ( `domain.csr`):

```
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in domain.csr -out domain.crt -days 365 -CAcreateserial -extfile domain.ext
```

因此，CA 签名的证书将在`domain.crt`文件中。

## 6.查看证书

我们可以使用`openssl`命令以纯文本方式查看我们的证书内容:

```
openssl x509 -text -noout -in domain.crt
```

输出将类似于:

```
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            64:1a:ad:0f:83:0f:21:33:ff:ac:9e:e6:a5:ec:28:95:b6:e8:8a:f4
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = AU, ST = stateA, L = cityA, O = companyA, OU = sectionA, CN = domain, emailAddress = [[email protected]](/web/20220928195219/https://www.baeldung.com/cdn-cgi/l/email-protection)
        Validity
            Not Before: Jul 12 07:18:18 2021 GMT
            Not After : Jul 12 07:18:18 2022 GMT
        Subject: C = AU, ST = stateA, L = cityA, O = companyA, OU = sectionA, CN = domain, emailAddress = [[email protected]](/web/20220928195219/https://www.baeldung.com/cdn-cgi/l/email-protection)
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:a2:6a:2e:a2:17:68:bd:83:a1:17:87:d8:9c:56:
                    ab:ac:1f:1e:d3:32:b2:91:4d:8e:fe:4f:9c:bf:54:
                    aa:a2:02:8a:bc:14:7c:3d:02:15:a9:df:d5:1b:78:
                    17:ff:82:6b:af:f2:21:36:a5:ad:1b:6d:67:6a:16:
                    26:f2:a9:2f:a8:b0:9a:44:f9:72:de:7a:a0:0a:1f:
                    dc:67:b0:4d:a7:f4:ea:bd:0e:83:7e:d2:ea:15:21:
                    6d:8d:18:65:ed:f8:cc:6a:7f:83:98:e2:a4:f4:d6:
                    00:b6:ed:69:95:4e:0d:59:ee:e8:3f:e7:5a:63:24:
                    98:d1:4b:a5:c9:14:a5:7d:ef:06:78:2e:08:25:3c:
                    fd:05:0c:67:ce:70:5d:34:9b:c4:12:e6:e3:b1:04:
                    6a:db:db:e9:47:31:77:80:4f:09:5e:25:73:75:e4:
                    57:36:34:f8:c3:ed:a2:21:57:0e:e3:c1:5c:fc:d9:
                    f2:a3:b1:d9:d9:4f:e2:3e:ad:21:77:20:98:ed:15:
                    39:99:1b:7e:29:60:14:eb:76:8b:8b:72:16:b1:68:
                    5c:10:51:27:fa:41:49:c5:b7:c4:79:69:5e:28:a2:
                    c3:55:ac:e8:05:0f:4b:4a:bd:4b:2c:8b:7d:92:b0:
                    2d:b3:1a:de:9f:1a:5b:46:65:c6:33:b2:2e:7a:0c:
                    b0:2f
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
         58:c0:cd:df:4f:c1:0b:5c:50:09:1b:a5:1f:6a:b9:9a:7d:07:
         51:ca:43:ec:ba:ab:67:69:c1:eb:cd:63:09:33:42:8f:16:fe:
         6f:05:ee:2c:61:15:80:85:0e:7a:e8:b2:62:ec:b7:15:10:3c:
         7d:fa:60:7f:ee:ee:f8:dc:70:6c:6d:b9:fe:ab:79:5d:1f:73:
         7a:6a:e1:1f:6e:c9:a0:ae:30:b2:a8:ee:c8:94:81:8e:9b:71:
         db:c7:8f:40:d6:2d:4d:f7:b4:d3:cf:32:04:e5:69:d7:31:9c:
         ea:a0:0a:56:79:fa:f9:a3:fe:c9:3e:ff:54:1c:ec:96:1c:88:
         e5:02:d3:d0:da:27:f6:8f:b4:97:09:10:33:32:87:a8:1f:08:
         dc:bc:4c:be:6b:cc:b9:0e:cf:18:12:55:17:44:47:2e:9c:99:
         99:3c:96:60:12:c6:fe:b0:ee:01:97:54:20:b0:13:51:4f:ee:
         1d:c0:3d:1a:30:aa:79:30:12:e2:4f:af:13:85:f8:c8:1e:f5:
         28:7c:55:66:66:10:f4:0a:69:c0:55:8a:9a:c7:eb:ec:15:f0:
         ef:bd:c1:d2:47:43:34:72:71:d2:c3:ff:f0:a3:c1:2c:63:56:
         f2:f5:cf:91:ec:a1:c0:1f:5d:af:c0:8e:7a:02:fe:08:ba:21:
         68:f2:dd:bd
```

## 7.转换证书格式

我们的证书(`domain.crt`)是**，一个 ASCII PEM 编码的 X.509 证书**。我们可以使用 OpenSSL 将其转换为其他格式，以便多用途使用。

### 7.1.将 PEM 转换为 DER

DER 格式通常用于 Java。让我们将 PEM 编码的证书转换为 DER 编码的证书:

```
openssl x509 -in domain.crt -outform der -out domain.der
```

### 7.2.将 PEM 转换为 PKCS12

PKCS12 文件也称为 PFX 文件，通常用于在 Microsoft IIS 中导入和导出证书链。

我们将使用以下命令获取我们的私钥和证书，然后将它们合并到一个 PKCS12 文件中:

```
openssl pkcs12 -inkey domain.key -in domain.crt -export -out domain.pfx
```

## 8.结论

在本文中，我们学习了如何**用 OpenSSL** 从头开始创建自签名证书，**查看该证书，并将其转换为其他格式**。我们希望这些东西对你的工作有所帮助。