# Spring Security 中的自定义访问决策投票者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-voter>

## 1。简介

大多数时候，当保护一个 Spring Web 应用程序或 REST API 时，Spring Security 提供的工具已经足够了，但有时我们会寻找一种更具体的行为。

在本教程中，我们将编写一个自定义的`AccessDecisionVoter`，并展示如何使用它来抽象出 web 应用程序的授权逻辑，并将其与应用程序的业务逻辑分离开来。

## 2.**场景**

为了演示`AccessDecisionVoter`是如何工作的，我们将实现一个有两种用户类型的场景，`USER`和`ADMIN,`，其中`USER`只能在偶数分钟访问系统，而`ADMIN`将总是被授权访问。

## 3。`AccessDecisionVoter` 实施

首先，我们将描述 Spring 提供的几个实现，它们将与我们的自定义投票器一起参与最终的授权决策。然后，我们将看看如何实现自定义投票器。

### 3.1。默认`AccessDecisionVoter`实现

Spring Security 提供了几个`AccessDecisionVoter`实现。在这里，我们将使用其中的一些作为我们安全解决方案的一部分。

让我们看看这些默认投票器实现如何以及何时投票。

`AuthenticatedVoter`将根据`Authentication`对象的认证级别进行投票——特别是寻找完全认证的委托人、通过“记住我”认证的委托人，或者最后是匿名委托人。

`RoleVoter`表决是否有任何配置属性以字符串 `“ROLE_”.`开头，如果有，它将在`Authentication`对象的`GrantedAuthority`列表中搜索角色。

`WebExpressionVoter`使我们能够使用 SpEL (Spring Expression Language)来授权使用`@PreAuthorize`注释的请求。

例如，如果我们使用 Java 配置:

```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/").hasAnyAuthority("ROLE_USER")
    ...
}
```

或者使用 XML 配置——我们可以在`intercept-url`标签中使用 SpEL，在`http`标签中:

```java
<http use-expressions="true">
    <intercept-url pattern="/"
      access="hasAuthority('ROLE_USER')"/>
    ...
</http>
```

### 3.2。自定义`AccessDecisionVoter`实现

现在让我们通过实现`AccessDecisionVoter`接口来创建一个自定义投票器:

```java
public class MinuteBasedVoter implements AccessDecisionVoter {
   ...
}
```

我们必须提供的三种方法中的第一种是`vote` 方法。`vote`方法是定制表决器最重要的部分，也是我们授权逻辑的所在。

`vote`方法可以返回三个可能的值:

*   投票者给出了肯定的答案
*   投票者给出了否定的答案
*   投票人放弃投票

现在让我们实现`vote`方法:

```java
@Override
public int vote(
  Authentication authentication, Object object, Collection collection) {
    return authentication.getAuthorities().stream()
      .map(GrantedAuthority::getAuthority)
      .filter(r -> "ROLE_USER".equals(r) 
        && LocalDateTime.now().getMinute() % 2 != 0)
      .findAny()
      .map(s -> ACCESS_DENIED)
      .orElseGet(() -> ACCESS_ABSTAIN);
}
```

在我们的`vote`方法中，我们检查请求是否来自`USER`。如果是，如果是偶数分钟，我们返回`ACCESS_GRANTED` ，否则，我们返回 `ACCESS_DENIED.` ，如果请求不是来自`USER,`，我们弃权并返回`ACCESS_ABSTAIN`。

第二个方法返回表决器是否支持特定的配置属性。在我们的例子中，表决器不需要任何定制的配置属性，所以我们返回`true`:

```java
@Override
public boolean supports(ConfigAttribute attribute) {
    return true;
}
```

第三个方法返回投票者是否可以投票给安全对象类型。因为我们的表决器不关心受保护的对象类型，所以我们返回`true`:

```java
@Override
public boolean supports(Class clazz) {
    return true;
}
```

## 4。 `AccessDecisionManager`

最终的授权决定由`AccessDecisionManager`处理。

`AbstractAccessDecisionManager`包含一个`AccessDecisionVoter`列表——它们负责独立投票。

有三种处理投票的实现来涵盖最常见的用例:

