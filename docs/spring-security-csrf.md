# 春季安全 CSRF 防护指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-csrf>

## 1。概述

在本教程中，我们将讨论跨站点请求伪造(CSRF)攻击以及如何使用 Spring Security 来防止它们。

## 延伸阅读:

## [用春天 MVC 和百里香叶保护 CSRF](/web/20220814172559/https://www.baeldung.com/csrf-thymeleaf-with-spring-security)

Quick and practical guide to preventing CSRF attacks with Spring Security, Spring MVC and Thymeleaf.[Read more](/web/20220814172559/https://www.baeldung.com/csrf-thymeleaf-with-spring-security) →

## [Spring Boot 安全自动配置](/web/20220814172559/https://www.baeldung.com/spring-boot-security-autoconfiguration)

A quick and practical guide to Spring Boot's default Spring Security configuration.[Read more](/web/20220814172559/https://www.baeldung.com/spring-boot-security-autoconfiguration) →

## [Spring 方法安全性介绍](/web/20220814172559/https://www.baeldung.com/spring-security-method-security)

A guide to method-level security using the Spring Security framework.[Read more](/web/20220814172559/https://www.baeldung.com/spring-security-method-security) →

## 2。两次简单的 CSRF 攻击

CSRF 袭击有多种形式。我们来讨论一些最常见的。

### 2.1。获取示例

让我们考虑登录用户使用的以下`GET`请求，将钱转移到特定的银行账户`1234`:

```java
GET http://bank.com/transfer?accountNo=1234&amount;=100
```

如果攻击者想把钱从受害者的账户转到他自己的账户，他需要让受害者触发请求:

```java
GET http://bank.com/transfer?accountNo=5678&amount;=1000
```

有多种方法可以实现这一点:

*   **链接**–攻击者可以说服受害者点击该链接，例如，执行转账:

```java
<a href="http://bank.com/transfer?accountNo=5678&amount;=1000">
Show Kittens Pictures
</a>
```

*   **图片**–攻击者可能使用一个带有目标 URL 的`<img/>`标签作为图片来源。换句话说，点击是不必要的。该请求将在页面加载时自动执行:

```java
<img src="http://bank.com/transfer?accountNo=5678&amount;=1000"/>
```

### 2.2。帖子示例

假设主请求需要是 POST 请求:

```java
POST http://bank.com/transfer
accountNo=1234&amount;=100
```

在这种情况下，攻击者需要让受害者运行类似的请求:

```java
POST http://bank.com/transfer
accountNo=5678&amount;=1000
```

在这种情况下， `<a>`和`<img/>`标签都不起作用。

攻击者需要一个`<form>`:

```java
<form action="http://bank.com/transfer" method="POST">
    <input type="hidden" name="accountNo" value="5678"/>
    <input type="hidden" name="amount" value="1000"/>
    <input type="submit" value="Show Kittens Pictures"/>
</form>
```

但是，可以使用 JavaScript 自动提交表单:

```java
<body onload="document.forms[0].submit()">
<form>
...
```

### 2.3。实战模拟

现在我们已经了解了 CSRF 攻击的样子，让我们在一个 Spring 应用程序中模拟这些例子。

我们将从一个简单的控制器实现开始，即`BankController`:

```java
@Controller
public class BankController {
    private Logger logger = LoggerFactory.getLogger(getClass());

    @RequestMapping(value = "/transfer", method = RequestMethod.GET)
    @ResponseBody
    public String transfer(@RequestParam("accountNo") int accountNo, 
      @RequestParam("amount") final int amount) {
        logger.info("Transfer to {}", accountNo);
        ...
    }

    @RequestMapping(value = "/transfer", method = RequestMethod.POST)
    @ResponseStatus(HttpStatus.OK)
    public void transfer2(@RequestParam("accountNo") int accountNo, 
      @RequestParam("amount") final int amount) {
        logger.info("Transfer to {}", accountNo);
        ...
    }
}
```

我们还有一个触发银行转帐操作的基本 HTML 页面:

```java
<html>
<body>
    <h1>CSRF test on Origin</h1>
    <a href="transfer?accountNo=1234&amount;=100">Transfer Money to John</a>

    <form action="transfer" method="POST">
        <label>Account Number</label> 
        <input name="accountNo" type="number"/>

        <label>Amount</label>         
        <input name="amount" type="number"/>

        <input type="submit">
    </form>
</body>
</html>
```

这是在原始域上运行的主应用程序的页面。

我们应该注意到，我们通过一个简单的链接实现了一个`GET`，通过一个简单的`<form>`实现了一个`POST`。

现在让我们看看攻击者页面会是什么样子:

```java
<html>
<body>
    <a href="http://localhost:8080/transfer?accountNo=5678&amount;=1000">Show Kittens Pictures</a>

    <img src="http://localhost:8080/transfer?accountNo=5678&amount;=1000"/>

    <form action="http://localhost:8080/transfer" method="POST">
        <input name="accountNo" type="hidden" value="5678"/>
        <input name="amount" type="hidden" value="1000"/>
        <input type="submit" value="Show Kittens Picture">
    </form>
</body>
</html>
```

该页面将在不同的域上运行，即攻击者域。

最后，让我们在本地运行原始应用程序和攻击者应用程序。

为了使攻击生效，用户需要使用会话 cookie 向原始应用程序进行验证。

让我们首先访问原始应用程序页面:

```java
http://localhost:8081/spring-rest-full/csrfHome.html
```

它将在我们的浏览器上设置`JSESSIONID` cookie。

然后，让我们访问攻击者页面:

```java
http://localhost:8081/spring-security-rest/api/csrfAttacker.html
```

如果我们跟踪来自这个攻击者页面的请求，我们将能够发现攻击原始应用程序的请求。由于这些请求会自动提交`JSESSIONID` cookie，Spring 会对它们进行认证，就好像它们来自原始域一样。

## 3。Spring MVC 应用程序

为了保护 MVC 应用程序，Spring 为每个生成的视图添加了一个 CSRF 令牌。这个令牌必须在每个修改状态的 HTTP 请求(修补、发布、上传和删除— 非获取)时提交给服务器。这保护了我们的应用程序免受 CSRF 攻击，因为攻击者不能从他们自己的页面获得这个令牌。

接下来，我们将了解如何配置我们的应用程序安全性，以及如何使我们的客户端符合安全性。

### 3.1。Spring 安全配置

在较旧的 XML 配置(pre-Spring Security 4)中，CSRF 保护在默认情况下是禁用的，我们可以根据需要启用它:

```java
<http>
    ...
    <csrf />
</http>
```

从 Spring Security 4.x 开始，默认情况下启用 CSRF 保护。

**这个默认配置将 CSRF 令牌添加到名为`_csrf`的`HttpServletRequest`属性中。**

如果需要，我们可以禁用此配置:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
      .csrf().disable();
}
```

### 3.2。客户端配置

现在我们需要在请求中包含 CSRF 令牌。

`_csrf`属性包含以下信息:

*   `token`–CSRF 代币价值
*   `parameterName`–HTML 表单参数的名称，必须包含标记值
*   `headerName`–HTTP 头的名称，必须包含令牌值

如果我们的视图使用 HTML 表单，我们将使用`parameterName`和`token`值来添加一个隐藏输入:

```java
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
```

如果我们的视图使用 JSON，我们需要使用`headerName`和`token` 值来添加一个 HTTP 头。

我们首先需要在 meta 标记中包含令牌值和头名称:

```java
<meta name="_csrf" content="${_csrf.token}"/>
<meta name="_csrf_header" content="${_csrf.headerName}"/>
```

然后让我们用 JQuery 检索元标记值:

```java
var token = $("meta[name='_csrf']").attr("content");
var header = $("meta[name='_csrf_header']").attr("content"); 
```

最后，让我们使用这些值来设置我们的 XHR 头:

```java
$(document).ajaxSend(function(e, xhr, options) {
    xhr.setRequestHeader(header, token);
});
```

## 4.无状态 Spring API

让我们回顾一下前端使用的无状态 Spring API 的情况。

正如我们的专门文章中所解释的，我们需要了解我们的无状态 API 是否需要 CSRF 保护。

如果我们的无状态 API 使用基于令牌的认证，比如 JWT，我们就不需要 CSRF 保护，我们必须像前面看到的那样禁用它。

然而，如果我们的无状态 API 使用会话 cookie 认证，我们需要启用 CSRF 保护 **，我们将在接下来看到。** 

### 4.1.后端配置

我们的无状态 API 不能像 MVC 配置那样添加 CSRF 令牌，因为它不生成任何 HTML 视图。

在这种情况下，我们可以使用`CookieCsrfTokenRepository`发送 cookie 中的 CSRF 令牌:

```java
@Configuration
public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws {
        http
          .csrf()
          .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }
}
```

这个配置将在前端设置一个`XSRF-TOKEN` cookie。因为我们将`HTTP-only`标志设置为`false`，前端将能够使用 JavaScript 检索这个 cookie。

### 4.2.前端配置

使用 JavaScript，我们需要从`document.cookie`列表中搜索`XSRF-TOKEN` cookie 值。

因为这个列表是以字符串的形式存储的，所以我们可以使用这个正则表达式来检索它:

```java
const csrfToken = document.cookie.replace(/(?:(?:^|.*;\s*)XSRF-TOKEN\s*\=\s*([^;]*).*$)|^.*$/, '$1');
```

然后，我们必须向每个修改 API 状态的 REST 请求发送令牌:POST、PUT、DELETE 和 PATCH。

**Spring 希望在`X-XSRF-TOKEN`头中接收它。**

我们可以简单地用 JavaScript `Fetch` API 设置它:

```java
fetch(url, {
  method: 'POST',
  body: /* data to send */,
  headers: { 'X-XSRF-TOKEN': csrfToken },
})
```

## 5。CSRF 残疾人测试

一切就绪后，让我们做一些测试。

让我们首先尝试在 CSRF 被禁用时提交一个简单的 POST 请求:

```java
@ContextConfiguration(classes = { SecurityWithoutCsrfConfig.class, ...})
public class CsrfDisabledIntegrationTest extends CsrfAbstractIntegrationTest {

