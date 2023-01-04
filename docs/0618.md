# 通过 Spring Security OAuth 使用 JWT

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-jwt>

## 1。概述

在本教程中，我们将讨论如何让我们的 Spring Security OAuth2 实现利用 JSON Web 令牌。

我们还将继续构建 OAuth 系列中的[Spring REST API+OAuth 2+Angular](/web/20221126233349/https://www.baeldung.com/rest-api-spring-oauth2-angular)文章。

## 延伸阅读:

## [在 OAuth 安全应用程序中注销](/web/20221126233349/https://www.baeldung.com/logout-spring-security-oauth)

A practical deep-dive into how to implement logout in a Spring Security OAuth2 application with JWT.[Read more](/web/20221126233349/https://www.baeldung.com/logout-spring-security-oauth) →

## [OAuth2 用刷新令牌记住我(使用 Spring Security OAuth 传统堆栈)](/web/20221126233349/https://www.baeldung.com/spring-security-oauth2-remember-me)

Learn how to implement remember-me functionality with an Angular frontend, for an application secured with Spring Security OAuth.[Read more](/web/20221126233349/https://www.baeldung.com/spring-security-oauth2-remember-me) →

## [用于 Spring REST API 的 OAuth2 处理 Angular 中的刷新令牌](/web/20221126233349/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular)

Have a look at how to refresh a token using the Spring Security 5 OAuth stack and leveraging a Zuul proxy.[Read more](/web/20221126233349/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular) →

## 2.OAuth2 授权服务器

以前，Spring Security OAuth 堆栈提供了将授权服务器设置为 Spring 应用程序的可能性。然后我们必须将其配置为使用`JwtTokenStore` ，这样我们就可以使用 JWT 令牌。

然而，OAuth 栈已经被 Spring 弃用，现在我们将使用 Keycloak 作为我们的授权服务器。

**所以这一次，我们将把我们的授权服务器设置为[一个 Spring Boot 应用](/web/20221126233349/https://www.baeldung.com/keycloak-embedded-in-a-spring-boot-application)** 中的嵌入式 Keycloak 服务器。默认情况下，它颁发 JWT 令牌，因此在这方面不需要任何其他配置。

## 3.资源服务器

现在让我们看看如何配置我们的资源服务器来使用 JWT。

我们将在一个`application.yml`文件中这样做:

```
server: 
  port: 8081
  servlet: 
    context-path: /resource-server

spring:
  jpa:
    defer-datasource-initialization: true
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8083/auth/realms/baeldung
          jwk-set-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/certs
```

jwt 包含了令牌内 的所有信息，因此资源服务器需要 验证令牌的签名 以确保数据没有被修改。**`jwk-set-uri`属性** **包含服务器可用于此目的的公钥****。**

 **`issuer-uri`属性指向基础授权服务器 URI，作为一种附加的安全措施，它也可以用来验证`iss`声明。

此外，如果没有设置`jwk-set-uri`属性，资源服务器将尝试使用`issuer-uri`从[授权服务器元数据端点](https://web.archive.org/web/20221126233349/https://tools.ietf.org/html/rfc8414#section-3)中确定这个键的位置。

值得注意的是，添加属性`issuer-uri`要求**在启动资源服务器应用程序**之前，我们应该运行授权服务器。

现在让我们看看如何使用 Java 配置来配置 JWT 支持:

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors()
            .and()
              .authorizeRequests()
                .antMatchers(HttpMethod.GET, "/user/info", "/api/foos/**")
                  .hasAuthority("SCOPE_read")
                .antMatchers(HttpMethod.POST, "/api/foos")
                  .hasAuthority("SCOPE_write")
                .anyRequest()
                  .authenticated()
            .and()
              .oauth2ResourceServer()
                .jwt();
    }
}
```

这里我们覆盖了默认的 Http 安全配置；我们需要明确指定**我们希望它作为一个资源服务器，并且我们将分别使用方法`oauth2ResourceServer()`和`jwt()`、**来使用 JWT 格式的访问令牌。

上面的 JWT 配置是默认的 Spring Boot 实例提供给我们的。我们很快就会看到，这也可以定制。

## 4.令牌中的自定义声明

现在让我们建立一些基础设施，以便能够在授权服务器返回的访问令牌中添加一些**自定义声明。框架提供的标准声明都很好，但是大多数时候我们需要令牌中的一些额外信息，以便在客户端使用。**

让我们以自定义声明`organization`为例，它将包含给定用户的组织名称。

### 4.1.授权服务器配置

为此，我们需要在我们的领域定义文件中添加一些配置，`baeldung-realm.json`:

*   给我们的用户`[[email protected]](/web/20221126233349/https://www.baeldung.com/cdn-cgi/l/email-protection)`添加一个属性`organization`:

    ```
    "attributes" : {
      "organization" : "baeldung"
    },
    ```

*   在`jwtClient`配置:

    ```
    "protocolMappers": [{
      "id": "06e5fc8f-3553-4c75-aef4-5a4d7bb6c0d1",
      "name": "organization",
      "protocol": "openid-connect",
      "protocolMapper": "oidc-usermodel-attribute-mapper",
      "consentRequired": false,
      "config": {
        "userinfo.token.claim": "true",
        "user.attribute": "organization",
        "id.token.claim": "true",
        "access.token.claim": "true",
        "claim.name": "organization",
        "jsonType.label": "String"
      }
    }],
    ```

    中添加一个名为*的`protocolMapper`组织*

对于独立的 Keycloak 设置，也可以使用管理控制台来完成。

重要的是要记住**上面的 JSON 配置是特定于 Keycloak 的，对于其他 OAuth 服务器**可能会有所不同。

随着这个新配置的启动和运行，我们将在`[[email protected]](/web/20221126233349/https://www.baeldung.com/cdn-cgi/l/email-protection)`的令牌有效负载中获得一个额外的属性`organization = baeldung`:

```
{
  jti: "989ce5b7-50b9-4cc6-bc71-8f04a639461e"
  exp: 1585242462
  nbf: 0
  iat: 1585242162
  iss: "http://localhost:8083/auth/realms/baeldung"
  sub: "a5461470-33eb-4b2d-82d4-b0484e96ad7f"
  typ: "Bearer"
  azp: "jwtClient"
  auth_time: 1585242162
  session_state: "384ca5cc-8342-429a-879c-c15329820006"
  acr: "1"
  scope: "profile write read"
  organization: "baeldung"
  preferred_username: "[[email protected]](/web/20221126233349/https://www.baeldung.com/cdn-cgi/l/email-protection)"
}
```

### 4.2.在 Angular 客户端中使用访问令牌

接下来，我们希望在我们的 Angular 客户端应用程序中使用令牌信息。为此，我们将使用 [angular2-jwt](https://web.archive.org/web/20221126233349/https://github.com/auth0/angular2-jwt) 库。

我们将在`AppService`中使用`organization`声明，并添加一个函数`getOrganization`:

```
getOrganization(){
  var token = Cookie.get("access_token");
  var payload = this.jwtHelper.decodeToken(token);
  this.organization = payload.organization; 
  return this.organization;
}
```

这个函数利用来自`angular2-jwt`库中的`JwtHelperService`来解码访问令牌并获得我们的自定义声明。现在我们需要做的就是把它显示在我们的`AppComponent`中:

```
@Component({
  selector: 'app-root',
  template: `<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand" href="/">Spring Security Oauth - Authorization Code</a>
    </div>
  </div>
  <div class="navbar-brand">
    <p>{{organization}}</p>
  </div>
</nav>
<router-outlet></router-outlet>`
})

export class AppComponent implements OnInit {
  public organization = "";
  constructor(private service: AppService) { }  

  ngOnInit() {  
    this.organization = this.service.getOrganization();
  }  
}
```

## 5.访问资源服务器中的额外声明

但是我们如何在资源服务器端访问这些信息呢？

### 5.1.访问身份验证服务器声明

这真的很简单，我们只需要从`org.springframework.security.oauth2.jwt.Jwt`的的`**AuthenticationPrincipal,** `中提取它，就像我们对`UserInfoController`中的任何其他属性所做的那样:

```
@GetMapping("/user/info")
public Map<String, Object> getUserInfo(@AuthenticationPrincipal Jwt principal) {
    Map<String, String> map = new Hashtable<String, String>();
    map.put("user_name", principal.getClaimAsString("preferred_username"));
    map.put("organization", principal.getClaimAsString("organization"));
    return Collections.unmodifiableMap(map);
} 
```

### 5.2.添加/删除/重命名声明的配置

现在，如果我们想在资源服务器端添加更多的声明，该怎么办呢？或者删除或重命名一些？

假设我们想要修改来自认证服务器的`organization`声明，以获得大写的值。但是，如果用户没有该声明，我们需要将其值设置为`unknown`。

为了实现这一点，我们必须**添加一个实现`Converter`接口并使用`MappedJwtClaimSetConverter` 转换声明**的类:

```
public class OrganizationSubClaimAdapter implements 
  Converter<Map<String, Object>, Map<String, Object>> {

    private final MappedJwtClaimSetConverter delegate = 
      MappedJwtClaimSetConverter.withDefaults(Collections.emptyMap());

    public Map<String, Object> convert(Map<String, Object> claims) {
        Map<String, Object> convertedClaims = this.delegate.convert(claims);
        String organization = convertedClaims.get("organization") != null ? 
          (String) convertedClaims.get("organization") : "unknown";

        convertedClaims.put("organization", organization.toUpperCase());

        return convertedClaims;
    }
}
```

然后，在我们的`SecurityConfig`类中，我们需要**添加我们自己的 *JwtDecoder* 实例**来覆盖 Spring Boot **提供的实例，并将我们的`OrganizationSubClaimAdapter`设置为它的声明转换器**:

```
@Bean
public JwtDecoder jwtDecoder(OAuth2ResourceServerProperties properties) {
    NimbusJwtDecoder jwtDecoder = NimbusJwtDecoder.withJwkSetUri(
      properties.getJwt().getJwkSetUri()).build();

    jwtDecoder.setClaimSetConverter(new OrganizationSubClaimAdapter());
    return jwtDecoder;
} 
```

现在，当我们为用户`[[email protected]](/web/20221126233349/https://www.baeldung.com/cdn-cgi/l/email-protection)`点击我们的`/user/info` API 时，我们将得到作为`UNKNOWN`的`organization`。

请注意，覆盖由 Spring Boot 配置的默认`JwtDecoder` bean 应该非常小心，以确保仍然包含所有必要的配置。

## 6.从 Java 密钥库中加载密钥

在我们之前的配置中，我们使用授权服务器的默认公钥来验证令牌的完整性。

我们还可以使用存储在 Java Keystore 文件中的密钥对和证书来完成签名过程。

### 6.1.生成 JKS Java 密钥库文件

让我们首先使用命令行工具:生成密钥，更具体地说是一个`.jks`文件

```
keytool -genkeypair -alias mytest 
                    -keyalg RSA 
                    -keypass mypass 
                    -keystore mytest.jks 
                    -storepass mypass
```

该命令将生成一个名为`mytest.jks`的文件，其中包含我们的密钥、公钥和私钥。

还要确保`keypass`和`storepass`相同。

### 6.2.导出公钥

接下来，我们需要从生成的 JKS 中导出我们的公钥。我们可以使用下面的命令来做到这一点:

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

### 6.3.Maven 配置

我们不希望 JKS 文件被 maven 过滤过程拾取，所以我们将确保在`pom.xml`中排除它:

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

如果我们使用 Spring Boot，我们需要确保我们的 JKS 文件通过 Spring Boot Maven 插件`addResources`被添加到应用程序类路径中:

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

### 6.4.授权服务器

现在，我们将配置 Keycloak 来使用来自`mytest.jks`的 Keypair，方法是将它添加到领域定义 JSON 文件的`KeyProvider`部分，如下所示:

```
{
  "id": "59412b8d-aad8-4ab8-84ec-e546900fc124",
  "name": "java-keystore",
  "providerId": "java-keystore",
  "subComponents": {},
  "config": {
    "keystorePassword": [ "mypass" ],
    "keyAlias": [ "mytest" ],
    "keyPassword": [ "mypass" ],
    "active": [ "true" ],
    "keystore": [
            "src/main/resources/mytest.jks"
          ],
    "priority": [ "101" ],
    "enabled": [ "true" ],
    "algorithm": [ "RS256" ]
  }
},
```

这里我们将`priority`设置为`101`，大于授权服务器的任何其他密钥对，并将`active`设置为`true`。这样做是为了确保我们的资源服务器将从我们之前指定的`jwk-set-uri`属性中挑选这个特殊的 Keypair。

同样，这个配置是特定于 Keycloak 的，对于其他 OAuth 服务器实现可能有所不同。

## 7.结论

在这篇简短的文章中，我们重点介绍了如何设置 Spring Security OAuth2 项目来使用 JSON Web 令牌。

这篇文章的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221126233349/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-jwt "The Full Registration/Authentication Example Project on Github ")**