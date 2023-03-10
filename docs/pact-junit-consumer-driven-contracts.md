# 消费者驱动的契约

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/pact-junit-consumer-driven-contracts>

## 1。概述

在这篇简短的文章中，我们将探讨消费者驱动的契约的概念。

我们将通过使用 [`Pact`](https://web.archive.org/web/20220626082031/https://docs.pact.io/) 库定义的契约来测试与外部 REST 服务的集成。**该合同可以由客户定义，然后由提供商获取并用于其服务的开发。**

我们还将为客户端和提供者应用程序创建基于契约的测试。

## 2。什么是`Pact`？

**使用`Pact` `,` 我们可以以契约的形式**(库的名字由此而来)定义消费者对给定提供者(可以是 HTTP REST 服务)的期望。

我们将使用由`Pact`提供的 DSL 来建立这个合同。定义好之后，我们可以使用基于定义好的契约创建的模拟服务来测试消费者和提供者之间的交互。此外，我们将通过使用模拟客户端来测试服务是否符合合同。

## 3。Maven 依赖关系

首先，我们需要将 Maven 依赖项添加到 [`pact-jvm-consumer-junit5_2.12`](https://web.archive.org/web/20220626082031/https://search.maven.org/search?q=g:au.com.dius%20AND%20a:pact-jvm-consumer-junit5_2.12) 库中:

```java
<dependency>
    <groupId>au.com.dius</groupId>
    <artifactId>pact-jvm-consumer-junit5_2.12</artifactId>
    <version>3.6.3</version>
    <scope>test</scope>
</dependency>
```

## 4。定义合同

当我们想要使用`Pact`创建一个测试时，首先我们需要用将要使用的提供者来注释我们的测试类:

```java
@PactTestFor(providerName = "test_provider", hostInterface="localhost")
public class PactConsumerDrivenContractUnitTest
```

我们将传递提供者名称和主机，服务器模拟(根据契约创建)将在其上启动。

假设服务已经为它可以处理的两个 HTTP 方法定义了契约。

第一个方法是 GET 请求，它返回带有两个字段的 JSON。当请求成功时，它返回 200 HTTP 响应代码和 JSON 的 C `ontent-Type` 头。

让我们用`Pact`来定义这样一个契约。

**我们需要使用`@Pact` 注释，并传递为其定义合同的消费者名称。**在带注释的方法里面，我们可以定义我们的 GET 契约:

```java
@Pact(consumer = "test_consumer")
public RequestResponsePact createPact(PactDslWithProvider builder) {
    Map<String, String> headers = new HashMap<>();
    headers.put("Content-Type", "application/json");

    return builder
      .given("test GET")
        .uponReceiving("GET REQUEST")
        .path("/pact")
        .method("GET")
      .willRespondWith()
        .status(200)
        .headers(headers)
        .body("{\"condition\": true, \"name\": \"tom\"}")
        (...)
}
```

使用`Pact` DSL，我们定义对于一个给定的 GET 请求，我们想要返回一个带有特定头和主体的 200 响应。

我们合同的第二部分是邮寄方式。当客户机用适当的 JSON 主体向路径`/pact` 发送 POST 请求时，它返回 201 HTTP 响应代码。

让我们用`Pact:`来定义这样的合同

```java
(...)
.given("test POST")
.uponReceiving("POST REQUEST")
  .method("POST")
  .headers(headers)
  .body("{\"name\": \"Michael\"}")
  .path("/pact")
.willRespondWith()
  .status(201)
.toPact();
```

注意，我们需要在契约的末尾调用`toPact()` 方法来返回`RequestResponsePact`的一个实例。

### 4.1。产生契约神器

默认情况下，Pact 文件将在`target/pacts`文件夹中生成。为了定制这个路径，我们可以配置`maven-surefire-plugin:`

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <systemPropertyVariables>
            <pact.rootDir>target/mypacts</pact.rootDir>
        </systemPropertyVariables>
    </configuration>
    ...
</plugin>
```

Maven 构建将在`target/mypacts`文件夹中生成一个名为`test_consumer-test_provider.json`的文件，其中包含请求和响应的结构:

```java
{
    "provider": {
        "name": "test_provider"
    },
    "consumer": {
        "name": "test_consumer"
    },
    "interactions": [
        {
            "description": "GET REQUEST",
            "request": {
                "method": "GET",
                "path": "/"
            },
            "response": {
                "status": 200,
                "headers": {
                    "Content-Type": "application/json"
                },
                "body": {
                    "condition": true,
                    "name": "tom"
                }
            },
            "providerStates": [
                {
                    "name": "test GET"
                }
            ]
        },
        {
            "description": "POST REQUEST",
            ...
        }
    ],
    "metadata": {
        "pact-specification": {
            "version": "3.0.0"
        },
        "pact-jvm": {
            "version": "3.6.3"
        }
    }
}
```

## 5。使用合同测试客户端和提供商

现在我们有了契约，我们可以使用为客户端和提供者创建测试。

这些测试中的每一项都将使用基于合同的模拟测试，这意味着:

*   客户端将使用模拟提供者
*   提供者将使用一个模拟客户端

实际上，测试是根据合同进行的。

### 5.1。测试客户端

一旦我们定义了契约，我们就可以测试与基于该契约创建的服务的交互。**我们可以创建普通的 JUnit 测试，但是我们需要记住将`@PactTestFor` 注释放在测试的开始。**

让我们为 GET 请求编写一个测试:

```java
@Test
@PactTestFor
public void givenGet_whenSendRequest_shouldReturn200WithProperHeaderAndBody() {

    // when
    ResponseEntity<String> response = new RestTemplate()
      .getForEntity(mockProvider.getUrl() + "/pact", String.class);

    // then
    assertThat(response.getStatusCode().value()).isEqualTo(200);
    assertThat(response.getHeaders().get("Content-Type").contains("application/json")).isTrue();
    assertThat(response.getBody()).contains("condition", "true", "name", "tom");
}
```

**`@PactTestFor` 注释负责启动 HTTP 服务，可以放在测试类或测试方法上。**在测试中，我们只需要发送 GET 请求并断言我们的响应符合约定。

让我们也为 POST 方法调用添加测试:

```java
HttpHeaders httpHeaders = new HttpHeaders();
httpHeaders.setContentType(MediaType.APPLICATION_JSON);
String jsonBody = "{\"name\": \"Michael\"}";

// when
ResponseEntity<String> postResponse = new RestTemplate()
  .exchange(
    mockProvider.getUrl() + "/create",
    HttpMethod.POST,
    new HttpEntity<>(jsonBody, httpHeaders), 
    String.class
);

//then
assertThat(postResponse.getStatusCode().value()).isEqualTo(201);
```

正如我们所看到的，POST 请求的响应代码等于 201——正如在`Pact` 契约中定义的那样。

当我们使用`@PactTestFor()` 注释时，`Pact` 库根据测试用例之前定义的契约启动 web 服务器。

### 5.2。测试供应商

我们的契约验证的第二步是使用基于契约的模拟客户机为提供者创建一个测试。

我们的提供者实现将由这个契约以 TDD 的方式驱动。

对于我们的例子，我们将使用一个 Spring Boot REST API。

首先，为了创建我们的 JUnit 测试，我们需要添加[pact-JVM-provider-JUnit 5 _ 2.12](https://web.archive.org/web/20220626082031/https://search.maven.org/search?q=a:pact-jvm-provider-junit5_2.12)依赖项:

```java
<dependency>
    <groupId>au.com.dius</groupId>
    <artifactId>pact-jvm-provider-junit5_2.12</artifactId>
    <version>3.6.3</version>
</dependency>
```

**这允许我们创建一个 JUnit 测试，指定提供者名称和 Pact 工件的位置:**

```java
@Provider("test_provider")
@PactFolder("pacts")
public class PactProviderLiveTest {
    //...
}
```

为了让这个配置工作，我们必须将`test_consumer-test_provider.json`文件放在 REST 服务项目的`pacts`文件夹中。

接下来，为了用 JUnit 5 编写 Pact 验证测试，我们需要使用带有`@TestTemplate`注释的`PactVerificationInvocationContextProvider`。我们需要给它传递`PactVerificationContext`参数，我们将使用它来设置目标 Spring Boot 应用程序的细节:

```java
private static ConfigurableWebApplicationContext application;

@TestTemplate
@ExtendWith(PactVerificationInvocationContextProvider.class)
void pactVerificationTestTemplate(PactVerificationContext context) {
    context.verifyInteraction();
}

@BeforeAll
public static void start() {
    application = (ConfigurableWebApplicationContext) SpringApplication.run(MainApplication.class);
}

@BeforeEach
void before(PactVerificationContext context) {
    context.setTarget(new HttpTestTarget("localhost", 8082, "/spring-rest"));
}
```

最后，我们将在契约中指定我们想要测试的状态:

```java
@State("test GET")
public void toGetState() { }

@State("test POST")
public void toPostState() { }
```

运行这个 JUnit 类将为两个 GET 和 POST 请求执行两个测试。让我们来看看日志:

```java
Verifying a pact between test_consumer and test_provider
  Given test GET
  GET REQUEST
    returns a response which
      has status code 200 (OK)
      includes headers
        "Content-Type" with value "application/json" (OK)
      has a matching body (OK)

Verifying a pact between test_consumer and test_provider
  Given test POST
  POST REQUEST
    returns a response which
      has status code 201 (OK)
      has a matching body (OK)
```

注意，这里我们没有包含创建 REST 服务的代码。完整的服务和测试可以在 GitHub 项目中找到。

## 6。结论

在这个快速教程中，我们看了一下消费者驱动的合同。

我们使用`Pact` 库创建了一个契约。一旦我们定义了契约，我们就能够根据契约测试客户机和服务，并断言它们符合规范。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220626082031/https://github.com/eugenp/tutorials/tree/master/libraries-5)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。