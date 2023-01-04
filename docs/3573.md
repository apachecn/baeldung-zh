# 弹簧响应状态异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-response-status-exception>

## 1.概观

在这个快速教程中，我们将讨论 Spring 5 中引入的新的`ResponseStatusException` 类。这个类支持对 HTTP 响应应用 HTTP 状态代码。

RESTful 应用程序可以通过**在对客户端**的响应中返回正确的状态代码来传达 HTTP 请求的成功或失败。简而言之，适当的状态代码可以帮助客户端识别应用程序处理请求时可能出现的问题。

## 2.`ResponseStatus`

在我们深入研究`ResponseStatusException,` 之前，让我们快速看一下`@ResponseStatus`注释。这个注释是在 Spring 3 中引入的，用于将 HTTP 状态代码应用于 HTTP 响应`.`

我们可以使用`@ResponseStatus`注释来设置 HTTP 响应中的状态和原因:

```
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "Actor Not Found")
public class ActorNotFoundException extends Exception {
    // ...
}
```

如果在处理 HTTP 请求时抛出这个异常，那么响应将包括这个注释中指定的 HTTP 状态。

`@ResponseStatus`方法的一个缺点是它创建了与异常的紧密耦合。在我们的例子中，所有类型为`ActorNotFoundException`的异常将在响应中生成相同的错误消息和状态代码。

## 3.`ResponseStatusException`

`ResponseStatusException`是`@ResponseStatus` 的编程替代，是用于将状态代码应用于 HTTP 响应的异常的基类。它是一个`RuntimeException`，因此不需要显式添加到方法签名中。

Spring 提供了 3 个构造函数来生成`ResponseStatusException:`

```
ResponseStatusException(HttpStatus status)
ResponseStatusException(HttpStatus status, java.lang.String reason)
ResponseStatusException(
  HttpStatus status, 
  java.lang.String reason, 
  java.lang.Throwable cause
)
```

`ResponseStatusException,`构造函数参数:

*   状态–设置为 HTTP 响应的 HTTP 状态
*   原因–解释 HTTP 响应异常设置的消息
*   原因–导致`ResponseStatusException`的`Throwable`原因

注意:在 Spring 中，`HandlerExceptionResolver`拦截并处理任何控制器没有处理的异常。

这些处理程序之一，`ResponseStatusExceptionResolver,`寻找任何由`@ResponseStatus`注释的`ResponseStatusException`或未捕获的异常，然后提取 HTTP 状态代码&原因，并将它们包含在 HTTP 响应中。

### 3.1.`ResponseStatusException`好处

使用好处不多:

*   首先，相同类型的异常可以单独处理，并且可以在响应上设置不同的状态代码，从而减少紧耦合
*   其次，它避免了创建不必要的额外异常类
*   最后，它提供了对异常处理的更多控制，因为异常可以通过编程方式创建

## 4.例子

### 4.1.生成`ResponseStatusException`

现在，让我们看一个生成`ResponseStatusException`的例子:

```
@GetMapping("/actor/{id}")
public String getActorName(@PathVariable("id") int id) {
    try {
        return actorService.getActor(id);
    } catch (ActorNotFoundException ex) {
        throw new ResponseStatusException(
          HttpStatus.NOT_FOUND, "Actor Not Found", ex);
    }
}
```

Spring Boot 提供了一个默认的`/error`映射，返回一个带有 HTTP 状态的 JSON 响应。

下面是回应的样子:

```
$ curl -i -s -X GET http://localhost:8081/actor/8
HTTP/1.1 404
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sat, 26 Dec 2020 19:38:09 GMT

{
    "timestamp": "2020-12-26T19:38:09.426+00:00",
    "status": 404,
    "error": "Not Found",
    "message": "",
    "path": "/actor/8"
}
```

从 2.3 版本开始，Spring Boot 不再在默认错误页面显示错误信息。原因是为了降低向客户泄露信息的风险

要改变默认行为，我们可以使用一个`server.error.include-message`属性。

让我们将它设置为`always`，看看会发生什么:

```
$ curl -i -s -X GET http://localhost:8081/actor/8
HTTP/1.1 404
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sat, 26 Dec 2020 19:39:11 GMT

{
    "timestamp": "2020-12-26T19:39:11.426+00:00",
    "status": 404,
    "error": "Not Found",
    "message": "Actor Not Found",
    "path": "/actor/8"
}
```

正如我们看到的，这一次，响应包含一个`“Actor Not Found”`错误消息。

### 4.2.不同的状态代码–相同的异常类型

现在，让我们看看当引发相同类型的异常时，如何将不同的状态代码设置为 HTTP 响应:

```
@PutMapping("/actor/{id}/{name}")
public String updateActorName(
  @PathVariable("id") int id, 
  @PathVariable("name") String name) {

    try {
        return actorService.updateActor(id, name);
    } catch (ActorNotFoundException ex) {
        throw new ResponseStatusException(
          HttpStatus.BAD_REQUEST, "Provide correct Actor Id", ex);
    }
}
```

下面是回应的样子:

```
$ curl -i -s -X PUT http://localhost:8081/actor/8/BradPitt
HTTP/1.1 400
...
{
    "timestamp": "2018-02-01T04:28:32.917+0000",
    "status": 400,
    "error": "Bad Request",
    "message": "Provide correct Actor Id",
    "path": "/actor/8/BradPitt"
}
```

## 5.结论

在这个快速教程中，我们讨论了如何在我们的程序中构造一个`ResponseStatusException`。

我们还强调了在 HTTP 响应中设置 HTTP 状态代码是一种比`@ResponseStatus`注释更好的编程方式。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221213031627/https://github.com/eugenp/tutorials/tree/master/spring-5)