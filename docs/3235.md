# 春天的安全——坚持记住我

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-persistent-remember-me>

## 1。概述

本文将展示如何在 Spring Security 中设置 **Remember Me 功能——不使用标准的仅 cookie 方法，而是使用**一个更安全的解决方案，使用持久性**。**

简单介绍一下——Spring 可以被配置为在浏览器会话之间记住登录细节。这允许您登录到一个网站，然后让它在您下次访问该网站时自动让您重新登录(即使您在此期间关闭了浏览器)。

## 2。两个“记住我”解决方案

Spring 提供了两个略有不同的实现来解决这个问题。两者都使用`UsernamePasswordAuthenticationFilter`，使用钩子来调用一个`RememberMeServices`实现。

在之前的一篇文章中，我们已经介绍了标准的只使用 cookie 的“记住我”解决方案。该解决方案使用一个名为`remember-me`的 cookie，其中包含用户名、到期时间和包含密码的 MD5 哈希。因为它包含了密码的散列，**如果 cookie 被捕获，这个解决方案就有可能受到**的攻击。

记住这一点——让我们看看第二种方法——使用`PersistentTokenBasedRememberMeServices`在会话之间将持久化的登录信息存储在数据库表中。

## 3。先决条件–创建数据库表

首先，我们需要在数据库中有登录信息，我们需要创建一个表来保存数据:

```
create table if not exists persistent_logins ( 
  username varchar_ignorecase(100) not null, 
  series varchar(64) primary key, 
  token varchar(64) not null, 
  last_used timestamp not null 
); 
```

这是在启动时通过以下 XML 配置自动创建的**(使用内存中的 H2 数据库):**

```
<!-- create H2 embedded database table on startup -->
<jdbc:embedded-database id="dataSource" type="H2">
    <jdbc:script location="classpath:/persisted_logins_create_table.sql"/> 
</jdbc:embedded-database>
```

为了完整起见，下面是建立持久性的方式:

```
@Configuration
@EnableTransactionManagement
@PropertySource({ "classpath:persistence-h2.properties" })
public class DatabaseConfig {

    @Autowired private Environment env;

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
        dataSource.setUrl(env.getProperty("jdbc.url"));
        dataSource.setUsername(env.getProperty("jdbc.user"));
        dataSource.setPassword(env.getProperty("jdbc.pass"));
        return dataSource;
    }
}
```

## 4。春季安全配置

第一个关键配置是 Remember-Me Http 配置(注意`dataSource`属性):

```
<http use-expressions="true">
    ...
    <remember-me data-source-ref="dataSource" token-validity-seconds="86400"/>
<http"> 
```

接下来——我们需要配置实际的`RememberMeService`和`JdbcTokenRepository`(这也利用了`dataSource)`:

```
<!-- Persistent Remember Me Service -->
<beans:bean id="rememberMeAuthenticationProvider" class=
  "org.springframework.security.web.authentication.rememberme.PersistentTokenBasedRememberMeServices">
    <beans:constructor-arg value="myAppKey" />
    <beans:constructor-arg ref="jdbcTokenRepository" />
    <beans:constructor-arg ref="myUserDetailsService" />
</beans:bean>

<!-- Uses a database table to maintain a set of persistent login data -->
<beans:bean id="jdbcTokenRepository" 
  class="org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl"> 
    <beans:property name="createTableOnStartup" value="false" /> 
    <beans:property name="dataSource" ref="dataSource" /> 
</beans:bean>

<!-- Authentication Manager (uses same UserDetailsService as RememberMeService)--> 
<authentication-manager alias="authenticationManager"> 
    <authentication-provider user-service-ref="myUserDetailsService"/> 
    </authentication-provider> 
</authentication-manager> 
```

## 5。曲奇饼

正如我们提到的，标准的`TokenBasedRememberMeServices`将散列的用户密码存储在 cookie 中。

这个解决方案——`PersistentTokenBasedRememberMeServices`为用户使用一个**唯一的系列标识符。这标识了用户的初始登录，并且在持续会话期间每次用户自动登录时保持不变。它还包含**一个随机令牌**，每当用户通过持久化的“记住我”功能登录时，这个令牌就会重新生成。**

这种随机生成的序列和令牌的组合是持久的，使得暴力攻击非常不可能。

## 6。在实践中

要在浏览器中查看“记住我”机制，您可以:

1.  激活“记住我”后登录
2.  关闭浏览器
3.  重新打开浏览器并返回同一页面。刷新。
4.  您仍将登录

**在没有激活“记住我”**的情况下，cookie 过期后，用户应被重定向回登录页面。**有了“记住我”**，用户现在可以在新令牌/cookie 的帮助下保持登录。

您还可以在浏览器中查看 cookies，以及数据库中的持久化数据(注意，您可能希望从嵌入式 H2 实现中切换过来)。

## 7。结论

本教程演示了如何**设置和配置数据库持久化的记住我令牌功能**。这也是上一篇文章的后续，上一篇文章讨论了[标准的基于 Cookie 令牌的功能](/web/20220809103334/https://www.baeldung.com/spring-security-remember-me "Spring Security Remember Me")。数据库方法更安全，因为密码细节不保存在 cookie 中——但它涉及的配置稍微多一点。

这个 Spring Security REST 教程的实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。