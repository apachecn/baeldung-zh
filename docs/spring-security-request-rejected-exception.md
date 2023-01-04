# Spring 安全–请求被拒绝异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-request-rejected-exception>

## 1。简介

Spring Framework 版本 5.0 到 5.0.4、4.3 到 4.3.14 以及其他旧版本在 Windows 系统上存在目录或路径遍历安全漏洞。

静态资源的错误配置使得恶意用户能够访问服务器的文件系统。**例如，使用 file: protocol 服务静态资源会提供对 Windows 上文件系统的非法访问**。

Spring 框架承认了[漏洞](https://web.archive.org/web/20220814172559/https://tanzu.vmware.com/security/cve-2018-1271)，并在以后的版本中修复了它。

因此，此修复保护应用程序免受路径遍历攻击。然而，通过这个修复，一些早期的 URL 现在抛出一个`org.springframework.security.web.firewall.RequestRejectedException` 异常`.`

最后，在本教程中，**让我们在路径遍历攻击**的背景下了解一下`org.springframework.security.web.firewall.RequestRejectedException`和`StrictHttpFirewall`。

## 2。路径遍历漏洞

路径遍历或目录遍历漏洞允许在 web 文档根目录之外进行非法访问。例如，操纵 URL 可以提供对文档根目录以外的文件的未经授权的访问。

尽管大多数最新和最流行的 web 服务器抵消了大部分攻击，攻击者仍然可以使用特殊字符的 URL 编码，如“.”。/", "../”来规避 web 服务器安全并获得非法访问。

另外， [OWASP](https://web.archive.org/web/20220814172559/https://owasp.org/www-community/attacks/Path_Traversal) 讨论了路径遍历漏洞以及解决这些漏洞的方法。

## 3。Spring 框架漏洞

现在，在我们学习如何修复它之前，让我们尝试复制这个漏洞。

首先，让我们克隆 Spring 框架 MVC 例子。**后面我们来修改一下`pom.xml` ，把现有的 Spring 框架版本换成有漏洞的版本。**T3


克隆存储库:

```java
git clone [[email protected]](/web/20220814172559/https://www.baeldung.com/cdn-cgi/l/email-protection):spring-projects/spring-mvc-showcase.git
```

在克隆的目录中，编辑`pom.xml` 以包含`5.0.0.RELEASE` 作为 Spring 框架版本:

```java
<org.springframework-version>5.0.0.RELEASE</org.springframework-version>
```

接下来，编辑 web 配置类`WebMvcConfig` 并修改`addResourceHandlers` 方法，以使用`file:`将资源映射到本地文件目录

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry
      .addResourceHandler("/resources/**")
      .addResourceLocations("file:./src/", "/resources/");
}
```

稍后，构建工件并运行我们的 web 应用程序:

```java
mvn jetty:run
```

现在，当服务器启动时，调用 URL:

```java
curl 'http://localhost:8080/spring-mvc-showcase/resources/%255c%255c%252e%252e%255c/%252e%252e%255c/%252e%252e%255c/%252e%252e%255c/%252e%252e%255c/windows/system.ini'
```

`%252e%252e%255c`是的双编码形式..而且`%255c%255c`是`\\.`的双重编码形式

**诚惶诚恐，响应的内容将是 Windows 系统文件`system.ini.`**

## 4.弹簧安全`HttpFirewall`界面

[Servlet 规范](https://web.archive.org/web/20220814172559/https://javaee.github.io/servlet-spec/downloads/servlet-4.0/servlet-4_0_FINAL.pdf)没有精确定义`servletPath`和`pathInfo.` 之间的区别，因此，在这些值的翻译中，Servlet 容器之间存在不一致。

例如，在`Tomcat 9`上，对于 URL `http://localhost:8080/api/v1/users/1`，URI `/1` 是一个路径变量。

另一方面，下面返回`/api/v1/users/1`:

```java
request.getServletPath()
```

但是，下面的命令会返回一个`null`:

```java
request.getPathInfo()
```

无法从 URI 中区分路径变量会导致潜在的攻击，如路径遍历/目录遍历攻击。例如，用户可以通过包含一个`\\, ` `/../, .`来利用服务器上的系统文件。\在 URL 中。不幸的是，只有一些 Servlet 容器规范了这些 URL。

[春安](https://web.archive.org/web/20220814172559/https://spring.io/projects/spring-security)来救援了。Spring Security 在容器间表现一致，并利用一个`HttpFirewall` 接口对这些恶意 URL 进行规范化。该接口有两个实现:

### 4.1.`DefaultHttpFirewall`

**首先，我们不要和实现类的名字混淆。换句话说，这不是默认的`HttpFirewall`实现。**

防火墙试图清理或标准化 URL，并标准化容器中的`servletPath`和`pathInfo`。此外，我们可以通过显式声明一个`@Bean`来覆盖默认的`HttpFirewall` 行为:

```java
@Bean
public HttpFirewall getHttpFirewall() {
    return new DefaultHttpFirewall();
}
```

然而，`StrictHttpFirewall`提供了一个健壮和安全的实现，并且是推荐的实现。

### 4.2.`StrictHttpFirewall`

**`StrictHttpFirewall`是`HttpFirewall.`** 的默认和更严格的实现相比之下，与`DefaultHttpFirewall`不同， [`StrictHttpFirewall`](https://web.archive.org/web/20220814172559/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/firewall/StrictHttpFirewall.html) 拒绝任何非规范化的 URL，提供更严格的保护。此外，这种实现保护应用程序免受其他几种攻击，如[跨站点跟踪(XST)](https://web.archive.org/web/20220814172559/https://owasp.org/www-community/attacks/Cross_Site_Tracing) 和 [HTTP 动词篡改](https://web.archive.org/web/20220814172559/https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/03-Testing_for_HTTP_Verb_Tampering)。

此外，这种实现是可定制的，并且具有合理的默认值。换句话说，我们可以禁用(不推荐)一些功能，比如允许分号作为 URI 的一部分:

```java
@Bean
public HttpFirewall getHttpFirewall() {
    StrictHttpFirewall strictHttpFirewall = new StrictHttpFirewall();
    strictHttpFirewall.setAllowSemicolon(true);
    return strictHttpFirewall;
}
```

简而言之， `StrictHttpFirewall`拒绝带有`org.springframework.security.web.firewall.RequestRejectedException`的可疑请求。

**最后，让我们使用 [Spring REST](/web/20220814172559/https://www.baeldung.com/rest-with-spring-series) 和 [Spring Security](/web/20220814172559/https://www.baeldung.com/security-spring) 开发一个对用户**进行 CRUD 操作的用户管理应用程序，看看`StrictHttpFirewall`的运行情况。

## 5.属国

让我们声明 [Spring Security](https://web.archive.org/web/20220814172559/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-security) 和 [Spring Web](https://web.archive.org/web/20220814172559/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-web) 的依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.4</version>
</dependency>
```

## 6.Spring 安全配置

接下来，让我们通过创建一个扩展了`WebSecurityConfigurerAdapter`的配置类，用基本认证来保护我们的应用程序:

```java
@Configuration
public class SpringSecurityHttpFirewallConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .csrf()
          .disable()
          .authorizeRequests()
            .antMatchers("/error").permitAll()
          .anyRequest()
          .authenticated()
          .and()
          .httpBasic();
    }
}
```

默认情况下，Spring Security 提供了一个默认密码，该密码会在每次重启时更改。因此，让我们在`application.properties`中创建一个默认的用户名和密码:

```java
spring.security.user.name=user
spring.security.user.password=password
```

从今以后，我们将使用这些凭证来访问我们的安全 REST APIs。

## 7.构建安全的 REST API

现在，让我们构建我们的用户管理 REST API:

```java
@PostMapping
public ResponseEntity<Response> createUser(@RequestBody User user) {
    userService.saveUser(user);
    Response response = new Response()
      .withTimestamp(System.currentTimeMillis())
      .withCode(HttpStatus.CREATED.value())
      .withMessage("User created successfully");
    URI location = URI.create("/users/" + user.getId());
    return ResponseEntity.created(location).body(response);
}

@DeleteMapping("/{userId}")
public ResponseEntity<Response> deleteUser(@PathVariable("userId") String userId) {
    userService.deleteUser(userId);
    return ResponseEntity.ok(new Response(200,
      "The user has been deleted successfully", System.currentTimeMillis()));
}
```

现在，让我们构建并运行应用程序:

```java
mvn spring-boot:run
```

## 8.测试 API

现在，让我们开始创建一个使用[卷曲](/web/20220814172559/https://www.baeldung.com/curl-rest)的`User`:

```java
curl -i --user user:password -d @request.json -H "Content-Type: application/json" 
     -H "Accept: application/json" http://localhost:8080/api/v1/users
```

这里有一个`request.json`:

```java
{
    "id":"1",
    "username":"navuluri",
    "email":"[[email protected]](/web/20220814172559/https://www.baeldung.com/cdn-cgi/l/email-protection)"
}
```

因此，答案是:

```java
HTTP/1.1 201
Location: /users/1
Content-Type: application/json
{
  "code":201,
  "message":"User created successfully",
  "timestamp":1632808055618
} 
```

现在，让我们配置我们的`StrictHttpFirewall` 来拒绝来自所有 HTTP 方法的请求:

```java
@Bean
public HttpFirewall configureFirewall() {
    StrictHttpFirewall strictHttpFirewall = new StrictHttpFirewall();
    strictHttpFirewall
      .setAllowedHttpMethods(Collections.emptyList());
    return strictHttpFirewall;
} 
```

接下来，让我们再次调用 API。由于我们配置了`StrictHttpFirewall`来限制所有的 HTTP 方法，这一次，我们得到了一个错误。

在日志中，我们有这样一个例外:

```java
org.springframework.security.web.firewall.RequestRejectedException: 
The request was rejected because the HTTP method "POST" was not included
  within the list of allowed HTTP methods []
```

**既然`Spring Security v5.4`有了`RequestRejectedException` :** 就可以用`RequestRejectedHandler`定制`HTTP Status`

```java
@Bean
public RequestRejectedHandler requestRejectedHandler() {
   return new HttpStatusRequestRejectedHandler();
}
```

注意，使用`HttpStatusRequestRejectedHandler` 时默认的 HTTP 状态代码是 `400\.` ，但是，我们可以通过在`HttpStatusRequestRejectedHandler` 类的构造函数中传递一个状态代码来定制它。

现在，让我们重新配置`StrictHttpFirewall` 以允许 URL 中的`\\` 以及`HTTP GET`、`POST`、`DELETE`和`OPTIONS`方法:

```java
strictHttpFirewall.setAllowBackSlash(true);
strictHttpFirewall.setAllowedHttpMethods(Arrays.asList("GET","POST","DELETE", "OPTIONS")
```

接下来，调用 API:

```java
curl -i --user user:password -d @request.json -H "Content-Type: application/json" 
     -H "Accept: application/json" http://localhost:8080/api<strong>\\</strong>v1/users
```

这里我们有一个回应:

```java
{
  "code":201,
  "message":"User created successfully",
  "timestamp":1632812660569
}
```

最后，让我们通过删除`@Bean` 声明来恢复`StrictHttpFirewall` 最初的严格功能。

接下来，让我们尝试使用可疑的 URL 来调用我们的 API:

```java
curl -i --user user:password -d @request.json -H "Content-Type: application/json" 
      -H "Accept: application/json" http://localhost:8080/api/v1<strong>//</strong>users
```

```java
curl -i --user user:password -d @request.json -H "Content-Type: application/json" 
      -H "Accept: application/json" http://localhost:8080/api/v1<strong>\\</strong>users
```

立即，所有上述请求失败，并显示错误日志:

```java
org.springframework.security.web.firewall.RequestRejectedException: 
The request was rejected because the URL contained a potentially malicious String "//"
```

## 9.结论

这篇文章解释了 Spring Security 对可能导致路径遍历/目录遍历攻击的恶意 URL 的保护。

`DefaultHttpFirewall`试图规范恶意网址。然而，`StrictHttpFirewall` 用一个`RequestRejectedException`拒绝了请求。除了路径遍历攻击，`StrictHttpFirewall` 还保护我们免受其他几种攻击。因此强烈推荐使用`StrictHttpFirewall`及其默认配置。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220814172559/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-3)