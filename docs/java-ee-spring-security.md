# 用 Spring Security 保护雅加达 EE

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee-spring-security>

 ![](img/30485ee3e93b9021a0233efb2ce8c7ba.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220523231103/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在这个快速教程中，我们将看看如何用 Spring Security 来保护 Jakarta EE web 应用程序。

## 2。 **美芬依存**

让我们从本教程所需的 [Spring 安全依赖项](/web/20220523231103/https://www.baeldung.com/spring-security-with-maven)开始`:`

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>4.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>4.2.3.RELEASE</version>
</dependency>
```

最新的 Spring 安全版本(撰写本教程时)是 4 . 2 . 3 . release；像往常一样，我们可以查看 Maven Central 的最新版本。

## 3。 **安全配置**

接下来，我们需要为现有的 Jakarta EE 应用程序设置安全配置:

```java
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig 
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth)
      throws Exception {
        auth.inMemoryAuthentication()
          .withUser("user1").password("user1Pass").roles("USER")
          .and()
          .withUser("admin").password("adminPass").roles("ADMIN");
    }
}
```

在`configure()`方法中，我们设置了`AuthenticationManager`。为了简单起见，我们实现了一个简单的内存认证。用户详细信息是硬编码的。

这是为了在不需要完整的持久性机制时用于快速原型开发。

接下来，让我们通过添加`SecurityWebApplicationInitializer` 类将安全性集成到现有系统中:

```java
public class SecurityWebApplicationInitializer
  extends AbstractSecurityWebApplicationInitializer {

    public SecurityWebApplicationInitializer() {
        super(SpringSecurityConfig.class);
    }
}
```

这个类将确保在应用程序启动时加载`SpringSecurityConfig`。在这个阶段，**我们已经基本实现了 Spring Security** 。通过这种实现，Spring Security 将默认要求对所有请求和路由进行身份验证。

## 4。 **配置安全规则**

我们可以通过重写`WebSecurityConfigurerAdapter`的`configure(HttpSecurity http)`方法来进一步定制 Spring 安全性:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
      .csrf().disable()
      .authorizeRequests()
      .antMatchers("/auth/login*").anonymous()
      .anyRequest().authenticated()
      .and()
      .formLogin()
      .loginPage("/auth/login")
      .defaultSuccessUrl("/home", true)
      .failureUrl("/auth/login?error=true")
      .and()
      .logout().logoutSuccessUrl("/auth/login");
}
```

使用`antMatchers()` 方法，我们配置 Spring Security 允许匿名访问`/auth/login` 和验证任何其他请求。

### 4.1。自定义登录页面

使用`formLogin()`方法配置自定义登录页面:

```java
http.formLogin()
  .loginPage("/auth/login")
```

如果没有指定，Spring Security 会在`/login`生成一个默认的登录页面:

```java
<html>
<head></head>
<body>
<h1>Login</h1>
<form name='f' action="/auth/login" method='POST'>
    <table>
        <tr>
            <td>User:</td>
            <td><input type='text' name='username' value=''></td>
        </tr>
        <tr>
            <td>Password:</td>
            <td><input type='password' name='password'/></td>
        </tr>
        <tr>
            <td><input name="submit" type="submit" 
              value="submit"/></td>
        </tr>
    </table>
</form>
</body>
</html>
```

### 4.2。自定义登陆页面

成功登录后，Spring Security 会将用户重定向到应用程序的根目录。我们可以通过指定一个默认的成功 URL 来覆盖它:

```java
http.formLogin()
  .defaultSuccessUrl("/home", true)
```

通过将`defaultSuccessUrl()`方法的`alwaysUse`参数设置为 true，用户将总是被重定向到指定的页面。

如果没有设置`alwaysUse`参数或者将其设置为 false，那么在提示用户进行身份验证之前，用户将被重定向到他试图访问的上一个页面。

同样，我们也可以指定一个定制的失败登录页面:

```java
http.formLogin()
  .failureUrl("/auth/login?error=true")
```

### 4.3。授权

我们可以按角色限制对资源的访问:

```java
http.formLogin()
  .antMatchers("/home/admin*").hasRole("ADMIN")
```

如果非管理员用户试图访问`/home/admin`端点，他/她将收到拒绝访问错误。

我们还可以基于用户的角色来限制 JSP 页面上的数据。这是使用`<security:authorize>`标签完成的:

