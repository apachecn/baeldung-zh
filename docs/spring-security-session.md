# 使用 Spring Security 控制会话

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-session>

## 1。概述

在本教程中，我们将说明 Spring Security 如何允许我们控制 HTTP 会话。

这种控制范围从会话超时到启用并发会话和其他高级安全配置。

## 延伸阅读:

## [在 Spring Security 中检索用户信息](/web/20220625233613/https://www.baeldung.com/get-user-in-spring-security)

How to get the currently logged in user with Spring Security.[Read more](/web/20220625233613/https://www.baeldung.com/get-user-in-spring-security) →

## [春天的安全记得我](/web/20220625233613/https://www.baeldung.com/spring-security-remember-me)

Cookie Remember Me example with Spring Security.[Read more](/web/20220625233613/https://www.baeldung.com/spring-security-remember-me) →

## [春安注销](/web/20220625233613/https://www.baeldung.com/spring-security-logout)

Spring Logout Example - how to configure the logout url, the logout-succcess-url and how to use a custom bean to handle advanced logout scenarios.[Read more](/web/20220625233613/https://www.baeldung.com/spring-security-logout) →

## 2。会话是何时创建的？

我们可以精确地控制何时创建会话，以及 Spring Security 将如何与之交互:

*   `**always**` –如果会话不存在，将始终创建会话。
*   `**ifRequired**` –仅在需要时才创建会话(**默认**)。
*   `**never**` –框架永远不会自己创建会话，但是如果它已经存在，它会使用它。
*   `**stateless**` –Spring Security 不会创建或使用任何会话。

```java
<http create-session="ifRequired">...</http>
```

以下是 Java 配置:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
}
```

理解这一点非常重要:这个配置只控制 Spring Security 做什么，而不是整个应用程序。如果我们指示 Spring Security 不创建会话，它就不会创建，但我们的应用程序可能会创建！

默认情况下， **Spring Security 会在需要一个**时创建一个会话—这是“`ifRequired`”。

对于**一个更无状态的应用程序**,`never`选项将确保 Spring Security 本身不会创建任何会话。但是如果应用程序创建了一个，Spring Security 就会使用它。

最后，最严格的会话创建选项“`stateless`”是一个**保证，应用程序根本不会创建任何会话。**

这是在 Spring 3.1 中引入了的[，它将有效地跳过 Spring 安全过滤器链的部分——主要是与会话相关的部分，如`HttpSessionSecurityContextRepository`、`SessionManagementFilter`和`RequestCacheFilter`。](https://web.archive.org/web/20220625233613/https://jira.springsource.org/browse/SEC-1424 "JIRA - Add new option create-session="stateless"")

这些更严格的控制机制直接意味着**cookie 不被使用**，因此**每个请求都需要重新认证。**

这种无状态架构与 REST APIs 及其无状态约束配合得很好。它们还可以很好地与基本身份验证和摘要式身份验证等身份验证机制配合使用。

## 3。引擎盖下

在运行认证过程之前，Spring Security 将运行一个过滤器，负责存储请求之间的安全上下文。这就是`SecurityContextPersistenceFilter`。

默认情况下，将根据策略`HttpSessionSecurityContextRepository`存储上下文，该策略使用 HTTP 会话作为存储。

对于严格的`create-session=”stateless”`属性，该策略将被另一个策略`NullSecurityContextRepository`替换，并且**不会创建任何会话，也不会使用**来保持上下文。

## 4。并发会话控制

当一个已经通过身份验证的用户试图再次通过身份验证时，应用程序可以用几种方法之一来处理这个事件。它可以使用户的活动会话无效，并用新的会话再次对用户进行身份验证，或者允许两个会话同时存在。

启用并发`session-control`支持的第一步是在`web.xml`中添加以下监听器:

```java
<listener>
    <listener-class>
      org.springframework.security.web.session.HttpSessionEventPublisher
    </listener-class>
</listener>
```

或者我们可以把它定义为一个 Bean:

```java
@Bean
public HttpSessionEventPublisher httpSessionEventPublisher() {
    return new HttpSessionEventPublisher();
}
```

这对于确保 Spring 安全会话注册表在会话被销毁时得到通知至关重要。

为了允许同一个用户有多个并发会话，应该在 XML 配置中使用`<session-management>`元素:

```java
<http ...>
    <session-management>
        <concurrency-control max-sessions="2" />
    </session-management>
</http>
```

或者我们可以通过 Java 配置来实现:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.sessionManagement().maximumSessions(2)
}
```

## 5。会话超时

### 5.1.处理会话超时

会话超时后，如果用户发送一个带有**过期会话 id** 的请求，他们将被重定向到一个可通过名称空间配置的 URL:

```java
<session-management>
    <concurrency-control expired-url="/sessionExpired.html" ... />
</session-management>
```

类似地，如果用户发送一个会话 id 未过期但完全无效的请求，他们也会被重定向到一个可配置的 URL:

```java
<session-management invalid-session-url="/invalidSession.html">
    ...
</session-management>
```

下面是相应的 Java 配置:

```java
http.sessionManagement()
  .expiredUrl("/sessionExpired.html")
  .invalidSessionUrl("/invalidSession.html");
```

### 5.2.使用 Spring Boot 配置会话超时

我们可以使用属性轻松配置嵌入式服务器的会话超时值:

```java
server.servlet.session.timeout=15m
```

如果我们不指定持续时间单位，Spring 将假定它是秒。

简而言之，使用这种配置，会话将在 15 分钟不活动后过期。超过这段时间后，会话将被视为无效。

如果我们配置我们的项目使用 Tomcat，我们必须记住，它只支持分钟精度的会话超时，最少一分钟。这意味着如果我们指定一个超时值`170s`，例如，它将导致两分钟的超时。

最后，值得一提的是，尽管 [Spring Session](/web/20220625233613/https://www.baeldung.com/spring-session) 为此目的支持一个类似的属性(`spring.session.timeout`)，如果没有指定，自动配置将退回到我们首先提到的属性值。

## 6。防止使用 URL 参数进行会话跟踪

在 URL 中暴露会话信息是一个不断增长的[安全风险](https://web.archive.org/web/20220625233613/https://www.sitelock.com/blog/owasp-top-10-broken-authentication-session-management/ "Top 10 2013-A2-Broken Authentication and Session Management")(从 2007 年的第七位上升到 2013 年 OWASP 十大名单中的第二位)。

[从 Spring 3.0](https://web.archive.org/web/20220625233613/https://jira.springsource.org/browse/SEC-1052 "JIRA - Add option to prevent URL rewriting of jsessionid") 开始，将`jsessionid`附加到 URL 的 URL 重写逻辑现在可以通过在`<http>`名称空间中设置`disable-url-rewriting=”true”`来禁用。

或者，从 Servlet 3.0 开始，会话跟踪机制也可以在`web.xml`中配置:

```java
<session-config>
     <tracking-mode>COOKIE</tracking-mode>
</session-config>
```

从程序上来说:

```java
servletContext.setSessionTrackingModes(EnumSet.of(SessionTrackingMode.COOKIE));
```

这将选择在哪里存储`JSESSIONID`——在 cookie 中还是在 URL 参数中。

## 7。具有 Spring 安全的会话固定保护

该框架通过配置当用户尝试再次进行身份验证时现有会话会发生什么来提供针对典型会话固定攻击的保护:

```java
<session-management session-fixation-protection="migrateSession"> ...
```

下面是相应的 Java 配置:

```java
http.sessionManagement()
  .sessionFixation().migrateSession()
```

默认情况下，Spring Security 启用了这种保护(“`migrateSession`”)。身份验证时，会创建一个新的 HTTP 会话，旧的会话会失效，旧会话的属性会被复制过来。

如果这不是我们想要的，还有两种选择:

*   当设置了“`none`”时，原始会话不会失效。
*   当设置了“`newSession`”时，将创建一个干净的会话，而不会复制旧会话的任何属性。

## 8.**安全会话 Cookie**

接下来，我们将讨论如何保护我们的会话 cookie。

**我们可以使用`httpOnly`和`secure`标志来保护我们的会话 cookie** :

*   `httpOnly`:如果为真，则浏览器脚本将无法访问 cookie
*   如果为真，那么 cookie 将只通过 HTTPS 连接发送

我们可以在`web.xml`中为我们的会话 cookie 设置这些标志:

```java
<session-config>
    <session-timeout>1</session-timeout>
    <cookie-config>
        <http-only>true</http-only>
        <secure>true</secure>
    </cookie-config>
</session-config>
```

这个配置选项从 Java servlet 3 开始可用。默认情况下，`http-only`为真，`secure`为假。

让我们也来看看相应的 Java 配置:

```java
public class MainWebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext sc) throws ServletException {
        // ...
        sc.getSessionCookieConfig().setHttpOnly(true);        
        sc.getSessionCookieConfig().setSecure(true);        
    }
}
```

**如果我们使用 Spring Boot，我们可以在我们的`application.properties`** 中设置这些标志:

```java
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true
```

最后，我们也可以通过使用`Filter`手动实现这一点:

```java
public class SessionFilter implements Filter {
    @Override
    public void doFilter(
      ServletRequest request, ServletResponse response, FilterChain chain)
      throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        Cookie[] allCookies = req.getCookies();
        if (allCookies != null) {
            Cookie session = 
              Arrays.stream(allCookies).filter(x -> x.getName().equals("JSESSIONID"))
                    .findFirst().orElse(null);

            if (session != null) {
                session.setHttpOnly(true);
                session.setSecure(true);
                res.addCookie(session);
            }
        }
        chain.doFilter(req, res);
    }
}
```

## 9。使用会话

### 9.1。会话范围的 bean

只需在 web 上下文中声明的 bean 上使用@Scope 注释，就可以用`session`范围定义 bean:

```java
@Component
@Scope("session")
public class Foo { .. }
```

或者使用 XML:

```java
<bean id="foo" scope="session"/>
```

然后可以将该 bean 注入到另一个 bean 中:

```java
@Autowired
private Foo theFoo;
```

Spring 会将新的 bean 绑定到 HTTP 会话的生命周期中。

### 9.2。将原始会话注入控制器

原始 HTTP 会话也可以直接注入到`Controller`方法中:

```java
@RequestMapping(..)
public void fooMethod(HttpSession session) {
    session.setAttribute(Constants.FOO, new Foo());
    //...
    Foo foo = (Foo) session.getAttribute(Constants.FOO);
}
```

### 9.3。获取原始会话

当前的 HTTP 会话也可以通过**原始 Servlet API** 以编程方式获得:

```java
ServletRequestAttributes attr = (ServletRequestAttributes) 
    RequestContextHolder.currentRequestAttributes();
HttpSession session= attr.getRequest().getSession(true); // true == allow create
```

## 10。结论

在本文中，我们讨论了使用 Spring Security 管理会话。

另外，Spring 参考包含了一个非常好的关于会话管理的 FAQ。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220625233613/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc)