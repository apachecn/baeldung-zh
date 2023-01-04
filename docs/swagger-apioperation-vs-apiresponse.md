# 招摇中的@ApiOperation vs @ApiResponse

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/swagger-apioperation-vs-apiresponse>

## 1.概观

在本教程中，我们将讨论 Swagger 的`@ApiOperation`和`@ApiResponse`注释之间的主要区别。

## 2.带有 Swagger 的描述性文档

当我们创建 REST API 时，创建其适当的规范也很重要。此外，这样的规范应该是可读的、可理解的，并提供所有必要的信息。

此外，文档应该有对 API 所做的每一个变更的描述。手动创建 REST API 文档会令人疲惫不堪，更重要的是，非常耗时。幸运的是，像 Swagger 这样的工具可以帮助我们完成这个过程。

**[Swagger](https://web.archive.org/web/20221010200543/https://swagger.io/tools/swagger-ui) 代表一组围绕 [OpenAPI 规范](/web/20221010200543/https://www.baeldung.com/spring-rest-openapi-documentation)构建的开源工具。它可以帮助我们设计、构建、记录和使用 REST APIs。**

Swagger 规范是记录 REST APIs 的标准。使用 Swagger 规范，我们可以描述整个 API，比如公开的端点、操作、参数、认证方法等等。

Swagger 提供了各种注释，可以帮助我们记录 REST API。此外，**为我们的 REST API** 提供了`@ApiOperation`和`@ApiResponse`注释来记录响应。在本教程的剩余部分，我们将使用下面的控制器类，看看如何使用这些注释:

```java
@RestController
@RequestMapping("/customers")
class CustomerController {

   private final CustomerService customerService;

   public CustomerController(CustomerService customerService) {
       this.customerService = customerService;
   }

   @GetMapping("/{id}")
   public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
       return ResponseEntity.ok(customerService.getById(id));
   }
}
```

## 3.`@ApiOperation`

**`@ApiOperation`注释用于描述单个操作。**操作是路径和 HTTP 方法的唯一组合。

另外，使用`@ApiOperation`，我们可以描述一个成功的 REST API 调用的结果。换句话说，我们可以使用这个注释来指定一般的返回类型。

让我们给我们的方法添加注释:

```java
@ApiOperation(value = "Gets customer by ID", 
        response = CustomerResponse.class, 
        notes = "Customer must exist")
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

接下来，我们将浏览一下`@ApiOperation`中一些最常用的属性。

### 3.1.`value`属性

必需的`value`属性包含操作的摘要字段。简单地说，它提供了操作的简短描述。然而，我们应该保持这个参数少于 120 个字符。

下面是我们如何在`@ApiOperation`注释中定义 value 属性:

```java
@ApiOperation(value = "Gets customer by ID")
```

### 3.2.`notes`属性

使用`notes`，我们可以提供关于操作的更多细节。例如，我们可以放置描述端点限制的文本:

```java
@ApiOperation(value = "Gets customer by ID", notes = "Customer must exist")
```

### 3.3.`response`属性

`response`属性包含操作的响应类型。此外，设置此属性将重写任何自动派生的数据类型。在`@ApiOperation`注释中定义的响应属性应该包含通用响应类型。

让我们创建一个类来表示我们的方法返回的成功响应:

```java
class CustomerResponse {

   private Long id;
   private String firstName;
   private String lastName;

   // getters and setters
}
```

接下来，让我们将`response`属性添加到注释中:

```java
@ApiOperation(value = "Gets customer by ID",
        response = CustomerResponse.class,
        notes = "Customer must exist")
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

### 3.4.`code`属性

code 属性表示响应代码的 HTTP 状态。 [HTTP 状态码](https://web.archive.org/web/20221010200543/https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)有几种定义。应该用其中一个。如果我们不提供，默认值将是 200。

## 4.`@ApiResponse`

使用 HTTP 状态代码返回错误是一种常见的做法。我们可以使用`@ApiResponse`注释来描述操作的具体可能响应。

**`@ApiOperation`注释描述了一个操作和一般的返回类型，而`@ApiResponse`注释描述了其余可能的返回代码。**

此外，注释可以应用于方法级和类级。此外，只有在方法级还没有定义具有相同代码的`@ApiResponse`注释时，才会解析放在类级的注释。换句话说，方法注释优先于类注释。

**我们应该使用 [`@ApiResponses`](/web/20221010200543/https://www.baeldung.com/java-swagger-set-list-response) 注释中的`@ApiResponse`注释，不管我们有一个还是多个响应。**如果我们直接使用这个注释，它不会被 Swagger 解析。

让我们在方法上定义`@ApiResponses`和`@ApiResponse`注释:

```java
@ApiResponses(value = {
        @ApiResponse(code = 400, message = "Invalid ID supplied"),
        @ApiResponse(code = 404, message = "Customer not found")})
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

我们也可以使用注释来指定成功响应:

```java
@ApiOperation(value = "Gets customer by ID", notes = "Customer must exist")
@ApiResponses(value = {
        @ApiResponse(code = 200, message = "OK", response = CustomerResponse.class),
        @ApiResponse(code = 400, message = "Invalid ID supplied"),
        @ApiResponse(code = 404, message = "Customer not found"),
        @ApiResponse(code = 500, message = "Internal server error", response = ErrorResponse.class)})
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

如果我们使用`@ApiResponse`注释指定成功的响应，就不需要在`@ApiOperation`注释中定义它。

现在，让我们浏览一下在`@ApiResponse`中使用的一些属性。

### 4.1.`code`和`message`属性

`code`和`message`属性都是`@ApiResponse`注释中的必需参数。

与`@ApiOperation`注释中的 code 属性一样，它应该包含响应的 HTTP 状态代码。值得一提的是，我们不能用相同的代码属性定义多个`@ApiResponse`。

消息属性通常包含伴随响应的可读消息:

```java
@ApiResponse(code = 400, message = "Invalid ID supplied")
```

### 4.2.`response`属性

有时，一个端点使用不同的响应类型。例如，我们可以有一种类型的成功响应和另一种类型的错误响应。我们可以通过将响应类与响应代码相关联，使用可选的`response`属性来描述它们。

首先，让我们定义一个在内部服务器出错时返回的类:

```java
class ErrorResponse {

    private String error;
    private String message;

    // getters and setters
}
```

其次，让我们为内部服务器错误添加一个新的`@ApiResponse`:

```java
@ApiResponses(value = {
        @ApiResponse(code = 400, message = "Invalid ID supplied"),
        @ApiResponse(code = 404, message = "Customer not found"),
        @ApiResponse(code = 500, message = "Internal server error", response = ErrorResponse.class)})
@GetMapping("/{id}")
public ResponseEntity<CustomerResponse> getCustomer(@PathVariable("id") Long id) {
    return ResponseEntity.ok(customerService.getById(id));
}
```

## 5.`@ApiOperation`和`@ApiResponse`的区别

总而言之，下表显示了`@ApiOperation`和`@ApiResponse`注释之间的主要区别:

| @ApiOperation | @ApiResponse |
| 用于描述操作 | 用于描述操作的可能响应 |
| 用于成功响应 | 用于成功和错误响应 |
| 只能在方法级别定义 | 可以在方法或类级别定义 |
| 可以直接使用 | 只能在`@ApiResponses`注释中使用 |
| 默认代码属性值为 200 | 没有默认的代码属性值 |

## 6.结论

在本文中，我们学习了`@ApiOperation`和`@ApiResponse`注释之间的区别。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221010200543/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-swagger)