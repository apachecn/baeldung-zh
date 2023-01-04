# 使用 Spring Security OAuth 提取主体和授权

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-principal-authorities-extractor>

## 1.概观

在本教程中，我们将说明如何使用 Spring Boot 和 Spring Security OAuth 创建一个将用户认证委托给第三方以及自定义授权服务器的应用程序。

同样，**我们将演示如何使用 Spring 的 [`PrincipalExtractor`](https://web.archive.org/web/20220628160531/https://docs.spring.io/spring-security-oauth2-boot/docs/current-SNAPSHOT/api/org/springframework/boot/autoconfigure/security/oauth2/resource/PrincipalExtractor.html) 和 [`AuthoritiesExtractor`](https://web.archive.org/web/20220628160531/https://docs.spring.io/spring-security-oauth2-boot/docs/current/api/org/springframework/boot/autoconfigure/security/oauth2/resource/AuthoritiesExtractor.html) 接口提取`Principal`和`Authorities`。**

关于 Spring Security OAuth2 的介绍，请参考这些文章。

## 2.Maven 依赖性

首先，我们需要将 [`spring-security-oauth2-autoconfigure`](https://web.archive.org/web/20220628160531/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.security.oauth.boot%22%20AND%20a%3A%22spring-security-oauth2-autoconfigure%22) 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

## 3.使用 Github 的 OAuth 认证

接下来，让我们创建应用程序的安全配置:

```java
@Configuration
@EnableOAuth2Sso
public class SecurityConfig 
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) 
      throws Exception {

        http.antMatcher("/**")
          .authorizeRequests()
          .antMatchers("/login**")
          .permitAll()
          .anyRequest()
          .authenticated()
          .and()
          .formLogin().disable();
    }
}
```

简而言之，我们说任何人都可以访问`/login`端点，所有其他端点都需要用户认证。

我们还用`@EnableOAuthSso`注释了我们的配置类，它将我们的应用程序转换成 OAuth 客户端，并为它的行为创建必要的组件。

虽然默认情况下 Spring 为我们创建了大部分组件，但是我们仍然需要配置一些属性:

```java
security.oauth2.client.client-id=89a7c4facbb3434d599d
security.oauth2.client.client-secret=9b3b08e4a340bd20e866787e4645b54f73d74b6a
security.oauth2.client.access-token-uri=https://github.com/login/oauth/access_token
security.oauth2.client.user-authorization-uri=https://github.com/login/oauth/authorize
security.oauth2.client.scope=read:user,user:email
security.oauth2.resource.user-info-uri=https://api.github.com/user
```

我们不再处理用户帐户管理，而是将它委托给第三方——在本例中是 Github——从而使我们能够专注于应用程序的逻辑。

## 4.提取主体和授权

当充当 OAuth 客户端并通过第三方认证用户时，我们需要考虑三个步骤:

1.  用户验证–用户通过第三方进行验证
2.  用户授权——在认证之后，用户允许我们的应用程序代表他们执行某些操作；这就是 [`scopes`](https://web.archive.org/web/20220628160531/https://www.oauth.com/oauth2-servers/scope/defining-scopes/) 出现的地方
3.  获取用户数据——使用我们获得的 OAuth 令牌来检索用户数据

一旦我们检索到用户的数据， **Spring 就能够自动创建用户的`Principal`和`Authorities`T3。**

虽然这可能是可以接受的，但更多的时候，我们会发现自己想要完全控制他们。

为此， **Spring 给了我们两个接口，我们可以用它们来覆盖它的默认行为**:

*   `PrincipalExtractor`–我们可以用来提供自定义逻辑以提取`Principal`的接口
*   `AuthoritiesExtractor`–类似于`PrincipalExtractor`，但用于定制`Authorities`提取

默认情况下， **Spring 提供了两个组件——[`FixedPrincipalExtractor`](https://web.archive.org/web/20220628160531/https://docs.spring.io/spring-boot/docs/2.0.0.M4/api/index.html?org/springframework/boot/autoconfigure/security/oauth2/resource/PrincipalExtractor.html)和`[FixedAuthoritiesExtractor](https://web.archive.org/web/20220628160531/https://docs.spring.io/spring-boot/docs/2.0.0.M4/api/org/springframework/boot/autoconfigure/security/oauth2/resource/FixedAuthoritiesExtractor.html)`** `–`，它们实现了这些接口，并有一个预定义的策略来为我们创建它们。

## 4.1.定制 Github 的认证

在我们的例子中，我们知道 Github 的用户数据看起来[像](https://web.archive.org/web/20220628160531/https://docs.github.com/en/rest/reference/users#get-the-authenticated-user)一样，我们可以使用什么来根据我们的需要定制它们。

因此，要覆盖 Spring 的默认组件，我们只需要创建两个`Beans`来实现这些接口。

对于我们的应用程序的`Principal`,我们将简单地使用用户的 Github 用户名:

```java
public class GithubPrincipalExtractor 
  implements PrincipalExtractor {

    @Override
    public Object extractPrincipal(Map<String, Object> map) {
        return map.get("login");
    }
}
```

根据我们用户的 Github 订阅——免费或不免费——我们会给他们一个`GITHUB_USER_SUBSCRIBED`或`GITHUB_USER_FREE`权限:

```java
public class GithubAuthoritiesExtractor 
  implements AuthoritiesExtractor {
    List<GrantedAuthority> GITHUB_FREE_AUTHORITIES
     = AuthorityUtils.commaSeparatedStringToAuthorityList(
     "GITHUB_USER,GITHUB_USER_FREE");
    List<GrantedAuthority> GITHUB_SUBSCRIBED_AUTHORITIES 
     = AuthorityUtils.commaSeparatedStringToAuthorityList(
     "GITHUB_USER,GITHUB_USER_SUBSCRIBED");

    @Override
    public List<GrantedAuthority> extractAuthorities
      (Map<String, Object> map) {

        if (Objects.nonNull(map.get("plan"))) {
            if (!((LinkedHashMap) map.get("plan"))
              .get("name")
              .equals("free")) {
                return GITHUB_SUBSCRIBED_AUTHORITIES;
            }
        }
        return GITHUB_FREE_AUTHORITIES;
    }
} 
```

然后，我们还需要使用这些类创建 beans:

```java
@Configuration
@EnableOAuth2Sso
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // ...

    @Bean
    public PrincipalExtractor githubPrincipalExtractor() {
        return new GithubPrincipalExtractor();
    }

    @Bean
    public AuthoritiesExtractor githubAuthoritiesExtractor() {
        return new GithubAuthoritiesExtractor();
    }
}
```

## 4.2.使用自定义授权服务器

我们还可以为我们的用户使用我们自己的授权服务器，而不是依赖第三方。

尽管我们决定使用授权服务器，但是我们需要定制的组件`Principal`和`Authorities`保持不变:一个 **`PrincipalExtractor`** 和一个 **`AuthoritiesExtractor`** 。

我们只需要**知道由`user-info-uri`端点**返回的数据，并在我们认为合适的时候使用它。

让我们更改我们的应用程序，使用本文中[描述的授权服务器来验证我们的用户:](/web/20220628160531/https://www.baeldung.com/sso-spring-security-oauth2)

```java
security.oauth2.client.client-id=SampleClientId
security.oauth2.client.client-secret=secret
security.oauth2.client.access-token-uri=http://localhost:8081/auth/oauth/token
security.oauth2.client.user-authorization-uri=http://localhost:8081/auth/oauth/authorize
security.oauth2.resource.user-info-uri=http://localhost:8081/auth/user/me
```

现在我们指向我们的授权服务器，我们需要创建两个提取器；在这种情况下，我们的`PrincipalExtractor`将使用`name`键从`Map`中提取`Principal`:

```java
public class BaeldungPrincipalExtractor 
  implements PrincipalExtractor {

    @Override
    public Object extractPrincipal(Map<String, Object> map) {
        return map.get("name");
    }
}
```

至于权限，我们的授权服务器已经将它们放入其`user-info-uri`的数据中。

因此，我们将提取并丰富它们:

```java
public class BaeldungAuthoritiesExtractor 
  implements AuthoritiesExtractor {

    @Override
    public List<GrantedAuthority> extractAuthorities
      (Map<String, Object> map) {
        return AuthorityUtils
          .commaSeparatedStringToAuthorityList(asAuthorities(map));
    }

    private String asAuthorities(Map<String, Object> map) {
        List<String> authorities = new ArrayList<>();
        authorities.add("BAELDUNG_USER");
        List<LinkedHashMap<String, String>> authz = 
          (List<LinkedHashMap<String, String>>) map.get("authorities");
        for (LinkedHashMap<String, String> entry : authz) {
            authorities.add(entry.get("authority"));
        }
        return String.join(",", authorities);
    }
}
```

然后我们将豆子添加到我们的`SecurityConfig`类中:

```java
@Configuration
@EnableOAuth2Sso
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // ...

    @Bean
    public PrincipalExtractor baeldungPrincipalExtractor() {
        return new BaeldungPrincipalExtractor();
    }

    @Bean
    public AuthoritiesExtractor baeldungAuthoritiesExtractor() {
        return new BaeldungAuthoritiesExtractor();
    }
}
```

## 5.结论

在本文中，我们实现了一个应用程序，它将用户身份验证委托给第三方以及一个定制的授权服务器，并演示了如何定制`Principal`和`Authorities`。

像往常一样，这个例子的实现可以在 Github 上找到[。](https://web.archive.org/web/20220628160531/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-5-security-oauth)

在本地运行时，您可以在`localhost:8082`运行和测试应用程序