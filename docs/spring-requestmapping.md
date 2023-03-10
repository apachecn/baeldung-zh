# Spring 请求映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-requestmapping>

## 1。概述

在本教程中，我们将关注 **Spring MVC 中的一个主要注释:`@RequestMapping.`**

简而言之，注释用于将 web 请求映射到 Spring 控制器方法。

## 延伸阅读:

## [用 Spring 服务静态资源](/web/20221130175138/https://www.baeldung.com/spring-mvc-static-resources)

How to map and handle static resources with Spring MVC - use the simple configuration, then the 3.1 more flexible one and finally the new 4.1 resource resolvers.[Read more](/web/20221130175138/https://www.baeldung.com/spring-mvc-static-resources) →

## [开始使用 Spring MVC 中的表单](/web/20221130175138/https://www.baeldung.com/spring-mvc-form-tutorial)

Learn how to work with forms using Spring MVC - mapping a basic entity, submit, displaying errors.[Read more](/web/20221130175138/https://www.baeldung.com/spring-mvc-form-tutorial) →

## [使用 Spring 框架的 Http 消息转换器](/web/20221130175138/https://www.baeldung.com/spring-httpmessageconverter-rest)

How to configure HttpMessageConverters for a REST API with Spring, and how to use these converters with the RestTemplate.[Read more](/web/20221130175138/https://www.baeldung.com/spring-httpmessageconverter-rest) →

## 2。@ `RequestMapping`基础知识

让我们从一个简单的例子开始:使用一些基本标准将一个 HTTP 请求映射到一个方法。

### 2.1。`@RequestMapping` —途经

```java
@RequestMapping(value = "/ex/foos", method = RequestMethod.GET)
@ResponseBody
public String getFoosBySimplePath() {
    return "Get some Foos";
}
```

要用一个简单的`curl`命令测试这个映射，运行:

```java
curl -i http://localhost:8080/spring-rest/ex/foos
```

### 2.2。`@RequestMapping`—HTTP 方法

HTTP `method`参数没有默认值**。**所以，如果我们不指定值，它将映射到任何 HTTP 请求。

这里有一个简单的例子，类似于上一个例子，但是这次映射到一个 HTTP POST 请求:

```java
@RequestMapping(value = "/ex/foos", method = POST)
@ResponseBody
public String postFoos() {
    return "Post some Foos";
}
```

通过`curl`命令测试开机自检:

```java
curl -i -X POST http://localhost:8080/spring-rest/ex/foos
```

## 3。`RequestMapping`和 HTTP 头

### 3.1。`@RequestMapping`带有`headers`属性

通过为请求指定一个标头，可以进一步缩小映射范围:

```java
@RequestMapping(value = "/ex/foos", headers = "key=val", method = GET)
@ResponseBody
public String getFoosWithHeader() {
    return "Get some Foos with Header";
}
```

为了测试操作，我们将使用`curl`标题支持:

```java
curl -i -H "key:val" http://localhost:8080/spring-rest/ex/foos
```

甚至通过`@RequestMapping`的`headers`属性实现多个头:

```java
@RequestMapping(
  value = "/ex/foos", 
  headers = { "key1=val1", "key2=val2" }, method = GET)
@ResponseBody
public String getFoosWithHeaders() {
    return "Get some Foos with Header";
}
```

我们可以使用以下命令对此进行测试:

```java
curl -i -H "key1:val1" -H "key2:val2" http://localhost:8080/spring-rest/ex/foos
```

注意，对于`curl`语法，冒号分隔头键和头值，与 HTTP 规范中的一样，而在 Spring 中，使用等号。

### 3.2。`@RequestMapping`消费和生产

由控制器方法产生的映射**媒体类型值得特别注意。**

我们可以通过上面介绍的`@RequestMapping` `headers`属性，基于请求的`Accept`头来映射请求:

```java
@RequestMapping(
  value = "/ex/foos", 
  method = GET, 
  headers = "Accept=application/json")
@ResponseBody
public String getFoosAsJsonFromBrowser() {
    return "Get some Foos with Header Old";
}
```

这种定义`Accept`头的方式的匹配是灵活的——它使用 contains 而不是 equals，所以像下面这样的请求仍然可以正确映射:

```java
curl -H "Accept:application/json,text/html" 
  http://localhost:8080/spring-rest/ex/foos
```

从 Spring 3.1 开始， **`@RequestMapping`注释现在有了`produces`和`consumes`属性**，专门用于这个目的:

```java
@RequestMapping(
  value = "/ex/foos", 
  method = RequestMethod.GET, 
  produces = "application/json"
)
@ResponseBody
public String getFoosAsJsonFromREST() {
    return "Get some Foos with Header New";
}
```

此外，从 Spring 3.1 开始，带有`headers`属性的旧类型映射将自动转换为新的`produces`机制，因此结果将是相同的。

这通过`curl`以同样的方式消耗:

```java
curl -H "Accept:application/json" 
  http://localhost:8080/spring-rest/ex/foos
```

此外，`produces`还支持多个值:

```java
@RequestMapping(
  value = "/ex/foos", 
  method = GET,
  produces = { "application/json", "application/xml" }
)
```

请记住，这些——指定`Accept`头的新旧方式——基本上是相同的映射，所以 Spring 不允许将它们放在一起。

同时激活这两种方法会导致:

```java
Caused by: java.lang.IllegalStateException: Ambiguous mapping found. 
Cannot map 'fooController' bean method 
java.lang.String 
org.baeldung.spring.web.controller
  .FooController.getFoosAsJsonFromREST()
to 
{ [/ex/foos],
  methods=[GET],params=[],headers=[],
  consumes=[],produces=[application/json],custom=[]
}: 
There is already 'fooController' bean method
java.lang.String 
org.baeldung.spring.web.controller
  .FooController.getFoosAsJsonFromBrowser() 
mapped.
```

关于新的`produces`和`consumes`机制的最后一点说明，它们的行为与大多数其他注释不同:当在类型级别指定时，**方法级别的注释不补充而是覆盖**类型级别的信息。

当然，如果你想更深入地用 Spring 构建一个 REST API，[请查看**新的 REST with Spring 课程**](/web/20221130175138/https://www.baeldung.com/rest-with-spring-course) 。

## 4。`RequestMapping`用路径变量

部分映射 URI 可以通过`@PathVariable`注释绑定到变量。

### 4.1。单身`@PathVariable`

一个带有单个路径变量的简单示例:

```java
@RequestMapping(value = "/ex/foos/{id}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariable(
  @PathVariable("id") long id) {
    return "Get a specific Foo with id=" + id;
}
```

这可以用`curl`来测试:

```java
curl http://localhost:8080/spring-rest/ex/foos/1
```

如果方法参数的名称与路径变量的名称完全匹配，那么这可以通过使用不带值的`@PathVariable`**来简化:**

```java
@RequestMapping(value = "/ex/foos/{id}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariable(
  @PathVariable String id) {
    return "Get a specific Foo with id=" + id;
}
```

注意，`@PathVariable`受益于自动类型转换，所以我们也可以将`id`声明为:

```java
@PathVariable long id
```

### 4.2。多个`@PathVariable`

更复杂的 URI 可能需要将 URI 的多个部分映射到多个值:

```java
@RequestMapping(value = "/ex/foos/{fooid}/bar/{barid}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariables
  (@PathVariable long fooid, @PathVariable long barid) {
    return "Get a specific Bar with id=" + barid + 
      " from a Foo with id=" + fooid;
}
```

这很容易用`curl`以同样的方式测试:

```java
curl http://localhost:8080/spring-rest/ex/foos/1/bar/2
```

### 4.3。`@PathVariable`同 Regex

映射`@PathVariable.`时也可以使用正则表达式

例如，我们将限制映射只接受`id`的数值:

```java
@RequestMapping(value = "/ex/bars/{numericId:[\\d]+}", method = GET)
@ResponseBody
public String getBarsBySimplePathWithPathVariable(
  @PathVariable long numericId) {
    return "Get a specific Bar with id=" + numericId;
}
```

这将意味着以下 URIs 将匹配:

```java
http://localhost:8080/spring-rest/ex/bars/1
```

但这不会:

```java
http://localhost:8080/spring-rest/ex/bars/abc
```

## 5。`RequestMapping`带请求参数

@RequestMapping 允许使用`@RequestParam`注释简单地**映射 URL 参数。******

我们现在将一个请求映射到一个 URI:

```java
http://localhost:8080/spring-rest/ex/bars?id=100
```

```java
@RequestMapping(value = "/ex/bars", method = GET)
@ResponseBody
public String getBarBySimplePathWithRequestParam(
  @RequestParam("id") long id) {
    return "Get a specific Bar with id=" + id;
}
```

然后，我们使用控制器方法签名中的`@RequestParam(“id”)`注释提取`id`参数的值。

要发送带有`id`参数的请求，我们将使用`curl`中的参数支持:

```java
curl -i -d id=100 http://localhost:8080/spring-rest/ex/bars
```

在本例中，参数是直接绑定的，没有先声明。

对于更高级的场景， **`@RequestMapping`可以选择性地定义参数**,作为缩小请求映射的另一种方式:

```java
@RequestMapping(value = "/ex/bars", params = "id", method = GET)
@ResponseBody
public String getBarBySimplePathWithExplicitRequestParam(
  @RequestParam("id") long id) {
    return "Get a specific Bar with id=" + id;
}
```

甚至允许更灵活的映射。可以设置多个`params`值，并且不必使用所有的值:

```java
@RequestMapping(
  value = "/ex/bars", 
  params = { "id", "second" }, 
  method = GET)
@ResponseBody
public String getBarBySimplePathWithExplicitRequestParams(
  @RequestParam("id") long id) {
    return "Narrow Get a specific Bar with id=" + id;
}
```

当然，还有对 URI 的请求，例如:

```java
http://localhost:8080/spring-rest/ex/bars?id=100&second;=something
```

将总是被映射到最佳匹配——更窄的匹配，它定义了`id`和`second`参数。

## 6。`RequestMapping`角落案例

### 6.1。`@RequestMapping` —多条路径映射到同一个控制器方法

虽然单个`@RequestMapping`路径值通常用于单个控制器方法(只是好的实践，而不是硬性规定)，但在某些情况下，将多个请求映射到同一个方法可能是必要的。

在这种情况下，`@RequestMapping`的`value`属性接受多个映射，而不仅仅是一个:

```java
@RequestMapping(
  value = { "/ex/advanced/bars", "/ex/advanced/foos" }, 
  method = GET)
@ResponseBody
public String getFoosOrBarsByPath() {
    return "Advanced - Get some Foos or Bars";
}
```

现在，这两个`curl`命令应该使用相同的方法:

```java
curl -i http://localhost:8080/spring-rest/ex/advanced/foos
curl -i http://localhost:8080/spring-rest/ex/advanced/bars
```

### 6.2。`@RequestMapping` —对同一个控制器方法的多个 HTTP 请求方法

使用不同 HTTP 动词的多个请求可以映射到同一个控制器方法:

```java
@RequestMapping(
  value = "/ex/foos/multiple", 
  method = { RequestMethod.PUT, RequestMethod.POST }
)
@ResponseBody
public String putAndPostFoos() {
    return "Advanced - PUT and POST within single method";
}
```

有了`curl`，这两者现在将达到相同的方法:

```java
curl -i -X POST http://localhost:8080/spring-rest/ex/foos/multiple
curl -i -X PUT http://localhost:8080/spring-rest/ex/foos/multiple
```

### 6.3。`@RequestMapping` —所有请求的回退

要使用特定的 HTTP 方法为所有请求实现简单的回退，例如，对于 GET:

```java
@RequestMapping(value = "*", method = RequestMethod.GET)
@ResponseBody
public String getFallback() {
    return "Fallback for GET Requests";
}
```

或者甚至针对所有请求:

```java
@RequestMapping(
  value = "*", 
  method = { RequestMethod.GET, RequestMethod.POST ... })
@ResponseBody
public String allFallback() {
    return "Fallback for All Requests";
}
```

### 6.4。不明确的映射错误

当 Spring 评估两个或更多请求映射对于不同的控制器方法是相同的时，就会出现不明确的映射错误。当请求映射具有相同的 HTTP 方法、URL、参数、头和媒体类型时，它是相同的。

例如，这是一个不明确的映射:

```java
@GetMapping(value = "foos/duplicate" )
public String duplicate() {
    return "Duplicate";
}

@GetMapping(value = "foos/duplicate" )
public String duplicateEx() {
    return "Duplicate";
}
```

抛出的异常通常包含如下错误消息:

```java
Caused by: java.lang.IllegalStateException: Ambiguous mapping.
  Cannot map 'fooMappingExamplesController' method 
  public java.lang.String org.baeldung.web.controller.FooMappingExamplesController.duplicateEx()
  to {[/ex/foos/duplicate],methods=[GET]}:
  There is already 'fooMappingExamplesController' bean method
  public java.lang.String org.baeldung.web.controller.FooMappingExamplesController.duplicate() mapped.
```

仔细阅读错误消息可以发现，Spring 无法映射方法`org.baeldung.web.controller.FooMappingExamplesController.duplicateEx(),` ，因为它与已经映射的`org.baeldung.web.controller.FooMappingExamplesController.duplicate().`有冲突映射

**下面的代码片段不会导致不明确的映射错误，因为两种方法返回不同的内容类型**:

```java
@GetMapping(value = "foos/duplicate", produces = MediaType.APPLICATION_XML_VALUE)
public String duplicateXml() {
    return "<message>Duplicate</message>";
}

@GetMapping(value = "foos/duplicate", produces = MediaType.APPLICATION_JSON_VALUE)
public String duplicateJson() {
    return "{\"message\":\"Duplicate\"}";
}
```

这种区别允许我们的控制器基于请求中提供的`Accepts`头返回正确的数据表示。

解决这个问题的另一种方法是更新分配给这两种方法之一的 URL。

## 7。新请求映射快捷方式

Spring Framework 4.3 引入了[几个新的](/web/20221130175138/https://www.baeldung.com/spring-new-requestmapping-shortcuts) HTTP 映射注释，全部基于`@RequestMapping` :

*   `@GetMapping`
*   `@PostMapping`
*   `@PutMapping`
*   `@DeleteMapping`
*   `@PatchMapping`

这些新注释可以提高代码的可读性，减少代码的冗长。

让我们通过创建一个支持 CRUD 操作的 RESTful API 来看看这些新注释的作用:

```java
@GetMapping("/{id}")
public ResponseEntity<?> getBazz(@PathVariable String id){
    return new ResponseEntity<>(new Bazz(id, "Bazz"+id), HttpStatus.OK);
}

@PostMapping
public ResponseEntity<?> newBazz(@RequestParam("name") String name){
    return new ResponseEntity<>(new Bazz("5", name), HttpStatus.OK);
}

@PutMapping("/{id}")
public ResponseEntity<?> updateBazz(
  @PathVariable String id,
  @RequestParam("name") String name) {
    return new ResponseEntity<>(new Bazz(id, name), HttpStatus.OK);
}

@DeleteMapping("/{id}")
public ResponseEntity<?> deleteBazz(@PathVariable String id){
    return new ResponseEntity<>(new Bazz(id), HttpStatus.OK);
}
```

深入探究这些可以在[这里](/web/20221130175138/https://www.baeldung.com/spring-new-requestmapping-shortcuts)找到。

## 8。弹簧配置

考虑到我们的`FooController`是在下面的包中定义的，Spring MVC 配置非常简单:

```java
package org.baeldung.spring.web.controller;

@Controller
public class FooController { ... }
```

我们只需要一个`@Configuration`类来启用完整的 MVC 支持并为控制器配置类路径扫描:

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "org.baeldung.spring.web.controller" })
public class MvcConfig {
    //
}
```

## 9。结论

本文主要关注 Spring 中的 **`@RequestMapping`注释，讨论一个简单的用例，HTTP 头的映射，用`@PathVariable`绑定 URI 的部分，以及使用 URI 参数和`@RequestParam`注释。**

如果你想学习如何在 Spring MVC 中使用另一个核心注释，你可以在这里探索[的`@ModelAttribute`注释。](/web/20221130175138/https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)

这篇文章的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221130175138/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)