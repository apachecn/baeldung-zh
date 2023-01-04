# 使用 Auth0 java-jwt 管理 JWT

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-auth0-jwt>

## 1.介绍

JWT(JSON Web Token)是一种标准，它定义了一种在双方之间传输数据和签名的简洁而安全的方式。JWT 中的有效负载是一个断言一些声明的 JSON 对象。验证者可以很容易地验证和信任这个有效载荷，因为它是数字签名的。**jwt 可以使用秘密密钥或者公开/私有密钥对**进行签名。

在本教程中，我们将学习如何使用 [Auth0 JWT Java 库](https://web.archive.org/web/20230103152516/https://github.com/auth0/java-jwt)来[创建和解码一个 JWT](/web/20230103152516/https://www.baeldung.com/java-jwt-token-decode) 。

## 2.JWT 的结构

JWT 基本上由三部分组成:

*   页眉
*   有效载荷
*   签名

每个部分代表一个由点('.'分隔的 Base64 编码的字符串)作为分隔符。

### 2.1.JWT 头球

JWT 报头通常由两部分组成:令牌类型，即`“JWT”,`和用于签署 JWT 的签署算法。

Auth0 Java JWT 库提供了各种算法实现来签署 JWT，如 HMAC、RSA 和 ECDSA。

让我们来看一个 JWT 题头的例子:

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后，上面的 header 对象经过 Base64 编码，形成 JWT 的第一部分。

### 2.2.JWT 有效载荷

JWT 有效载荷包含一组声明。声明基本上是关于一个实体以及一些附加数据的陈述。

索赔有三种类型:

*   注册–这些是一组有用的预定义声明，建议使用，但不是强制性的。为了保持 JWT 契约，这些声明名称只有三个字符长。一些登记的债权包括`iss`(发行人)`exp`(到期时间)`sub`(主体)等。
*   public——这些可以由使用 jwt 的人随意定义。
*   私有–我们可以使用这些声明来创建自定义声明。

让我们来看一个样本 JWT 有效载荷:

```
{
  "sub": "Baeldung Details",
  "nbf": 1669463994,
  "iss": "Baeldung",
  "exp": 1669463998,
  "userId": "1234",
  "iat": 1669463993,
  "jti": "b44bd6c6-f128-4415-8458-6d8b4bc98e4a"
}
```

在这里，我们可以看到有效载荷包含一个私有声明`userId`,表示登录用户的 ID。此外，我们还可以找到一些其他有用的限制性声明，它们定义了有关 JWT 的更多细节。

然后，JWT 有效载荷经过 Base64 编码，形成 JWT 的第二部分。

### 2.3.JWT 签名

最后， **JWT 签名是在我们使用带有秘密密钥**的签名算法对编码报头和编码有效载荷进行签名时生成的。然后，该签名可用于验证 JWT 中的数据是否有效。

重要的是要注意到，任何能够访问 JWT 的人都可以很容易地解码和查看其内容。签名令牌可以验证其中包含的声明的完整性。如果令牌是使用公钥/私钥对签名的，则签名还证明只有持有私钥的一方才是签名者。

最后，结合所有三个部分，我们得到我们的 JWT:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJCYWVsZHVuZyBEZXRhaWxzIiwibmJmIjoxNjY5NDYzOTk0LCJpc3MiOiJCYWVsZHVuZyIsImV4cCI6MTY2OTQ2Mzk5OCwidXNlcklkIjoiMTIzNCIsImlhdCI6MTY2OTQ2Mzk5MywianRpIjoiYjQ0YmQ2YzYtZjEyOC00NDE1LTg0NTgtNmQ4YjRiYzk4ZTRhIn0.14jm1FVPXFDJCUBARDTQkUErMmUTqdt5uMTGW6hDuV0
```

接下来，让我们看看如何使用 Auth0 Java JWT 库创建和管理 JWT。

## 3.使用 Auth0

Auth0 为创建和管理 jwt 提供了一个易于使用的 Java 库。

### 3.1.属国

首先，我们将 Auth0 Java JWT 库的 [Maven 依赖项](https://web.archive.org/web/20230103152516/https://search.maven.org/search?q=g:com.auth0%20AND%20a:java-jwt)添加到我们项目的`pom.xml`文件中:

```
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.2.1</version>
</dependency>
```

### 3.2.配置算法和验证器

我们首先创建一个`Algorithm`类的实例。在本教程中，我们将使用 HMAC256 算法来签署我们的 JWT:

```
Algorithm algorithm = Algorithm.HMAC256("baeldung");
```

这里，我们用一个密钥初始化一个`Algorithm`的实例。我们稍后将在令牌的创建和验证过程中使用它。

此外，让我们初始化一个`JWTVerifier`实例，我们将使用它来验证创建的令牌:

```
JWTVerifier verifier = JWT.require(algorithm)
  .withIssuer("Baeldung")
  .build();
```

**为了初始化验证器，我们使用了`JWT.require(Algorithm)`方法**。这个方法返回一个`Verification`的实例，然后我们可以用它来构建一个`JWTVerifier`的实例。

我们现在准备创建我们的 JWT。

### 3.3.创造一个 JWT

**要创建一个 JWT，我们使用`JWT.create()`方法**。该方法返回一个`JWTCreator.Builder`类的实例。我们将使用这个`Builder`类通过使用`Algorithm`实例签署声明来构建 JWT 令牌:

```
String jwtToken = JWT.create()
  .withIssuer("Baeldung")
  .withSubject("Baeldung Details")
  .withClaim("userId", "1234")
  .withIssuedAt(new Date())
  .withExpiresAt(new Date(System.currentTimeMillis() + 5000L))
  .withJWTId(UUID.randomUUID()
    .toString())
  .withNotBefore(new Date(System.currentTimeMillis() + 1000L))
  .sign(algorithm);
```

上面的代码片段返回了一个 JWT:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJCYWVsZHVuZyBEZXRhaWxzIiwibmJmIjoxNjY5NDYzOTk0LCJpc3MiOiJCYWVsZHVuZyIsImV4cCI6MTY2OTQ2Mzk5OCwidXNlcklkIjoiMTIzNCIsImlhdCI6MTY2OTQ2Mzk5MywianRpIjoiYjQ0YmQ2YzYtZjEyOC00NDE1LTg0NTgtNmQ4YjRiYzk4ZTRhIn0.14jm1FVPXFDJCUBARDTQkUErMmUTqdt5uMTGW6hDuV0
```

让我们讨论一下上面用来设置一些声明的一些`JWTCreator.Builder`类方法:

*   `withIssuer()`–标识创建令牌并对其签名的一方
*   `withSubject()`–识别 JWT 的主题
*   `withIssuedAt()`–标识 JWT 的创建时间；我们可以用这个来确定 JWT 的年龄
*   `withExpiresAt()`–标识 JWT 的到期时间
*   `withJWTId()`–JWT 的唯一标识符
*   `withNotBefore()`–确定不接受 JWT 进行处理的时间
*   `withClaim()`–用于设置任何自定义索赔

### 3.4.验证 JWT

此外，**为了验证一个 JWT，我们使用前面初始化的`JWTVerifier`中的`JWTVerifier.verify(String)`方法**。如果 JWT 有效，该方法解析 JWT 并返回一个`DecodedJWT`的实例。

`DecodedJWT`实例提供了各种方便的方法，我们可以使用这些方法来获取 JWT 中包含的声明。如果 JWT 无效，该方法抛出一个`JWTVerificationException`。

让我们解码我们之前创建的 JWT:

```
try {
    DecodedJWT decodedJWT = verifier.verify(jwtToken);
} catch (JWTVerificationException e) {
    System.out.println(e.getMessage());
}
```

一旦我们获得了`DecodedJWT`实例的一个实例，我们就可以使用它的各种 getter 方法来获得声明。

例如，为了获得自定义声明，我们使用了`DecodedJWT.getClaim(String)`方法。这个方法返回一个`Claim`的实例:

```
Claim claim = decodedJWT.getClaim("userId");
```

这里，我们将获取我们在创建 JWT 时设置的自定义声明`userId`。我们现在可以通过调用`Claim.asString()`或任何其他基于索赔数据类型的可用方法来获得索赔值:

```
String userId = claim.asString();
```

上面的代码片段返回了我们的自定义声明的`String` " `1234″`。

除了 Auth0 Java JWT 库，Auth0 还提供了一个直观的基于网络的 [JWT 调试器](https://web.archive.org/web/20230103152516/https://jwt.io/)来帮助我们解码和验证一个 JWT。

## 4.结论

在本文中，我们研究了 JWT 的结构以及如何将其用于身份验证。

然后，我们使用 Auth0 Java JWT 库，通过令牌的签名、算法和密钥来创建和验证令牌的完整性。

和往常一样，所有例子的完整代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20230103152516/https://github.com/eugenp/tutorials/tree/master/security-modules/jwt)