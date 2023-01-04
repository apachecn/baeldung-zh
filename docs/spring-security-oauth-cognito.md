# 使用 Spring Security 向 Amazon Cognito 认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-cognito>

## 1.介绍

在本教程中，我们将看看如何使用 [Spring Security](/web/20221206014514/https://www.baeldung.com/security-spring) 的 OAuth 2.0 支持与 [Amazon Cognito](https://web.archive.org/web/20221206014514/https://aws.amazon.com/cognito/) 进行认证。

在这个过程中，我们将简要地了解一下 Amazon Cognito 是什么以及它支持什么样的 [OAuth 2.0](https://web.archive.org/web/20221206014514/https://oauth.net/2/) 流。

最后，我们将有一个简单的单页应用程序。没什么特别的。

## 2.亚马逊 Cognito 是什么？

**Cognito 是一种用户身份和数据同步服务**，它让我们可以轻松地跨多种设备管理应用程序的用户数据。

借助 Amazon Cognito，我们可以:

*   **为我们的应用程序创建、验证和授权用户**
*   为使用谷歌、脸书或 Twitter 等其他公共身份提供商的用户创建身份
*   以键值对的形式保存我们应用的用户数据

## 3.设置

### 3.1 .亚马逊认知设置

作为身份提供者，Cognito 支持 [`authorization_code, implicit,` 和`client_credentials`授权](https://web.archive.org/web/20221206014514/https://oauth.net/2/grant-types/)。出于我们的目的，让我们设置使用`authorization_code`授权类型。

首先，我们需要一点认知设置:

*   [创建用户池](https://web.archive.org/web/20221206014514/https://docs.aws.amazon.com/cognito/latest/developerguide/tutorial-create-user-pool.html)
*   [添加一个用户](https://web.archive.org/web/20221206014514/https://docs.aws.amazon.com/cognito/latest/developerguide/signing-up-users-in-your-app.html)——我们将使用这个用户登录我们的 Spring 应用程序
*   [创建应用客户端](https://web.archive.org/web/20221206014514/https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-client-apps.html)
*   [配置 App 客户端](https://web.archive.org/web/20221206014514/https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-app-idp-settings.html)

在应用程序客户端的配置中，**确保`CallbackURL`与 Spring 配置文件中的`redirect-uri`** 相匹配。在我们的案例中，这将是:

```
http://localhost:8080/login/oauth2/code/cognito
```

`Allowed OAuth flow`应该是`Authorization code grant.` 然后，在同一个页面上`,`我们需要设置`Allowed OAuth scope`为`openid.`

为了将用户重定向到 Cognito 的自定义登录页面，我们还需要添加一个 [`User Pool Domain`](https://web.archive.org/web/20221206014514/https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-assign-domain.html) 。

### 3.2.弹簧设置

由于我们想要使用 OAuth 2.0 登录，我们需要添加[spring-security-OAuth 2-client](https://web.archive.org/web/20221206014514/https://search.maven.org/search?q=a:spring-security-oauth2-client%20g:org.springframework.security)和[spring-security-OAuth 2-Jose](https://web.archive.org/web/20221206014514/https://search.maven.org/search?q=a:spring-security-oauth2-jose%20g:org.springframework.security)依赖项到我们的应用程序:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-jose</artifactId>
</dependency>
```

然后，我们需要一些配置来将所有东西绑定在一起:

```
spring:
  security:
    oauth2:
      client:
        registration:
          cognito:
            clientId: clientId
            clientSecret: clientSecret
            scope: openid
            redirect-uri: http://localhost:8080/login/oauth2/code/cognito
            clientName: clientName
        provider:
          cognito:
            issuerUri: https://cognito-idp.{region}.amazonaws.com/{poolId}
            user-name-attribute: cognito:username
```

在上面的配置中，属性`clientId`、`clientSecret`、`clientName`和`issuerUri`应该根据我们在 AWS 上创建的`User Pool`和`App Client`来填充。

有了这个，我们就应该有 Spring 和 Amazon Cognito 了！本教程的其余部分定义了我们的应用程序的安全配置，然后只是整理了一些细节。

### 3.3.Spring 安全配置

现在我们将添加一个安全配置类:

```
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf()
            .and()
            .authorizeRequests(authz -> authz.mvcMatchers("/")
                .permitAll()
                .anyRequest()
                .authenticated())
            .oauth2Login()
            .and()
            .logout()
            .logoutSuccessUrl("/");
        return http.build();
    }
}
```

在这里，我们首先指定我们需要防止 CSRF 攻击，然后允许每个人访问我们的登录页面。之后，我们添加了对`oauth2Login`的调用，以连接 Cognito 客户端注册。

## 4.添加登录页面

接下来，我们添加一个简单的[百里香叶](/web/20221206014514/https://www.baeldung.com/thymeleaf-in-spring-mvc)登录页面，这样我们就知道自己何时登录:

```
<div>
    <h1 class="title">OAuth 2.0 Spring Security Cognito Demo</h1>
    <div sec:authorize="isAuthenticated()">
        <div class="box">
            Hello, <strong th:text="${#authentication.name}"></strong>!
        </div>
    </div>
    <div sec:authorize="isAnonymous()">
        <div class="box">
            <a class="button login is-primary" th:href="@{/oauth2/authorization/cognito}">
              Log in with Amazon Cognito</a>
        </div>
    </div>
</div>
```

简单地说，这将在我们登录时显示我们的用户名，或者在我们未登录时显示一个登录链接。请密切注意链接的样子，因为**从我们的配置文件中提取了`cognito`部分。**

然后让我们确保**我们将应用程序根绑定到我们的欢迎页面:**

```
@Configuration
public class CognitoWebConfiguration implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

## 5.运行应用程序

这是一个将与 auth 相关的所有东西都投入运行的类:

```
@SpringBootApplication
public class SpringCognitoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringCognitoApplication.class, args);
    }
}
```

**现在我们可以启动我们的应用程序**，转到`[http://localhost:8080](https://web.archive.org/web/20221206014514/http://localhost:8080/),`并点击登录链接。在输入我们在 AWS 上创建的用户的凭证时，我们应该能够看到一条`Hello, username`消息。

## 6.结论

在本教程中，我们看了如何通过一些简单的配置将 Spring Security 与 Amazon Cognito 集成。然后我们用几段代码把所有的东西组合在一起。

和往常一样，本文中的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221206014514/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-cognito)