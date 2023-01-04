# 如何在 Spring MVC 中设置 JSON 内容类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-set-json-content-type>

## 1.介绍

内容类型指示如何解释请求/响应中的数据。每当控制器接收到 web 请求时，它都会消费或产生一些媒体类型。在这个请求-响应模型中，可以消费/生产几种媒体类型，JSON 就是其中之一。

在这个快速教程中，我们将看看在 Spring MVC 中设置内容类型的不同方法。

## 2.在春天

简单地说， [`@RequestMapping`](/web/20220626120933/https://www.baeldung.com/spring-requestmapping) 是将 web 请求映射到 Spring 控制器的一个重要注释。它有各种属性，包括 HTTP 方法、请求参数、头和媒体类型。

通常，媒体类型分为两类:可消费的和可生产的。除了这两个，我们还可以在 Spring 中定义一个自定义媒体类型[。**主要目的是，将主映射限制到我们的请求处理程序的媒体类型列表。**](/web/20220626120933/https://www.baeldung.com/spring-rest-custom-media-type)

### 2.1.可消费媒体类型

使用`consumes` 属性，我们指定控制器将从客户机接受的媒体类型。我们也可以提供媒体类型列表。让我们定义一个简单的端点:

```
@RequestMapping(value = "/greetings", method = RequestMethod.POST, consumes="application/json")
public void addGreeting(@RequestBody ContentType type, Model model) {
    // code here
}
```

**如果客户端指定了无法被资源使用的媒体类型，系统将生成 HTTP“415 不支持的媒体类型”错误。**

### 2.2.可生产的媒体类型

与`consumes`属性相反，`produces`指定了资源可以生成并发送回客户机的媒体类型。毫无疑问，我们可以使用一个选项列表。**如果一个资源不能产生请求的资源，系统将产生一个 HTTP“406 不可接受”错误。**

让我们从一个简单的例子开始——一个公开 JSON 字符串的 API。

这是我们的终点:

```
@RequestMapping(
  value = "/greetings-with-response-body", 
  method = RequestMethod.GET, 
  produces="application/json"
) 
@ResponseBody
public String getGreetingWhileReturnTypeIsString() { 
    return "{\"test\": \"Hello using @ResponseBody\"}";
}
```

让我们用 CURL 来测试一下:

```
curl http://localhost:8080/greetings-with-response-body
```

上面的命令会产生响应:

```
{ "test": "Hello using @ResponseBody" } 
```

根据头中的内容类型，`@ResponseBody` 只将一个方法返回值绑定到 web 响应体。

## 3.内容类型设置不正确

当方法具有返回类型`String,` 并且类路径中没有 JSON 映射器时。在这种情况下，返回值由`StringHttpMessageConverter` 类处理，该类将内容类型设置为“text/plain”。这通常会导致控制器无法产生预期内容类型的问题。

让我们看看解决这个问题的不同方法。

### 3.1.将@ `ResponseBody` 与 JSON Mapper 一起使用

[Jackson](/web/20220626120933/https://www.baeldung.com/jackson) `[ObjectMapper](/web/20220626120933/https://www.baeldung.com/jackson-object-mapper-tutorial)`类从字符串、流或文件中解析 JSON。如果 Jackson 在类路径上，Spring 应用程序中的任何控制器都会默认呈现 JSON 响应。

为了将[杰克森](https://web.archive.org/web/20220626120933/https://search.maven.org/search?q=g:com.fasterxml.jackson.core%20AND%20a:jackson-databind)包含在类路径中，我们需要在`pom.xml`中添加以下依赖项:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.4</version>
</dependency> 
```

让我们添加一个单元测试来验证响应:

```
@Test
public void givenReturnTypeIsString_whenJacksonOnClasspath_thenDefaultContentTypeIsJSON() 
  throws Exception {

    // Given
    String expectedMimeType = "application/json";

    // Then
    String actualMimeType = this.mockMvc.perform(MockMvcRequestBuilders.get("/greetings-with-response-body", 1))
      .andReturn().getResponse().getContentType();

    Assert.assertEquals(expectedMimeType, actualMimeType);
}
```

### 3.2.使用`ResponseEntiy`

与 [`@ResponseBody`](/web/20220626120933/https://www.baeldung.com/spring-request-response-body) 相反， [`ResponseEntity`](/web/20220626120933/https://www.baeldung.com/spring-response-entity) 是一个通用类型，表示整个 HTTP 响应。因此，我们可以控制进入其中的任何内容:状态代码、标题和主体。

让我们定义一个新的端点:

```
@RequestMapping(
  value = "/greetings-with-response-entity",
  method = RequestMethod.GET, 
  produces = "application/json"
)
public ResponseEntity<String> getGreetingWithResponseEntity() {
    final HttpHeaders httpHeaders= new HttpHeaders();
    httpHeaders.setContentType(MediaType.APPLICATION_JSON);
    return new ResponseEntity<String>("{\"test\": \"Hello with ResponseEntity\"}", httpHeaders, HttpStatus.OK);
}
```

在我们浏览器的开发人员控制台中，我们可以看到以下响应:

```
{"test": "Hello with ResponseEntity"}
```

使用`ResponseEntity`，我们应该在 [dispatcher servlet](/web/20220626120933/https://www.baeldung.com/spring-dispatcherservlet) 中有注释驱动的标签:

```
<mvc:annotation-driven />
```

简单地说，上面的标签对 [Spring MVC](/web/20220626120933/https://www.baeldung.com/spring-mvc) 的内部工作给予了更大的控制。

让我们用一个测试用例来验证响应的内容类型:

```
@Test
public void givenReturnTypeIsResponseEntity_thenDefaultContentTypeIsJSON() throws Exception {

    // Given
    String expectedMimeType = "application/json";

    // Then
    String actualMimeType = this.mockMvc.perform(MockMvcRequestBuilders.get("/greetings-with-response-entity", 1))
      .andReturn().getResponse().getContentType();

    Assert.assertEquals(expectedMimeType, actualMimeType);
} 
```

### 3.3.使用`Map<String, Object>`返回类型

最后，我们还可以通过将返回类型从`String` 改为`Map`来设置内容类型。这个`Map`返回类型将需要封送处理并返回 JSON 对象。

这是我们的新端点:

```
@RequestMapping(
  value = "/greetings-with-map-return-type", 
  method = RequestMethod.GET, 
  produces = "application/json"
)
@ResponseBody
public Map<String, Object> getGreetingWhileReturnTypeIsMap() {
    HashMap<String, Object> map = new HashMap<String, Object>();
    map.put("test", "Hello from map");
    return map;
}
```

让我们来看看实际情况:

```
curl http://localhost:8080/greetings-with-map-return-type
```

curl 命令返回一个 JSON 响应:

```
{ "test": "Hello from map" }
```

## 4.结论

本文解释了如何在 Spring MVC 中设置内容类型，首先在类路径中添加 Json mapper，然后使用 ResponseEntity，最后将返回类型从 String 改为 Map。

和往常一样，所有的代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20220626120933/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-4)