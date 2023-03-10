# Spring 的 RequestBody 和 ResponseBody 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-request-response-body>

## 1。简介

在这个快速教程中，我们提供了 Spring `@RequestBody`和`@ResponseBody`注释的简要概述。

## 延伸阅读:

## [弹簧处理器映射指南](/web/20220727020632/https://www.baeldung.com/spring-handler-mappings)

The article explains how HandlerMapping implementation resolve URL to a particular Handler.[Read more](/web/20220727020632/https://www.baeldung.com/spring-handler-mappings) →

## [弹簧控制器快速指南](/web/20220727020632/https://www.baeldung.com/spring-controllers)

A quick and practical guide to Spring Controllers  - both for typical MVC apps and for REST APIs.[Read more](/web/20220727020632/https://www.baeldung.com/spring-controllers) →

## [Spring @ Controller 和@RestController 注释](/web/20220727020632/https://www.baeldung.com/spring-controller-vs-restcontroller)

Learn about the differences between @Controller and @RestController annotations in Spring MVC.[Read more](/web/20220727020632/https://www.baeldung.com/spring-controller-vs-restcontroller) →

## 2。`@RequestBody`

简而言之， **`@RequestBody`注释将`HttpRequest`主体映射到一个传输或域对象，使入站`HttpRequest`主体的自动反序列化**成为一个 Java 对象。

首先，让我们来看看弹簧控制器的方法:

```java
@PostMapping("/request")
public ResponseEntity postController(
  @RequestBody LoginForm loginForm) {

    exampleService.fakeAuthenticate(loginForm);
    return ResponseEntity.ok(HttpStatus.OK);
}
```

Spring 自动将 JSON 反序列化为 Java 类型，假设指定了一个合适的类型。

默认情况下，**我们用`@RequestBody`注释的类型必须对应于从我们的客户端控制器发送的 JSON:**

```java
public class LoginForm {
    private String username;
    private String password;
    // ...
}
```

这里，我们用来表示`HttpRequest`身体的对象映射到我们的`LoginForm`对象。

让我们用 CURL 来测试一下:

```java
curl -i \
-H "Accept: application/json" \
-H "Content-Type:application/json" \
-X POST --data 
  '{"username": "johnny", "password": "password"}' "https://localhost:8080/.../request"
```

对于 Spring REST API 和使用@ `RequestBody`注释的 Angular 客户机来说，这就是我们所需要的。

## 3。`@ResponseBody`

`@ResponseBody`注释告诉控制器，返回的对象被自动序列化为 JSON，并传递回`HttpResponse`对象。

假设我们有一个定制的`Response`对象:

```java
public class ResponseTransfer {
    private String text; 

    // standard getters/setters
}
```

接下来，可以实现相关联的控制器:

```java
@Controller
@RequestMapping("/post")
public class ExamplePostController {

    @Autowired
    ExampleService exampleService;

    @PostMapping("/response")
    @ResponseBody
    public ResponseTransfer postResponseController(
      @RequestBody LoginForm loginForm) {
        return new ResponseTransfer("Thanks For Posting!!!");
     }
}
```

在我们浏览器的开发人员控制台中或使用类似 Postman 的工具，我们可以看到以下响应:

```java
{"text":"Thanks For Posting!!!"}
```

**记住，我们不需要用`@ResponseBody`注释**来注释 `@RestController-`注释过的控制器，因为 Spring 默认会这样做。

### 3.1.设置内容类型

当我们使用`@ResponseBody`注释时，我们仍然能够显式地设置方法返回的内容类型。

为此，**我们可以使用`@RequestMapping`的`produces`属性。**注意像`@PostMapping`、`@GetMapping`等标注。为该参数定义别名。

现在让我们添加一个发送 JSON 响应的新端点:

```java
@PostMapping(value = "/content", produces = MediaType.APPLICATION_JSON_VALUE)
@ResponseBody
public ResponseTransfer postResponseJsonContent(
  @RequestBody LoginForm loginForm) {
    return new ResponseTransfer("JSON Content!");
}
```

在这个例子中，我们使用了`MediaType.APPLICATION_JSON_VALUE`常量。或者，我们可以直接使用`application/json`。

接下来，让我们实现一个新方法，映射到同一个`/content`路径，但是返回 XML 内容:

```java
@PostMapping(value = "/content", produces = MediaType.APPLICATION_XML_VALUE)
@ResponseBody
public ResponseTransfer postResponseXmlContent(
  @RequestBody LoginForm loginForm) {
    return new ResponseTransfer("XML Content!");
}
```

现在，**根据请求头中发送的`Accept`参数的值，我们将得到不同的响应。**

让我们来看看实际情况:

```java
curl -i \ 
-H "Accept: application/json" \ 
-H "Content-Type:application/json" \ 
-X POST --data 
  '{"username": "johnny", "password": "password"}' "https://localhost:8080/.../content"
```

CURL 命令返回一个 JSON 响应:

```java
HTTP/1.1 200
Content-Type: application/json
Transfer-Encoding: chunked
Date: Thu, 20 Feb 2020 19:43:06 GMT

{"text":"JSON Content!"}
```

现在，让我们更改`Accept`参数:

```java
curl -i \
-H "Accept: application/xml" \
-H "Content-Type:application/json" \
-X POST --data
  '{"username": "johnny", "password": "password"}' "https://localhost:8080/.../content"
```

不出所料，我们这次得到了一个 XML 内容:

```java
HTTP/1.1 200
Content-Type: application/xml
Transfer-Encoding: chunked
Date: Thu, 20 Feb 2020 19:43:19 GMT

<ResponseTransfer><text>XML Content!</text></ResponseTransfer>
```

## 4。结论

我们为 Spring 应用程序构建了一个简单的 Angular 客户端，演示了如何使用 `@RequestBody`和`@ResponseBody`注释。

此外，我们展示了如何在使用`@ResponseBody`时设置内容类型。

和往常一样，代码样本可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest)