# Spring 安全和 OpenID 连接(遗留)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-openid-connect-legacy>

**请注意，此内容已经过时，并且正在使用传统的 OAuth 堆栈。看看 [Spring Security 最新的 OAuth 支持](/web/20220701021819/https://www.baeldung.com/spring-security-openid-connect)。**

## 1。概述

在这个快速教程中，我们将重点介绍如何使用 Spring Security OAuth2 实现来设置 OpenID Connect。

[OpenID Connect](https://web.archive.org/web/20220701021819/https://openid.net/connect/) 是建立在 OAuth 2.0 协议之上的简单身份层。

更具体地说，我们将学习如何使用来自 [谷歌](https://web.archive.org/web/20220701021819/https://developers.google.com/identity/protocols/OpenIDConnect)的 [OpenID Connect 实现来认证用户。](https://web.archive.org/web/20220701021819/https://developers.google.com/identity/protocols/OpenIDConnect)

## 2。Maven 配置

首先，我们需要向我们的 Spring Boot 应用程序添加以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency>
```

## 3。Id 令牌

在我们深入研究实现细节之前，让我们快速了解一下 OpenID 是如何工作的，以及我们将如何与之交互。

此时，理解 OAuth2 当然很重要，因为 OpenID 是建立在 OAuth 之上的。

首先，为了使用身份功能，我们将使用一个名为`openid`的新 OAuth2 范围。**这将在我们的访问令牌中产生一个额外的字段——“T1”。**

`id_token`是一个 JWT (JSON Web Token ),包含用户的身份信息，由身份提供者(在我们的例子中是 Google)签名。

最后，`server(Authorization Code)`和`implicit`流都是最常用的获取`id_token`的方式，在我们的例子中，我们将使用**服务器流**。

## 3。OAuth2 客户端配置

接下来，让我们配置我们的 OAuth2 客户端，如下所示:

```java
@Configuration
@EnableOAuth2Client
public class GoogleOpenIdConnectConfig {
    @Value("${google.clientId}")
    private String clientId;

    @Value("${google.clientSecret}")
    private String clientSecret;

    @Value("${google.accessTokenUri}")
    private String accessTokenUri;

    @Value("${google.userAuthorizationUri}")
    private String userAuthorizationUri;

    @Value("${google.redirectUri}")
    private String redirectUri;

    @Bean
    public OAuth2ProtectedResourceDetails googleOpenId() {
        AuthorizationCodeResourceDetails details = new AuthorizationCodeResourceDetails();
        details.setClientId(clientId);
        details.setClientSecret(clientSecret);
        details.setAccessTokenUri(accessTokenUri);
        details.setUserAuthorizationUri(userAuthorizationUri);
        details.setScope(Arrays.asList("openid", "email"));
        details.setPreEstablishedRedirectUri(redirectUri);
        details.setUseCurrentUri(false);
        return details;
    }

    @Bean
    public OAuth2RestTemplate googleOpenIdTemplate(OAuth2ClientContext clientContext) {
        return new OAuth2RestTemplate(googleOpenId(), clientContext);
    }
}
```

这里是`application.properties`:

```java
google.clientId=<your app clientId>
google.clientSecret=<your app clientSecret>
google.accessTokenUri=https://www.googleapis.com/oauth2/v3/token
google.userAuthorizationUri=https://accounts.google.com/o/oauth2/auth
google.redirectUri=http://localhost:8081/google-login
```

请注意:

*   你首先需要从[谷歌开发者控制台](https://web.archive.org/web/20220701021819/https://console.developers.google.com/project/_/apiui/credential)获得你的谷歌网络应用的 OAuth 2.0 证书。
*   我们使用范围`openid`来获取`id_token`。
*   我们还使用了一个额外的作用域`email`来将用户电子邮件包含在`id_token`身份信息中。
*   重定向 URI `http://localhost:8081/google-login`与我们的谷歌网络应用中使用的是同一个。

## 4。自定义 OpenID 连接过滤器

现在，我们需要创建我们自己的自定义`OpenIdConnectFilter`来从`id_token`中提取身份验证，如下所示:

```java
public class OpenIdConnectFilter extends AbstractAuthenticationProcessingFilter {

    public OpenIdConnectFilter(String defaultFilterProcessesUrl) {
        super(defaultFilterProcessesUrl);
        setAuthenticationManager(new NoopAuthenticationManager());
    }
    @Override
    public Authentication attemptAuthentication(
      HttpServletRequest request, HttpServletResponse response) 
      throws AuthenticationException, IOException, ServletException {
        OAuth2AccessToken accessToken;
        try {
            accessToken = restTemplate.getAccessToken();
        } catch (OAuth2Exception e) {
            throw new BadCredentialsException("Could not obtain access token", e);
        }
        try {
            String idToken = accessToken.getAdditionalInformation().get("id_token").toString();
            String kid = JwtHelper.headers(idToken).get("kid");
            Jwt tokenDecoded = JwtHelper.decodeAndVerify(idToken, verifier(kid));
            Map<String, String> authInfo = new ObjectMapper()
              .readValue(tokenDecoded.getClaims(), Map.class);
            verifyClaims(authInfo);
            OpenIdConnectUserDetails user = new OpenIdConnectUserDetails(authInfo, accessToken);
            return new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
        } catch (InvalidTokenException e) {
            throw new BadCredentialsException("Could not obtain user details from token", e);
        }
    }
}
```

这里是我们简单的`OpenIdConnectUserDetails`:

```java
public class OpenIdConnectUserDetails implements UserDetails {
    private String userId;
    private String username;
    private OAuth2AccessToken token;

    public OpenIdConnectUserDetails(Map<String, String> userInfo, OAuth2AccessToken token) {
        this.userId = userInfo.get("sub");
        this.username = userInfo.get("email");
        this.token = token;
    }
}
```

请注意:

*   弹簧安全`JwtHelper`解码`id_token`。
*   `id_token`始终包含“`sub”`字段，这是用户的唯一标识符。
*   当我们在请求中添加了`email`范围时，`id_token`也将包含`email`字段。

### 4.1。验证 ID 令牌

在上面的例子中，我们使用了`JwtHelper`的`decodeAndVerify()`方法从`id_token,`中提取信息，并对其进行验证。

第一步是验证它是用 [Google Discovery](https://web.archive.org/web/20220701021819/https://developers.google.com/identity/protocols/OpenIDConnect#discovery) 文档中指定的证书之一签名的。

这些大约每天变化一次，所以我们将使用一个名为 [jwks-rsa](https://web.archive.org/web/20220701021819/https://search.maven.org/classic/#search%7Cga%7C1%7Cjwks) 的实用程序库来读取它们:

```java
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>jwks-rsa</artifactId>
    <version>0.3.0</version>
</dependency>
```

让我们将包含证书的 URL 添加到`application.properties`文件中:

```java
google.jwkUrl=https://www.googleapis.com/oauth2/v2/certs
```

现在我们可以读取这个属性并构建`RSAVerifier`对象:

```java
@Value("${google.jwkUrl}")
private String jwkUrl;    

private RsaVerifier verifier(String kid) throws Exception {
    JwkProvider provider = new UrlJwkProvider(new URL(jwkUrl));
    Jwk jwk = provider.get(kid);
    return new RsaVerifier((RSAPublicKey) jwk.getPublicKey());
}
```

最后，我们还将验证解码后的 id 令牌中的声明:

```java
public void verifyClaims(Map claims) {
    int exp = (int) claims.get("exp");
    Date expireDate = new Date(exp * 1000L);
    Date now = new Date();
    if (expireDate.before(now) || !claims.get("iss").equals(issuer) || 
      !claims.get("aud").equals(clientId)) {
        throw new RuntimeException("Invalid claims");
    }
}
```

`verifyClaims()`方法检查 id 令牌是由 Google 发布的，并且没有过期。

你可以在谷歌文档中找到更多相关信息。

## 5。安全配置

接下来，让我们讨论一下我们的安全配置:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private OAuth2RestTemplate restTemplate;

    @Bean
    public OpenIdConnectFilter openIdConnectFilter() {
        OpenIdConnectFilter filter = new OpenIdConnectFilter("/google-login");
        filter.setRestTemplate(restTemplate);
        return filter;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .addFilterAfter(new OAuth2ClientContextFilter(), 
          AbstractPreAuthenticatedProcessingFilter.class)
        .addFilterAfter(OpenIdConnectFilter(), 
          OAuth2ClientContextFilter.class)
        .httpBasic()
        .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/google-login"))
        .and()
        .authorizeRequests()
        .anyRequest().authenticated();
    }
}
```

请注意:

*   我们在`OAuth2ClientContextFilter`后添加了我们的自定义`OpenIdConnectFilter`
*   我们使用一个简单的安全配置将用户重定向到“`/google-login`”以获得 Google 的认证

## 6。用户控制器

接下来，这里有一个简单的控制器来测试我们的应用程序:

```java
@Controller
public class HomeController {
    @RequestMapping("/")
    @ResponseBody
    public String home() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        return "Welcome, " + username;
    }
}
```

示例响应(重定向至 Google 以批准应用授权后) :

```java
Welcome, [[email protected]](/web/20220701021819/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

## 7。OpenID 连接流程示例

最后，让我们来看一个 OpenID Connect 身份验证流程示例。

首先，我们将发送一个**认证请求**:

```java
https://accounts.google.com/o/oauth2/auth?
    client_id=sampleClientID
    response_type=code&
    scope=openid%20email&
    redirect_uri=http://localhost:8081/google-login&
    state=abc
```

响应(**在用户批准**之后)是重定向到:

```java
http://localhost:8081/google-login?state=abc&code;=xyz
```

接下来，我们将把`code`换成访问令牌和`id_token`:

```java
POST https://www.googleapis.com/oauth2/v3/token 
    code=xyz&
    client_id= sampleClientID&
    client_secret= sampleClientSecret&
    redirect_uri=http://localhost:8081/google-login&
    grant_type=authorization_code
```

下面是一个回答示例:

```java
{
    "access_token": "SampleAccessToken",
    "id_token": "SampleIdToken",
    "token_type": "bearer",
    "expires_in": 3600,
    "refresh_token": "SampleRefreshToken"
}
```

最后，下面是实际的`id_token` 的信息:

```java
{
    "iss":"accounts.google.com",
    "at_hash":"AccessTokenHash",
    "sub":"12345678",
    "email_verified":true,
    "email":"[[email protected]](/web/20220701021819/https://www.baeldung.com/cdn-cgi/l/email-protection)",
     ...
}
```

因此，您可以立即看到令牌中的用户信息对于向我们自己的应用程序提供身份信息是多么有用。

## 8。结论

在这个快速介绍教程中，我们学习了如何使用 Google 的 OpenID Connect 实现来认证用户。

和往常一样，你可以在 GitHub 上找到源代码[。](https://web.archive.org/web/20220701021819/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-legacy-oidc)