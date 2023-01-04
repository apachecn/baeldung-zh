# Spring 安全表达式简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-expressions>

## 1。简介

在本教程中，我们将关注 Spring 安全表达式，以及使用这些表达式的实际例子。

在研究更复杂的实现(如 ACL)之前，对安全表达式有一个坚实的理解是很重要的，因为如果使用正确，它们会非常灵活和强大。

## 2。Maven 依赖关系

为了使用 Spring Security，我们需要在我们的`pom.xml`文件中包含以下部分:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.6.0</version>
   </dependency>
</dependencies>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220926185655/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-security-web%22)

请注意，这种依赖只涉及 Spring 安全性；对于一个完整的 web 应用程序，我们需要添加 s `pring-core`和`spring-context`。

## 3。配置

首先，我们来看一个 Java 配置。

我们将扩展`WebSecurityConfigurerAdapter`,这样我们就可以选择挂接基类提供的任何扩展点:

```java
@Configuration
@EnableAutoConfiguration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityJavaConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

当然，我们也可以进行 XML 配置:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans ...>
    <global-method-security pre-post-annotations="enabled"/>
</beans:beans>
```

## 4。网络安全表达式

现在让我们来研究一下安全表达式:

*   `hasRole`，`hasAnyRole`
*   `hasAuthority`，`hasAnyAuthority`
*   `permitAll`，`denyAll`
*   `isAnonymous`、`isRememberMe`、`isAuthenticated`、`isFullyAuthenticated`
*   `principal`，`authentication`
*   `hasPermission`

### 4.1。`hasRole, hasAnyRole`

这些表达式负责在我们的应用程序中定义对特定 URL 和方法的访问控制或授权:

```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/auth/admin/*").hasRole("ADMIN")
    .antMatchers("/auth/*").hasAnyRole("ADMIN","USER")
    ...
} 
```

在上面的例子中，我们指定了对所有以`/auth/,`开头的链接的访问，将它们限制为使用角色`USER`或角色`ADMIN.`登录的用户。此外，要访问以`/auth/admin/,` 开头的链接，我们需要在系统中有一个`ADMIN`角色。

我们可以通过编写以下代码在 XML 文件中实现相同的配置:

```java
<http>
    <intercept-url pattern="/auth/admin/*" access="hasRole('ADMIN')"/>
    <intercept-url pattern="/auth/*" access="hasAnyRole('ADMIN','USER')"/>
</http> 
```

### 4.2。`hasAuthority, hasAnyAuthority`

春天的角色和权限都差不多。

主要区别在于角色有特殊的语义。从 Spring Security 4 开始，任何与角色相关的方法都会自动添加前缀'`ROLE_`'(如果还没有的话)。

所以`hasAuthority(‘ROLE_ADMIN')`类似于`hasRole(‘ADMIN')`,因为前缀`ROLE_`是自动添加的。

**使用授权的好处是我们根本不必使用`ROLE_`前缀。**

