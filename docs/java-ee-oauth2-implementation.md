# 使用 Jakarta EE 实现 OAuth 2.0 授权框架

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-oauth2-implementation>

## 1。概述

在本教程中，我们将使用 Jakarta EE 和 MicroProfile 为 [OAuth 2.0 授权框架](https://web.archive.org/web/20220628143947/https://tools.ietf.org/html/rfc6749)提供实现。最重要的是，我们将通过[授权码授权类型](https://web.archive.org/web/20220628143947/https://tools.ietf.org/html/rfc6749#page-24)实现 [OAuth 2.0 角色](https://web.archive.org/web/20220628143947/https://tools.ietf.org/html/rfc6749#page-6)的交互。撰写本文的动机是为了支持使用 Jakarta EE 实现的项目，因为它还不支持 OAuth。

对于最重要的角色，即授权服务器，**我们将实现授权端点、令牌端点以及 JWK 密钥端点**，这对资源服务器检索公钥很有用。

因为我们希望实现简单且易于快速设置，所以我们将使用一个预先注册的客户端和用户商店，显然还有一个用于访问令牌的 JWT 商店。

在进入主题之前，需要注意的是，本教程中的例子是出于教育目的。对于生产系统，强烈推荐使用成熟的、经过良好测试的解决方案，比如 [Keycloak](/web/20220628143947/https://www.baeldung.com/spring-boot-keycloak) 。

## 2.OAuth 2.0 概述

在本节中，我们将简要概述 OAuth 2.0 的角色和授权代码授权流程。

### 2.1.角色

OAuth 2.0 框架意味着以下四个角色之间的协作:

*   通常，这是最终用户——拥有一些值得保护的资源的实体
*   保护资源所有者数据的服务，通常通过 REST API 发布
*   `Client`:使用资源所有者数据的应用程序
*   `Authorization Server`:以过期令牌的形式向客户端授予权限的应用程序

### 2.2.授权类型

`grant type `是客户端如何获得使用资源所有者数据的许可，最终以访问令牌的形式。

自然，不同类型的客户[喜欢不同类型的赠款](https://web.archive.org/web/20220628143947/https://oauth2.thephpleague.com/authorization-server/which-grant/):

*   `Authorization Code` : **最常首选** `– `无论是**web 应用、本地应用还是单页面应用**，尽管本地和单页面应用需要称为 PKCE 的额外保护
*   `Refresh Token`:一种特殊的更新授权，**适用于 web 应用**更新他们现有的令牌
*   `Client Credentials`:首选用于**服务对服务通信**，比如说当资源所有者不是最终用户时
*   `Resource Owner` `Password`:当手机 app 需要自己的登录页面时，首选本机应用`, ` 的**第一方认证**

此外，客户端可以使用`implicit`授权类型。然而，在 PKCE 中使用授权码通常更安全。

### 2.3.授权码授权流

由于授权代码授权流是最常见的，让我们也回顾一下它是如何工作的，这实际上是我们将在本教程中构建的。

应用程序——客户端—**,通过重定向到授权服务器的`/authorize`端点来请求许可。**对于这个端点，应用程序给出一个`callback` 端点。

授权服务器通常会请求最终用户(资源所有者)的许可。如果终端用户授予许可，那么**授权服务器重定向回带有`code`的回调**。

应用程序接收这个代码，然后**对授权服务器的`/token `端点进行认证调用。**通过“认证”，我们的意思是应用程序证明谁是这个调用的一部分。如果一切正常，授权服务器将使用令牌进行响应。

有了令牌，**应用程序向 API** (资源服务器)发出请求，API 将验证令牌。它可以要求授权服务器使用其`/introspect`端点来验证令牌。或者，如果令牌是独立的，资源服务器可以通过**本地验证令牌的签名来进行优化，就像 JWT 的情况一样。**

### 2.4.雅加达 EE 支持什么？

还不多。在本教程中，我们将从头开始构建大多数东西。

## 3.OAuth 2.0 授权服务器

在这个实现中，我们将关注最常用的授权类型:授权码**。**

### 3.1.客户端和用户注册

当然，授权服务器在授权客户和用户的请求之前，需要了解他们的情况。对于授权服务器来说，拥有这样的用户界面是很常见的。

不过，为了简单起见，我们将使用一个预配置的客户端:

```java
INSERT INTO clients (client_id, client_secret, redirect_uri, scope, authorized_grant_types) 
VALUES ('webappclient', 'webappclientsecret', 'http://localhost:9180/callback', 
  'resource.read resource.write', 'authorization_code refresh_token');
```

```java
@Entity
@Table(name = "clients")
public class Client {
    @Id
    @Column(name = "client_id")
    private String clientId;
    @Column(name = "client_secret")
    private String clientSecret;

    @Column(name = "redirect_uri")
    private String redirectUri;

    @Column(name = "scope")
    private String scope;

    // ...
}
```

和预先配置的用户:

```java
INSERT INTO users (user_id, password, roles, scopes)
VALUES ('appuser', 'appusersecret', 'USER', 'resource.read resource.write');
```

```java
@Entity
@Table(name = "users")
public class User implements Principal {
    @Id
    @Column(name = "user_id")
    private String userId;

    @Column(name = "password")
    private String password;

    @Column(name = "roles")
    private String roles;

    @Column(name = "scopes")
    private String scopes;

    // ...
}
```

请注意，出于本教程的目的，我们使用了纯文本的密码，**，但是在生产环境中，它们应该被散列化**。

在本教程的剩余部分，我们将展示`appuser –` 资源所有者如何通过实现授权代码来授予对`webappclient`(应用程序)的访问权。

### 3.2.授权端点

授权端点的主要作用是首先**认证用户，然后请求应用程序想要的权限**——或者范围。

按照 OAuth2 规范的指示[，这个端点应该支持 HTTP GET 方法，尽管它也可以支持 HTTP POST 方法。在这个实现中，我们将只支持 HTTP GET 方法。](https://web.archive.org/web/20220628143947/https://tools.ietf.org/html/rfc6749#section-3.1)

首先，**授权端点要求对用户进行身份验证**。规范在这里不要求某种方式，所以让我们使用来自 [Jakarta EE 8 安全 API](/web/20220628143947/https://www.baeldung.com/java-ee-8-security) 的表单认证:

```java
@FormAuthenticationMechanismDefinition(
  loginToContinue = @LoginToContinue(loginPage = "/login.jsp", errorPage = "/login.jsp")
)
```

用户将被重定向到`/login.jsp`进行身份验证，然后将作为`CallerPrincipal`通过 S `ecurityContext` API 提供:

```java
Principal principal = securityContext.getCallerPrincipal();
```

我们可以用 JAX 遥感器把这些放在一起:

```java
@FormAuthenticationMechanismDefinition(
  loginToContinue = @LoginToContinue(loginPage = "/login.jsp", errorPage = "/login.jsp")
)
@Path("authorize")
public class AuthorizationEndpoint {
    //...    
    @GET
    @Produces(MediaType.TEXT_HTML)
    public Response doGet(@Context HttpServletRequest request,
      @Context HttpServletResponse response,
      @Context UriInfo uriInfo) throws ServletException, IOException {

        MultivaluedMap<String, String> params = uriInfo.getQueryParameters();
        Principal principal = securityContext.getCallerPrincipal();
        // ...
    }
}
```

此时，授权端点可以开始处理应用程序的请求，该请求必须包含 **`response_type`和`client_id`参数，以及`redirect_uri, scope,` 和`state`参数——这是可选的，但也是推荐的。**

在我们的例子中，`client_id` 应该是一个有效的客户机，来自于`clients` 数据库表。

如果指定了`redirect_uri`，它也应该与我们在`clients` 数据库表中找到的相匹配。

因为我们做的是授权码，`response_type` 就是`code. `

由于授权是一个多步骤的过程，我们可以在会话中临时存储这些值:

```java
request.getSession().setAttribute("ORIGINAL_PARAMS", params);
```

然后准备询问用户应用程序可以使用哪些权限，重定向到该页面:

```java
String allowedScopes = checkUserScopes(user.getScopes(), requestedScope);
request.setAttribute("scopes", allowedScopes);
request.getRequestDispatcher("/authorize.jsp").forward(request, response);
```

### 3.3.用户范围批准

此时，浏览器为用户呈现一个授权 UI，**用户做出选择。**然后，浏览器**提交用户在** **的选择，一个 HTTP POST** :

```java
@POST
@Consumes(MediaType.APPLICATION_FORM_URLENCODED)
@Produces(MediaType.TEXT_HTML)
public Response doPost(@Context HttpServletRequest request, @Context HttpServletResponse response,
  MultivaluedMap<String, String> params) throws Exception {
    MultivaluedMap<String, String> originalParams = 
      (MultivaluedMap<String, String>) request.getSession().getAttribute("ORIGINAL_PARAMS");

    // ...

    String approvalStatus = params.getFirst("approval_status"); // YES OR NO

    // ... if YES

    List<String> approvedScopes = params.get("scope");

    // ...
}
```

接下来，我们生成一个临时代码，该代码引用了 **`user_id, client_id,` 和** **`redirect_uri,`** ，应用程序稍后在到达令牌端点时将使用所有这些代码。

因此，让我们创建一个带有自动生成的 id `:`的`AuthorizationCode` JPA 实体

```java
@Entity
@Table(name ="authorization_code")
public class AuthorizationCode {
@Id
@GeneratedValue(strategy=GenerationType.AUTO)
@Column(name = "code")
private String code;

//...

}
```

然后填充它:

```java
AuthorizationCode authorizationCode = new AuthorizationCode();
authorizationCode.setClientId(clientId);
authorizationCode.setUserId(userId);
authorizationCode.setApprovedScopes(String.join(" ", authorizedScopes));
authorizationCode.setExpirationDate(LocalDateTime.now().plusMinutes(2));
authorizationCode.setRedirectUri(redirectUri);
```

当我们保存 bean 时，code 属性是自动填充的，因此我们可以获取它并将其发送回客户端:

```java
appDataRepository.save(authorizationCode);
String code = authorizationCode.getCode();
```

请注意，**我们的授权码将在两分钟后**到期——我们应该尽可能保守地对待这个到期日。它可以很短，因为客户端会立即将其交换为访问令牌。

然后我们重定向回应用程序的`redirect_uri,` ，给它代码以及应用程序在其`/authorize` 请求中指定的任何`state` 参数:

```java
StringBuilder sb = new StringBuilder(redirectUri);
// ...

sb.append("?code=").append(code);
String state = params.getFirst("state");
if (state != null) {
    sb.append("&state;=").append(state);
}
URI location = UriBuilder.fromUri(sb.toString()).build();
return Response.seeOther(location).build();
```

再次注意， **`redirectUri`是存在于`clients`表中的任何东西，而不是`redirect_uri` 请求参数。**

因此，我们的下一步是让客户端接收此代码，并使用令牌端点将其交换为访问令牌。

### 3.4.令牌端点

与授权端点相反，令牌端点**不需要浏览器来与客户端**通信，因此，我们将把它实现为 JAX-RS 端点:

```java
@Path("token")
public class TokenEndpoint {

    List<String> supportedGrantTypes = Collections.singletonList("authorization_code");

    @Inject
    private AppDataRepository appDataRepository;

    @Inject
    Instance<AuthorizationGrantTypeHandler> authorizationGrantTypeHandlers;

    @POST
    @Produces(MediaType.APPLICATION_JSON)
    @Consumes(MediaType.APPLICATION_FORM_URLENCODED)
    public Response token(MultivaluedMap<String, String> params,
       @HeaderParam(HttpHeaders.AUTHORIZATION) String authHeader) throws JOSEException {
        //...
    }
}
```

令牌端点需要 POST，以及使用`application/x-www-form-urlencoded`媒体类型对参数进行编码。

正如我们所讨论的，我们将只支持`authorization code`授权类型:

```java
List<String> supportedGrantTypes = Collections.singletonList("authorization_code");
```

因此，应该支持接收到的`grant_type`作为必需参数:

```java
String grantType = params.getFirst("grant_type");
Objects.requireNonNull(grantType, "grant_type params is required");
if (!supportedGrantTypes.contains(grantType)) {
    JsonObject error = Json.createObjectBuilder()
      .add("error", "unsupported_grant_type")
      .add("error_description", "grant type should be one of :" + supportedGrantTypes)
      .build();
    return Response.status(Response.Status.BAD_REQUEST)
      .entity(error).build();
}
```

接下来，我们通过 HTTP 基本身份验证检查客户端身份验证。也就是说，我们检查**是否通过`Authorization` 报头接收到的`client_id`和`client_secret`** `,` ，**匹配一个注册的客户:**

```java
String[] clientCredentials = extract(authHeader);
String clientId = clientCredentials[0];
String clientSecret = clientCredentials[1];
Client client = appDataRepository.getClient(clientId);
if (client == null || clientSecret == null || !clientSecret.equals(client.getClientSecret())) {
    JsonObject error = Json.createObjectBuilder()
      .add("error", "invalid_client")
      .build();
    return Response.status(Response.Status.UNAUTHORIZED)
      .entity(error).build();
}
```

最后，我们将`TokenResponse`的生产委托给相应的授权类型处理程序:

```java
public interface AuthorizationGrantTypeHandler {
    TokenResponse createAccessToken(String clientId, MultivaluedMap<String, String> params) throws Exception;
}
```

由于我们对授权代码授权类型更感兴趣，我们提供了一个适当的 CDI bean 实现，并用`Named`注释对其进行了修饰:

```java
@Named("authorization_code")
```

运行时，根据接收到的`grant_type`值，通过 [CDI 实例机制](https://web.archive.org/web/20220628143947/https://javaee.github.io/javaee-spec/javadocs/javax/enterprise/inject/Instance.html)激活相应的实现:

```java
String grantType = params.getFirst("grant_type");
//...
AuthorizationGrantTypeHandler authorizationGrantTypeHandler = 
  authorizationGrantTypeHandlers.select(NamedLiteral.of(grantType)).get();
```

现在是时候做出`/token`的回应了。

### 3.5.`RSA`私钥和公钥

在生成令牌之前，我们需要一个 RSA 私钥来签署令牌。

为此，我们将使用 OpenSSL:

```java
# PRIVATE KEY
openssl genpkey -algorithm RSA -out private-key.pem -pkeyopt rsa_keygen_bits:2048
```

使用文件`META-INF/microprofile-config.properties:`通过微文件配置`signingKey`属性将`private-key.pem`提供给服务器

```java
signingkey=/META-INF/private-key.pem
```

服务器可以使用注入的`Config`对象读取属性:

```java
String signingkey = config.getValue("signingkey", String.class);
```

类似地，我们可以生成相应的公钥:

```java
# PUBLIC KEY
openssl rsa -pubout -in private-key.pem -out public-key.pem
```

并使用微文件配置`verificationKey`来读取它:

```java
verificationkey=/META-INF/public-key.pem
```

出于验证的目的，服务器应该使其对资源服务器可用。这是通过 JWK 端点**完成的。**

**[光轮穆+JWT](https://web.archive.org/web/20220628143947/https://connect2id.com/products/nimbus-jose-jwt)** 是一个可以在这里帮上大忙的库。让我们先添加[的`nimbus-jose-jwt` 依赖](https://web.archive.org/web/20220628143947/https://search.maven.org/search?q=g:com.nimbusds AND a:nimbus-jose-jwt&core=gav):

```java
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>7.7</version>
</dependency>
```

现在，我们可以利用 Nimbus 的 JWK 支持来简化我们的端点:

```java
@Path("jwk")
@ApplicationScoped
public class JWKEndpoint {

    @GET
    public Response getKey(@QueryParam("format") String format) throws Exception {
        //...

        String verificationkey = config.getValue("verificationkey", String.class);
        String pemEncodedRSAPublicKey = PEMKeyUtils.readKeyAsString(verificationkey);
        if (format == null || format.equals("jwk")) {
            JWK jwk = JWK.parseFromPEMEncodedObjects(pemEncodedRSAPublicKey);
            return Response.ok(jwk.toJSONString()).type(MediaType.APPLICATION_JSON).build();
        } else if (format.equals("pem")) {
            return Response.ok(pemEncodedRSAPublicKey).build();
        }

        //...
    }
}
```

我们已经使用格式`parameter`在 PEM 和 JWK 格式之间切换。我们将用来实现资源服务器的微文件 JWT 支持这两种格式。

### 3.6.令牌端点响应

现在是给定的`AuthorizationGrantTypeHandler`创建令牌响应的时候了。在这个实现中，我们将只支持结构化的 JWT 令牌。

为了创建这种格式的令牌，我们将再次使用 [Nimbus JOSE+JWT](https://web.archive.org/web/20220628143947/https://connect2id.com/products/nimbus-jose-jwt) 库，但是还有[许多其他的 JWT 库](https://web.archive.org/web/20220628143947/https://jwt.io/#libraries-io)。

因此，要创建一个签名的 JWT，**我们首先要构造 JWT 报头:**

```java
JWSHeader jwsHeader = new JWSHeader.Builder(JWSAlgorithm.RS256).type(JOSEObjectType.JWT).build();
```

**然后，我们构建有效负载**，它是标准化和定制声明的`Set`:

```java
Instant now = Instant.now();
Long expiresInMin = 30L;
Date in30Min = Date.from(now.plus(expiresInMin, ChronoUnit.MINUTES));

JWTClaimsSet jwtClaims = new JWTClaimsSet.Builder()
  .issuer("http://localhost:9080")
  .subject(authorizationCode.getUserId())
  .claim("upn", authorizationCode.getUserId())
  .audience("http://localhost:9280")
  .claim("scope", authorizationCode.getApprovedScopes())
  .claim("groups", Arrays.asList(authorizationCode.getApprovedScopes().split(" ")))
  .expirationTime(in30Min)
  .notBeforeTime(Date.from(now))
  .issueTime(Date.from(now))
  .jwtID(UUID.randomUUID().toString())
  .build();
SignedJWT signedJWT = new SignedJWT(jwsHeader, jwtClaims);
```

除了标准的 JWT 声明之外，我们还添加了两个声明——`upn`和`groups`——因为它们是微概要文件 JWT 所需要的。`upn`将被映射到雅加达 EE 安全`CallerPrincipal`，而`groups`将被映射到雅加达 EE `Roles.`

现在我们有了头部和有效载荷，**我们需要用 RSA 私钥**对访问令牌进行签名。相应的 RSA 公钥将通过 JWK 端点公开，或者通过其他方式提供，以便资源服务器可以使用它来验证访问令牌。

因为我们已经以 PEM 格式提供了私钥，所以我们应该检索它并将其转换成一个`RSAPrivateKey:`

```java
SignedJWT signedJWT = new SignedJWT(jwsHeader, jwtClaims);
//...
String signingkey = config.getValue("signingkey", String.class);
String pemEncodedRSAPrivateKey = PEMKeyUtils.readKeyAsString(signingkey);
RSAKey rsaKey = (RSAKey) JWK.parseFromPEMEncodedObjects(pemEncodedRSAPrivateKey);
```

接下来，**我们签署并序列化 JWT:**

```java
signedJWT.sign(new RSASSASigner(rsaKey.toRSAPrivateKey()));
String accessToken = signedJWT.serialize();
```

最后**我们构造一个令牌响应:**

```java
return Json.createObjectBuilder()
  .add("token_type", "Bearer")
  .add("access_token", accessToken)
  .add("expires_in", expiresInMin * 60)
  .add("scope", authorizationCode.getApprovedScopes())
  .build();
```

多亏了 JSON-P，它被序列化为 JSON 格式并发送给客户端:

```java
{
  "access_token": "acb6803a48114d9fb4761e403c17f812",
  "token_type": "Bearer",  
  "expires_in": 1800,
  "scope": "resource.read resource.write"
}
```

## 4.OAuth 2.0 客户端

在本节中，我们将**使用 Servlet、MicroProfile Config 和 JAX RS 客户端 API 构建一个基于 web 的 OAuth 2.0 客户端**。

更准确地说，我们将实现两个主要的 servlet:一个用于请求授权服务器的授权端点并使用授权代码授权类型获取代码，另一个 servlet 用于使用接收到的代码并从授权服务器的令牌端点请求访问令牌。

此外，我们将实现另外两个 servlets:一个用于使用刷新令牌授权类型获取新的访问令牌，另一个用于访问资源服务器的 API。

### 4.1.OAuth 2.0 客户端详细信息

由于客户端已经在授权服务器中注册，我们首先需要提供客户端注册信息:

*   客户端标识符，通常由授权服务器在注册过程中发布。
*   `client_secret:`客户秘密。
*   `redirect_uri:`接收授权码的位置。
*   `scope:`客户端请求的权限。

此外，客户端应该知道授权服务器的授权和令牌端点:

*   `authorization_uri:`我们可以用来获取代码的授权服务器授权端点的位置。
*   `token_uri:`我们可以用来获取令牌的授权服务器令牌端点的位置。

所有这些信息都是通过微配置文件`META-INF/microprofile-config.properties:`提供的

```java
# Client registration
client.clientId=webappclient
client.clientSecret=webappclientsecret
client.redirectUri=http://localhost:9180/callback
client.scope=resource.read resource.write

# Provider
provider.authorizationUri=http://127.0.0.1:9080/authorize
provider.tokenUri=http://127.0.0.1:9080/token
```

### 4.2.授权码请求

**获取授权码的流程从客户端开始，将浏览器重定向到授权服务器的授权端点。**

通常，当用户试图在未经授权的情况下访问受保护的资源 API 时，或者通过显式调用客户端`/authorize`路径时，会发生这种情况:

```java
@WebServlet(urlPatterns = "/authorize")
public class AuthorizationCodeServlet extends HttpServlet {

    @Inject
    private Config config;

    @Override
    protected void doGet(HttpServletRequest request, 
      HttpServletResponse response) throws ServletException, IOException {
        //...
    }
}
```

在`doGet()`方法中，我们从生成和存储安全状态值开始:

```java
String state = UUID.randomUUID().toString();
request.getSession().setAttribute("CLIENT_LOCAL_STATE", state);
```

然后，我们检索客户端配置信息:

```java
String authorizationUri = config.getValue("provider.authorizationUri", String.class);
String clientId = config.getValue("client.clientId", String.class);
String redirectUri = config.getValue("client.redirectUri", String.class);
String scope = config.getValue("client.scope", String.class);
```

然后，我们将这些信息作为查询参数附加到授权服务器的授权端点:

```java
String authorizationLocation = authorizationUri + "?response_type=code"
  + "&client;_id=" + clientId
  + "&redirect;_uri=" + redirectUri
  + "&scope;=" + scope
  + "&state;=" + state;
```

最后，我们将浏览器重定向到以下 URL:

```java
response.sendRedirect(authorizationLocation);
```

在处理请求之后，**授权服务器的授权端点将生成代码**，并且除了接收到的状态参数之外，将代码附加到`redirect_uri`，并且将重定向回浏览器 [`http://localhost:9081/callback?code=A123&state;=Y`](https://web.archive.org/web/20220628143947/http://localhost:9081/callback?code=A123&state=Y) 。

### 4.3.访问令牌请求

客户端回调 servlet，`/callback,`通过验证接收到的`state:`开始

```java
String localState = (String) request.getSession().getAttribute("CLIENT_LOCAL_STATE");
if (!localState.equals(request.getParameter("state"))) {
    request.setAttribute("error", "The state attribute doesn't match!");
    dispatch("/", request, response);
    return;
}
```

接下来，**我们将使用之前收到的代码，通过授权服务器的令牌端点请求访问令牌**:

```java
String code = request.getParameter("code");
Client client = ClientBuilder.newClient();
WebTarget target = client.target(config.getValue("provider.tokenUri", String.class));

Form form = new Form();
form.param("grant_type", "authorization_code");
form.param("code", code);
form.param("redirect_uri", config.getValue("client.redirectUri", String.class));

TokenResponse tokenResponse = target.request(MediaType.APPLICATION_JSON_TYPE)
  .header(HttpHeaders.AUTHORIZATION, getAuthorizationHeaderValue())
  .post(Entity.entity(form, MediaType.APPLICATION_FORM_URLENCODED_TYPE), TokenResponse.class);
```

正如我们所看到的，这个调用没有浏览器交互，请求是直接使用 JAX-RS 客户端 API 作为 HTTP POST 发出的。

由于令牌端点需要客户端身份验证，我们在`Authorization`头中包含了客户端凭证`client_id`和`client_secret`。

客户机可以使用这个访问令牌来调用资源服务器 API，这是下一小节的主题。

### 4.4.受保护的资源访问

**此时，我们有了一个有效的访问令牌，我们可以调用资源服务器的/ `read`和/`write`API。**

为此，**我们必须提供`Authorization`报头**。使用 JAX-RS 客户端 API，这可以通过`Invocation.Builder header()`方法简单地完成:

```java
resourceWebTarget = webTarget.path("resource/read");
Invocation.Builder invocationBuilder = resourceWebTarget.request();
response = invocationBuilder
  .header("authorization", tokenResponse.getString("access_token"))
  .get(String.class);
```

## 5.OAuth 2.0 资源服务器

在这一节中，我们将基于 JAX-RS、微文件 JWT 和微文件配置构建一个安全的 web 应用程序。微配置文件 JWT 负责验证收到的 JWT，并将 JWT 范围映射到 Jakarta EE 角色。

### 5.1.Maven 依赖性

除了对 [Java EE Web API](https://web.archive.org/web/20220628143947/https://search.maven.org/search?q=g:javax AND a:javaee-web-api&core=gav) 的依赖，我们还需要[微配置](https://web.archive.org/web/20220628143947/https://search.maven.org/search?q=g:org.eclipse.microprofile.config AND a:microprofile-config-api&core=gav)和[微配置 JWT](https://web.archive.org/web/20220628143947/https://search.maven.org/search?q=g:org.eclipse.microprofile.jwt AND a:microprofile-jwt-auth-api&core=gav)API:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-web-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.eclipse.microprofile.config</groupId>
    <artifactId>microprofile-config-api</artifactId>
    <version>1.3</version>
</dependency>
<dependency>
    <groupId>org.eclipse.microprofile.jwt</groupId>
    <artifactId>microprofile-jwt-auth-api</artifactId>
    <version>1.1</version>
</dependency>
```

### 5.2.JWT 认证机制

微配置文件 JWT 提供了承载令牌认证机制的实现。这负责处理出现在`Authorization`头中的 JWT，使 Jakarta EE 安全主体作为保存 JWT 声明的`JsonWebToken`可用，并将范围映射到 Jakarta EE 角色。查看 [Jakarta EE 安全 API](/web/20220628143947/https://www.baeldung.com/java-ee-8-security) 了解更多背景信息。

为了在服务器中启用 **JWT 认证机制，我们需要**在 JAX-RS 应用程序中添加`LoginConfig`注释**:**

```java
@ApplicationPath("/api")
@DeclareRoles({"resource.read", "resource.write"})
@LoginConfig(authMethod = "MP-JWT")
public class OAuth2ResourceServerApplication extends Application {
}
```

此外，**JWT 微档案需要 RSA 公钥来验证 JWT 签名**。我们可以通过自省来提供，或者为了简单起见，通过从授权服务器手动复制密钥来提供。无论哪种情况，我们都需要提供公钥的位置:

```java
mp.jwt.verify.publickey.location=/META-INF/public-key.pem
```

最后，微文件 JWT 需要验证传入 JWT 的`iss`声明，它应该存在并与微文件配置属性的值相匹配:

```java
mp.jwt.verify.issuer=http://127.0.0.1:9080
```

通常，这是授权服务器的位置。

### 5.3.安全端点

出于演示的目的，我们将添加一个带有两个端点的资源 API。一个是具有`resource.read`范围的用户可以访问的`read`端点，另一个是具有`resource.write`范围的用户可以访问的`write`端点。

对作用域的限制是通过`@RolesAllowed`注释完成的:

```java
@Path("/resource")
@RequestScoped
public class ProtectedResource {

    @Inject
    private JsonWebToken principal;

    @GET
    @RolesAllowed("resource.read")
    @Path("/read")
    public String read() {
        return "Protected Resource accessed by : " + principal.getName();
    }

    @POST
    @RolesAllowed("resource.write")
    @Path("/write")
    public String write() {
        return "Protected Resource accessed by : " + principal.getName();
    }
}
```

## 6.运行所有服务器

要运行一个服务器，我们只需要调用相应目录中的 Maven 命令:

```java
mvn package liberty:run-server
```

授权服务器、客户端和资源服务器将分别在以下位置运行和可用:

```java
# Authorization Server
http://localhost:9080/

# Client
http://localhost:9180/

# Resource Server
http://localhost:9280/ 
```

因此，我们可以访问客户端主页，然后单击“Get Access Token”启动授权流程。收到访问令牌后，我们可以访问资源服务器的`read`和`write`API。

根据被授予的作用域，资源服务器将通过一个成功的消息进行响应，或者我们将得到一个 HTTP 403 禁止状态。

## 7.结论

在本文中，我们提供了一个 OAuth 2.0 授权服务器的实现，它可以用于任何兼容的 OAuth 2.0 客户端和资源服务器。

为了解释总体框架，我们还提供了客户端和资源服务器的实现。为了实现所有这些组件，我们使用了 Jakarta EE 8 APIs，特别是 CDI、Servlet、JAX RS 和 Jakarta EE Security。此外，我们还使用了微文件的伪 Jakarta EE APIs:微文件配置和微文件 JWT。

GitHub 上的[提供了示例的完整源代码。注意，该代码包括授权代码和刷新令牌授权类型的示例。](https://web.archive.org/web/20220628143947/https://github.com/eugenp/tutorials/tree/master/oauth2-framework-impl)

最后，重要的是要意识到本文的教育性，并且给出的例子不应该用于生产系统。