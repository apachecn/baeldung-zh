# 如何使用 Spring Security 手动认证用户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/manually-set-user-authentication-spring-security>

## 1。概述

在这篇简短的文章中，我们将关注如何在 Spring Security 和 Spring MVC 中以编程方式设置一个经过身份验证的用户。

## 2。弹簧安全

简而言之，Spring Security 将每个经过身份验证的用户的主要信息保存在一个`ThreadLocal`对象中，表示为一个`Authentication`对象。

为了构造和设置这个`Authentication`对象——我们需要使用 Spring Security 通常用来在标准身份验证的基础上构建对象的相同方法。

让我们手动触发身份验证，然后将得到的`Authentication`对象设置为当前的`SecurityContext`,框架使用它来保存当前登录的用户:

```
UsernamePasswordAuthenticationToken authReq
 = new UsernamePasswordAuthenticationToken(user, pass);
Authentication auth = authManager.authenticate(authReq);
SecurityContext sc = SecurityContextHolder.getContext();
sc.setAuthentication(auth);
```

在上下文中设置了`Authentication`之后，我们现在可以使用`securityContext.getAuthentication().isAuthenticated()`来检查当前用户是否通过了身份验证。

## 3。Spring MVC

默认情况下，Spring Security 在 Spring Security 过滤器链中添加了一个额外的过滤器——它能够持久化安全上下文(`SecurityContextPersistenceFilter`类)。

反过来，它将安全上下文的持久性委托给一个实例`SecurityContextRepository`，默认为`HttpSessionSecurityContextRepository`类。

因此，为了对请求设置身份验证，从而使**对来自客户端**的所有后续请求可用，我们需要在 HTTP 会话中手动设置包含`Authentication`的`SecurityContext`:

```
public void login(HttpServletRequest req, String user, String pass) { 
    UsernamePasswordAuthenticationToken authReq
      = new UsernamePasswordAuthenticationToken(user, pass);
    Authentication auth = authManager.authenticate(authReq);

    SecurityContext sc = SecurityContextHolder.getContext();
    sc.setAuthentication(auth);
    HttpSession session = req.getSession(true);
    session.setAttribute(SPRING_SECURITY_CONTEXT_KEY, sc);
}
```

`SPRING_SECURITY_CONTEXT_KEY` 是静态导入的`HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY`。

需要注意的是，我们不能直接使用`HttpSessionSecurityContextRepository`——因为它与`SecurityContextPersistenceFilter.`一起工作

这是因为过滤器使用存储库，以便在执行链中其余已定义的过滤器之前和之后加载和存储安全上下文，但是它在传递给链的响应上使用自定义包装器。

所以在这种情况下，您应该知道所使用的包装器的类类型，并将其传递给存储库中适当的 save 方法。

## 4。结论

在这个快速教程中，我们讨论了如何在 Spring 安全上下文中手动设置用户`Authentication`，以及如何使其可用于 Spring MVC 目的，重点是说明实现它的最简单方法的代码示例。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221012100327/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc-custom)