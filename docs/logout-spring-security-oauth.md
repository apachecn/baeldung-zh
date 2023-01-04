# 在 OAuth 安全应用程序中注销

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/logout-spring-security-oauth>

## 1.概观

在这个快速教程中，我们将展示如何向 OAuth Spring 安全应用程序添加注销功能。

我们将看到实现这一点的几种方法。首先，我们将看到如何从 OAuth 应用程序中注销我们的 Keycloak 用户，如[用 OAuth2 创建 REST API](/web/20221004045854/https://www.baeldung.com/rest-api-spring-oauth2-angular)中所述，然后，使用我们前面看到的 [Zuul 代理](/web/20221004045854/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#zuul)。

我们将在 Spring Security 5 中使用 OAuth 堆栈。如果你想使用 Spring Security OAuth 遗留堆栈，看看之前的这篇文章:[在 OAuth 安全应用程序中注销(使用遗留堆栈)](/web/20221004045854/https://www.baeldung.com/logout-spring-security-oauth-legacy)。

## 2.使用前端应用程序注销

由于访问令牌是由授权服务器管理的，因此需要在这一级使它们无效。根据您使用的授权服务器的不同，具体步骤会略有不同。

在我们的例子中，根据 Keycloak [文档](https://web.archive.org/web/20221004045854/https://www.keycloak.org/docs/latest/securing_apps/#logout)，为了直接从浏览器应用退出，我们可以将浏览器重定向到`http://auth-server/auth/realms/{realm-name}/protocol/openid-connect/logout?redirect_uri=encodedRedirectUri`。

**在发送重定向 URI 的同时，我们还需要向 Keycloak 的[注销端点](https://web.archive.org/web/20221004045854/https://www.keycloak.org/docs-api/10.0/javadocs/org/keycloak/protocol/oidc/endpoints/LogoutEndpoint.html#logout-java.lang.String-java.lang.String-java.lang.String-java.lang.String-java.lang.String-)传递一个`id_token_hint`** 。这应该携带编码的`id_token`值。

让我们回忆一下我们是如何保存`access_token`的，我们将同样保存`id_token`:

```java
saveToken(token) {
  var expireDate = new Date().getTime() + (1000 * token.expires_in);
  Cookie.set("access_token", token.access_token, expireDate);
  Cookie.set("id_token", token.id_token, expireDate);
  this._router.navigate(['/']);
} 
```

**重要的是，为了获得授权服务器响应载荷中的 [ID 令牌](https://web.archive.org/web/20221004045854/https://www.oauth.com/oauth2-servers/openid-connect/id-tokens/)，我们应该在[范围参数](/web/20221004045854/https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service)** 中包含`openid`。

现在让我们来看看注销过程的运行情况。

我们将修改`[App Service](/web/20221004045854/https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service-1)`中的函数`logout`:

```java
logout() {
  let token = Cookie.get('id_token');
  Cookie.delete('access_token');
  Cookie.delete('id_token');
  let logoutURL = "http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/logout?
    id_token_hint=" + token + "&post;_logout_redirect_uri=" + this.redirectUri;

  window.location.href = logoutURL;
}
```

除了重定向，**我们还需要丢弃从授权服务器获得的访问和 ID 令牌**。

因此，在上面的代码中，我们首先删除了令牌，然后将浏览器重定向到 Keycloak 的`logout` API。

值得注意的是，我们将重定向 URI 作为`http://localhost:8089/` ——我们在整个应用程序中使用的那个——传入，所以我们将在注销后到达登录页面。

在授权服务器端执行对应于当前会话的访问、ID 和刷新令牌的删除。在这种情况下，我们的浏览器应用程序根本没有保存刷新令牌。

## 3。 **使用 Zuul 代理注销**

在关于处理刷新令牌的前一篇文章中，我们已经设置了我们的应用程序，能够使用刷新令牌来刷新访问令牌。这个实现使用了带有定制过滤器的 Zuul 代理。

在这里，我们将看到如何将注销功能添加到上面。

这一次，我们将利用另一个 Keycloak API 来注销用户。**我们将通过非浏览器调用**在`logout`端点上调用 POST 来注销会话，而不是我们在上一节中使用的 URL 重定向。

### 3.1.定义注销路线

首先，让我们为我们的 [`application.yml`](/web/20221004045854/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#zuul) 中的代理添加另一条路由:

```java
zuul:
  routes:
    //...
    auth/refresh/revoke:
      path: /auth/refresh/revoke/**
      sensitiveHeaders:
      url: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/logout

    //auth/refresh route
```

事实上，我们在已经存在的`auth/refresh`上增加了一条子路线。**重要的是，我们要在主路线之前添加子路线，否则，Zuul 将总是映射主路线**的 URL。

我们添加了一个子路由而不是主路由，以便访问 HTTP-only `refreshToken` cookie，[，它被设置为具有非常有限的路径](/web/20221004045854/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#extractToken)作为`/auth/refresh`(及其子路径)。我们将在下一节看到为什么我们需要 cookie。

### 3.2.发布到授权服务器的`/logout`

现在让我们增强`CustomPreZuulFilter`实现来拦截`/auth/refresh/revoke` URL，并添加必要的信息传递给授权服务器。

**注销需要的表单参数与[刷新令牌](/web/20221004045854/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#injectToken)请求的表单参数类似，只是没有`grant_type`** :

```java
@Component 
public class CustomPostZuulFilter extends ZuulFilter { 
    //... 
    @Override 
    public Object run() { 
        //...
        if (requestURI.contains("auth/refresh/revoke")) {
            String cookieValue = extractCookie(req, "refreshToken");
            String formParams = String.format("client_id=%s&client;_secret=%s&refresh;_token=%s", 
              CLIENT_ID, CLIENT_SECRET, cookieValue);
            bytes = formParams.getBytes("UTF-8");
        }
        //...
    }
}
```

这里，我们简单地提取了`refreshToken` cookie 并发送了所需的`formParams.`

### 3.3.删除刷新令牌

**正如我们前面看到的，当使用`logout`重定向撤销访问令牌时，与之相关联的刷新令牌也被授权服务器无效。**

然而，在这种情况下，`httpOnly` cookie 将在客户机上保持设置。鉴于我们不能通过 JavaScript 移除它，我们需要从服务器端移除它。

为此，让我们添加拦截`/auth/refresh/revoke` URL 的`CustomPostZuulFilter`实现，这样当遇到这个 URL 时，它将**删除`refreshToken` cookie** :

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    //...
    @Override
    public Object run() {
        //...
        String requestMethod = ctx.getRequest().getMethod();
        if (requestURI.contains("auth/refresh/revoke")) {
            Cookie cookie = new Cookie("refreshToken", "");
            cookie.setMaxAge(0);
            ctx.getResponse().addCookie(cookie);
        }
        //...
    }
}
```

### 3.4.从 Angular 客户端删除访问令牌

除了撤销刷新令牌之外，`access_token` cookie 也需要从客户端删除。

让我们为角度控制器添加一个方法，该方法清除`access_token` cookie 并调用`/auth/refresh/revoke` POST 映射:

```java
logout() {
  let headers = new HttpHeaders({
    'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});

  this._http.post('auth/refresh/revoke', {}, { headers: headers })
    .subscribe(
      data => {
        Cookie.delete('access_token');
        window.location.href = 'http://localhost:8089/';
        },
      err => alert('Could not logout')
    );
}
```

当点击注销按钮时，将调用此函数:

```java
<a class="btn btn-default pull-right"(click)="logout()" href="#">Logout</a>
```

## 4.结论

在这篇快速而深入的教程中，我们展示了如何让用户从一个`OAuth`安全的应用程序中注销，并使该用户的令牌失效。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221004045854/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-rest)