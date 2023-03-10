# 春天安全还记得我吗

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-remember-me>

## 1。概述

本教程将向**展示如何使用 Spring Security 在 web 应用**中启用和配置 Remember Me 功能。设置[带有安全性和简单表单登录的 MVC 应用程序](/web/20221127083815/https://www.baeldung.com/spring-security-login "Spring Security Form Login")已经讨论过了。

该机制将能够**跨多个会话识别用户**——所以首先要理解的是，记住我只有在会话超时后才生效。默认情况下，这发生在 30 分钟不活动之后，但是可以在`web.xml`中配置的[超时。](/web/20221127083815/https://www.baeldung.com/servlet-session-timeout "Java Session Timeout")

注意:本教程重点介绍标准的基于 cookie 的方法。对于持续的方法，看看[春季安全-持续记住我](/web/20221127083815/https://www.baeldung.com/spring-security-persistent-remember-me)指南。

## 2。安全配置

让我们看看如何使用 Java 设置安全配置:

```java
@Configuration
@EnableWebSecurity
public class SecSecurityConfig {

    @Bean
    public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {
        return http.getSharedObject(AuthenticationManagerBuilder.class)
            .build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.withUsername("user1")
            .password("{noop}user1Pass")
            .authorities("ROLE_USER")
            .build();
        UserDetails admin = User.withUsername("admin1")
            .password("{noop}admin1Pass")
            .authorities("ROLE_ADMIN")
            .build();
        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/anonymous*")
            .anonymous()
            .antMatchers("/login*")
            .permitAll()
            .anyRequest()
            .authenticated()
            .and()
            .formLogin()
            .loginPage("/login.html")
            .loginProcessingUrl("/login")
            .failureUrl("/login.html?error=true")
            .and()
            .logout()
            .deleteCookies("JSESSIONID")
            .and()
            .rememberMe()
            .key("uniqueAndSecret");
        return http.build();
    }
}
```

正如你所看到的，**使用`rememberMe()`方法**的基本配置非常简单，同时通过附加选项保持非常灵活。这里的`**key**`很重要——它是整个应用程序的私有值秘密，在生成令牌内容时会用到。

此外，令牌有效的**时间可以使用`tokenValiditySeconds()`配置**，从默认的 2 周至例如 1 天:

```java
rememberMe().key("uniqueAndSecret").tokenValiditySeconds(86400)
```

我们还可以看看等效的 XML 配置:

```java
<http use-expressions="true">
    <intercept-url pattern="/anonymous*" access="isAnonymous()" />
    <intercept-url pattern="/login*" access="permitAll" />
    <intercept-url pattern="/**" access="isAuthenticated()" />

    <form-login login-page='/login.html' 
      authentication-failure-url="/login.html?error=true" />
    <logout delete-cookies="JSESSIONID" />

    <remember-me key="uniqueAndSecret"/>
</http>

<authentication-manager id="authenticationManager">
    <authentication-provider>
        <user-service>
            <user name="user1" password="{noop}user1Pass" authorities="ROLE_USER" />
            <user name="admin1" password="{noop}admin1Pass" authorities="ROLE_ADMIN" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

## 3。登录表单

登录表单类似于[我们用于表单登录](/web/20221127083815/https://www.baeldung.com/spring-security-login#login-form "A basic Login Form")的表单:

```java
<html>
<head></head>

<body>
    <h1>Login</h1>

    <form name='f' action="login" method='POST'>
        <table>
            <tr>
                <td>User:</td>
                <td><input type='text' name='username' value=''></td>
            </tr>
            <tr>
                <td>Password:</td>
                <td><input type='password' name='password' /></td>
            </tr>
            <tr>
                <td>Remember Me:</td>
                <td><input type="checkbox" name="remember-me" /></td>
            </tr>
            <tr>
                <td><input name="submit" type="submit" value="submit" /></td>
            </tr>
        </table>
    </form>

</body>
</html>
```

注意新添加的`checkbox`输入——映射到`remember-me`。这个添加的输入足以在激活“记住我”的情况下登录。

此默认路径也可以更改如下:

```java
.rememberMe().rememberMeParameter("remember-me-new")
```

## 4。曲奇饼

当用户登录时，该机制将创建一个额外的 cookie——“记住我”cookie。

**记住我 cookie** 包含以下数据:

*   **`username`**–识别登录的主体
*   **`expirationTime`**–将 cookie 过期；默认值为 2 周
*   **MD5 哈希**–前两个值——`username`和`expirationTime`，加上`password`和预定义的`key`

这里要注意的第一件事是，`username`和`password`都是 cookie 的一部分——这意味着，如果其中任何一个被更改，cookie 就不再有效。另外，`username`可以从 cookie 中读取。

此外，重要的是要理解，如果捕获了“记住我”cookie，这种机制可能会受到攻击。**cookie 将保持有效和可用**，直到过期或凭证被更改。

## 5。在实践中

要轻松查看“记住我”机制的工作情况，您可以:

*   以“记住我”活动状态登录
*   等待会话到期(或删除浏览器中的`JSESSIONID` cookie)
*   刷新页面

如果没有激活“记住我”，在 cookie 过期后，用户应该被**重定向回登录页面**。使用“记住我”,现在的用户**在新令牌/cookie 的帮助下保持登录状态**。

## 6。结论

本教程展示了如何在安全配置中设置和配置“记住我”功能，并简要描述了 cookie 中包含的数据类型。

这个实现可以在 Github 项目示例中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

当项目在本地运行时，可以在 [localhost](https://web.archive.org/web/20221127083815/http://localhost:8080/spring-security-mvc-custom/login.html "Access the project on localhost") 上访问`login.html`。