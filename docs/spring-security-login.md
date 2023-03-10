# Spring 安全表单登录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-login>

## 1。简介

本教程将重点介绍使用 Spring Security 的**登录。我们将在[之前的 Spring MVC 示例](/web/20220727020632/https://www.baeldung.com/spring-mvc-tutorial "MVC Tutorial")的基础上构建，因为这是设置 web 应用程序和登录机制的必要部分。**

## 延伸阅读:

## [Spring Security–登录后重定向到以前的 URL](/web/20220727020632/https://www.baeldung.com/spring-security-redirect-login)

A short example of redirection after login in Spring Security[Read more](/web/20220727020632/https://www.baeldung.com/spring-security-redirect-login) →

## [两个带有 Spring Security 的登录页面](/web/20220727020632/https://www.baeldung.com/spring-security-two-login-pages)

A quick and practical guide to configuring Spring Security with two separate login pages.[Read more](/web/20220727020632/https://www.baeldung.com/spring-security-two-login-pages) →

## [春季安全表单登录](/web/20220727020632/https://www.baeldung.com/spring-security-login)

A Spring Login Example - How to Set Up a simple Login Form, a Basic Security XML Configuration and some more Advanced Configuration Techniques.[Read more](/web/20220727020632/https://www.baeldung.com/spring-security-login) →

## 2。美芬依赖

使用 Spring Boot 时， [`spring-boot-starter-security`](https://web.archive.org/web/20220727020632/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security) 启动器将自动包含所有依赖项，例如`spring-security-core`、`spring-security-web`和`spring-security-config`等等:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
```

如果我们不使用 Spring Boot，请参见 [Spring Security with Maven 文章](/web/20220727020632/https://www.baeldung.com/spring-security-with-maven "Maven Spring Security tutorial")，其中描述了如何添加所有必需的依赖项。标准`spring-security-web`和`spring-security-config`都需要。

## 3。Spring 安全 Java 配置

让我们从创建一个扩展了`WebSecurityConfigurerAdapter.`的 Spring 安全配置类开始

通过添加`@EnableWebSecurity`，我们获得了 Spring 安全性和 MVC 集成支持:

```java
@Configuration
@EnableWebSecurity
public class SecSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(final AuthenticationManagerBuilder auth) throws Exception {
        // authentication manager (see below)
    }

    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        // http builder configurations for authorize requests and form login (see below)
    }
}
```

在这个例子中，我们使用了内存认证，并定义了三个用户。

接下来，我们将检查用于创建表单登录配置的元素。

让我们从构建身份验证管理器开始。

### 3.1。认证管理器

身份验证提供者由一个简单的内存实现`InMemoryUserDetailsManager`提供支持。当还不需要完整的持久性机制时，这对于快速原型开发很有用:

```java
protected void configure(final AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user1").password(passwordEncoder().encode("user1Pass")).roles("USER")
        .and()
        .withUser("user2").password(passwordEncoder().encode("user2Pass")).roles("USER")
        .and()
        .withUser("admin").password(passwordEncoder().encode("adminPass")).roles("ADMIN");
}
```

这里我们将为三个用户配置硬编码的用户名、密码和角色。

从 Spring 5 开始，我们还必须定义一个密码编码器。在我们的例子中，我们将使用`BCryptPasswordEncoder:`

```java
@Bean 
public PasswordEncoder passwordEncoder() { 
    return new BCryptPasswordEncoder(); 
}
```

接下来让我们配置`HttpSecurity.`

### 3.2。授权请求的配置

我们将从授权请求的必要配置开始。

这里我们允许匿名访问`/login`,这样用户就可以进行身份验证。我们将把`/admin`限制为`ADMIN`角色，并保护其他一切:

```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    http
      .csrf().disable()
      .authorizeRequests()
      .antMatchers("/admin/**").hasRole("ADMIN")
      .antMatchers("/anonymous*").anonymous()
      .antMatchers("/login*").permitAll()
      .anyRequest().authenticated()
      .and()
      // ...
}
```

请注意，`antMatchers()`元素的顺序非常重要；**首先需要更具体的规则，然后是更通用的规则**。

### 3.3。表单登录配置

接下来，我们将扩展上述表单登录和注销的配置:

```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    http
      // ...
      .and()
      .formLogin()
      .loginPage("/login.html")
      .loginProcessingUrl("/perform_login")
      .defaultSuccessUrl("/homepage.html", true)
      .failureUrl("/login.html?error=true")
      .failureHandler(authenticationFailureHandler())
      .and()
      .logout()
      .logoutUrl("/perform_logout")
      .deleteCookies("JSESSIONID")
      .logoutSuccessHandler(logoutSuccessHandler());
}
```

*   `loginPage() `–自定义登录页面
*   `loginProcessingUrl()`–向其提交用户名和密码的 URL
*   `defaultSuccessUrl()`–成功登录后的登录页面
*   `failureUrl()`–登录失败后的登录页面
*   `logoutUrl()`–自定义注销

## 4.向 Web 应用程序添加 Spring 安全性

为了使用上面定义的 Spring 安全配置，我们需要将其附加到 web 应用程序。

我们将使用`WebApplicationInitializer`，所以我们不需要提供任何`web.xml:`

```java
public class AppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext sc) {

        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(SecSecurityConfig.class);