下面是一个定义具有特定权限的用户的简单示例:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
      .withUser("user1").password(encoder().encode("user1Pass"))
      .authorities("USER")
      .and().withUser("admin").password(encoder().encode("adminPass"))
      .authorities("ADMIN");
} 
```

然后我们可以使用这些权威表达:

```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/auth/admin/*").hasAuthority("ADMIN")
    .antMatchers("/auth/*").hasAnyAuthority("ADMIN", "USER")
    ...
}
```

正如我们所看到的，我们在这里根本没有提到角色。

另外，从 Spring 5 开始，我们需要一个 [`PasswordEncoder`](/web/20220926185655/https://www.baeldung.com/spring-security-5-default-password-encoder) bean:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

最后，我们还可以选择使用 XML 配置来实现相同的功能:

```java
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="user1Pass" authorities="ROLE_USER"/>
            <user name="admin" password="adminPass" authorities="ROLE_ADMIN"/>
        </user-service>
    </authentication-provider>
</authentication-manager>
<bean name="passwordEncoder" 
  class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
```

并且:

```java
<http>
    <intercept-url pattern="/auth/admin/*" access="hasAuthority('ADMIN')"/>
    <intercept-url pattern="/auth/*" access="hasAnyAuthority('ADMIN','USER')"/>
</http>
```

### 4.3。`permitAll, denyAll`

这两个注释也很简单。我们可以允许或拒绝访问我们服务中的某些 URL。

让我们来看看这个例子:

```java
...
.antMatchers("/*").permitAll()
...
```

使用这个配置，我们将授权所有用户(包括匿名用户和登录用户)访问以“/”开头的页面(例如，我们的主页)。

我们还可以拒绝访问我们的整个 URL 空间:

```java
...
.antMatchers("/*").denyAll()
...
```

同样，我们也可以对 XML 进行同样的配置:

```java
<http auto-config="true" use-expressions="true">
    <intercept-url access="permitAll" pattern="/*" /> <!-- Choose only one -->
    <intercept-url access="denyAll" pattern="/*" /> <!-- Choose only one -->
</http>
```

### 4.4。 `isAnonymous, isRememberMe, isAuthenticated, isFullyAuthenticated`

在这一小节中，我们将关注与用户登录状态相关的表达式。让我们从一个没有登录到我们页面的用户开始。通过在 Java 配置中指定以下内容，我们将允许所有未经授权的用户访问我们的主页:

```java
...
.antMatchers("/*").anonymous()
...
```

在 XML 配置中也是如此:

```java
<http>
    <intercept-url pattern="/*" access="isAnonymous()"/>
</http>
```

如果我们想保护网站的安全，以便每个使用它的人都需要登录，我们需要使用`isAuthenticated()` 方法:

```java
...
.antMatchers("/*").authenticated()
...
```

或者我们可以使用 XML 版本:

```java
<http>
    <intercept-url pattern="/*" access="isAuthenticated()"/>
</http>
```

我们还有两个额外的表达式，`isRememberMe()`和`isFullyAuthenticated()`。通过使用 cookies，Spring 实现了“记住我”的功能，所以不需要每次都登录系统。我们可以在这里阅读[更多关于`Remember Me`的内容。](/web/20220926185655/https://www.baeldung.com/spring-security-remember-me)

为了向通过“记住我”功能登录的用户提供访问权限，我们可以使用:

```java
...
.antMatchers("/*").rememberMe()
...
```

我们也可以使用 XML 版本:

```java
<http>
    <intercept-url pattern="*" access="isRememberMe()"/>
</http>
```

最后，我们服务的某些部分要求用户再次被认证，即使用户已经登录。例如，假设用户想要更改设置或支付信息；在系统的更敏感区域要求手动身份验证是一个好的做法。

为此，我们可以指定`isFullyAuthenticated()`，如果用户不是匿名用户或“记住我”用户，则返回`true`:

```java
...
.antMatchers("/*").fullyAuthenticated()
...
```

下面是 XML 版本:

```java
<http>
    <intercept-url pattern="*" access="isFullyAuthenticated()"/>
</http>
```

### 4.5。`principal, authentication`

这些表达式允许我们分别从`SecurityContext`访问代表当前授权(或匿名)用户的`principal`对象和当前的`Authentication`对象。

例如，我们可以使用`principal` 来加载用户的电子邮件、头像或任何其他可以从登录用户那里获得的数据。

并且`authentication` 提供关于完整的`Authentication`对象的信息，以及它被授予的权限。

这两个表达式在文章[在 Spring Security](/web/20220926185655/https://www.baeldung.com/get-user-in-spring-security) 中检索用户信息中有更详细的描述。

### 4.6。`hasPermission` 原料药

这个表达式被[记录为](https://web.archive.org/web/20220926185655/https://docs.spring.io/spring-security/site/docs/5.1.5.RELEASE/reference/html/authorization.html)，它旨在成为表达式系统和 Spring Security 的 ACL 系统之间的桥梁，允许我们基于抽象权限在单个域对象上指定授权约束。

让我们看一个例子。想象一下，我们有一个允许合作写文章的服务，有一个主编辑决定作者提出的哪些文章应该发表。

为了允许使用这样的服务，我们可以用访问控制方法创建以下方法:

```java
@PreAuthorize("hasPermission(#articleId, 'isEditor')")
public void acceptArticle(Article article) {
   …
}
```

只有授权用户才能调用这个方法，他们需要在服务中拥有`isEditor` 权限。

我们还需要记住在我们的应用程序上下文中显式配置一个`PermissionEvaluator`，其中`customInterfaceImplementation`将是实现`PermissionEvaluator`的类:

```java
<global-method-security pre-post-annotations="enabled">
    <expression-handler ref="expressionHandler"/>
</global-method-security>

<bean id="expressionHandler"
    class="org.springframework.security.access.expression
      .method.DefaultMethodSecurityExpressionHandler">
    <property name="permissionEvaluator" ref="customInterfaceImplementation"/>
</bean>
```

当然，我们也可以用 Java 配置做到这一点:

```java
@Override
protected MethodSecurityExpressionHandler expressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler = 
      new DefaultMethodSecurityExpressionHandler();
    expressionHandler.setPermissionEvaluator(new CustomInterfaceImplementation());
    return expressionHandler;
}
```

## 5。结论

本文是对 Spring 安全表达式的全面介绍和指导。

这里讨论的所有例子都可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20220926185655/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest "Spring Security REST Tutorial")