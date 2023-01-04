# Spring @RequestMapping 新快捷方式注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-new-requestmapping-shortcuts>

## 1。概述

春天 4.3。[引入了](https://web.archive.org/web/20220812062535/https://jira.spring.io/browse/SPR-13442)一些非常酷的方法级组合注释来平滑典型 Spring MVC 项目中的处理`@RequestMapping`。

在这篇文章中，我们将学习如何有效地使用它们。

## 2。新注释

通常，如果我们想使用传统的`@RequestMapping`注释来实现 URL 处理程序，应该是这样的:

```java
@RequestMapping(value = "/get/{id}", method = RequestMethod.GET)
```

新的方法可以将其简化为:

```java
@GetMapping("/get/{id}")
```

Spring 目前支持五种类型的内置注释来处理不同类型的传入 HTTP 请求方法，它们是`GET, POST, PUT, DELETE`和`PATCH`。这些注释是:

*   `@GetMapping`
*   `@PostMapping`
*   `@PutMapping`
*   `@DeleteMapping`
*   `@PatchMapping`

从命名约定中我们可以看出，每个注释都是为了处理各自传入的请求方法类型，即`@GetMapping` 用于处理`GET`类型的请求方法，`@PostMapping`用于处理`POST`类型的请求方法，等等。

## 3。工作原理

上面所有的注释都已经用`@RequestMapping`和`method`元素中各自的值进行了内部注释。

例如，如果我们看一下`@GetMapping`注释的源代码，我们可以看到它已经用下面的方式用`RequestMethod.GET` 进行了注释:

```java
@Target({ java.lang.annotation.ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = { RequestMethod.GET })
public @interface GetMapping {
    // abstract codes
}
```

所有其他注释都以同样的方式创建，即`@PostMapping`用`RequestMethod.POST`注释，`@PutMapping`用`RequestMethod.PUT,`注释，等等。

注释的完整源代码可从[这里](https://web.archive.org/web/20220812062535/https://github.com/spring-projects/spring-framework/tree/master/spring-web/src/main/java/org/springframework/web/bind/annotation)获得。

## 4。实施

让我们尝试使用这些注释来构建一个快速休息应用程序。

请注意，因为我们将使用 Maven 构建项目，使用 Spring MVC 创建应用程序，所以我们需要在`pom.xml:`中添加必要的依赖项

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

最新版本的`spring-webmvc`可以在[中央 Maven 资源库](https://web.archive.org/web/20220812062535/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-webmvc%22)中获得。

现在，我们需要创建控制器来映射传入的请求 URL。在这个控制器中，我们将一个接一个地使用所有这些注释。

### 4.1。`@GetMapping`

```java
@GetMapping("/get")
public @ResponseBody ResponseEntity<String> get() {
    return new ResponseEntity<String>("GET Response", HttpStatus.OK);
} 
```

```java
@GetMapping("/get/{id}")
public @ResponseBody ResponseEntity<String>
  getById(@PathVariable String id) {
    return new ResponseEntity<String>("GET Response : " 
      + id, HttpStatus.OK);
}
```

### 4.2。`@PostMapping`

```java
@PostMapping("/post")
public @ResponseBody ResponseEntity<String> post() {
    return new ResponseEntity<String>("POST Response", HttpStatus.OK);
}
```

### 4.3。`@PutMapping`

```java
@PutMapping("/put")
public @ResponseBody ResponseEntity<String> put() {
    return new ResponseEntity<String>("PUT Response", HttpStatus.OK);
}
```

### 4.4。`@DeleteMapping`

```java
@DeleteMapping("/delete")
public @ResponseBody ResponseEntity<String> delete() {
    return new ResponseEntity<String>("DELETE Response", HttpStatus.OK);
}
```

### 4.5。`@PatchMapping`

```java
@PatchMapping("/patch")
public @ResponseBody ResponseEntity<String> patch() {
    return new ResponseEntity<String>("PATCH Response", HttpStatus.OK);
}
```

**注意点:**

*   我们使用了必要的注释来处理适当的传入 HTTP 方法，例如，用 URI `.` 、`@GetMapping`来处理“/get”URI、`@PostMapping`来处理“/post”URI 等等
*   因为我们正在创建一个基于 REST 的应用程序，所以我们返回一个包含 200 个响应代码的常量字符串(对于每个请求类型都是唯一的),以简化应用程序。在这种情况下，我们使用了 Spring 的`@ResponseBody`注释。
*   如果我们必须处理任何 URL 路径变量，我们可以简单地用更少的方式来处理，就像我们以前使用`@RequestMapping.`一样

## 5。测试应用程序

为了测试应用程序，我们需要使用 JUnit 创建几个测试用例。我们将使用`SpringJUnit4ClassRunner`来初始化测试类。我们将创建五个不同的测试用例来测试我们在控制器中声明的每个注释和每个处理程序。

让我们简化一下`@GetMapping:`的示例测试用例

```java
@Test 
public void giventUrl_whenGetRequest_thenFindGetResponse() 
  throws Exception {

    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders
      .get("/get");

    ResultMatcher contentMatcher = MockMvcResultMatchers.content()
      .string("GET Response");

    this.mockMvc.perform(builder).andExpect(contentMatcher)
      .andExpect(MockMvcResultMatchers.status().isOk());

}
```

正如我们所看到的，一旦我们点击了`GET` URL "/get "，我们就期待一个常量字符串"`GET Response`"。

现在，让我们创建测试用例来测试`@PostMapping`:

```java
@Test 
public void givenUrl_whenPostRequest_thenFindPostResponse() 
  throws Exception {

    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders
      .post("/post");

    ResultMatcher contentMatcher = MockMvcResultMatchers.content()
      .string("POST Response");

    this.mockMvc.perform(builder).andExpect(contentMatcher)
      .andExpect(MockMvcResultMatchers.status().isOk());

}
```

以同样的方式，我们创建了剩余的测试用例来测试所有的 HTTP 方法。

或者，我们总是可以使用任何常见的 REST 客户端，例如，PostMan、REST client 等，来测试我们的应用程序。在这种情况下，在使用 rest 客户端时，我们需要小心选择正确的 HTTP 方法类型。否则，它将抛出 405 错误状态。

## 6。结论

在本文中，我们快速介绍了使用传统 Spring MVC 框架进行快速 web 开发的不同类型的`@RequestMapping`快捷方式。我们可以利用这些快捷方式来创建一个干净的代码库。

像往常一样，您可以在 [Github 项目](https://web.archive.org/web/20220812062535/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics)中找到本教程的源代码。