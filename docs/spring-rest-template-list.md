# 用 RestTemplate 获取和发布对象列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-template-list>

## 1.介绍

`RestTemplate`类是 Spring 中执行客户端 HTTP 操作的核心工具。它为构建 HTTP 请求和处理响应提供了几个实用方法。

由于`RestTemplate`与 [Jackson、](https://web.archive.org/web/20221212032415/https://github.com/FasterXML/jackson)集成得很好，它可以不费吹灰之力地在 JSON 中序列化/反序列化大多数对象。然而，**处理对象集合并不是那么简单**。

在本教程中，我们将学习如何使用`RestTemplate`到`GET`和`POST`对象列表。

## 延伸阅读:

## [Spring RestTemplate 错误处理](/web/20221212032415/https://www.baeldung.com/spring-rest-template-error-handling)

Learn how to handle errors with Spring's RestTemplate[Read more](/web/20221212032415/https://www.baeldung.com/spring-rest-template-error-handling) →

## [使用 JSON 的 RestTemplate Post 请求](/web/20221212032415/https://www.baeldung.com/spring-resttemplate-post-json)

Learn how to use Spring's RestTemplate to send requests with JSON content.[Read more](/web/20221212032415/https://www.baeldung.com/spring-resttemplate-post-json) →

## 2.示例服务

我们将使用一个员工 API，它有两个 HTTP 端点，get all 和 create:

*   `GET /employees`
*   `POST /employees`

对于客户机和服务器之间的通信，我们将使用一个简单的 DTO 来封装基本的雇员数据:

```java
public class Employee {
    public long id;
    public String title;

    // standard constructor and setters/getters
}
```

现在我们准备编写代码，使用`RestTemplate`来获取和创建`Employee`对象的列表。

## 3.用`RestTemplate`获取对象列表

通常在调用 GET 时，我们可以使用`RestTemplate`中的一个简化方法，比如`:`

`getForObject(URI url, Class<T> responseType)`

这将使用 GET 动词向指定的 URI 发送请求，并将响应主体转换为请求的 Java 类型。这对大多数类来说都很有效，但是它有一个限制；我们无法发送对象列表。

该问题是由于 Java 泛型的类型擦除造成的。当应用程序运行时，它不知道列表中是什么类型的对象。这意味着列表中的数据不能被反序列化成合适的类型。

幸运的是，我们有两个选择来解决这个问题。

### 3.1.使用数组

首先，我们可以使用`RestTemplate.` `getForEntity()` 通过`responseType`参数获得一个对象数组。无论我们在那里指定什么`class` ，都将匹配`ResponseEntity`的参数类型:

```java
ResponseEntity<Employee[]> response =
  restTemplate.getForEntity(
  "http://localhost:8080/employees/",
  Employee[].class);
Employee[] employees = response.getBody();
```

我们也可以使用`RestTemplate.exchange` 来获得相同的结果。

注意，在这里承担重任的合作者是`ResponseExtractor, `，所以如果我们需要进一步定制，我们可以调用`execute` 并提供我们自己的实例。

### 3.2.使用包装类

有些 API 会返回一个包含员工列表的顶级对象，而不是直接返回列表。为了处理这种情况，我们可以使用一个包含雇员列表的包装类。

```java
public class EmployeeList {
    private List<Employee> employees;

    public EmployeeList() {
        employees = new ArrayList<>();
    }

    // standard constructor and getter/setter
}
```

现在我们可以使用更简单的`getForObject()`方法来获得雇员列表:

```java
EmployeeList response = restTemplate.getForObject(
  "http://localhost:8080/employees",
  EmployeeList.class);
List<Employee> employees = response.getEmployees();
```

这段代码要简单得多，但是需要一个额外的包装对象。

## 4.张贴带有`RestTemplate`的对象列表

现在让我们看看如何将对象列表从我们的客户机发送到服务器。和上面一样，`RestTemplate`提供了一个调用 POST 的简化方法:

`postForObject(URI url, Object request, Class<T> responseType)`

这会向给定的 URI 发送一个 HTTP POST，带有可选的请求体，并将响应转换为指定的类型。与上面的 GET 场景不同，**我们不必担心类型删除**。

这是因为现在我们正从 Java 对象转向 JSON。JVM 知道对象列表及其类型，所以它们将被正确地序列化:

```java
List<Employee> newEmployees = new ArrayList<>();
newEmployees.add(new Employee(3, "Intern"));
newEmployees.add(new Employee(4, "CEO"));

restTemplate.postForObject(
  "http://localhost:8080/employees/",
  newEmployees,
  ResponseEntity.class);
```

### 4.1.使用包装类

如果我们需要使用一个包装类来与上面的 GET 场景保持一致，那也很简单。我们可以使用`RestTemplate`发送新列表:

```java
List<Employee> newEmployees = new ArrayList<>();
newEmployees.add(new Employee(3, "Intern"));
newEmployees.add(new Employee(4, "CEO"));

restTemplate.postForObject(
  "http://localhost:8080/employees",
  new EmployeeList(newEmployees),
  ResponseEntity.class);
```

## 5.结论

使用`RestTemplate`是构建 HTTP 客户端与我们的服务通信的一种简单方式。

它提供了许多处理每个 HTTP 方法和简单对象的方法。通过一点额外的代码，我们可以很容易地用它来处理对象列表。

像往常一样，完整的代码可以在 [Github 项目](https://web.archive.org/web/20221212032415/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-3)中获得。