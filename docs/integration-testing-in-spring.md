# 春季集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/integration-testing-in-spring>

## 1。概述

集成测试通过验证系统的端到端行为，在应用程序开发周期中扮演着重要的角色。

在本教程中，我们将学习如何利用 Spring MVC 测试框架来编写和运行测试控制器的集成测试，而无需显式启动 Servlet 容器。

## 2。准备工作

我们需要几个 Maven 依赖项来运行我们将在本文中使用的集成测试。首先，我们需要最新的 [junit-jupiter-engine](https://web.archive.org/web/20221212193357/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-engine%22) 、 [junit-jupiter-api](https://web.archive.org/web/20221212193357/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-api%22) 和 [Spring test](https://web.archive.org/web/20221212193357/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-test%22) 依赖项:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.3</version>
    <scope>test</scope>
</dependency> 
```

为了有效地断言结果，我们还将使用 [Hamcrest](https://web.archive.org/web/20221212193357/https://search.maven.org/classic/#search|gav|1|g%3A%22org.hamcrest%22%20AND%20a%3A%22hamcrest-library%22) 和 [JSON 路径](https://web.archive.org/web/20221212193357/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.jayway.jsonpath%22%20AND%20a%3A%22json-path%22):

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-library</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.5.0</version>
    <scope>test</scope>
</dependency>
```

## 3。Spring MVC 测试配置

现在让我们看看如何配置和运行启用了 Spring 的测试。

### 3.1。用 JUnit 5 在测试中启用 Spring

JUnit 5 定义了一个扩展接口，通过该接口，类可以与 JUnit 测试集成。

我们可以通过向我们的测试类添加`@ExtendWith`注释并指定扩展类来加载来启用这个扩展**。为了运行 Spring 测试，我们使用了`SpringExtension.class.`**

我们还需要 **`@ContextConfiguration`注释来加载上下文配置，并需要** **引导我们的测试将使用的**上下文。

让我们来看看:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { ApplicationConfig.class })
@WebAppConfiguration
public class GreetControllerIntegrationTest {
    ....
}
```

注意，在`@ContextConfiguration,`中，我们提供了`ApplicationConfig.class` config 类，它加载了这个特定测试所需的配置。

我们将在这里使用一个 Java 配置类来指定上下文配置。类似地，我们可以使用基于 XML 的配置:

```java
@ContextConfiguration(locations={""})
```

最后，我们还将使用 **@ `WebAppConfiguration`对测试进行注释，这将加载 web 应用程序上下文**。

默认情况下，它在路径`src/main/webapp.`中查找根 web 应用程序。我们可以通过简单地传递`value`属性来覆盖这个位置:

```java
@WebAppConfiguration(value = "")
```

### 3.2。`WebApplicationContext`对象

`WebApplicationContext`提供 web 应用程序配置。它将所有应用程序 beans 和控制器加载到上下文中。

现在，我们将能够将 web 应用程序上下文连接到测试中:

```java
@Autowired
private WebApplicationContext webApplicationContext;
```

### 3.3。模仿 Web 上下文 bean

`MockMvc`为 Spring MVC 测试提供支持。**它封装了所有的 web 应用程序 beans，并使它们可用于测试。**

让我们看看如何使用它:

```java
private MockMvc mockMvc;
@BeforeEach
public void setup() throws Exception {
    this.mockMvc = MockMvcBuilders.webAppContextSetup(this.webApplicationContext).build();
}
```

我们将在`@BeforeEach`注释方法中初始化`mockMvc`对象，这样我们就不必在每个测试中初始化它。

### 3.4。验证测试配置

让我们验证一下我们是否正确加载了`WebApplicationContext`对象(`webApplicationContext`)。我们还将检查是否连接了正确的`servletContext`:

```java
@Test
public void givenWac_whenServletContext_thenItProvidesGreetController() {
    ServletContext servletContext = webApplicationContext.getServletContext();

    Assert.assertNotNull(servletContext);
    Assert.assertTrue(servletContext instanceof MockServletContext);
    Assert.assertNotNull(webApplicationContext.getBean("greetController"));
}
```

注意，我们还检查 web 上下文中是否存在一个`GreetController.java` bean。这确保了 Spring beans 被正确加载。至此，集成测试的设置已经完成。现在，我们将看看如何使用`MockMvc`对象测试资源方法。

## 4。编写集成测试

在这一节中，我们将回顾测试框架中可用的基本操作。

我们将看看如何发送带有路径变量和参数的请求。我们还将通过几个例子展示如何断言正确的视图名称已被解析，或者响应体是预期的。

下面显示的代码片段使用了来自 M `ockMvcRequestBuilders` 或`MockMvcResultMatchers` 类的静态导入。

### 4.1。验证视图名称

我们可以调用测试中的`/homePage`端点作为`:`

```java
http://localhost:8080/spring-mvc-test/
```

或者

```java
http://localhost:8080/spring-mvc-test/homePage
```

首先，让我们看看测试代码:

```java
@Test
public void givenHomePageURI_whenMockMVC_thenReturnsIndexJSPViewName() {
    this.mockMvc.perform(get("/homePage")).andDo(print())
      .andExpect(view().name("index"));
}
```

让我们来分解一下:

*   `perform()`方法将调用一个 GET 请求方法，该方法返回`ResultActions`。使用这个结果，我们可以得到关于响应的断言期望，比如它的内容、HTTP 状态或头。
*   `andDo(print())`将打印请求和响应。这有助于在出现错误时获得详细的视图。
*   `andExpect()` 将期待所提供的参数。在我们的例子中，我们期望通过`MockMvcResultMatchers.view().`返回“索引”

### 4.2。验证响应正文

我们将调用测试中的`/greet`端点，如下所示:

```java
http://localhost:8080/spring-mvc-test/greet
```

预期产出将是:

```java
{
    "id": 1,
    "message": "Hello World!!!"
}
```

让我们看看测试代码:

```java
@Test
public void givenGreetURI_whenMockMVC_thenVerifyResponse() {
    MvcResult mvcResult = this.mockMvc.perform(get("/greet"))
      .andDo(print()).andExpect(status().isOk())
      .andExpect(jsonPath("$.message").value("Hello World!!!"))
      .andReturn();

    Assert.assertEquals("application/json;charset=UTF-8", 
      mvcResult.getResponse().getContentType());
}
```

让我们来看看到底发生了什么:

*   `andExpect(MockMvcResultMatchers.status().isOk())`将验证响应 HTTP 状态是否为`Ok` ( `200)`)。这确保了请求被成功执行。
*   `andExpect(MockMvcResultMatchers.jsonPath(“$.message”).value(“Hello World!!!”))`将验证响应内容是否与参数`Hello World!!!`匹配。这里，我们使用了`jsonPath`，它提取响应内容并提供请求的值。
*   `andReturn()` 将返回`MvcResult`对象，当我们需要验证库不能直接实现的东西时，就会用到这个对象。在本例中，我们添加了`assertEquals`来匹配从`MvcResult`对象中提取的响应的内容类型。

### 4。 **3。发送带有路径变量**的 GET 请求

我们将调用测试中的`/greetWithPathVariable/{name}`端点，如下所示:

```java
http://localhost:8080/spring-mvc-test/greetWithPathVariable/John
```

预期产出将是:

```java
{
    "id": 1,
    "message": "Hello World John!!!"
}
```

让我们看看测试代码:

```java
@Test
public void givenGreetURIWithPathVariable_whenMockMVC_thenResponseOK() {
    this.mockMvc
      .perform(get("/greetWithPathVariable/{name}", "John"))
      .andDo(print()).andExpect(status().isOk())

      .andExpect(content().contentType("application/json;charset=UTF-8"))
      .andExpect(jsonPath("$.message").value("Hello World John!!!"));
}
```

`MockMvcRequestBuilders.get(“/greetWithPathVariable/{name}”, “John”)`将以“`/greetWithPathVariable/John.`”的形式发送请求

就可读性和知道在 URL 中动态设置什么参数而言，这变得更容易。注意，我们可以根据需要传递任意多的路径参数。

### 4.4。发送带有查询参数的 GET 请求

我们将调用测试中的`/greetWithQueryVariable?name={name}`端点，如下所示:

```java
http://localhost:8080/spring-mvc-test/greetWithQueryVariable?name=John%20Doe
```

在这种情况下，预期输出将是:

```java
{
    "id": 1,
    "message": "Hello World John Doe!!!"
}
```

现在，让我们看看测试代码:

```java
@Test
public void givenGreetURIWithQueryParameter_whenMockMVC_thenResponseOK() {
    this.mockMvc.perform(get("/greetWithQueryVariable")
      .param("name", "John Doe")).andDo(print()).andExpect(status().isOk())
      .andExpect(content().contentType("application/json;charset=UTF-8"))
      .andExpect(jsonPath("$.message").value("Hello World John Doe!!!"));
}
```

**`param(“name”, “John Doe”)`会在 GET 请求**中追加查询参数。这类似于 `/greetWithQueryVariable?name=John%20Doe.` `“`

查询参数也可以使用 URI 模板样式来实现:

```java
this.mockMvc.perform(
  get("/greetWithQueryVariable?name={name}", "John Doe"));
