# REST API 的自定义错误消息处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/global-error-handler-in-a-spring-rest-api>

## 1。概述

在本教程中，我们将讨论如何为 Spring REST API 实现一个全局错误处理器。

我们将使用每个异常的语义为客户端构建有意义的错误消息，明确的目标是为客户端提供所有信息，以便轻松地诊断问题。

## 延伸阅读:

## [弹簧响应状态异常](/web/20220720100342/https://www.baeldung.com/spring-response-status-exception)

Learn how to apply status codes to HTTP responses in Spring with ResponseStatusException.[Read more](/web/20220720100342/https://www.baeldung.com/spring-response-status-exception) →

## [带弹簧的刀架错误处理](/web/20220720100342/https://www.baeldung.com/exception-handling-for-rest-with-spring)

Exception Handling for a REST API - illustrate the new Spring 3.2 recommended approach as well as earlier solutions .[Read more](/web/20220720100342/https://www.baeldung.com/exception-handling-for-rest-with-spring) →

## 2。自定义错误消息

让我们从实现一个通过网络发送错误的简单结构开始——`ApiError`:

```java
public class ApiError {

    private HttpStatus status;
    private String message;
    private List<String> errors;

    public ApiError(HttpStatus status, String message, List<String> errors) {
        super();
        this.status = status;
        this.message = message;
        this.errors = errors;
    }

    public ApiError(HttpStatus status, String message, String error) {
        super();
        this.status = status;
        this.message = message;
        errors = Arrays.asList(error);
    }
}
```

这里的信息应该很简单:

*   `status`–HTTP 状态代码
*   `message`–与异常相关的错误信息
*   `error`–构建的错误信息列表

当然，对于 Spring 中实际的异常处理逻辑，[我们将使用](/web/20220720100342/https://www.baeldung.com/exception-handling-for-rest-with-spring)的`@ControllerAdvice`注释:

```java
@ControllerAdvice
public class CustomRestExceptionHandler extends ResponseEntityExceptionHandler {
    ...
}
```

## 3。处理错误请求异常

### 3.1。异常处理

现在让我们看看如何处理最常见的客户端错误——基本上是客户端向 API 发送无效请求的场景:

*   `BindException`–当出现致命的绑定错误时，抛出该异常。
*   `MethodArgumentNotValidException`–当用`@Valid`注释的参数验证失败时，抛出该异常:

```java
@Override
protected ResponseEntity<Object> handleMethodArgumentNotValid(
  MethodArgumentNotValidException ex, 
  HttpHeaders headers, 
  HttpStatus status, 
  WebRequest request) {
    List<String> errors = new ArrayList<String>();
    for (FieldError error : ex.getBindingResult().getFieldErrors()) {
        errors.add(error.getField() + ": " + error.getDefaultMessage());
    }
    for (ObjectError error : ex.getBindingResult().getGlobalErrors()) {
        errors.add(error.getObjectName() + ": " + error.getDefaultMessage());
    }

    ApiError apiError = 
      new ApiError(HttpStatus.BAD_REQUEST, ex.getLocalizedMessage(), errors);
    return handleExceptionInternal(
      ex, apiError, headers, apiError.getStatus(), request);
} 
```

请注意，**我们从`ResponseEntityExceptionHandler`中重写了一个基础方法，并提供了我们自己的定制实现。**

情况并不总是这样。有时，我们需要处理一个自定义的异常，这个异常在基类中没有默认的实现，我们将在后面看到。

接下来:

*   `MissingServletRequestPartException`–当找不到多部分请求的一部分时，抛出该异常。

*   `MissingServletRequestParameterException`–当请求缺少参数时，抛出该异常:

```java
@Override
protected ResponseEntity<Object> handleMissingServletRequestParameter(
  MissingServletRequestParameterException ex, HttpHeaders headers, 
  HttpStatus status, WebRequest request) {
    String error = ex.getParameterName() + " parameter is missing";

    ApiError apiError = 
      new ApiError(HttpStatus.BAD_REQUEST, ex.getLocalizedMessage(), error);
    return new ResponseEntity<Object>(
      apiError, new HttpHeaders(), apiError.getStatus());
}
```

*   `ConstraintViolationException`–此异常报告违反约束的结果:

```java
@ExceptionHandler({ ConstraintViolationException.class })
public ResponseEntity<Object> handleConstraintViolation(
  ConstraintViolationException ex, WebRequest request) {
    List<String> errors = new ArrayList<String>();
    for (ConstraintViolation<?> violation : ex.getConstraintViolations()) {
        errors.add(violation.getRootBeanClass().getName() + " " + 
          violation.getPropertyPath() + ": " + violation.getMessage());
    }

    ApiError apiError = 
      new ApiError(HttpStatus.BAD_REQUEST, ex.getLocalizedMessage(), errors);
    return new ResponseEntity<Object>(
      apiError, new HttpHeaders(), apiError.getStatus());
}
```

*   `TypeMismatchException`–试图用错误的类型设置 bean 属性时抛出此异常。

*   `MethodArgumentTypeMismatchException`–当方法参数不是预期的类型时，抛出此异常:

```java
@ExceptionHandler({ MethodArgumentTypeMismatchException.class })
public ResponseEntity<Object> handleMethodArgumentTypeMismatch(
  MethodArgumentTypeMismatchException ex, WebRequest request) {
    String error = 
      ex.getName() + " should be of type " + ex.getRequiredType().getName();

    ApiError apiError = 
      new ApiError(HttpStatus.BAD_REQUEST, ex.getLocalizedMessage(), error);
    return new ResponseEntity<Object>(
      apiError, new HttpHeaders(), apiError.getStatus());
}
```

### 3.2。从客户端使用 API

现在让我们来看一个遇到`MethodArgumentTypeMismatchException`的测试。

我们将**发送一个请求，将`id`作为`String`而不是`long`** :

```java
@Test
public void whenMethodArgumentMismatch_thenBadRequest() {
    Response response = givenAuth().get(URL_PREFIX + "/api/foos/ccc");
    ApiError error = response.as(ApiError.class);

    assertEquals(HttpStatus.BAD_REQUEST, error.getStatus());
    assertEquals(1, error.getErrors().size());
    assertTrue(error.getErrors().get(0).contains("should be of type"));
}
```

最后，考虑同样的请求:

```java
Request method:	GET
Request path:	http://localhost:8080/spring-security-rest/api/foos/ccc 
```

**下面是这种 JSON 错误响应的样子**:

```java
{
    "status": "BAD_REQUEST",
    "message": 
      "Failed to convert value of type [java.lang.String] 
       to required type [java.lang.Long]; nested exception 
       is java.lang.NumberFormatException: For input string: \"ccc\"",
    "errors": [
        "id should be of type java.lang.Long"
    ]
}
```

## 4。把手`NoHandlerFoundException`

接下来，我们可以定制 servlet 来抛出这个异常，而不是发送 404 响应:

```java
<servlet>
    <servlet-name>api</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet</servlet-class>        
    <init-param>
        <param-name>throwExceptionIfNoHandlerFound</param-name>
        <param-value>true</param-value>
    </init-param>
</servlet>
```

然后，一旦发生这种情况，我们可以像处理任何其他异常一样简单地处理它:

```java
@Override
protected ResponseEntity<Object> handleNoHandlerFoundException(
  NoHandlerFoundException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
    String error = "No handler found for " + ex.getHttpMethod() + " " + ex.getRequestURL();

    ApiError apiError = new ApiError(HttpStatus.NOT_FOUND, ex.getLocalizedMessage(), error);
    return new ResponseEntity<Object>(apiError, new HttpHeaders(), apiError.getStatus());
}
```

这里有一个简单的测试:

```java
@Test
public void whenNoHandlerForHttpRequest_thenNotFound() {
    Response response = givenAuth().delete(URL_PREFIX + "/api/xx");
    ApiError error = response.as(ApiError.class);

    assertEquals(HttpStatus.NOT_FOUND, error.getStatus());
    assertEquals(1, error.getErrors().size());
    assertTrue(error.getErrors().get(0).contains("No handler found"));
}
```

让我们看看完整的请求:

```java
Request method:	DELETE
Request path:	http://localhost:8080/spring-security-rest/api/xx
```

以及**错误 JSON 响应**:

```java
{
    "status":"NOT_FOUND",
    "message":"No handler found for DELETE /spring-security-rest/api/xx",
    "errors":[
        "No handler found for DELETE /spring-security-rest/api/xx"
    ]
}
```

接下来，我们将看看另一个有趣的异常。

## 5。把手`HttpRequestMethodNotSupportedException`

当我们用不支持的 HTTP 方法发送请求时，会发生`HttpRequestMethodNotSupportedException`:

```java
@Override
protected ResponseEntity<Object> handleHttpRequestMethodNotSupported(
  HttpRequestMethodNotSupportedException ex, 
  HttpHeaders headers, 
  HttpStatus status, 
  WebRequest request) {
    StringBuilder builder = new StringBuilder();
    builder.append(ex.getMethod());
    builder.append(
      " method is not supported for this request. Supported methods are ");
    ex.getSupportedHttpMethods().forEach(t -> builder.append(t + " "));

    ApiError apiError = new ApiError(HttpStatus.METHOD_NOT_ALLOWED, 
      ex.getLocalizedMessage(), builder.toString());
    return new ResponseEntity<Object>(
      apiError, new HttpHeaders(), apiError.getStatus());
}
```

下面是重现该异常的简单测试:

```java
@Test
public void whenHttpRequestMethodNotSupported_thenMethodNotAllowed() {
    Response response = givenAuth().delete(URL_PREFIX + "/api/foos/1");
    ApiError error = response.as(ApiError.class);

    assertEquals(HttpStatus.METHOD_NOT_ALLOWED, error.getStatus());
    assertEquals(1, error.getErrors().size());
    assertTrue(error.getErrors().get(0).contains("Supported methods are"));
}
```

这是完整的请求:

```java
Request method:	DELETE
Request path:	http://localhost:8080/spring-security-rest/api/foos/1
```

以及**错误 JSON 响应**:

```java
{
    "status":"METHOD_NOT_ALLOWED",
    "message":"Request method 'DELETE' not supported",
    "errors":[
        "DELETE method is not supported for this request. Supported methods are GET "
    ]
}
```

## 6。把手`HttpMediaTypeNotSupportedException`

现在让我们来处理`HttpMediaTypeNotSupportedException`，它发生在客户端发送一个带有不支持的媒体类型的请求时:

```java
@Override
protected ResponseEntity<Object> handleHttpMediaTypeNotSupported(
  HttpMediaTypeNotSupportedException ex, 
  HttpHeaders headers, 
  HttpStatus status, 
  WebRequest request) {
    StringBuilder builder = new StringBuilder();
    builder.append(ex.getContentType());
    builder.append(" media type is not supported. Supported media types are ");
    ex.getSupportedMediaTypes().forEach(t -> builder.append(t + ", "));

    ApiError apiError = new ApiError(HttpStatus.UNSUPPORTED_MEDIA_TYPE, 
      ex.getLocalizedMessage(), builder.substring(0, builder.length() - 2));
    return new ResponseEntity<Object>(
      apiError, new HttpHeaders(), apiError.getStatus());
}
```

下面是一个关于这个问题的简单测试:

```java
@Test
public void whenSendInvalidHttpMediaType_thenUnsupportedMediaType() {
    Response response = givenAuth().body("").post(URL_PREFIX + "/api/foos");
    ApiError error = response.as(ApiError.class);

    assertEquals(HttpStatus.UNSUPPORTED_MEDIA_TYPE, error.getStatus());
    assertEquals(1, error.getErrors().size());
    assertTrue(error.getErrors().get(0).contains("media type is not supported"));
}
```

最后，这里有一个请求示例:

```java
Request method:	POST
Request path:	http://localhost:8080/spring-security-
Headers:	Content-Type=text/plain; charset=ISO-8859-1
```

和**错误 JSON 响应:**

```java
{
    "status":"UNSUPPORTED_MEDIA_TYPE",
    "message":"Content type 'text/plain;charset=ISO-8859-1' not supported",
    "errors":["text/plain;charset=ISO-8859-1 media type is not supported. 
       Supported media types are text/xml 
       application/x-www-form-urlencoded 
       application/*+xml 
       application/json;charset=UTF-8 
       application/*+json;charset=UTF-8 */"
    ]
}
```

## 7。默认处理程序

最后，我们将实现一个回退处理程序—一种无所不包的逻辑类型，用于处理所有其他没有特定处理程序的异常:

```java
@ExceptionHandler({ Exception.class })
public ResponseEntity<Object> handleAll(Exception ex, WebRequest request) {
    ApiError apiError = new ApiError(
      HttpStatus.INTERNAL_SERVER_ERROR, ex.getLocalizedMessage(), "error occurred");
    return new ResponseEntity<Object>(
      apiError, new HttpHeaders(), apiError.getStatus());
}
```

## 8。结论

为 Spring REST API 构建一个合适的、成熟的错误处理程序是困难的，并且肯定是一个迭代的过程。希望本教程将是一个很好的起点，也是一个很好的锚，帮助 API 客户快速、轻松地诊断错误并克服它们。

本教程的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220720100342/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest)中找到。这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。