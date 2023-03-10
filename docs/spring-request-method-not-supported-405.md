# Spring 中不支持的请求方法(405)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-request-method-not-supported-405>

## 1.概观

这篇简短的文章关注一个常见的错误——“不支持请求方法——405”——开发者在使用 Spring MVC 为特定 HTTP 动词公开 API 时会遇到这个错误。

自然，我们还将讨论这个错误的常见原因。

## 2.请求方法基础

在着手解决常见问题之前，如果您刚刚开始学习 Spring MVC，这里有一篇很好的介绍文章。

让我们快速浏览一下基础知识——理解 Spring 支持的请求方法和一些感兴趣的常见类。

简单来说，MVC HTTP 方法是请求可以在服务器上触发的基本操作。例如，一些方法从服务器`fetch`得到数据，一些方法`submit`得到数据，一些方法可能`delete`得到数据等等。

`[@RequestMapping](/web/20220727020632/https://www.baeldung.com/spring-requestmapping) annotation `指定了请求所支持的方法。

Spring 在一个 enum `RequestMethod`下声明所有支持的请求方法；它指定了标准的`GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE` 动词。

[弹簧`DispatcherServlet`](/web/20220727020632/https://www.baeldung.com/spring-dispatcherservlet) 默认支持除`OPTIONS` 和`TRACE`之外的所有弹簧；`@RequestMapping `使用`RequestMethod enum`来指定支持哪些方法。

## 3。简单的 MVC 场景

现在，让我们来看一个映射所有 HTTP 方法的代码示例:

```java
@RestController
@RequestMapping(value="/api")
public class RequestMethodController {

    @Autowired
    private EmployeeService service;

    @RequestMapping(value = "/employees", produces = "application/json")
    public List<Employee> findEmployees()
      throws InvalidRequestException {
        return service.getEmployeeList();
    }
}
```

注意这个例子是如何声明`findEmployee()`方法的。它没有指定任何特定的请求方法，这意味着该 URL 支持所有默认方法。

我们可以使用不同的支持方法请求 API，例如，使用 curl:

```java
$ curl --request POST http://localhost:8080/api/employees
[{"id":100,"name":"Steve Martin","contactNumber":"333-777-999"},
{"id":200,"name":"Adam Schawn","contactNumber":"444-111-777"}]
```

自然，我们可以通过多种方式发送请求——通过简单的`curl`命令、Postman、AJAX 等。

当然，如果请求被正确映射并且成功，我们希望得到`200 OK`响应。

## 4。问题场景–HTTP 405

但是，我们在这里讨论的是——当然——请求不成功的情况。

`405 Method Not Allowed`'是我们在处理 Spring 请求时观察到的最常见的错误之一。

让我们看看如果我们在 Spring MVC 中专门定义和处理 GET 请求会发生什么，就像这样:

```java
@RequestMapping(
  value = "/employees", 
  produces = "application/json", 
  method = RequestMethod.GET)
public List<Employee> findEmployees() {
    ...
}

// send the PUT request using CURL
$ curl --request PUT http://localhost:8080/api/employees
{"timestamp":1539720588712,"status":405,"error":"Method Not Allowed",
"exception":"org.springframework.web.HttpRequestMethodNotSupportedException",
"message":"Request method 'PUT' not supported","path":"/api/employees"} 
```

## 5.405 不支持–原因、解决方案

在前面的场景中，我们得到的是带有 405 状态代码的 HTTP 响应，这是一个客户端错误，表明服务器不支持请求中发送的方法/动词。

顾名思义，此错误的原因是用不支持的方法发送请求。

如您所料，我们可以通过在现有的方法映射中为 PUT 定义一个显式映射来解决这个问题:

```java
@RequestMapping(
  value = "/employees", 
  produces = "application/json", 
  method = {RequestMethod.GET, RequestMethod.PUT}) ...
```

或者，我们可以单独定义新的方法/映射:

```java
@RequestMapping(value = "/employees", 
  produces = "application/json", 
  method=RequestMethod.PUT)
public List<Employee> postEmployees() ... 
```

## 6.结论

请求方法/动词是 HTTP 通信中的一个关键方面，我们需要注意我们在服务器端定义的操作的确切语义，以及我们发送的确切请求。

和往常一样，本文展示的例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-2)