    @Test
    public void givenNotAuth_whenAddFoo_thenUnauthorized() throws Exception {
        mvc.perform(
          post("/foos").contentType(MediaType.APPLICATION_JSON)
            .content(createFoo())
          ).andExpect(status().isUnauthorized());
    }

    @Test 
    public void givenAuth_whenAddFoo_thenCreated() throws Exception {
        mvc.perform(
          post("/foos").contentType(MediaType.APPLICATION_JSON)
            .content(createFoo())
            .with(testUser())
        ).andExpect(status().isCreated()); 
    } 
}
```

这里我们使用一个基类来保存公共测试助手逻辑——`CsrfAbstractIntegrationTest`:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
public class CsrfAbstractIntegrationTest {
    @Autowired
    private WebApplicationContext context;

    @Autowired
    private Filter springSecurityFilterChain;

    protected MockMvc mvc;

    @Before
    public void setup() {
        mvc = MockMvcBuilders.webAppContextSetup(context)
          .addFilters(springSecurityFilterChain)
          .build();
    }

    protected RequestPostProcessor testUser() {
        return user("user").password("userPass").roles("USER");
    }

    protected String createFoo() throws JsonProcessingException {
        return new ObjectMapper().writeValueAsString(new Foo(randomAlphabetic(6)));
    }
}
```