```

### 4.5。发送发布请求

我们将调用测试中的`/greetWithPost`端点，如下所示:

```java
http://localhost:8080/spring-mvc-test/greetWithPost
```

我们应该获得输出:

```java
{
    "id": 1,
    "message": "Hello World!!!"
}
```

我们的测试代码是:

```java
@Test
public void givenGreetURIWithPost_whenMockMVC_thenVerifyResponse() {
    this.mockMvc.perform(post("/greetWithPost")).andDo(print())
      .andExpect(status().isOk()).andExpect(content()
      .contentType("application/json;charset=UTF-8"))
      .andExpect(jsonPath("$.message").value("Hello World!!!"));
}
```

`**MockMvcRequestBuilders.post(“/greetWithPost”)**` **会发帖子请求**。我们可以像以前一样设置路径变量和查询参数，而表单数据只能通过`param()`方法设置，类似于查询参数:

```java
http://localhost:8080/spring-mvc-test/greetWithPostAndFormData
```

那么数据将是:

```java
id=1;name=John%20Doe
```

所以我们应该得到:

```java
{
    "id": 1,
    "message": "Hello World John Doe!!!"
}
```

让我们看看我们的测试:

```java
@Test
public void givenGreetURI_whenMockMVC_thenVerifyResponse() throws Exception {
    MvcResult mvcResult = this.mockMvc.perform(MockMvcRequestBuilders.get("/greet"))
      .andDo(print())
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.message").value("Hello World!!!"))
      .andReturn();

   assertEquals("application/json;charset=UTF-8", mvcResult.getResponse().getContentType());
}
```

在上面的代码片段中，我们添加了两个参数:`id`表示“1”，而`name`表示“John Doe”

## 5.`MockMvc`限制

`MockMvc` 提供了一个优雅易用的 API 来调用 web 端点，同时检查和断言它们的响应。尽管有这么多好处，它还是有一些局限性。

首先，它使用了`[DispatcherServlet](https://web.archive.org/web/20221212193357/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html) `的一个子类来处理测试请求。更具体地说，`[TestDispatcherServlet](https://web.archive.org/web/20221212193357/https://github.com/spring-projects/spring-framework/blob/622ccc57672ffed758220b33a08f8215334cdb2d/spring-test/src/main/java/org/springframework/test/web/servlet/TestDispatcherServlet.java#L53) `负责调用控制器并执行所有熟悉的弹簧魔术。

`MockMvc`类[在内部包装](https://web.archive.org/web/20221212193357/https://github.com/spring-projects/spring-framework/blob/622ccc57672ffed758220b33a08f8215334cdb2d/spring-test/src/main/java/org/springframework/test/web/servlet/MockMvc.java#L72)这个`TestDispatcherServlet `。所以每次我们使用`perform() `方法发送请求时，`MockMvc `会直接使用底层的`TestDispatcherServlet `。因此，没有真正的网络连接，因此，我们不会在使用`MockMvc`时测试整个网络堆栈。

另外，**因为 Spring 准备了一个假的 web 应用程序上下文来模拟 HTTP 请求和响应，所以它可能不支持一个成熟的 Spring 应用程序的所有特性**。

例如，这个模拟设置不支持 [HTTP 重定向](https://web.archive.org/web/20221212193357/https://github.com/spring-projects/spring-boot/issues/7321)。乍一看，这似乎并不重要。然而，Spring Boot 通过将当前请求重定向到`/error `端点来处理一些错误。因此，如果我们使用`MockMvc, `，我们可能无法测试一些 API 故障。

作为对`MockMvc, `的替代，我们可以建立一个更真实的应用环境，然后使用`[RestTemplate](/web/20221212193357/https://www.baeldung.com/rest-template),` 甚至[放心](/web/20221212193357/https://www.baeldung.com/rest-assured-tutorial)，来测试我们的应用。

例如，使用 Spring Boot 很容易做到:

```java
@SpringBootTest(webEnvironment = DEFINED_PORT)
public class GreetControllerRealIntegrationTest {

    @Before
    public void setUp() {
        RestAssured.port = DEFAULT_PORT;
    }

    @Test
    public void givenGreetURI_whenSendingReq_thenVerifyResponse() {
        given().get("/greet")
          .then()
          .statusCode(200);
    }
}
```

在这里，我们甚至不需要添加`@ExtendWith(SpringExtension.class)`。

这样，每个测试都会向在随机 TCP 端口上侦听的应用程序发出一个真正的 HTTP 请求。

## 6。结论

在本文中，我们实现了一些简单的支持 Spring 的集成测试。

我们还查看了`WebApplicationContext`和`MockMvc`对象的创建，它们在调用应用程序的端点时起着重要的作用。

进一步看，我们讨论了如何发送带有各种参数传递的 GET 和 POST 请求，以及如何验证 HTTP 响应状态、头和内容。

然后，我们评估了`MockMvc.`的一些局限性，了解这些局限性可以指导我们就如何实现我们的测试做出明智的决定。

最后，所有这些例子和代码片段的实现都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221212193357/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java)