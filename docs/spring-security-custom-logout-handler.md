# Spring 安全定制注销处理程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-logout-handler>

## 1.概观

Spring 安全框架为[认证](/web/20220822105735/https://www.baeldung.com/spring-security-authentication-and-registration)提供了非常灵活和强大的支持。除了用户识别，我们通常还想处理用户注销事件，在某些情况下，添加一些定制的注销行为。一个这样的用例可能是使用户缓存无效或关闭已验证的会话。

为此，Spring 提供了`LogoutHandler`接口，在本教程中，我们将看看如何实现我们自己的定制注销处理程序。

## 2.处理注销请求

每个让用户登录的 web 应用程序都必须在某一天让他们注销。Spring 安全处理器通常控制注销过程。基本上，我们有两种处理注销的方法。正如我们将要看到的，其中之一就是实现了 [`LogoutHandler`](https://web.archive.org/web/20220822105735/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/LogoutHandler.html) 接口。

### 2.1.`LogoutHandler`界面

`LogoutHandler`接口有如下定义:

```java
public interface LogoutHandler {
    void logout(HttpServletRequest request, HttpServletResponse response,Authentication authentication);
} 
```

我们可以根据需要在应用程序中添加任意数量的注销处理程序。实现的一个要求是不抛出异常。这是因为处理程序动作不能在注销时破坏应用程序状态。

例如，其中一个处理程序可能会进行一些缓存清理，其方法必须成功完成。在教程示例中，我们将展示这个用例。

### 2.2.`LogoutSuccessHandler`界面

另一方面，我们可以使用异常来控制用户注销策略。为此，我们有了 [`LogoutSuccessHandler`](/web/20220822105735/https://www.baeldung.com/spring-security-track-logged-in-users#2-implementing-logoutsuccesshandler) 接口和`onLogoutSuccess`方法。此方法可能会引发异常，将用户重定向设置到适当的目标。

此外，**在使用`LogoutSuccessHandler`类型的**时，不可能添加多个处理程序，所以应用程序只有一个可能的实现。总的来说，原来是注销策略的最后一点。

## 3.`LogoutHandler`实践中的界面

现在，让我们创建一个简单的 web 应用程序来演示注销处理过程。我们将实现一些简单的缓存逻辑来检索用户数据，以避免对数据库不必要的访问。

让我们从`application.properties` 文件开始，它包含我们的示例应用程序的数据库连接属性:

```java
spring.datasource.url=jdbc:postgresql://localhost:5432/test
spring.datasource.username=test
spring.datasource.password=test
spring.jpa.hibernate.ddl-auto=create 
```

### 3.1.Web 应用程序设置

接下来，我们将添加一个简单的`User`实体，用于登录和数据检索。如我们所见，`User`类映射到数据库中的`users`表:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(unique = true)
    private String login;

    private String password;

    private String role;

    private String language;

    // standard setters and getters
} 
```

出于应用程序缓存的目的，我们将实现一个缓存服务，该服务在内部使用一个`ConcurrentHashMap`来存储用户:

```java
@Service
public class UserCache {
    @PersistenceContext
    private EntityManager entityManager;

    private final ConcurrentMap<String, User> store = new ConcurrentHashMap<>(256);
} 
```

使用此服务，我们可以通过用户名(登录名)从数据库中检索用户，并将其内部存储在我们的地图中:

```java
public User getByUserName(String userName) {
    return store.computeIfAbsent(userName, k -> 
      entityManager.createQuery("from User where login=:login", User.class)
        .setParameter("login", k)
        .getSingleResult());
} 
```

此外，有可能将用户逐出商店。正如我们将在后面看到的，这将是我们从注销处理程序中调用的主要操作:

```java
public void evictUser(String userName) {
    store.remove(userName);
} 
```

为了检索用户数据和语言信息，我们将使用一个标准的 [Spring `Controller`](/web/20220822105735/https://www.baeldung.com/spring-controllers) :

```java
@Controller
@RequestMapping(path = "/user")
public class UserController {

    private final UserCache userCache;

    public UserController(UserCache userCache) {
        this.userCache = userCache;
    }

    @GetMapping(path = "/language")
    @ResponseBody
    public String getLanguage() {
        String userName = UserUtils.getAuthenticatedUserName();
        User user = userCache.getByUserName(userName);
        return user.getLanguage();
    }
} 
```

### 3.2.Web 安全配置

我们将在应用程序中关注两个简单的操作——登录和注销。首先，我们需要设置 MVC 配置类，以允许用户使用[基本 HTTP Auth](/web/20220822105735/https://www.baeldung.com/httpclient-4-basic-authentication) 进行身份验证:

```java
@Configuration
@EnableWebSecurity
public class MvcConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomLogoutHandler logoutHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic()
            .and()
                .authorizeRequests()
                    .antMatchers(HttpMethod.GET, "/user/**")
                    .hasRole("USER")
            .and()
                .logout()
                    .logoutUrl("/user/logout")
                    .addLogoutHandler(logoutHandler)
                    .logoutSuccessHandler(new HttpStatusReturningLogoutSuccessHandler(HttpStatus.OK))
                    .permitAll()
            .and()
                .csrf()
                    .disable()
                .formLogin()
                    .disable();
    }

    // further configuration
} 
```

上述配置中需要注意的重要部分是`addLogoutHandler`方法。我们通过并且**在注销处理**结束时触发我们的`CustomLogoutHandler`。其余设置微调 HTTP 基本身份验证。

### 3.3.自定义注销处理程序

最后，也是最重要的，我们将编写我们的定制注销处理程序来处理必要的用户缓存清理:

```java
@Service
public class CustomLogoutHandler implements LogoutHandler {

