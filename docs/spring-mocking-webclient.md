# 嘲笑 Spring 中的 WebClient

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mocking-webclient>

## 1.概观

如今，我们期望在大多数服务中调用 REST APIs。Spring 提供了几个构建 REST 客户端的选项，推荐使用**`WebClient`** 。

在这个快速教程中，我们将学习如何使用`WebClient`调用 API 的**单元测试服务。**

## 2.嘲弄的

我们在测试中有两个主要的模拟选项:

*   使用[模仿](/web/20220707143817/https://www.baeldung.com/mockito-series)模仿`WebClient`的行为
*   真正使用`WebClient`，但是用 [`MockWebServer` (okhttp)](https://web.archive.org/web/20220707143817/https://github.com/square/okhttp/tree/master/mockwebserver) 模拟它调用的服务

## 3.使用 Mockito

Mockito 是 Java 最常见的模仿库。它擅长为方法调用提供预定义的响应，但是当模仿流畅的 API 时，事情变得具有挑战性。这是因为在一个流畅的 API 中，许多对象在调用代码和模拟代码之间传递。

例如，让一个带有`getEmployeeById` 方法的`EmployeeService`类使用`WebClient`通过 HTTP 获取数据:

```java
public class EmployeeService {

    public EmployeeService(String baseUrl) {
        this.webClient = WebClient.create(baseUrl);
    }
    public Mono<Employee> getEmployeeById(Integer employeeId) {
        return webClient
                .get()
                .uri("http://localhost:8080/employee/{id}", employeeId)
                .retrieve()
                .bodyToMono(Employee.class);
    }
} 
```

我们可以用 Mockito 来模拟这个:

```java
@ExtendWith(MockitoExtension.class)
public class EmployeeServiceTest {

    @Test
    void givenEmployeeId_whenGetEmployeeById_thenReturnEmployee() {

        Integer employeeId = 100;
        Employee mockEmployee = new Employee(100, "Adam", "Sandler", 
          32, Role.LEAD_ENGINEER);
        when(webClientMock.get())
          .thenReturn(requestHeadersUriSpecMock);
        when(requestHeadersUriMock.uri("/employee/{id}", employeeId))
          .thenReturn(requestHeadersSpecMock);
        when(requestHeadersMock.retrieve())
          .thenReturn(responseSpecMock);
        when(responseMock.bodyToMono(Employee.class))
          .thenReturn(Mono.just(mockEmployee));

        Mono<Employee> employeeMono = employeeService.getEmployeeById(employeeId);

        StepVerifier.create(employeeMono)
          .expectNextMatches(employee -> employee.getRole()
            .equals(Role.LEAD_ENGINEER))
          .verifyComplete();
    }

}
```

正如我们所看到的，我们需要为链中的每个调用提供不同的模拟对象，需要四个不同的`when` / `thenReturn`调用。**这是冗长而繁琐的**。它还要求我们知道我们的服务如何使用`WebClient,`的实现细节，这使得这成为一种脆弱的测试方式。

那么我们如何为`WebClient?` 编写更好的测试呢

## 4.使用`MockWebServer`

由 Square 团队构建的 [`MockWebServer`](https://web.archive.org/web/20220707143817/https://github.com/square/okhttp/tree/master/mockwebserver) 是一个小型 web 服务器，可以接收和响应 HTTP 请求。

**与来自我们测试用例的`MockWebServer`交互允许我们的代码使用真实的 HTTP 调用到本地端点**。我们得到了测试预期的 HTTP 交互的好处，并且没有模拟复杂的流畅客户端的挑战。

**使用** **`MockWebServer`是 Spring 团队推荐的[用于编写集成测试*。*](https://web.archive.org/web/20220707143817/https://github.com/spring-projects/spring-framework/issues/19852#issuecomment-453452354)**

### 4.1.`MockWebServer`依赖关系

要使用`MockWebServer`，我们需要将 [okhttp](https://web.archive.org/web/20220707143817/https://search.maven.org/search?q=g:com.squareup.okhttp3%20AND%20a:okhttp&core=gav) 和 [mockwebserver](https://web.archive.org/web/20220707143817/https://search.maven.org/search?q=g:com.squareup.okhttp3%20AND%20a:mockwebserver&core=gav) 的 Maven 依赖项添加到我们的 pom.xml:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.0.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>4.0.1</version>
    <scope>test</scope>
</dependency>
```

### 4.2.添加`MockWebServer`到我们的测试中

让我们用`MockWebServer`来测试我们的`EmployeeService`:

```java
public class EmployeeServiceMockWebServerTest {

    public static MockWebServer mockBackEnd;

    @BeforeAll
    static void setUp() throws IOException {
        mockBackEnd = new MockWebServer();
        mockBackEnd.start();
    }

    @AfterAll
    static void tearDown() throws IOException {
        mockBackEnd.shutdown();
    }
}
```

在上面的 JUnit 测试类中，`setUp`和`tearDown`方法负责创建和关闭`MockWebServer.`

下一步是**将实际 REST 服务调用的端口映射到`MockWebServer's`端口:**

```java
@BeforeEach
void initialize() {
    String baseUrl = String.format("http://localhost:%s", 
      mockBackEnd.getPort());
    employeeService = new EmployeeService(baseUrl);
} 
```

现在是时候创建一个存根了，这样`MockWebServer`就可以响应`HttpRequest`了。

### 4.3.拒绝回答

让我们使用`MockWebServer's`便利的`enqueue` 方法在 web 服务器上排队测试响应:

```java
@Test
void getEmployeeById() throws Exception {
    Employee mockEmployee = new Employee(100, "Adam", "Sandler", 
      32, Role.LEAD_ENGINEER);
    mockBackEnd.enqueue(new MockResponse()
      .setBody(objectMapper.writeValueAsString(mockEmployee))
      .addHeader("Content-Type", "application/json"));

    Mono<Employee> employeeMono = employeeService.getEmployeeById(100);

    StepVerifier.create(employeeMono)
      .expectNextMatches(employee -> employee.getRole()
        .equals(Role.LEAD_ENGINEER))
      .verifyComplete();
} 
```

**当从我们的`EmployeeService `类中的`getEmployeeById(Integer employeeId)`方法发出实际的 API 调用**时， **`MockWebServer`将用排队的存根**进行响应。

### 4.4.检查请求

我们可能还想确保`MockWebServer`被发送了正确的`HttpRequest`。

`MockWebServer`有一个名为`takeRequest` 的简便方法，它返回一个`RecordedRequest`的实例:

```java
RecordedRequest recordedRequest = mockBackEnd.takeRequest();

assertEquals("GET", recordedRequest.getMethod());
assertEquals("/employee/100", recordedRequest.getPath()); 
```

有了`RecordedRequest`，我们可以验证收到的`HttpRequest` ，以确保我们的`WebClient`正确发送了`.`

## 5.结论

在本文中，我们展示了基于 REST 客户端代码的**模拟`WebClient`可用的两个主要选项。**

虽然 Mockito 有效，并且对于简单的例子来说可能是一个很好的选择，但是推荐的方法是使用`MockWebServer`。

和往常一样，这篇文章的源代码可以在 GitHub 上找到。