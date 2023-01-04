# Spring REST API 的 OAuth2 处理 Angular 中的刷新令牌

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular>

 ![](img/9a85e7b00890e39c43fce777070ff81b.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220521221350/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在本教程中，我们将继续探索我们在[上一篇文章](/web/20220521221350/https://www.baeldung.com/rest-api-spring-oauth2-angular)和**中开始整理的 OAuth2 授权代码流，我们将重点关注如何在 Angular 应用程序中处理刷新令牌。我们也会利用祖尔的代理人。**

我们将在 Spring Security 5 中使用 OAuth 堆栈。如果你想使用 Spring Security OAuth 遗留堆栈，看看之前的文章:[OAuth 2 for a Spring REST API——处理 AngularJS(遗留 OAuth 堆栈)中的刷新令牌](/web/20220521221350/https://www.baeldung.com/spring-security-oauth2-refresh-token-angular-js-legacy)

## 2。访问令牌到期

首先，请记住，客户端分两步使用授权码授权类型来获取访问令牌。在第一步中，[我们获得授权码](/web/20220521221350/https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service)。在第二步，我们实际上[获得了访问令牌](/web/20220521221350/https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service-1)。

我们的访问令牌存储在一个 cookie 中，该 cookie 将根据令牌本身的过期时间而过期:

```
var expireDate = new Date().getTime() + (1000 * token.expires_in);
Cookie.set("access_token", token.access_token, expireDate);
```

需要理解的重要一点是**cookie 本身仅用于存储**,它不驱动 OAuth2 流中的任何其他内容。例如，浏览器永远不会自动向服务器发送带有请求的 cookie，所以我们在这里是安全的。

但是请注意我们实际上是如何定义这个`retrieveToken()`函数来获取访问令牌的:

```
retrieveToken(code) {
  let params = new URLSearchParams();
  params.append('grant_type','authorization_code');
  params.append('client_id', this.clientId);
  params.append('client_secret', 'newClientSecret');
  params.append('redirect_uri', this.redirectUri);
  params.append('code',code);

  let headers =
    new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});

  this._http.post('http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token',
    params.toString(), { headers: headers })
    .subscribe(
      data => this.saveToken(data),
      err => alert('Invalid Credentials'));
}
```

我们在`params`中发送客户端秘密，这并不是一种安全的处理方式。让我们看看如何避免这样做。

## 3。代理人

因此，**我们现在将有一个 Zuul 代理运行在前端应用程序中，基本上位于前端客户端和授权服务器之间**。所有敏感信息都将在这一层处理。

前端客户端现在将作为引导应用程序托管，以便我们可以使用 Spring Cloud Zuul starter 无缝连接到我们的嵌入式 Zuul 代理。 ****

如果你想复习 Zuul 的基础知识，可以快速阅读 Zuul 的主要文章。

现在**让我们配置代理**的路由:

```
zuul:
  routes:
    auth/code:
      path: /auth/code/**
      sensitiveHeaders:
      url: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/auth
    auth/token:
      path: /auth/token/**
      sensitiveHeaders:
      url: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token
    auth/refresh:
      path: /auth/refresh/**
      sensitiveHeaders:
      url: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token
    auth/redirect:
      path: /auth/redirect/**
      sensitiveHeaders:
      url: http://localhost:8089/
    auth/resources:
      path: /auth/resources/**
      sensitiveHeaders:
      url: http://localhost:8083/auth/resources/
```

我们已经设置了处理以下事项的路线:

*   `auth/code`–获取授权码并保存在 cookie 中
*   `auth/redirect`–处理到授权服务器登录页面的重定向
*   `auth/resources`–映射到授权服务器登录页面资源的对应路径(`css`和`js`)
*   `auth/token`–获取访问令牌，从有效负载中移除`refresh_token`并将其保存在 cookie 中
*   `auth/refresh`–获取刷新令牌，将其从有效负载中移除并保存在 cookie 中

有趣的是，我们只是将流量代理到授权服务器，而不是其他任何东西。我们只在客户端获得新令牌时才真正需要代理。

接下来，我们一个一个来看这些。

## 4。使用 Zuul 预过滤器获取代码

**代理的第一次使用很简单——我们建立一个获取授权码的请求:**

```
@Component
public class CustomPreZuulFilter extends ZuulFilter {
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest req = ctx.getRequest();
        String requestURI = req.getRequestURI();
        if (requestURI.contains("auth/code")) {
            Map<String, List> params = ctx.getRequestQueryParams();
            if (params == null) {
	        params = Maps.newHashMap();
	    }
            params.put("response_type", Lists.newArrayList(new String[] { "code" }));
            params.put("scope", Lists.newArrayList(new String[] { "read" }));
            params.put("client_id", Lists.newArrayList(new String[] { CLIENT_ID }));
            params.put("redirect_uri", Lists.newArrayList(new String[] { REDIRECT_URL }));
            ctx.setRequestQueryParams(params);
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        boolean shouldfilter = false;
        RequestContext ctx = RequestContext.getCurrentContext();
        String URI = ctx.getRequest().getRequestURI();

        if (URI.contains("auth/code") || URI.contains("auth/token") || 
          URI.contains("auth/refresh")) {		
            shouldfilter = true;
	}
        return shouldfilter;
    }

    @Override
    public int filterOrder() {
        return 6;
    }

    @Override
    public String filterType() {
        return "pre";
    }
}
```

我们使用一个过滤器类型`pre`在传递请求之前处理它。

**在过滤器的`run()`方法中，我们为`response_type`、`scope`、`client_id`和`redirect_uri`、**添加了查询参数——我们的授权服务器将我们带到其登录页面并发回代码所需的一切。

还要注意`shouldFilter()`方法。我们只使用提到的 3 个 URIs 过滤请求，其他的不使用`run` 方法。

## 5。使用 **Zuul Post 过滤器**将代码放入 Cookie

 **我们在这里计划做的是将代码保存为 cookie，这样我们就可以将它发送到授权服务器以获取访问令牌。该代码作为查询参数出现在请求 URL 中，授权服务器在登录后将我们重定向到该 URL。

**我们将设置一个 Zuul 后置过滤器来提取这段代码，并将其设置在 cookie 中。**这不仅仅是一个普通的 cookie，而是一个**安全的、只有 HTTP 的 cookie，具有非常有限的路径(`/auth/token` )** :

```
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    private ObjectMapper mapper = new ObjectMapper();

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        try {
            Map<String, List> params = ctx.getRequestQueryParams();

            if (requestURI.contains("auth/redirect")) {
                Cookie cookie = new Cookie("code", params.get("code").get(0));
                cookie.setHttpOnly(true);
                cookie.setPath(ctx.getRequest().getContextPath() + "/auth/token");
                ctx.getResponse().addCookie(cookie);
            }
        } catch (Exception e) {
            logger.error("Error occured in zuul post filter", e);
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        boolean shouldfilter = false;
        RequestContext ctx = RequestContext.getCurrentContext();
        String URI = ctx.getRequest().getRequestURI();

        if (URI.contains("auth/redirect") || URI.contains("auth/token") || URI.contains("auth/refresh")) {
            shouldfilter = true;
        }
        return shouldfilter;
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

为了增加一层额外的保护来抵御 CSRF 攻击，**我们将为我们所有的 cookie**添加一个[同站点 cookie](https://web.archive.org/web/20220521221350/https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00) 头。

为此，我们将创建一个配置类:

```
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

## 6。从 Cookie 中获取并使用代码

现在我们已经在 cookie 中有了代码，当前端 Angular 应用程序试图触发令牌请求时，它将在`/auth/token`发送请求，因此浏览器当然会发送该 cookie。

因此，我们现在将在代理中的`pre`过滤器中有另一个条件，即**将从 cookie 中提取代码，并将其与其他表单参数一起发送，以获得令牌**:

```
public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    ...
    else if (requestURI.contains("auth/token"))) {
        try {
            String code = extractCookie(req, "code");
            String formParams = String.format(
              "grant_type=%s&client;_id=%s&client;_secret=%s&redirect;_uri=%s&code;=%s",
              "authorization_code", CLIENT_ID, CLIENT_SECRET, REDIRECT_URL, code);

            byte[] bytes = formParams.getBytes("UTF-8");
            ctx.setRequest(new CustomHttpServletRequest(req, bytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
    } 
    ...
}

private String extractCookie(HttpServletRequest req, String name) {
    Cookie[] cookies = req.getCookies();
    if (cookies != null) {
        for (int i = 0; i < cookies.length; i++) {
            if (cookies[i].getName().equalsIgnoreCase(name)) {
                return cookies[i].getValue();
            }
        }
    }
    return null;
}
```

这里是我们的**`CustomHttpServletRequest` ——用于发送我们的请求体，将所需的表单参数转换为字节**:

```
public class CustomHttpServletRequest extends HttpServletRequestWrapper {

    private byte[] bytes;

    public CustomHttpServletRequest(HttpServletRequest request, byte[] bytes) {
        super(request);
        this.bytes = bytes;
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        return new ServletInputStreamWrapper(bytes);
    }

    @Override
    public int getContentLength() {
        return bytes.length;
    }

    @Override
    public long getContentLengthLong() {
        return bytes.length;
    }

    @Override
    public String getMethod() {
        return "POST";
    }
}
```

这将在响应中从授权服务器获得一个访问令牌。接下来，我们将了解如何转变应对方式。

## 7.将刷新令牌放在 Cookie 中

开始有趣的事情。

我们在这里计划做的是让客户机以 cookie 的形式获得刷新令牌。

**我们将添加 Zuul 后置过滤器，从响应的 JSON 主体中提取刷新令牌，并将其设置在 cookie 中。**这也是一个安全的、只有 HTTP 的 cookie，具有非常有限的路径(`/auth/refresh`):

```
public Object run() {
...
    else if (requestURI.contains("auth/token") || requestURI.contains("auth/refresh")) {
        InputStream is = ctx.getResponseDataStream();
        String responseBody = IOUtils.toString(is, "UTF-8");
        if (responseBody.contains("refresh_token")) {
            Map<String, Object> responseMap = mapper.readValue(responseBody, 
              new TypeReference<Map<String, Object>>() {});
            String refreshToken = responseMap.get("refresh_token").toString();
            responseMap.remove("refresh_token");
            responseBody = mapper.writeValueAsString(responseMap);

            Cookie cookie = new Cookie("refreshToken", refreshToken);
            cookie.setHttpOnly(true);
            cookie.setPath(ctx.getRequest().getContextPath() + "/auth/refresh");
            cookie.setMaxAge(2592000); // 30 days
            ctx.getResponse().addCookie(cookie);
        }
        ctx.setResponseBody(responseBody);
    }
    ...
}
```

正如我们所看到的，这里我们在 Zuul 后置过滤器中添加了一个条件来读取响应并提取路由`auth/token`和`auth/refresh`的刷新令牌。我们对这两者做了完全相同的事情，因为授权服务器在获得访问令牌和刷新令牌的同时发送相同的有效负载。

**然后我们从 JSON 响应中删除了`refresh_token`，以确保 cookie 之外的前端永远无法访问它。**

这里需要注意的另一点是，我们将 cookie 的最长期限设置为 30 天，因为这与令牌的到期时间相匹配。

## 8.从 Cookie 中获取并使用刷新令牌

既然 cookie 中已经有了刷新令牌，**当前端 Angular 应用试图触发令牌刷新**时，它将在`/auth/refresh`发送请求，因此浏览器当然会发送该 cookie。

**因此，我们现在将在代理中的`pre`过滤器中有另一个条件，它将从 cookie 中提取刷新令牌，并将其作为 HTTP 参数**转发——以便请求有效:

```
public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    ...
    else if (requestURI.contains("auth/refresh"))) {
        try {
            String token = extractCookie(req, "token");                       
            String formParams = String.format(
              "grant_type=%s&client;_id=%s&client;_secret=%s&refresh;_token=%s", 
              "refresh_token", CLIENT_ID, CLIENT_SECRET, token);

            byte[] bytes = formParams.getBytes("UTF-8");
            ctx.setRequest(new CustomHttpServletRequest(req, bytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
    } 
    ...
}
```

这类似于我们第一次获得访问令牌时所做的。但是请注意，窗体是不同的。**现在我们发送一个`refresh_token`的`grant_type`而不是`authorization_code`以及我们之前保存在 cookie** 中的令牌。

获得响应后，它再次在`pre`过滤器中进行相同的转换，正如我们在第 7 节中看到的。

## 9。从 Angular 刷新访问令牌

最后，让我们修改简单的前端应用程序，并实际利用刷新令牌:

下面是我们的函数`refreshAccessToken()`:

```
refreshAccessToken() {
  let headers = new HttpHeaders({
    'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});
  this._http.post('auth/refresh', {}, {headers: headers })
    .subscribe(
      data => this.saveToken(data),
      err => alert('Invalid Credentials')
    );
}
```

请注意我们是如何简单地使用现有的`saveToken()`函数的——只是向它传递不同的输入。

还要注意的是，**我们没有自己给`refresh_token`添加任何表单参数——因为这将由 Zuul 过滤器**来处理。

## 10.跑前端

由于我们的前端 Angular 客户端现在作为引导应用程序托管，运行它将与以前略有不同。

**第一步也一样。我们需要构建应用程序**:

```
mvn clean install
```

这将触发我们的`pom.xml`中定义的`frontend-maven-plugin`来构建角度代码，并将 UI 工件复制到`target/classes/static`文件夹中。这个过程覆盖了我们在`src/main/resources`目录中的所有内容。因此，我们需要确保在复制过程中包含该文件夹中的任何所需资源，例如`application.yml`。

**第二步，我们需要运行我们的`SpringBootApplication`类`UiApplication`** 。我们的客户端应用程序将在`application.yml`中指定的端口 8089 上启动并运行。

## 11。结论

在这篇 OAuth2 教程中，我们学习了如何在 Angular 客户端应用程序中存储刷新令牌，如何刷新过期的访问令牌，以及如何利用 Zuul 代理来完成所有这些工作。

本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220521221350/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-rest "The Full Registration/Authentication Example Project on Github")**