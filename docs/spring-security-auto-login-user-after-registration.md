# spring Security–注册后自动登录用户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-auto-login-user-after-registration>

## 1。概述

在这个快速教程中，我们将讨论如何在注册过程之后立即自动认证用户——在 Spring 安全实现中。

简单地说，一旦用户完成注册，他们通常会被重定向到登录页面，现在必须重新输入他们的用户名和密码。

让我们看看如何通过自动认证用户来避免这种情况。

在我们开始之前，请注意我们正在网站上的[注册系列](/web/20221101023954/https://www.baeldung.com/spring-security-registration)的范围内工作。

## 2。使用`HttpServletRequest`

一种非常简单的以编程方式强制认证的方法是利用`HttpServletRequest [login()](https://web.archive.org/web/20221101023954/https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#login%28java.lang.String,%20java.lang.String%29)`方法:

```java
public void authWithHttpServletRequest(HttpServletRequest request, String username, String password) {
    try {
        request.login(username, password);
    } catch (ServletException e) {
        LOGGER.error("Error while login ", e);
    }
}
```

现在，在幕后，`HttpServletRequest.login()` API 确实使用了`AuthenticationManager`来执行认证。

理解和处理在这一级可能发生的`ServletException`也很重要。

## 3。使用`AuthenticationManager`

接下来，我们也可以直接创建一个`UsernamePasswordAuthenticationToken`–然后手动通过标准的`AuthenticationManager` :

```java
public void authWithAuthManager(HttpServletRequest request, String username, String password) {
    UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password);
    authToken.setDetails(new WebAuthenticationDetails(request));

    Authentication authentication = authenticationManager.authenticate(authToken);

    SecurityContextHolder.getContext().setAuthentication(authentication);
}
```

注意我们是如何创建令牌请求的，如何通过标准的身份验证流传递它，然后在当前的安全上下文中显式地设置结果。

## 4。复杂注册

在一些更复杂的场景中，注册过程有多个阶段，例如，在用户可以登录系统之前的确认步骤。

在这种情况下，了解我们可以在哪里自动认证用户当然很重要。我们不能在他们注册后立即这样做，因为此时新创建的帐户仍处于禁用状态。

简而言之，在他们确认他们的账户后，我们必须执行自动认证。

此外，请记住，在这一点上，我们不再能够访问他们实际的原始凭据。我们只能访问用户的编码密码，这就是我们在这里使用的密码:

```java
public void authWithoutPassword(User user){

    List<Privilege> privileges = user.getRoles().stream().map(Role::getPrivileges)
      .flatMap(Collection::stream).distinct().collect(Collectors.toList());
    List<GrantedAuthority> authorities = privileges.stream()
        .map(p -> new SimpleGrantedAuthority(p.getName()))
        .collect(Collectors.toList());

    Authentication authentication = new UsernamePasswordAuthenticationToken(user, null, authorities);
    SecurityContextHolder.getContext().setAuthentication(authentication);
}
```

请注意我们在这里是如何正确设置认证权限的，就像在`AuthenticationProvider.`中通常会做的那样

## 5。结论

我们讨论了在注册过程之后自动认证用户的不同方法。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221101023954/https://github.com/Baeldung/spring-security-registration)