        sc.addListener(new ContextLoaderListener(root));

        sc.addFilter("securityFilter", new DelegatingFilterProxy("springSecurityFilterChain"))
          .addMappingForUrlPatterns(null, false, "/*");
    }
}
```

注意，如果我们使用 Spring Boot 应用程序，这个初始化器是不必要的。关于 Spring Boot 如何加载安全配置的更多细节，请看我们关于 [Spring Boot 安全自动配置的文章](/web/20220727020632/https://www.baeldung.com/spring-boot-security-autoconfiguration)。

## 5。Spring 安全 XML 配置

让我们看看相应的 XML 配置。

整个项目使用 Java 配置，因此我们需要通过 Java `@Configuration`类导入 XML 配置文件:

```java
@Configuration
@ImportResource({ "classpath:webSecurityConfig.xml" })
public class SecSecurityConfig {
   public SecSecurityConfig() {
      super();
   }
}
```

和 Spring 安全 XML 配置，`webSecurityConfig.xml`:

```java
<http use-expressions="true">
    <intercept-url pattern="/login*" access="isAnonymous()" />
    <intercept-url pattern="/**" access="isAuthenticated()"/>

    <form-login login-page='/login.html' 
      default-target-url="/homepage.html" 
      authentication-failure-url="/login.html?error=true" />
    <logout logout-success-url="/login.html" />
</http>

<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="user1Pass" authorities="ROLE_USER" />
        </user-service>
        <password-encoder ref="encoder" />
    </authentication-provider>
</authentication-manager>

<beans:bean id="encoder" 
  class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder">
</beans:bean>
```

## 6。`web.xml`

**在引入 Spring 4** 之前，我们曾经在`web.xml;` 中配置 Spring 安全，只是在标准的 Spring MVC `web.xml`中增加了一个额外的过滤器:

```java
<display-name>Spring Secured Application</display-name>

<!-- Spring MVC -->
<!-- ... -->

<!-- Spring Security -->
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

过滤器—`DelegatingFilterProxy`—只是委托给一个 Spring 管理的 bean—`FilterChainProxy`——它本身能够受益于完整的 Spring bean 生命周期管理等等。

## 7 .**。登录表单**

