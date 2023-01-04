# Spring @Controller 和@RestController 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-controller-vs-restcontroller>

## 1.概观

在这个简短的教程中，我们将讨论 Spring MVC 中`@Controller`和`@RestController` 注释的区别。

我们可以将第一个注释用于传统的 Spring 控制器，它已经成为框架的一部分很长时间了。

为了简化 RESTful web 服务的创建，Spring 4.0 引入了`@RestController`注释。**是一个结合了`@Controller`和`@ResponseBody`的方便的注释，它消除了用`@ResponseBody`注释来注释控制器类的每个请求处理方法的需要。**

## 延伸阅读:

## [春季请求映射](/web/20220625081626/https://www.baeldung.com/spring-requestmapping)

Spring @RequestMapping - Basic Example, @RequestParam, @PathVariable, Header mapping[Read more](/web/20220625081626/https://www.baeldung.com/spring-requestmapping) →

## [Spring @RequestParam 注释](/web/20220625081626/https://www.baeldung.com/spring-request-param)

A detailed guide to Spring's @RequestParam annotation[Read more](/web/20220625081626/https://www.baeldung.com/spring-request-param) →

## 2.Spring MVC `@Controller`

我们可以用`@Controller`注释来注释传统的控制器。这只是对`@Component`类的专门化，它允许我们通过类路径扫描自动检测实现类。

对于请求处理方法，我们通常结合使用` @Controller`和`@RequestMapping`注释。

让我们看一个 Spring MVC 控制器的简单例子:

```java
@Controller
@RequestMapping("books")
public class SimpleBookController {

    @GetMapping("/{id}", produces = "application/json")
    public @ResponseBody Book getBook(@PathVariable int id) {
        return findBookById(id);
    }

    private Book findBookById(int id) {
        // ...
    }
} 
```

我们用`@ResponseBody`注释了请求处理方法。该注释支持将返回对象自动序列化到`HttpResponse`中。

## 3.Spring MVC `@RestController`

`@RestController`是控制器的专用版本。它包括`@Controller`和`@ResponseBody`注释，因此简化了控制器的实现:

```java
@RestController
@RequestMapping("books-rest")
public class SimpleBookRestController {

    @GetMapping("/{id}", produces = "application/json")
    public Book getBook(@PathVariable int id) {
        return findBookById(id);
    }

    private Book findBookById(int id) {
        // ...
    }
} 
```

**控制器用`@RestController`标注进行标注；因此，不需要`@ResponseBody`。**

控制器类的每个请求处理方法都会自动将返回对象序列化到`HttpResponse`中。

## 4.结论

在本文中，我们研究了 Spring 框架中可用的经典和专用 REST 控制器。

GitHub 项目中提供了示例的完整源代码。这是一个 Maven 项目，因此可以导入并按原样使用。