# Spring 安全性:检查用户在 Java 中是否有角色

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-check-user-role>

## 1.介绍

在 Spring Security 中，有时有必要检查经过身份验证的用户是否具有特定的角色。**这有助于启用或禁用我们应用程序中的特定功能**。

在本教程中，我们将看到在 Spring Security 的 Java 中检查用户角色的各种方法。

## 2.在 Java 中检查用户角色

Spring Security 提供了几种方法来检查 Java 代码中的用户角色。我们将在下面逐一介绍。

### 2.1.`@PreAuthorize`

在 Java 中检查用户角色的第一种方法是使用 Spring Security 提供的 [`@PreAuthorize`注释](/web/20221022193028/https://www.baeldung.com/spring-security-method-security)。这个注释可以应用于一个类或方法，它接受一个表示一个 [SpEL 表达式](/web/20221022193028/https://www.baeldung.com/spring-expression-language)的字符串值。

在使用这个注释之前，我们必须首先启用全局方法安全性。这可以在 Java 代码中通过向任何配置类添加`@EnableGlobalMethodSecurity`注释来完成。

然后， [Spring Security 提供了两个表达式](/web/20221022193028/https://www.baeldung.com/basic-and-digest-authentication-for-a-rest-api-with-spring-security),我们可以使用它们和`@PreAuthorize`注释来检查用户角色:

```java
@PreAuthorize("hasRole('ROLE_ADMIN')")
@GetMapping("/user/{id}")
public String getUser(@PathVariable("id") String id) {
    ...
}
```

我们还可以在一个表达式中检查多个角色:

```java
@PreAuthorize("hasAnyRole('ROLE_ADMIN','ROLE_MANAGER')")
@GetMapping("/users")
public String getUsers() {
    ...
}
```

在这种情况下，如果用户拥有指定角色的`any`，请求将被允许。

如果在没有适当角色的情况下调用该方法，Spring Security 会抛出一个异常，并重定向到[错误页面](/web/20221022193028/https://www.baeldung.com/spring-boot-custom-error-page)。

### 2.2.`SecurityContext`

我们可以检查 Java 代码中用户角色的下一种方法是使用`SecurityContext`类。

默认情况下，Spring Security 使用该类的线程本地副本。这意味着我们应用程序中的每个请求都有其安全上下文，其中包含发出请求的用户的详细信息。

要使用它，我们只需调用`SecurityContextHolder`中的静态方法:

```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
if (auth != null && auth.getAuthorities().stream().anyMatch(a -> a.getAuthority().equals("ADMIN"))) {
    ...
}
```

注意，我们在这里使用普通的权限名称，而不是完整的角色名称。

当我们需要更细粒度的检查时——例如，单个方法的特定部分——这很有用。然而，**如果我们在 Spring Security** 中使用全局上下文容器模式，这种方法将不起作用。

### 2.3.`UserDetailsService`

我们在 Java 代码中查找用户角色的第三种方法是使用 [`UserDetailsService`](/web/20221022193028/https://www.baeldung.com/spring-security-authentication-with-a-database) 。我们可以将这个 bean 注入到应用程序的任何地方，并根据需要调用它:

```java
@GetMapping("/users")
public String getUsers() {
    UserDetails details = userDetailsService.loadUserByUsername("mike");
    if (details != null && details.getAuthorities().stream()
      .anyMatch(a -> a.getAuthority().equals("ADMIN"))) {
        // ...
    }
}
```

同样，我们必须在这里使用权威名称，而不是带前缀的完整角色名称。

这种方法的好处是我们可以检查任何用户的角色，而不仅仅是发出请求的那个用户。

### 2.4.Servlet 请求

如果我们使用的是 [Spring MVC](/web/20221022193028/https://www.baeldung.com/intro-to-servlets) ，我们也可以使用`HttpServletRequest`类在 Java 中检查用户角色:

```java
@GetMapping("/users")
public String getUsers(HttpServletRequest request) {
    if (request.isUserInRole("ROLE_ADMIN")) {
        ...
    }
}
```

## 3.结论

在本文中，我们已经看到了使用 Java 代码和 Spring Security 检查角色的几种不同方法。

和往常一样，本文中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221022193028/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)