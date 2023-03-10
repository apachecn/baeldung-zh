# 用 Spring MockMvc 测试异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-test-exceptions>

## 1。概述

在这篇短文中，我们将了解如何在控制器中抛出异常，以及如何使用 Spring MockMvc 测试这些异常。

## 2。控制器中抛出异常

让我们开始学习**如何从控制器**发起异常。

我们可以把从控制器公开的服务想象成普通的 Java 函数:

```java
@GetMapping("/exception/throw")
public void getException() throws Exception {
    throw new Exception("error");
} 
```

现在，让我们看看当我们调用这个服务时会发生什么。首先，我们会注意到服务的响应代码是 500，这意味着内部服务器错误。

其次，我们会收到这样的响应正文:

```java
{
    "timestamp": 1592074599854,
    "status": 500,
    "error": "Internal Server Error",
    "message": "No message available",
    "trace": "java.lang.Exception
              at com.baeldung.controllers.ExceptionController.getException(ExceptionController.java:26)
              ..."
}
```

总之，当我们从一个`RestController`抛出一个异常时，服务响应被自动映射到一个 500 响应代码，并且异常的堆栈跟踪被包含在响应体中。

## 3。将异常映射到 HTTP 响应代码

现在我们将学习**如何将我们的异常映射到不同的响应代码**而不是 500。

为了实现这一点，我们将创建自定义异常并使用 Spring 提供的`ResponseStatus`注释。让我们创建这些自定义异常:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BadArgumentsException extends RuntimeException {

    public BadArgumentsException(String message) {
        super(message);
    }
}
```

```java
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
public class InternalException extends RuntimeException {

    public InternalException(String message) {
        super(message);
    }
}
```

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

第二步也是最后一步是在控制器中创建一个简单的服务来抛出这些异常:

```java
@GetMapping("/exception/{exception_id}")
public void getSpecificException(@PathVariable("exception_id") String pException) {
    if("not_found".equals(pException)) {
        throw new ResourceNotFoundException("resource not found");
    }
    else if("bad_arguments".equals(pException)) {
        throw new BadArgumentsException("bad arguments");
    }
    else {
        throw new InternalException("internal error");
    }
}
```

现在，让我们看看服务对我们映射的不同异常的不同响应:

*   对于`not_found`，我们收到响应代码 404
*   给定值`bad_arguments`，我们收到响应代码 400
*   对于任何其他值，我们仍然接收 500 作为响应代码

除了响应代码之外，我们还将收到一个与上一节中收到的响应正文格式相同的正文。

## 4。测试我们的控制器

最后，我们将看到**如何测试我们的控制器正在抛出正确的异常**。

第一步是创建一个测试类，并创建一个`MockMvc`的实例:

```java
@Autowired
private MockMvc mvc; 
```

接下来，让我们为我们的服务可以接收的每个值创建测试用例:

```java
@Test
public void givenNotFound_whenGetSpecificException_thenNotFoundCode() throws Exception {
    String exceptionParam = "not_found";

    mvc.perform(get("/exception/{exception_id}", exceptionParam)
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isNotFound())
      .andExpect(result -> assertTrue(result.getResolvedException() instanceof ResourceNotFoundException))
      .andExpect(result -> assertEquals("resource not found", result.getResolvedException().getMessage()));
}

@Test
public void givenBadArguments_whenGetSpecificException_thenBadRequest() throws Exception {
    String exceptionParam = "bad_arguments";

    mvc.perform(get("/exception/{exception_id}", exceptionParam)
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isBadRequest())
      .andExpect(result -> assertTrue(result.getResolvedException() instanceof BadArgumentsException))
      .andExpect(result -> assertEquals("bad arguments", result.getResolvedException().getMessage()));
}

@Test
public void givenOther_whenGetSpecificException_thenInternalServerError() throws Exception {
    String exceptionParam = "dummy";

    mvc.perform(get("/exception/{exception_id}", exceptionParam)
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isInternalServerError())
      .andExpect(result -> assertTrue(result.getResolvedException() instanceof InternalException))
      .andExpect(result -> assertEquals("internal error", result.getResolvedException().getMessage()));
}
```

通过这些测试，我们检查响应代码、引发的异常类型和异常消息是否是每个值的预期值。

## 5。结论

在本教程中，我们学习了如何处理 Spring `RestController`中的异常，以及如何测试每个公开的服务是否抛出了预期的异常。

和往常一样，本文的完整源代码可以在 [GitHub](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-testing) 中找到。