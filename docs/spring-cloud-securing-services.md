# spring Cloud–保护服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-securing-services>

## 1。概述

在上一篇文章[Spring Cloud-Bootstrapping](/web/20220124013546/https://www.baeldung.com/spring-cloud-bootstrapping)中，我们已经构建了一个基本的`Spring Cloud`应用程序。本文展示了如何保护它。

我们自然会使用`Spring Security`来共享使用`Spring Session` 和`Redis`的会话。这种方法设置简单，并且易于扩展到许多业务场景。如果你对`Spring Session`不熟悉，可以看看[这篇文章](/web/20220124013546/https://www.baeldung.com/spring-session)。

共享会话使我们能够让用户登录我们的网关服务，并将认证传播到我们系统的任何其他服务。

如果你对`Redis or` `Spring Security`不熟悉，此时快速回顾一下这些主题是个好主意。虽然文章的大部分内容都可以直接复制粘贴到应用程序中，但是对于理解幕后发生的事情来说，这是无可替代的。

关于`Redis`的介绍，请阅读[这篇](/web/20220124013546/https://www.baeldung.com/spring-data-redis-tutorial)教程。关于`Spring Security` 的介绍，请阅读 [spring-security-login](/web/20220124013546/https://www.baeldung.com/spring-security-login) 、[role-and-privilege-for-spring-security-registration](/web/20220124013546/https://www.baeldung.com/role-and-privilege-for-spring-security-registration)和 [spring-security-session](/web/20220124013546/https://www.baeldung.com/spring-security-session) 。要全面了解`Spring Security,`，请看一下[学-春-安-大师班](https://web.archive.org/web/20220124013546/http://courses.baeldung.com/p/learn-spring-security-the-master-class)。

## 2。Maven 设置

让我们首先向系统中的每个模块添加[spring-boot-starter-security](https://web.archive.org/web/20220124013546/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-security%22)依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

因为我们使用`Spring`依赖性管理，所以我们可以省略`spring-boot-starter`依赖性的版本。

第二步，让我们用 [spring-session](https://web.archive.org/web/20220124013546/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.session%22%20AND%20a%3A%22spring-session%22) 、[spring-boot-starter-data-redis](https://web.archive.org/web/20220124013546/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-data-redis%22)依赖项修改每个应用程序的`pom.xml` :

```java
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

我们的应用程序中只有四个可以绑定到`Spring Session` : **探索**，**网关**，**预订服务**，以及**评级服务**。

接下来，在与主应用程序文件相同的目录中，在所有三个服务中添加一个会话配置类:

```java
@EnableRedisHttpSession
public class SessionConfig
  extends AbstractHttpSessionApplicationInitializer {
}
```

最后，将这些属性添加到 git 存储库中的三个`*.properties`文件中:

```java
spring.redis.host=localhost 
spring.redis.port=6379
```

现在，让我们进入特定于服务的配置。

## 3。保护配置服务

配置服务包含通常与数据库连接和 API 键相关的敏感信息。我们不能泄露这些信息，所以让我们直接进入并保护这项服务。

让我们向配置服务的`src/main/resources`中的`application.properties`文件添加安全属性:

```java
eureka.client.serviceUrl.defaultZone=
  http://discUser:[[email protected]](/web/20220124013546/https://www.baeldung.com/cdn-cgi/l/email-protection):8082/eureka/
security.user.name=configUser
security.user.password=configPassword
security.user.role=SYSTEM
```

这将设置我们的服务登录 discovery。此外，我们正在用`application.properties`文件配置我们的安全性。

现在让我们配置我们的发现服务。

## 4。保护发现服务

我们的发现服务保存着应用程序中所有服务位置的敏感信息。它还注册这些服务的新实例。

如果恶意客户端获得访问权限，他们将了解我们系统中所有服务的网络位置，并能够将他们自己的恶意服务注册到我们的应用程序中。确保发现服务的安全至关重要。

### 4.1。安全配置

让我们添加一个安全过滤器来保护其他服务将使用的端点:

```java
@Configuration
@EnableWebSecurity
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

   @Autowired
   public void configureGlobal(AuthenticationManagerBuilder auth) {
       auth.inMemoryAuthentication().withUser("discUser")
         .password("discPassword").roles("SYSTEM");
   }

   @Override
   protected void configure(HttpSecurity http) {
       http.sessionManagement()
         .sessionCreationPolicy(SessionCreationPolicy.ALWAYS)
         .and().requestMatchers().antMatchers("/eureka/**")
         .and().authorizeRequests().antMatchers("/eureka/**")
         .hasRole("SYSTEM").anyRequest().denyAll().and()
         .httpBasic().and().csrf().disable();
   }
}
```

这将为'`SYSTEM`'用户设置我们的服务。这是一个基本的`Spring Security` 配置，有一些变化。让我们来看看这些转折:

*   `@Order(1)` –告诉`Spring`首先连接此安全过滤器，以便在任何其他过滤器之前尝试
*   `.sessionCreationPolicy` –告诉`Spring`当用户登录该过滤器时，总是创建一个会话
*   `.requestMatchers` –限制此过滤器适用的端点

我们刚刚设置的安全过滤器配置了一个仅适用于发现服务的隔离身份验证环境。

### 4.2。保护尤里卡仪表板

因为我们的 discovery 应用程序有一个很好的 UI 来查看当前注册的服务，所以让我们使用第二个安全过滤器来公开它，并将这个过滤器绑定到应用程序其余部分的身份验证中。请记住，没有`@Order()`标记意味着这是要评估的最后一个安全过滤器:

```java
@Configuration
public static class AdminSecurityConfig
  extends WebSecurityConfigurerAdapter {

@Override
protected void configure(HttpSecurity http) {
   http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.NEVER)
     .and().httpBasic().disable().authorizeRequests()
     .antMatchers(HttpMethod.GET, "/").hasRole("ADMIN")
     .antMatchers("/info", "/health").authenticated().anyRequest()
     .denyAll().and().csrf().disable();
   }
}
```

在`SecurityConfig` 类中添加这个配置类。这将创建第二个安全过滤器来控制对我们 UI 的访问。这个过滤器有一些不寻常的特征，让我们来看看:

*   `httpBasic().disable()`–告知 spring security 禁用该过滤器的所有验证过程
*   `sessionCreationPolicy`–我们将此设置为`NEVER`,以表明我们要求用户在访问受此过滤器保护的资源之前已经过身份验证

该过滤器永远不会设置用户会话，而是依赖于`Redis`来填充共享的安全上下文。因此，它依赖于另一个服务(网关)来提供身份验证。

### 4.3。正在向配置服务认证

在发现项目中，让我们向 src/main/resources 中的`bootstrap.properties`添加两个属性:

```java
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
```

这些属性将允许发现服务在启动时通过配置服务进行身份验证。

让我们更新 Git 存储库中的 `discovery.properties`

```java
eureka.client.serviceUrl.defaultZone=
  http://discUser:[[email protected]](/web/20220124013546/https://www.baeldung.com/cdn-cgi/l/email-protection):8082/eureka/
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

我们已经向我们的**发现**服务添加了基本认证凭证，以允许它与**配置**服务进行通信。此外，我们通过告诉我们的服务不要向自身注册来配置`Eureka` 在独立模式下运行。

让我们将文件提交给`git` 存储库。否则，将不会检测到更改。

## 5。保护网关服务

我们的网关服务是我们的应用程序中唯一希望向外界公开的部分。因此，它需要安全性来确保只有经过身份验证的用户才能访问敏感信息。

### 5.1。安全配置

让我们创建一个类似于我们的发现服务的`SecurityConfig`类，并用以下内容覆盖这些方法:

```java
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) {
    auth.inMemoryAuthentication().withUser("user").password("password")
      .roles("USER").and().withUser("admin").password("admin")
      .roles("ADMIN");
}

@Override
protected void configure(HttpSecurity http) {
    http.authorizeRequests().antMatchers("/book-service/books")
      .permitAll().antMatchers("/eureka/**").hasRole("ADMIN")
      .anyRequest().authenticated().and().formLogin().and()
      .logout().permitAll().logoutSuccessUrl("/book-service/books")
      .permitAll().and().csrf().disable();
}
```

这种配置非常简单。我们用表单登录声明一个安全过滤器，保护各种端点。

/eureka/**上的安全性是为了保护我们将从我们的网关服务为`Eureka`状态页面提供的一些静态资源。如果您使用文章构建项目，将 [Github](https://web.archive.org/web/20220124013546/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-bootstrap) 上的网关项目中的`resource/static` 文件夹复制到您的项目中。

现在我们修改 config 类上的`@EnableRedisHttpSession`注释:

```java
@EnableRedisHttpSession(
  redisFlushMode = RedisFlushMode.IMMEDIATE)
```

我们将刷新模式设置为 immediate，以便立即保存会话中的任何更改。这有助于为重定向准备身份验证令牌。

最后，让我们添加一个`ZuulFilter`，它将在登录后转发我们的身份验证令牌:

```java
@Component
public class SessionSavingZuulPreFilter
  extends ZuulFilter {

    @Autowired
    private SessionRepository repository;

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext context = RequestContext.getCurrentContext();
        HttpSession httpSession = context.getRequest().getSession();
        Session session = repository.getSession(httpSession.getId());

        context.addZuulRequestHeader(
          "Cookie", "SESSION=" + httpSession.getId());
        return null;
    }

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }
}
```

该过滤器将在登录后重定向请求时捕获请求，并将会话密钥作为 cookie 添加到标题中。这将在登录后将身份验证传播到任何支持服务。

### 5.2。使用配置和发现服务进行身份验证

让我们将以下认证属性添加到网关服务的`src/main/resources`中的`bootstrap.properties`文件中:

```java
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:[[email protected]](/web/20220124013546/https://www.baeldung.com/cdn-cgi/l/email-protection):8082/eureka/
```

接下来，让我们更新 Git 存储库中的 `gateway.properties`

```java
management.security.sessions=always

zuul.routes.book-service.path=/book-service/**
zuul.routes.book-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.book-service.execution.isolation.thread
    .timeoutInMilliseconds=600000

zuul.routes.rating-service.path=/rating-service/**
zuul.routes.rating-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.rating-service.execution.isolation.thread
    .timeoutInMilliseconds=600000

zuul.routes.discovery.path=/discovery/**
zuul.routes.discovery.sensitive-headers=Set-Cookie,Authorization
zuul.routes.discovery.url=http://localhost:8082
hystrix.command.discovery.execution.isolation.thread
    .timeoutInMilliseconds=600000
```

我们添加了会话管理以始终生成会话，因为我们只有一个安全过滤器，我们可以在属性文件中设置它。接下来，我们添加我们的`Redis`主机和服务器属性。

此外，我们添加了一个路由，将请求重定向到我们的发现服务。由于独立的发现服务不会向自身注册，因此我们必须使用 URL 方案来定位该服务。

我们可以从配置 git 存储库中的`gateway.properties`文件中删除`serviceUrl.defaultZone` 属性。该值在`bootstrap`文件中重复。

让我们将文件提交给 Git 存储库，否则，将不会检测到更改。

## 6。保护图书服务

图书服务服务器将保存由不同用户控制的敏感信息。这项服务必须得到保护，以防止我们系统中受保护信息的泄露。

### 6.1。安全配置

为了保护我们的 book 服务，我们将从网关复制`SecurityConfig`类并用以下内容覆盖该方法:

```java
@Override
protected void configure(HttpSecurity http) {
    http.httpBasic().disable().authorizeRequests()
      .antMatchers("/books").permitAll()
      .antMatchers("/books/*").hasAnyRole("USER", "ADMIN")
      .authenticated().and().csrf().disable();
}
```

### 6.2。属性

将这些属性添加到图书服务的`src/main/resources`中的`bootstrap.properties`文件:

```java
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:[[email protected]](/web/20220124013546/https://www.baeldung.com/cdn-cgi/l/email-protection):8082/eureka/
```

让我们向 git 存储库中的`book-service.properties`文件添加属性:

```java
management.security.sessions=never
```

我们可以从配置 git 存储库中的`book-service.properties`文件中删除`serviceUrl.defaultZone` 属性。该值在`bootstrap`文件中重复。

请记住提交这些更改，这样图书服务将会获取它们。

## 7 .**。安全评级服务**

评级服务也需要得到保护。

### 7.1。安全配置

为了保护我们的评级服务，我们将从网关复制`SecurityConfig`类并用以下内容覆盖该方法:

```java
@Override
protected void configure(HttpSecurity http) {
    http.httpBasic().disable().authorizeRequests()
      .antMatchers("/ratings").hasRole("USER")
      .antMatchers("/ratings/all").hasAnyRole("USER", "ADMIN").anyRequest()
      .authenticated().and().csrf().disable();
}
```

我们可以从**网关**服务中删除`configureGlobal()`方法。

### 7.2。属性

将这些属性添加到评级服务的`src/main/resources`中的`bootstrap.properties`文件中:

```java
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:[[email protected]](/web/20220124013546/https://www.baeldung.com/cdn-cgi/l/email-protection):8082/eureka/
```

让我们将属性添加到 git 存储库中的评级服务`.properties`文件中:

```java
management.security.sessions=never
```

我们可以从配置 git 存储库中的 rating-service `.properties`文件中删除`serviceUrl.defaultZone` 属性。该值在`bootstrap`文件中重复。

请记住提交这些更改，以便分级服务能够获取这些更改。

## 8。运行和测试

启动`Redis`和应用的所有服务:**配置、发现、**、网关、预订服务、和**分级服务**。现在我们来测试一下！

首先，让我们在 **gateway** 项目中创建一个测试类，并为我们的测试创建一个方法:

```java
public class GatewayApplicationLiveTest {
    @Test
    public void testAccess() {
        ...
    }
}
```

接下来，让我们设置我们的测试，并通过在我们的测试方法中添加以下代码片段来验证我们可以访问不受保护的`/book-service/books`资源:

```java
TestRestTemplate testRestTemplate = new TestRestTemplate();
String testUrl = "http://localhost:8080";

ResponseEntity<String> response = testRestTemplate
  .getForEntity(testUrl + "/book-service/books", String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
Assert.assertNotNull(response.getBody());
```

运行此测试并验证结果。如果我们看到失败，请确认整个应用程序成功启动，并且从我们的配置 git 存储库中加载了配置。

现在，让我们通过将以下代码附加到测试方法的末尾来测试我们的用户在以未经身份验证的用户身份访问受保护的资源时会被重定向到登录状态:

```java
response = testRestTemplate
  .getForEntity(testUrl + "/book-service/books/1", String.class);
Assert.assertEquals(HttpStatus.FOUND, response.getStatusCode());
Assert.assertEquals("http://localhost:8080/login", response.getHeaders()
  .get("Location").get(0));
```

再次运行测试，并确认测试成功。

接下来，让我们实际登录，然后使用我们的会话来访问用户保护的结果:

```java
MultiValueMap<String, String> form = new LinkedMultiValueMap<>();
form.add("username", "user");
form.add("password", "password");
response = testRestTemplate
  .postForEntity(testUrl + "/login", form, String.class); 
```

现在，让我们从 cookie 中提取会话，并将其传播到以下请求:

```java
String sessionCookie = response.getHeaders().get("Set-Cookie")
  .get(0).split(";")[0];
HttpHeaders headers = new HttpHeaders();
headers.add("Cookie", sessionCookie);
HttpEntity<String> httpEntity = new HttpEntity<>(headers); 
```

并请求受保护的资源:

```java
response = testRestTemplate.exchange(testUrl + "/book-service/books/1",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
Assert.assertNotNull(response.getBody());
```

再次运行测试以确认结果。

现在，让我们尝试用同一个会话访问管理部分:

```java
response = testRestTemplate.exchange(testUrl + "/rating-service/ratings/all",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.FORBIDDEN, response.getStatusCode());
```

再次运行测试，不出所料，我们被限制以普通老用户的身份访问管理区。

下一个测试将验证我们能否以管理员身份登录并访问受管理员保护的资源:

```java
form.clear();
form.add("username", "admin");
form.add("password", "admin");
response = testRestTemplate
  .postForEntity(testUrl + "/login", form, String.class);

sessionCookie = response.getHeaders().get("Set-Cookie").get(0).split(";")[0];
headers = new HttpHeaders();
headers.add("Cookie", sessionCookie);
httpEntity = new HttpEntity<>(headers);

response = testRestTemplate.exchange(testUrl + "/rating-service/ratings/all",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
Assert.assertNotNull(response.getBody());
```

我们的考验越来越大了！但是我们可以看到，当我们运行它时，通过以 admin 身份登录，我们可以访问 admin 资源。

我们最后的测试是通过网关访问我们的发现服务器。为此，请将此代码添加到我们测试的末尾:

```java
response = testRestTemplate.exchange(testUrl + "/discovery",
  HttpMethod.GET, httpEntity, String.class);
Assert.assertEquals(HttpStatus.OK, response.getStatusCode());
```

最后一次运行该测试，以确认一切正常。成功！！！

你错过了吗？因为我们登录了我们的网关服务，并在我们的图书、评级和发现服务上查看内容，而无需登录四台独立的服务器！

通过利用`Spring Session`在服务器之间传播我们的身份验证对象，我们能够在网关上登录一次，并使用该身份验证来访问任意数量的后台服务上的控制器。

## 9。结论

云中的安全性无疑变得更加复杂。但是在`Spring Security` 和`Spring Session`的帮助下，我们可以轻松解决这个关键问题。

我们现在有了一个围绕我们服务的安全云应用程序。使用`Zuul` 和`Spring Session` ,我们可以让用户只登录一个服务，并将认证传播到我们的整个应用程序。这意味着我们可以轻松地将我们的应用程序划分到适当的域中，并在我们认为合适的时候保护每个域。

和往常一样，你可以在 GitHub 上找到源代码。