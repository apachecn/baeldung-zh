# Spring 安全基本认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-basic-authentication>

## 1。概述

本教程将解释如何使用 Spring 设置、配置和定制**基本认证。我们将构建在简单的 [Spring MVC 示例](/web/20220722025919/https://www.baeldung.com/spring-mvc-tutorial "Spring MVC Tutorial")之上，并使用 Spring Security 提供的基本授权机制来保护 MVC 应用程序的 UI。**

## 延伸阅读:

## [Spring Boot 安全自动配置](/web/20220722025919/https://www.baeldung.com/spring-boot-security-autoconfiguration)

A quick and practical guide to Spring Boot's default Spring Security configuration.[Read more](/web/20220722025919/https://www.baeldung.com/spring-boot-security-autoconfiguration) →

## [Spring 安全认证提供者](/web/20220722025919/https://www.baeldung.com/spring-security-authentication-provider)

How to Set Up a Custom Authentication Provider with Spring Security and the namespace configuration.[Read more](/web/20220722025919/https://www.baeldung.com/spring-security-authentication-provider) →

## [春季安全表单登录](/web/20220722025919/https://www.baeldung.com/spring-security-login)

A Spring Login Example - How to Set Up a simple Login Form, a Basic Security XML Configuration and some more Advanced Configuration Techniques.[Read more](/web/20220722025919/https://www.baeldung.com/spring-security-login) →

## 2。春季安全配置

我们可以使用 Java config 来配置 Spring 安全性:

```java
@Configuration
@EnableWebSecurity
public class CustomWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyBasicAuthenticationEntryPoint authenticationEntryPoint;

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
          .withUser("user1").password(passwordEncoder().encode("user1Pass"))
          .authorities("ROLE_USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .antMatchers("/securityNone").permitAll()
          .anyRequest().authenticated()
          .and()
          .httpBasic()
          .authenticationEntryPoint(authenticationEntryPoint);

        http.addFilterAfter(new CustomFilter(),
          BasicAuthenticationFilter.class);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

这里我们使用`httpBasic()`元素来定义扩展`WebSecurityConfigurerAdapter.`的类的`configure()`方法中的基本认证

我们也可以使用 XML 获得同样的结果:

```java
<http pattern="/securityNone" security="none"/>
<http use-expressions="true">
    <intercept-url pattern="/**" access="isAuthenticated()" />
    <http-basic />
</http>

<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="{noop}user1Pass" authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

这里相关的是配置的主`<http>`元素中的`<http-basic>`元素。这足以为整个应用程序启用基本身份验证。因为我们在本教程中不关注身份验证管理器，所以我们将使用内存管理器，用明文定义用户和密码。

启用 Spring 安全的 web 应用程序的`web.xml`已经在 [Spring 注销教程](/web/20220722025919/https://www.baeldung.com/spring-security-login#web_xml "Spring Security web.xml")中讨论过了。

## 3。消费安全应用程序

`curl`命令是我们使用安全应用程序的首选工具。

首先，让我们尝试在不提供任何安全凭证的情况下请求`/homepage.html`:

```java
curl -i http://localhost:8080/spring-security-rest-basic-auth/api/foos/1
```

我们得到预期的`401 Unauthorized`和[认证挑战](https://web.archive.org/web/20220722025919/https://datatracker.ietf.org/doc/html/rfc1945#section-10.16 "Basic Authentication Challenge"):

```java
HTTP/1.1 401 Unauthorized
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=E5A8D3C16B65A0A007CFAACAEEE6916B; Path=/spring-security-mvc-basic-auth/; HttpOnly
WWW-Authenticate: Basic realm="Spring Security Application"
Content-Type: text/html;charset=utf-8
Content-Length: 1061
Date: Wed, 29 May 2013 15:14:08 GMT
```

通常浏览器会解释这个挑战，并通过一个简单的对话框提示我们输入凭证，但是因为我们使用了`curl`，所以情况不是这样。

现在让我们请求相同的资源，主页，但是**也提供了访问它的凭证**:

```java
curl -i --user user1:user1Pass 
  http://localhost:8080/spring-security-rest-basic-auth/api/foos/1
```

因此，来自服务器的响应是`200 OK`和`Cookie`:

```java
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=301225C7AE7C74B0892887389996785D; Path=/spring-security-mvc-basic-auth/; HttpOnly
Content-Type: text/html;charset=ISO-8859-1
Content-Language: en-US
Content-Length: 90
Date: Wed, 29 May 2013 15:19:38 GMT
```

从浏览器，我们可以正常消费应用；唯一的区别是登录页面不再是一个硬性要求，因为所有浏览器都支持基本身份验证，并使用对话框提示用户输入凭证。

## 4。进一步配置–t**he 入口点**

默认情况下，Spring Security 提供的`BasicAuthenticationEntryPoint`会将一整页的`401 Unauthorized`响应返回给客户端。这个错误的 HTML 表示在浏览器中呈现得很好。相反，它不太适合其他场景，比如 REST API，其中 json 表示可能是首选。

名称空间对于这个新需求也足够灵活。为了解决这个问题，可以覆盖入口点:

```java
<http-basic entry-point-ref="myBasicAuthenticationEntryPoint" />
```

新的入口点被定义为一个标准 bean:

```java
@Component
public class MyBasicAuthenticationEntryPoint extends BasicAuthenticationEntryPoint {

    @Override
    public void commence(
      HttpServletRequest request, HttpServletResponse response, AuthenticationException authEx) 
      throws IOException, ServletException {
        response.addHeader("WWW-Authenticate", "Basic realm="" + getRealmName() + """);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        PrintWriter writer = response.getWriter();
        writer.println("HTTP Status 401 - " + authEx.getMessage());
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        setRealmName("Baeldung");
        super.afterPropertiesSet();
    }
}
```

通过直接写入 HTTP 响应，我们现在可以完全控制响应体的格式。

## 5。美芬依赖

Spring 安全性的 Maven 依赖性在之前的 [Spring Security with Maven 文章](/web/20220722025919/https://www.baeldung.com/spring-security-with-maven "Spring Security with Maven")中已经讨论过了。我们需要`spring-security-web`和`spring-security-config`在运行时都可用。

## 6。结论

在本文中，我们用 Spring Security 和基本认证保护了一个 MVC 应用程序。我们讨论了 XML 配置，并用简单的 curl 命令使用了这个应用程序。最后，我们控制了确切的错误消息格式，从标准的 HTML 错误页面转变为定制的文本或 JSON 格式。

本文的完整实现可以在 GitHub 项目中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。

当项目在本地运行时，可以在以下位置访问示例 HTML:

[http://localhost:8080/spring-security-rest-basic-auth/API/foos/1](https://web.archive.org/web/20220722025919/http://localhost:8080/spring-security-mvc-basic-auth/homepage.html)。