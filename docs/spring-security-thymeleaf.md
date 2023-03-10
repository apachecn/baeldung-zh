# 百里香叶的春天安全

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-thymeleaf>

## 1。概述

在这个快速教程中，我们将关注百里香的春季安全性。我们将创建一个 Spring Boot 应用程序，演示安全术语的用法。

我们对前端技术的选择是[thyme leaf](https://web.archive.org/web/20221208143914/http://www.thymeleaf.org/index.html)——一个现代的服务器端 web 模板引擎，与 Spring MVC 框架有很好的集成。更多详情，请看我们的[介绍文章](/web/20221208143914/https://www.baeldung.com/thymeleaf-in-spring-mvc)吧。

最后，Spring Security Dialect 是一个百里香附加模块，它自然有助于将这两者集成在一起。

我们将使用我们在 [Spring Boot 教程](/web/20221208143914/https://www.baeldung.com/spring-boot-start)文章中构建的简单项目；我们也有一个[百里香叶教程与春天](/web/20221208143914/https://www.baeldung.com/thymeleaf-in-spring-mvc)，在那里可以找到标准的百里香叶配置。

## 2。依赖性

首先，让我们将新的依赖项添加到我们的 Maven `pom.xml`:

```java
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```

建议始终使用最新版本——我们可以在 [Maven Central](https://web.archive.org/web/20221208143914/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22thymeleaf-extras-springsecurity5%22) 上获得。

## 3。Spring 安全配置

接下来，让我们定义 Spring 安全性的配置。

我们还需要至少两个不同的用户来演示安全方言的用法:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {

    // [...] 
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) 
      throws Exception {
        auth
          .inMemoryAuthentication()
          .withUser("user").password(passwordEncoder().encode("password")).roles("USER")
          .and()
          .withUser("admin").password(passwordEncoder().encode("admin")).roles("ADMIN");
    }

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

正如我们所见，在`configureGlobal(AuthenticationManagerBuilder auth)`中，我们用用户名和密码定义了两个用户。我们可以使用这些来访问我们的应用程序。

我们的用户分别有不同的角色:`ADMIN`和`USER`，这样我们就可以根据角色向他们呈现特定的内容。

## 4。安全方言

**Spring 安全方言允许我们根据用户角色、权限或其他安全表达式有条件地显示内容。**它也让我们可以访问弹簧`Authentication`对象。

让我们看一下索引页，其中包含安全术语的示例:

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Welcome to Spring Security Thymeleaf tutorial</title>
    </head>
    <body>
        <h2>Welcome</h2>
        <p>Spring Security Thymeleaf tutorial</p>
        <div sec:authorize="hasRole('USER')">Text visible to user.</div>
        <div sec:authorize="hasRole('ADMIN')">Text visible to admin.</div>
        <div sec:authorize="isAuthenticated()">
            Text visible only to authenticated users.
        </div>
        Authenticated username:
        <div sec:authentication="name"></div>
        Authenticated user roles:
        <div sec:authentication="principal.authorities"></div>
    </body>
</html>
```

我们可以看到 Spring 安全方言特有的属性:`sec:authorize` 和`sec:authentication`。

这些我们来讨论一下，一个一个来。

### 4.1.理解`sec:authorize`

**简单来说，我们使用`sec:authorize` 属性来控制显示的内容。**

例如，如果我们只想向角色为用户的用户显示内容，我们可以:`<div sec:authorize=”hasRole(‘USER')”>.`

而且，如果我们想扩大对所有经过身份验证的用户的访问，我们可以使用下面的表达式:

`<div sec:authorize=”isAuthenticated()”>.`

### 4.2.理解`sec:authentication`

Spring Security `[Authentication](https://web.archive.org/web/20221208143914/https://docs.spring.io/spring-security/site/docs/5.0.3.RELEASE/api/org/springframework/security/core/Authentication.html)` 接口公开了关于认证主体或认证请求的有用方法。

为了让**访问百里香**内的认证对象，我们可以简单地使用`<div sec:authentication=”name”>` 或 `<div sec:authentication=”principal.authorities”>.`

前者允许我们访问经过身份验证的用户的名称，后者允许我们访问经过身份验证的用户的角色。

## 5。总结

在本文中，我们在一个简单的 Spring Boot 应用程序中使用了百里香中的 Spring 安全支持。

和往常一样，本文中显示的代码的工作版本可以在我们的 [GitHub 库](https://web.archive.org/web/20221208143914/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-thymeleaf)中获得。