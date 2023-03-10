# RAML 介绍 RESTful API 建模语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial>

[This article is part of a series:](javascript:void(0);)• Introduction to RAML – The RESTful API Modeling Language (current article)[• Eliminate Redundancies in RAML with Resource Types and Traits](/web/20220523224944/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)
[• Modular RAML Using Includes, Libraries, Overlays and Extensions](/web/20220523224944/https://www.baeldung.com/modular-raml-includes-overlays-libraries-extensions)
[• Define Custom RAML Properties Using Annotations](/web/20220523224944/https://www.baeldung.com/raml-custom-properties-with-annotations)

## 1。概述

在本文中，我们介绍了 [RESTful API 建模语言(RAML)](https://web.archive.org/web/20220523224944/http://www.raml.org/) ，这是一种厂商中立的开放规范语言，构建在 [YAML 1.2](https://web.archive.org/web/20220523224944/http://yaml.org/spec/1.2/spec.html) 和 JSON 之上，用于描述 RESTful API。

在演示如何定义一个简单的基于 JSON 的 API 时，我们将介绍基本的 RAML 1.0 语法和文件结构。我们还将展示如何通过使用`includes`来简化 RAML 文件的维护。如果您有使用 JSON 模式的遗留 API，我们将展示如何将模式合并到 RAML 中。

然后，我们将介绍一些可以增强您进入 RAML 之旅的工具，包括创作工具、文档生成器等等。

最后，我们将通过描述 RAML 规范的当前状态进行总结。

## 延伸阅读:

## [Spring REST API 的二进制数据格式](/web/20220523224944/https://www.baeldung.com/spring-rest-api-with-binary-data-formats)

In this article we explore how to configure Spring REST mechanism to utilize binary data formats which we illustrate with Kryo. Moreover we show how to support multiple data formats with Google Protocol buffers.[Read more](/web/20220523224944/https://www.baeldung.com/spring-rest-api-with-binary-data-formats) →

## [放心指南](/web/20220523224944/https://www.baeldung.com/rest-assured-tutorial)

Explore the basics of REST-assured - a library that simplifies the testing and validation of REST APIs.[Read more](/web/20220523224944/https://www.baeldung.com/rest-assured-tutorial) →

## [用 Javalin 创建 REST 微服务](/web/20220523224944/https://www.baeldung.com/javalin-rest-microservices)

Learn how to implement REST APIs with the Javalin framework[Read more](/web/20220523224944/https://www.baeldung.com/javalin-rest-microservices) →

## 2。定义你的 API(创建`.raml`文件)

我们将定义的 API 相当简单:给定实体类型`Foo`，定义基本的 CRUD 操作和一些查询操作。以下是我们将为 API 定义的资源:

*   `GET /api/v1/foos`
*   `POST /api/v1/foos`
*   `GET /api/v1/foos/{id}`
*   `PUT /api/v1/foos/{id}`
*   `DELETE /api/v1/foos/{id}`
*   `GET /api/v1/foos/name/{name}`
*   `GET /api/v1/foos?name={name}&ownerName;={ownerName}`

让我们将我们的 API 定义为无状态的，使用 HTTP 基本认证，并通过 HTTPS 加密交付。最后，让我们选择 JSON 作为我们的数据传输格式(也支持 XML)。

### 2.1。根级别设置

我们将首先创建一个名为`api.raml`的简单文本文件(建议使用前缀`.raml`；名称是任意的)并在第一行添加 RAML 版本头。在文件的根级别，我们定义了适用于整个 API 的设置:

```java
#%RAML 1.0
title: Baeldung Foo REST Services API using Data Types
version: v1
protocols: [ HTTPS ] 
baseUri: http://myapi.mysite.com/api/{version}
mediaType: application/json 
```

请注意第 3 行中“`version`”一词周围使用了大括号{ }。这就是我们如何告诉 RAML "`version”`指的是一个属性，并且要被扩展。因此实际的`baseUri`将是:**http://myapi.mysite.com/v1**

[注意:`version`属性是可选的，不必是`baseUri`的一部分。]

### 2.2。安全性

安全性也在`.raml`文件的根级别定义。因此，让我们添加我们的 HTTP 基本安全方案定义:

```java
securitySchemes:
  basicAuth:
    description: Each request must contain the headers necessary for
                 basic authentication
    type: Basic Authentication
    describedBy:
      headers:
        Authorization:
          description: Used to send the Base64-encoded "username:password"
                       credentials
          type: string
      responses:
        401:
          description: |
            Unauthorized. Either the provided username and password
            combination is invalid, or the user is not allowed to access
            the content provided by the requested URL.
```

### 2.3。数据类型

接下来，我们将定义 API 将使用的数据类型:

```java
types:
  Foo:
    type: object
    properties:
      id:
        required: true
        type: integer
      name:
        required: true
        type: string
      ownerName:
        required: false
        type: string
```

上面的例子使用扩展的语法来定义我们的数据类型。RAML 提供了一些语法快捷方式，使我们的类型定义不那么冗长。下面是使用这些快捷方式的等效数据类型部分:

```java
types:
  Foo:
    properties:
      id: integer
      name: string
      ownerName?: string
  Error:
    properties:
      code: integer
      message: string
```

那个“？”属性名后的字符声明该属性不是必需的。

### 2.4。资源

现在，我们将定义 API 的顶级资源(URI ):

```java
/foos:
```

### 2.5。URI 参数

接下来，我们将从顶级资源开始扩展资源列表:

```java
/foos:
  /{id}:
  /name/{name}: 
```

这里，属性名两边的大括号{ }定义了 URI 参数。它们代表每个 URI 中的占位符，并不像我们在上面的`baseUri`声明中看到的那样引用根级 RAML 文件属性。添加的线代表资源`/foos/{id}`和`/foos/name/{name}`。

### 2.6。方法

下一步是定义适用于每个资源的 HTTP 方法:

```java
/foos:
  get:
  post:
  /{id}:
    get:
    put:
    delete:
  /name/{name}:
    get:
```

### 2.7。查询参数

现在我们将定义一种使用查询参数查询`foos`集合的方法。请注意，查询参数是使用我们在上面为数据类型使用的相同语法来定义的:

```java
/foos:
  get:
    description: List all Foos matching query criteria, if provided;
                 otherwise list all Foos
    queryParameters:
      name?: string
      ownerName?: string
```

### 2.8。回应

既然我们已经定义了 API 的所有资源，包括 URI 参数、HTTP 方法和查询参数，那么是时候定义预期的响应和状态代码了。响应格式通常根据数据类型和示例来定义。

JSON 模式可以用来代替数据类型，以向后兼容早期版本的 RAML。我们将在第 3 节介绍 JSON 模式。

[注意:在下面的代码片段中，仅包含三个点(…)的一行表示为了简洁起见跳过了一些行。]

让我们从`/foos/{id}:`上的简单 GET 操作开始

```java
/foos:
  ...
  /{id}:
    get:
      description: Get a Foo by id
      responses:
        200:
          body:
            application/json:
              type: Foo
              example: { "id" : 1, "name" : "First Foo" } 
```

这个例子显示，通过对资源`/foos/{id}`执行 GET 请求，我们应该以 JSON 对象和 HTTP 状态代码 200 的形式获得匹配的`Foo`。

下面是我们如何定义对`/foos`资源的 GET 请求:

```java
/foos:
  get:
    description: List all Foos matching query criteria, if provided;
                 otherwise list all Foos
    queryParameters:
      name?: string
      ownerName?: string
    responses:
      200:
        body:
          application/json:
            type: Foo[]
            example: |
              [
                { "id" : 1, "name" : "First Foo" },
                { "id" : 2, "name" : "Second Foo" }
              ] 
```

注意附加到`Foo`类型的方括号[]的使用。这演示了我们如何定义一个包含一组`Foo`对象的响应体，这个例子是一组 JSON 对象。

### 2.9。请求正文

接下来，我们将定义对应于每个 POST 和 PUT 请求的请求体。让我们从创建一个新的`Foo`对象开始:

```java
/foos:
  ...
  post:
    description: Create a new Foo
    body:
      application/json:
        type: Foo
        example: { "id" : 5, "name" : "Another foo" }
    responses:
      201:
        body:
          application/json:
            type: Foo
            example: { "id" : 5, "name" : "Another foo" } 
```

### 2.10。状态代码

注意，在上面的例子中，当创建一个新对象时，我们返回一个 HTTP 状态 201。更新对象的 PUT 操作将返回 HTTP 状态 200，使用与 POST 操作相同的请求和响应主体。

除了请求成功时返回的预期响应和状态代码之外，我们还可以定义发生错误时预期的响应类型和状态代码。

让我们看看，当没有找到具有给定 id 的资源时，我们将如何定义对`/foos/{id}`资源的 GET 请求的预期响应:

```java
 404:
          body:
            application/json:
              type: Error
              example: { "message" : "Not found", "code" : 1001 } 
```

## 3。带有 JSON 模式的 RAML】

在 RAML 1.0 中引入`data types`之前，对象、请求体和响应体是使用 JSON 模式定义的。

使用`data types`可能非常强大，但是有些情况下您仍然希望使用 JSON 模式。在 RAML 0.8 中，您使用根级别`schemas`部分定义了您的模式。

这仍然有效，但是建议使用`types`部分，因为在将来的版本中可能不推荐使用`schemas`。`types`和`schemas`以及`type`和`schema`都是同义词。

下面是如何使用 JSON 模式在`.raml`文件的根级别定义 Foo 对象类型:

```java
types:
  foo: |
    { "$schema": "http://json-schema.org/schema",
       "type": "object",
       "description": "Foo details",
       "properties": {
         "id": { "type": integer },
         "name": { "type": "string" },
         "ownerName": { "type": "string" }
       },
       "required": [ "id", "name" ]
    }
```

下面是如何在 GET `/foos/{id}`资源定义中引用模式:

```java
/foos:
  ...
  /{id}:
    get:
      description: Get a Foo by its id
      responses:
        200:
          body:
            application/json:
              type: foo
              ...
```

## 4。重构包含

从上面几节我们可以看到，我们的 API 变得相当冗长和重复。

RAML 规范提供了一种包含机制，允许我们将重复和冗长的代码段外部化。

我们可以使用 includes 重构我们的 API 定义，使它更加简洁，并且不太可能包含由“到处复制/粘贴/修复”方法导致的错误类型。

例如，我们可以将一个`Foo`对象的数据类型放在文件`types/Foo.raml`中，将一个`Error`对象的数据类型放在`types/Error.raml`中。那么我们的`types`部分将如下所示:

```java
types:
  Foo: !include types/Foo.raml
  Error: !include types/Error.raml
```

如果我们使用 JSON 模式，我们的`types`部分可能看起来像这样:

```java
types:
  foo: !include schemas/foo.json
  error: !include schemas/error.json
```

## 5。完成 API

在将所有数据类型和示例外部化到它们的文件中之后，我们可以使用 include 工具来重构我们的 API:

```java
#%RAML 1.0
title: Baeldung Foo REST Services API
version: v1
protocols: [ HTTPS ]
baseUri: http://rest-api.baeldung.com/api/{version}
mediaType: application/json
securedBy: basicAuth
securitySchemes:
  basicAuth:
    description: Each request must contain the headers necessary for
                 basic authentication
    type: Basic Authentication
    describedBy:
      headers:
        Authorization:
          description: Used to send the Base64 encoded "username:password"
                       credentials
          type: string
      responses:
        401:
          description: |
            Unauthorized. Either the provided username and password
            combination is invalid, or the user is not allowed to access
            the content provided by the requested URL.
types:
  Foo:   !include types/Foo.raml
  Error: !include types/Error.raml
/foos:
  get:
    description: List all Foos matching query criteria, if provided;
                 otherwise list all Foos
    queryParameters:
      name?: string
      ownerName?: string
    responses:
      200:
        body:
          application/json:
            type: Foo[]
            example: !include examples/Foos.json
  post:
    description: Create a new Foo
    body:
      application/json:
        type: Foo
        example: !include examples/Foo.json
    responses:
      201:
        body:
          application/json:
            type: Foo
            example: !include examples/Foo.json
  /{id}:
    get:
      description: Get a Foo by id
      responses:
        200:
          body:
            application/json:
              type: Foo
              example: !include examples/Foo.json
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
    put:
      description: Update a Foo by id
      body:
        application/json:
          type: Foo
          example: !include examples/Foo.json
      responses:
        200:
          body:
            application/json:
              type: Foo
              example: !include examples/Foo.json
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
    delete:
      description: Delete a Foo by id
      responses:
        204:
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
  /name/{name}:
    get:
      description: List all Foos with a certain name
      responses:
        200:
          body:
            application/json:
              type: Foo[]
              example: !include examples/Foos.json 
```

## 6。RAML 工具

RAML 的一大优点是工具支持。

有用于解析、验证和创作 RAML APIs 的工具；客户端代码生成工具；用于生成 HTML 和 PDF 格式的 API 文档的工具；以及帮助我们根据 RAML API 规范进行测试的工具。

甚至有一个工具可以将 Swagger JSON API 转换成 RAML。

以下是可用工具的示例:

*   [API Designer](https://web.archive.org/web/20220523224944/https://github.com/mulesoft/api-designer)–一款基于网络的工具，旨在实现快速高效的 API 设计
*   [API 工作台](https://web.archive.org/web/20220523224944/https://atom.io/packages/api-workbench)–用于设计、构建、测试和记录支持 RAML 0.8 和 1.0 的 RESTful APIs 的 IDE
*   [RAML Cop](https://web.archive.org/web/20220523224944/https://github.com/thebinarypenguin/raml-cop)-一个验证 RAML 文件的工具
*   用于 JAX-RS 的 RAML——一套工具，用于从 RAML 规范生成 Java + JAX-RS 应用程序代码的框架，或者从现有的 JAX-RS 应用程序生成 RAML 规范
*   RAML Sublime 插件-Sublime 文本编辑器的语法高亮插件
*   从 RAML 到 HTML 的工具
*   [raml 2 pdf](https://web.archive.org/web/20220523224944/https://github.com/our-bts/raml2pdf)–从 RAML 生成 PDF 文档的工具
*   ram L2 Wiki–一个生成 Wiki 文档的工具(使用 Confluence/JIRA 标记)
*   SoapUI RAML 插件——一个流行的 SoapUI 功能 API 测试套件的 RAML 插件
*   Vigia T1——一个能够基于 RAML 定义生成测试用例的集成测试套件

有关 RAML 工具和相关项目的完整列表，请访问 [RAML 项目](https://web.archive.org/web/20220523224944/https://raml.org/projects)页面。

## 7。RAML 的当前状态

RAML 1.0 (RC)规范在 2015 年 11 月 3 日获得了发布候选状态，在撰写本文时，1.0 版本有望在本月内完成。

它的前身 RAML 0.8 最初是在 2014 年秋季发布的，现在仍有许多工具支持它。

## 8。延伸阅读

这里有一些链接，在我们的 RAML 之旅中可能会有用。

*   [RAML.org](https://web.archive.org/web/20220523224944/http://raml.org/)——RAML 规范的官方网站
*   [json-schema.org](https://web.archive.org/web/20220523224944/https://json-schema.org/)——JSON 模式的发源地
*   [了解 JSON 模式](https://web.archive.org/web/20220523224944/https://spacetelescope.github.io/understanding-json-schema/)
*   [JSON 模式生成器](https://web.archive.org/web/20220523224944/http://jsonschema.net/)
*   [维基百科 RAML 页面](https://web.archive.org/web/20220523224944/https://en.wikipedia.org/wiki/RAML_(software))

## 9。结论

本文介绍了 RESTful API 建模语言(RAML)。我们演示了使用 RAML 1.0 (RC)规范编写简单 API 规范的一些基本语法。

我们还看到了通过使用语法快捷方式和将示例、数据类型和模式外化到“include”文件中来使我们的定义更加简洁的方法。

然后，我们介绍了一组功能强大的工具，它们与 RAML 规范一起工作，帮助完成日常的 API 设计、开发、测试和文档任务。

随着该规范 1.0 版本的正式发布，加上工具开发人员的大力支持，RAML 似乎会继续存在下去。

Next **»**[Eliminate Redundancies in RAML with Resource Types and Traits](/web/20220523224944/https://www.baeldung.com/simple-raml-with-resource-types-and-traits)