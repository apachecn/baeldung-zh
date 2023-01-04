# 将 JWT 与 Spring Security OAuth 一起使用(传统堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-jwt-legacy>

## 1。概述

在本教程中，我们将讨论如何让我们的 Spring Security OAuth2 实现利用 JSON Web 令牌。

我们还将继续在 OAuth 系列的上一篇文章的基础上进行构建。

在我们开始之前，有一点很重要。请记住，**Spring 安全核心团队正在实现一个新的 OAuth2 堆栈**——有些方面已经完成，有些还在进行中。

关于本文使用新的 Spring Security 5 堆栈的版本，请看我们的文章[使用 JWT 和 Spring Security OAuth](/web/20220629003903/https://www.baeldung.com/spring-security-oauth-jwt) 。

好吧，让我们开始吧。

## 2。Maven 配置

首先，我们需要将`spring-security-jwt`依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
</dependency>
```

注意，我们需要向授权服务器和资源服务器添加`spring-security-jwt`依赖关系。

## 3。授权服务器

接下来，我们将配置我们的授权服务器来使用`JwtTokenStore`–如下所示:

```
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) 
      throws Exception {
        endpoints.tokenStore(tokenStore())
                 .accessTokenConverter(accessTokenConverter())
                 .authenticationManager(authenticationManager);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123");
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        return defaultTokenServices;
    }
}
```

注意，我们在`JwtAccessTokenConverter`中使用了一个**对称密钥**来签署我们的令牌——这意味着我们也需要为资源服务器使用完全相同的密钥。

## 4。资源服务器

现在，让我们来看看我们的资源服务器配置——它与授权服务器的配置非常相似:

```
@Configuration
@EnableResourceServer
public class OAuth2ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(ResourceServerSecurityConfigurer config) {
        config.tokenServices(tokenServices());
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123");
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        return defaultTokenServices;
    }
}
```

请记住，我们将这两个服务器定义为完全分离和可独立部署的。这就是为什么我们需要在新的配置中再次声明一些相同的 beans。

## 5。令牌中的自定义声明

现在让我们建立一些基础设施，以便能够在访问令牌中添加一些**自定义声明。框架提供的标准声明都很好，但是大多数时候我们需要令牌中的一些额外信息，以便在客户端使用。**

我们将定义一个`TokenEnhancer`来用这些附加声明定制我们的访问令牌。

在下面的例子中，我们将向我们的访问令牌添加一个额外的字段“`organization`”——使用这个`CustomTokenEnhancer`:

```
public class CustomTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(
      OAuth2AccessToken accessToken, 
      OAuth2Authentication authentication) {
        Map<String, Object> additionalInfo = new HashMap<>();
        additionalInfo.put(
          "organization", authentication.getName() + randomAlphabetic(4));
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(
          additionalInfo);
        return accessToken;
    }
}
```

然后，我们将它连接到我们的**授权服务器**配置中，如下所示:

```
@Override
public void configure(
  AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
    tokenEnhancerChain.setTokenEnhancers(
      Arrays.asList(tokenEnhancer(), accessTokenConverter()));

    endpoints.tokenStore(tokenStore())
             .tokenEnhancer(tokenEnhancerChain)
             .authenticationManager(authenticationManager);
}

@Bean
public TokenEnhancer tokenEnhancer() {
    return new CustomTokenEnhancer();
}
```

随着这一新配置的启动和运行，令牌令牌有效负载将会是这样的:

```
{
    "user_name": "john",
    "scope": [
        "foo",
        "read",
        "write"
    ],
    "organization": "johnIiCh",
    "exp": 1458126622,
    "authorities": [
        "ROLE_USER"
    ],
    "jti": "e0ad1ef3-a8a5-4eef-998d-00b26bc2c53f",
    "client_id": "fooClientIdPassword"
}
```

### 5.1。使用 JS 客户端中的访问令牌

最后，我们希望利用 AngualrJS 客户端应用程序中的令牌信息。为此，我们将使用 [angular-jwt](https://web.archive.org/web/20220629003903/https://github.com/auth0/angular-jwt) 库。

所以我们要做的是利用`index.html`中的“`organization`”声明:

```
<p class="navbar-text navbar-right">{{organization}}</p>

<script type="text/javascript" 
  src="https://cdn.rawgit.com/auth0/angular-jwt/master/dist/angular-jwt.js">
</script>

<script>
var app = 
  angular.module('myApp', ["ngResource","ngRoute", "ngCookies", "angular-jwt"]);

app.controller('mainCtrl', function($scope, $cookies, jwtHelper,...) {
    $scope.organiztion = "";

    function getOrganization(){
    	var token = $cookies.get("access_token");
    	var payload = jwtHelper.decodeToken(token);
    	$scope.organization = payload.organization;
    }
    ...
});
```

## 6。访问资源服务器上的额外声明

但是，我们如何在资源服务器端访问这些信息呢？

我们在这里要做的是–从访问令牌中提取额外的声明:

```
public Map<String, Object> getExtraInfo(OAuth2Authentication auth) {
    OAuth2AuthenticationDetails details =
      (OAuth2AuthenticationDetails) auth.getDetails();
    OAuth2AccessToken accessToken = tokenStore
      .readAccessToken(details.getTokenValue());
    return accessToken.getAdditionalInformation();
}
```

在下一节中，我们将讨论如何通过使用自定义的`AccessTokenConverter`将额外的信息添加到我们的`Authentication`细节中

### 6.1。`AccessTokenConverter`风俗

让我们创建`CustomAccessTokenConverter`并用访问令牌声明设置身份验证细节:

```
@Component
public class CustomAccessTokenConverter extends DefaultAccessTokenConverter {

