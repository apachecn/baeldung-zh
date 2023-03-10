# Spring REST API 的 OAuth2 处理 AngularJS(遗留 OAuth 堆栈)中的刷新令牌

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular-js-legacy>

## 1。概述

在本教程中，我们将继续探索 OAuth 密码流，这是我们在上一篇文章中开始整理的，我们将重点关注如何在 AngularJS 应用程序中处理刷新令牌。

**注**:本文使用的是 [Spring OAuth 遗留项目](https://web.archive.org/web/20220703141848/https://spring.io/projects/spring-security-oauth)。**关于本文使用新的 Spring Security 5 堆栈的版本，请看我们的文章[oauth 2 For a Spring REST API——处理 Angular 中的刷新令牌](/web/20220703141848/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular)。**

## 2。访问令牌到期

首先，请记住，当用户登录到应用程序时，客户端正在获取访问令牌:

```java
function obtainAccessToken(params) {
    var req = {
        method: 'POST',
        url: "oauth/token",
        headers: {"Content-type": "application/x-www-form-urlencoded; charset=utf-8"},
        data: $httpParamSerializer(params)
    }
    $http(req).then(
        function(data) {
            $http.defaults.headers.common.Authorization= 'Bearer ' + data.data.access_token;
            var expireDate = new Date (new Date().getTime() + (1000 * data.data.expires_in));
            $cookies.put("access_token", data.data.access_token, {'expires': expireDate});
            window.location.href="index";
        },function() {
            console.log("error");
            window.location.href = "login";
        });   
}
```

请注意我们的访问令牌是如何存储在一个 cookie 中的，这个 cookie 将根据令牌本身的过期时间而过期。

需要理解的重要一点是**cookie 本身仅用于存储**,它不驱动 OAuth 流中的任何其他东西。例如，浏览器永远不会自动向服务器发送带有请求的 cookie。

还要注意我们实际是如何调用这个`obtainAccessToken()`函数的:

```java
$scope.loginData = {
    grant_type:"password", 
    username: "", 
    password: "", 
    client_id: "fooClientIdPassword"
};

$scope.login = function() {   
    obtainAccessToken($scope.loginData);
}
```

## 3。代理人

我们现在将有一个 Zuul 代理运行在前端应用程序中，基本上位于前端客户端和授权服务器之间。

让我们配置代理的路由:

```java
zuul:
  routes:
    oauth:
      path: /oauth/**
      url: http://localhost:8081/spring-security-oauth-server/oauth
```

有趣的是，我们只是将流量代理到授权服务器，而不是其他任何东西。我们只在客户端获得新令牌时才真正需要代理。

如果你想复习 Zuul 的基础知识，可以快速阅读 Zuul 的主要文章。

## 4。执行基本认证的 Zuul 过滤器

代理的第一个用途很简单——我们将使用 Zuul 预过滤器添加授权头来访问令牌请求，而不是在 javascript 中显示我们的应用程序“`client secret`”:

```java
@Component
public class CustomPreZuulFilter extends ZuulFilter {
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        if (ctx.getRequest().getRequestURI().contains("oauth/token")) {
            byte[] encoded;
            try {
                encoded = Base64.encode("fooClientIdPassword:secret".getBytes("UTF-8"));
                ctx.addZuulRequestHeader("Authorization", "Basic " + new String(encoded));
            } catch (UnsupportedEncodingException e) {
                logger.error("Error occured in pre filter", e);
            }
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public int filterOrder() {
        return -2;
    }

    @Override
    public String filterType() {
        return "pre";
    }
}
```

现在请记住，这不会增加任何额外的安全性，我们这样做的唯一原因是因为令牌端点通过使用客户端凭据的基本身份验证来保护。

从实现的角度来看，过滤器的类型尤其值得注意。在传递请求之前，我们使用“pre”过滤器类型来处理请求。

## 5。将刷新令牌放入 Cookie

开始有趣的事情。

我们在这里计划做的是让客户机以 cookie 的形式获得刷新令牌。不仅仅是一个普通的 cookie，而是一个安全的、只有 HTTP 的 cookie，具有非常有限的路径(`/oauth/token`)。

我们将设置一个 Zuul 后置过滤器，从响应的 JSON 主体中提取刷新令牌，并将其设置在 cookie 中:

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    private ObjectMapper mapper = new ObjectMapper();

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        try {
            InputStream is = ctx.getResponseDataStream();
            String responseBody = IOUtils.toString(is, "UTF-8");
            if (responseBody.contains("refresh_token")) {
                Map<String, Object> responseMap = mapper.readValue(
                  responseBody, new TypeReference<Map<String, Object>>() {});
                String refreshToken = responseMap.get("refresh_token").toString();
                responseMap.remove("refresh_token");
                responseBody = mapper.writeValueAsString(responseMap);

                Cookie cookie = new Cookie("refreshToken", refreshToken);
                cookie.setHttpOnly(true);
                cookie.setSecure(true);
                cookie.setPath(ctx.getRequest().getContextPath() + "/oauth/token");
                cookie.setMaxAge(2592000); // 30 days
                ctx.getResponse().addCookie(cookie);
            }
            ctx.setResponseBody(responseBody);
        } catch (IOException e) {
            logger.error("Error occured in zuul post filter", e);
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public int filterOrder() {
        return 10;
    }

    @Override
    public String filterType() {
        return "post";
    }
}
```

这里需要了解一些有趣的事情:

*   我们使用一个 Zuul 后置过滤器来读取响应和**提取刷新令牌**
*   我们从 JSON 响应中移除了`refresh_token`的值，以确保 cookie 之外的前端永远无法访问它
*   我们将 cookie 的最大年龄设置为 **30 天**，因为这与令牌的到期时间相匹配

为了增加一层额外的保护来抵御 CSRF 攻击，**我们将为我们所有的 cookie**添加一个[同站点 cookie](https://web.archive.org/web/20220703141848/https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00) 头。

为此，我们将创建一个配置类:

```java
@Configuration
public class SameSiteConfig implements WebMvcConfigurer {
    @Bean
    public TomcatContextCustomizer sameSiteCookiesConfig() {
        return context -> {
            final Rfc6265CookieProcessor cookieProcessor = new Rfc6265CookieProcessor();
            cookieProcessor.setSameSiteCookies(SameSiteCookies.STRICT.getValue());
            context.setCookieProcessor(cookieProcessor);
        };
    }
}
```

这里我们将属性设置为`strict`，这样 cookies 的任何跨站点传输都会被严格禁止。

## 6。从 Cookie 中获取并使用刷新令牌

现在我们在 cookie 中有了刷新令牌，当前端 AngularJS 应用程序试图触发令牌刷新时，它将在`/oauth/token`发送请求，因此浏览器当然会发送该 cookie。

因此，我们现在将在代理中有另一个过滤器，它将从 cookie 中提取刷新令牌，并将其作为 HTTP 参数转发，这样请求就是有效的:

```java
public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    ...
    HttpServletRequest req = ctx.getRequest();
    String refreshToken = extractRefreshToken(req);
    if (refreshToken != null) {
        Map<String, String[]> param = new HashMap<String, String[]>();
        param.put("refresh_token", new String[] { refreshToken });
        param.put("grant_type", new String[] { "refresh_token" });
        ctx.setRequest(new CustomHttpServletRequest(req, param));
    }
    ...
}

