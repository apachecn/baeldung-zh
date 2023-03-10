# 带弹簧的刀架的错误处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/exception-handling-for-rest-with-spring>

## 1。概述

本教程将展示如何用 Spring 为 REST API 实现异常处理。我们还将了解一些历史概况，看看不同版本引入了哪些新选项。

**在 Spring 3.2 之前，Spring MVC 应用程序中处理异常的两种主要方法是`HandlerExceptionResolver`或`@ExceptionHandler`注释。两者都有一些明显的缺点。**

**从 3.2 开始，我们有了`@ControllerAdvice`注释**来解决前两种解决方案的局限性，并在整个应用程序中促进统一的异常处理。

现在 **Spring 5 引入了`ResponseStatusException `类**——REST API 中基本错误处理的快速方法。

所有这些都有一个共同点:它们很好地处理了关注点的分离。该应用程序通常会抛出异常来指示某种类型的故障，这些故障将被单独处理。

最后，我们将看到 Spring Boot 带来了什么，以及我们如何配置它来满足我们的需求。

## 延伸阅读:

## 【REST API 的自定义错误消息处理

Implement a Global Exception Handler for a REST API with Spring.[Read more](/web/20220813073028/https://www.baeldung.com/global-error-handler-in-a-spring-rest-api) →

## [Spring 数据剩余验证器指南](/web/20220813073028/https://www.baeldung.com/spring-data-rest-validators)

Quick and practical guide to Spring Data REST Validators[Read more](/web/20220813073028/https://www.baeldung.com/spring-data-rest-validators) →

## [Spring MVC 自定义验证](/web/20220813073028/https://www.baeldung.com/spring-mvc-custom-validator)

Learn how to build a custom validation annotation and use it in Spring MVC.[Read more](/web/20220813073028/https://www.baeldung.com/spring-mvc-custom-validator) →

## 2。解决方案 1:控制器级`@ExceptionHandler`

第一种解决方案在`@Controller`级别有效。我们将定义一个方法来处理异常，并用`@ExceptionHandler`进行注释:

```java
public class FooController{

    //...
    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        //
    }
}
```

这种方法有一个主要的缺点:带注释的方法仅对特定的控制器有效，而不是对整个应用程序有效。当然，将这一点添加到每个控制器会使它不太适合一般的异常处理机制。

我们可以通过让**所有控制器扩展一个基本控制器类来解决这个限制。**

然而，这种解决方案对于应用程序来说可能是个问题，因为无论什么原因，这都是不可能的。例如，控制器可能已经从另一个基类扩展，该基类可能在另一个 jar 中或者不可直接修改，或者它们本身不可直接修改。

接下来，我们将看看解决异常处理问题的另一种方法——这种方法是全局的，不包括对现有构件(如控制器)的任何更改。

## 3。`HandlerExceptionResolver`方案二:

第二个解决方案是定义一个`HandlerExceptionResolver.`,这将解决应用程序抛出的任何异常。它还允许我们在 REST API 中实现一个统一的异常处理机制。

在使用自定义解析器之前，让我们先看一下现有的实现。

### 3.1。`ExceptionHandlerExceptionResolver`

这个解析器是在 Spring 3.1 中引入的，默认情况下在`DispatcherServlet`中启用。这实际上是前面介绍的@ `ExceptionHandler`机制如何工作的核心组件。

### 3.2。`DefaultHandlerExceptionResolver`

这个解析器是在 Spring 3.0 中引入的，默认情况下在`DispatcherServlet`中启用。

它用于解决标准的 Spring 异常到它们相应的 [HTTP 状态代码](/web/20220813073028/https://www.baeldung.com/cs/http-status-codes)，即客户端错误`4xx`和服务器错误`5xx`状态代码。[这里是它处理的 Spring 异常的**完整列表**](https://web.archive.org/web/20220813073028/http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-ann-rest-spring-mvc-exceptions "Handling Standard Spring MVC Exceptions") 以及它们如何映射到状态代码。

虽然它确实正确地设置了响应的状态代码，但是一个**限制是它没有为响应的主体设置任何内容。**对于 REST API 来说——状态代码真的不足以向客户端呈现信息——响应还必须有一个主体，以允许应用程序给出关于失败的附加信息。

这可以通过`ModelAndView`配置视图分辨率和渲染错误内容来解决，但解决方案显然不是最优的。这就是为什么 Spring 3.2 引入了一个更好的选项，我们将在后面的部分讨论。

### 3.3。`ResponseStatusExceptionResolver`

这个解析器也是在 Spring 3.0 中引入的，默认情况下在`DispatcherServlet`中启用。

它的主要职责是使用定制异常上可用的`@ResponseStatus`注释，并将这些异常映射到 HTTP 状态代码。

这种自定义异常可能如下所示:

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class MyResourceNotFoundException extends RuntimeException {
    public MyResourceNotFoundException() {
        super();
    }
    public MyResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
    public MyResourceNotFoundException(String message) {
        super(message);
    }
    public MyResourceNotFoundException(Throwable cause) {
        super(cause);
    }
}
```

与`DefaultHandlerExceptionResolver`相同，这个解析器在处理响应主体的方式上有所限制——它确实在响应上映射了状态代码，但是主体仍然是`null.`

### 3.4。 **风俗`HandlerExceptionResolver`**

`DefaultHandlerExceptionResolver`和`ResponseStatusExceptionResolver`的结合为 Spring RESTful 服务提供了良好的错误处理机制。不利的一面是，如前所述，**无法控制身体的反应。**

理想情况下，我们希望能够输出 JSON 或 XML，这取决于客户要求的格式(通过`Accept` 头)。

仅此一点就证明了创建一个新的定制异常解析器的合理性:

```java
@Component
public class RestResponseStatusExceptionResolver extends AbstractHandlerExceptionResolver {

    @Override
    protected ModelAndView doResolveException(
      HttpServletRequest request, 
      HttpServletResponse response, 
      Object handler, 
      Exception ex) {
        try {
            if (ex instanceof IllegalArgumentException) {
                return handleIllegalArgument(
                  (IllegalArgumentException) ex, response, handler);
            }
            ...
        } catch (Exception handlerException) {
            logger.warn("Handling of [" + ex.getClass().getName() + "] 
              resulted in Exception", handlerException);
        }
        return null;
    }

    private ModelAndView 
      handleIllegalArgument(IllegalArgumentException ex, HttpServletResponse response) 
      throws IOException {
        response.sendError(HttpServletResponse.SC_CONFLICT);
        String accept = request.getHeader(HttpHeaders.ACCEPT);
        ...
        return new ModelAndView();
    }
}
```

这里需要注意的一个细节是，我们可以访问`request`本身，所以我们可以考虑客户端发送的`Accept`头的值。

例如，如果客户端请求`application/json`，那么在出现错误的情况下，我们希望确保返回用`application/json`编码的响应体。

另一个重要的实现细节是**我们返回一个`ModelAndView`——这是响应**的主体，它允许我们在上面设置任何必要的内容。

对于 Spring REST 服务的错误处理，这种方法是一种一致且易于配置的机制。

然而，它确实有局限性:它与底层的`HtttpServletResponse`交互，并且适合使用`ModelAndView`的旧 MVC 模型，所以仍然有改进的空间。

## 4。`@ControllerAdvice`方案三:

Spring 3.2 通过`@ControllerAdvice`注释为**带来了对全局`@ExceptionHandler` 的支持。**

这使得一种机制脱离了旧的 MVC 模型，并利用了`ResponseEntity`以及`@ExceptionHandler`的类型安全性和灵活性:

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value 
      = { IllegalArgumentException.class, IllegalStateException.class })
    protected ResponseEntity<Object> handleConflict(
      RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return handleExceptionInternal(ex, bodyOfResponse, 
          new HttpHeaders(), HttpStatus.CONFLICT, request);
    }
}
```

`@ControllerAdvice`注释允许我们**将我们以前的多个分散的`@ExceptionHandler`整合成一个单一的、全局的错误处理组件。**

实际机制非常简单，但也非常灵活:

*   它让我们可以完全控制响应的主体以及状态代码。
*   它提供了同一个方法的几个异常的映射，以便一起处理。
*   它很好地利用了较新的 RESTful `ResposeEntity`响应。

这里需要记住的一点是，**将使用`@ExceptionHandler`声明的异常与用作方法参数的异常相匹配。**

如果这些不匹配，编译器不会抱怨——没有理由应该抱怨 Spring 也不会抱怨。

然而，当异常在运行时实际抛出时，**异常解决机制将失败，并出现**:

```java
java.lang.IllegalStateException: No suitable resolver for argument [0] [type=...]
HandlerMethod details: ...
```

## 5。解决方案 4: `ResponseStatusException`(春 5 及以上)

Spring 5 引入了`ResponseStatusException`类。

我们可以创建它的一个实例，提供一个`HttpStatus`和一个可选的`reason`和一个`cause`:

```java
@GetMapping(value = "/{id}")
public Foo findById(@PathVariable("id") Long id, HttpServletResponse response) {
    try {
        Foo resourceById = RestPreconditions.checkFound(service.findOne(id));

        eventPublisher.publishEvent(new SingleResourceRetrievedEvent(this, response));
        return resourceById;
     }
    catch (MyResourceNotFoundException exc) {
         throw new ResponseStatusException(
           HttpStatus.NOT_FOUND, "Foo Not Found", exc);
    }
}
```

使用`ResponseStatusException`有什么好处？

*   非常适合原型开发:我们可以很快实现一个基本的解决方案。
*   一种类型，多种状态代码:一种异常类型可以导致多种不同的响应。**与`@ExceptionHandler`相比，这减少了紧密耦合。**
*   我们不必创建那么多定制的异常类。
*   我们对异常处理有了更多的控制，因为异常可以通过编程来创建。

权衡呢？

*   没有统一的异常处理方式:与提供全局方法的`@ControllerAdvice`相比，实施一些应用程序范围的约定更加困难。
*   代码复制:我们可能会发现自己在多个控制器中复制代码。

我们还应该注意，在一个应用程序中结合不同的方法是可能的。

**例如，我们可以在全球范围内实现一个`@ControllerAdvice`，但也可以在本地实现`ResponseStatusException`。**

然而，我们需要小心:如果同一个异常可以用多种方式处理，我们可能会注意到一些令人惊讶的行为。一个可能的惯例是总是以一种方式处理一种特定的异常。

更多细节和例子，请看我们关于`ResponseStatusException` 的[教程。](/web/20220813073028/https://www.baeldung.com/spring-response-status-exception)

## 6。处理 Spring Security 中被拒绝的访问

当经过身份验证的用户试图访问他没有足够权限访问的资源时，就会发生访问被拒绝。

### 6.1。 **休息和方法级安全**

最后，让我们看看如何处理由方法级安全注释引发的拒绝访问异常——`@PreAuthorize`、`@PostAuthorize`和`@Secure`。

当然，我们将使用我们之前讨论过的全局异常处理机制来处理`AccessDeniedException`:

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ AccessDeniedException.class })
    public ResponseEntity<Object> handleAccessDeniedException(
      Exception ex, WebRequest request) {
        return new ResponseEntity<Object>(
          "Access denied message here", new HttpHeaders(), HttpStatus.FORBIDDEN);
    }

    ...
}
```

## 7。Spring Boot 支持

**Spring Boot 提供了一个`ErrorController` 实现来以合理的方式处理错误。**

简而言之，它为浏览器提供了一个回退错误页面(也称为白色标签错误页面)，并为 RESTful、非 HTML 请求提供了一个 JSON 响应:

```java
{
    "timestamp": "2019-01-17T16:12:45.977+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "Error processing the request!",
    "path": "/my-endpoint-with-exceptions"
}
```

像往常一样，Spring Boot 允许用属性配置这些特性:

*   `server.error.whitelabel.enabled`:可用于禁用白标错误页面，并依靠 servlet 容器提供 HTML 错误消息
*   `server.error.include-stacktrace`:带有一个`always `值；在 HTML 和 JSON 默认响应中包含 stacktrace
*   `server.error.include-message: `从 2.3 版本开始，Spring Boot 在响应中隐藏了`message`字段，避免泄露敏感信息；我们可以使用这个带有`always `值的属性来启用它

除了这些属性，**我们还可以为/ `error, `覆盖白标页面提供自己的视图解析器映射。**

我们还可以通过在上下文中包含一个`ErrorAttributes ` bean 来定制希望在响应中显示的属性。我们可以扩展 Spring Boot 提供的`DefaultErrorAttributes`类来简化事情:

```java
@Component
public class MyCustomErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(
      WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> errorAttributes = 
          super.getErrorAttributes(webRequest, options);
        errorAttributes.put("locale", webRequest.getLocale()
            .toString());
        errorAttributes.remove("error");

        //...

        return errorAttributes;
    }
}
```

如果我们想进一步定义(或覆盖)应用程序如何处理特定内容类型的错误，我们可以注册一个`ErrorController ` bean。

同样，我们可以利用 Spring Boot 提供的缺省值`BasicErrorController `来帮助我们。

例如，假设我们想要定制应用程序如何处理 XML 端点中触发的错误。我们所要做的就是使用`@RequestMapping`定义一个公共方法，并声明它产生`application/xml`媒体类型:

```java
@Component
public class MyErrorController extends BasicErrorController {

    public MyErrorController(
      ErrorAttributes errorAttributes, ServerProperties serverProperties) {
        super(errorAttributes, serverProperties.getError());
    }

    @RequestMapping(produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<Map<String, Object>> xmlError(HttpServletRequest request) {

    // ...

    }
}
```

注意:这里我们仍然依赖于我们可能已经在项目中定义的`server.error.*`引导属性，这些属性被绑定到`ServerProperties ` bean。

## 8。结论

本文讨论了在 Spring 中为 REST API 实现异常处理机制的几种方法，从旧的机制开始，继续 Spring 3.2 支持，直到 4.x 和 5.x。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220813073028/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest)

对于 Spring 安全相关的代码，可以查看 [spring-security-rest](https://web.archive.org/web/20220813073028/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest) 模块。