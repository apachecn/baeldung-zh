# 春季安全-来自 JWT 的地图权威

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-map-authorities-jwt>

## 1.介绍

在本教程中，我们将展示如何定制从 [JWT](/web/20220815050014/https://www.baeldung.com/java-json-web-tokens-jjwt) (JSON Web Token)声明到 Spring Security 的`Authorities`声明的映射。

## 2.背景

当一个正确配置的基于 Spring 安全的应用程序收到一个请求时，它会经历一系列的步骤，本质上，这些步骤旨在实现两个目标:

*   对请求进行身份验证，这样应用程序就可以知道是谁在访问它
*   决定经过身份验证的请求是否可以执行相关联的操作

对于使用 [JWT 作为其主要安全机制](/web/20220815050014/https://www.baeldung.com/spring-security-oauth-jwt)的应用，授权方面包括:

*   从 JWT 有效载荷中提取索赔值，通常是`scope`或`scp`索赔
*   将这些声明映射到一组`GrantedAuthority`对象中

**一旦安全引擎设置了这些权限，它就可以评估是否有任何访问限制适用于当前请求，并决定是否可以继续进行**。

## 3.缺省映象

开箱即用，Spring 使用直接的策略将声明转换成`GrantedAuthority`实例。首先，它提取出`scope`或`scp`声明，并将其拆分成一个字符串列表。接下来，对于每个字符串，它使用前缀`SCOPE_`后跟范围值创建一个新的`SimpleGrantedAuthority`。

为了说明这个策略，让我们创建一个简单的端点，它允许我们检查应用程序可用的`Authentication`实例的一些关键属性:

```
@RestController
@RequestMapping("/user")
public class UserRestController {

    @GetMapping("/authorities")
    public Map<String,Object> getPrincipalInfo(JwtAuthenticationToken principal) {

        Collection<String> authorities = principal.getAuthorities()
          .stream()
          .map(GrantedAuthority::getAuthority)
          .collect(Collectors.toList());

        Map<String,Object> info = new HashMap<>();
        info.put("name", principal.getName());
        info.put("authorities", authorities);
        info.put("tokenAttributes", principal.getTokenAttributes());

        return info;
    }
} 
```

这里，我们使用一个`JwtAuthenticationToken`参数，因为我们知道，当使用基于 JWT 的认证时，这将是由 Spring Security 创建的实际的`Authentication`实现。我们从它的`name`属性、可用的`GrantedAuthority`实例和 JWT 的原始属性中提取结果。

现在，让我们假设我们调用这个端点传递和编码并签名的 JWT，它包含这个有效负载:

```
{
  "aud": "api://f84f66ca-591f-4504-960a-3abc21006b45",
  "iss": "https://sts.windows.net/2e9fde3a-38ec-44f9-8bcd-c184dc1e8033/",
  "iat": 1648512013,
  "nbf": 1648512013,
  "exp": 1648516868,
  "email": "[[email protected]](/web/20220815050014/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "family_name": "Sevestre",
  "given_name": "Philippe",
  "name": "Philippe Sevestre",
  "scp": "profile.read",
  "sub": "eXWysuqIJmK1yDywH3gArS98PVO1SV67BLt-dvmQ-pM",
  ... more claims omitted
}
```

响应应该是一个 JSON 对象，具有三个属性:

```
{
  "tokenAttributes": {
     // ... token claims omitted
  },
  "name": "0047af40-473a-4dd3-bc46-07c3fe2b69a5",
  "authorities": [
    "SCOPE_profile",
    "SCOPE_email",
    "SCOPE_openid"
  ]
}
```

我们可以使用这些作用域来限制对应用程序某些部分的访问，方法是创建一个`SecurityFilterChain`:

```
@Bean
SecurityFilterChain customJwtSecurityChain(HttpSecurity http) throws Exception {
    return http.authorizeRequests(auth -> {
      auth.antMatchers("/user/**")
        .hasAuthority("SCOPE_profile");
    })
    .build();
}
```

注意，我们有意避免使用`WebSecurityConfigureAdapter`。**正如[所描述的](https://web.archive.org/web/20220815050014/https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)，这个类将在 Spring Security 版本中被弃用，所以最好尽快开始迁移到新的方法**。

或者，我们可以使用方法级注释和 SpEL 表达式来达到相同的结果:

```
@GetMapping("/authorities")
@PreAuthorize("hasAuthority('SCOPE_profile.read')")
public Map<String,Object> getPrincipalInfo(JwtAuthenticationToken principal) {
    // ... same code as before
}
```

最后，对于更复杂的场景，我们也可以直接访问当前的`JwtAuthenticationToken`，从这里我们可以直接访问所有的`GrantedAuthorities`

## 4.自定义`SCOPE_`前缀

作为如何更改 Spring Security 的默认声明映射行为的第一个例子，让我们看看如何将前缀`SCOPE_`更改为其他内容。如文档中所述，此任务涉及两个类:

*   `JwtAuthenticationConverter`:将原始 JWT 转换成`AbstractAuthenticationToken`
*   `JwtGrantedAuthoritiesConverter`:从原始 JWT 中提取一组`GrantedAuthority`实例。

在内部，`JwtAuthenticationConverter`使用`JwtGrantedAuthoritiesConverter`用`GrantedAuthority`对象和其他属性填充`JwtAuthenticationToken`。

**改变这个前缀最简单的方法是提供我们自己的`JwtAuthenticationConverter` bean** ，用`JwtGrantedAuthoritiesConverter`配置成我们自己选择的一个:

```
@Configuration
@EnableConfigurationProperties(JwtMappingProperties.class)
@EnableMethodSecurity
public class SecurityConfig {
    // ... fields and constructor omitted
    @Bean
    public Converter<Jwt, Collection<GrantedAuthority>> jwtGrantedAuthoritiesConverter() {
        JwtGrantedAuthoritiesConverter converter = new JwtGrantedAuthoritiesConverter();
        if (StringUtils.hasText(mappingProps.getAuthoritiesPrefix())) {
            converter.setAuthorityPrefix(mappingProps.getAuthoritiesPrefix().trim());
        }
        return converter;
    }

    @Bean
    public JwtAuthenticationConverter customJwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(jwtGrantedAuthoritiesConverter();
        return converter;
    } 
```

在这里，`JwtMappingProperties` 只是一个`@ConfigurationProperties`类，我们将使用它来具体化映射属性。虽然在这个代码片段中没有显示，但是我们将使用构造函数注入来初始化`mappingProps`字段，该字段包含一个从任何已配置的`PropertySource`中填充的实例，因此给了我们足够的灵活性来在部署时更改它的值。

这个`@Configuration` 类有两个`@Bean`方法:`jwtGrantedAuthoritiesConverter()`创建所需的`Converter `，后者创建`GrantedAuthority`集合。在这种情况下，我们使用配置属性中设置了前缀的库存`JwtGrantedAuthoritiesConverter`。

接下来，我们有了 `customJwtAuthenticationConverter()`，在这里我们构造了配置为使用我们的定制转换器的`JwtAuthenticationConverter`。从那里，Spring Security 将把它作为其标准自动配置过程的一部分，并替换默认配置。

现在，一旦我们将`baeldung.jwt.mapping.authorities-prefix`属性设置为某个值，例如`MY_SCOPE`，并调用`/user/authorities,`，我们将看到定制的权限:

```
{
  "tokenAttributes": {
    // ... token claims omitted 
  },
  "name": "0047af40-473a-4dd3-bc46-07c3fe2b69a5",
  "authorities": [
    "MY_SCOPE_profile",
    "MY_SCOPE_email",
    "MY_SCOPE_openid"
  ]
}
```

## 5.在安全结构中使用自定义前缀

**需要注意的是，通过更改授权前缀，我们将影响任何依赖于其名称的授权规则。**例如，如果我们将前缀改为`MY_PREFIX_`，任何采用默认前缀的`@PreAuthorize`表达式都将不再工作。这同样适用于基于`HttpSecurity`的授权结构。

然而，解决这个问题很简单。首先，让我们向我们的`@Configuration`类添加一个`@Bean`方法，该方法返回已配置的前缀。因为这个配置是可选的，所以我们必须确保在没有人给它的情况下返回默认值:

```
@Bean
public String jwtGrantedAuthoritiesPrefix() {
  return mappingProps.getAuthoritiesPrefix() != null ?
    mappingProps.getAuthoritiesPrefix() : 
      "SCOPE_";
} 
```

现在，我们可以在 SpEL 表达式中使用 [`@<bean-name>`语法来引用这个 bean。这就是我们使用前缀 bean 和`@PreAuthorize`的方式:](https://web.archive.org/web/20220815050014/https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#el-access-web-beans)

```
@GetMapping("/authorities")
@PreAuthorize("hasAuthority(@jwtGrantedAuthoritiesPrefix + 'profile.read')")
public Map<String,Object> getPrincipalInfo(JwtAuthenticationToken principal) {
    // ... method implementation omitted
}
```

在定义`SecurityFilterChain`时，我们也可以使用类似的方法:

```
@Bean
SecurityFilterChain customJwtSecurityChain(HttpSecurity http) throws Exception {
    return http.authorizeRequests(auth -> {
        auth.antMatchers("/user/**")
          .hasAuthority(mappingProps.getAuthoritiesPrefix() + "profile");
      })
      // ... other customizations omitted
      .build();
} 
```

## 6.定制`Principal`的名称

有时，标准的`sub`声明到`Authentication’`的`name`属性的 Spring 映射带有一个不是很有用的值。Keycloak 生成的 jwt 就是一个很好的例子:

```
{
  // ... other claims omitted
  "sub": "0047af40-473a-4dd3-bc46-07c3fe2b69a5",
  "scope": "openid profile email",
  "email_verified": true,
  "name": "User Primo",
  "preferred_username": "user1",
  "given_name": "User",
  "family_name": "Primo"
}
```

在这种情况下，`sub`带有一个内部标识符，但是我们可以看到,`preferred_username`声明有一个更友好的值。**我们可以通过将`JwtAuthenticationConverter`的`principalClaimName`属性设置为所需的声明名称**来轻松修改其行为:

```
@Bean
public JwtAuthenticationConverter customJwtAuthenticationConverter() {

    JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
    converter.setJwtGrantedAuthoritiesConverter(jwtGrantedAuthoritiesConverter());

    if (StringUtils.hasText(mappingProps.getPrincipalClaimName())) {
        converter.setPrincipalClaimName(mappingProps.getPrincipalClaimName());
    }
    return converter;
} 
```

现在，如果我们将`baeldung.jwt.mapping.authorities-prefix`属性设置为“preferred_username”，那么`/user/authorities`结果将会相应地改变:

```
{
  "tokenAttributes": {
    // ... token claims omitted 
  },
  "name": "user1",
  "authorities": [
    "MY_SCOPE_profile",
    "MY_SCOPE_email",
    "MY_SCOPE_openid"
  ]
}
```

## 7.作用域名映射

有时，我们可能需要将 JWT 中收到的作用域名称映射到内部名称。例如，在这种情况下，同一个应用程序需要使用由不同授权服务器生成的令牌，这取决于其部署的环境。

我们可能会尝试扩展`JwtGrantedAuthoritiesConverter,`，但是因为这是一个最终类，我们不能使用这种方法。相反，我们必须编写自己的转换器类，并将其注入到`JwtAuthorizationConverter`中。这个增强的映射器`MappingJwtGrantedAuthoritiesConverter`，实现了`Converter<Jwt, Collection<GrantedAuthority>>`，看起来很像原来的那个:

```
public class MappingJwtGrantedAuthoritiesConverter implements Converter<Jwt, Collection<GrantedAuthority>> {
    private static Collection<String> WELL_KNOWN_AUTHORITIES_CLAIM_NAMES = Arrays.asList("scope", "scp");
    private Map<String,String> scopes;
    private String authoritiesClaimName = null;
    private String authorityPrefix = "SCOPE_";

    // ... constructor and setters omitted

    @Override
    public Collection<GrantedAuthority> convert(Jwt jwt) {

        Collection<String> tokenScopes = parseScopesClaim(jwt);
        if (tokenScopes.isEmpty()) {
            return Collections.emptyList();
        }

        return tokenScopes.stream()
          .map(s -> scopes.getOrDefault(s, s))
          .map(s -> this.authorityPrefix + s)
          .map(SimpleGrantedAuthority::new)
          .collect(Collectors.toCollection(HashSet::new));
    }

    protected Collection<String> parseScopesClaim(Jwt jwt) {
       // ... parse logic omitted 
    }
}
```

**这里，这个类的关键方面是映射步骤，这里我们使用提供的`scopes`映射将原始范围转换成映射的**。此外，任何没有可用映射的传入范围都将被保留。

最后，我们在`@Configuration`的`jwtGrantedAuthoritiesConverter()`方法中使用了这个增强的转换器:

```
@Bean
public Converter<Jwt, Collection<GrantedAuthority>> jwtGrantedAuthoritiesConverter() {
    MappingJwtGrantedAuthoritiesConverter converter = new MappingJwtGrantedAuthoritiesConverter(mappingProps.getScopes());

    if (StringUtils.hasText(mappingProps.getAuthoritiesPrefix())) {
        converter.setAuthorityPrefix(mappingProps.getAuthoritiesPrefix());
    }
    if (StringUtils.hasText(mappingProps.getAuthoritiesClaimName())) {
        converter.setAuthoritiesClaimName(mappingProps.getAuthoritiesClaimName());
    }
    return converter;
} 
```

## 8.使用自定义`JwtAuthenticationConverter`

在这个场景中，我们将完全控制`JwtAuthenticationToken`生成过程。我们可以使用这种方法返回这个类的扩展版本，其中包含从数据库中恢复的额外数据。

有两种方法可以替代标准`JwtAuthenticationConverter`。第一种，我们在前面的章节中已经使用过，是创建一个返回我们的自定义转换器的`@Bean`方法。然而，这意味着我们的定制版本必须扩展 Spring 的`JwtAuthenticationConverter`,这样自动配置过程才能选择它。

第二个选择是使用基于`HttpSecurity`的 DSL 方法，在这里我们可以提供定制的转换器。我们将使用`oauth2ResourceServer`定制器来完成这项工作，它允许我们插入任何实现更通用接口`Converter<Jwt, AbstractAuthorizationToken>`的转换器:

```
@Bean
SecurityFilterChain customJwtSecurityChain(HttpSecurity http) throws Exception {
    return http.oauth2ResourceServer(oauth2 -> {
        oauth2.jwt()
          .jwtAuthenticationConverter(customJwtAuthenticationConverter());
      })
      .build();
} 
```

我们的`CustomJwtAuthenticationConverter`使用一个`AccountService`(在线可用)基于用户名声明值检索一个`Account`对象。然后使用它为帐户数据创建一个带有额外访问器方法的`CustomJwtAuthenticationToken`:

```
public class CustomJwtAuthenticationConverter implements Converter<Jwt, AbstractAuthenticationToken> {

    // ...private fields and construtor omitted
    @Override
    public AbstractAuthenticationToken convert(Jwt source) {

        Collection<GrantedAuthority> authorities = jwtGrantedAuthoritiesConverter.convert(source);
        String principalClaimValue = source.getClaimAsString(this.principalClaimName);
        Account acc = accountService.findAccountByPrincipal(principalClaimValue);
        return new AccountToken(source, authorities, principalClaimValue, acc);
    }
} 
```

现在，让我们修改我们的`/user/authorities`处理程序来使用我们增强的`Authentication`:

```
@GetMapping("/authorities")
public Map<String,Object> getPrincipalInfo(JwtAuthenticationToken principal) {

    // ... create result map as before (omitted)
    if (principal instanceof AccountToken) {
        info.put( "account", ((AccountToken)principal).getAccount());
    }
    return info;
} 
```

**采用这种方法的一个优点是，我们现在可以在应用程序的其他部分轻松使用我们的增强认证对象**。例如，我们可以直接从内置变量`authentication`访问 [SpEL](/web/20220815050014/https://www.baeldung.com/spring-expression-language) 表达式中的账户信息:

```
@GetMapping("/account/{accountNumber}")
@PreAuthorize("authentication.account.accountNumber == #accountNumber")
public Account getAccountById(@PathVariable("accountNumber") String accountNumber, AccountToken authentication) {
    return authentication.getAccount();
} 
```

这里，`@PreAuthorize`表达式强制在 path 变量中传递的`accountNumber`属于用户。这种方法在与 Spring 数据 JPA 结合使用时特别有用，如[官方文档](https://web.archive.org/web/20220815050014/https://docs.spring.io/spring-security/reference/servlet/integrations/data.html)中所述。

## 9.测试技巧

到目前为止给出的例子都假设我们有一个功能正常的身份提供者(IdP ),它颁发基于 JWT 的访问令牌。一个好的选择是使用我们已经在这里介绍过的嵌入式 Keycloak 服务器。在我们的[键盘锁](/web/20220815050014/https://www.baeldung.com/spring-boot-keycloak)快速使用指南中也有额外的配置说明。

请注意，这些说明涵盖了如何注册 OAuth `client.`进行实时测试，Postman 是一个支持授权代码流的好工具。**这里重要的细节是如何正确配置`Valid Redirect URI`参数**。由于 Postman 是一个桌面应用程序，它使用一个位于`https://oauth.pstmn.io/v1/callback`的助手站点来捕获授权代码。因此，我们必须确保我们在测试期间有互联网连接。如果这是不可能的，我们可以使用不太安全的密码授权流来代替。

**不管所选的 IdP 和客户机选择如何，我们都必须配置我们的资源服务器，以便它能够正确地验证接收到的 jwt**。对于标准的 OIDC 提供者，这意味着为`spring.security.oauth2.resourceserver.jwt.issuer-uri`属性提供一个合适的值。然后，Spring 将使用那里可用的`.well-known/openid-configuration`文档获取所有配置细节。

在我们的例子中，Keycloak 领域的发行者 URI 是`http://localhost:8083/auth/realms/baeldung.` ，我们可以让浏览器在`http://localhost:8083/auth/realms/baeldung/.well-known/openid-configuration`检索整个文档。

## 10.结论

在本文中，我们展示了自定义 JWT Spring Security map authorities 声明方式的不同方法。像往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220815050014/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-oidc)