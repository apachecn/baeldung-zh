# 使用 Auth0 的 Spring 安全性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-auth0>

## 1.概观

[Auth0](https://web.archive.org/web/20221126215056/https://auth0.com/) 为各种类型的应用提供**认证和授权服务，如本地、单页面应用和 Web** 。此外，它允许**实现各种功能，如单点登录、社交登录和多因素认证**。

在本教程中，我们将通过一步一步的指导来探索 Auth0 的 Spring 安全性，以及 Auth0 帐户的关键配置。

## 2.正在设置 Auth0

### 2.1.Auth0 注册

首先，我们将 **[注册一个免费的 Auth0 计划](https://web.archive.org/web/20221126215056/https://auth0.com/signup)，为多达 7k 的活跃用户提供无限制登录。**但是，如果我们已经有了一个，我们可以跳过这一部分:

[![](img/c9756fcb8240b8e692aeaf93851f70f2.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_2_1.png)

### 2.2.仪表盘

登录 Auth0 帐户后，我们将看到一个仪表板，其中突出显示了登录活动、最新登录和新注册等详细信息:

[![](img/ca22fe5ed8681b1a3eba72a97a8d2c89.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_2_2.png)

### 2.3.创建新的应用程序

然后，从应用程序菜单，我们将为 Spring Boot 创建一个新的 [OpenID Connect (OIDC)应用程序。](/web/20221126215056/https://www.baeldung.com/spring-security-openid-connect)

此外，我们将**从`Native`、`Single-Page Apps`和`Machine to Machine Apps`等可用选项中选择 `Regular Web Applications` 作为`application type` 、**:

[![](img/04923fc99f3def32829bf1cb0f30f7db.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_2_3.png)

### 2.4.应用程序设置

接下来，我们将配置几个`Application URIs`，如`Callback URLs`和`Logout URLs`指向我们的应用程序:

[![](img/e589eb2e896196fe0135da87a28a6c38.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_2_4.png)

### 2.5.客户端凭据

最后，我们将获得与我们的应用程序相关联的`Domain`、`Client ID,`和`Client Secret` 的值:

[![](img/63e189d685c05d1664ddc70e18dd10cc.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_2_5.png)

请准备好这些凭据，因为我们的 Spring Boot 应用程序中的 Auth0 设置需要这些凭据。

## 3.Spring Boot 应用程序设置

现在，我们的 Auth0 帐户已准备好关键配置，我们准备将 Auth0 安全性集成到 Spring Boot 应用程序中。

### 3.1.专家

首先，让我们将最新的 [`mvc-auth-commons`](https://web.archive.org/web/20221126215056/https://search.maven.org/search?q=g:com.auth0%20a:mvc-auth-commons) Maven 依赖添加到我们的`pom.xm` l:

```java
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>mvc-auth-commons</artifactId>
    <version>1.2.0</version>
</dependency>
```

### 3.2\. Gradle

类似地，当使用 Gradle 时，我们可以在`build.gradle`文件中添加`mvc-auth-commons`依赖项:

```java
compile 'com.auth0:mvc-auth-commons:1.2.0'
```

### 3.3.`application.properties`

我们的 Spring Boot 应用程序需要像`Client Id`和`Client Secret`这样的信息来启用 Auth0 帐户的认证。所以，我们将它们添加到`application.properties`文件中:

```java
com.auth0.domain: dev-example.auth0.com
com.auth0.clientId: {clientId}
com.auth0.clientSecret: {clientSecret}
```

### 3.4.`AuthConfig`

接下来，我们将创建`AuthConfig` 类来从`application.properties`文件中读取 Auth0 属性:

```java
@Configuration
@EnableWebSecurity
public class AuthConfig {
    @Value(value = "${com.auth0.domain}")
    private String domain;

    @Value(value = "${com.auth0.clientId}")
    private String clientId;

    @Value(value = "${com.auth0.clientSecret}")
    private String clientSecret;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf()
          .disable()
          .authorizeRequests()
          .antMatchers("/callback", "/login", "/").permitAll()
          .anyRequest().authenticated()
          .and()
          .formLogin()
          .loginPage("/login")
          .and()
          .logout().logoutSuccessHandler(logoutSuccessHandler()).permitAll();
        return http.build();
    }
}
```

此外，`AuthConfig`类被配置为[通过创建一个`SecurityFilterChain` bean](/web/20221126215056/https://www.baeldung.com/spring-boot-security-autoconfiguration#configuring-spring-boot-security) `.`来启用 web 安全性

### 3.5.`AuthenticationController`

最后，我们将为已经讨论过的`AuthConfig`类添加一个 [`AuthenticationController`](https://web.archive.org/web/20221126215056/https://javadoc.io/doc/com.auth0/mvc-auth-commons/latest/com/auth0/AuthenticationController.html) 类的 bean 引用:

```java
@Bean
public AuthenticationController authenticationController() throws UnsupportedEncodingException {
    JwkProvider jwkProvider = new JwkProviderBuilder(domain).build();
    return AuthenticationController.newBuilder(domain, clientId, clientSecret)
      .withJwkProvider(jwkProvider)
      .build();
}
```

这里，我们在构建`AuthenticationController`类的实例时使用了 [`JwkProviderBuilder`](https://web.archive.org/web/20221126215056/https://javadoc.io/doc/com.auth0/jwks-rsa/latest/com/auth0/jwk/JwkProviderBuilder.html) 类。我们将使用它来获取公钥以验证令牌的签名(默认情况下，令牌使用 RS256 非对称签名算法进行签名)。

此外，`authenticationController` bean 为登录提供授权 URL，并处理回调请求。

## 4.`AuthController`

接下来，我们将为登录和回调特性创建`AuthController`类:

```java
@Controller
public class AuthController {
    @Autowired
    private AuthConfig config;

    @Autowired 
    private AuthenticationController authenticationController;
}
```

这里，我们已经注入了上一节中讨论的`AuthConfig`和`AuthenticationController`类的依赖关系。

### 4.1.注册

让我们创建`login`方法，它允许我们的 Spring Boot 应用对用户进行身份验证:

```java
@GetMapping(value = "/login")
protected void login(HttpServletRequest request, HttpServletResponse response) {
    String redirectUri = "http://localhost:8080/callback";
    String authorizeUrl = authenticationController.buildAuthorizeUrl(request, response, redirectUri)
      .withScope("openid email")
      .build();
    response.sendRedirect(authorizeUrl);
}
```

`buildAuthorizeUrl` 方法生成 Auth0 authorize URL 并重定向到默认的 Auth0 登录屏幕。

### 4.2.回收

一旦用户使用 Auth0 凭据登录，回调请求将被发送到我们的 Spring Boot 应用程序。为此，让我们创建`callback`方法:

```java
@GetMapping(value="/callback")
public void callback(HttpServletRequest request, HttpServletResponse response) {
    Tokens tokens = authenticationController.handle(request, response);

    DecodedJWT jwt = JWT.decode(tokens.getIdToken());
    TestingAuthenticationToken authToken2 = new TestingAuthenticationToken(jwt.getSubject(),
      jwt.getToken());
    authToken2.setAuthenticated(true);

    SecurityContextHolder.getContext().setAuthentication(authToken2);
    response.sendRedirect(config.getContextPath(request) + "/"); 
}
```

我们处理回调请求以获得代表成功认证的`accessToken`和`idToken`。然后，我们创建了 [`TestingAuthenticationToken`](https://web.archive.org/web/20221126215056/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/TestingAuthenticationToken.html) 对象来[设置`SecurityContextHolder`](/web/20221126215056/https://www.baeldung.com/manually-set-user-authentication-spring-security) 中的认证。

然而，为了更好的可用性，我们可以创建我们的 [`AbstractAuthenticationToken`](https://web.archive.org/web/20221126215056/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AbstractAuthenticationToken.html) 类的实现。

## 5.`HomeController`

最后，我们将为应用程序的登录页面创建带有默认映射的`HomeController`:

```java
@Controller
public class HomeController {
    @GetMapping(value = "/")
    @ResponseBody
    public String home(final Authentication authentication) {
        TestingAuthenticationToken token = (TestingAuthenticationToken) authentication;
        DecodedJWT jwt = JWT.decode(token.getCredentials().toString());
        String email = jwt.getClaims().get("email").asString();
        return "Welcome, " + email + "!";
    }
}
```

这里，我们从`idToken`中提取了`[DecodedJWT](https://web.archive.org/web/20221126215056/https://www.javadoc.io/doc/com.auth0/java-jwt/latest/com/auth0/jwt/interfaces/DecodedJWT.html)`对象。此外，像电子邮件这样的用户信息是从声明中获取的。

就是这样！我们的 Spring Boot 应用已准备好提供 Auth0 安全支持。让我们使用 Maven 命令运行我们的应用程序:

```java
mvn spring-boot:run
```

在`[localhost:8080/login](https://web.archive.org/web/20221126215056/http://localhost:8080/login),`访问应用程序时，我们将看到 Auth0 提供的默认登录页面:

[![](img/3dcc1ee05202f5feadc5a1d09b55c33c.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_5_1.png)

使用注册用户的凭据登录后，将显示一条欢迎消息，其中包含用户的电子邮件:

[![](img/cb0a3af5d55841d408797cd0d7c89800.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_5_2.png)

此外，我们会在默认登录屏幕上找到一个“注册”按钮(在“登录”旁边)，用于自行注册。

## 6.注册

### 6.1.自行注册

第一次，我们可以使用“注册”按钮创建一个 Auth0 帐户，然后提供电子邮件和密码等信息:

[![](img/a20afc6adf45c4210ba61bd4afe60eff.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_6_1.png)

### 6.2.创建用户

或者，我们可以从 Auth0 帐户中的`Users`菜单创建一个新用户:

[![](img/ae99261ded72268372e88c3b60f9dd20.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_6_2.png)

### 6.3.连接设置

此外，我们可以选择各种类型的连接，如数据库和社交登录，以注册/登录我们的 Spring Boot 应用程序:

[![](img/1158e092b3e38df9fff33aee118029c4.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_6_3.png)

此外，还有一系列社会关系可供选择:

[![](img/fe01f971e4dcd639f1c0bd0bfc8b8104.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_6_4.png)

## 7.`LogoutController`

既然我们已经看到了登录和回调特性，我们可以在我们的 Spring Boot 应用程序中添加一个[注销特性。](/web/20221126215056/https://www.baeldung.com/spring-security-logout)

让我们创建实现`LogoutSuccessHandler`类的`LogoutController`类:

```java
@Controller
public class LogoutController implements LogoutSuccessHandler {
    @Autowired
    private AuthConfig config;

    @Override
    public void onLogoutSuccess(HttpServletRequest req, HttpServletResponse res, 
      Authentication authentication) {
        if (req.getSession() != null) {
            req.getSession().invalidate();
        }
        String returnTo = "http://localhost:8080/";
        String logoutUrl = "https://dev-example.auth0.com/v2/logout?client_id=" +
          config.getClientId() + "&returnTo;=" +returnTo;
        res.sendRedirect(logoutUrl);
    }
}
```

这里，`onLogoutSuccess`方法被覆盖以调用`/v2/logout` Auth0 注销 URL。

## 8.Auth0 管理 API

到目前为止，我们已经在 Spring Boot 应用中看到了 Auth0 安全集成。现在，让我们在同一个 app 中与 Auth0 管理 API(系统 API)进行交互。

### 8.1.创建新的应用程序

首先，为了访问 Auth0 管理 API，我们将在 Auth0 帐户中创建一个`Machine to Machine Application` :

[![](img/6a9ee91cf88345060deec1439062686c.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_8_1.png)

### 8.2.批准

然后，我们将向 Auth0 管理 API 添加授权，允许读取/创建用户:

[![](img/5c4e7ea1181cd7538cff4100fb2b558a.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_8_2.png)

### 8.3.客户端凭据

最后，我们将收到从我们的 Spring Boot 应用程序访问 Auth0 管理应用程序的`Client Id`和`Client Secret`:

[![](img/9dbcb707f04bdc6796f2f55393e5169c.png)](/web/20221126215056/https://www.baeldung.com/wp-content/uploads/2020/05/auto0_8_3.png)

### 8.4.访问令牌

让我们使用上一节中收到的客户端凭据为 Auth0 管理应用程序生成一个访问令牌:

```java
public String getManagementApiToken() {
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);

    JSONObject requestBody = new JSONObject();
    requestBody.put("client_id", "auth0ManagementAppClientId");
    requestBody.put("client_secret", "auth0ManagementAppClientSecret");
    requestBody.put("audience", "https://dev-example.auth0.com/api/v2/");
    requestBody.put("grant_type", "client_credentials"); 

    HttpEntity<String> request = new HttpEntity<String>(requestBody.toString(), headers);

    RestTemplate restTemplate = new RestTemplate();
    HashMap<String, String> result = restTemplate
      .postForObject("https://dev-example.auth0.com/oauth/token", request, HashMap.class);

    return result.get("access_token");
}
```

这里，我们向`/oauth/token` Auth0 令牌 URL 发出了一个 REST 请求，以获取访问和刷新令牌。

此外，我们可以将这些客户端凭证存储在`application.properties`文件中，并使用`AuthConfig`类读取它。

### 8.5.`UserController`

之后，让我们用`users`方法创建`UserController`类:

```java
@Controller
public class UserController {
    @GetMapping(value="/users")
    @ResponseBody
    public ResponseEntity<String> users(HttpServletRequest request, HttpServletResponse response) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.set("Authorization", "Bearer " + getManagementApiToken());

        HttpEntity<String> entity = new HttpEntity<String>(headers);

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> result = restTemplate
          .exchange("https://dev-example.auth0.com/api/v2/users", HttpMethod.GET, entity, String.class);
        return result;
    }
}
```

`users`方法通过使用上一节中生成的访问令牌向`/api/v2/users` Auth0 API 发出 GET 请求来获取所有用户的列表。

因此，让我们访问 [`localhost:8080/users`](https://web.archive.org/web/20221126215056/http://localhost:8080/users) 来接收一个包含所有用户的 JSON 响应:

```java
[{
    "created_at": "2020-05-05T14:38:18.955Z",
    "email": "[[email protected]](/web/20221126215056/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "email_verified": true,
    "identities": [
        {
            "user_id": "5eb17a5a1cc1ac0c1487c37f78758",
            "provider": "auth0",
            "connection": "Username-Password-Authentication",
            "isSocial": false
        }
    ],
    "name": "[[email protected]](/web/20221126215056/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "nickname": "ansh",
    "logins_count": 64
    // ...
}]
```

### 8.6.创造用户

类似地，我们可以通过向`/api/v2/users` Auth0 API 发出 POST 请求来创建一个用户:

```java
@GetMapping(value = "/createUser")
@ResponseBody
public ResponseEntity<String> createUser(HttpServletResponse response) {
    JSONObject request = new JSONObject();
    request.put("email", "[[email protected]](/web/20221126215056/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    request.put("given_name", "Norman");
    request.put("family_name", "Lewis");
    request.put("connection", "Username-Password-Authentication");
    request.put("password", "Pa33w0rd");

    // ...
    ResponseEntity<String> result = restTemplate
      .postForEntity("https://dev-example.auth0.com/api/v2/users", request.toString(), String.class);
    return result;
}
```

然后，让我们访问 [`localhost:8080/createUser`](https://web.archive.org/web/20221126215056/http://localhost:8080/createUser) 并验证新用户的详细信息:

```java
{
    "created_at": "2020-05-10T12:30:15.343Z",
    "email": "[[email protected]](/web/20221126215056/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "email_verified": false,
    "family_name": "Lewis",
    "given_name": "Norman",
    "identities": [
        {
            "connection": "Username-Password-Authentication",
            "user_id": "5eb7f3d76b69bc0c120a8901576",
            "provider": "auth0",
            "isSocial": false
        }
    ],
    "name": "[[email protected]](/web/20221126215056/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "nickname": "norman.lewis",
    // ...
}
```

类似地，我们可以执行**各种操作，比如列出所有连接、创建一个连接、列出所有客户端，以及使用 Auth0 APIs 创建一个客户端**，这取决于我们的权限。

## 9.结论

在本教程中，我们探索了 Auth0 的 Spring 安全性。

首先，我们用基本配置设置 Auth0 帐户。然后，我们创建了一个 Spring Boot 应用程序，并为 Spring 安全集成配置了`application.properties`和 Auth0。

接下来，我们研究了为 Auth0 管理 API 创建 API 令牌。最后，我们研究了获取所有用户和创建用户之类的特性。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20221126215056/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-auth0)