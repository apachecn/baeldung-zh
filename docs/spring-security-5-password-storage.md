# Spring Security 5 中的新密码存储

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-5-password-storage>

## 1。简介

随着最新的 Spring Security 发布，很多事情都变了。其中一个变化是我们如何在应用程序中处理密码编码。

在本教程中，我们将探索其中的一些变化。

稍后，我们将看到如何配置新的委托机制，以及如何更新现有的密码编码，而不被用户识别。

## 2。Spring Security 5.x 的相关变化

Spring 安全团队声明`org.springframework.security.authentication.encoding` 中的`PasswordEncoder`已被弃用。这是一个合乎逻辑的举动，因为旧的接口不是为随机生成的 salt 设计的。因此，版本 5 删除了这个接口。

此外，Spring Security 改变了它处理编码密码的方式。在以前的版本中，每个应用程序只采用一种密码编码算法。

默认情况下，`StandardPasswordEncoder` 处理了那个。它使用 SHA-256 进行编码。通过改变密码编码器，我们可以切换到另一种算法。但是我们的应用程序必须严格遵循一种算法。

**5.0 版本引入了密码编码委托的概念。**现在，我们可以对不同的密码使用不同的编码。Spring 通过在编码密码前添加一个标识符来识别算法。

以下是 bcrypt 编码密码的示例:

```java
{bcrypt}$2b$12$FaLabMRystU4MLAasNOKb.HUElBAabuQdX59RWHq5X.9Ghm692NEi
```

注意 bcrypt 是如何在最开始用花括号指定的。

## 3。委托配置

**如果密码哈希没有前缀，授权过程将使用默认编码器。**因此，默认情况下，我们得到了`StandardPasswordEncoder.`

这使得它与以前的 Spring Security 版本的默认配置兼容。

在版本 5 中，Spring Security 引入了`PasswordEncoderFactories.createDelegatingPasswordEncoder().`，这个工厂方法返回一个配置好的`DelegationPasswordEncoder`实例。

对于没有前缀的密码，该实例确保了刚才提到的默认行为。对于包含前缀的密码散列，相应地进行委托。

Spring 安全团队列出了最新版本的[对应 JavaDoc](https://web.archive.org/web/20221222004958/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/factory/PasswordEncoderFactories.html#createDelegatingPasswordEncoder--) 中支持的算法。

当然，Spring 允许我们配置这种行为。

假设我们想要支持:

*   `bcrypt`作为我们的新默认值
*   作为一种选择
*   SHA-256 作为当前使用的算法。

该设置的配置如下所示:

```java
@Bean
public PasswordEncoder delegatingPasswordEncoder() {
    PasswordEncoder defaultEncoder = new StandardPasswordEncoder();
    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put("bcrypt", new BCryptPasswordEncoder());
    encoders.put("scrypt", new SCryptPasswordEncoder());

    DelegatingPasswordEncoder passworEncoder = new DelegatingPasswordEncoder(
      "bcrypt", encoders);
    passworEncoder.setDefaultPasswordEncoderForMatches(defaultEncoder);

    return passworEncoder;
}
```

## 4。迁移密码编码算法

在上一节中，我们探讨了如何根据需要配置密码编码。因此，现在我们将研究如何将已经编码的密码转换成新的算法。

假设我们想将编码从`SHA-256` 更改为`bcrypt`，但是，我们不希望用户更改他们的密码。

一个可能的解决方案是使用登录请求。此时，我们可以以纯文本方式访问凭证。此时，我们可以获取当前密码并重新编码。

**因此，我们可以使用 Spring 的`AuthenticationSuccessEvent`来实现。该事件在用户成功登录我们的应用程序后触发。**

以下是示例代码:

```java
@Bean
public ApplicationListener<AuthenticationSuccessEvent>
  authenticationSuccessListener( PasswordEncoder encoder) {
    return (AuthenticationSuccessEvent event) -> {
        Authentication auth = event.getAuthentication();

        if (auth instanceof UsernamePasswordAuthenticationToken
          && auth.getCredentials() != null) {

            CharSequence clearTextPass = (CharSequence) auth.getCredentials();
            String newPasswordHash = encoder.encode(clearTextPass);

            // [...] Update user's password

            ((UsernamePasswordAuthenticationToken) auth).eraseCredentials();
        }
    };
}
```

在前面的代码片段中:

*   我们从提供的身份验证详细信息中以明文形式检索了用户密码
*   使用新算法创建了新的密码哈希
*   从身份验证令牌中删除了明文密码

默认情况下，不可能以明文形式提取密码，因为 Spring Security 会尽快删除它。

因此，我们需要配置 Spring，让它保存密码的明文版本。

此外，我们需要注册我们的编码委托:

```java
@Configuration
public class PasswordStorageWebSecurityConfigurer {

    @Bean
    public AuthenticationManager authManager(HttpSecurity http) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder = 
            http.getSharedObject(AuthenticationManagerBuilder.class);
        authenticationManagerBuilder.eraseCredentials(false)
            .userDetailsService(getUserDefaultDetailsService())
            .passwordEncoder(passwordEncoder());
        return authenticationManagerBuilder.build();
    }

   // ...
}
```

## 5。结论

在这篇简短的文章中，我们讨论了 5.x 中一些新的密码编码特性。

我们还了解了如何配置多种密码编码算法来对密码进行编码。此外，我们探索了一种在不破坏现有密码编码的情况下更改密码编码的方法。

最后，我们描述了如何使用 Spring events 透明地更新加密的用户密码，允许我们无缝地改变我们的编码策略，而不会泄露给我们的用户。

最后，和往常一样，所有代码示例都可以在我们的 [GitHub 库](https://web.archive.org/web/20221222004958/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest-basic-auth)中找到。