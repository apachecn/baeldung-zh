# Spring Security 中的多个身份验证提供者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-multiple-auth-providers>

## 1。概述

在这篇简短的文章中，我们将重点关注在 Spring Security 中使用多种机制来认证用户。

我们将通过配置多个身份认证提供者来实现这一点。

## 2。认证提供商

一个 [`AuthenticationProvider`](https://web.archive.org/web/20221003085830/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AuthenticationProvider.html) 是一个抽象，用于从一个特定的存储库中获取用户信息(比如一个[数据库](/web/20221003085830/https://www.baeldung.com/spring-security-authentication-with-a-database)、 [LDAP](/web/20221003085830/https://www.baeldung.com/spring-security-ldap) 、[自定义第三方源](/web/20221003085830/https://www.baeldung.com/spring-security-authentication-provider)等)。).它使用获取的用户信息来验证提供的凭证。

简单地说，当定义了多个身份验证提供者时，将按照它们被声明的顺序查询这些提供者。

为了快速演示，我们将配置两个身份验证提供程序—一个自定义身份验证提供程序和一个内存中身份验证提供程序。

## 3。Maven 依赖关系

让我们首先将必要的 Spring 安全依赖项添加到我们的 web 应用程序中:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency> 
```

没有 Spring Boot:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

这些依赖项的最新版本可以在 [spring-security-web](https://web.archive.org/web/20221003085830/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-web%22) 、 [spring-security-core](https://web.archive.org/web/20221003085830/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-core%22) 和 [spring-security-config](https://web.archive.org/web/20221003085830/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-config%22) 找到。

## 4。自定义认证提供商

现在让我们通过实现`AuthneticationProvider` 接口`.` 来创建一个定制的身份验证提供者

我们将实现`authenticate`方法——尝试认证。输入`Authentication`对象包含用户提供的用户名和密码凭证。

如果认证成功，`authenticate`方法返回一个完全填充的`Authentication`对象。如果认证失败，它抛出一个`AuthenticationException`类型的异常:

```
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication auth) 
      throws AuthenticationException {
        String username = auth.getName();
        String password = auth.getCredentials()
            .toString();

        if ("externaluser".equals(username) && "pass".equals(password)) {
            return new UsernamePasswordAuthenticationToken
              (username, password, Collections.emptyList());
        } else {
            throw new 
              BadCredentialsException("External system authentication failed");
        }
    }

    @Override
    public boolean supports(Class<?> auth) {
        return auth.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

自然，对于我们这里的例子来说，这是一个简单的实现。

## 5。配置多个认证提供者

现在让我们将`CustomAuthenticationProvider`和内存中的身份验证提供者添加到我们的 Spring 安全配置中。

### 5.1。Java 配置

在我们的配置类中，现在让我们使用`AuthenticationManagerBuilder`来创建和添加身份验证提供者。

首先是`CustomAuthenticationProvider`，然后是使用`inMemoryAuthentication()`的内存认证提供者。

我们还确保对 URL 模式“`/api/**`”的访问需要经过身份验证:

```
@EnableWebSecurity
public class MultipleAuthProvidersSecurityConfig 
  extends WebSecurityConfigurerAdapter {
    @Autowired
    CustomAuthenticationProvider customAuthProvider;

    @Override
    public void configure(AuthenticationManagerBuilder auth) 
      throws Exception {

        auth.authenticationProvider(customAuthProvider);
        auth.inMemoryAuthentication()
            .withUser("memuser")
            .password(encoder().encode("pass"))
            .roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic()
            .and()
            .authorizeRequests()
            .antMatchers("/api/**")
            .authenticated();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 5.2。XML 配置

或者，如果我们想使用 XML 配置而不是 Java 配置:

```
<security:authentication-manager>
    <security:authentication-provider>
        <security:user-service>
            <security:user name="memuser" password="pass" 
              authorities="ROLE_USER" />
        </security:user-service>
    </security:authentication-provider>
    <security:authentication-provider
      ref="customAuthenticationProvider" />
</security:authentication-manager>

<security:http>
    <security:http-basic />
    <security:intercept-url pattern="/api/**" 
      access="isAuthenticated()" />
</security:http>
```

## 6。应用程序

接下来，让我们创建一个简单的 REST 端点，它由我们的两个身份验证提供者保护。

要访问此端点，必须提供有效的用户名和密码。我们的身份验证提供商将验证凭据，并确定是否允许访问:

```
@RestController
public class MultipleAuthController {
    @GetMapping("/api/ping")
    public String getPing() {
        return "OK";
    }
}
```

## 7。测试

最后，现在让我们测试对安全应用程序的访问。只有在提供有效凭据的情况下，才允许访问:

```
@Autowired
private TestRestTemplate restTemplate;

@Test
public void givenMemUsers_whenGetPingWithValidUser_thenOk() {
    ResponseEntity<String> result 
      = makeRestCallToGetPing("memuser", "pass");

    assertThat(result.getStatusCodeValue()).isEqualTo(200);
    assertThat(result.getBody()).isEqualTo("OK");
}

@Test
public void givenExternalUsers_whenGetPingWithValidUser_thenOK() {
    ResponseEntity<String> result 
      = makeRestCallToGetPing("externaluser", "pass");

    assertThat(result.getStatusCodeValue()).isEqualTo(200);
    assertThat(result.getBody()).isEqualTo("OK");
}

@Test
public void givenAuthProviders_whenGetPingWithNoCred_then401() {
    ResponseEntity<String> result = makeRestCallToGetPing();

    assertThat(result.getStatusCodeValue()).isEqualTo(401);
}

@Test
public void givenAuthProviders_whenGetPingWithBadCred_then401() {
    ResponseEntity<String> result 
      = makeRestCallToGetPing("user", "bad_password");

    assertThat(result.getStatusCodeValue()).isEqualTo(401);
}

private ResponseEntity<String> 
  makeRestCallToGetPing(String username, String password) {
    return restTemplate.withBasicAuth(username, password)
      .getForEntity("/api/ping", String.class, Collections.emptyMap());
}

private ResponseEntity<String> makeRestCallToGetPing() {
    return restTemplate
      .getForEntity("/api/ping", String.class, Collections.emptyMap());
}
```

## 8。结论

在这个快速教程中，我们看到了如何在 Spring Security 中配置多个身份验证提供者。我们已经使用自定义身份验证提供程序和内存中身份验证提供程序保护了一个简单的应用程序。

我们还编写了测试来验证对我们的应用程序的访问需要能够被至少一个身份验证提供者验证的凭证。

和往常一样，实现的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221003085830/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-2 "The Full Registration/Authentication Example Project on Github ")