    @Override
    public OAuth2Authentication extractAuthentication(Map<String, ?> claims) {
        OAuth2Authentication authentication =
          super.extractAuthentication(claims);
        authentication.setDetails(claims);
        return authentication;
    }
}
```

注意:`DefaultAccessTokenConverter`用于将认证细节设置为空。

### 6.2。配置`JwtTokenStore`

接下来，我们将配置我们的`JwtTokenStore`来使用我们的`CustomAccessTokenConverter`:

```
@Configuration
@EnableResourceServer
public class OAuth2ResourceServerConfigJwt
 extends ResourceServerConfigurerAdapter {

    @Autowired
    private CustomAccessTokenConverter customAccessTokenConverter;

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setAccessTokenConverter(customAccessTokenConverter);
    }
    // ...
}
```

### 6.3。认证对象中可用的额外声明

既然授权服务器在令牌中添加了一些额外的声明，我们现在可以在资源服务器端直接在身份验证对象中访问:

```
public Map<String, Object> getExtraInfo(Authentication auth) {
    OAuth2AuthenticationDetails oauthDetails =
      (OAuth2AuthenticationDetails) auth.getDetails();
    return (Map<String, Object>) oauthDetails
      .getDecodedDetails();
}
```

### 6.4。认证细节测试

让我们确保我们的身份验证对象包含额外的信息:

```
@RunWith(SpringRunner.class)
@SpringBootTest(
  classes = ResourceServerApplication.class, 
  webEnvironment = WebEnvironment.RANDOM_PORT)
public class AuthenticationClaimsIntegrationTest {

    @Autowired
    private JwtTokenStore tokenStore;

    @Test
    public void whenTokenDoesNotContainIssuer_thenSuccess() {
        String tokenValue = obtainAccessToken("fooClientIdPassword", "john", "123");
        OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);
        Map<String, Object> details = (Map<String, Object>) auth.getDetails();

        assertTrue(details.containsKey("organization"));
    }

    private String obtainAccessToken(
      String clientId, String username, String password) {

        Map<String, String> params = new HashMap<>();
        params.put("grant_type", "password");
        params.put("client_id", clientId);
        params.put("username", username);
        params.put("password", password);
        Response response = RestAssured.given()
          .auth().preemptive().basic(clientId, "secret")
          .and().with().params(params).when()
          .post("http://localhost:8081/spring-security-oauth-server/oauth/token");
        return response.jsonPath().getString("access_token");
    }
}
```

注意:我们从授权服务器获得了带有额外声明的访问令牌，然后我们从中读取了包含细节对象中额外信息“organization”的`Authentication`对象。

## 7。非对称密钥对

在我们之前的配置中，我们使用对称密钥对令牌进行签名:

```
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("123");
    return converter;
}
```

我们还可以使用非对称密钥(公钥和私钥)来完成签名过程。

### 7.1。生成 JKS Java 密钥库文件

让我们首先使用命令行工具:生成密钥——更具体地说是一个`.jks`文件

```
keytool -genkeypair -alias mytest 
                    -keyalg RSA 
                    -keypass mypass 
                    -keystore mytest.jks 
                    -storepass mypass
