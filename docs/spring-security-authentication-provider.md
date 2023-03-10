# Spring 安全认证提供者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-authentication-provider>

## 1。概述

在本教程中，我们将学习如何在 Spring Security、中设置一个**身份验证提供者，与使用简单的`UserDetailsService`的标准场景相比，它提供了额外的灵活性。**

## 2。认证提供者

Spring Security 为执行身份验证提供了多种选项。这些选项遵循一个简单的契约；**一个`Authentication`请求由一个`AuthenticationProvider,`** 处理，并返回一个具有完整凭证的完全认证的对象。

标准且最常见的实现是`DaoAuthenticationProvider,`，它从一个简单的只读用户 DAO`UserDetailsService`中检索用户详细信息。这个用户详细信息服务**只能够访问用户名**，以便检索完整的用户实体，这对于大多数场景来说已经足够了。

更多的定制场景仍然需要访问完整的`Authentication`请求才能执行认证过程。例如，当对一些外部的第三方服务(如[群](https://web.archive.org/web/20220902184906/https://www.atlassian.com/software/crowd "Crowd - Identity Management"))**进行认证时，来自认证请求的`username`和`password`都是必需的**。

对于这些更高级的场景，我们需要**定义一个定制的身份验证提供者**:

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) 
      throws AuthenticationException {

        String name = authentication.getName();
        String password = authentication.getCredentials().toString();

        if (shouldAuthenticateAgainstThirdPartySystem()) {

            // use the credentials
            // and authenticate against the third-party system
            return new UsernamePasswordAuthenticationToken(
              name, password, new ArrayList<>());
        } else {
            return null;
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

注意，在返回的`Authentication`对象上设置的授权是空的。这是因为权限当然是特定于应用程序的。

## 3。注册授权提供者

既然我们已经定义了身份验证提供者，我们需要使用可用的名称空间支持在 XML 安全配置中指定它:

```java
<http use-expressions="true">
    <intercept-url pattern="/**" access="isAuthenticated()"/>
    <http-basic/>
</http>

<authentication-manager>
    <authentication-provider
      ref="customAuthenticationProvider" />
</authentication-manager>
```

## 4。Java 配置

接下来，我们来看看相应的 Java 配置:

```java
@Configuration
@EnableWebSecurity
@ComponentScan("com.baeldung.security")
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomAuthenticationProvider authProvider;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(authProvider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
            .and().httpBasic();
    }
}
```

## 5。执行认证

无论后端有没有这个自定义身份验证提供程序，从客户端请求身份验证基本上都是一样的。

我们将使用一个简单的`curl`命令来发送一个经过验证的请求:

```java
curl --header "Accept:application/json" -i --user user1:user1Pass 
    http://localhost:8080/spring-security-custom/api/foo/1
```

出于本例的目的，我们用基本身份验证来保护 REST API。

我们从服务器获得预期的 200 OK:

```java
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=B8F0EFA81B78DE968088EBB9AFD85A60; Path=/spring-security-custom/; HttpOnly
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sun, 02 Jun 2013 17:50:40 GMT
```

## 6。结论

在本文中，我们探索了一个用于 Spring 安全性的定制身份验证提供者的例子。

本文的完整实现可以在 GitHub 项目中找到。