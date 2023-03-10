# Spring Security OAuth2 中的新功能—验证声明

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-2-verify-claims>

## 1。概述

在这个快速教程中，我们将使用 Spring Security OAuth2 实现，我们将学习如何使用新的`JwtClaimsSetVerifier`——在[Spring Security OAuth 2 . 2 . 0 . release](https://web.archive.org/web/20220629012419/https://spring.io/blog/2017/07/28/spring-security-oauth-2-2-released)中引入——来验证 JWT 声明。

## 2。Maven 配置

首先，我们需要将最新版本的`spring-security-oauth2`添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

## 3。令牌存储配置

接下来，让我们在资源服务器中配置我们的`TokenStore`:

```java
@Bean
public TokenStore tokenStore() {
    return new JwtTokenStore(accessTokenConverter());
}

@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("123");
    converter.setJwtClaimsSetVerifier(jwtClaimsSetVerifier());
    return converter;
}
```

注意我们是如何将新的验证器添加到我们的`JwtAccessTokenConverter`中的。

关于如何配置`JwtTokenStore`的更多细节，请查看关于使用 JWT 和 Spring Security OAuth 的文章。

现在，在下面的部分中，我们将讨论不同类型的声明验证器，以及如何让它们一起工作。

## 4。`IssuerClaimVerifier`

我们从简单的开始，通过使用`IssuerClaimVerifier`验证发卡行的`iss`声明，如下所示:

```java
@Bean
public JwtClaimsSetVerifier issuerClaimVerifier() {
    try {
        return new IssuerClaimVerifier(new URL("http://localhost:8081"));
    } catch (MalformedURLException e) {
        throw new RuntimeException(e);
    }
}
```

在这个例子中，我们添加了一个简单的`IssuerClaimVerifier`来验证我们的发行者。如果 JWT 令牌包含发行者“iss”声明的不同值，将抛出一个简单的`InvalidTokenException`。

当然，如果令牌确实包含发行者“iss”声明，则不会抛出异常，令牌被视为有效。

## 5。自定义声明验证器

但是，有趣的是，我们还可以构建我们的自定义声明验证器:

```java
@Bean
public JwtClaimsSetVerifier customJwtClaimVerifier() {
    return new CustomClaimVerifier();
}
```

下面是一个简单的实现——检查`user_name`声明是否存在于我们的 JWT 令牌中:

```java
public class CustomClaimVerifier implements JwtClaimsSetVerifier {
    @Override
    public void verify(Map<String, Object> claims) throws InvalidTokenException {
        String username = (String) claims.get("user_name");
        if ((username == null) || (username.length() == 0)) {
            throw new InvalidTokenException("user_name claim is empty");
        }
    }
}
```

请注意，我们在这里只是简单地实现了`JwtClaimsSetVerifier`接口，然后为 verify 方法提供了一个完全定制的实现——这为我们提供了所需的任何类型检查的充分灵活性。

## 6。组合多个声明验证器

最后，让我们看看如何使用`DelegatingJwtClaimsSetVerifier`组合多个声明验证器，如下所示:

```java
@Bean
public JwtClaimsSetVerifier jwtClaimsSetVerifier() {
    return new DelegatingJwtClaimsSetVerifier(Arrays.asList(
      issuerClaimVerifier(), customJwtClaimVerifier()));
}
```

`DelegatingJwtClaimsSetVerifier`接受一个`JwtClaimsSetVerifier`对象的列表，并将声明验证过程委托给这些验证者。

## 7。简单集成测试

现在我们已经完成了实现，让我们用一个简单的[集成测试](https://web.archive.org/web/20220629012419/https://github.com/Baeldung/spring-security-oauth/blob/master/oauth-legacy/oauth-resource-server-legacy-2/src/test/java/com/baeldung/test/JwtClaimsVerifierIntegrationTest.java)来测试我们的声明验证器:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
  classes = ResourceServerApplication.class, 
  webEnvironment = WebEnvironment.RANDOM_PORT)
public class JwtClaimsVerifierIntegrationTest {

    @Autowired
    private JwtTokenStore tokenStore;

    ...
}
```

我们将从一个不包含发行者(但包含一个`user_name`)的令牌开始，它应该是有效的:

```java
@Test
public void whenTokenDontContainIssuer_thenSuccess() {
    String tokenValue = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

这是有效的原因很简单——只有当令牌中存在发行者声明时，第一个验证者才是活动的。如果该声明不存在，验证者不会介入。

接下来，让我们看看一个令牌，它包含一个有效的发布者(`http://localhost:8081`)和一个`user_name`。这也应该是有效的:

```java
@Test
public void whenTokenContainValidIssuer_thenSuccess() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

当令牌包含无效的颁发者(`http://localhost:8082`)时，它将被验证并确定为无效:

```java
@Test(expected = InvalidTokenException.class)
public void whenTokenContainInvalidIssuer_thenException() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

接下来，当令牌不包含`user_name`声明时，它将是无效的:

```java
@Test(expected = InvalidTokenException.class)
public void whenTokenDontContainUsername_thenException() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

最后，当令牌包含空的`user_name`声明时，它也是无效的:

```java
@Test(expected = InvalidTokenException.class)
public void whenTokenContainEmptyUsername_thenException() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

## 8。结论

在这篇简短的文章中，我们了解了 Spring Security OAuth 中新的验证器功能。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220629012419/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy)