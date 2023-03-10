# 具有 Spring 安全性的 Spring 数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-security>

 ![](img/322c46af3bd1a99bb065a7965f76d5f9.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524033438/https://www.baeldung.com/lightrun-n-security)

## 1。概述

Spring Security 为集成 Spring 数据提供了很好的支持。前者处理应用程序的安全方面，后者提供对包含应用程序数据的数据库的方便访问。

在本文中，我们将讨论如何将 **Spring Security 与 Spring Data 集成，以支持更多特定于用户的查询**。

## 2。Spring 安全+ Spring 数据配置

在我们对 Spring 数据 JPA 的[介绍中，我们看到了如何在 Spring 项目中设置 Spring 数据。为了启用 spring 安全性和 spring 数据，通常，我们可以采用基于 Java 或 XML 的配置。](/web/20220524033438/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

### 2.1。Java 配置

回想一下，从 [Spring Security 登录表单](/web/20220524033438/https://www.baeldung.com/spring-security-login)(第 4 节&第 5 节)，我们可以使用基于注释的配置将 Spring Security 添加到我们的项目中:

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    // Bean definitions
}
```

其他配置细节包括过滤器、beans 和其他安全规则的定义。

**为了在 Spring Security 中启用 Spring 数据，我们只需将这个 bean 添加到`WebSecurityConfig` :**

```java
@Bean
public SecurityEvaluationContextExtension securityEvaluationContextExtension() {
    return new SecurityEvaluationContextExtension();
}
```

上面的定义激活了在类上标注的特定于 spring 数据的表达式的自动解析。

### 2.2。XML 配置

基于 XML 的配置从包含 Spring 安全名称空间开始:

```java
<beans:beans 
  xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
  http://www.springframework.org/schema/security
  http://www.springframework.org/schema/security/spring-security.xsd">
...
</beans:beans>
```

就像在基于 Java 的配置中一样，对于基于 XML 或名称空间的配置，我们将向 XML 配置文件添加**securityevaluationcontextension**bean:

```java
<bean class="org.springframework.security.data.repository
  .query.SecurityEvaluationContextExtension"/>
```

定义`SecurityEvaluationContextExtension`使得 Spring Security 中的所有通用表达式都可以在 Spring 数据查询中使用。

这样的常用表达有`principal, authentication, isAnonymous(), hasRole([role]), isAuthenticated,` 等。

## 3。用法示例

让我们考虑一些 Spring 数据和 Spring 安全性的用例。

### 3.1。限制`AppUser`字段更新

在这个例子中，我们将着眼于将`App` `User`的`lastLogin`字段更新限制为当前唯一经过身份验证的用户。

我们的意思是，每当触发`updateLastLogin`方法时，它只更新当前认证用户的`lastLogin`字段。

为了实现这一点，我们将下面的查询添加到我们的`UserRepository`接口:

```java
@Query("UPDATE AppUser u SET u.lastLogin=:lastLogin WHERE" 
  +" u.username = ?#{ principal?.username }")
void updateLastLogin (Date lastLogin);
```

如果没有 Spring 数据和 Spring 安全集成，我们通常必须将用户名作为参数传递给`updateLastLogin`。

在提供了错误的用户凭证的情况下，登录过程将会失败，我们不需要费心确保访问的有效性。

### 3.2。通过分页获取特定的`AppUser'`内容

Spring Data 和 Spring Security 完美结合的另一个场景是，我们需要从数据库中检索当前认证用户拥有的内容。

例如，如果我们有一个 tweetser 应用程序，我们可能希望在当前用户的个性化 feeds 页面上显示他们创建或喜欢的 tweet。

当然，这可能涉及到编写查询来与数据库中的一个或多个表进行交互。有了 Spring 数据和 Spring 安全，这就像写:

```java
public interface TweetRepository extends PagingAndSortingRepository<Tweet, Long> {
    @Query("SELECT twt FROM Tweet twt JOIN twt.likes AS lk WHERE lk = ?#{ principal?.username }" +
      " OR twt.owner = ?#{ principal?.username }")
    Page<Tweet> getMyTweetsAndTheOnesILiked(Pageable pageable);
}
```

因为我们希望我们的结果分页，所以我们的`TweetRepository`在上面的接口定义中扩展了`PagingAndSortingRepository`。

## 4。结论

Spring 数据和 Spring 安全集成为管理 Spring 应用程序中的认证状态带来了很大的灵活性。

在本节中，我们已经了解了如何将 Spring 安全性添加到 Spring 数据中。更多关于 Spring 数据或 Spring 安全的其他强大特性可以在我们收集的 [Spring 数据](/web/20220524033438/https://www.baeldung.com/spring-data)和 [Spring 安全](/web/20220524033438/https://www.baeldung.com/security-spring)文章中找到。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220524033438/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-1)