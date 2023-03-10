# 在 Spring 中获取 Keycloak 用户 ID

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-keycloak-get-user-id>

## 1.概观

Keycloak 是一个开源的身份和访问管理(IAM)系统，可以很好地与 Spring Boot 应用程序集成。在本教程中，我们将描述如何在 Spring Boot 应用程序中获取 Keycloak 用户 ID。

## 2.问题陈述

Keycloak 提供了一些特性，比如保护 REST API、用户联盟、细粒度授权、社交登录、双因素身份验证(2FA)等。此外，我们可以用它来实现使用 OpenID Connect()的单点登录( [SSO](/web/20220811170118/https://www.baeldung.com/java-sso-solutions) )。**让我们假设我们有一个由 OIDC 使用 Keycloak 保护的 Spring Boot 应用程序，我们想在 Spring Boot 应用程序中获得一个用户 ID。在这种情况下，我们需要在 Spring Boot 应用程序中获得一个访问令牌或安全上下文。**

### 2.1.Keycloak 服务器作为授权服务器

为了简单起见，我们将使用嵌入在 Spring Boot 应用程序中的 [Keycloak。让我们假设我们正在使用 GitHub](/web/20220811170118/https://www.baeldung.com/keycloak-embedded-in-spring-boot-app) 上可用的授权服务器项目[。首先，我们将在我们的嵌入式 Keycloak 服务器](https://web.archive.org/web/20220811170118/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-resource-server/authorization-server) [![](img/c946eac2d2043de44afb9bffb024116c.png)](/web/20220811170118/https://www.baeldung.com/wp-content/uploads/2022/05/keycloak-spring-boot.png) 中定义领域`baeldung`中的`customerClient`客户端，然后，我们将领域细节导出为`customer-realm.json`，并在我们的`application-customer.yml`中设置领域文件:

```java
keycloak:
  server:
    contextPath: /auth
    adminUser:
      username: bael-admin
      password: pass
    realmImportFile: customer-realm.json
```

最后，我们可以使用`–spring.profiles.active=customer`选项运行应用程序。现在，授权服务器准备好了。运行服务器后，我们可以在`http://localhost:8083/auth/.`访问授权服务器的欢迎页面

### 2.2.资源服务器

现在我们已经配置了授权服务器，让我们设置资源服务器。为此，我们将使用 GitHub 上的资源服务器项目[。首先，让我们添加`application-embedded.properties`文件作为资源:](/web/20220811170118/https://www.baeldung.com/spring-boot-keycloak)

```java
keycloak.auth-server-url=http://localhost:8083/auth
keycloak.realm=baeldung
keycloak.resource=customerClient
keycloak.public-client=true
keycloak.principal-attribute=preferred_username
```

现在，使用 OAuth2 授权服务器的资源服务器是安全的，我们必须登录到 SSO 服务器来访问资源。我们可以使用`–spring.profiles.active=embedded`选项运行应用程序。

## 3.获取 Keycloak 用户 ID

从 Keycloak 获取用户 ID 有几种方法:使用访问令牌或客户端映射器。

### 3.1.通过访问令牌

在 [Spring Boot 应用程序](/web/20220811170118/https://www.baeldung.com/keycloak-custom-user-attributes) `CustomUserAttrController`类的基础上，让我们修改`getUserInfo()`方法来获取用户 ID:

```java
@GetMapping(path = "/users")
public String getUserInfo(Model model) {

    KeycloakAuthenticationToken authentication = 
      (KeycloakAuthenticationToken) SecurityContextHolder.getContext().getAuthentication();

    Principal principal = (Principal) authentication.getPrincipal();

    String userIdByToken = "";

    if (principal instanceof KeycloakPrincipal) {
        KeycloakPrincipal<KeycloakSecurityContext> kPrincipal = (KeycloakPrincipal<KeycloakSecurityContext>) principal;
        IDToken token = kPrincipal.getKeycloakSecurityContext().getIdToken();
        userIdByToken = token.getSubject();
    }

    model.addAttribute("userIDByToken", userIdByToken);
    return "userInfo";
} 
```

正如我们所看到的，首先我们从`KeycloakAuthenticationToken`类中获得了`Principal`。然后，我们从`IDToken using the`方法中提取用户 ID。

### 3.2.按客户端映射器

我们可以在客户端映射器中添加一个用户 ID，并在 Spring Boot 应用程序中获取它。首先，我们在`customerClient` 客户端: [![](img/e75b95592787b36d6d0f02b5fa31f229.png)](/web/20220811170118/https://www.baeldung.com/wp-content/uploads/2022/05/keycloak-spring-boot-2.png) 中定义一个客户端映射器，然后，我们在`CustomUserAttrController`类中获取用户 ID:

```java
@GetMapping(path = "/users")
public String getUserInfo(Model model) {

    KeycloakAuthenticationToken authentication = 
      (KeycloakAuthenticationToken) SecurityContextHolder.getContext().getAuthentication();

    Principal principal = (Principal) authentication.getPrincipal();

    String userIdByMapper = "";

    if (principal instanceof KeycloakPrincipal) {
        KeycloakPrincipal<KeycloakSecurityContext> kPrincipal = (KeycloakPrincipal<KeycloakSecurityContext>) principal;
        IDToken token = kPrincipal.getKeycloakSecurityContext().getIdToken();
        userIdByMapper = token.getOtherClaims().get("user_id").toString();
    }

    model.addAttribute("userIDByMapper", userIdByMapper);
    return "userInfo";
} 
```

我们使用来自`IDToken`的`getOtherClaims()`方法来获取映射器。然后，我们将用户 ID 添加到模型属性中。

### 3.3.百里香叶

我们将修改`userInfo.html` 模板来显示用户 ID 信息:

```java
<div id="container">
    <h1>
	User ID By Token: <span th:text="${userIDByToken}">--userID--</span>.
    </h1>
    <h1>
        User ID By Mapper: <span th:text="${userIDByMapper}">--userID--</span>.
    </h1>
</div>
```

### 3.4.试验

运行应用程序后，我们可以导航到`http://localhost:8081/users`。输入`baeldung:baeldung`作为凭证，将返回以下内容:

[![](img/2f105f1a7ad6f8fc0530175fbe9d080b.png)](/web/20220811170118/https://www.baeldung.com/wp-content/uploads/2022/05/keycloak-spring-boot-3.png)

## 4.结论

在本文中，我们研究了在 Spring Boot 应用程序中从 Keycloak 获取用户 ID。我们首先设置调用安全应用程序所需的环境。然后，我们描述了使用`IDToken`和客户端映射器在 Spring Boot 应用程序中获取 Keycloak 用户 ID。和往常一样，本教程的完整源代码可以在 GitHub 上找到。此外，授权服务器源代码[可以在 GitHub](https://web.archive.org/web/20220811170118/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-resource-server/authorization-server) 上获得。