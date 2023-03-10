# Spring 安全摘要认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-digest-authentication>

## 1。概述

本教程展示了如何使用 Spring 设置、配置和定制摘要式认证。类似于前一篇涉及基本认证的[文章](/web/20220625074556/https://www.baeldung.com/spring-security-basic-authentication "Basic Authentication Tutorial")，我们将构建在 Spring MVC 教程之上，并使用 Spring Security 提供的摘要认证机制来保护应用程序。

## 2。安全 XML 配置

关于配置，首先要理解的是，虽然 Spring Security 对摘要认证机制有完整的开箱即用支持，但这种支持并没有像基本认证那样很好地集成到名称空间中。

在这种情况下，我们需要手动**定义将要组成安全配置的原始 bean**—`DigestAuthenticationFilter`和`DigestAuthenticationEntryPoint`:

```java
<beans:bean id="digestFilter" 
  class="org.springframework.security.web.authentication.www.DigestAuthenticationFilter">
    <beans:property name="userDetailsService" ref="userService" />
    <beans:property name="authenticationEntryPoint" ref="digestEntryPoint" />
</beans:bean>
<beans:bean id="digestEntryPoint" 
  class="org.springframework.security.web.authentication.www.DigestAuthenticationEntryPoint">
    <beans:property name="realmName" value="Contacts Realm via Digest Authentication" />
    <beans:property name="key" value="acegi" />
</beans:bean>

<!-- the security namespace configuration -->
<http use-expressions="true" entry-point-ref="digestEntryPoint">
    <intercept-url pattern="/**" access="isAuthenticated()" />

    <custom-filter ref="digestFilter" after="BASIC_AUTH_FILTER" />
</http>

<authentication-manager>
    <authentication-provider>
        <user-service id="userService">
            <user name="user1" password="user1Pass" authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

接下来，我们需要将这些 beans 集成到整体安全配置中——在这种情况下，名称空间仍然足够灵活，允许我们这样做。

第一部分是通过主`<http>`元素的`entry-point-ref`属性指向定制入口点 bean。

第二部分是**将新定义的摘要过滤器添加到安全过滤器链中**。由于这个过滤器在功能上等同于`BasicAuthenticationFilter`，我们在链中使用相同的相对位置——这由整个 [Spring 安全标准过滤器](https://web.archive.org/web/20220625074556/http://static.springsource.org/spring-security/site/docs/3.1.x/reference/ns-config.html#ns-custom-filters "The Security Filters and aliases")中的`BASIC_AUTH_FILTER`别名指定。

最后，注意摘要过滤器被配置为**指向用户服务 bean**——这里，名称空间再次非常有用，因为它允许我们为由`<user-service>`元素创建的默认用户服务指定一个 bean 名称:

```java
<user-service id="userService">
```

## 3。消费安全应用程序

我们将使用**`curl`命令**来使用安全应用程序，并了解客户端如何与之交互。

让我们从请求主页–**开始，在请求中不提供安全凭证**:

```java
curl -i http://localhost/spring-security-mvc-digest-auth/homepage.html
```

正如所料，我们得到了一个带有`401 Unauthorized`状态代码的响应:

```java
HTTP/1.1 401 Unauthorized
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=CF0233C...; Path=/spring-security-mvc-digest-auth/; HttpOnly
WWW-Authenticate: Digest realm="Contacts Realm via Digest Authentication", qop="auth", 
  nonce="MTM3MzYzODE2NTg3OTo3MmYxN2JkOWYxZTc4MzdmMzBiN2Q0YmY0ZTU0N2RkZg=="
Content-Type: text/html;charset=utf-8
Content-Length: 1061
Date: Fri, 12 Jul 2013 14:04:25 GMT
```

如果这个请求是由浏览器发送的，身份验证质询将使用一个简单的用户/密码对话框提示用户输入凭据。

现在让我们**提供正确的凭证**并再次发送请求:

```java
curl -i --digest --user 
   user1:user1Pass http://localhost/spring-security-mvc-digest-auth/homepage.html
```

注意，我们正在通过`–digest`标志为`curl`命令启用摘要认证。

来自服务器的第一个响应将是相同的——`401 Unauthorized`,但是现在将由第二个请求解释和处理该挑战——第二个请求将通过`200 OK`成功:

```java
HTTP/1.1 401 Unauthorized
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=A961E0D...; Path=/spring-security-mvc-digest-auth/; HttpOnly
WWW-Authenticate: Digest realm="Contacts Realm via Digest Authentication", qop="auth", 
  nonce="MTM3MzYzODgyOTczMTo3YjM4OWQzMGU0YTgwZDg0YmYwZjRlZWJjMDQzZWZkOA=="
Content-Type: text/html;charset=utf-8
Content-Length: 1061
Date: Fri, 12 Jul 2013 14:15:29 GMT

HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=55F996B...; Path=/spring-security-mvc-digest-auth/; HttpOnly
Content-Type: text/html;charset=ISO-8859-1
Content-Language: en-US
Content-Length: 90
Date: Fri, 12 Jul 2013 14:15:29 GMT

<html>
<head></head>

<body>
	<h1>This is the homepage</h1>
</body>
</html>
```

关于这种交互的最后一点是，客户端可以在第一个请求中**抢先发送正确的`Authorization`报头**，从而完全**避免服务器安全挑战**和第二个请求。

## 4。美芬依赖

安全依赖性在[春季安全专家教程](/web/20220625074556/https://www.baeldung.com/spring-security-with-maven "Spring Security pom")中有深入的讨论。简而言之，我们需要将`spring-security-web`和`spring-security-config`定义为`pom.xml`中的依赖项。

## 5。结论

在本教程中，我们通过利用框架中的摘要认证支持，将安全性引入到一个简单的 Spring MVC 项目中。

这些例子的实现可以在 Github 项目中找到——这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。

当项目在本地运行时，可以在以下位置访问主页 html(或者，对于最小的 Tomcat 配置，可以在端口 80 上访问):

[http://localhost:8080/spring-security-MVC-digest-auth/home page . html](https://web.archive.org/web/20220625074556/http://localhost:8080/spring-security-mvc-digest-auth/homepage.html "Spring Security Digest Auth app")

最后，应用程序没有理由需要[在基本身份验证和摘要式身份验证之间做出选择](/web/20220625074556/https://www.baeldung.com/basic-and-digest-authentication-for-a-rest-api-with-spring-security "Basic and Digest Authentication for a REST Service with Spring Security")–**两者都可以在同一个 URI 结构上同时设置**，这样客户端在使用 web 应用程序时就可以在两种机制之间做出选择。