# Spring 安全中的授权与角色

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-granted-authority-vs-role>

 ![](img/6472b0e9f137859006ef3de7db58e799.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524021339/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在这篇简短的文章中，我们将向**解释在 Spring Security** 中`Role`和`GrantedAuthority`之间微妙而显著的区别。有关角色和权限的更多详细信息，请点击查看文章[。](/web/20220524021339/https://www.baeldung.com/role-and-privilege-for-spring-security-registration)

## 延伸阅读:

## [Spring 安全基础认证](/web/20220524021339/https://www.baeldung.com/spring-security-basic-authentication)

Set up Basic Authentication in Spring - the XML Configuration, the Error Messages, and example of consuming the secured URLs with curl.[Read more](/web/20220524021339/https://www.baeldung.com/spring-security-basic-authentication) →

## 2。`GrantedAuthority`

在 Spring Security 中，我们可以**将每个`GrantedAuthority`视为一个单独的特权**。例子可能包括`READ_AUTHORITY`、`WRITE_PRIVILEGE`，甚至`CAN_EXECUTE_AS_ROOT`。需要理解的重要一点是**这个名字是任意的**。

当直接使用一个`GrantedAuthority`时，比如通过使用像`hasAuthority(‘READ_AUTHORITY'),` 这样的表达式，我们以一种细粒度的方式限制访问**。**

您可能已经猜到，我们也可以通过使用`privilege`来引用`authority` 的概念。

## 3。权威角色

类似地，在 Spring Security 中，我们可以**将每个`Role as`看作一个粗粒度的`GrantedAuthority`，它被表示为一个`String`，并带有前缀`ROLE`**。当直接使用`Role`时，比如通过像`hasRole(“ADMIN”)`这样的表达式，我们以粗粒度的方式限制访问。

值得注意的是，默认的“`ROLE”`前缀是可配置的，但是解释如何配置超出了本文的范围。

这两者之间的核心区别是我们如何使用特性的语义。对于框架来说，差别很小——它基本上以完全相同的方式处理这些问题。

## 4。作为容器的角色

既然我们已经看到了框架如何使用`role`概念，让我们也快速讨论一个替代方案——那就是**使用角色作为权限/特权**的容器。

这是一个更高层次的角色方法——使它们成为一个更面向业务的概念，而不是一个以实现为中心的概念。

Spring 安全框架并没有就我们应该如何使用这个概念给出任何指导，所以这个选择完全是特定于实现的。

## 5。Spring 安全配置

我们可以通过限制使用`READ_AUTHORITY`的用户对`/protectedbyauthority`的访问来演示一个细粒度的授权需求。

我们可以通过限制使用`ROLE_USER`的用户对`/protectedbyrole`的访问来演示粗粒度授权需求。

让我们在安全配置中配置这样一个场景:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // ...
    .antMatchers("/protectedbyrole").hasRole("USER")
    .antMatchers("/protectedbyauthority").hasAuthority("READ_PRIVILEGE")
    // ...
}
```

## 6。简单数据初始化

现在我们更好地理解了核心概念，让我们讨论一下在应用程序启动时创建一些设置数据。

当然，这是一种非常简单的方法，在开发过程中与一些初步测试用户一起投入使用——而不是在生产中处理数据的方法。

我们将监听上下文刷新事件:

```java
@Override
@Transactional
public void onApplicationEvent(ContextRefreshedEvent event) {
    MyPrivilege readPrivilege
      = createPrivilegeIfNotFound("READ_PRIVILEGE");
    MyPrivilege writePrivilege
      = createPrivilegeIfNotFound("WRITE_PRIVILEGE"); 
}
```

这里的实际实现并不重要——通常取决于您使用的持久性解决方案。要点是——我们坚持在代码中使用的权限。

## 7。`UserDetailsService`

我们的 **`UserDetailsService`实现是权限映射发生的地方**。一旦用户通过了身份验证，我们的`getAuthorities()`方法就会填充并返回一个`UserDetails`对象:

```java
private Collection<? extends GrantedAuthority> getAuthorities(
  Collection<Role> roles) {
    List<GrantedAuthority> authorities
      = new ArrayList<>();
    for (Role role: roles) {
        authorities.add(new SimpleGrantedAuthority(role.getName()));
        role.getPrivileges().stream()
         .map(p -> new SimpleGrantedAuthority(p.getName()))
         .forEach(authorities::add);
    }

    return authorities;
}
```

## 8。运行和测试示例

我们可以执行示例`RolesAuthoritiesApplication` Java 应用程序，在 [GitHub 项目](https://web.archive.org/web/20220524021339/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-1)中找到。

**要查看基于角色的授权的运行情况，我们需要:**

*   访问 http://localhost:8082/protected byrole
*   认证为`[[email protected]](/web/20220524021339/https://www.baeldung.com/cdn-cgi/l/email-protection)`(密码为`“user”`)
*   注意授权成功
*   访问 http://localhost:8082/protectedbyauthority
*   注意不成功的授权

**要查看基于授权的授权的运行情况，我们需要注销应用程序，然后:**

*   访问 http://localhost:8082/protectedbyauthority
*   认证为[【电子邮件保护】](/web/20220524021339/https://www.baeldung.com/cdn-cgi/l/email-protection) /管理员
*   注意授权成功
*   访问 http://localhsot:8082/protected byblole
*   注意不成功的授权

## 9。结论

在这个快速教程中，我们看到了 Spring Security 中的一个`Role`和一个`GrantedAuthority`之间微妙但重要的区别。