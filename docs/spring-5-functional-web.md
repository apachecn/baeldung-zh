# Spring 5 中的功能性 Web 框架简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-5-functional-web>

## 1。简介

Spring WebFlux 是一个新的功能性 web 框架，使用反应原理构建。

在本教程中，我们将学习如何在实践中使用它。

我们将基于现有的 Spring 5 WebFlux 的[指南。在该指南中，我们使用基于注释的组件创建了一个简单的反应式 REST 应用程序。这里，我们将使用函数框架。](/web/20220701022854/https://www.baeldung.com/spring-webflux)

## 2。Maven 依赖关系

我们将需要与上一篇文章中定义的相同的`[spring-boot-starter-webflux](https://web.archive.org/web/20220701022854/https://search.maven.org/classic#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-webflux%22%20AND%20g%3A%22org.springframework.boot%22)`依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
    <version>2.6.4</version>
</dependency>
```

## 3。功能性网络框架

功能性 web 框架引入了一种新的编程模型，在这种模型中，我们使用函数来路由和处理请求。

与我们使用注释映射的基于注释的模型相反，这里我们将使用 [`HandlerFunction`](https://web.archive.org/web/20220701022854/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/server/HandlerFunction.html) 和`[RouterFunction](https://web.archive.org/web/20220701022854/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/server/RouterFunction.html)` [s](https://web.archive.org/web/20220701022854/https://docs.spring.io/spring/docs/5.0.0.M5/javadoc-api/org/springframework/web/reactive/function/server/RouterFunction.html) 。

类似地，在带注释的控制器中，功能端点方法建立在相同的反应堆栈上。

### T2`3.1\. HandlerFunction`

`HandlerFunction`代表一个为发送给它们的请求生成响应的函数:

```java
@FunctionalInterface
public interface HandlerFunction<T extends ServerResponse> {
    Mono<T> handle(ServerRequest request);
}
```

这个接口主要是一个`Function<Request, Response<T>>`，它的行为非常像一个 servlet。

虽然，与标准的`Servlet#service(ServletRequest req, ServletResponse res)`，`HandlerFunction`不把响应作为输入参数。

### 3.2。`RouterFunction`

`RouterFunction`作为`@RequestMapping`注释的替代。我们可以使用它将请求路由到处理函数:

```java
@FunctionalInterface
public interface RouterFunction<T extends ServerResponse> {
    Mono<HandlerFunction<T>> route(ServerRequest request);
    // ...
}
```

通常，我们可以导入助手函数`[RouterFunctions.route()](https://web.archive.org/web/20220701022854/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/server/RouterFunctions.html#route-org.springframework.web.reactive.function.server.RequestPredicate-org.springframework.web.reactive.function.server.HandlerFunction-)`来创建路由，而不是编写一个完整的路由器函数。

当谓词匹配时，它允许我们通过应用一个`RequestPredicate.` 来路由请求，然后返回第二个参数，即处理函数:

```java
public static <T extends ServerResponse> RouterFunction<T> route(
  RequestPredicate predicate,
  HandlerFunction<T> handlerFunction)
```

因为`route()`方法返回一个`RouterFunction`，所以我们可以链接它来构建强大而复杂的路由方案。

## 4.使用功能 Web 的反应式 REST 应用程序

在我们之前的指南中，我们使用`@RestController` 和`WebClient.` 创建了一个简单的`EmployeeManagement` REST 应用程序

现在，让我们使用路由器和处理器函数实现相同的逻辑。

首先，**我们需要使用`RouterFunction`来创建路由，以发布和消费我们的`Employee` s `.`** 的反应流

路由注册为 Spring beans，可以在任何配置类中创建。

### 4.1.单一资源

让我们使用发布单个`Employee`资源的`RouterFunction` 创建第一个路由:

```java
@Bean
RouterFunction<ServerResponse> getEmployeeByIdRoute() {
  return route(GET("/employees/{id}"), 
    req -> ok().body(
      employeeRepository().findEmployeeById(req.pathVariable("id")), Employee.class));
}
```

第一个参数是请求谓词。注意我们在这里是如何使用静态导入的`RequestPredicates.GET`方法的。第二个参数定义了一个处理函数，如果谓词适用，将使用该函数。

换句话说，上面的例子将所有对`/employees/{id}`的 GET 请求路由到`EmployeeRepository#findEmployeeById(String id)`方法。

### 4.2.馆藏资源

接下来，为了发布集合资源，让我们添加另一个路由:

```java
@Bean
RouterFunction<ServerResponse> getAllEmployeesRoute() {
  return route(GET("/employees"), 
    req -> ok().body(
      employeeRepository().findAllEmployees(), Employee.class));
}
```

### 4.3.单一资源更新

最后，让我们添加一个更新`Employee` 资源的路径:

```java
@Bean
RouterFunction<ServerResponse> updateEmployeeRoute() {
  return route(POST("/employees/update"), 
    req -> req.body(toMono(Employee.class))
      .doOnNext(employeeRepository()::updateEmployee)
      .then(ok().build()));
}
```

## 5.构成路线

我们还可以在单个路由器功能中将路由组合在一起。

让我们看看如何组合上面创建的路线:

```java
@Bean
RouterFunction<ServerResponse> composedRoutes() {
  return 
    route(GET("/employees"), 
      req -> ok().body(
        employeeRepository().findAllEmployees(), Employee.class))

    .and(route(GET("/employees/{id}"), 
      req -> ok().body(
        employeeRepository().findEmployeeById(req.pathVariable("id")), Employee.class)))

    .and(route(POST("/employees/update"), 
      req -> req.body(toMono(Employee.class))
        .doOnNext(employeeRepository()::updateEmployee)
        .then(ok().build())));
}
```

这里，我们使用了 [`RouterFunction.and()`](https://web.archive.org/web/20220701022854/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/server/RouterFunction.html#and-org.springframework.web.reactive.function.server.RouterFunction-) 来组合我们的路线。

最后，我们使用路由器和处理程序实现了我们的`EmployeeManagement` 应用程序所需的完整 REST API。

要运行这个应用程序，我们可以使用单独的路由，也可以使用我们上面创建的单一的组合路由。

## 6.测试路线

**我们可以使用`WebTestClient` 来测试我们的路线。**

为此，我们首先需要使用`bindToRouterFunction`方法绑定路由，然后构建测试客户端实例。

让我们来测试一下我们的`getEmployeeByIdRoute`:

```java
@Test
void givenEmployeeId_whenGetEmployeeById_thenCorrectEmployee() {
    WebTestClient client = WebTestClient
      .bindToRouterFunction(config.getEmployeeByIdRoute())
      .build();

    Employee employee = new Employee("1", "Employee 1");

    given(employeeRepository.findEmployeeById("1")).willReturn(Mono.just(employee));

    client.get()
      .uri("/employees/1")
      .exchange()
      .expectStatus()
      .isOk()
      .expectBody(Employee.class)
      .isEqualTo(employee);
}
```

类似地`getAllEmployeesRoute`:

```java
@Test
void whenGetAllEmployees_thenCorrectEmployees() {
    WebTestClient client = WebTestClient
      .bindToRouterFunction(config.getAllEmployeesRoute())
      .build();

    List<Employee> employees = Arrays.asList(
      new Employee("1", "Employee 1"),
      new Employee("2", "Employee 2"));

    Flux<Employee> employeeFlux = Flux.fromIterable(employees);
    given(employeeRepository.findAllEmployees()).willReturn(employeeFlux);

    client.get()
      .uri("/employees")
      .exchange()
      .expectStatus()
      .isOk()
      .expectBodyList(Employee.class)
      .isEqualTo(employees);
}
```

我们还可以通过断言我们的`Employee` 实例通过`EmployeeRepository`更新来测试我们的`updateEmployeeRoute`:

```java
@Test
void whenUpdateEmployee_thenEmployeeUpdated() {
    WebTestClient client = WebTestClient
      .bindToRouterFunction(config.updateEmployeeRoute())
      .build();

    Employee employee = new Employee("1", "Employee 1 Updated");

    client.post()
      .uri("/employees/update")
      .body(Mono.just(employee), Employee.class)
      .exchange()
      .expectStatus()
      .isOk();

    verify(employeeRepository).updateEmployee(employee);
}
```

关于使用`WebTestClient`进行测试的更多细节，请参考我们关于[使用`WebClient`和`WebTestClient`](/web/20220701022854/https://www.baeldung.com/spring-5-webclient) 的教程。

## 7 .**。总结**

在本教程中，我们介绍了 Spring 5 中新的功能性 web 框架，并研究了它的两个核心接口——`RouterFunction` 和`HandlerFunction.`,我们还学习了如何创建各种路由来处理请求和发送响应。

此外，我们用功能端点模型重新创建了 Spring 5 WebFlux 指南中介绍的`EmployeeManagement` 应用程序。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220701022854/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-reactive)