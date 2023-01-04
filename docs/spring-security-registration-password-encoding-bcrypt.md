# 向 Spring Security 注册-密码编码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20220628095952/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20220628095952/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20220628095952/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20220628095952/https://www.baeldung.com/spring-security-registration-verification-email)
• Registration with Spring Security – Password Encoding (current article)[• The Registration API becomes RESTful](/web/20220628095952/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20220628095952/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220628095952/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220628095952/https://www.baeldung.com/updating-your-password/)

## 1。概述

本文讨论了注册过程中的一个关键部分——**密码编码**——基本上不是以明文形式存储密码。

Spring Security 支持一些编码机制——对于本文，**我们将使用 BCrypt** ,因为它通常是可用的最佳解决方案。

大多数其他机制，如`MD5PasswordEncoder`和`ShaPasswordEncoder`使用较弱的算法，现在已被否决。

## 延伸阅读:

## [Spring Security 5 中的新密码存储](/web/20220628095952/https://www.baeldung.com/spring-security-5-password-storage)

A quick guide to understanding password encryption in Spring Security 5 and migrating to better encryption algorithms.[Read more](/web/20220628095952/https://www.baeldung.com/spring-security-5-password-storage) →

## [仅通过 Spring Security 允许来自认可位置的认证](/web/20220628095952/https://www.baeldung.com/spring-security-restrict-authentication-by-geography)

Learn how to only allow users to authenticate from accepted locations only with Spring Security.[Read more](/web/20220628095952/https://www.baeldung.com/spring-security-restrict-authentication-by-geography) →

## [Spring Security–注册后自动登录用户](/web/20220628095952/https://www.baeldung.com/spring-security-auto-login-user-after-registration)

Learn how to quickly auto-authenticate a user after they complete the registration process.[Read more](/web/20220628095952/https://www.baeldung.com/spring-security-auto-login-user-after-registration) →

## 2。定义密码编码器

我们首先将简单的 BCryptPasswordEncoder 定义为配置中的 bean:

```java
@Bean
public PasswordEncoder encoder() {
    return new BCryptPasswordEncoder();
}
```

旧的实现——比如`SHAPasswordEncoder`——会要求客户端在对密码进行编码时传递一个 salt 值。

然而，BCrypt，**将在内部生成一个随机盐**来代替。理解这一点很重要，因为这意味着每次调用都会有不同的结果，所以我们只需要对密码进行一次编码。

为了使这种随机盐生成工作，BCrypt 将盐存储在哈希值本身中。例如，在下面的哈希值中:

```java
$2a$10$ZLhnHxdpHETcxmtEStgpI./Ri1mksgJ9iDP36FmfMdYyVg9g0b2dq
```

有三个由＄分隔的字段:

1.  `“2a” `代表 BCrypt 算法版本
2.  `“10” `代表算法的强度
3.  `“ZLhnHxdpHETcxmtEStgpI.” `部分其实就是随机生成的盐。基本上前 22 个字都是盐。最后一个字段的剩余部分是纯文本的实际散列版本

另外，请注意，`BCrypt`算法会生成一个长度为 60 的字符串，因此我们需要确保密码将被存储在一个能够容纳它的列中。一个常见的错误是创建一个不同长度的列，然后在验证时得到一个`Invalid Username or Password`错误。

## 3。注册时对密码进行编码

我们现在将在用户注册过程中使用`UserService`中的`PasswordEncoder` 来散列密码:

**例 3.1。—`UserServic``e`散列密码**

```java
@Autowired
private PasswordEncoder passwordEncoder;

@Override
public User registerNewUserAccount(UserDto accountDto) throws EmailExistsException {
    if (emailExist(accountDto.getEmail())) {
        throw new EmailExistsException(
          "There is an account with that email adress:" + accountDto.getEmail());
    }
    User user = new User();
    user.setFirstName(accountDto.getFirstName());
    user.setLastName(accountDto.getLastName());

    user.setPassword(passwordEncoder.encode(accountDto.getPassword()));

    user.setEmail(accountDto.getEmail());
    user.setRole(new Role(Integer.valueOf(1), user));
    return repository.save(user);
}
```

## 4。认证时对密码进行编码

现在让我们处理这个过程的另一半，并在用户验证时对密码进行编码。

首先，我们需要将我们之前定义的密码编码器 bean 注入到我们的身份验证提供程序中:

```java
@Autowired
private UserDetailsService userDetailsService;

@Bean
public DaoAuthenticationProvider authProvider() {
    DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
    authProvider.setUserDetailsService(userDetailsService);
    authProvider.setPasswordEncoder(encoder());
    return authProvider;
}
```

安全配置很简单:

*   我们正在注入用户详细信息服务的实现
*   我们正在定义一个引用我们的细节服务的身份验证提供者
*   我们还启用了密码编码器

最后，我们需要在我们的安全 XML 配置中引用这个身份验证提供者:

```java
<authentication-manager>
    <authentication-provider ref="authProvider" />
</authentication-manager>
```

或者，如果您使用 Java 配置:

```java
@Configuration
@ComponentScan(basePackages = { "com.baeldung.security" })
@EnableWebSecurity
public class SecSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(authProvider());
    }

    ...
}
```

## 5。结论

这个快速教程是注册系列的继续，展示了如何利用简单但非常强大的 BCrypt 实现在数据库中正确存储密码。

这个注册 Spring 安全教程的**完整实现**可以在 GitHub 上面的[找到。](https://web.archive.org/web/20220628095952/https://github.com/Baeldung/spring-security-registration "The Full Registration Example Project on Github ")

Next **»**[The Registration API becomes RESTful](/web/20220628095952/https://www.baeldung.com/registration-restful-api)**«** Previous[Spring Security Registration – Resend Verification Email](/web/20220628095952/https://www.baeldung.com/spring-security-registration-verification-email)