```java
<security:authorize access="hasRole('ADMIN')">
    This text is only visible to an admin
    <br/>
    <a href="<c:url value="/home/admin" />">Admin Page</a>
    <br/>
</security:authorize>
```

要使用这个标签，我们必须在页面顶部包含 Spring 安全标签 taglib:

```java
<%@ taglib prefix="security" 
  uri="http://www.springframework.org/security/tags" %>
```

## 5.Spring 安全 XML 配置

到目前为止，我们已经了解了如何在 Java 中配置 Spring 安全性。让我们来看一个等价的 XML 配置。

首先，我们需要在包含 XML 配置的`web/WEB-INF/spring`文件夹中创建一个`security.xml`文件。本文末尾提供了这样一个`security.xml`配置文件的例子。

让我们从配置身份验证管理器和身份验证提供者开始。为了简单起见，我们使用简单的硬编码用户凭证:

```java
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user" 
              password="user123" 
              authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

我们刚才所做的是创建一个具有用户名、密码和角色的用户。

或者，我们可以用密码编码器配置我们的身份验证提供程序:

```java
<authentication-manager>
    <authentication-provider>
        <password-encoder hash="sha"/>
        <user-service>
            <user name="user"
              password="d7e6351eaa13189a5a3641bab846c8e8c69ba39f" 
              authorities="ROLE_USER" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

我们还可以指定 Spring 的`UserDetailsService`或`Datasource`的定制实现作为我们的认证提供者。更多细节可以在[这里找到。](https://web.archive.org/web/20220523231103/https://docs.spring.io/spring-security/site/docs/3.0.x/reference/ns-config.html)

现在我们已经配置了身份验证管理器，让我们设置安全规则并应用访问控制:

```java
<http auto-config='true' use-expressions="true">
    <form-login default-target-url="/secure.jsp" />
    <intercept-url pattern="/" access="isAnonymous()" />
    <intercept-url pattern="/index.jsp" access="isAnonymous()" />
    <intercept-url pattern="/secure.jsp" access="hasRole('ROLE_USER')" />
</http>
```

在上面的代码片段中，我们已经将`HttpSecurity` 配置为使用表单登录，并将`/secure.jsp`设置为登录成功 URL。我们允许匿名访问`/index.jsp`和`“/”`路径。此外，我们指定对`/secure.jsp`的访问应该需要认证，并且经过认证的用户应该至少拥有`ROLE_USER`级别的权限。

将`http`标签的`auto-config`属性设置为`true`会指示 Spring Security 实现默认行为，我们不必在配置中覆盖这些行为。因此，`/login`和`/logout`将分别用于用户登录和注销。还提供了默认的登录页面。

我们可以用定制的登录和注销页面、URL 来进一步定制`form-login`标签，以处理认证失败和成功。[安全名称空间附录](https://web.archive.org/web/20220523231103/https://docs.spring.io/spring-security/site/docs/3.0.x/reference/appendix-namespace.html)列出了`form-login` (和其他)标签所有可能的属性。一些 ide 也可以通过在按下`ctrl`键的同时点击标签来进行检查。

最后，对于要在应用程序启动期间加载的`security.xml`配置，我们需要向我们的`web.xml`添加以下定义:

```java
<context-param>                                                                           
    <param-name>contextConfigLocation</param-name>                                        
    <param-value>                                                                         
      /WEB-INF/spring/*.xml                                                             
    </param-value>                                                                        
</context-param>                                                                          

<filter>                                                                                  
    <filter-name>springSecurityFilterChain</filter-name>                                  
    <filter-class>
      org.springframework.web.filter.DelegatingFilterProxy</filter-class>     
</filter>                                                                                 

<filter-mapping>                                                                          
    <filter-name>springSecurityFilterChain</filter-name>                                  
    <url-pattern>/*</url-pattern>                                                         
</filter-mapping>                                                                         

<listener>                                                                                
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

请注意，试图在同一个 JEE 应用程序中同时使用基于 XML 和 Java 的配置可能会导致错误。

## 6。结论

在本文中，我们看到了如何使用 Spring Security 保护 Jakarta EE 应用程序，并演示了基于 Java 和基于 XML 的配置。

我们还讨论了基于用户角色授予或撤销对特定资源的访问权的方法。

完整的源代码和 XML 定义可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220523231103/https://github.com/eugenp/tutorials/tree/master/jee-7-security)