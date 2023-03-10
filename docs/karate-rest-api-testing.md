# 用空手道进行 REST API 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/karate-rest-api-testing>

## 1。概述

在本文中，我们将介绍[空手道](https://web.archive.org/web/20220524064828/https://github.com/intuit/karate)，一个 Java 的行为驱动开发(BDD)测试框架。

## 2。空手道和 BDD

**空手道是建立在黄瓜**之上的 **，另一个 [BDD 测试](/web/20220524064828/https://www.baeldung.com/cs/bdd-guide)框架，并且共享一些相同的概念。其中之一是**使用一个小黄瓜文件，它描述了被测试的特性**。然而，与 Cucumber 不同，测试不是用 Java 编写的，而是在 Gherkin 文件中有完整的描述。**

小黄瓜文件以“`.feature”`扩展名保存。它以`Feature`关键字开始，后面是同一行的特性名。它还包含不同的测试场景，每一个场景都以关键字`Scenario`开始，并由关键字`Given`、`When`、`Then`、`And`和`But`的多个步骤组成。

更多关于黄瓜和小黄瓜结构的信息可以在这里找到。

## 3。Maven 依赖关系

为了在 Maven 项目中使用空手道，我们需要向`pom.xml`添加 [`karate-apache`](https://web.archive.org/web/20220524064828/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.intuit.karate%22%20AND%20a%3A%22karate-apache%22) 依赖项:

```java
<dependency>
    <groupId>com.intuit.karate</groupId>
    <artifactId>karate-apache</artifactId>
    <version>0.6.0</version>
</dependency>
```

我们还需要 [`karate-junit4`](https://web.archive.org/web/20220524064828/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.intuit.karate%22%20AND%20a%3A%22karate-junit4%22) 依赖来促进 JUnit 测试:

```java
<dependency>
    <groupId>com.intuit.karate</groupId>
    <artifactId>karate-junit4</artifactId>
    <version>0.6.0</version>
</dependency>
```

## 4。创建测试

我们将从在一个小黄瓜`Feature`文件中为一些常见场景编写测试开始。

### 4.1。测试状态代码

让我们编写一个测试 GET 端点并检查它是否返回一个`200` (OK) HTTP 状态代码的场景:

```java
Scenario: Testing valid GET endpoint
Given url 'http://localhost:8097/user/get'
When method GET
Then status 200
```

这显然适用于所有可能的 HTTP 状态代码。

### 4.2。测试响应

让我们编写另一个测试 REST 端点返回特定响应的场景:

```java
Scenario: Testing the exact response of a GET endpoint
Given url 'http://localhost:8097/user/get'
When method GET
Then status 200
And match $ == {id:"1234",name:"John Smith"}
```

**`match`操作用于确认**，其中‘`$'`代表响应。因此，上面的场景检查响应是否与'`{id:”1234″,name:”John Smith”}'.`'完全匹配

我们还可以专门检查`id`字段的值:

```java
And match $.id == "1234"
```

**`match`操作也可用于检查响应是否包含某些字段。**当仅需要检查某些字段或并非所有响应字段都已知时，这很有帮助:

```java
Scenario: Testing that GET response contains specific field
Given url 'http://localhost:8097/user/get'
When method GET
Then status 200
And match $ contains {id:"1234"}
```

### 4.3。用标记验证响应值

在我们不知道返回的确切值的情况下，我们仍然可以使用`markers` —响应中匹配字段的占位符来验证该值。

例如，我们可以使用一个标记来表示我们是否期望一个`null`值:

*   `#null`
*   `#notnull`

或者，我们可以使用标记来匹配字段中的特定类型的值:

*   `#boolean`
*   `#number`
*   `#string`

当我们期望一个字段包含一个 JSON 对象或数组时，可以使用其他标记:

*   `#array`
*   `#object`

有一些标记用于匹配特定的格式或正则表达式，还有一个标记用于计算布尔表达式:

*   `#uuid —`值符合 UUID 格式
*   `#regex STR —`值匹配正则表达式`STR`
*   `#? EXPR —`断言 JavaScript 表达式`EXPR`的计算结果为`true`

最后，如果我们不想对字段进行任何检查，我们可以使用`#ignore`标记。

让我们重写上面的场景，以检查`id`字段不是`null`:

```java
Scenario: Test GET request exact response
Given url 'http://localhost:8097/user/get'
When method GET
Then status 200
And match $ == {id:"#notnull",name:"John Smith"}
```

### 4.4。用请求体测试 POST 端点

让我们来看最后一个场景，它测试一个 POST 端点并接受一个请求体:

```java
Scenario: Testing a POST endpoint with request body
Given url 'http://localhost:8097/user/create'
And request { id: '1234' , name: 'John Smith'}
When method POST
Then status 200
And match $ contains {id:"#notnull"}
```

## 5。运行测试

既然测试场景已经完成，我们可以通过集成空手道和 JUnit 来运行我们的测试。

我们将使用`@CucumberOptions`注释来指定`Feature`文件的确切位置:

```java
@RunWith(Karate.class)
@CucumberOptions(features = "classpath:karate")
public class KarateUnitTest {
//...     
}
```

为了演示 REST API，我们将使用一个 [WireMock 服务器。](/web/20220524064828/https://www.baeldung.com/introduction-to-wiremock)

对于这个例子，我们模拟了在用`@BeforeClass`标注的方法中测试的所有端点。我们将在用`@AfterClass`标注的方法中关闭 WireMock 服务器:

```java
private static WireMockServer wireMockServer
  = new WireMockServer(WireMockConfiguration.options().port(8097));

@BeforeClass
public static void setUp() throws Exception {
    wireMockServer.start();
    configureFor("localhost", 8097);
    stubFor(
      get(urlEqualTo("/user/get"))
        .willReturn(aResponse()
          .withStatus(200)
          .withHeader("Content-Type", "application/json")
          .withBody("{ \"id\": \"1234\", name: \"John Smith\" }")));

    stubFor(
      post(urlEqualTo("/user/create"))
        .withHeader("content-type", equalTo("application/json"))
        .withRequestBody(containing("id"))
        .willReturn(aResponse()
          .withStatus(200)
          .withHeader("Content-Type", "application/json")
          .withBody("{ \"id\": \"1234\", name: \"John Smith\" }")));

}

@AfterClass
public static void tearDown() throws Exception {
    wireMockServer.stop();
}
```

当我们运行`KarateUnitTest`类时，REST 端点由 WireMock 服务器创建，并且运行指定特性文件中的所有场景。

## 6。结论

在本教程中，我们学习了如何使用空手道测试框架来测试 REST APIs。

本文的完整源代码和所有代码片段可以在 GitHub 的[中找到。](https://web.archive.org/web/20220524064828/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-testing)