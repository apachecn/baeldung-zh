# Spring Security 的 Java 配置简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-config-spring-security>

## 1。概述

这篇文章是对 Spring Security 的 Java 配置的介绍，它使用户能够轻松地配置 Spring Security，而不需要使用 XML T2。

Java 配置在 [Spring 3.1](https://web.archive.org/web/20220703144957/https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/new-in-3.1.html) 中被添加到 Spring 框架，在 [Spring 3.2](https://web.archive.org/web/20220703144957/https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/new-in-3.2.html) 中被扩展到 Spring 安全，并在一个带注释的类`@Configuration`中被定义。

## 2。Maven 设置

要在 Maven 项目中使用 Spring 安全性，我们首先需要在项目`pom.xml`中拥有`spring-security-core`依赖项:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>5.3.3.RELEASE</version>
</dependency>
```

最新版本总是可以在[这里](https://web.archive.org/web/20220703144957/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-security-core%22)找到。

## 3。Java 配置的网络安全

让我们从一个 Spring Security Java 配置的基本示例开始:

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) 
      throws Exception {
        auth.inMemoryAuthentication().withUser("user")
          .password(passwordEncoder().encode("password")).roles("USER");
    }
}
```

您可能已经注意到，该配置设置了一个基本的内存中身份验证配置。另外，从春天 5 号开始，我们需要一个 [`PasswordEncoder`](/web/20220703144957/https://www.baeldung.com/spring-security-5-default-password-encoder) bean:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

## 4。HTTP 安全

为了在 Spring 中启用 HTTP 安全性，我们需要扩展`WebSecurityConfigurerAdapter`以在`configure(HttpSecurity http)`方法中提供默认配置:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .anyRequest().authenticated()
      .and().httpBasic();
} 
```

上述默认配置确保对应用程序的任何请求都通过基于表单的登录或 HTTP 基本身份验证进行身份验证。

此外，它与以下 XML 配置完全相似:

```java
<http>
    <intercept-url pattern="/**" access="isAuthenticated()"/>
    <form-login />
    <http-basic />
</http>
```

## 5。表单登录

有趣的是，Spring Security 自动生成一个登录页面，基于启用的特性并使用 URL 的标准值来处理提交的登录:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .anyRequest().authenticated()
      .and().formLogin()
      .loginPage("/login").permitAll();
}
```

在这里，自动生成的登录页面便于快速启动和运行。

## 6。角色授权

现在让我们使用角色在每个 URL 上配置一些简单的授权:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .antMatchers("/", "/home").access("hasRole('USER')")
      .antMatchers("/admin/**").hasRole("ADMIN")
      .and()
      // some more method calls
      .formLogin();
}
```

注意我们是如何通过`access.`使用类型安全 API—`hasRole`——以及基于表达式的 API 的

## 7。注销

和 Spring 安全性的许多其他方面一样，logout 有一些框架提供的很好的缺省值。

默认情况下，注销请求会使会话无效，清除任何身份验证缓存，清除`SecurityContextHolder`并重定向到登录页面。

下面是一个简单的注销配置:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.logout();
}
```

但是，如果您想对可用的处理程序进行更多的控制，下面是一个更完整的实现:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.logout().logoutUrl("/my/logout")
      .logoutSuccessUrl("/my/index")
      .logoutSuccessHandler(logoutSuccessHandler) 
      .invalidateHttpSession(true)
      .addLogoutHandler(logoutHandler)
      .deleteCookies(cookieNamesToClear)
      .and()
      // some other method calls
}
```

## 8。认证

让我们来看看允许使用 Spring Security 进行认证的另一种方式。

### 8.1。内存认证

我们将从一个简单的内存配置开始:

```java
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) 
  throws Exception {
    auth.inMemoryAuthentication()
      .withUser("user").password(passwordEncoder().encode("password")).roles("USER")
      .and()
      .withUser("admin").password(passwordEncoder().encode("password")).roles("USER", "ADMIN");
} 
```

### 8.2。JDBC 认证

要将其迁移到 JDBC，您只需在应用程序中定义一个数据源，并直接使用它:

```java
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) 
  throws Exception {
    auth.jdbcAuthentication().dataSource(dataSource)
      .withDefaultSchema()
      .withUser("user").password(passwordEncoder().encode("password")).roles("USER")
      .and()
      .withUser("admin").password(passwordEncoder().encode("password")).roles("USER", "ADMIN");
} 
```

当然，对于以上两个例子，我们还需要定义第 3 节中概述的`PasswordEncoder` bean。

## 9。结论

在这个快速教程中，我们回顾了 Spring Security 的 Java 配置基础，并重点关注了说明最简单配置场景的代码示例。