# GraphQL 与 REST

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/graphql-vs-rest>

## 1.概观

当创建 web 服务来支持我们的应用程序时，我们可以选择使用 [REST](/web/20220617075716/https://www.baeldung.com/category/rest/) 或 GraphQL 作为通信模式。虽然两者都最有可能使用 JSON 而不是 HTTP，但是它们有不同的优缺点。

在本教程中，我们将比较 GraphQL 和 REST。我们将创建一个产品数据库示例，并比较这两种解决方案在执行相同的客户端操作时有何不同。

## 2.示例服务

我们的示例服务将允许我们:

*   创建草稿状态的产品
*   更新产品详细信息
*   获取产品列表
*   获取单个产品及其订单的详细信息

让我们从使用 REST 创建应用程序开始。

## 3.休息

REST(表述性状态转移)是分布式超媒体系统的一种架构风格。REST 中的主要数据元素被称为[资源](https://web.archive.org/web/20220617075716/https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#tab_5_1)。在本例中，资源是`“product”`。

### 3.1.创建产品

为了创建一个产品，我们将在 API 上使用一个`POST`方法:

```java
curl -X POST -H "Content-Type:application/json" -d '
{
  "name": "Watch",
  "description": "Special Swiss Watch",
  "status": "Draft",
  "currency": "USD",
  "price": null,
  "image_url": null,
  "video_url": null,
  "stock": null,
  "average_rating": null
}' https://graphqlvsrest.com/product
```

因此，系统将创建一个新产品。

### 3.2.更新产品

在 REST 中，我们通常使用`PUT`方法更新产品:

```java
curl -X PUT -H "Content-Type:application/json" -d '
{
  "name": "Watch",
  "description": "Special Swiss Watch",
  "status": "Draft",
  "currency": "USD",
  "price": 1200.0,
  "image_url": ["https://graphqlvsrest.com/imageurl/product-id"],
  "video_url": ["https://graphqlvsrest.com/videourl/product-id"],
  "stock": 10,
  "average_rating": 0.0
}' 
https://graphqlvsrest.com/product/{product-id}
```

因此，`{product-id}`对象上将会有一个更新。

### 3.3.获取产品列表

列出产品通常是一个`GET`操作，我们可以使用一个查询字符串来指定分页:

```java
curl -X GET https://graphqlvsrest.com/product?size=10&page;=0
```

第一个响应对象是:

```java
{
  "id": 1,
  "name": "T-Shirt",
  "description": "Special beach T-Shirt",
  "status": Published,
  "currency": "USD",
  "price": 30.0,
  "image_url": ["https://graphqlvsrest.com/imageurl/1"], 
  "video_url": ["https://graphqlvsrest.com/videourl/1"], 
  "stock": 10, 
  "average_rating": 3.5 
}
```

### 3.4.通过订单获得单一产品

为了获得一个产品及其订单，我们通常期望获得一个带有以前 API 的产品列表，然后调用一个`orde` `r` 资源来查找相关订单:

```java
curl -X GET https://graphqlvsrest.com/order?product-id=1
```

第一个响应对象是:

```java
{
  "id": 1,
  "product_id": 1,
  "customer_uuid": "de68a771-2fcc-4e6b-a05d-e30a8dd0d756",
  "status": "Delivered",
  "address": "43-F 12th Street",
  "creation_date": "Mon Jan 17 01:00:18 GST 2022"
}
```

当我们通过`product-id`查询订单时，将 id 作为查询参数提供给`GET`操作是有意义的。但是，我们应该注意，除了获取所有产品的原始操作之外，我们还需要对我们感兴趣的每个产品执行一次该操作。这与`[N + 1 Select Problem](/web/20220617075716/https://www.baeldung.com/hibernate-common-performance-problems-in-logs#2-n1-select-issue)`有关。

## 4.GraphQL

GraphQL 是一种用于 API 的查询语言，它带有一个运行时，可以使用现有的数据服务来完成这些查询。

GraphQL 的构建模块是[查询和变异](https://web.archive.org/web/20220617075716/https://graphql.org/learn/queries/)。查询负责获取数据，而突变用于创建和更新。

查询和变异都定义了一个[模式](https://web.archive.org/web/20220617075716/https://graphql.org/learn/schema/)。该模式定义了可能的客户端请求和响应。

让我们使用 [GraphQL 服务器](/web/20220617075716/https://www.baeldung.com/spring-graphql)重新实现我们的示例。

### 4.1.创建产品

让我们使用一个名为`saveProduct`的突变:

```java
curl -X POST -H "Content-Type:application/json" -d'
{
  "query": "mutation {saveProduct (
    product: {
      name: \"Bed-Side Lamp\",
      price: 24.0,
      status: \"Draft\",
      currency: \"USD\"
    }){ id name currency price status}
  }"
}'
https://graphqlvsrest.com/graphql
```

在这个`saveProduct`函数中，圆括号内的一切都是[输入](https://web.archive.org/web/20220617075716/https://graphql.org/learn/schema/#input-types) [类型](https://web.archive.org/web/20220617075716/https://graphql.org/learn/schema/#input-types) [模式](https://web.archive.org/web/20220617075716/https://graphql.org/learn/schema/#input-types)。后面的花括号描述了服务器返回的[字段](https://web.archive.org/web/20220617075716/https://graphql.org/learn/queries/#fields)。

当我们运行变异时，我们应该期待一个带有所选字段的响应:

```java
{
  "data": {
    "saveProduct": {
      "id": "12",
      "name": "Bed-Side Lamp",
      "currency": "USD",
      "price": 24.0,
      "status": "Draft"
    }
  }
}
```

这个请求非常类似于我们在 REST 版本中发出的 POST 请求，尽管我们现在可以稍微定制一下响应。

### 4.2.更新产品

同样，我们可以使用另一个名为`updateProduct`的突变来修改产品:

```java
curl -X POST -H "Content-Type:application/json" -d'
{
  "query": "mutation {updateProduct (id:12
    product: {
      price: 14.0,
      status: \"Publish\"
   }){ id name currency price, status}
  }"
}'
https://graphqlvsrest.com/graphql
```

我们在响应中收到选择的字段:

```java
{
  "data": {
    "updateProduct": {
      "id": "12",
      "name": "Bed-Side Lamp",
      "currency": "USD",
      "price": 14.0,
      "status": "Published"
    }
  }
}
```

正如我们所见， **GraphQL 在响应的格式上提供了灵活性**。

### 4.3.获取产品列表

为了从服务器获取数据，我们将使用一个查询:

```java
curl -X POST -H "Content-Type:application/json" -d
'{
    "query": "query {products(size:10,page:0){ id name status}}"
}'
https://graphqlvsrest.com/graphql
```

在这里，我们还描述了我们希望看到的结果页面:

```java
{
  "data": {
    "products": [
      {
        "id": "1",
        "name": "T-Shirt",
        "status": "Published"
      },...
    ]
  }
}
```

### 4.4.通过订单获得单一产品

使用 GraphQL，我们可以要求 GraphQL 服务器将产品和订单连接在一起:

```java
curl -X POST -H "Content-Type:application/json" -d 
'{
    "query": "query {product(id:1){ id name orders{customer_uuid address status creation_date}}}"
}'
https://graphqlvsrest.com/graphql
```

在这个查询中，我们获取了`id`等于 1 的产品及其订单。这使我们能够在单个操作中发出请求，让我们检查响应:

```java
{
  "data": {
    "product": {
      "id": "1",
      "name": "T-Shirt",
      "orders": [
        {
          "customer_uuid": "de68a771-2fcc-4e6b-a05d-e30a8dd0d756",
          "status": "Delivered",
          "address": "43-F 12th Street",
          "creation_date": "Mon Jan 17 01:00:18 GST 2022"
        }, ...
      ]
    }
  }
}
```

正如我们在这里看到的，响应包含产品的详细信息及其各自的订单。

为了用 REST 实现这一点，我们需要发送两个请求——第一个请求获取产品，第二个请求获取各自的订单。

## 5.比较

这些例子展示了如何使用 GraphQL 来减少客户机和服务器之间的通信量，并允许客户机为响应提供一些格式化规则。

值得注意的是，这些 API 背后的数据源可能仍然必须执行相同的操作来修改或获取数据，但客户端和服务器之间的接口丰富性允许客户端使用 GraphQL 做更少的工作。

让我们进一步比较这两种方法。

### 5.1.灵活且充满活力

GraphQL 允许灵活和动态的查询:

*   客户端应用程序只能请求必填字段
*   [别名](https://web.archive.org/web/20220617075716/https://graphql.org/learn/queries/#aliases)可用于请求带有自定义关键字的字段
*   客户端可以使用查询来管理结果顺序
*   客户端可以更好地与 API 中的任何变化分离，因为没有单一版本的响应对象结构可以遵循

### 5.2.更便宜的操作

每个服务器请求都有往返时间和负载大小的代价。

在 REST 中，我们可能最终会发送多个请求来实现所需的功能。这些多重请求将是一个昂贵的操作。此外，响应负载可能包含客户端应用程序不需要的不必要的数据。

GraphQL 倾向于避免昂贵的操作。我们通常可以使用 GraphQL 在一个请求中获取我们需要的所有数据。

### 5.3.什么时候使用 REST？

GraphQL 不能替代 REST。两者甚至可以在同一个应用程序中共存。托管 GraphQL 端点增加的复杂性可能是值得的，这取决于用例。

在以下情况下，我们可能更喜欢休息:

*   我们的应用程序自然是资源驱动的，其中操作非常直接且完全地与单个资源实体相联系
*   需要 web 缓存，因为 GraphQL 本身不支持它
*   我们需要文件上传，因为 GraphQL 本身不支持它

## 6.结论

在本文中，我们使用一个实际例子比较了 REST 和 GraphQL。

我们看到了如何按照惯例使用所学的每一种方法。

然后，我们讨论了两种方法都没有明显的优势。我们的需求将是在它们之间做出选择的驱动力。偶尔，两者也可以共存。

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220617075716/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries-comparison)