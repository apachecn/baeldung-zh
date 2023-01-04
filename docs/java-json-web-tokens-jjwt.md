# 使用 JSON Web 令牌(jwt)增强 Java 身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-json-web-tokens-jjwt>

准备好在 Java 应用程序中构建安全认证了吗？不确定使用令牌(尤其是 JSON web 令牌)的好处，或者应该如何部署它们？我很高兴在本教程中为你回答这些问题，甚至更多！

在我们深入研究 JSON Web 令牌([jwt](https://web.archive.org/web/20220701015023/https://en.wikipedia.org/wiki/JSON_Web_Token))和 [JJWT 库](https://web.archive.org/web/20220701015023/https://github.com/jwtk/jjwt)(由 Stormpath 的 CTO Les Hazlewood 创建，并由[的贡献者社区](https://web.archive.org/web/20220701015023/https://github.com/jwtk/jjwt/graphs/contributors)维护)之前，让我们先了解一些基础知识。

## 1。认证与令牌认证

应用程序用来确认用户身份的一组协议就是身份验证。应用程序传统上通过会话 cookies 来保持身份。这种模式依赖于会话 id 的服务器端存储，这迫使开发人员创建唯一的和特定于服务器的会话存储，或者实现为完全独立的会话存储层。

开发令牌认证是为了解决服务器端会话 id 没有也不能解决的问题。就像传统的身份验证一样，用户提供可验证的凭据，但现在发给他们的是一组令牌，而不是会话 ID。初始凭证可以是标准的用户名/密码对、API 密钥，甚至是来自另一个服务的令牌。(Stormpath 的 API 密钥认证特性就是一个例子。)

### 1.1。为什么是代币？

非常简单，使用令牌代替会话 id 可以降低服务器负载，简化权限管理，并为支持分布式或基于云的基础设施提供更好的工具。在 JWT 的例子中，这主要是通过这些类型的令牌的无状态特性来实现的(更多内容见下文)。

令牌提供了各种各样的应用，包括:跨站点请求伪造( [CSRF](https://web.archive.org/web/20220701015023/https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) )保护方案、 [OAuth 2.0](https://web.archive.org/web/20220701015023/https://tools.ietf.org/html/rfc6749) 交互、会话 id 以及(在 cookies 中)作为认证表示。在大多数情况下，标准并没有为令牌指定特定的格式。下面是一个典型的 HTML 格式的 [Spring 安全 CSRF 令牌](https://web.archive.org/web/20220701015023/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CsrfToken.html)的例子:

```
<input name="_csrf" type="hidden" 
  value="f3f42ea9-3104-4d13-84c0-7bcb68202f16"/>
```

如果您尝试在没有正确的 CSRF 令牌的情况下发布表单，您会得到一个错误响应，这就是令牌的效用。上面的例子是一个“哑”令牌。这意味着没有内在的意义可以从令牌本身中收集。这也是 jwt 发挥巨大作用的地方。

## 延伸阅读:

## [使用带有 Spring 安全 OAuth 的 JWT](/web/20220701015023/https://www.baeldung.com/spring-security-oauth-jwt)

A guide to using JWT tokens with Spring Security 5.[Read more](/web/20220701015023/https://www.baeldung.com/spring-security-oauth-jwt) →

## [弹簧座 API + OAuth2 +角度](/web/20220701015023/https://www.baeldung.com/rest-api-spring-oauth2-angular)

Learn how to set up OAuth2 for a Spring REST API using Spring Security 5 and how to consume that from an Angular client.[Read more](/web/20220701015023/https://www.baeldung.com/rest-api-spring-oauth2-angular) →

## [用于 Spring REST API 的 OAuth2 处理 Angular 中的刷新令牌](/web/20220701015023/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular)

Have a look at how to refresh a token using the Spring Security 5 OAuth stack and leveraging a Zuul proxy.[Read more](/web/20220701015023/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular) →

## 2。JWT 里有什么？

jwt(读作“jots”)是 URL 安全的、编码的、加密签名的(有时是加密的)字符串，可以在各种应用程序中用作令牌。这里有一个 JWT 被用作 CSRF 代币的例子:

```
<input name="_csrf" type="hidden" 
  value="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJlNjc4ZjIzMzQ3ZTM0MTBkYjdlNjg3Njc4MjNiMmQ3MCIsImlhdCI6MTQ2NjYzMzMxNywibmJmIjoxNDY2NjMzMzE3LCJleHAiOjE0NjY2MzY5MTd9.rgx_o8VQGuDa2AqCHSgVOD5G68Ld_YYM7N7THmvLIKc"/>
```

在这种情况下，您可以看到令牌比上一个示例长得多。就像我们之前看到的一样，如果提交的表单没有令牌，您会得到一个错误响应。

那么，为什么是 JWT？

上面的令牌是加密签名的，因此可以被验证，证明它没有被篡改。此外，jwt 编码有各种附加信息。

让我们来看看 JWT 的解剖，以便更好地理解我们是如何从中榨取所有这些好处的。您可能已经注意到，这里有三个由句点(`.`)分隔的不同部分:

| 页眉 | eyJhbGciOiJIUzI1NiJ9 |
| 有效载荷 | eyjqdgkioijlnjc4zjizmzq 3 ztm 0 mtbkyjdlnjg 3 njc 4 mjnimmq 3 mcisimlhdc
i6 mtq 2 njyzmxnywibmjmijoxndy 2 njmzmze 3 lcjlehaioje 0 njy 2 mzy 5 MTD 9 |
| 签名 | rgx _ o8 vqguda 2 aqsgvod 5 g68 LD _ yym7 n7 tmvli KC |

每个部分都是用 URL 编码的。这确保了它可以在 URL 中安全地使用(稍后会有更多介绍)。让我们仔细看看每一部分。

### 2.1。标题

如果你用 base64 来解码这个头，你会得到下面的 JSON 字符串:

```
{"alg":"HS256"}
```

这说明 JWT 是和 [HMAC](https://web.archive.org/web/20220701015023/https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) 用 [SHA-256](https://web.archive.org/web/20220701015023/https://en.wikipedia.org/wiki/SHA-2) 签的。

### 2.2。有效载荷

如果您对有效负载进行解码，您将获得以下 JSON 字符串(为清晰起见进行了格式化):

```
{
  "jti": "e678f23347e3410db7e68767823b2d70",
  "iat": 1466633317,
  "nbf": 1466633317,
  "exp": 1466636917
}
```

如您所见，在有效负载中，有许多带值的键。这些密钥被称为“声明”, [JWT 规范](https://web.archive.org/web/20220701015023/https://tools.ietf.org/html/rfc7519#section-4.1)将其中的七个指定为“注册”声明。它们是:

| 出版（issue 的简写） | 发行人 |
| 潜水艇 | 科目 |
| 澳元 | 观众 |
| 经历 | 呼气 |
| nbf | 之前不会 |
| 进气温度传感器 | 发布于 |
| jti | JWT 身份证 |

构建 JWT 时，您可以随意添加任何自定义声明。上面的列表简单地表示了在使用的键和预期类型中保留的声明。我们的 CSRF 有一个 JWT ID，一个“发布时间”，一个“不早于”时间，和一个到期时间。过期时间正好比发布时间晚一分钟。

### 2.3。签名

最后，签名部分是通过将报头和有效负载放在一起(用。中间)并通过指定的算法(在这种情况下，HMAC 使用 SHA-256)将它与已知的秘密一起传递。注意，这个秘密是一个字节数组`always`,它的长度应该对所使用的算法有意义。下面，我使用一个随机的 base64 编码的字符串(为了可读性)，它被转换成一个字节数组。

伪代码是这样的:

```
computeHMACSHA256(
    header + "." + payload, 
    base64DecodeToByteArray("4pE8z3PBoHjnV1AhvGk+e8h2p+ShZpOnpr8cwHmMh1w=")
)
```

只要您知道秘密，就可以自己生成签名，并将结果与 JWT 的签名部分进行比较，以验证它没有被篡改。从技术上讲，一个经过加密签名的 JWT 被称为一个 JWS。jwt 也可以被加密，然后被称为 T2 JWE T3。(在实际操作中，术语 JWT 用于描述 JWEs 和 JWSs。)

这使我们回到使用 JWT 作为我们的 CSRF 令牌的好处。我们可以验证签名，并且可以使用 JWT 中编码的信息来确认其有效性。因此，不仅 JWT 的字符串表示需要匹配存储在服务器端的内容，我们还可以通过检查`exp`声明来确保它不会过期。这使服务器不必维护额外的状态。

嗯，我们已经谈了很多了。让我们深入一些代码！

## 3。设置 JJWT 教程

JJWT([https://github.com/jwtk/jjwt](https://web.archive.org/web/20220701015023/https://github.com/jwtk/jjwt))是一个 Java 库，提供端到端的 JSON Web 令牌创建和验证。永久免费和开源(Apache License，2.0 版)，它被设计成一个以构建器为中心的界面，隐藏了它的大部分复杂性。

使用 JJWT 的主要操作包括构建和解析 JWT。接下来我们将查看这些操作，然后深入了解 JJWT 的一些扩展特性，最后，我们将看到 JWT 在 Spring Security，Spring Boot 应用程序中作为 CSRF 令牌运行。

**下面几节演示的代码可以在[这里](https://web.archive.org/web/20220701015023/https://github.com/eugenp/tutorials/tree/master/jjwt)找到。注意:该项目从一开始就使用 Spring Boot，因为它易于与它公开的 API 进行交互。**

Spring Boot 最大的优点之一就是构建和启动一个应用程序是多么容易。要运行 JJWT Fun 应用程序，只需执行以下操作:

```
mvn clean spring-boot:run
```

在这个示例应用程序中公开了十个端点(我使用 httpie 与应用程序进行交互。在这里可以找到[。)](https://web.archive.org/web/20220701015023/https://github.com/jkbrzt/httpie)

```
http localhost:8080
```

```
Available commands (assumes httpie - https://github.com/jkbrzt/httpie):

  http http://localhost:8080/
	This usage message

  http http://localhost:8080/static-builder
	build JWT from hardcoded claims

  http POST http://localhost:8080/dynamic-builder-general claim-1=value-1 ... [claim-n=value-n]
	build JWT from passed in claims (using general claims map)

  http POST http://localhost:8080/dynamic-builder-specific claim-1=value-1 ... [claim-n=value-n]
	build JWT from passed in claims (using specific claims methods)

  http POST http://localhost:8080/dynamic-builder-compress claim-1=value-1 ... [claim-n=value-n]
	build DEFLATE compressed JWT from passed in claims

  http http://localhost:8080/parser?jwt=<jwt>
	Parse passed in JWT

  http http://localhost:8080/parser-enforce?jwt=<jwt>
	Parse passed in JWT enforcing the 'iss' registered claim and the 'hasMotorcycle' custom claim

  http http://localhost:8080/get-secrets
	Show the signing keys currently in use.

  http http://localhost:8080/refresh-secrets
	Generate new signing keys and show them.

  http POST http://localhost:8080/set-secrets 
    HS256=base64-encoded-value HS384=base64-encoded-value HS512=base64-encoded-value
	Explicitly set secrets to use in the application.
```

在接下来的小节中，我们将检查每个端点以及处理程序中包含的 JJWT 代码。

## 4。用 JJWT 构建 JWT

因为 JJWT 的[流畅界面](https://web.archive.org/web/20220701015023/https://en.wikipedia.org/wiki/Fluent_interface)，JWT 的创建基本上是一个三步的过程:

1.  令牌内部声明的定义，如颁发者、主题、到期时间和 ID。
2.  JWT 的加密签名(使其成为 JWS)。
3.  根据 [JWT 压缩序列化](https://web.archive.org/web/20220701015023/https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41#section-3.1)规则，将 JWT 压缩为 URL 安全字符串。

最终的 JWT 将是一个由三部分组成的 base64 编码的字符串，使用指定的签名算法和提供的密钥进行签名。在这一点之后，令牌准备好与另一方共享。

这里有一个 JJWT 的例子:

```
String jws = Jwts.builder()
  .setIssuer("Stormpath")
  .setSubject("msilverman")
  .claim("name", "Micah Silverman")
  .claim("scope", "admins")
  // Fri Jun 24 2016 15:33:42 GMT-0400 (EDT)
  .setIssuedAt(Date.from(Instant.ofEpochSecond(1466796822L)))
  // Sat Jun 24 2116 15:33:42 GMT-0400 (EDT)
  .setExpiration(Date.from(Instant.ofEpochSecond(4622470422L)))
  .signWith(
    SignatureAlgorithm.HS256,
    TextCodec.BASE64.decode("Yn2kjibddFAWtnPJ2AFlL8WXmohJMCvigQggaEypa5E=")
  )
  .compact();
```

这与代码项目的`StaticJWTController.fixedBuilder`方法中的代码非常相似。

此时，有必要讨论几个与 jwt 和签名相关的反模式。如果您以前看过 JWT 的例子，您可能会遇到这些签名反模式场景之一:

1.  ```
    .signWith(
        SignatureAlgorithm.HS256,
       "secret".getBytes("UTF-8")    
    )
    ```

2.  ```
    .signWith(
        SignatureAlgorithm.HS256,
        "Yn2kjibddFAWtnPJ2AFlL8WXmohJMCvigQggaEypa5E=".getBytes("UTF-8")
    )
    ```

3.  ```
    .signWith(
        SignatureAlgorithm.HS512,
        TextCodec.BASE64.decode("Yn2kjibddFAWtnPJ2AFlL8WXmohJMCvigQggaEypa5E=")
    )
    ```

任何`HS`类型的签名算法都采用一个字节数组。对于人类来说，读取一个字符串并将其转换为字节数组是很方便的。

上面的反模式 1 演示了这一点。这是有问题的，因为这个秘密因为太短而被削弱了，而且它不是一个原始形式的字节数组。因此，为了保持可读性，我们可以对字节数组进行 base64 编码。

但是，上面的反模式 2 采用 base64 编码的字符串，并将其直接转换为字节数组。应该做的是将 base64 字符串解码回原始字节数组。

上面的数字 3 证明了这一点。那么，为什么这个也是反模式呢？这是一个微妙的原因。注意，签名算法是 HS512。字节数组不是`HS512`可以支持的最大长度，这使得它比该算法可能的更弱的秘密。

示例代码包括一个名为`SecretService`的类，该类确保适当强度的秘密被用于给定的算法。在应用程序启动时，为每个 HS 算法创建一组新的秘密。存在用于刷新秘密以及显式设置秘密的端点。

如果您的项目按上述方式运行，请执行以下命令，以便下面的 JWT 示例与您的项目响应相匹配。

```
http POST localhost:8080/set-secrets \
  HS256="Yn2kjibddFAWtnPJ2AFlL8WXmohJMCvigQggaEypa5E=" \
  HS384="VW96zL+tYlrJLNCQ0j6QPTp+d1q75n/Wa8LVvpWyG8pPZOP6AA5X7XOIlI90sDwx" \
  HS512="cd+Pr1js+w2qfT2BoCD+tPcYp9LbjpmhSMEJqUob1mcxZ7+Wmik4AYdjX+DlDjmE4yporzQ9tm7v3z/j+QbdYg=="
```

现在，您可以到达`/static-builder`终点:

```
http http://localhost:8080/static-builder
```

这产生了一个如下所示的 JWT:

```
eyJhbGciOiJIUzI1NiJ9.
eyJpc3MiOiJTdG9ybXBhdGgiLCJzdWIiOiJtc2lsdmVybWFuIiwibmFtZSI6Ik1pY2FoIFNpbHZlcm1hbiIsInNjb3BlIjoiYWRtaW5zIiwiaWF0IjoxNDY2Nzk2ODIyLCJleHAiOjQ2MjI0NzA0MjJ9.
kP0i_RvTAmI8mgpIkDFhRX3XthSdP-eqqFKGcU92ZIQ
```

现在，点击:

```
http http://localhost:8080/parser?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJzdWIiOiJtc2lsdmVybWFuIiwibmFtZSI6Ik1pY2FoIFNpbHZlcm1hbiIsInNjb3BlIjoiYWRtaW5zIiwiaWF0IjoxNDY2Nzk2ODIyLCJleHAiOjQ2MjI0NzA0MjJ9.kP0i_RvTAmI8mgpIkDFhRX3XthSdP-eqqFKGcU92ZIQ
```

响应包含了我们在创建 JWT 时包含的所有声明。

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
...
{
    "jws": {
        "body": {
            "exp": 4622470422,
            "iat": 1466796822,
            "iss": "Stormpath",
            "name": "Micah Silverman",
            "scope": "admins",
            "sub": "msilverman"
        },
        "header": {
            "alg": "HS256"
        },
        "signature": "kP0i_RvTAmI8mgpIkDFhRX3XthSdP-eqqFKGcU92ZIQ"
    },
    "status": "SUCCESS"
}
```

这是解析操作，我们将在下一节讨论。

现在，让我们点击一个端点，该端点将声明作为参数，并将为我们构建一个定制的 JWT。

```
http -v POST localhost:8080/dynamic-builder-general iss=Stormpath sub=msilverman hasMotorcycle:=true
```

**注意**:在`hasMotorcycle`声明和其他声明之间有细微的区别。默认情况下，httpie 假设 JSON 参数是字符串。要使用 httpie 提交原始 JSON，可以使用 `:=`表单，而不是`=`。没有那个，它就会提交`“hasMotorcycle”: “true”`，这不是我们想要的。

以下是输出结果:

```
POST /dynamic-builder-general HTTP/1.1
Accept: application/json
...
{
    "hasMotorcycle": true,
    "iss": "Stormpath",
    "sub": "msilverman"
}

HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
...
{
    "jwt": 
      "eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJzdWIiOiJtc2lsdmVybWFuIiwiaGFzTW90b3JjeWNsZSI6dHJ1ZX0.OnyDs-zoL3-rw1GaSl_KzZzHK9GoiNocu-YwZ_nQNZU",
    "status": "SUCCESS"
} 
```

让我们看看支持这个端点的代码:

```
@RequestMapping(value = "/dynamic-builder-general", method = POST)
public JwtResponse dynamicBuilderGeneric(@RequestBody Map<String, Object> claims) 
  throws UnsupportedEncodingException {
    String jws =  Jwts.builder()
        .setClaims(claims)
        .signWith(
            SignatureAlgorithm.HS256,
            secretService.getHS256SecretBytes()
        )
        .compact();
    return new JwtResponse(jws);
}
```

第 2 行确保传入的 JSON 被自动转换成 Java `Map <string, object>`，这对 JJWT 来说非常方便，因为第 5 行的方法只需获取该 Map，然后一次性设置所有声明。

尽管这段代码很简洁，但我们需要一些更具体的东西来确保传递的声明是有效的。当您已经知道地图中表示的声明是有效的时，使用`.setClaims(Map<String, Object> claims)`方法会很方便。这就是 JJWT 库中 Java 类型安全的来源。

对于 JWT 规范中定义的每个注册声明，JJWT 中都有一个对应的 Java 方法，它采用规范正确的类型。

让我们点击示例中的另一个端点，看看会发生什么:

```
http -v POST localhost:8080/dynamic-builder-specific iss=Stormpath sub:=5 hasMotorcycle:=true
```

注意，我们已经为“sub”声明传递了一个整数 5。以下是输出结果:

```
POST /dynamic-builder-specific HTTP/1.1
Accept: application/json
...
{
    "hasMotorcycle": true,
    "iss": "Stormpath",
    "sub": 5
}

HTTP/1.1 400 Bad Request
Connection: close
Content-Type: application/json;charset=UTF-8
...
{
    "exceptionType": "java.lang.ClassCastException",
    "message": "java.lang.Integer cannot be cast to java.lang.String",
    "status": "ERROR"
}
```

现在，我们得到一个错误响应，因为代码强制注册声明的类型。在这种情况下，`sub`必须是一个字符串。以下是支持该端点的代码:

```
@RequestMapping(value = "/dynamic-builder-specific", method = POST)
public JwtResponse dynamicBuilderSpecific(@RequestBody Map<String, Object> claims) 
  throws UnsupportedEncodingException {
    JwtBuilder builder = Jwts.builder();

    claims.forEach((key, value) -> {
        switch (key) {
            case "iss":
                builder.setIssuer((String) value);
                break;
            case "sub":
                builder.setSubject((String) value);
                break;
            case "aud":
                builder.setAudience((String) value);
                break;
            case "exp":
                builder.setExpiration(Date.from(
                    Instant.ofEpochSecond(Long.parseLong(value.toString()))
                ));
                break;
            case "nbf":
                builder.setNotBefore(Date.from(
                    Instant.ofEpochSecond(Long.parseLong(value.toString()))
                ));
                break;
            case "iat":
                builder.setIssuedAt(Date.from(
                    Instant.ofEpochSecond(Long.parseLong(value.toString()))
                ));
                break;
            case "jti":
                builder.setId((String) value);
                break;
            default:
                builder.claim(key, value);
        }
    });

    builder.signWith(SignatureAlgorithm.HS256, secretService.getHS256SecretBytes());

    return new JwtResponse(builder.compact());
}
```

与之前一样，该方法接受一个`Map<String, Object>`声明作为其参数。然而，这一次，我们为每个注册的声明调用特定的方法，该方法强制执行类型。

对此的一个改进是使错误消息更加具体。现在，我们只知道我们的一个声明不是正确的类型。我们不知道哪个声明是错误的，也不知道它应该是什么。这里有一个方法可以给我们一个更具体的错误信息。它还处理了当前代码中的一个错误。

```
private void ensureType(String registeredClaim, Object value, Class expectedType) {
    boolean isCorrectType =
        expectedType.isInstance(value) ||
        expectedType == Long.class && value instanceof Integer;

    if (!isCorrectType) {
        String msg = "Expected type: " + expectedType.getCanonicalName() + 
		    " for registered claim: '" + registeredClaim + "', but got value: " + 
			value + " of type: " + value.getClass().getCanonicalName();
        throw new JwtException(msg);
    }
}
```

第 3 行检查传入的值是否是预期的类型。如果没有，抛出一个`JwtException`并给出具体的错误。让我们通过进行与前面相同的调用来看一下实际情况:

```
http -v POST localhost:8080/dynamic-builder-specific iss=Stormpath sub:=5 hasMotorcycle:=true
```

```
POST /dynamic-builder-specific HTTP/1.1
Accept: application/json
...
User-Agent: HTTPie/0.9.3

{
    "hasMotorcycle": true,
    "iss": "Stormpath",
    "sub": 5
}

HTTP/1.1 400 Bad Request
Connection: close
Content-Type: application/json;charset=UTF-8
...
{
    "exceptionType": "io.jsonwebtoken.JwtException",
    "message": 
      "Expected type: java.lang.String for registered claim: 'sub', but got value: 5 of type: java.lang.Integer",
    "status": "ERROR"
}
```

现在，我们有一个非常具体的错误消息，告诉我们`sub`声明是错误的。

让我们回到代码中的错误。这个问题与 JJWT 库无关。问题是内置于 Spring Boot 的 JSON 到 Java 对象映射器对我们来说太聪明了。

如果有接受 Java 对象的方法，JSON mapper 会自动将小于或等于 2，147，483，647 的传入数字转换成 Java `Integer`。同样，它会自动将大于 2，147，483，647 的传入数字转换成 Java `Long`。对于 JWT 的`iat`、`nbf`和`exp`声明，无论映射的对象是整数还是长整型，我们都希望我们的 ensureType 测试通过。这就是为什么我们有额外的子句来确定传入的值是否是正确的类型:

```
 boolean isCorrectType =
     expectedType.isInstance(value) ||
     expectedType == Long.class && value instanceof Integer;
```

如果我们期待一个长整型，但值是一个整数的实例，我们仍然说它是正确的类型。了解了这种验证的情况后，我们现在可以将它集成到我们的`dynamicBuilderSpecific`方法中:

```
@RequestMapping(value = "/dynamic-builder-specific", method = POST)
public JwtResponse dynamicBuilderSpecific(@RequestBody Map<String, Object> claims) 
  throws UnsupportedEncodingException {
    JwtBuilder builder = Jwts.builder();

    claims.forEach((key, value) -> {
        switch (key) {
            case "iss":
                ensureType(key, value, String.class);
                builder.setIssuer((String) value);
                break;
            case "sub":
                ensureType(key, value, String.class);
                builder.setSubject((String) value);
                break;
            case "aud":
                ensureType(key, value, String.class);
                builder.setAudience((String) value);
                break;
            case "exp":
                ensureType(key, value, Long.class);
                builder.setExpiration(Date.from(
				    Instant.ofEpochSecond(Long.parseLong(value.toString()))
				));
                break;
            case "nbf":
                ensureType(key, value, Long.class);
                builder.setNotBefore(Date.from(
					Instant.ofEpochSecond(Long.parseLong(value.toString()))
				));
                break;
            case "iat":
                ensureType(key, value, Long.class);
                builder.setIssuedAt(Date.from(
					Instant.ofEpochSecond(Long.parseLong(value.toString()))
				));
                break;
            case "jti":
                ensureType(key, value, String.class);
                builder.setId((String) value);
                break;
            default:
                builder.claim(key, value);
        }
    });

    builder.signWith(SignatureAlgorithm.HS256, secretService.getHS256SecretBytes());

    return new JwtResponse(builder.compact());
}
```

**注意**:在本节的所有示例代码中，jwt 都使用 SHA-256 算法通过 HMAC 进行签名。这是为了保持例子的简单。JJWT 库支持 12 种不同的签名算法，您可以在自己的代码中利用这些算法。

## 5。用 JJWT 解析 JWT

我们前面看到，我们的代码示例有一个解析 JWT 的端点。到达此终点:

```
http http://localhost:8080/parser?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJzdWIiOiJtc2lsdmVybWFuIiwibmFtZSI6Ik1pY2FoIFNpbHZlcm1hbiIsInNjb3BlIjoiYWRtaW5zIiwiaWF0IjoxNDY2Nzk2ODIyLCJleHAiOjQ2MjI0NzA0MjJ9.kP0i_RvTAmI8mgpIkDFhRX3XthSdP-eqqFKGcU92ZIQ
```

产生以下响应:

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
...
{
    "claims": {
        "body": {
            "exp": 4622470422,
            "iat": 1466796822,
            "iss": "Stormpath",
            "name": "Micah Silverman",
            "scope": "admins",
            "sub": "msilverman"
        },
        "header": {
            "alg": "HS256"
        },
        "signature": "kP0i_RvTAmI8mgpIkDFhRX3XthSdP-eqqFKGcU92ZIQ"
    },
    "status": "SUCCESS"
}
```

`StaticJWTController`类的`parser`方法如下所示:

```
@RequestMapping(value = "/parser", method = GET)
public JwtResponse parser(@RequestParam String jwt) throws UnsupportedEncodingException {
    Jws<Claims> jws = Jwts.parser()
        .setSigningKeyResolver(secretService.getSigningKeyResolver())
        .parseClaimsJws(jwt);

    return new JwtResponse(jws);
}
```

第 4 行表明我们期望传入的字符串是一个有符号的 JWT(一个 JWS)。而且，我们在解析 JWT 时使用了与签名时相同的秘密。第 5 行解析了 JWT 的声明。在内部，它验证签名，如果签名无效，它将抛出异常。

注意，在这种情况下，我们传入的是一个`SigningKeyResolver`而不是一个键本身。这是 JJWT 最强大的方面之一。JWT 的报头表示用于签名的算法。然而，在我们信任 JWT 之前，我们需要验证它。这似乎是一个第二十二条军规。我们来看一下`SecretService.getSigningKeyResolver`的方法:

```
private SigningKeyResolver signingKeyResolver = new SigningKeyResolverAdapter() {
    @Override
    public byte[] resolveSigningKeyBytes(JwsHeader header, Claims claims) {
        return TextCodec.BASE64.decode(secrets.get(header.getAlgorithm()));
    }
};
```

使用对`JwsHeader`的访问，我可以检查算法并返回用于签署 JWT 的秘密的正确字节数组。现在，JJWT 将使用这个字节数组作为密钥来验证 JWT 没有被篡改。

如果我删除 JWT 中传递的最后一个字符(它是签名的一部分)，这是响应:

```
HTTP/1.1 400 Bad Request
Connection: close
Content-Type: application/json;charset=UTF-8
Date: Mon, 27 Jun 2016 13:19:08 GMT
Server: Apache-Coyote/1.1
Transfer-Encoding: chunked

{
    "exceptionType": "io.jsonwebtoken.SignatureException",
    "message": 
      "JWT signature does not match locally computed signature. JWT validity cannot be asserted and should not be trusted.",
    "status": "ERROR"
}
```

## 6。实践中的 jwt:Spring Security CSRF 令牌

虽然这篇文章的重点不是 Spring 安全，但是我们将在这里深入研究一下，展示 JJWT 库的一些真实用法。

[跨站点请求伪造](https://web.archive.org/web/20220701015023/https://en.wikipedia.org/wiki/Cross-site_request_forgery)是一个安全漏洞，恶意网站利用它诱骗您向您已经建立信任的网站提交请求。常见的补救方法之一是实现一个[同步器令牌模式](https://web.archive.org/web/20220701015023/https://docs.spring.io/spring-security/site/docs/5.0.x/reference/html/csrf.html)。这种方法将一个令牌插入到 web 表单中，应用服务器对照其存储库检查传入的令牌，以确认它是正确的。如果令牌丢失或无效，服务器将响应一个错误。

Spring Security 内置了同步器令牌模式。更好的是，如果你正在使用 [Spring Boot 和百里香模板](https://web.archive.org/web/20220701015023/https://spring.io/guides/gs/serving-web-content/)，同步令牌会自动为你插入。

默认情况下，Spring Security 使用的令牌是一个“哑”令牌。它只是一系列的字母和数字。这种方法很好，也很有效。在本节中，我们通过使用 jwt 作为令牌来增强基本功能。除了验证提交的令牌是预期的令牌之外，我们还要验证 JWT，以进一步证明令牌没有被篡改，并确保它没有过期。

首先，我们将使用 Java 配置来配置 Spring Security。默认情况下，所有路径都需要身份验证，所有发布端点都需要 CSRF 令牌。我们将稍微放松一下，这样我们到目前为止构建的内容仍然可以工作。

```
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    private String[] ignoreCsrfAntMatchers = {
        "/dynamic-builder-compress",
        "/dynamic-builder-general",
        "/dynamic-builder-specific",
        "/set-secrets"
    };

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf()
                .ignoringAntMatchers(ignoreCsrfAntMatchers)
            .and().authorizeRequests()
                .antMatchers("/**")
                .permitAll();
    }
}
```

我们在这里做两件事。首先，我们说当发布到我们的 REST API 端点时，CSRF 令牌是必需的(第 15 行)。第二，我们说所有路径都应该允许未经身份验证的访问(第 17–18 行)。

让我们确认一下 Spring Security 是否按照我们预期的方式运行。启动应用程序，在浏览器中点击以下网址:

```
http://localhost:8080/jwt-csrf-form
```

这是该视图的百里香模板:

```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <!--/*/ <th:block th:include="fragments/head :: head"/> /*/-->
    </head>
    <body>
        <div class="container-fluid">
            <div class="row">
                <div class="box col-md-6 col-md-offset-3">
                    <p/>
                    <form method="post" th:action="@{/jwt-csrf-form}">
                        <input type="submit" class="btn btn-primary" value="Click Me!"/>
                    </form>
                </div>
            </div>
        </div>
    </body>
</html>
```

这是一个非常基本的表单，提交后会发布到同一个端点。请注意，表单中没有显式引用 CSRF 令牌。如果您查看源代码，您会看到如下内容:

```
<input type="hidden" name="_csrf" value="5f375db2-4f40-4e72-9907-a290507cb25e" />
```

这就是你需要知道的所有确认，Spring 安全正在运行，百里香模板正在自动插入 CSRF 令牌。

为了使该值成为 JWT，我们将启用一个自定义的`CsrfTokenRepository`。下面是我们的 Spring 安全配置的变化:

```
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    CsrfTokenRepository jwtCsrfTokenRepository;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf()
                .csrfTokenRepository(jwtCsrfTokenRepository)
                .ignoringAntMatchers(ignoreCsrfAntMatchers)
            .and().authorizeRequests()
                .antMatchers("/**")
                .permitAll();
    }
}
```

为此，我们需要一个配置来公开一个返回自定义令牌存储库的 bean。配置如下:

```
@Configuration
public class CSRFConfig {

    @Autowired
    SecretService secretService;

    @Bean
    @ConditionalOnMissingBean
    public CsrfTokenRepository jwtCsrfTokenRepository() {
        return new JWTCsrfTokenRepository(secretService.getHS256SecretBytes());
    }
}
```

这里是我们的自定义存储库(重要的部分):

```
public class JWTCsrfTokenRepository implements CsrfTokenRepository {

    private static final Logger log = LoggerFactory.getLogger(JWTCsrfTokenRepository.class);
    private byte[] secret;

    public JWTCsrfTokenRepository(byte[] secret) {
        this.secret = secret;
    }

    @Override
    public CsrfToken generateToken(HttpServletRequest request) {
        String id = UUID.randomUUID().toString().replace("-", "");

        Date now = new Date();
        Date exp = new Date(System.currentTimeMillis() + (1000*30)); // 30 seconds

        String token;
        try {
            token = Jwts.builder()
                .setId(id)
                .setIssuedAt(now)
                .setNotBefore(now)
                .setExpiration(exp)
                .signWith(SignatureAlgorithm.HS256, secret)
                .compact();
        } catch (UnsupportedEncodingException e) {
            log.error("Unable to create CSRf JWT: {}", e.getMessage(), e);
            token = id;
        }

        return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", token);
    }

    @Override
    public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
        ...
    }

    @Override
    public CsrfToken loadToken(HttpServletRequest request) {
        ...
    }
}
```

`generateToken`方法创建一个 JWT，它在创建后 30 秒到期。准备就绪后，我们可以再次启动应用程序并查看`/jwt-csrf-form`的源代码。

现在，隐藏字段看起来像这样:

```
<input type="hidden" name="_csrf" 
  value="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIxZjIyMDdiNTVjOWM0MjU0YjZlMjY4MjQwYjIwNzZkMSIsImlhdCI6MTQ2NzA3MDQwMCwibmJmIjoxNDY3MDcwNDAwLCJleHAiOjE0NjcwNzA0MzB9.2kYLO0iMWUheAncXAzm0UdQC1xUC5I6RI_ShJ_74e5o" />
```

万岁。现在我们的 CSRF 代币是 JWT。这并不难。

然而，这只是谜题的一半。默认情况下，Spring Security 只是保存 CSRF 令牌，并确认在 web 表单中提交的令牌与保存的令牌相匹配。我们希望扩展该功能来验证 JWT，并确保它没有过期。为此，我们将添加一个过滤器。下面是我们的 Spring 安全配置现在的样子:

```
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    ...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .addFilterAfter(new JwtCsrfValidatorFilter(), CsrfFilter.class)
            .csrf()
                .csrfTokenRepository(jwtCsrfTokenRepository)
                .ignoringAntMatchers(ignoreCsrfAntMatchers)
            .and().authorizeRequests()
                .antMatchers("/**")
                .permitAll();
    }

    ...
}
```

在第 9 行，我们添加了一个过滤器，并把它放在过滤器链中默认的`CsrfFilter`之后。因此，当我们的过滤器被点击时，JWT 令牌(作为一个整体)将已经被确认为 Spring Security 保存的正确值。

下面是`JwtCsrfValidatorFilter`(它是私有的，因为它是我们 Spring 安全配置的内部类):

```
private class JwtCsrfValidatorFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
      HttpServletRequest request, 
      HttpServletResponse response, 
      FilterChain filterChain) throws ServletException, IOException {
        // NOTE: A real implementation should have a nonce cache so the token cannot be reused
        CsrfToken token = (CsrfToken) request.getAttribute("_csrf");

        if (
            // only care if it's a POST
            "POST".equals(request.getMethod()) &&
            // ignore if the request path is in our list
            Arrays.binarySearch(ignoreCsrfAntMatchers, request.getServletPath()) < 0 &&
            // make sure we have a token
            token != null
        ) {
            // CsrfFilter already made sure the token matched. 
            // Here, we'll make sure it's not expired
            try {
                Jwts.parser()
                    .setSigningKey(secret.getBytes("UTF-8"))
                    .parseClaimsJws(token.getToken());
            } catch (JwtException e) {
                // most likely an ExpiredJwtException, but this will handle any
                request.setAttribute("exception", e);
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                RequestDispatcher dispatcher = request.getRequestDispatcher("expired-jwt");
                dispatcher.forward(request, response);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

请看第 23 行。我们像以前一样解析 JWT。在这种情况下，如果抛出异常，请求将被转发到`expired-jwt`模板。如果 JWT 验证通过，那么处理将照常继续。

这就结束了用 JWT 令牌库和验证器覆盖默认 Spring Security CSRF 令牌行为的循环。

如果您启动应用程序，浏览到`/jwt-csrf-form`，等待 30 秒多一点，然后单击按钮，您将看到如下内容:

[![jwt_expired](img/613a59ce1e0f51a6d2cfb3ccdb79701f.png)](/web/20220701015023/https://www.baeldung.com/wp-content/uploads/2016/08/jwt_expired.png)

## 7。JJWT 扩展功能

我们将用一些超出规范的特性来结束我们的 JJWT 之旅。

### 7.1。强制执行索赔

作为解析过程的一部分，JJWT 允许您指定必需的声明以及这些声明应该具有的值。如果您的 jwt 中有某些信息必须存在，以便您认为它们是有效的，这将非常方便。它避免了大量手动验证声明的分支逻辑。下面是服务于我们示例项目的`/parser-enforce`端点的方法。

```
@RequestMapping(value = "/parser-enforce", method = GET)
public JwtResponse parserEnforce(@RequestParam String jwt) 
  throws UnsupportedEncodingException {
    Jws<Claims> jws = Jwts.parser()
        .requireIssuer("Stormpath")
        .require("hasMotorcycle", true)
        .setSigningKeyResolver(secretService.getSigningKeyResolver())
        .parseClaimsJws(jwt);

    return new JwtResponse(jws);
}
```

第 5 行和第 6 行显示了注册声明和定制声明的语法。在本例中，如果 iss 声明不存在或不具有值:Stormpath，则 JWT 将被视为无效。如果自定义 hasMotorcycle 声明不存在或没有值:true，它也将无效。

让我们首先创建一个 JWT，沿着快乐的道路:

```
http -v POST localhost:8080/dynamic-builder-specific \
  iss=Stormpath hasMotorcycle:=true sub=msilverman
```

```
POST /dynamic-builder-specific HTTP/1.1
Accept: application/json
...
{
    "hasMotorcycle": true,
    "iss": "Stormpath",
    "sub": "msilverman"
}

HTTP/1.1 200 OK
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Content-Type: application/json;charset=UTF-8
...
{
    "jwt": 
      "eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJoYXNNb3RvcmN5Y2xlIjp0cnVlLCJzdWIiOiJtc2lsdmVybWFuIn0.qrH-U6TLSVlHkZdYuqPRDtgKNr1RilFYQJtJbcgwhR0",
    "status": "SUCCESS"
}
```

现在，让我们验证 JWT:

```
http -v localhost:8080/parser-enforce?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJoYXNNb3RvcmN5Y2xlIjp0cnVlLCJzdWIiOiJtc2lsdmVybWFuIn0.qrH-U6TLSVlHkZdYuqPRDtgKNr1RilFYQJtJbcgwhR0
```

```
GET /parser-enforce?jwt=http 
  -v localhost:8080/parser-enforce?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJoYXNNb3RvcmN5Y2xlIjp0cnVlLCJzdWIiOiJtc2lsdmVybWFuIn0.qrH-U6TLSVlHkZdYuqPRDtgKNr1RilFYQJtJbcgwhR0 HTTP/1.1
Accept: */*
...
HTTP/1.1 200 OK
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Content-Type: application/json;charset=UTF-8
...
{
    "jws": {
        "body": {
            "hasMotorcycle": true,
            "iss": "Stormpath",
            "sub": "msilverman"
        },
        "header": {
            "alg": "HS256"
        },
        "signature": "qrH-U6TLSVlHkZdYuqPRDtgKNr1RilFYQJtJbcgwhR0"
    },
    "status": "SUCCESS"
}
```

到目前为止，一切顺利。现在，这一次，让我们把摩托车放在外面:

```
http -v POST localhost:8080/dynamic-builder-specific iss=Stormpath sub=msilverman
```

这一次，如果我们试图验证 JWT:

```
http -v localhost:8080/parser-enforce?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJzdWIiOiJtc2lsdmVybWFuIn0.YMONlFM1tNgttUYukDRsi9gKIocxdGAOLaJBymaQAWc
```

我们得到:

```
GET /parser-enforce?jwt=http -v localhost:8080/parser-enforce?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJzdWIiOiJtc2lsdmVybWFuIn0.YMONlFM1tNgttUYukDRsi9gKIocxdGAOLaJBymaQAWc HTTP/1.1
Accept: */*
...
HTTP/1.1 400 Bad Request
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Connection: close
Content-Type: application/json;charset=UTF-8
...
{
    "exceptionType": "io.jsonwebtoken.MissingClaimException",
    "message": 
      "Expected hasMotorcycle claim to be: true, but was not present in the JWT claims.",
    "status": "ERROR"
}
```

这表明我们的 hasMotorcycle 索赔是预期的，但却丢失了。

让我们再举一个例子:

```
http -v POST localhost:8080/dynamic-builder-specific iss=Stormpath hasMotorcycle:=false sub=msilverman
```

这一次，所需的声明是存在的，但是它具有错误的值。让我们看看以下内容的输出:

```
http -v localhost:8080/parser-enforce?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJoYXNNb3RvcmN5Y2xlIjpmYWxzZSwic3ViIjoibXNpbHZlcm1hbiJ9.8LBq2f0eINB34AzhVEgsln_KDo-IyeM8kc-dTzSCr0c
```

```
GET /parser-enforce?jwt=http 
  -v localhost:8080/parser-enforce?jwt=eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJoYXNNb3RvcmN5Y2xlIjpmYWxzZSwic3ViIjoibXNpbHZlcm1hbiJ9.8LBq2f0eINB34AzhVEgsln_KDo-IyeM8kc-dTzSCr0c HTTP/1.1
Accept: */*
...
HTTP/1.1 400 Bad Request
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Connection: close
Content-Type: application/json;charset=UTF-8
...
{
    "exceptionType": "io.jsonwebtoken.IncorrectClaimException",
    "message": "Expected hasMotorcycle claim to be: true, but was: false.",
    "status": "ERROR"
}
```

这表明我们的 hasMotorcycle 声明存在，但是有一个不期望的值。

`MissingClaimException`和`IncorrectClaimException`是你在 JWT 中执行声明时的朋友，也是只有 JJWT 库才有的特性。

### 7.2。JWT 压缩

如果你对一个 JWT 有很多声明，它可能会变得很大——非常大，以至于在一些浏览器中它可能不适合一个 GET url。

让我们来做一个大 JWT:

```
http -v POST localhost:8080/dynamic-builder-specific \
  iss=Stormpath hasMotorcycle:=true sub=msilverman the=quick brown=fox jumped=over lazy=dog \
  somewhere=over rainbow=way up=high and=the dreams=you dreamed=of
```

这是 JWT 生产的:

```
eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJTdG9ybXBhdGgiLCJoYXNNb3RvcmN5Y2xlIjp0cnVlLCJzdWIiOiJtc2lsdmVybWFuIiwidGhlIjoicXVpY2siLCJicm93biI6ImZveCIsImp1bXBlZCI6Im92ZXIiLCJsYXp5IjoiZG9nIiwic29tZXdoZXJlIjoib3ZlciIsInJhaW5ib3ciOiJ3YXkiLCJ1cCI6ImhpZ2giLCJhbmQiOiJ0aGUiLCJkcmVhbXMiOiJ5b3UiLCJkcmVhbWVkIjoib2YifQ.AHNJxSTiDw_bWNXcuh-LtPLvSjJqwDvOOUcmkk7CyZA
```

那玩意儿真大！现在，让我们使用相同的声明来达到稍微不同的终点:

```
http -v POST localhost:8080/dynamic-builder-compress \
  iss=Stormpath hasMotorcycle:=true sub=msilverman the=quick brown=fox jumped=over lazy=dog \
  somewhere=over rainbow=way up=high and=the dreams=you dreamed=of
```

这一次，我们得到:

```
eyJhbGciOiJIUzI1NiIsImNhbGciOiJERUYifQ.eNpEzkESwjAIBdC7sO4JegdXnoC2tIk2oZLEGB3v7s84jjse_AFe5FOikc5ZLRycHQ3kOJ0Untu8C43ZigyUyoRYSH6_iwWOyGWHKd2Kn6_QZFojvOoDupRwyAIq4vDOzwYtugFJg1QnJv-5sY-TVjQqN7gcKJ3f-j8c-6J-baDFhEN_uGn58XtnpfcHAAD__w.3_wc-2skFBbInk0YAQ96yGWwr8r1xVdbHn-uGPTFuFE
```

短了 62 个字符！以下是用于生成 JWT 的方法的代码:

```
@RequestMapping(value = "/dynamic-builder-compress", method = POST)
public JwtResponse dynamicBuildercompress(@RequestBody Map<String, Object> claims) 
  throws UnsupportedEncodingException {
    String jws =  Jwts.builder()
        .setClaims(claims)
        .compressWith(CompressionCodecs.DEFLATE)
        .signWith(
            SignatureAlgorithm.HS256,
            secretService.getHS256SecretBytes()
        )
        .compact();
    return new JwtResponse(jws);
}
```

请注意，在第 6 行，我们指定了要使用的压缩算法。这就是全部了。

解析压缩的 jwt 怎么样？JJWT 库自动检测压缩，并使用相同的算法来解压缩:

```
GET /parser?jwt=eyJhbGciOiJIUzI1NiIsImNhbGciOiJERUYifQ.eNpEzkESwjAIBdC7sO4JegdXnoC2tIk2oZLEGB3v7s84jjse_AFe5FOikc5ZLRycHQ3kOJ0Untu8C43ZigyUyoRYSH6_iwWOyGWHKd2Kn6_QZFojvOoDupRwyAIq4vDOzwYtugFJg1QnJv-5sY-TVjQqN7gcKJ3f-j8c-6J-baDFhEN_uGn58XtnpfcHAAD__w.3_wc-2skFBbInk0YAQ96yGWwr8r1xVdbHn-uGPTFuFE HTTP/1.1
Accept: */*
...
HTTP/1.1 200 OK
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Content-Type: application/json;charset=UTF-8
...
{
    "claims": {
        "body": {
            "and": "the",
            "brown": "fox",
            "dreamed": "of",
            "dreams": "you",
            "hasMotorcycle": true,
            "iss": "Stormpath",
            "jumped": "over",
            "lazy": "dog",
            "rainbow": "way",
            "somewhere": "over",
            "sub": "msilverman",
            "the": "quick",
            "up": "high"
        },
        "header": {
            "alg": "HS256",
            "calg": "DEF"
        },
        "signature": "3_wc-2skFBbInk0YAQ96yGWwr8r1xVdbHn-uGPTFuFE"
    },
    "status": "SUCCESS"
}
```

注意标题中的`calg`声明。这被自动编码到 JWT 中，并提示解析器使用什么算法进行解压缩。

注意:JWE 规范不支持压缩。在 JJWT 库即将发布的版本中，我们将支持 JWE 和压缩 JWEs。我们将继续在其他类型的 jwt 中支持压缩，即使它没有被指定。

## 8。Java 开发人员的令牌工具

虽然本文的核心重点不是 Spring Boot 或 Spring Security，但是使用这两种技术可以很容易地演示本文中讨论的所有特性。您应该能够启动服务器并开始使用我们讨论过的各种端点。只需点击:

```
http http://localhost:8080
```

Stormpath 也很高兴为 Java 社区带来大量开源开发工具。其中包括:

### 8.1。JJWT(我们一直在谈论的)

JJWT 是一个易于使用的[工具，开发者可以用 Java](https://web.archive.org/web/20220701015023/https://stormpath.com/blog/jjwt-how-it-works-why) 创建和验证 JWT。和 Stormpath 支持的很多库一样，JJWT 是完全免费开源的(Apache License，2.0 版)，所以每个人都可以看到它做了什么，怎么做的。不要犹豫报告任何问题，建议改进，甚至提交一些代码！

### 8.2。jsonwebtoken.io 和 java.jsonwebtoken.io

jsonwebtoken.io 是一个开发者工具，我们创建它是为了使 jwt 的解码变得容易。只需将现有的 JWT 粘贴到适当的字段中，即可解码其报头、有效载荷和签名。jsonwebtoken.io 由 [nJWT](https://web.archive.org/web/20220701015023/https://github.com/jwtk/njwt) 提供支持，这是面向 Node.js 开发者的最干净的免费开源(Apache License，2.0 版)JWT 库。您还可以在这个网站上看到为各种语言生成的代码。网站本身是开源的，可以在[这里](https://web.archive.org/web/20220701015023/https://github.com/stormpath/jsonwebtoken.io)找到。

[java.jsonwebtoken.io](https://web.archive.org/web/20220701015023/http://java.jsonwebtoken.io/) 专门针对 JJWT 库。您可以在右上角的框中修改头部和有效负载，在左上角的框中查看由 JJWT 生成的 JWT，在下方的框中查看构建器和解析器 Java 代码的示例。网站本身是开源的，可以在[这里](https://web.archive.org/web/20220701015023/https://github.com/stormpath/JJWTsite)找到。

### 8.3。JWT 督察

新来的家伙， [JWT 检查器](https://web.archive.org/web/20220701015023/https://www.jwtinspector.io/)是一个开源的 Chrome 扩展，允许开发者直接在浏览器中检查和调试 JWT。JWT 检查员将发现你的网站上的 jwt(在 cookies，本地/会话存储，和头),并通过你的导航栏和 DevTools 面板轻松访问它们。

## 9.JWT 该下来了！

jwt 为普通令牌增加了一些智能。加密签名和验证、内置到期时间以及将其他信息编码到 jwt 中的能力为真正的无状态会话管理奠定了基础。这对扩展应用程序的能力有很大的影响。

在 Stormpath，我们将 jwt 用于 OAuth2 令牌、CSRF 令牌和微服务之间的断言，以及其他用途。

一旦你开始使用 jwt，你可能永远不会回到过去的愚蠢的令牌。有什么问题吗？在推特上用 [@afitnerd](https://web.archive.org/web/20220701015023/https://twitter.com/afitnerd) 联系我。