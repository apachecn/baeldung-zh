# Spring Security 5 中的默认密码编码器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-5-default-password-encoder>

## 1.概观

在 Spring Security 4 中，可以使用内存认证以纯文本形式存储密码。

版本 5 中对密码管理过程进行了重大改进，引入了更安全的默认密码编码和解码机制。这意味着，如果您的 Spring 应用程序以纯文本形式存储密码，升级到 Spring Security 5 可能会导致问题。

在这个简短的教程中，我们将描述其中一个潜在的问题，并演示一个解决方案。

## 2.春季安全 4

我们将首先展示一个标准的安全配置，它提供简单的内存认证(对 Spring 4 有效):

```java
@Configuration
public class InMemoryAuthWebSecurityConfigurer 
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) 
      throws Exception {
        auth.inMemoryAuthentication()
          .withUser("spring")
          .password("secret")
          .roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .antMatchers("/private/**")
          .authenticated()
          .antMatchers("/public/**")
          .permitAll()
          .and()
          .httpBasic();
    }
} 
```

这个配置定义了所有`/private/`映射方法的认证和`/public/.`下所有内容的公共访问

如果我们在 Spring Security 5 下使用相同的配置，我们将得到以下错误:

```java
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```

该错误告诉我们给定的密码**无法被解码，因为没有为我们的内存认证**配置密码编码器。

## 3.春季安全 5

我们可以通过用`PasswordEncoderFactories` 类定义一个`Delegating` `PasswordEncoder` 来修复这个错误。

我们使用这个编码器来配置我们的用户:

```java
@Configuration
public class InMemoryAuthWebSecurityConfigurer {

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
        UserDetails user = User.withUsername("spring")
            .password(encoder.encode("secret"))
            .roles("USER")
            .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

现在，使用这种配置，我们使用 BCrypt 以如下格式存储内存中的密码:

```java
{bcrypt}$2a$10$MF7hYnWLeLT66gNccBgxaONZHbrSMjlUofkp50sSpBw2PJjUqU.zS 
```

尽管我们可以定义自己的一组密码编码器，但还是建议坚持使用`PasswordEncoderFactories`中提供的[默认编码器](https://web.archive.org/web/20221006085131/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/factory/PasswordEncoderFactories.html)。

自 Spring Security 版本 5.7.0-M2 以来，Spring **不赞成**使用 WebSecurityConfigureAdapter，并建议创建没有它的配置。这篇[文章](https://web.archive.org/web/20221006085131/http://staging5.baeldung.com/spring-deprecated-websecurityconfigureradapter)对此有更详细的解释。

### 3.2.`NoOpPasswordEncoder`

如果出于任何原因，我们不想对配置的密码进行编码，我们可以使用 [`NoOpPasswordEncoder`](https://web.archive.org/web/20221006085131/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/password/NoOpPasswordEncoder.html) 。

为此，我们只需在提供给`password()`方法的密码前面加上`{noop}`标识符:

```java
@Configuration
public class InMemoryNoOpAuthWebSecurityConfigurer {

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.withUsername("spring")
            .password("{noop}secret")
            .roles("USER")
            .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

这样，当 Spring Security 将用户提供的密码与我们在上面配置的密码进行比较时，它将使用引擎盖下的`NoOpPasswordEncoder`。

然而，请注意，**我们不应该在生产应用程序上使用这种方法！**正如官方文档所说， **`NoOpPasswordEncoder`已经被弃用** **来表示它是一个遗留实现，使用它被认为是不安全的**。

### 3.3.迁移现有密码

我们可以通过以下方式将现有密码更新为推荐的 Spring Security 5 标准:

*   用编码值更新纯文本存储的密码:

```java
String encoded = new BCryptPasswordEncoder().encode(plainTextPassword); 
```

*   以已知的编码器标识符作为哈希存储密码的前缀:

```java
{bcrypt}$2a$10$MF7hYnWLeLT66gNccBgxaONZHbrSMjlUofkp50sSpBw2PJjUqU.zS
{sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0 
```

*   当存储密码的编码机制未知时，请求用户更新他们的密码

## 4.结论

在这个简单的例子中，我们使用新的密码存储机制将一个有效的 Spring 4 内存认证配置更新到了 Spring 5。

和往常一样，你可以在 GitHub 项目上找到源代码。