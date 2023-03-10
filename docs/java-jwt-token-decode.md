# 用 Java 解码 JWT 令牌

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jwt-token-decode>

## 1.概观

REST API 安全中经常使用一个 [JSON Web 令牌](https://web.archive.org/web/20221013062022/https://tools.ietf.org/html/rfc7519) (JWT)。即使令牌可以被像 [Spring Security OAuth](/web/20221013062022/https://www.baeldung.com/spring-security-5-oauth2-login) 这样的框架解析，我们也可能希望在自己的代码中处理令牌。

在本教程中，我们将[解码并验证 JWT](https://web.archive.org/web/20221013062022/https://jwt.io/) 的完整性。

## 2.JWT 的结构

首先，让我们了解一下 [JWT](https://web.archive.org/web/20221013062022/https://datatracker.ietf.org/doc/html/rfc7519#section-3) 的结构:

*   页眉
*   有效载荷(通常称为主体)
*   签名

签名是可选的。一个有效的 JWT 可以只包含报头和有效载荷部分。然而，**我们使用签名部分来验证[安全授权](/web/20221013062022/https://www.baeldung.com/java-json-web-tokens-jjwt)的报头和有效载荷**的内容。

**部分表示为 base64url 编码的字符串**，由句点('.'分隔)分隔符。根据设计，任何人都可以解码 JWT 并读取报头和有效载荷部分的内容。但是我们需要访问用于创建签名的密钥来验证令牌的完整性。

最常见的是，JWT 包含用户的“声明”这些表示关于用户的数据，API 可以使用这些数据来授予权限或跟踪提供令牌的用户。解码令牌允许应用程序使用数据，验证允许应用程序相信 JWT 是由可信来源生成的。

让我们看看如何在 Java 中解码和验证令牌。

## 3.解码 JWT

我们可以使用内置的 Java 函数来解码令牌。

首先，让我们将令牌分成几个部分:

```java
String[] chunks = token.split("\\.");
```

我们应该注意，传递给`String.split`的正则表达式使用了一个转义的`‘.'`字符来避免“.”意思是“任何字符”

我们的`chunks`数组现在应该有两到三个元素对应于 JWT 的部分。

接下来，让我们使用 base64url 解码器对报头和有效载荷部分进行解码:

```java
Base64.Decoder decoder = Base64.getUrlDecoder();

String header = new String(decoder.decode(chunks[0]));
String payload = new String(decoder.decode(chunks[1]));
```

让我们用 JWT 运行这段代码(我们可以[在线解码](https://web.archive.org/web/20221013062022/https://jwt.io/#encoded-jwt)来比较结果):

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkJhZWxkdW5nIFVzZXIiLCJpYXQiOjE1MTYyMzkwMjJ9.qH7Zj_m3kY69kxhaQXTa-ivIpytKXXjZc1ZSmapZnGE
```

输出将为我们提供任何有效载荷的解码头:

```java
{"alg":"HS256","typ":"JWT"}{"sub":"1234567890","name":"Baeldung User","iat":1516239022}
```

如果在 JWT 中只定义了报头和有效载荷部分，我们就完成了，并且成功地解码了信息。

## 4.验证 JWT

接下来，我们可以通过使用签名部分来验证报头和有效载荷的完整性，以确保它们没有被更改。

### 4.1.属国

为了验证，我们可以将 [jjwt](https://web.archive.org/web/20221013062022/https://search.maven.org/artifact/io.jsonwebtoken/jjwt) 添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```

我们应该注意到我们需要从版本`0.7.0`开始的这个库的版本。

### 4.2.配置签名算法和密钥规范

要开始验证有效负载和报头，我们需要最初用于签名令牌的签名算法和密钥:

```java
SignatureAlgorithm sa = HS256;
SecretKeySpec secretKeySpec = new SecretKeySpec(secretKey.getBytes(), sa.getJcaName());
```

在这个例子中，我们已经将我们的签名算法硬编码为`HS256`。然而，我们可以解码头部的 JSON 并读取`alg`字段来获得这个值。

我们还应该注意到，变量`secretKey `是密钥的字符串表示。我们可以通过应用程序的配置或者通过发布 JWT 的服务所公开的 REST API 来将它提供给我们的应用程序。

### 4.3.执行验证

现在我们已经有了签名算法和密钥，我们可以开始执行验证了。

让我们将报头和有效载荷重新组合成一个无符号的 JWT，用“.”将它们连接起来分隔符:

```java
String tokenWithoutSignature = chunks[0] + "." + chunks[1];
String signature = chunks[2];
```

现在我们有了未签名的令牌和提供的签名。我们可以使用这个库来验证它:

```java
DefaultJwtSignatureValidator validator = new DefaultJwtSignatureValidator(sa, secretKeySpec);

if (!validator.isValid(tokenWithoutSignature, signature)) {
    throw new Exception("Could not verify JWT token integrity!");
}
```

我们来分析一下。

首先，我们用选择的算法和秘密创建一个验证器。然后，我们向它提供未签名的令牌数据和提供的签名。

然后，验证器生成一个新的签名，并将其与提供的签名进行比较。如果它们相等，我们就验证了报头和有效载荷的完整性。

## 5.结论

在本文中，我们研究了 JWT 的结构以及如何将其解码成 JSON。

然后，我们使用一个库，通过令牌的签名、算法和密钥来验证令牌的完整性。

和往常一样，本文中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221013062022/https://github.com/eugenp/tutorials/tree/master/security-modules/jjwt)