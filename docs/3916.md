# 在 OAuth 安全应用程序中注销(使用 Spring Security OAuth 遗留堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/logout-spring-security-oauth-legacy>

## 1。概述

在这个快速教程中，我们将展示如何向 OAuth Spring 安全应用程序添加注销功能。

当然，我们将使用前一篇文章中描述的 OAuth 应用程序——[用 OAuth2](/web/20220628155641/https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy) 创建 REST API。

**注**:本文使用的是 [Spring OAuth 遗留项目](https://web.archive.org/web/20220628155641/https://spring.io/projects/spring-security-oauth)。关于本文使用新的 Spring Security 5 堆栈的版本，请看我们的文章“OAuth 安全应用程序中的 T4 注销”。

## 2。移除访问令牌

简单地说，在 OAuth 安全的环境中注销涉及到**使用户的访问令牌无效**——因此它不能再被使用。

在基于`JdbcTokenStore-`的实现中，这意味着从`TokenStore`中移除令牌。

让我们为令牌实现一个删除操作。我们将在这里使用 parimary `/oauth/token` URL 结构，并简单地为它引入一个新的删除操作。

现在，因为我们实际上在这里使用了`/oauth/token`URI——我们需要小心处理它。我们不能简单地将它添加到任何控制器中——因为框架已经有映射到 URI 的操作——使用 POST 和 GET。

相反，我们需要做的是定义这是一个`@FrameworkEndpoint –` ，以便它被`FrameworkEndpointHandlerMapping`而不是标准的`RequestMappingHandlerMapping`拾取和解析。这样我们就不会遇到任何部分匹配，也不会有任何冲突:

```
@FrameworkEndpoint
public class RevokeTokenEndpoint {

    @Resource(name = "tokenServices")
    ConsumerTokenServices tokenServices;

    @RequestMapping(method = RequestMethod.DELETE, value = "/oauth/token")
    @ResponseBody
    public void revokeToken(HttpServletRequest request) {
        String authorization = request.getHeader("Authorization");
        if (authorization != null && authorization.contains("Bearer")){
            String tokenId = authorization.substring("Bearer".length()+1);
            tokenServices.revokeToken(tokenId);
        }
    }
}
```

注意我们是如何从请求中提取令牌的，只需使用标准的`Authorization`头。

## 3。移除刷新令牌

在之前关于[处理刷新令牌](/web/20220628155641/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular-js-legacy)的文章中，我们已经设置我们的应用程序能够使用刷新令牌来刷新访问令牌。这个实现使用了一个 Zuul 代理——用一个`CustomPostZuulFilter`将从授权服务器收到的`refresh_token`值添加到一个`refreshToken` cookie 中。

如前一节所示，在撤销访问令牌时，与之关联的刷新令牌也会失效。然而，`httpOnly` cookie 将在客户端保持设置，因为我们不能通过 JavaScript 删除它——所以我们需要从服务器端删除它。

让我们增强拦截`/oauth/token/revoke` URL 的`CustomPostZuulFilter`实现，以便它在遇到这个 URL 时删除`refreshToken` cookie:

```
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    //...
    @Override
    public Object run() {
        //...
        String requestMethod = ctx.getRequest().getMethod();
        if (requestURI.contains("oauth/token") && requestMethod.equals("DELETE")) {
            Cookie cookie = new Cookie("refreshToken", "");
            cookie.setMaxAge(0);
            cookie.setPath(ctx.getRequest().getContextPath() + "/oauth/token");
            ctx.getResponse().addCookie(cookie);
        }
        //...
    }
}
```

## 4。从 AngularJS 客户端删除访问令牌

除了从令牌存储中撤销访问令牌之外，`access_token` cookie 也需要从客户端删除。

让我们给我们的`AngularJS`控制器添加一个方法，该方法清除`access_token` cookie 并调用`/oauth/token/revoke`删除映射:

```
$scope.logout = function() {
    logout($scope.loginData);
}
function logout(params) {
    var req = {
        method: 'DELETE',
        url: "oauth/token"
    }
    $http(req).then(
        function(data){
            $cookies.remove("access_token");
            window.location.href="login";
        },function(){
            console.log("error");
        }
    );
}
```

点击`Logout`链接时将调用该函数:

```
<a class="btn btn-info" href="#" ng-click="logout()">Logout</a>
```

## 5。结论

在这篇快速而深入的教程中，我们展示了如何让用户从一个`OAuth`安全的应用程序中注销，并使该用户的令牌失效。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628155641/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy)