# 415 Spring 应用程序中不支持的 MediaType

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-415-unsupported-mediatype>

## 1.概观

在本教程中，我们将展示 Spring 应用程序中 POST 请求的 HTTP 响应代码 415 不支持的 MediaType 的原因和解决方案。

## 2.背景

我们的一个老客户要求我们为他们的产品设计和开发一个新的桌面应用程序。这个应用程序的目的是管理用户。我们以前从未开发过这种产品。

由于时间紧迫，我们决定使用他们不久前编写的现有后端 API。摆在我们面前的挑战是这些 API 的文档不是非常广泛。因此，我们只确定可用的 API 端点及其方法。因此，我们决定不接触这些服务——相反，我们将开始开发我们的应用程序，它将使用来自这个服务的 API。

## 3.API 请求

我们的应用程序已经开始使用 API。让我们练习这个 API 来获得所有用户:

```java
curl -X GET https://baeldung.service.com/user
```

万岁！API 以成功的响应进行了回应。之后，让我们请求一个用户:

```java
curl -X GET https://baeldung.service.com/user/{user-id}
```

让我们来看看回应:

```java
{
    "id": 1,
    "name": "Jason",
    "age": 23,
    "address": "14th Street"
}
```

这似乎也在起作用。因此，事情看起来很顺利。根据这个响应，我们可以得出用户有以下参数:`id`、`name`、`age`和`address`。

现在，让我们尝试添加一个新用户:

```java
curl -X POST -d '{"name":"Abdullah", "age":28, "address":"Apartment 2201"}' https://baeldung.service.com/user/
```

因此，**我们收到了 HTTP 状态为 415** 的错误响应:

```java
{
    "timestamp": "yyyy-MM-ddThh:mm:ss.SSS+00:00",
    "status": 415,
    "error": "Unsupported Media Type",
    "path": "/user/"
}
```

在我们弄清楚“为什么会出现这个错误？”，我们需要研究“这个错误是什么？”。

## 4.状态代码 415:不支持的媒体类型

根据规范 [RFC 7231 标题 HTTP/1.1 语义和内容第 6.5.13 节](https://web.archive.org/web/20221211183521/https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.13):

> 415(不支持的媒体类型)状态代码表示源服务器拒绝为请求提供服务，因为目标资源上的此方法不支持有效负载的格式。

正如规范所建议的，API 不支持我们选择的媒体类型。选择 JSON 作为媒体类型的原因是因为 GET 请求的响应。响应数据格式是 JSON。因此，**我们假设 POST 请求也会接受 JSON** 。然而，这一假设被证明是错误的。

为了找到 API 支持的格式，我们决定深入研究服务器端后端代码，我们找到了 API 定义:

```java
@PostMapping(value = "/", consumes = {"application/xml"})
void AddUser(@RequestBody User user)
```

这非常清楚地解释了 API 将只支持 XML 格式。这里有人可能会问:Spring 中的这个“`consumes`”元素的目的是什么？

根据 Spring 框架[文档](https://web.archive.org/web/20221211183521/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html#consumes--),`consumes`元素的用途是:

> 根据映射的处理程序可以使用的媒体类型缩小主映射。由一种或多种媒体类型组成，其中一种类型必须与请求内容类型头匹配

## 5.解决

我们面前有两个解决问题的选择。第一种选择是根据服务器的期望改变请求有效负载的格式。第二个选项是更新 API 请求，以便它开始支持 JSON 格式。

### 5.1.将请求的有效负载更改为 XML

第一种选择是**以 XML 格式发送请求，而不是 JSON** :

```java
curl -X POST -d '<user><name>Abdullah</name><age>28</age><address>Apartment 2201</address></user>' https://baeldung.service.com/user/
```

不幸的是，由于上述请求，我们得到了同样的错误。如果我们记得，我们曾经问过[问题](#1-consumes-method-in-spring)，API 中的那个`consumes`元素的用途是什么。这为我们指出了方向，即**我们的一个头标(`Content-Type`)丢失了**。让我们发送请求，这一次带有丢失的头:

```java
curl -X POST -H "Content-Type: application/xml" -d '<user><name>Abdullah</name><age>28</age><address>Apartment 2201</address></user>' https://baeldung.service.com/user/
```

这一次，我们收到了成功的回应。**然而，我们可能会遇到客户端应用程序无法以支持的格式**发送请求的情况。在这种情况下，我们必须在服务器上进行更改，以使事情变得相对灵活。

### 5.2.更新服务器上的 API

假设我们的客户决定允许我们更改后端服务。上面提到的第二个选项是更新 API 请求，开始接受 JSON 格式。关于如何更新 API 请求，还有三个选项。让我们逐一探索。

**第一个也是最业余的选择是在 API 上用 JSON 替换 XML 格式**:

```java
@PostMapping(value = "/", consumes = {"application/json"}) 
void AddUser(@RequestBody User user)
```

让我们从客户端应用程序以 JSON 格式再次发送请求:

```java
curl -X POST -H "Content-Type: application/json" -d '{"name":"Abdullah", "age":28, "address":"Apartment 2201"} https://baeldung.service.com/user/'
```

响应将是成功的。然而，我们将面临这样一种情况，所有以 XML 格式发送请求的现有客户端现在将开始收到 415 个不支持的媒体类型错误。

**第二个更简单的选择是允许请求有效载荷中的每种格式**:

```java
@PostMapping(value = "/", consumes = {"*/*"}) 
void AddUser(@RequestBody User user
```

根据 JSON 格式的请求，响应将是成功的。然而，这里的问题是**太灵活了**。我们不希望允许各种各样的格式。**这将导致整个代码库(客户端和服务器端)的行为不一致。**

第三个也是推荐的选项是专门添加客户端应用程序当前使用的那些格式。由于 API 已经支持 XML 格式，我们将保留它，因为已经有客户端应用程序向 API 发送 XML。为了使 API 也支持我们的应用程序格式，我们将做一个简单的 API 代码更改:

```java
@PostMapping(value = "/", consumes = {"application/xml","application/json"}) 
void AddUser(@RequestBody User user
```

以 JSON 格式发送我们的请求后，响应将会成功。在这种特殊情况下，这是推荐的方法。

## 6.结论

在本文中，我们了解到“`Content-Type`”头必须从客户端应用程序请求中发送，以避免 415 不支持的媒体类型错误。此外，RFC 清楚地解释了客户端应用程序和服务器端应用程序的“`Content-Type`”报头必须同步，以避免在发送 POST 请求时出现这种错误。

这篇文章的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221211183521/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http-2)