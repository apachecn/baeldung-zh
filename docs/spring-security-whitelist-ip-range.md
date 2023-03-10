# Spring 安全–白名单 IP 范围

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-whitelist-ip-range>

## 1.概观

在本教程中，**我们将讨论如何在 Spring Security** 中将 IP 范围列入白名单。

我们将看看 Java 和 XML 配置。我们还将看到如何使用自定义的`AuthenticationProvider`将 IP 范围列入白名单。

## 2.Java 配置

首先，让我们探讨一下 Java 配置。

**我们可以使用`hasIpAddress()`只允许给定 IP 地址的用户访问特定的资源**。

下面是一个使用`hasIpAddress()`的简单安全配置:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .antMatchers("/login").permitAll()
          .antMatchers("/foos/**").hasIpAddress("11.11.11.11")
          .anyRequest().authenticated()
          .and()
          .formLogin().permitAll()
          .and()
          .csrf().disable();
    }

    // ...

}
```

在此配置中，只有 IP 地址为“11.11.11.11”的用户才能访问“/foos”资源。拥有白名单 IP 的用户在访问“/foos/”URL 之前也不需要登录。

如果我们希望拥有“11 . 11 . 11 . 11”IP 的用户首先登录，我们可以在以下形式的表达式中使用该方法:

```java
//...
.antMatchers("/foos/**")
.access("isAuthenticated() and hasIpAddress('11.11.11.11')")
//...
```

## 3.XML 配置

接下来，让我们看看如何使用 XML 配置将 IP 范围列入白名单:

我们在这里也将使用`hasIpAddress()`:

```java
<security:http>
    <security:form-login/>
    <security:intercept-url pattern="/login" access="permitAll()" />
    <security:intercept-url pattern="/foos/**" access="hasIpAddress('11.11.11.11')" />
    <security:intercept-url pattern="/**" access="isAuthenticated()" />
</security:http>

// ...
```

## 4.现场测试

现在，这里有一个简单的现场测试，以确保一切正常工作。

首先，我们将确保任何用户登录后都可以访问主页:

```java
@Test
public void givenUser_whenGetHomePage_thenOK() {
    Response response = RestAssured.given().auth().form("john", "123")
      .get("http://localhost:8082/");

    assertEquals(200, response.getStatusCode());
    assertTrue(response.asString().contains("Welcome"));
}
```

接下来，我们将确保即使经过身份验证的用户也无法访问“/foos”资源，除非他们的 IP 被列入白名单:

```java
@Test
public void givenUserWithWrongIP_whenGetFooById_thenForbidden() {
    Response response = RestAssured.given().auth().form("john", "123")
      .get("http://localhost:8082/foos/1");

    assertEquals(403, response.getStatusCode());
    assertTrue(response.asString().contains("Forbidden"));
}
```

请注意，我们无法从 localhost“127 . 0 . 0 . 1”访问“/foos”资源，因为只有版本为“11.11.11.11”的用户才能访问它。

## 5.使用自定义的白名单`AuthenticationProvider`

最后，**我们将了解如何通过构建自定义的`AuthenticationProvider`** 将 IP 范围列入白名单。

我们已经看到了如何使用`hasIpAddress()`将 IP 范围列入白名单，以及如何将它与其他表达式混合使用。但是有时候，**我们需要更多定制**。

在以下示例中，我们将多个 IP 地址列入白名单，只有来自这些 IP 地址的用户才能登录我们的系统:

```java
@Component
public class CustomIpAuthenticationProvider implements AuthenticationProvider {

   Set<String> whitelist = new HashSet<String>();

    public CustomIpAuthenticationProvider() {
        whitelist.add("11.11.11.11");
        whitelist.add("12.12.12.12");
    }

    @Override
    public Authentication authenticate(Authentication auth) throws AuthenticationException {
        WebAuthenticationDetails details = (WebAuthenticationDetails) auth.getDetails();
        String userIp = details.getRemoteAddress();
        if(! whitelist.contains(userIp)){
            throw new BadCredentialsException("Invalid IP Address");
        }
        //...
}
```

现在，我们将在安全配置中使用我们的`CustomIpAuthenticationProvider`:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomIpAuthenticationProvider authenticationProvider;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       auth.authenticationProvider(authenticationProvider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .antMatchers("/login").permitAll()
          .anyRequest().authenticated()
          .and().formLogin().permitAll()
          .and().csrf().disable();
    }

}
```

这里，我们使用了`WebAuthenticationDetails getRemoteAddress()`方法来获取用户的 IP 地址。

因此，只有拥有白名单 IP 的用户才能访问我们的系统。

这是一个基本的实现，但是我们可以使用用户的 IP 来定制我们的`AuthenticationProvider`。例如，我们可以在注册时存储 IP 地址和用户详细信息，并在认证期间在我们的`AuthenticationProvider.`中进行比较

## 6.结论

我们学习了如何使用 Java 和 XML 配置在 Spring Security 中将 IP 范围列入白名单。我们还学习了如何通过构建自定义的`AuthenticationProvider`将 IP 范围列入白名单。

完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628130556/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-1)