# 在百里香中显示登录用户的信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-user-info>

## 1.概观

在这个快速教程中，**我们将看看如何** **在百里叶**中显示登录用户的信息。

我们将扩展我们在文章 [Spring Security with 百里香叶](/web/20220923120147/https://www.baeldung.com/spring-security-thymeleaf)中构建的项目。首先，我们将添加一个自定义模型来存储用户信息和检索这些信息的服务。之后，我们将使用来自百里香附加模块的 Spring 安全方言来显示它。

## 2.`UserDetails` 实施

是 Spring Security 的一个接口，用于保存与安全性无关的用户信息。

我们将用一些定制字段创建我们的`UserDetails`接口的实现，作为存储我们的已验证用户详细信息的模型。但是，为了处理更少的字段和方法，我们将扩展默认的框架实现，即`User`类:

```java
public class CustomUserDetails extends User {

    private final String firstName;
    private final String lastName;
    private final String email;

    private CustomUserDetails(Builder builder) {
        super(builder.username, builder.password, builder.authorities);
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.email = builder.email;
    }

    // omitting getters and static Builder class
}
```

## 3.`UserDetailsService` 实施

框架的`UserDetailsService`单一方法接口负责在认证过程中获取`UserDetails`。

因此，为了能够加载我们的`CustomUserDetails,`，我们需要实现`UserDetailsService `接口。对于我们的例子，我们将把用户详细信息硬编码并存储在一个`Map`中，把用户名作为键:

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final PasswordEncoder passwordEncoder;
    private final Map<String, CustomUserDetails> userRegistry = new HashMap<>();

    // omitting constructor

    @PostConstruct
    public void init() {
        userRegistry.put("user", new CustomUserDetails.Builder().withFirstName("Mark")
          .withLastName("Johnson")
          .withEmail("[[email protected]](/web/20220923120147/https://www.baeldung.com/cdn-cgi/l/email-protection)")
          .withUsername("user")
          .withPassword(passwordEncoder.encode("password"))
          .withAuthorities(Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER")))
          .build());
        userRegistry.put("admin", new CustomUserDetails.Builder().withFirstName("James")
          .withLastName("Davis")
          .withEmail("[[email protected]](/web/20220923120147/https://www.baeldung.com/cdn-cgi/l/email-protection)")
          .withUsername("admin")
          .withPassword(passwordEncoder.encode("password"))
          .withAuthorities(Collections.singletonList(new SimpleGrantedAuthority("ROLE_ADMIN")))
          .build());
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        CustomUserDetails userDetails = userRegistry.get(username);
        if (userDetails == null) {
            throw new UsernameNotFoundException(username);
        }
        return userDetails;
    }
}
```

此外，为了实现所需的`loadUserByUsername()`方法，我们通过用户名从注册表`Map`中获取相应的`CustomUserDetails`对象。但是，用户详细信息将存储在生产环境的存储库中，并从存储库中检索。

## 4.Spring 安全配置

首先，我们需要在 Spring Security 的配置中添加`UserDetailsService`，它将被连接到`CustomUserDetailsService`实现。进一步，我们将通过相应的方法在`HttpSecurity`实例上设置它。剩下的只是最低限度的安全配置，要求对用户进行身份验证，并配置`/login`、`/logout,`和`/index`端点:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    private final UserDetailsService userDetailsService;

    // omitting constructor

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.userDetailsService(userDetailsService)
            .authorizeRequests()
            .anyRequest()
            .authenticated()
            .and()
            .formLogin()
            .loginPage("/login")
            .permitAll()
            .successForwardUrl("/index")
            .and()
            .logout()
            .permitAll()
            .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
            .logoutSuccessUrl("/login");
    }
}
```

## 5.显示登录的用户信息

**百里香附加模块提供了对`Authentication`对象、**的访问，通过安全方言，我们可以在百里香页面上显示登录的用户信息。

通过`Authentication`对象上的`principal`字段可以访问`CustomUserDetails`对象。例如，我们可以使用` sec:authentication=”principal.firstName”`来访问`firstName`字段:

```java
<!DOCTYPE html>
<html xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head>
<title>Welcome to Spring Security Thymeleaf tutorial</title>
</head>
<body>
    <h2>Welcome</h2>
    <p>Spring Security Thymeleaf tutorial</p>
    <div sec:authorize="hasRole('USER')">Text visible to user.</div>
    <div sec:authorize="hasRole('ADMIN')">Text visible to admin.</div>
    <div sec:authorize="isAuthenticated()">Text visible only to authenticated users.</div>
    Authenticated username:
    <div sec:authentication="name"></div>
    Authenticated user's firstName:
    <div sec:authentication="principal.firstName"></div>
    Authenticated user's lastName:
    <div sec:authentication="principal.lastName"></div>
    Authenticated user's email:
    <div sec:authentication="principal.lastName"></div>
    Authenticated user roles:
    <div sec:authentication="principal.authorities"></div>
</body>
</html>
```

或者，编写不带`sec:authentication`属性的安全方言表达式的等效语法是使用 Spring 表达式语言。因此，我们可以使用 Spring 表达式语言格式显示`firstName`字段，如果我们对它更熟悉的话:

```java
<div th:text="${#authentication.principal.firstName}"></div>
```

## 6.结论

在本文中，我们看到了如何在 Spring Boot 应用程序中使用 Spring Security 的支持，在 Thymeleaf 中显示登录用户的信息。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220923120147/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-thymeleaf)