# Spring 安全–缓存控制头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-cache-control-headers>

## 1.介绍

在本文中，我们将探索如何使用 Spring Security 控制 HTTP 缓存。

我们将演示它的默认行为，并解释其背后的原因。然后，我们将研究改变这种行为的方法，部分或全部。

## 2.默认缓存行为

通过有效地使用缓存控制头，我们可以指示浏览器缓存资源并避免网络跳跃。这减少了延迟，也减少了服务器的负载。

默认情况下，Spring Security 会为我们设置特定的缓存控制头值，而无需我们进行任何配置。

首先，让我们为我们的应用程序设置 Spring 安全性:

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {}
}
```

我们覆盖了`configure()` 不做任何事情，这意味着我们不需要被认证来访问一个端点，使我们能够专注于纯粹的测试缓存。

接下来，让我们实现一个简单的 REST 端点:

```
@GetMapping("/default/users/{name}")
public ResponseEntity<UserDto> getUserWithDefaultCaching(@PathVariable String name) {
    return ResponseEntity.ok(new UserDto(name));
}
```

产生的`cache-control` 标题将如下所示:

```
[cache-control: no-cache, no-store, max-age=0, must-revalidate]
```

最后，让我们实现一个命中端点的测试，并断言在响应中发送了什么报头:

```
given()
  .when()
  .get(getBaseUrl() + "/default/users/Michael")
  .then()
  .header("Cache-Control", "no-cache, no-store, max-age=0, must-revalidate")
  .header("Pragma", "no-cache");
```

本质上，这意味着浏览器永远不会缓存这个响应。

虽然这看起来效率不高，但这种默认行为实际上有一个很好的理由—**如果一个用户注销，另一个用户登录，我们不希望他们能够看到以前的用户资源**。默认情况下不缓存任何东西要安全得多，让我们负责显式启用缓存。

## 3.覆盖默认缓存行为

有时我们可能会处理我们确实想要缓存的资源。如果我们要启用它，最安全的做法是基于每个资源。这意味着默认情况下，任何其他资源都不会被缓存。

为此，让我们通过使用`[CacheControl](https://web.archive.org/web/20220815040909/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/CacheControl.html)`缓存，尝试在单个处理程序方法中覆盖缓存控制头。`CacheControl`类是一个流畅的构建器，这使得我们可以很容易地创建不同类型的缓存:

```
@GetMapping("/users/{name}")
public ResponseEntity<UserDto> getUser(@PathVariable String name) { 
    return ResponseEntity.ok()
      .cacheControl(CacheControl.maxAge(60, TimeUnit.SECONDS))
      .body(new UserDto(name));
}
```

让我们在测试中达到这个端点，并断言我们已经更改了头:

```
given()
  .when()
  .get(getBaseUrl() + "/users/Michael")
  .then()
  .header("Cache-Control", "max-age=60");
```

如我们所见，我们已经覆盖了默认值，现在我们的响应将被浏览器缓存 60 秒。

## 4.关闭默认缓存行为

我们也可以完全关闭 Spring Security 的默认缓存控制头。这是一件非常冒险的事情，不建议这样做。但是，如果我们真的想这么做，我们可以通过覆盖`WebSecurityConfigurerAdapter:`的`configure` 方法来尝试

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.headers().disable();
}
```

现在，让我们再次向我们的端点发出请求，看看我们得到什么响应:

```
given()
  .when()
  .get(getBaseUrl() + "/default/users/Michael")
  .then()
  .headers(new HashMap<String, Object>());
```

正如我们所看到的，根本没有设置任何缓存头。同样，**这是不安全的，但是证明了如果我们想的话，我们可以关闭默认的头。**

## 5.结论

本文演示了 Spring Security 如何在默认情况下禁用 HTTP 缓存，并解释了这是因为我们不想缓存安全的资源。我们还看到了如何在我们认为合适的时候禁用或修改这种行为。

所有这些例子和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220815040909/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-3)中找到。