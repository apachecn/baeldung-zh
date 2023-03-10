# 用 Spring 在 JSON 中呈现异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-exceptions-json>

## 1.介绍

快乐路径 REST 很容易理解，Spring 使得这在 Java 中很容易实现。

但是当事情出错的时候呢？

在本教程中，我们将回顾使用 Spring 传递 Java 异常作为 JSON 响应的一部分。

更广泛地看，查看我们关于用 Spring 和[创建 Java 全局异常处理器](/web/20221205232741/https://www.baeldung.com/java-global-exception-handler)对 REST 进行[错误处理的帖子。](/web/20221205232741/https://www.baeldung.com/exception-handling-for-rest-with-spring)

## 2.带注释的解决方案

我们将使用三个基本的 Spring MVC 注释来解决这个问题:

*   **`@RestControllerAdvice`** ，其中包含`@ControllerAdvice`将周围的类注册为每个`@Controller`都应该知道的东西，以及`@ResponseBody`告诉 Spring 将该方法的响应呈现为 JSON
*   **`@ExceptionHandler`告诉 Spring 对于给定的异常应该调用我们的哪个方法**

这些一起创建了一个 Spring bean，它处理我们为其配置的任何异常。这里有更多关于[与`@ControllerAdvice`和`@ExceptionHandler`结合](/web/20221205232741/https://www.baeldung.com/exception-handling-for-rest-with-spring#controlleradvice)使用的细节。

## 3.例子

首先，让我们创建一个任意的自定义异常返回给客户端:

```java
public class CustomException extends RuntimeException {
    // constructors
}
```

其次，让我们定义一个类来处理异常，并将其作为 JSON 传递给客户机:

```java
@RestControllerAdvice
public class ErrorHandler {

    @ExceptionHandler(CustomException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public CustomException handleCustomException(CustomException ce) {
        return ce;
    }

}
```

注意，我们添加了 [`@ResponseStatus`](/web/20221205232741/https://www.baeldung.com/spring-response-status#error-handling) 注释。这将指定发送给客户机的状态代码，在我们的例子中是内部服务器错误。此外，`@ResponseBody`将确保对象以 JSON 序列化的形式发送回客户机。最后，下面是一个虚拟控制器，展示了如何抛出异常的示例:

```java
@Controller
public class MainController {

    @GetMapping("/")
    public void index() throws CustomException {
        throw new CustomException();
    }

}
```

## 4.结论

在这篇文章中，我们展示了如何在 Spring 中处理异常。此外，我们展示了如何将它们以 JSON 序列化的形式发送回客户机。

这篇文章的完整实现可以在 Github 上找到[。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221205232741/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data)