```

该命令将生成一个名为`mytest.jks`的文件，其中包含我们的密钥——公钥和私钥。

还要确保`keypass`和`storepass`相同。

### 7.2。导出公钥

接下来，我们需要从生成的 JKS 中导出我们的公钥，我们可以使用下面的命令来这样做:

```
keytool -list -rfc --keystore mytest.jks | openssl x509 -inform pem -pubkey
```

示例响应如下所示:

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgIK2Wt4x2EtDl41C7vfp
OsMquZMyOyteO2RsVeMLF/hXIeYvicKr0SQzVkodHEBCMiGXQDz5prijTq3RHPy2
/5WJBCYq7yHgTLvspMy6sivXN7NdYE7I5pXo/KHk4nz+Fa6P3L8+L90E/3qwf6j3
DKWnAgJFRY8AbSYXt1d5ELiIG1/gEqzC0fZmNhhfrBtxwWXrlpUDT0Kfvf0QVmPR
xxCLXT+tEe1seWGEqeOLL5vXRLqmzZcBe1RZ9kQQm43+a9Qn5icSRnDfTAesQ3Cr
lAWJKl2kcWU1HwJqw+dZRSZ1X4kEXNMyzPdPBbGmU6MHdhpywI7SKZT7mX4BDnUK
eQIDAQAB
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE-----
MIIDCzCCAfOgAwIBAgIEGtZIUzANBgkqhkiG9w0BAQsFADA2MQswCQYDVQQGEwJ1
czELMAkGA1UECBMCY2ExCzAJBgNVBAcTAmxhMQ0wCwYDVQQDEwR0ZXN0MB4XDTE2
MDMxNTA4MTAzMFoXDTE2MDYxMzA4MTAzMFowNjELMAkGA1UEBhMCdXMxCzAJBgNV
BAgTAmNhMQswCQYDVQQHEwJsYTENMAsGA1UEAxMEdGVzdDCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAICCtlreMdhLQ5eNQu736TrDKrmTMjsrXjtkbFXj
Cxf4VyHmL4nCq9EkM1ZKHRxAQjIhl0A8+aa4o06t0Rz8tv+ViQQmKu8h4Ey77KTM
urIr1zezXWBOyOaV6Pyh5OJ8/hWuj9y/Pi/dBP96sH+o9wylpwICRUWPAG0mF7dX
eRC4iBtf4BKswtH2ZjYYX6wbccFl65aVA09Cn739EFZj0ccQi10/rRHtbHlhhKnj
iy+b10S6ps2XAXtUWfZEEJuN/mvUJ+YnEkZw30wHrENwq5QFiSpdpHFlNR8CasPn
WUUmdV+JBFzTMsz3TwWxplOjB3YacsCO0imU+5l+AQ51CnkCAwEAAaMhMB8wHQYD
VR0OBBYEFOGefUBGquEX9Ujak34PyRskHk+WMA0GCSqGSIb3DQEBCwUAA4IBAQB3
1eLfNeq45yO1cXNl0C1IQLknP2WXg89AHEbKkUOA1ZKTOizNYJIHW5MYJU/zScu0
yBobhTDe5hDTsATMa9sN5CPOaLJwzpWV/ZC6WyhAWTfljzZC6d2rL3QYrSIRxmsp
/J1Vq9WkesQdShnEGy7GgRgJn4A8CKecHSzqyzXulQ7Zah6GoEUD+vjb+BheP4aN
hiYY1OuXD+HsdKeQqS+7eM5U7WW6dz2Q8mtFJ5qAxjY75T0pPrHwZMlJUhUZ+Q2V
FfweJEaoNB9w9McPe1cAiE+oeejZ0jq0el3/dJsx3rlVqZN+lMhRJJeVHFyeb3XF
lLFCUGhA7hxn2xf3x1JW
-----END CERTIFICATE-----
```

我们只取我们的公钥，并将其复制到我们的**资源服务器** `src/main/resources/public.txt`:

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgIK2Wt4x2EtDl41C7vfp
OsMquZMyOyteO2RsVeMLF/hXIeYvicKr0SQzVkodHEBCMiGXQDz5prijTq3RHPy2
/5WJBCYq7yHgTLvspMy6sivXN7NdYE7I5pXo/KHk4nz+Fa6P3L8+L90E/3qwf6j3
DKWnAgJFRY8AbSYXt1d5ELiIG1/gEqzC0fZmNhhfrBtxwWXrlpUDT0Kfvf0QVmPR
xxCLXT+tEe1seWGEqeOLL5vXRLqmzZcBe1RZ9kQQm43+a9Qn5icSRnDfTAesQ3Cr
lAWJKl2kcWU1HwJqw+dZRSZ1X4kEXNMyzPdPBbGmU6MHdhpywI7SKZT7mX4BDnUK
eQIDAQAB
-----END PUBLIC KEY-----
```

或者，我们可以通过添加`-noout`参数只导出公钥:

```
keytool -list -rfc --keystore mytest.jks | openssl x509 -inform pem -pubkey -noout
```

### 7.3。Maven 配置

接下来，我们不希望 JKS 文件被 maven 过滤过程拾取——所以我们将确保在`pom.xml`中排除它:

```
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>*.jks</exclude>
            </excludes>
        </resource>
    </resources>
</build>
```

如果我们使用 Spring Boot，我们需要确保通过 Spring Boot Maven 插件将我们的 JKS 文件添加到应用程序类路径中。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 7.4。授权服务器

现在，我们将配置`JwtAccessTokenConverter`使用来自`mytest.jks`的密钥对，如下所示:

```
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    KeyStoreKeyFactory keyStoreKeyFactory = 
      new KeyStoreKeyFactory(new ClassPathResource("mytest.jks"), "mypass".toCharArray());
    converter.setKeyPair(keyStoreKeyFactory.getKeyPair("mytest"));
    return converter;
}
```

### 7.5。资源服务器

最后，我们需要配置我们的资源服务器来使用公钥——如下所示:

```
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    Resource resource = new ClassPathResource("public.txt");
    String publicKey = null;
    try {
        publicKey = IOUtils.toString(resource.getInputStream());
    } catch (final IOException e) {
        throw new RuntimeException(e);
    }
    converter.setVerifierKey(publicKey);
    return converter;
}
```

## 8。结论

在这篇简短的文章中，我们重点介绍了如何设置 Spring Security OAuth2 项目来使用 JSON Web 令牌。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220629003903/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。