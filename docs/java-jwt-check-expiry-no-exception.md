# 检查 JWT 到期，不抛出异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jwt-check-expiry-no-exception>

## 1.介绍

一个 [JWT (JSON Web 令牌)](https://web.archive.org/web/20221221184144/https://www.rfc-editor.org/rfc/rfc7519)基本上是一个 JSON 对象，用于在 Web 上安全地传输信息。此信息可以被验证和信任，因为它是数字签名的。

在本教程中，我们将首先看看[验证 JWT](/web/20221221184144/https://www.baeldung.com/java-auth0-jwt) 和[解码 JWT](/web/20221221184144/https://www.baeldung.com/java-jwt-token-decode) 之间的区别。然后我们将学习如何在 Java 中检查 JWT 的到期，而不抛出任何异常。

## 2.验证和解码 JWT 的区别

在我们开始研究如何检查 JWT 到期之前，让我们先了解一些基本原理。

众所周知，**JWT 的紧凑形式是一个 [Base64 编码的](/web/20221221184144/https://www.baeldung.com/java-base64-encode-and-decode)字符串，包含三个部分:头部、有效载荷和签名**。任何访问 JWT 的人都可以轻松解码并查看其内容。因此，**为了信任一个令牌，我们必须验证包含在 JWT** 中的签名。

有各种各样的 Java JWT 库可以用来创建和管理 JWT。对于我们的代码示例，我们将使用 [Auth0 JWT Java 库](https://web.archive.org/web/20221221184144/https://github.com/auth0/java-jwt)。这是一个易于使用的库，用于创建和管理 jwt。

### 2.1.属国

首先，我们将 Auth0 Java JWT 库的 [Maven 依赖项](https://web.archive.org/web/20221221184144/https://search.maven.org/search?q=g:com.auth0%20AND%20a:java-jwt)添加到我们项目的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.2.1</version>
</dependency>
```

接下来，让我们了解解码和验证 JWT 的区别。

### 2.2.解码 JWT

我们可以通过简单的 Base64 解码它的各个部分来解码 JWT。解码 JWT 返回解码的有效载荷，而不验证 JWT 签名是否有效。不建议将此操作用于任何不可信的邮件，它仅用于查看 JWT 内容。

**要解码 JWT，我们使用`JWT.decode(String)`方法。**这个方法解析 JWT 并返回一个`DecodedJWT`的实例。

`DecodedJWT`实例提供了各种方便的方法，我们可以使用这些方法来获取 JWT 中包含的数据。如果 JWT 不是有效的 Base64 编码的字符串，该方法抛出一个`JWTDecodeException`。

让我们来看看解码 JWT 的代码:

```java
try {
    DecodedJWT decodedJWT = JWT.decode(jwtString);
    return decodedJWT;
} catch (JWTDecodeException e) {
    // ...
}
```

一旦我们获得了`DecodedJWT`实例的一个实例，我们就可以使用它的各种 getter 方法来获得解码的数据。

例如，为了获得令牌到期时间，我们使用了`DecodedJWT.getExpiresAt()`方法。该方法返回一个包含令牌到期时间的`java.util.Date`的实例:

```java
Date expiresAt = decodedJWT.getExpiresAt();
```

接下来，让我们看看 JWT 验证操作。

### 2.3.验证 JWT

验证 JWT 可以确保包含的签名是有效的。或者，它还检查到期时间、失效时间之前、发行者、受众或任何其他声明(如果 JWT 包含任何声明)。

**为了验证 JWT，我们使用`JWTVerifier.verify(String)`方法**。如果签名有效，验证操作也返回一个`DecodedJWT`的实例。只有当签名和所有声明都有效时，它才返回解码后的 JWT。如果签名无效或者任何声明验证失败，它将抛出一个`JWTVerificationException`。

让我们检查代码来验证一个 JWT:

```java
try {
    DecodedJWT decodedJWT = verifier.verify(jwtString);
} catch (JWTVerificationException e) {
    // ...
}
```

从上面的代码片段可以清楚地看出，如果 JWT 无效，`verify()`方法会抛出一个异常。由于该方法还在验证后对令牌进行解码，因此它提供了一种更安全的令牌解码方式。另一方面，`decode()`方法只是解码提供的 JWT 令牌。因此，**为了验证令牌的到期时间而不抛出任何异常，我们使用了`JWT.decode()`方法**。

## 3.检查 JWT 到期

为了简单地读取 JWT 中包含的数据，我们可以解码 JWT 并解析数据。让我们看一下 Java 代码，检查 JWT 是否已经过期:

```java
boolean isJWTExpired(DecodedJWT decodedJWT) {
    Date expiresAt = decodedJWT.getExpiresAt();
    return expiresAt.before(new Date());
}
```

如前所述，我们使用`DecodedJWT.getExpiresAt()`方法来获取 JWT 的到期时间。然后，我们将到期时间与当前时间进行匹配，以检查令牌是否已经到期。

**`JWT.decode()`方法和`JWTVerifier.verify()`方法都返回一个`DecodedJWT`** 的实例。唯一的区别是`verify()`方法也检查签名的有效性，如果无效就返回一个异常。因此，**我们必须仅对可信消息使用`decode()`方法**。**对于任何不可信的消息，我们应该总是使用`verify()`方法，**确保这些 jwt 内的有效签名和任何其他声明。

## 4.结论

在本文中，我们首先看了 JWT 解码和 JWT 验证操作之间的区别。

然后，我们看了如何使用 decode 操作来检查 JWT 的到期时间，而不抛出任何异常。

和往常一样，所有例子的完整代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221221184144/https://github.com/eugenp/tutorials/tree/master/security-modules/jwt)