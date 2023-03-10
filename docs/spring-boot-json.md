# Spring Boot 消费和生产 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-json>

## 1。概述

在本教程中，我们将演示**如何构建一个 [REST 服务](/web/20220627171303/https://www.baeldung.com/rest-with-spring-series)来使用 Spring Boot** 消费和生产 JSON 内容。

我们还将看看如何轻松地使用 RESTful HTTP 语义。

为了简单起见，我们不会在[中包含一个持久层](/web/20220627171303/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)，但是 [Spring Data](/web/20220627171303/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 也使得这个很容易添加。

## 2。休息服务

在 Spring Boot 编写 JSON REST 服务很简单，因为当 [Jackson](/web/20220627171303/https://www.baeldung.com/jackson) 在类路径中时，这是它的默认观点:

```java
@RestController
@RequestMapping("/students")
public class StudentController {

    @Autowired
    private StudentService service;

    @GetMapping("/{id}")
    public Student read(@PathVariable String id) {
        return service.find(id);
    }

... 
```

通过用 [`@RestController`](/web/20220627171303/https://www.baeldung.com/spring-controller-vs-restcontroller) ，**注释我们的`StudentController `，我们已经告诉 Spring Boot 将`read`方法**的返回类型写到响应体。**因为我们在类级别**也有一个`@RequestMapping`，对于我们添加的任何更多的公共方法也是一样的。

虽然简单，但是这种方法缺乏 HTTP 语义。例如，如果我们找不到请求的学生，会发生什么？我们可能希望返回 404，而不是返回 200 或 500 状态代码。

让我们看看如何对 HTTP 响应本身进行更多的控制，并进而向我们的控制器添加一些典型的 RESTful 行为。

## 3。创建

当我们需要控制响应的各个方面而不是主体时，比如状态代码，我们可以返回一个`ResponseEntity`:

```java
@PostMapping("/")
public ResponseEntity<Student> create(@RequestBody Student student) 
    throws URISyntaxException {
    Student createdStudent = service.create(student);
    if (createdStudent == null) {
        return ResponseEntity.notFound().build();
    } else {
        URI uri = ServletUriComponentsBuilder.fromCurrentRequest()
          .path("/{id}")
          .buildAndExpand(createdStudent.getId())
          .toUri();

        return ResponseEntity.created(uri)
          .body(createdStudent);
    }
} 
```

这里我们所做的不仅仅是在响应中返回创建的`Student`。**我们也用一个语义清晰的 HTTP 状态来响应，如果创建成功，就给新资源一个 URI。**

## 4。阅读

如前所述，如果我们想读取一个单独的`Student`，如果我们找不到学生，返回 404 在语义上更清楚:

```java
@GetMapping("/{id}")
public ResponseEntity<Student> read(@PathVariable("id") Long id) {
    Student foundStudent = service.read(id);
    if (foundStudent == null) {
        return ResponseEntity.notFound().build();
    } else {
        return ResponseEntity.ok(foundStudent);
    }
} 
```

这里我们可以清楚地看到与我们最初的`read()`实现的不同。

**这样，`Student`对象将被正确地映射到响应体，同时返回一个正确的状态。**

## 5。更新

更新与创建非常相似，只是它被映射到 PUT 而不是 POST，并且 URI 包含我们正在更新的资源的`id`:

```java
@PutMapping("/{id}")
public ResponseEntity<Student> update(@RequestBody Student student, @PathVariable Long id) {
    Student updatedStudent = service.update(id, student);
    if (updatedStudent == null) {
        return ResponseEntity.notFound().build();
    } else {
        return ResponseEntity.ok(updatedStudent);
    }
} 
```

## 6。删除

删除操作映射到 delete 方法。URI 还包含资源的`id`:

```java
@DeleteMapping("/{id}")
public ResponseEntity<Object> deleteStudent(@PathVariable Long id) {
    service.delete(id);
    return ResponseEntity.noContent().build();
} 
```

我们没有[实现特定的错误处理](/web/20220627171303/https://www.baeldung.com/exception-handling-for-rest-with-spring)，因为`delete()`方法实际上通过抛出`Exception.`而失败

## 7。结论

在本文中，我们学习了如何在 Spring Boot 开发的典型 CRUD REST 服务中消费和生产 JSON 内容。此外，我们还演示了如何实现正确的响应状态控制和错误处理。

为了简单起见，我们这次没有讨论持久性，但是 [Spring Data REST](/web/20220627171303/https://www.baeldung.com/spring-data-rest-intro) 提供了一种快速有效的方法来构建 RESTful 数据服务。

这个例子的完整源代码在 GitHub 上可以找到。