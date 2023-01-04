# Spring 中的接口驱动控制器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-interface-driven-controllers>

## 1.介绍

在本教程中，我们考虑 Spring MVC 的一个新特性，它允许我们使用普通的 Java 接口来指定 web 请求。

## 2.概观

通常，在 Spring MVC 中定义控制器时，我们用各种指定请求的注释来修饰它的方法:端点的 URL、HTTP 请求方法、路径变量等等。

例如，我们可以在一个简单的方法上使用所述注释引入`/save/{id} `端点:

```
@PostMapping("/save/{id}")
@ResponseBody
public Book save(@RequestBody Book book, @PathVariable int id) {
    // implementation
}
```

自然，当我们只有一个控制器来处理请求时，这根本不是问题。当我们有不同的具有相同方法签名的控制器时，情况会有所改变。

例如，由于迁移或类似原因，我们可能有两个不同版本的控制器，它们具有相同的方法签名。在这种情况下，我们会有大量伴随方法定义的重复注释。显然，这将违反干燥(`don't repeat yourself`)原则。

如果纯 Java 类会出现这种情况，我们只需定义一个接口，并让这些类实现这个接口。在控制器中，方法的主要负担不是由于方法签名，而是由于方法注释。

然而，Spring 5.1 引入了一个新的[特性:](https://web.archive.org/web/20220628093309/https://github.com/spring-projects/spring-framework/wiki/What%27s-New-in-Spring-Framework-5.x#general-web-revision-1)

> 在接口上也可以检测到控制器参数注释:允许在控制器接口中完成映射契约。

让我们研究一下如何使用这个特性。

## 3.控制器接口

### 3.1.上下文设置

我们通过一个非常简单的管理书籍的 REST 应用程序的例子来说明这个新特性。它将只由一个控制器和允许我们检索和修改书籍的方法组成。

在本教程中，我们只关注与该功能相关的问题。应用程序的所有实现问题都可以在我们的 [GitHub 库](https://web.archive.org/web/20220628093309/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-5-mvc)中找到。

### 3.2.连接

让我们定义一个普通的 Java 接口，其中我们不仅定义了方法的签名，还定义了它们应该处理的 web 请求的类型:

```
@RequestMapping("/default")
public interface BookOperations {

    @GetMapping("/")
    List<Book> getAll();

    @GetMapping("/{id}")
    Optional<Book> getById(@PathVariable int id);

    @PostMapping("/save/{id}")
    public void save(@RequestBody Book book, @PathVariable int id);
}
```

注意，我们可能有类级别的注释，也可能有方法级别的注释。现在，我们可以创建一个实现该接口的控制器:

```
@RestController
@RequestMapping("/book")
public class BookController implements BookOperations {

    @Override
    public List<Book> getAll() {...}

    @Override
    public Optional<Book> getById(int id) {...}

    @Override
    public void save(Book book, int id) {...}

}
```

**我们仍然应该将类级注释`@RestController`或`@Controller`添加到控制器中。**以这种方式定义，控制器继承所有与映射 web 请求相关的注释。

为了检查控制器现在是否按预期工作，让我们运行应用程序并通过发出相应的请求来点击`getAll()`方法:

```
curl http://localhost:8081/book/
```

尽管控制器实现了接口，但我们可以通过添加 web 请求注释来进一步微调它。我们可以像对接口那样做:要么在类级别，要么在方法级别。事实上，我们在定义控制器时已经使用了这种可能性:

```
@RequestMapping("/book")
public class BookController implements BookOperations {...}
```

如果我们向控制器添加 web 请求注释，它们将优先于界面的注释。换句话说， **Spring 解释控制器接口的方式类似于 Java 处理继承的方式。**

我们在界面中定义了所有常见的 web 请求属性，但是在控制器中，我们可能总是会对它们进行微调。

### 3.3.注意事项

当我们有一个接口和实现它的各种控制器时，我们可能会遇到一个 web 请求可能被多种方法处理的情况。自然，Spring 会抛出一个异常:

```
Caused by: java.lang.IllegalStateException: Ambiguous mapping.
```

如果我们用`@RequestMapping`来修饰控制器，我们可以减少不明确映射的风险。

## 4.结论

在本教程中，我们考虑了 Spring 5.1 中引入的一个新特性。现在，当 Spring MVC 控制器实现一个接口时，它们不仅以标准的 Java 方式实现，还继承了接口中定义的所有 web 请求相关的功能。

和往常一样，我们可能会在我们的 [GitHub 库](https://web.archive.org/web/20220628093309/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-5-mvc)上找到相应的代码片段。