我们应该注意到，当用户拥有正确的安全凭证时，请求被成功执行——不需要额外的信息。

这意味着攻击者可以简单地使用前面讨论的任何攻击手段来危害系统。

## 6。CSRF 启用测试

现在，让我们启用 CSRF 保护，看看不同之处:

```java
@ContextConfiguration(classes = { SecurityWithCsrfConfig.class, ...})
public class CsrfEnabledIntegrationTest extends CsrfAbstractIntegrationTest {

    @Test
    public void givenNoCsrf_whenAddFoo_thenForbidden() throws Exception {
        mvc.perform(
          post("/foos").contentType(MediaType.APPLICATION_JSON)
            .content(createFoo())
            .with(testUser())
          ).andExpect(status().isForbidden());
    }

    @Test
    public void givenCsrf_whenAddFoo_thenCreated() throws Exception {
        mvc.perform(
          post("/foos").contentType(MediaType.APPLICATION_JSON)
            .content(createFoo())
            .with(testUser()).with(csrf())
          ).andExpect(status().isCreated());
    }
}
```

我们可以看到该测试如何使用不同的安全配置—启用了 CSRF 保护的配置。

现在，如果不包含 CSRF 令牌，POST 请求将会失败，这当然意味着以前的攻击不再是一种选择。

此外，测试中的`[csrf()](https://web.archive.org/web/20220814172559/https://docs.spring.io/spring-security/site/docs/4.0.2.RELEASE/apidocs/org/springframework/security/test/web/servlet/request/SecurityMockMvcRequestPostProcessors.html#csrf())`方法创建了一个`RequestPostProcessor`,它会自动填充请求中的有效 CSRF 令牌，以便进行测试。

## 7。结论

在本文中，我们讨论了一些 CSRF 攻击，以及如何使用 Spring Security 来防止它们。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220814172559/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc-custom "The Full Registration/Authentication Example Project on Github ")