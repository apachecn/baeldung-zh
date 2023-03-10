# 在 Spring REST APIs 中使用 JSON 补丁

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-json-patch>

## 1.介绍

在各种可用的 HTTP 方法中，HTTP PATCH 方法扮演着独特的角色。它允许我们对 HTTP 资源应用部分更新。

在本教程中，我们将了解如何使用 HTTP 补丁方法和 JSON 补丁文档格式对 RESTful 资源进行部分更新。

## 2.使用案例

让我们首先考虑一个由 JSON 文档表示的示例 HTTP `Customer`资源:

```java
{ 
    "id":"1",
    "telephone":"001-555-1234",
    "favorites":["Milk","Eggs"],
    "communicationPreferences": {"post":true, "email":true}
}
```

让我们假设这位顾客的电话号码已经改变，并且该顾客在他们最喜欢的产品列表中添加了一个新项目。这意味着我们只需要更新`Customer`的`telephone`和`favorites`字段。

我们该怎么做？

首先想到的是流行的 HTTP PUT 方法。然而，因为 PUT 完全替换了资源，所以它不是一种恰当地应用部分更新的合适方法。此外，客户机必须在应用和保存更新之前执行 GET。

这就是 [HTTP PATCH](https://web.archive.org/web/20220629003311/https://tools.ietf.org/html/rfc5789#page-8) 方法派上用场的地方。

让我们了解 HTTP 补丁方法和 JSON 补丁格式。

## 3.HTTP 补丁方法和 JSON 补丁格式

HTTP PATCH 方法提供了一种对资源应用部分更新的好方法。因此，客户端只需要发送它们请求中的差异。

让我们看一个 HTTP 补丁请求的简单例子:

```java
PATCH /customers/1234 HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

[description of changes]
```

**HTTP 补丁请求体描述了应该如何修改目标资源以产生新版本。**此外，用于表示`[description of changes]`的格式因资源类型而异。**对于 JSON 资源类型，用来描述变化的格式是 [JSON 补丁](https://web.archive.org/web/20220629003311/https://tools.ietf.org/html/rfc6902)。**

简单地说，JSON 补丁格式使用“一系列操作”来描述应该如何修改目标资源。JSON 补丁文档是 JSON 对象的数组。数组中的每个对象恰好代表一个 JSON 补丁操作。

现在让我们看看 JSON 补丁操作和一些例子。

## 4.JSON 补丁操作

JSON 补丁操作由单个`op`对象表示。

例如，这里我们定义了一个 JSON 补丁操作来更新客户的电话号码:

```java
{
    "op":"replace",
    "path":"/telephone",
    "value":"001-555-5678"
}
```

每个操作必须有一个`path`成员。同样，一些操作对象也必须包含一个`from`成员。**`path`和`from`成员的值是一个 [JSON 指针](https://web.archive.org/web/20220629003311/https://tools.ietf.org/html/rfc6901)。它指的是目标文档中的一个位置。**这个位置可以指向目标对象中的特定键或者数组元素。

现在让我们简要地看一下可用的 JSON 补丁操作。

### 4.1。`add`行动

我们使用`add`操作向对象添加一个新成员。此外，我们可以用它来更新一个现有的成员，并在数组的指定索引处插入一个新值。

例如，让我们将“面包”添加到客户的`favorites` 列表中，索引为 0:

```java
{
    "op":"add",
    "path":"/favorites/0",
    "value":"Bread"
}
```

在`add`操作之后，修改后的客户详细信息将是:

```java
{
    "id":"1",
    "telephone":"001-555-1234",
    "favorites":["Bread","Milk","Eggs"],
    "communicationPreferences": {"post":true, "email":true}
}
```

### 4.2。`remove`行动

`remove`操作删除目标位置的值。此外，它可以从数组中移除指定索引处的元素。

例如，让我们为我们的客户删除`communcationPreferences`:

```java
{
    "op":"remove",
    "path":"/communicationPreferences"
}
```

在`remove`操作之后，修改后的客户详细信息将是:

```java
{
    "id":"1",
    "telephone":"001-555-1234",
    "favorites":["Bread","Milk","Eggs"],
    "communicationPreferences":null
}
```

### 4.3。`replace`行动

`replace`操作用新值更新目标位置的值。

例如，让我们更新客户的电话号码:

```java
{
    "op":"replace",
    "path":"/telephone",
    "value":"001-555-5678"
}
```

在`replace`操作之后，修改后的客户详细信息将是:

```java
{ 
    "id":"1", 
    "telephone":"001-555-5678", 
    "favorites":["Bread","Milk","Eggs"], 
    "communicationPreferences":null
}
```

### 4.4。`move`行动

`move`操作删除指定位置的值，并将其添加到目标位置。

例如，让我们将“面包”从客户的`favorites`列表顶部移到列表底部:

```java
{
    "op":"move",
    "from":"/favorites/0",
    "path":"/favorites/-"
}
```

在`move`操作之后，修改后的客户详细信息将是:

```java
{ 
    "id":"1", 
    "telephone":"001-555-5678", 
    "favorites":["Milk","Eggs","Bread"], 
    "communicationPreferences":null
} 
```

上例中的`/favorites/0`和`/favorites/-`是指向`favorites`数组的开始和结束索引的 JSON 指针。

### 4.5。`copy`行动

`copy`操作将指定位置的值复制到目标位置。

例如，让我们在`favorites`列表中复制“牛奶”:

```java
{
    "op":"copy",
    "from":"/favorites/0",
    "path":"/favorites/-"
}
```

在`copy`操作之后，修改后的客户详细信息将是:

```java
{ 
    "id":"1", 
    "telephone":"001-555-5678", 
    "favorites":["Milk","Eggs","Bread","Milk"], 
    "communicationPreferences":null
}
```

### 4.6。`test`行动

`test`操作测试“路径”上的值是否等于“值”。因为修补操作是原子性的，所以如果它的任何操作失败，就应该丢弃该修补程序。`test`操作可用于验证先决条件和后置条件是否得到满足。

例如，让我们测试对客户的`telephone`字段的更新是否成功:

```java
{
    "op":"test", 
    "path":"/telephone",
    "value":"001-555-5678"
} 
```

现在让我们看看如何将上述概念应用到我们的示例中。

## 5.使用 JSON 补丁格式的 HTTP 补丁请求

我们将重温我们的`Customer`用例。

下面是使用 JSON 补丁格式对客户的`telephone`和`favorites` 列表执行部分更新的 HTTP 补丁请求:

```java
curl -i -X PATCH http://localhost:8080/customers/1 -H "Content-Type: application/json-patch+json" -d '[
    {"op":"replace","path":"/telephone","value":"+1-555-56"},
    {"op":"add","path":"/favorites/0","value":"Bread"}
]' 
```

**最重要的是，JSON 补丁请求的`Content-Type`是`application/json-patch+json`。同样，请求体是一个 JSON 补丁操作对象的数组:**

```java
[
    {"op":"replace","path":"/telephone","value":"+1-555-56"},
    {"op":"add","path":"/favorites/0","value":"Bread"}
]
```

我们如何在服务器端处理这样的请求？

一种方法是编写一个定制框架，该框架按顺序评估操作，并将它们作为一个原子单元应用于目标资源。显然，这种方法听起来很复杂。此外，它还会导致使用补丁文档的非标准化方式。

幸运的是，我们不必手工处理 JSON 补丁请求。

最初在 [JSR 353](https://web.archive.org/web/20220629003311/https://jcp.org/en/jsr/detail?id=353) 中定义的用于 JSON 处理的 Java API 1.0，或 JSON-P 1.0，在 [JSR 374](https://web.archive.org/web/20220629003311/https://www.jcp.org/en/jsr/detail?id=374) 中引入了对 JSON 补丁的支持。JSON-P API 提供了`JsonPatch`类型来表示 JSON 补丁实现。

但是，JSON-P 只是一个 API。为了使用 JSON-P API，我们需要使用一个实现它的库。对于本文中的例子，我们将使用一个名为 [json-patch](https://web.archive.org/web/20220629003311/https://github.com/java-json-tools/json-patch) 的库。

现在让我们看看如何使用上述 JSON 补丁格式构建一个使用 HTTP 补丁请求的 REST 服务。

## 6.在 Spring Boot 应用程序中实现 JSON 补丁

### 6.1.属国

json 补丁的最新版本可以在 Maven 中央存储库中找到。

首先，让我们将依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>com.github.java-json-tools</groupId>
    <artifactId>json-patch</artifactId>
    <version>1.12</version>
</dependency>
```

现在，让我们定义一个模式类来表示`Customer` JSON 文档:

```java
public class Customer {
    private String id;
    private String telephone;
    private List<String> favorites;
    private Map<String, Boolean> communicationPreferences;

    // standard getters and setters
}
```

接下来，我们将看看我们的控制器方法。

### 6.2.静止控制器方法

然后，我们可以为我们的客户用例实现 HTTP 补丁:

```java
@PatchMapping(path = "/{id}", consumes = "application/json-patch+json")
public ResponseEntity<Customer> updateCustomer(@PathVariable String id, @RequestBody JsonPatch patch) {
    try {
        Customer customer = customerService.findCustomer(id).orElseThrow(CustomerNotFoundException::new);
        Customer customerPatched = applyPatchToCustomer(patch, customer);
        customerService.updateCustomer(customerPatched);
        return ResponseEntity.ok(customerPatched);
    } catch (JsonPatchException | JsonProcessingException e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
    } catch (CustomerNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).build();
    }
} 
```

现在让我们来理解这种方法是怎么回事:

*   首先，我们使用`@PatchMapping` 注释将该方法标记为补丁处理程序方法
*   当带有`application/json-patch+json`“Content-Type”的补丁请求到达时，Spring Boot 使用默认的`MappingJackson2HttpMessageConverter` 将请求有效负载转换成一个`JsonPatch`实例。因此，我们的控制器方法将接收请求体作为一个`JsonPatch`实例

在该方法中:

1.  首先，我们调用`customerService.findCustomer(id)`方法来查找客户记录
2.  随后，如果找到了客户记录，我们就调用`applyPatchToCustomer(patch, customer)`方法。这将`JsonPatch`应用于客户(稍后将详细介绍)
3.  然后我们调用`customerService.updateCustomer(customerPatched)`来保存客户记录
4.  最后，我们向客户端返回一个`200 OK`响应，响应中包含修补过的`Customer`细节

最重要的是，真正的魔法发生在`applyPatchToCustomer(patch, customer)`方法中:

```java
private Customer applyPatchToCustomer(
  JsonPatch patch, Customer targetCustomer) throws JsonPatchException, JsonProcessingException {
    JsonNode patched = patch.apply(objectMapper.convertValue(targetCustomer, JsonNode.class));
    return objectMapper.treeToValue(patched, Customer.class);
} 
```

1.  首先，我们有一个`JsonPatch`实例，它保存了应用于目标`Customer`的操作列表
2.  然后我们将目标`Customer`转换成`com.fasterxml.jackson.databind.JsonNode`的实例，并将其传递给`JsonPatch.apply`方法来应用补丁。在幕后，`JsonPatch.apply`处理将操作应用到目标。补丁的结果也是一个`com.fasterxml.jackson.databind.JsonNode`实例
3.  然后我们调用`objectMapper.treeToValue`方法，将修补后的`com.fasterxml.jackson.databind.JsonNode`中的数据绑定到`Customer`类型。这是我们打了补丁的`Customer`实例
4.  最后，我们返回打了补丁的`Customer`实例

现在让我们针对我们的 API 运行一些测试。

### 6.3.测试

首先，让我们使用对 API 的 POST 请求来创建一个客户:

```java
curl -i -X POST http://localhost:8080/customers -H "Content-Type: application/json" 
  -d '{"telephone":"+1-555-12","favorites":["Milk","Eggs"],"communicationPreferences":{"post":true,"email":true}}' 
```

我们收到一个`201 Created` 响应:

```java
HTTP/1.1 201
Location: http://localhost:8080/customers/1 
```

`Location`响应头被设置为新资源的位置。表示新`Customer`的`id`为 1。

接下来，让我们使用补丁请求向该客户请求部分更新:

```java
curl -i -X PATCH http://localhost:8080/customers/1 -H "Content-Type: application/json-patch+json" -d '[
    {"op":"replace","path":"/telephone","value":"+1-555-56"}, 
    {"op":"add","path":"/favorites/0","value": "Bread"}
]'
```

我们收到一个`200` `OK`响应，其中包含修补后的客户详细信息:

```java
HTTP/1.1 200
Content-Type: application/json
Transfer-Encoding: chunked
Date: Fri, 14 Feb 2020 21:23:14 GMT

{"id":"1","telephone":"+1-555-56","favorites":["Bread","Milk","Eggs"],"communicationPreferences":{"post":true,"email":true}}
```

## 7.结论

在本文中，我们研究了如何在 Spring REST APIs 中实现 JSON 补丁。

首先，我们研究了 HTTP 补丁方法及其执行部分更新的能力。

然后，我们研究了什么是 JSON 补丁，并理解了各种 JSON 补丁操作。

最后，我们讨论了如何使用 json-patch 库在 Spring Boot 应用程序中处理 HTTP 补丁请求。

与往常一样，本文中使用的示例的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220629003311/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)