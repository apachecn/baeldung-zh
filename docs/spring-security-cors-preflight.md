# 用 CORS 预照明和 Spring Security 修复 401

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-cors-preflight>

## 1.概观

在这个简短的教程中，我们将学习如何解决错误“预检响应有无效的 HTTP 状态代码 401”，这可能发生在支持跨源通信和使用 Spring 安全的应用程序中。

首先，我们将看看什么是跨来源请求，然后我们将修复一个有问题的例子。

## 2.跨来源请求

简而言之，跨源请求是 HTTP 请求，其中请求的源和目标是不同的。例如，当一个 web 应用程序从一个域提供服务，而浏览器向另一个域中的服务器发送 AJAX 请求时，就是这种情况。

为了管理跨源请求，服务器需要启用一种称为 CORS 或跨源资源共享的特殊机制。

CORS 的第一步是一个`OPTIONS`请求，以确定请求的目标是否支持它。**这称为飞行前请求。**

然后，服务器可以用一组头来响应飞行前请求:

*   **`Access-Control-Allow-Origin` :** 定义哪些来源可以访问资源。“*”代表任何原点
*   **`Access-Control-Allow-Methods` :** 表示跨来源请求允许的 HTTP 方法
*   **`Access-Control-Allow-Headers` :** 表示允许跨来源请求的请求题头
*   **`Access-Control-Max-Age` :** 定义缓存的预检请求结果的到期时间

因此，如果飞行前请求不满足这些响应头确定的条件，实际的后续请求将抛出与跨起点请求相关的错误。

**很容易[将 CORS 支持添加到我们的 Spring-powered 服务](/web/20220625235645/https://www.baeldung.com/spring-cors)中，但是如果配置不正确，这个飞行前请求将总是以 401 失败。**

## 3.创建支持 CORS 的 REST API

为了模拟这个问题，让我们首先创建一个支持跨源请求的简单 REST API:

```java
@RestController
@CrossOrigin("http://localhost:4200")
public class ResourceController {

    @GetMapping("/user")
    public String user(Principal principal) {
        return principal.getName();
    }
}
```

`@CrossOrigin`注释确保我们的 API 只能从它的参数中提到的原点访问。

## 4.保护我们的 REST API

现在让我们用 Spring Security 来保护我们的 REST API:

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .httpBasic();
    }
}
```

在这个配置类中，我们已经对所有传入的请求实施了授权。因此，它将拒绝所有没有有效授权令牌的请求。

## 5.提出飞行前请求

现在我们已经创建了 REST API，让我们使用`curl`尝试一个飞行前请求:

```java
curl -v -H "Access-Control-Request-Method: GET" -H "Origin: http://localhost:4200" 
  -X OPTIONS http://localhost:8080/user
...
< HTTP/1.1 401
...
< WWW-Authenticate: Basic realm="Realm"
...
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Access-Control-Allow-Origin: http://localhost:4200
< Access-Control-Allow-Methods: POST
< Access-Control-Allow-Credentials: true
< Allow: GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH
...
```

从这个命令的输出中，我们可以看到**请求被 401 拒绝。**

由于这是一个`curl`命令，我们不会在输出中看到错误“预检响应有无效的 HTTP 状态代码 401”。

但是我们可以通过创建一个前端应用程序来重现这个错误，这个前端应用程序从不同的域使用我们的 REST API，并在浏览器中运行它。

## 6.解决方案

**在我们的 Spring 安全配置**中，我们没有明确地将预检请求从授权中排除。请记住，Spring Security 默认保护`all `端点。

因此，**我们的 API 也期望 OPTIONS 请求中有一个授权令牌。**

Spring 提供了一个开箱即用的解决方案来从授权检查中排除选项请求:

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // ...
        http.cors();
    }
}
```

`cors()`方法将把 Spring 提供的`CorsFilter`添加到应用程序上下文中，从而绕过选项请求的授权检查。

现在我们可以再次测试我们的应用程序，看看它是否正常工作。

## 7.结论

在这篇短文中，我们学习了如何修复错误“预检响应有无效的 HTTP 状态代码 401”，它与 Spring 安全和跨源请求相关联。

**注意，在这个例子中，客户端和 API 应该在不同的域或端口上运行，以重现问题。**例如，当在本地机器上运行时，我们可以将默认主机名映射到客户机，将机器 IP 地址映射到 REST API。

和往常一样，本教程中展示的例子可以在 Github 上找到[。](https://web.archive.org/web/20220625235645/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-3)