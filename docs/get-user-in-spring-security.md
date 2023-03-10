# 在 Spring Security 中检索用户信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/get-user-in-spring-security>

## 1。概述

本教程将展示如何在 Spring Security 中检索用户详细信息。

在 Spring 中，当前经过身份验证的用户可以通过许多不同的机制获得。让我们先来看看最常见的解决方案——编程访问。

## 延伸阅读:

## [使用 Spring Security 跟踪登录的用户](/web/20220707143845/https://www.baeldung.com/spring-security-track-logged-in-users)

A quick guide to track logged in users in an application built using Spring Security.[Read more](/web/20220707143845/https://www.baeldung.com/spring-security-track-logged-in-users) →

## [Spring Security–角色和权限](/web/20220707143845/https://www.baeldung.com/role-and-privilege-for-spring-security-registration)

How to map Roles and Privileges for a Spring Security application: the setup, the authentication and the registration process.[Read more](/web/20220707143845/https://www.baeldung.com/role-and-privilege-for-spring-security-registration) →

## [Spring Security–重置您的密码](/web/20220707143845/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)

Every app should enable users to change their own password in case they forget it.[Read more](/web/20220707143845/https://www.baeldung.com/spring-security-registration-i-forgot-my-password) →

## 2。获取 Bean 中的用户

**检索当前已认证主体的最简单方法是通过静态调用`SecurityContextHolder`** :

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
String currentPrincipalName = authentication.getName();
```

对该代码片段的改进是，在尝试访问之前，首先检查是否有经过身份验证的用户:

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
if (!(authentication instanceof AnonymousAuthenticationToken)) {
    String currentUserName = authentication.getName();
    return currentUserName;
}
```

当然，像这样进行静态调用也有不利的一面，降低代码的可测试性是最明显的一点。相反，我们将探索这个非常普遍的需求的替代解决方案。

## 3。让用户进入控制器

我们在一个`@Controller`注释 bean 中有额外的选项。

**我们可以直接将主体定义为方法参数**，它会被框架正确解析:

```java
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Principal principal) {
        return principal.getName();
    }
}
```

或者，**我们也可以使用认证令牌**:

```java
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Authentication authentication) {
        return authentication.getName();
    }
}
```

`Authentication`类的 API 非常开放，因此框架尽可能保持灵活性。因此，**Spring 安全主体只能作为`Object`被检索，并且需要被转换为正确的`UserDetails`实例**:

```java
UserDetails userDetails = (UserDetails) authentication.getPrincipal();
System.out.println("User has authorities: " + userDetails.getAuthorities());
```

最后，这里是直接来自 HTTP 请求的**:**

```java
@Controller
public class GetUserWithHTTPServletRequestController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserNameSimple(HttpServletRequest request) {
        Principal principal = request.getUserPrincipal();
        return principal.getName();
    }
}
```

## 4。通过自定义界面获得用户

为了充分利用 Spring 依赖注入并能够在任何地方检索认证，而不仅仅是在`@Controller beans`中，我们需要将静态访问隐藏在一个简单的外观后面:

```java
public interface IAuthenticationFacade {
    Authentication getAuthentication();
}
@Component
public class AuthenticationFacade implements IAuthenticationFacade {

    @Override
    public Authentication getAuthentication() {
        return SecurityContextHolder.getContext().getAuthentication();
    }
}
```

facade 公开了`Authentication`对象，同时隐藏了静态状态并保持代码解耦和完全可测试:

```java
@Controller
public class GetUserWithCustomInterfaceController {
    @Autowired
    private IAuthenticationFacade authenticationFacade;

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserNameSimple() {
        Authentication authentication = authenticationFacade.getAuthentication();
        return authentication.getName();
    }
}
```

## 5。获取 JSP 中的用户

通过利用 Spring Security Taglib 支持，还可以在 JSP 页面中访问当前认证的主体**。**

首先，我们需要在页面中定义标签:

```java
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```

接下来，我们可以**参考委托人**:

```java
<security:authorize access="isAuthenticated()">
    authenticated as <security:authentication property="principal.username" /> 
</security:authorize>
```

## 6.获取百里香中的用户

百里香叶是一个现代的服务器端 web 模板引擎，与 Spring MVC 框架有很好的 T2 集成。

让我们看看如何在一个带有百里香引擎的页面中访问当前已认证的主体。

首先，我们需要添加 [`thymeleaf-spring5`](https://web.archive.org/web/20220707143845/https://search.maven.org/search?q=a:thymeleaf-spring5) 和 [`thymeleaf-extras-springsecurity5`](https://web.archive.org/web/20220707143845/https://search.maven.org/search?q=a:thymeleaf-extras-springsecurity5) 依赖项来集成百里叶和春天的安全性:

```java
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>
```

现在**我们可以使用`sec:authorize`属性**在 HTML 页面中引用主体:

```java
<html xmlns:th="https://www.thymeleaf.org" 
  xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
<body>
    <div sec:authorize="isAuthenticated()">
      Authenticated as <span sec:authentication="name"></span></div>
</body>
</html>
```

## 7。结论

本文展示了如何在 Spring 应用程序中获取用户信息，从常见的静态访问机制开始，然后是注入主体的几种更好的方法。

这些例子的实现可以在[GitHub 项目](https://web.archive.org/web/20220707143845/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest-custom "Spring Security User retrieval example project")中找到。这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。在本地运行项目时，我们可以在此处访问主页 HTML:

```java
http://localhost:8080/spring-security-rest-custom/foos/1
```