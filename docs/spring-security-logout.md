# Spring 安全注销

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-logout>

## 1。概述

这篇文章建立在我们的[表单登录教程](/web/20220628060112/https://www.baeldung.com/spring-security-login "Spring Security Form Login")的基础上，并且将集中在如何用 Spring Security 配置**注销。**

## 延伸阅读:

## [Spring Security:使用数据库支持的用户详细信息服务进行身份验证](/web/20220628060112/https://www.baeldung.com/spring-security-authentication-with-a-database)

A quick guide to to create a custom database-backed UserDetailsService for authentication with Spring Security.[Read more](/web/20220628060112/https://www.baeldung.com/spring-security-authentication-with-a-database) →

## [Spring 方法安全性介绍](/web/20220628060112/https://www.baeldung.com/spring-security-method-security)

A guide to method-level security using the Spring Security framework.[Read more](/web/20220628060112/https://www.baeldung.com/spring-security-method-security) →

## [Spring Security–登录后重定向到以前的 URL](/web/20220628060112/https://www.baeldung.com/spring-security-redirect-login)

A short example of redirection after login in Spring Security[Read more](/web/20220628060112/https://www.baeldung.com/spring-security-redirect-login) →

## 2。基本配置

使用`logout()`方法的 **Spring 注销功能**的基本配置非常简单:

```java
@Configuration
@EnableWebSecurity
public class SecSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        http
          //...
          .logout()
          //...
   }
   //...
}
```

并使用 XML 配置:

```java
<http>

    ...    
    <logout/>

</http>
```

该元素启用了默认的注销机制——该机制被配置为使用下面的**注销 url** : `/logout`，它以前是 [Spring Security 4](https://web.archive.org/web/20220628060112/https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-xml.html#m3to4-xmlnamespace-logout) `.`之前的`/j_spring_security_logout`

## 3。JSP 和注销链接

继续这个简单的例子，在 web 应用程序中提供**注销链接**的方法是:

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
   <head></head>
   <body>
      <a href="<c:url value="/logout" />">Logout</a>
   </body>
</html>
```

## 4。高级定制

### 4.1。`logoutSuccessUrl()`

注销过程成功执行后，Spring Security 会将用户重定向到指定的页面。默认情况下，这是根页面(`“/”`)，但这是可配置的:

```java
//...
.logout()
.logoutSuccessUrl("/afterlogout.html")
//...
```

这也可以使用 XML 配置来完成:

```java
<logout logout-success-url="/afterlogout.html" />
```

根据应用程序的不同，一个好的做法是将用户重定向回登录页面:

```java
//...
.logout()
.logoutSuccessUrl("/login.html")
//...
```

### 4.2。`logoutUrl()`

与 Spring Security 中的其他默认设置类似，实际触发注销机制的 URL 也有一个默认设置—`/logout`。

但是，更改这个默认值是一个好主意，这样可以确保不会发布关于使用什么框架来保护应用程序的信息:

```java
.logout()
.logoutUrl("/perform_logout")
```

通过 XML:

```java
<logout 
  logout-success-url="/anonymous.html" 
  logout-url="/perform_logout" />
```

### 4.3。`invalidateHttpSession`和`deleteCookies`

这两个高级属性控制会话失效以及用户注销时要删除的 cookies 列表。因此，`invalidateHttpSession`允许建立会话，这样当注销发生时会话不会失效(默认为`true`)。

`deleteCookies`方法也很简单:

```java
.logout()
.logoutUrl("/perform_logout")
.invalidateHttpSession(true)
.deleteCookies("JSESSIONID")
```

XML 版本是:

```java
<logout 
  logout-success-url="/anonymous.html" 
  logout-url="/perform_logout"
  delete-cookies="JSESSIONID" />
```

### 4.4。`logoutSuccessHandler()`

对于更高级的场景，名称空间不够灵活，Spring 上下文中的`LogoutSuccessHandler` bean 可以替换为自定义引用:

```java
@Bean
public LogoutSuccessHandler logoutSuccessHandler() {
    return new CustomLogoutSuccessHandler();
}

//...
.logout()
.logoutSuccessHandler(logoutSuccessHandler());
//...
```

等效的 XML 配置是:

```java
<logout 
  logout-url="/perform_logout"
  delete-cookies="JSESSIONID"
  success-handler-ref="customLogoutSuccessHandler" />

...
<beans:bean name="customUrlLogoutSuccessHandler" />
```

当用户成功注销时需要运行的任何**定制应用程序逻辑都可以用定制注销成功处理程序来实现。例如，一个简单的审计机制跟踪用户触发注销时所在的最后一个页面:**

```java
public class CustomLogoutSuccessHandler extends 
  SimpleUrlLogoutSuccessHandler implements LogoutSuccessHandler {

    @Autowired 
    private AuditService auditService; 

    @Override
    public void onLogoutSuccess(
      HttpServletRequest request, 
      HttpServletResponse response, 
      Authentication authentication) 
      throws IOException, ServletException {

        String refererUrl = request.getHeader("Referer");
        auditService.track("Logout from: " + refererUrl);

        super.onLogoutSuccess(request, response, authentication);
    }
}
```

此外，请记住，这个自定义 bean 负责确定用户注销后被定向到的目的地。因此，将`logoutSuccessHandler`属性与`logoutSuccessUrl`配对是行不通的，因为两者涵盖了相似的功能。

## 5。结论

在这个例子中，我们首先用 Spring Security 设置了一个简单的注销示例，然后讨论了更高级的可用选项。

这个 Spring 注销教程的实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

当项目在本地运行时，可以在以下位置访问示例 HTML:

[http://localhost:8080/spring-security-MVC-log in/log in . html](https://web.archive.org/web/20220628060112/http://localhost:8080/spring-security-login/login.html)