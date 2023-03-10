# 与 Spring Boot 的 AWS AppSync

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-appsync-spring>

## 1.介绍

在本文中，我们将和 Spring Boot 一起探索 AWS AppSync。 **AWS AppSync 是一个全面管理的企业级 [GraphQL](/web/20220628150656/https://www.baeldung.com/graphql) 服务，具有实时数据同步和离线编程功能**。

## 2.设置 AWS AppSync

首先，我们需要一个有效的 [AWS 账户](https://web.archive.org/web/20220628150656/https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc)。一旦解决了这个问题，我们就可以从 AWS 控制台搜索 AppSync 了。然后我们再点击 [`Getting Started with AppSync`](https://web.archive.org/web/20220628150656/https://docs.aws.amazon.com/appsync/latest/devguide/welcome.html) 链接。

## 2.1.创建 AppSync API

按照[快速入门指南](https://web.archive.org/web/20220628150656/https://docs.aws.amazon.com/appsync/latest/devguide/quickstart-launch-a-sample-schema.html)创建我们的 API，我们将使用`Event App`示例项目。然后点击`Start`命名并创建应用程序:

[![](img/8b10785a7eca1a8a1d463b44b62a866f.png)](/web/20220628150656/https://www.baeldung.com/wp-content/uploads/2020/05/aws_appsync.jpg)

这将把我们带到 AppSync 应用程序控制台。现在让我们看看我们的 GraphQL 模型。

### 2.2.GraphQL 事件模型

GraphQL 使用一个模式来定义哪些数据对客户机可用，以及如何与 GraphQL 服务器交互。该模式包含查询、变异和各种声明的类型。

为了简单起见，让我们看一下默认 AWS AppSync GraphQL 模式的一部分，我们的`Event` 模型:

```java
type Event {
  id: ID!
  name: String
  where: String
  when: String
  description: String
  # Paginate through all comments belonging to an individual post.
  comments(limit: Int, nextToken: String): CommentConnection
}
```

`Event` 是一个带有一些`String`字段和一个`CommentConnection` 类型的声明类型。**注意`ID`栏上的感叹号。这意味着它是必填/非空字段。**

这应该足以理解我们的模式的基础。然而，要了解更多信息，请访问 GraphQL 网站。

## 3.Spring Boot

现在我们已经在 AWS 端设置好了一切，让我们看看我们的 Spring Boot 客户端应用程序。

### 3.1.Maven 依赖性

为了访问我们的 API，我们将使用 Spring Boot Starter WebFlux 库来访问`WebClient,` Spring 对`RestTemplate`的新替代:

```java
 <dependency> 
      <groupId>org.springframework.boot</groupId> 
      <artifactId>spring-boot-starter-webflux</artifactId> 
    </dependency>
```

查看[我们关于](/web/20220628150656/https://www.baeldung.com/spring-5-webclient) `WebClient `的文章了解更多信息。

### 3.2 .GraphQL 客户端

为了向我们的 API 发出请求，我们将首先使用提供 AWS AppSync API URL 和 API 密钥的`WebClient`构建器`,` 来创建我们的`RequestBodySpec`:

```java
WebClient.RequestBodySpec requestBodySpec = WebClient
    .builder()
    .baseUrl(apiUrl)
    .defaultHeader("x-api-key", apiKey)
    .build()
    .method(HttpMethod.POST)
    .uri("/graphql");
```

**别忘了 API 密钥头，`x-api-key`。API 密钥对我们的 AppSync 应用进行认证。**

## 4.使用 GraphQL 类型

### 4.1.问题

设置我们的查询包括将它添加到消息体中的一个`query`元素:

```java
Map<String, Object> requestBody = new HashMap<>();
requestBody.put("query", "query ListEvents {" 
  + " listEvents {"
  + "   items {"
  + "     id"
  + "     name"
  + "     where"
  + "     when"
  + "     description"
  + "   }"
  + " }"
  + "}");
```

使用我们的`requestBody, `让我们调用我们的`WebClient`来检索响应体:

```java
WebClient.ResponseSpec response = requestBodySpec
    .body(BodyInserters.fromValue(requestBody))
    .accept(MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML)
    .acceptCharset(StandardCharsets.UTF_8)
    .retrieve(); 
```

最后，我们可以得到身体作为`String`:

```java
String bodyString = response.bodyToMono(String.class).block();
assertNotNull(bodyString);
assertTrue(bodyString.contains("My First Event"));
```

### 4.2.突变

GraphQL 允许通过使用突变来更新和删除数据。突变根据需要修改服务器端数据，并遵循与查询类似的语法。

让我们添加一个带有`add`突变查询的新事件:

```java
String queryString = "mutation add {"
  + "    createEvent("
  + "        name:\"My added GraphQL event\""
  + "        where:\"Day 2\""
  + "        when:\"Saturday night\""
  + "        description:\"Studying GraphQL\""
  + "    ){"
  + "        id"
  + "        name"
  + "        description"
  + "    }"
  + "}";

requestBody.put("query", queryString);
```

AppSync 和 GraphQL 最大的优点之一是一个端点 URL 提供了整个模式的所有 CRUD 功能。

我们可以重用同一个`WebClient`来添加、更新和删除数据。我们将简单地根据查询或变异中的回调得到一个新的响应。

```java
assertNotNull(bodyString);
assertTrue(bodyString.contains("My added GraphQL event"));
assertFalse(bodyString.contains("where"));
```

## 5.结论

在本文中，我们研究了如何快速地使用 AWS AppSync 设置 GraphQL 应用程序，并使用 Spring Boot 客户端访问它。

AppSync 通过单个端点为开发者提供了强大的 GraphQL API。要了解更多信息，请看我们关于创建一个 [GraphQL Spring Boot 服务器](/web/20220628150656/https://www.baeldung.com/spring-graphql)的教程。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220628150656/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-app-sync)