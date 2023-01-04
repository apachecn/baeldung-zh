# OpenAPI JSON 对象作为查询参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/openapi-json-query-parameters>

## 1.概观

在本教程中，我们将学习如何使用 OpenAPI 将 **JSON 对象作为查询参数。**

## 2.OpenAPI 2 中的查询参数

**OpenAPI 2 不支持对象作为查询参数**；仅支持基元值和基元数组。

正因为如此，我们想将我们的 JSON 参数定义为一个`string.`

为了看到这一点，让我们将名为`params` 的参数定义为`string`，尽管我们将在后端将其解析为 JSON:

```java
swagger: "2.0"
...
paths:
  /tickets:
    get:
      tags:
      - "tickets"
      summary: "Send an JSON Object as a query param"
      parameters:
      - name: "params"
        in: "path"
        description: "{\"type\":\"foo\",\"color\":\"green\"}"
        required: true
        type: "string" 
```

因此，代替:

```java
GET http://localhost:8080/api/tickets?type=foo&color;=green
```

我们会做:

```java
GET http://localhost:8080/api/tickets?params={"type":"foo","color":"green"}
```

## 3\. Query Params in OpenAPI 3

OpenAPI 3 引入了对对象作为查询参数的支持。

要指定 JSON 参数，我们需要在定义中添加一个`content` 部分，其中包含 MIME 类型和模式:

```java
openapi: 3.0.1
...
paths:
  /tickets:
    get:
      tags:
      - tickets
      summary: Send an JSON Object as a query param
      parameters:
      - name: params
        in: query
        description: '{"type":"foo","color":"green"}'
        required: true
        schema:
          type: object
          properties:
            type:
              type: "string"
            color:
              type: "string" 
```

我们的请求现在看起来像这样:

```java
GET http://localhost:8080/api/tickets?params[type]=foo&params;[color]=green
```

实际上，它看起来仍然像:

```java
GET http://localhost:8080/api/tickets?params={"type":"foo","color":"green"}
```

第一个选项允许我们使用参数验证，这将让我们在发出请求之前知道是否有问题。

对于第二种选择，我们用它来换取对后端更好的控制以及 OpenAPI 2 的兼容性。

## 4.URL 编码

值得注意的是，在决定将请求参数作为 JSON 对象传输时，我们希望对参数进行 URL 编码以确保安全传输。

所以，要发送以下 URL:

```java
GET /tickets?params={"type":"foo","color":"green"}
```

我们实际上会做:

```java
GET /tickets?params=%7B%22type%22%3A%22foo%22%2C%22color%22%3A%22green%22%7D
```

## 5.限制

此外，让我们记住将 JSON 对象作为一组查询参数传递的限制:

*   安全性降低
*   参数的有限长度

例如，**我们在查询参数**中放置的数据越多，出现在服务器日志中的数据就越多，**暴露敏感数据的可能性就越大。**

此外，单个查询参数不能超过 2048 个字符。当然，我们都可以想象 JSON 对象比这个大的场景。实际上，JSON 字符串**的 URL 编码实际上将我们的有效负载限制在 1000 个字符左右。**

一种解决方法是在主体中发送更大的 JSON 对象。通过这种方式，我们解决了安全问题和 JSON 长度限制。

其实， **GET 或者 POST 都支持这个。**通过 GET 发送 body 的一个原因是为了维护 API 的 RESTful 语义。

当然，这有点不寻常，也没有得到普遍支持。例如，一些 JavaScript HTTP 库不允许 GET 请求有请求体。

简而言之，这种选择是语义和通用兼容性之间的权衡。

## 6.结论

总之，在本文中，我们学习了如何使用 OpenAPI 将 JSON 对象指定为查询参数。然后，我们观察了后端的一些影响。

GitHub 上的[提供了这些例子的完整 OpenAPI 定义。](https://web.archive.org/web/20220626080516/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)