*   `AffirmativeBased`–如果任何一个`AccessDecisionVoter`投赞成票，则授予访问权
*   `ConsensusBased`–如果赞成票多于反对票，则授予访问权限(忽略弃权的用户)
*   `UnanimousBased`–如果每个投票者都弃权或投赞成票，则授予访问权

当然，你可以用你定制的决策逻辑实现你自己的`AccessDecisionManager`。

## 5。配置

在教程的这一部分，我们将看看用一个`AccessDecisionManager`配置我们的自定义`AccessDecisionVoter`的基于 Java 和基于 XML 的方法。

### 5.1。Java 配置

让我们为 Spring Web Security 创建一个配置类:

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
...
}

```

让我们定义一个`AccessDecisionManager` bean，它使用一个`UnanimousBased`管理器和我们定制的投票者列表:

```java
@Bean
public AccessDecisionManager accessDecisionManager() {
    List<AccessDecisionVoter<? extends Object>> decisionVoters 
      = Arrays.asList(
        new WebExpressionVoter(),
        new RoleVoter(),
        new AuthenticatedVoter(),
        new MinuteBasedVoter());
    return new UnanimousBased(decisionVoters);
}
```

最后，让我们配置 Spring Security，使用之前定义的 bean 作为默认的`AccessDecisionManager`:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
    ...
    .anyRequest()
    .authenticated()
    .accessDecisionManager(accessDecisionManager());
}

```

### 5.2。XML 配置

如果使用 XML 配置，您需要修改您的`spring-security.xml`文件(或者任何包含您的安全设置的文件)。

首先，您需要修改`<http>`标签:

```java
<http access-decision-manager-ref="accessDecisionManager">
  <intercept-url
    pattern="/**"
    access="hasAnyRole('ROLE_ADMIN', 'ROLE_USER')"/>
  ...
</http>
```

接下来，为自定义投票者添加一个 bean:

```java
<beans:bean
  id="minuteBasedVoter"
  class="org.baeldung.voter.MinuteBasedVoter"/>

```

然后为`AccessDecisionManager`添加一个 bean:

```java
<beans:bean 
  id="accessDecisionManager" 
  class="org.springframework.security.access.vote.UnanimousBased">
    <beans:constructor-arg>
        <beans:list>
            <beans:bean class=
              "org.springframework.security.web.access.expression.WebExpressionVoter"/>
            <beans:bean class=
              "org.springframework.security.access.vote.AuthenticatedVoter"/>
            <beans:bean class=
              "org.springframework.security.access.vote.RoleVoter"/>
            <beans:bean class=
              "org.baeldung.voter.MinuteBasedVoter"/>
        </beans:list>
    </beans:constructor-arg>
</beans:bean>

```

这里有一个支持我们场景的示例`<authentication-manager>`标签:

```java
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user" password="pass" authorities="ROLE_USER"/>
            <user name="admin" password="pass" authorities="ROLE_ADMIN"/>
        </user-service>
    </authentication-provider>
</authentication-manager>
```

如果使用 Java 和 XML 配置的组合，可以将 XML 导入配置类:

```java
@Configuration
@ImportResource({"classpath:spring-security.xml"})
public class XmlSecurityConfig {
    public XmlSecurityConfig() {
        super();
    }
}
```

## 6.**结论**

在本教程中，我们看了一种通过使用`AccessDecisionVoter` s 为 Spring Web 应用程序定制安全性的方法。我们看到了 Spring Security 提供的一些投票者对我们的解决方案做出了贡献。然后我们讨论了如何实现自定义的`AccessDecisionVoter`。

然后我们讨论了`AccessDecisionManager`如何做出最终的授权决定，我们展示了如何使用 Spring 提供的实现在所有投票者投票后做出这个决定。

然后我们通过 Java 和 XML 用一个`AccessDecisionManager`配置了一个`AccessDecisionVoters` 的列表。

实现可以在 [Github 项目](https://web.archive.org/web/20190803230642/https://github.com/eugenp/tutorials/tree/master/spring-security-mvc-boot)中找到。

当项目在本地运行时，登录页面可在以下位置访问:

```java
http://localhost:8080/spring-security-custom-permissions/login
```

`USER`的凭证是“用户”和“通行证”,`ADMIN`的凭证是“管理员”和“通行证”。