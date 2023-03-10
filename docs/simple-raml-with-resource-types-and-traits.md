# 消除资源类型和特征中的冗余

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/simple-raml-with-resource-types-and-traits>

[This article is part of a series:](javascript:void(0);)[• Introduction to RAML – The RESTful API Modeling Language](/web/20220122043704/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial)
• Eliminate Redundancies in RAML with Resource Types and Traits (current article)[• Modular RAML Using Includes, Libraries, Overlays and Extensions](/web/20220122043704/https://www.baeldung.com/modular-raml-includes-overlays-libraries-extensions)
[• Define Custom RAML Properties Using Annotations](/web/20220122043704/https://www.baeldung.com/raml-custom-properties-with-annotations)

## 1。概述

在我们的 [RAML 教程](/web/20220122043704/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial)文章中，我们介绍了`RESTful API Modeling Language`并基于一个名为`Foo`的实体创建了一个简单的 API 定义。现在想象一个真实的 API，其中有几个实体类型的资源，都有相同或相似的 GET、POST、PUT 和 DELETE 操作。您可以看到您的 API 文档是如何迅速变得乏味和重复的。

在本文中，我们展示了如何使用 RAML 中的`resource types`和`traits`特性通过提取和参数化公共部分来**消除资源和方法定义中的冗余**，从而消除复制粘贴错误，同时使 API 定义更加简洁。

## 2。我们的 API

为了展示`resource types`和`traits`的好处，我们将通过添加第二个实体类型`Bar`的资源来扩展我们的原始 API。以下是构成我们修订版 API 的资源:

*   `GET /api/v1/foos`
*   `POST /api/v1/foos`
*   `GET /api/v1/foos/{fooId}`
*   `PUT /api/v1/foos/{fooId}`
*   `DELETE /api/v1/foos/{fooId}`
*   `GET /api/v1/foos/name/{name}`
*   `GET /api/v1/foos?name={name}&ownerName;={ownerName}`
*   **T2`GET /api/v1/bars`**
*   **T2`POST /api/v1/bars`**
*   **T2`GET /api/v1/bars/{barId}`**
*   **T2`PUT /api/v1/bars/{barId}`**
*   **T2`DELETE /api/v1/bars/{barId}`**
*   **T2`GET /api/v1/bars/fooId/{fooId}`**

## 3。识别模式

当我们通读 API 中的资源列表时，我们开始看到一些模式的出现。例如，用于创建、读取、更新和删除单个实体的 URIs 和方法有一种模式，用于检索实体集合的 URIs 和方法也有一种模式。collection 和 collection-item 模式是用于在 RAML 定义中提取`resource types`的更常见的模式之一。

让我们看看 API 的几个部分:

[注意:在下面的代码片段中，仅包含三个点(…)的一行表示为了简洁起见跳过了一些行。]

```java
/foos:
  get:
    description: |
      List all foos matching query criteria, if provided;
      otherwise list all foos
    queryParameters:
      name?: string
      ownerName?: string
    responses:
      200:
        body:
          application/json:
            type: Foo[]
  post:
    description: Create a new foo
    body:
      application/json:
        type: Foo
    responses:
      201:
        body:
          application/json:
            type: Foo
...
/bars:
  get:
    description: |
      List all bars matching query criteria, if provided;
      otherwise list all bars
    queryParameters:
      name?: string
      ownerName?: string
    responses:
      200:
        body:
          application/json:
            type: Bar[]
  post:
    description: Create a new bar
    body:
      application/json:
        type: Bar
    responses:
      201:
        body:
          application/json:
            type: Bar
```

当我们比较`/foos`和`/bars`资源的 RAML 定义(包括使用的 HTTP 方法)时，我们可以看到每个资源的各种属性之间有一些冗余，我们再次看到模式开始出现。

只要资源或方法定义中有模式，就有机会使用 RAML `resource type`或`trait`。

## 4。资源类型

为了实现 API 中的模式，`resource types`使用由双尖括号(< <和> >)包围的保留和用户定义的参数。

### 4.1 保留参数

资源类型定义中可以使用两个保留参数:

*   `<<resourcePath>>`代表整个 URI(在`baseURI`之后)，并且
*   `<<resourcePathName>>`代表 URI 最右边的正斜杠(/)后面的部分，忽略任何大括号{ }。

当在资源定义中处理时，它们的值是根据正在定义的资源计算的。

例如，给定资源`/foos`,`<<resourcePath>>`将计算为“/foos”，而`<<resourcePathName>>` 将计算为“foos”。

给定资源`/foos/{fooId}` , `<<resourcePath>>`将计算为“/foos/{fooId}”，而`<<resourcePathName>>` 将计算为“foos”。

### 4.2 用户自定义参数

`resource type`定义也可能包含用户定义的参数。与保留参数不同，保留参数的值是根据定义的资源动态确定的，用户定义的参数必须在使用包含它们的`resource type`时赋值，并且这些值不会改变。

用户定义的参数可以在`resource type`定义的开头声明，尽管这样做不是必需的，也不是常见的做法，因为读者通常可以根据它们的名称和使用它们的上下文来判断它们的预期用途。

### 4.3 参数功能

只要使用参数，就可以使用一些有用的文本函数，以便在资源定义中处理参数时转换参数的扩展值。

以下是可用于参数转换的函数:

*   ！`singularize`
*   ！`pluralize`
*   ！`uppercase`
*   ！`lowercase`
*   ！`uppercamelcase`
*   ！`lowercamelcase`
*   ！`upperunderscorecase`
*   ！`lowerunderscorecase`
*   ！`upperhyphencase`
*   ！`lowerhyphencase`

使用以下结构将函数应用于参数:

< <`parameterName` |！`functionName` > >

如果需要使用多个函数来实现所需的转换，可以用管道符号(“|”)分隔每个函数名，并在前面加上感叹号(！)放在每个函数之前。

例如，给定资源`/foos`，其中< < `resourcePathName` > >计算为“foos”:

*   < <`resourcePathName` |！`singularize` > > == >【喷火】
*   < <`resourcePathName` |！`uppercase`>>= =>【FOOS】
*   < <`resourcePathName` |！`singularize` |！`uppercase` > > == >【喷火】

而给定资源`/bars/{barId}`，其中< < `resourcePathName` > >评估为“酒吧”:

*   < <`resourcePathName` |！`uppercase` > > == >【酒吧】
*   << 【 | !*大写字母* > > == >【酒吧】

## 5。提取集合的资源类型

让我们重构上面显示的`/foos`和`/bars`资源定义，使用一个`resource type`来捕获公共属性。我们将使用保留参数`<<resourcePathName>>`和用户定义参数`<<typeName>>`来表示所使用的数据类型。

### 5.1 定义

下面是一个代表项目集合的`resource type`定义:

```java
resourceTypes:
  collection:
    usage: Use this resourceType to represent any collection of items
    description: A collection of <<resourcePathName>>
    get:
      description: Get all <<resourcePathName>>, optionally filtered 
      responses:
        200:
          body:
            application/json:
              type: <<typeName>>[]
    post:
      description: Create a new <<resourcePathName|!singularize>> 
      responses:
        201:
          body:
            application/json:
              type: <<typeName>>
```

请注意，在我们的 API 中，因为我们的数据类型仅仅是我们的基本资源名称的大写单数形式，所以我们可以将函数应用于< <`resourcePathName` > >参数，而不是引入用户定义的< < `typeName` > >参数，从而为 API 的这一部分实现相同的结果:

```java
resourceTypes:
  collection:
  ...
    get:
      ...
            type: <<resourcePathName|!singularize|!uppercamelcase>>[]
    post:
      ...
            type: <<resourcePathName|!singularize|!uppercamelcase>>
```

### 5.2 应用

使用上面包含< <`typeName` > >参数的定义，下面是如何将“集合”`resource type`应用于资源`/foos`和/ `bars`:

```java
/foos:
  type: { collection: { "typeName": "Foo" } }
  get:
    queryParameters:
      name?: string
      ownerName?: string
...
/bars:
  type: { collection: { "typeName": "Bar" } }
```

请注意，我们仍然能够合并两个资源之间的差异——在本例中是`queryParameters`部分——同时仍然能够利用`resource type`定义所提供的所有优势。

## 6。为集合的单个项目提取资源类型

现在让我们把重点放在 API 中处理集合中单个项目的部分:资源`/foos/{fooId}`和`/bars/{barId}`。这里是`/foos/{fooId}`的代码:

```java
/foos:
...
  /{fooId}:
    get:
      description: Get a Foo
      responses:
        200:
          body:
            application/json:
              type: Foo
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
    put:
      description: Update a Foo
      body:
        application/json:
          type: Foo
      responses:
        200:
          body:
            application/json:
              type: Foo
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
    delete:
      description: Delete a Foo
      responses:
        204:
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
```

`/bars/{barId}`资源定义也有 GET、PUT 和 DELETE 方法，并且与/ `foos/{fooId}`定义相同，除了出现的字符串“foo”和“bar”(以及它们各自的复数和/或大写形式)。

### 6.1 定义

提取我们刚刚确定的模式，下面是我们如何为集合中的单个项目定义一个`resource type`:

```java
resourceTypes:
...
  item:
    usage: Use this resourceType to represent any single item
    description: A single <<typeName>>
    get:
      description: Get a <<typeName>>
      responses:
        200:
          body:
            application/json:
              type: <<typeName>>
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
    put:
      description: Update a <<typeName>>
      body:
        application/json:
          type: <<typeName>>
      responses:
        200:
          body:
            application/json:
              type: <<typeName>>
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
    delete:
      description: Delete a <<typeName>>
      responses:
        204:
        404:
          body:
            application/json:
              type: Error
              example: !include examples/Error.json
```

### 6.2 应用

下面是我们如何应用“item”`resource type`:

```java
/foos:
...
  /{fooId}:
    type: { item: { "typeName": "Foo" } }
```

```java
... 
/bars: 
... 
  /{barId}: 
    type: { item: { "typeName": "Bar" } }
```

## 7 .**。特质**

一个`resource type`用于从资源定义中提取模式，一个`trait`用于从跨资源通用的方法定义中提取模式。

### 7.1 参数

与< <`resourcePath` > >和< < `resourcePathName` > >一起，一个附加的保留参数可用于特征定义:< < `methodName` > >计算为定义了`trait`的 HTTP 方法(GET、POST、PUT、DELETE 等)。用户定义的参数也可以出现在特征定义中，并且在应用时，采用它们所应用的资源的值。

### 7.2 定义

请注意，“item”`resource type`仍然充满冗余。让我们看看`traits`如何帮助消除它们。我们将从提取任何包含请求体的方法的`trait`开始:

```java
traits:
  hasRequestItem:
    body:
      application/json:
        type: <<typeName>>
```

现在让我们为那些正常响应包含主体的方法提取`traits`:

```java
 hasResponseItem:
    responses:
      200:
        body:
          application/json:
            type: <<typeName>>
  hasResponseCollection:
    responses:
      200:
        body:
          application/json:
            type: <<typeName>>[]
```

最后，这里是任何可能返回 404 错误响应的方法的`trait`:

```java
 hasNotFound:
    responses:
      404:
        body:
          application/json:
            type: Error
            example: !include examples/Error.json
```

### 7.3 应用

然后我们将这个`trait`应用到我们的`resource types`:

```java
resourceTypes:
  collection:
    usage: Use this resourceType to represent any collection of items
    description: A collection of <<resourcePathName|!uppercamelcase>>
    get:
      description: |
        Get all <<resourcePathName|!uppercamelcase>>,
        optionally filtered
      is: [ hasResponseCollection: { typeName: <<typeName>> } ]
    post:
      description: Create a new <<resourcePathName|!singularize>>
      is: [ hasRequestItem: { typeName: <<typeName>> } ]
  item:
    usage: Use this resourceType to represent any single item
    description: A single <<typeName>>
    get:
      description: Get a <<typeName>>
      is: [ hasResponseItem: { typeName: <<typeName>> }, hasNotFound ]
    put:
      description: Update a <<typeName>>
      is: | [ hasRequestItem: { typeName: <<typeName>> }, hasResponseItem: { typeName: <<typeName>> }, hasNotFound ]
    delete:
      description: Delete a <<typeName>>
      is: [ hasNotFound ]
      responses:
        204:
```

我们还可以将`traits`应用于资源中定义的方法。这对于资源-方法组合匹配一个或多个`traits`但不匹配任何已定义的`resource type`的“一次性”场景尤其有用:

```java
/foos:
...
  /name/{name}:
    get:
      description: List all foos with a certain name
      is: [ hasResponseCollection: { typeName: Foo } ]
```

## 8。结论

在本教程中，我们已经展示了如何从 RAML API 定义中显著减少或者在某些情况下消除冗余。

首先，我们识别资源的冗余部分，识别它们的模式，并提取`resource types`。然后我们对跨资源通用的方法做了同样的操作来提取`traits`。然后，我们能够通过将`traits`应用到我们的`resource types`和不严格匹配我们定义的`resource types`之一的“一次性”资源-方法组合来进一步消除冗余。

结果，我们的简单 API(只包含两个实体的资源)从 177 行代码减少到 100 多行代码。要了解更多关于 RAML `resource types`和`traits`的信息，请访问 RAML.org[1.0 规范](https://web.archive.org/web/20220122043704/https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#resource-types-and-traits)。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220122043704/https://github.com/eugenp/tutorials/tree/master/raml/resource-types-and-traits)中找到。

以下是我们最终的完整 RAML API:

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
    description: |
      Each request must contain the headers necessary for
      basic authentication
    type: Basic Authentication
    describedBy:
      headers:
        Authorization:
          description: |
            Used to send the Base64 encoded "username:password"
            credentials
            type: string
      responses:
        401:
          description: |
            Unauthorized. Either the provided username and password
            combination is invalid, or the user is not allowed to
            access the content provided by the requested URL.
types:
  Foo:   !include types/Foo.raml
  Bar:   !include types/Bar.raml
  Error: !include types/Error.raml
resourceTypes:
  collection:
    usage: Use this resourceType to represent a collection of items
    description: A collection of <<resourcePathName|!uppercamelcase>>
    get:
      description: |
        Get all <<resourcePathName|!uppercamelcase>>,
        optionally filtered
      is: [ hasResponseCollection: { typeName: <<typeName>> } ]
    post:
      description: |
        Create a new <<resourcePathName|!uppercamelcase|!singularize>>
      is: [ hasRequestItem: { typeName: <<typeName>> } ]
  item:
    usage: Use this resourceType to represent any single item
    description: A single <<typeName>>
    get:
      description: Get a <<typeName>>
      is: [ hasResponseItem: { typeName: <<typeName>> }, hasNotFound ]
    put:
      description: Update a <<typeName>>
      is: [ hasRequestItem: { typeName: <<typeName>> }, hasResponseItem: { typeName: <<typeName>> }, hasNotFound ]
    delete:
      description: Delete a <<typeName>>
      is: [ hasNotFound ]
      responses:
        204:
traits:
  hasRequestItem:
    body:
      application/json:
        type: <<typeName>>
  hasResponseItem:
    responses:
      200:
        body:
          application/json:
            type: <<typeName>>
  hasResponseCollection:
    responses:
      200:
        body:
          application/json:
            type: <<typeName>>[]
  hasNotFound:
    responses:
      404:
        body:
          application/json:
            type: Error
            example: !include examples/Error.json
/foos:
  type: { collection: { typeName: Foo } }
  get:
    queryParameters:
      name?: string
      ownerName?: string
  /{fooId}:
    type: { item: { typeName: Foo } }
  /name/{name}:
    get:
      description: List all foos with a certain name
      is: [ hasResponseCollection: { typeName: Foo } ]
/bars:
  type: { collection: { typeName: Bar } }
  /{barId}:
    type: { item: { typeName: Bar } }
  /fooId/{fooId}:
    get:
      description: Get all bars for the matching fooId
      is: [ hasResponseCollection: { typeName: Bar } ]
```

Next **»**[Modular RAML Using Includes, Libraries, Overlays and Extensions](/web/20220122043704/https://www.baeldung.com/modular-raml-includes-overlays-libraries-extensions)**«** Previous[Introduction to RAML – The RESTful API Modeling Language](/web/20220122043704/https://www.baeldung.com/raml-restful-api-modeling-language-tutorial)