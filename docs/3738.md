# spring Security–自定义 403 禁止/拒绝访问页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-access-denied-page>

## 1。简介

在本文中，我们将展示如何在 Spring 安全项目中**定制拒绝访问页面。**

这可以通过 Spring 安全配置或`web.xml`文件中的 web 应用程序配置来实现。

在接下来的部分中，我们将更深入地了解这些选项。

## 2。自定义 JSP

每当用户试图访问被限制为他们没有的角色的页面时，应用程序将返回状态代码 403，这意味着`Access Denied`。

为了用自定义页面替换 Spring 403 状态响应页面，**让我们首先创建一个名为`accessDenied.jsp` :** 的`JSP`文件

```
<body>
<h2>Sorry, you do not have permission to view this page.</h2>

Click <a href="<c:url value="/homepage.html" /> ">here</a>
to go back to the Homepage.
</body>
```

## 3。Spring 安全配置

默认情况下，Spring Security 定义了一个`ExceptionTranslationFilter`，用于处理`AuthenticationException`和`AccessDeniedException`类型的异常。后者是通过一个名为`accessDeniedHandler,`的属性完成的，该属性使用了`AccessDeniedHandlerImpl`类。

为了定制这个行为来使用我们在上面创建的页面，我们需要覆盖`ExceptionTranslationFilter`类的属性。这可以通过 Java 配置或 XML 配置来完成。

### 3.1。拒绝访问页面

使用 Java，**我们可以在配置`HttpSecurity`元素时，通过使用 `accessDeniedPage()`或`accessDeniedHandler()`方法**来定制 403 错误处理过程。

让我们创建一个身份验证配置，将`“/admin/**`URL 限制为`ADMIN`角色，并将拒绝访问页面设置为我们的自定义`accessDenied.jsp`页面:

```
@Override
protected void configure(final HttpSecurity http) throws Exception {
    http
      // ...
      .and()
      .exceptionHandling().accessDeniedPage("/accessDenied.jsp");
}
```

让我们看看拒绝访问页面的等效 XML 配置:

```
<http use-expressions="true">
    <access-denied-handler error-page="/accessDenied"/>
 </http>
```

### 3.2。拒绝访问处理程序

使用拒绝访问处理程序代替页面的优点是，我们可以在重定向到 403 页面之前定义要执行的自定义逻辑。为此，**我们需要创建一个实现`AccessDeniedHandler`接口**并覆盖`handle()`方法的类。

让我们创建一个定制的`AccessDeniedHandler`类，它为每个拒绝访问的尝试记录一条警告消息，其中包含进行尝试的用户和他们试图访问的受保护的 URL:

```
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    public static final Logger LOG
      = Logger.getLogger(CustomAccessDeniedHandler.class);

    @Override
    public void handle(
      HttpServletRequest request,
      HttpServletResponse response, 
      AccessDeniedException exc) throws IOException, ServletException {

        Authentication auth 
          = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null) {
            LOG.warn("User: " + auth.getName() 
              + " attempted to access the protected URL: "
              + request.getRequestURI());
        }

        response.sendRedirect(request.getContextPath() + "/accessDenied");
    }
}
```

在安全配置中，**我们将定义 bean 并设置自定义的`AccessDeniedHandler` :**

```
@Bean
public AccessDeniedHandler accessDeniedHandler(){
    return new CustomAccessDeniedHandler();
}

//...
.exceptionHandling().accessDeniedHandler(accessDeniedHandler()); 
```

如果我们想使用 XML 配置上面定义的`CustomAccessDeniedHandler`类，配置看起来会略有不同:

```
<bean name="customAccessDeniedHandler" 
  class="com.baeldung.security.CustomAccessDeniedHandler" />

<http use-expressions="true">
    <access-denied-handler ref="customAccessDeniedHandler"/>
</http>
```

## 4。应用程序配置

**通过定义一个`error-page`标签，可以通过 web 应用程序的`web.xml`文件来处理拒绝访问错误。**它包含两个子标记`error-code,`，指定要拦截的状态代码，以及`location,`，表示遇到错误代码时用户将被重定向到的 URL:

```
<error-page>
    <error-code>403</error-code>
    <location>/accessDenied</location>
</error-page>
```

如果一个应用程序没有一个`web.xml`文件，就像 Spring Boot 的情况一样，Spring 注解目前并没有提供一个准确的`error-page`标签的替代。根据 Spring 文档，在这种情况下，推荐的方法是使用第 3 节中介绍的方法`accessDeniedPage()`和`accessDeniedHandler()`。

## 5。结论

在这篇简短的文章中，我们详细介绍了使用自定义 403 页面处理拒绝访问错误的各种方法。

文章的完整源代码可以在 [GitHub 项目](https://web.archive.org/web/20220122052741/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-login)中找到。