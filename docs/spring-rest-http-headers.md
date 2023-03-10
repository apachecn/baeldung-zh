# 如何在 Spring REST 控制器中读取 HTTP 头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-http-headers>

## 1.概观

在这个快速教程中，我们将看看如何在一个 [Spring Rest 控制器](/web/20221120205507/https://www.baeldung.com/rest-with-spring-series)中访问 HTTP 头。

首先，我们将使用`@RequestHeader`注释来单独和一起读取头。

之后，我们将更深入地研究一下`@RequestHeader`属性。

## 延伸阅读:

## [春季请求映射](/web/20221120205507/https://www.baeldung.com/spring-requestmapping)

Spring @RequestMapping - Basic Example, @RequestParam, @PathVariable, Header mapping[Read more](/web/20221120205507/https://www.baeldung.com/spring-requestmapping) →

## [如何用弹簧 5 在响应上设置标题](/web/20221120205507/https://www.baeldung.com/spring-response-header)

Learn how to set a header on a specific response or on all response in Spring.[Read more](/web/20221120205507/https://www.baeldung.com/spring-response-header) →

## [使用 Spring ResponseEntity 操作 HTTP 响应](/web/20221120205507/https://www.baeldung.com/spring-response-entity)

Learn how to manipulate the HTTP response using the ResponseEntity class.[Read more](/web/20221120205507/https://www.baeldung.com/spring-response-entity) →

## 2.访问 HTTP 头

### 2.1.单独地

如果我们需要访问特定的标题，**,我们可以用标题名**配置`@RequestHeader`:

```java
@GetMapping("/greeting")
public ResponseEntity<String> greeting(@RequestHeader(HttpHeaders.ACCEPT_LANGUAGE) String language) {
    // code that uses the language variable
    return new ResponseEntity<String>(greeting, HttpStatus.OK);
}
```

然后，我们可以使用传递给我们的方法的变量来访问该值。如果在请求中没有找到名为`accept-language`的头，该方法返回“400 错误请求”错误。

我们的头不一定是字符串。如果我们知道我们的头是一个数字，我们可以将变量声明为数字类型:

```java
@GetMapping("/double")
public ResponseEntity<String> doubleNumber(@RequestHeader("my-number") int myNumber) {
    return new ResponseEntity<String>(String.format("%d * 2 = %d", 
      myNumber, (myNumber * 2)), HttpStatus.OK);
}
```

### 2.2.突然

如果我们不确定哪些头文件将会出现，或者我们需要比我们在方法签名中想要的更多的头文件，我们可以使用没有特定名称的`@RequestHeader`注释。

对于我们的变量类型，我们有几个选择:一个`Map`、`MultiValueMap`或`HttpHeaders`对象。

首先，让我们将请求头作为一个`Map`:

```java
@GetMapping("/listHeaders")
public ResponseEntity<String> listAllHeaders(
  @RequestHeader Map<String, String> headers) {
    headers.forEach((key, value) -> {
        LOG.info(String.format("Header '%s' = %s", key, value));
    });

    return new ResponseEntity<String>(
      String.format("Listed %d headers", headers.size()), HttpStatus.OK);
}
```

如果我们使用一个`Map`并且其中一个头有多个值，**我们将只得到第一个值。**这相当于在`MultiValueMap`上使用`getFirst`方法。

**如果我们的头可能有多个值，我们可以把它们作为`MultiValueMap`** :

```java
@GetMapping("/multiValue")
public ResponseEntity<String> multiValue(
  @RequestHeader MultiValueMap<String, String> headers) {
    headers.forEach((key, value) -> {
        LOG.info(String.format(
          "Header '%s' = %s", key, value.stream().collect(Collectors.joining("|"))));
    });

    return new ResponseEntity<String>(
      String.format("Listed %d headers", headers.size()), HttpStatus.OK);
}
```

我们也可以将我们的标题**作为一个`HttpHeaders`对象**:

```java
@GetMapping("/getBaseUrl")
public ResponseEntity<String> getBaseUrl(@RequestHeader HttpHeaders headers) {
    InetSocketAddress host = headers.getHost();
    String url = "http://" + host.getHostName() + ":" + host.getPort();
    return new ResponseEntity<String>(String.format("Base URL = %s", url), HttpStatus.OK);
}
```

`HttpHeaders`对象有通用应用程序头的访问器。

**当我们从一个`Map`、`MultiValueMap`或`HttpHeaders`对象按名称访问一个头时，如果它不存在，我们将得到一个`null`。**

## 3.`@RequestHeader`属性

既然我们已经了解了使用`@RequestHeader`注释访问请求头的基本知识，让我们更仔细地看看它的属性。

当我们专门命名我们的头时，我们已经隐式地使用了`name`或`value`属性:

```java
public ResponseEntity<String> greeting(@RequestHeader(HttpHeaders.ACCEPT_LANGUAGE) String language) {}
```

我们可以通过使用`name` 属性来完成同样的事情:

```java
public ResponseEntity<String> greeting(
  @RequestHeader(name = HttpHeaders.ACCEPT_LANGUAGE) String language) {}
```

接下来，让我们以完全相同的方式使用`value`属性:

```java
public ResponseEntity<String> greeting(
  @RequestHeader(value = HttpHeaders.ACCEPT_LANGUAGE) String language) {}
```

**当我们专门命名一个表头时，默认情况下表头是必选的。**如果在请求中没有找到报头，控制器返回 400 错误。

让我们使用`required`属性来表明我们的头不是必需的:

```java
@GetMapping("/nonRequiredHeader")
public ResponseEntity<String> evaluateNonRequiredHeader(
  @RequestHeader(value = "optional-header", required = false) String optionalHeader) {
    return new ResponseEntity<String>(String.format(
      "Was the optional header present? %s!",
        (optionalHeader == null ? "No" : "Yes")),HttpStatus.OK);
}
```

因为**如果请求**中没有头，我们的变量将会是`null`，我们需要确保进行适当的`null`检查。

让我们使用`defaultValue`属性为我们的头提供一个默认值:

```java
@GetMapping("/default")
public ResponseEntity<String> evaluateDefaultHeaderValue(
  @RequestHeader(value = "optional-header", defaultValue = "3600") int optionalHeader) {
    return new ResponseEntity<String>(
      String.format("Optional Header is %d", optionalHeader), HttpStatus.OK);
}
```

## 4.结论

在这个简短的教程中，我们学习了如何在 Spring REST 控制器中访问请求头。

首先，我们使用`@RequestHeader`注释为控制器方法提供请求头。

检查完基础知识后，我们详细查看了`@RequestHeader`注释的属性。

GitHub 上的[提供了示例代码。](https://web.archive.org/web/20221120205507/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-3)