登录表单页面将用 Spring MVC 注册，使用简单的机制[将视图名称映射到 URL](/web/20220727020632/https://www.baeldung.com/spring-mvc-tutorial#configviews "Spring MVC View Configuration")。此外，中间不需要显式控制器:

```java
registry.addViewController("/login.html");
```

这当然对应于`login.jsp`:

```java
<html>
<head></head>
<body>
   <h1>Login</h1>
   <form name='f' action="login" method='POST'>
      <table>
         <tr>
            <td>User:</td>
            <td><input type='text' name='username' value=''></td>
         </tr>
         <tr>
            <td>Password:</td>
            <td><input type='password' name='password' /></td>
         </tr>
         <tr>
            <td><input name="submit" type="submit" value="submit" /></td>
         </tr>
      </table>
  </form>
</body>
</html>
```

**Spring 登录表单**有以下相关构件:

*   `login`–发布表单以触发认证过程的 URL
*   `username`–用户名
*   `password`–密码

## 8。进一步配置 Spring 登录

当我们在上面介绍 Spring 安全配置时，我们简要讨论了登录机制的一些配置。现在让我们进入一些更详细的内容。

在 Spring Security 中覆盖大多数默认设置的一个原因是**隐藏应用程序受到 Spring Security 的保护。**我们还希望尽量减少潜在攻击者对应用程序的了解。

完全配置后，登录元素如下所示:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
      .loginPage("/login.html")
      .loginProcessingUrl("/perform_login")
      .defaultSuccessUrl("/homepage.html",true)
      .failureUrl("/login.html?error=true")
}
```

或者相应的 XML 配置:

```java
<form-login 
  login-page='/login.html' 
  login-processing-url="/perform_login" 
  default-target-url="/homepage.html"
  authentication-failure-url="/login.html?error=true" 
  always-use-default-target="true"/>
```

### 8.1。登录页面

接下来，我们将使用`loginPage() method:`配置一个定制的登录页面

```java
http.formLogin()
  .loginPage("/login.html")
```

类似地，我们可以使用 XML 配置:

```java
login-page='/login.html'
```

如果我们不指定这个，Spring Security 将在`/login` URL 生成一个非常基本的登录表单。

### 8.2。用于登录的帖子 URL

Spring 登录触发认证过程的默认 URL 是`/login,`，以前是 [Spring Security 4](https://web.archive.org/web/20220727020632/https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-xml.html#m3to4-xmlnamespace-form-login) 之前的`/j_spring_security_check`。

我们可以使用`loginProcessingUrl`方法来覆盖这个 URL:

```java
http.formLogin()
  .loginProcessingUrl("/perform_login")
```

我们还可以使用 XML 配置:

```java
login-processing-url="/perform_login"
```

通过覆盖这个默认的 URL，我们隐藏了应用程序实际上是受 Spring 安全保护的。此信息不应对外公开。

### 8.3。成功登陆页面

成功登录后，我们将被重定向到默认情况下是 web 应用程序根目录的页面。

我们可以通过`defaultSuccessUrl()`方法覆盖它:

```java
http.formLogin()
  .defaultSuccessUrl("/homepage.html")
```

或者使用 XML 配置:

```java
default-target-url="/homepage.html"
```

如果`always-use-default-target`属性设置为 true，那么用户总是被重定向到这个页面。如果该属性设置为 false，那么在提示用户进行身份验证之前，用户将被重定向到他们想要访问的上一个页面。

### 8.4。登陆页面失败

与登录页面类似，默认情况下，登录失败页面由 Spring Security 在`/login?`错误时自动生成。

要覆盖它，我们可以使用`failureUrl()`方法:

```java
http.formLogin()
  .failureUrl("/login.html?error=true")
```

或者使用 XML:

```java
authentication-failure-url="/login.html?error=true"
```

## 9。结论

在这个 **Spring 登录示例**中，我们配置了一个简单的认证过程。我们还讨论了 Spring 安全登录表单、安全配置和一些更高级的定制。

本文的实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。

当项目在本地运行时，可以在以下位置访问示例 HTML:

```java
http://localhost:8080/spring-security-mvc-login/login.html
```