private String extractRefreshToken(HttpServletRequest req) {
    Cookie[] cookies = req.getCookies();
    if (cookies != null) {
        for (int i = 0; i < cookies.length; i++) {
            if (cookies[i].getName().equalsIgnoreCase("refreshToken")) {
                return cookies[i].getValue();
            }
        }
    }
    return null;
}
```

这里是我们的`CustomHttpServletRequest` ——用来**注入我们的刷新令牌参数**:

```java
public class CustomHttpServletRequest extends HttpServletRequestWrapper {
    private Map<String, String[]> additionalParams;
    private HttpServletRequest request;

    public CustomHttpServletRequest(
      HttpServletRequest request, Map<String, String[]> additionalParams) {
        super(request);
        this.request = request;
        this.additionalParams = additionalParams;
    }

    @Override
    public Map<String, String[]> getParameterMap() {
        Map<String, String[]> map = request.getParameterMap();
        Map<String, String[]> param = new HashMap<String, String[]>();
        param.putAll(map);
        param.putAll(additionalParams);
        return param;
    }
}
```

同样，这里有许多重要的实施注意事项:

*   代理正在从 Cookie 中提取刷新令牌
*   然后将它设置到`refresh_token`参数中
*   它还将`grant_type`设置为`refresh_token`
*   如果没有`refreshToken` cookie(过期或首次登录)，那么访问令牌请求将被重定向，不做任何更改

## 7。从 AngularJS 刷新访问令牌

最后，让我们修改简单的前端应用程序，并实际利用刷新令牌:

下面是我们的函数`refreshAccessToken()`:

```java
$scope.refreshAccessToken = function() {
    obtainAccessToken($scope.refreshData);
}
```

这里是我们的`$scope.refreshData`:

```java
$scope.refreshData = {grant_type:"refresh_token"};
```

请注意我们是如何简单地使用现有的`obtainAccessToken` 函数的——只是向它传递不同的输入。

还要注意，我们没有自己添加`refresh_token`——因为这将由 Zuul 过滤器来处理。

## 8。结论

在本 OAuth 教程中，我们学习了如何在 AngularJS 客户端应用程序中存储刷新令牌，如何刷新过期的访问令牌，以及如何利用 Zuul 代理来完成所有这些工作。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220703141848/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy "The Full Registration/Authentication Example Project on Github ")中找到。