# Java hover fly 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hoverfly>

## 1.概观

在本文中，我们将看看 [Hoverfly](https://web.archive.org/web/20220524034856/https://hoverfly.readthedocs.io/en/latest/) Java 库——它提供了一种创建真正的 API 存根/模拟的简单方法。

## 2。Maven 依赖关系

要使用 Hoverfly，我们需要添加一个 Maven 依赖项:

```java
<dependency>
    <groupId>io.specto</groupId>
    <artifactId>hoverfly-java</artifactId>
    <version>0.8.1</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220524034856/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.specto%22%20AND%20a%3A%22hoverfly-java%22)

## 3.模拟 API

首先，我们将配置 Hoverfly 在模拟模式下运行。定义模拟最简单的方法是使用 DSL。

让我们从一个简单的例子开始，实例化`HoverflyRule`实例:

```java
public static final HoverflyRule rule
  = HoverflyRule.inSimulationMode(dsl(
    service("http://www.baeldung.com")
      .get("/api/courses/1")
      .willReturn(success().body(
        jsonWithSingleQuotes("{'id':'1','name':'HCI'}"))));
```

`SimulationSource`类提供了一个用于初始化 API 定义的`dsl`方法。同样，`HoverflyDSL`的`service`方法允许我们定义一个端点和相关的请求路径。

然后我们调用`willReturn`来声明我们想要得到哪个响应作为回报。我们还使用了`ResponseBuilder`的`success`方法来设置响应状态和主体。

## 4.使用`JUnit`进行测试

使用 JUnit 可以很容易地测试存根 API。

让我们创建一个发送 HTTP 请求的简单测试，看看它是否到达端点:

```java
responseEntity<String> courseResponse
  = restTemplate.getForEntity("http://www.baeldung.com/api/courses/1", String.class);

assertEquals(HttpStatus.OK, courseResponse.getStatusCode());
assertEquals("{\"id\":\"1\",\"name\":\"HCI\"}", courseResponse.getBody());
```

我们使用 Spring Web 模块的`RestTemplate`类实例来发送 HTTP 请求。

## 5.添加延迟

对于特定的 HTTP 方法或特定的 API 调用，可以全局添加延迟。

下面是使用 POST 方法设置请求延迟的代码示例:

```java
SimulationSource.dsl(
  service("http://www.baeldung.com")
    .post("/api/courses")
    .willReturn(success())
    .andDelay(3, TimeUnit.SECONDS)
    .forMethod("POST")
)
```

## 6.请求匹配器

工厂类为 URL 提供了几个匹配器，包括`exactMatch`和`globMatch`。对于 HTTP 主体，它提供。

对于 HTTP 主体，它提供了`JSON/XML`精确匹配和`JSONPath/XPath`匹配。

默认情况下，`exactMatch`匹配器用于 URL 和正文匹配。

以下是不同匹配器的使用示例:

```java
SimulationSource.dsl(
  service(matches("www.*dung.com"))
    .get(startsWith("/api/student")) 
    .queryParam("page", any()) 
    .willReturn(success())

    .post(equalsTo("/api/student"))
    .body(equalsToJson(jsonWithSingleQuotes("{'id':'1','name':'Joe'}")))
    .willReturn(success())

    .put("/api/student/1")
    .body(matchesJsonPath("$.name")) 
    .willReturn(success())

    .post("/api/student")
    .body(equalsToXml("<student><id>2</id><name>John</name></student>"))
    .willReturn(success())

    .put("/api/student/2")
    .body(matchesXPath("/student/name")) 
    .willReturn(success()));
)
```

在这个例子中，`matches`方法用允许通配符搜索的`globMatch`检查 URL。

然后`startsWith` 检查请求路径是否以“`/` api `/student`”开始。我们使用`any`匹配器来允许页面查询参数中所有可能的值。

`equalsToJson` 匹配器确保主体负载与这里给出的 JSON 完全匹配。 *matchesJsonPath* 方法用于检查特定 JSON 路径上的元素是否存在。

类似地，`equalsToXml` 将请求体中给出的 XML 与这里给出的 XML 进行匹配。`matchesXPath` 用一个 XPath 表达式匹配一个主体。

## 7.结论

在这个快速教程中，我们讨论了 Hoverfly Java 库的用法。我们研究了模拟 HTTP 服务、配置端点的 DSL、添加延迟和使用请求匹配器。我们还研究了使用 JUnit 测试这些服务。

一如既往，代码片段一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220524034856/https://github.com/eugenp/tutorials/tree/master/libraries-testing)