# 使用 Spring MVC 测试 OAuth 安全 API(使用 Spring Security OAuth 遗留堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/oauth-api-testing-with-spring-mvc>

## 1。概述

在这篇文章中，我们将展示如何用 Spring MVC 测试支持测试一个使用`OAuth`保护的 API。

**注**:本文使用的是 [Spring OAuth 遗留项目](https://web.archive.org/web/20221208143917/https://spring.io/projects/spring-security-oauth)。

## 2。授权和资源服务器

要获得关于如何设置授权和资源服务器的教程，请阅读上一篇文章:[Spring REST API+oauth 2+angular js](/web/20221208143917/https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)。

我们的授权服务器使用`JdbcTokenStore`并定义了一个 id 为`“fooClientIdPassword”`和密码为`“secret”`的客户端，并支持`password`授权类型。

资源服务器将`/employee` URL 限制为管理员角色。

从 Spring Boot 版本 1.5.0 开始，安全适配器优先于 OAuth 资源适配器，所以为了颠倒顺序，我们必须用`@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)`注释`WebSecurityConfigurerAdapter`类。

否则，Spring 将尝试基于 Spring 安全规则而不是 Spring OAuth 规则来访问请求的 URL，当使用令牌认证时，我们将收到 403 错误。

## 3。定义一个示例 API

首先，让我们创建一个名为`Employee`的简单 POJO，它有两个属性，我们将通过 API 来操作:

```java
public class Employee {
    private String email;
    private String name;

    // standard constructor, getters, setters
}
```

接下来，让我们定义一个具有两个请求映射的控制器，用于获取并保存一个`Employee`对象到一个列表:

```java
@Controller
public class EmployeeController {

    private List<Employee> employees = new ArrayList<>();

    @GetMapping("/employee")
    @ResponseBody
    public Optional<Employee> getEmployee(@RequestParam String email) {
        return employees.stream()
          .filter(x -> x.getEmail().equals(email)).findAny();
    }

    @PostMapping("/employee")
    @ResponseStatus(HttpStatus.CREATED)
    public void postMessage(@RequestBody Employee employee) {
        employees.add(employee);
    }
} 
```

请记住，为了使这个工作，我们需要一个额外的 JDK8 杰克逊模块。否则，`Optional`类将不会被正确地序列化/反序列化。最新版本的 [jackson-datatype-jdk8](https://web.archive.org/web/20221208143917/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jackson-datatype-jdk8%22) 可以从 Maven Central 下载。

## 4。测试 API

### 4.1。设置测试类别

为了测试我们的 API，我们将创建一个用`@SpringBootTest`注释的测试类，它使用`AuthorizationServerApplication`类来读取应用程序配置。

为了测试具有 Spring MVC 测试支持的安全 API，我们需要注入`WebAppplicationContext`和 `Spring Security Filter Chain`bean。在测试运行之前，我们将使用这些来获得一个`MockMvc`实例:

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@SpringBootTest(classes = AuthorizationServerApplication.class)
public class OAuthMvcTest {

    @Autowired
    private WebApplicationContext wac;

    @Autowired
    private FilterChainProxy springSecurityFilterChain;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac)
          .addFilter(springSecurityFilterChain).build();
    }
}
```

### 4.2。获得访问令牌

简单地说，用`OAuth2` **保护的 API 期望接收一个值为`Bearer <access_token>`的`Authorization`报头**。

为了发送所需的`Authorization`头，我们首先需要通过向`/oauth/token`端点发出 POST 请求来获得有效的访问令牌。这个端点需要一个 HTTP 基本认证，带有 OAuth 客户端的 id 和秘密，以及一个指定`client_id`、`grant_type`、`username`和`password`的参数列表。

使用 Spring MVC 测试支持，可以将参数包装在一个`MultiValueMap`中，并使用`httpBasic`方法发送客户端认证。

**让我们创建一个方法，发送 POST 请求以获取令牌**，并从 JSON 响应中读取`access_token`值:

```java
private String obtainAccessToken(String username, String password) throws Exception {

    MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
    params.add("grant_type", "password");
    params.add("client_id", "fooClientIdPassword");
    params.add("username", username);
    params.add("password", password);

    ResultActions result 
      = mockMvc.perform(post("/oauth/token")
        .params(params)
        .with(httpBasic("fooClientIdPassword","secret"))
        .accept("application/json;charset=UTF-8"))
        .andExpect(status().isOk())
        .andExpect(content().contentType("application/json;charset=UTF-8"));

    String resultString = result.andReturn().getResponse().getContentAsString();

    JacksonJsonParser jsonParser = new JacksonJsonParser();
    return jsonParser.parseMap(resultString).get("access_token").toString();
}
```

### 4.3。测试获取和发布请求

可以使用`header(“Authorization”, “Bearer “+ accessToken)`方法将访问令牌添加到请求中。

让我们尝试访问一个没有`Authorization`头的安全映射，并验证我们收到了一个`unauthorized`状态代码:

```java
@Test
public void givenNoToken_whenGetSecureRequest_thenUnauthorized() throws Exception {
    mockMvc.perform(get("/employee")
      .param("email", EMAIL))
      .andExpect(status().isUnauthorized());
}
```

我们已经指定只有管理员角色的用户才能访问`/employee` URL。让我们创建一个测试，在这个测试中，我们为角色为`USER`的用户获取一个访问令牌，并验证我们收到了一个`forbidden`状态代码:

```java
@Test
public void givenInvalidRole_whenGetSecureRequest_thenForbidden() throws Exception {
    String accessToken = obtainAccessToken("user1", "pass");
    mockMvc.perform(get("/employee")
      .header("Authorization", "Bearer " + accessToken)
      .param("email", "[[email protected]](/web/20221208143917/https://www.baeldung.com/cdn-cgi/l/email-protection)"))
      .andExpect(status().isForbidden());
}
```

接下来，让我们使用有效的访问令牌测试我们的 API，通过发送 POST 请求来创建一个`Employee`对象，然后发送 GET 请求来读取创建的对象:

```java
@Test
public void givenToken_whenPostGetSecureRequest_thenOk() throws Exception {
    String accessToken = obtainAccessToken("admin", "nimda");

    String employeeString = "{\"email\":\"[[email protected]](/web/20221208143917/https://www.baeldung.com/cdn-cgi/l/email-protection)\",\"name\":\"Jim\"}";

    mockMvc.perform(post("/employee")
      .header("Authorization", "Bearer " + accessToken)
      .contentType(application/json;charset=UTF-8)
      .content(employeeString)
      .accept(application/json;charset=UTF-8))
      .andExpect(status().isCreated());

    mockMvc.perform(get("/employee")
      .param("email", "[[email protected]](/web/20221208143917/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .header("Authorization", "Bearer " + accessToken)
      .accept("application/json;charset=UTF-8"))
      .andExpect(status().isOk())
      .andExpect(content().contentType(application/json;charset=UTF-8))
      .andExpect(jsonPath("$.name", is("Jim")));
}
```

## 5。结论

在这篇快速教程中，我们展示了如何使用 Spring MVC 测试支持来测试 OAuth 保护的 API。

示例的完整源代码可以在 [GitHub 项目](https://web.archive.org/web/20221208143917/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy)中找到。

为了运行测试，项目有一个`mvc`概要文件，可以使用命令`mvn clean install -Pmvc.`来执行