    private final UserCache userCache;

    public CustomLogoutHandler(UserCache userCache) {
        this.userCache = userCache;
    }

    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, 
      Authentication authentication) {
        String userName = UserUtils.getAuthenticatedUserName();
        userCache.evictUser(userName);
    }
} 
```

正如我们所看到的，我们覆盖了`logout`方法，并简单地从用户缓存中驱逐给定的用户。

## 4.集成测试

现在让我们测试功能。首先，我们需要验证缓存是否如预期的那样工作— **也就是说，它将授权用户加载到其内部存储中**:

```java
@Test
public void whenLogin_thenUseUserCache() {
    assertThat(userCache.size()).isEqualTo(0);

    ResponseEntity<String> response = restTemplate.withBasicAuth("user", "pass")
        .getForEntity(getLanguageUrl(), String.class);

    assertThat(response.getBody()).contains("english");

    assertThat(userCache.size()).isEqualTo(1);

    HttpHeaders requestHeaders = new HttpHeaders();
    requestHeaders.add("Cookie", response.getHeaders()
        .getFirst(HttpHeaders.SET_COOKIE));

    response = restTemplate.exchange(getLanguageUrl(), HttpMethod.GET, 
      new HttpEntity<String>(requestHeaders), String.class);
    assertThat(response.getBody()).contains("english");

    response = restTemplate.exchange(getLogoutUrl(), HttpMethod.GET, 
      new HttpEntity<String>(requestHeaders), String.class);
    assertThat(response.getStatusCode()
        .value()).isEqualTo(200);
} 
```

让我们分解步骤来理解我们做了什么:

*   首先，我们检查缓存是否为空
*   接下来，我们通过`withBasicAuth`方法验证用户
*   现在我们可以验证检索到的用户数据和语言值
*   因此，我们可以验证用户现在一定在缓存中
*   同样，我们通过访问语言端点并使用会话 cookie 来检查用户数据
*   最后，我们验证注销用户

在我们的第二个测试中，我们将验证当我们注销时，用户**缓存是否被清理。这是我们的注销处理程序将被调用的时刻:**

```java
@Test
public void whenLogout_thenCacheIsEmpty() {
    assertThat(userCache.size()).isEqualTo(0);

    ResponseEntity<String> response = restTemplate.withBasicAuth("user", "pass")
        .getForEntity(getLanguageUrl(), String.class);

    assertThat(response.getBody()).contains("english");

    assertThat(userCache.size()).isEqualTo(1);

    HttpHeaders requestHeaders = new HttpHeaders();
    requestHeaders.add("Cookie", response.getHeaders()
        .getFirst(HttpHeaders.SET_COOKIE));

    response = restTemplate.exchange(getLogoutUrl(), HttpMethod.GET, 
      new HttpEntity<String>(requestHeaders), String.class);
    assertThat(response.getStatusCode()
        .value()).isEqualTo(200);

    assertThat(userCache.size()).isEqualTo(0);

    response = restTemplate.exchange(getLanguageUrl(), HttpMethod.GET, 
      new HttpEntity<String>(requestHeaders), String.class);
    assertThat(response.getStatusCode()
        .value()).isEqualTo(401);
} 
```

同样，一步一步来:

*   和以前一样，我们首先检查缓存是否为空
*   然后，我们对用户进行身份验证，并检查用户是否在缓存中
*   接下来，我们执行注销并检查用户是否已经从缓存中删除
*   最后，试图击中语言端点的结果是 401 HTTP 未授权响应代码

## 5.结论

在本教程中，我们学习了如何使用 Spring 的`LogoutHandler`接口实现一个定制的注销处理程序来从用户缓存中驱逐用户。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220822105735/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-2)