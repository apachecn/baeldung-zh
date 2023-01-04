# 在 Spring 控制器中验证列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-validate-list-controller>

## 1.介绍

在任何应用程序中，验证用户输入都是一个常见的需求。在本教程中，我们将回顾**验证对象的`List`作为弹簧控制器**的参数的方法。

我们将在控制器层添加验证，以确保用户指定的数据满足指定的条件。

## 2.向 Bean 添加约束

对于我们的例子，我们将使用一个简单的 Spring 控制器来管理电影数据库。我们将重点关注一种方法，该方法接受电影列表，并在对列表执行验证后将它们添加到数据库中。

所以，让我们从使用 [javax 验证](/web/20220628123722/https://www.baeldung.com/javax-validation "Java Bean Validation Basics")为`Movie` bean 添加约束开始:

```
public class Movie {

    private String id;

    @NotEmpty(message = "Movie name cannot be empty.")
    private String name;

    // standard setters and getters
}
```

## 3.在控制器中添加验证注释

让我们看看我们的控制器。首先，我们将**向控制器类**添加`@Validated`注释:

```
@Validated
@RestController
@RequestMapping("/movies")
public class MovieController {

    @Autowired
    private MovieService movieService;

    //...
}
```

接下来，让我们编写控制器方法，其中我们将验证传入的`Movie`对象列表。

我们将**将`@NotEmpty` 注释添加到我们的电影**列表中，以验证列表中至少应该有一个元素。同时，我们将添加`@Valid`注释以确保`Movie`对象本身是有效的:

```
@PostMapping
public void addAll(
  @RequestBody 
  @NotEmpty(message = "Input movie list cannot be empty.")
  List<@Valid Movie> movies) {
    movieService.addAll(movies);
}
```

如果我们用一个空的`Movie`列表输入调用控制器方法，那么验证将会因为`@NotEmpty`注释而失败，我们将会看到消息:

```
Input movie list cannot be empty.
```

`@Valid`注释将确保为列表中的每个对象评估`Movie`类中指定的约束。因此，如果我们在列表中传递一个名字为空的`Movie`，验证将会失败，并显示消息:

```
Movie name cannot be empty.
```

## 4.自定义验证程序

我们还可以将[自定义约束验证器](/web/20220628123722/https://www.baeldung.com/spring-mvc-custom-validator "Spring MVC Custom Validation")添加到输入列表中。

对于我们的示例，自定义约束将验证输入列表大小被限制为最多四个元素的条件。让我们创建这个自定义约束注释:

```
@Constraint(validatedBy = MaxSizeConstraintValidator.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface MaxSizeConstraint {
    String message() default "The input list cannot contain more than 4 movies.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

现在，我们将创建一个应用上述约束的验证器:

```
public class MaxSizeConstraintValidator implements ConstraintValidator<MaxSizeConstraint, List<Movie>> {
    @Override
    public boolean isValid(List<Movie> values, ConstraintValidatorContext context) {
        return values.size() <= 4;
    }
}
```

最后，我们将把`@MaxSizeConstraint` 注释添加到控制器方法中:

```
@PostMapping
public void addAll(
  @RequestBody
  @NotEmpty(message = "Input movie list cannot be empty.")
  @MaxSizeConstraint
  List<@Valid Movie> movies) {
    movieService.addAll(movies);
}
```

这里，`@MaxSizeConstraint`将验证输入的大小。因此，如果我们在输入列表中传递超过四个`Movie`对象，验证将会失败。

## 5.处理异常

如果任何验证失败，就会抛出`[ConstraintViolationException](https://web.archive.org/web/20220628123722/https://javaee.github.io/javaee-spec/javadocs/javax/validation/ConstraintViolationException.html "ConstraintViolationException javadoc")`。现在，让我们看看如何添加一个[异常处理](/web/20220628123722/https://www.baeldung.com/exception-handling-for-rest-with-spring "Error Handling for REST with Spring")组件来捕捉这个异常。

```
@ExceptionHandler(ConstraintViolationException.class)
public ResponseEntity handle(ConstraintViolationException constraintViolationException) {
    Set<ConstraintViolation<?>> violations = constraintViolationException.getConstraintViolations();
    String errorMessage = "";
    if (!violations.isEmpty()) {
        StringBuilder builder = new StringBuilder();
        violations.forEach(violation -> builder.append(" " + violation.getMessage()));
        errorMessage = builder.toString();
    } else {
        errorMessage = "ConstraintViolationException occured.";
    }
    return new ResponseEntity<>(errorMessage, HttpStatus.BAD_REQUEST);
 }
```

## 6.测试 API

现在，我们将使用有效和无效输入来测试控制器。

首先，让我们为 API 提供有效的输入:

```
curl -v -d [{"name":"Movie1"}] -H "Content-Type: application/json" -X POST http://localhost:8080/movies
```

在这个场景中，我们将得到一个 HTTP 状态 200 响应:

```
...
HTTP/1.1 200
...
```

接下来，当我们传递无效输入时，我们将检查我们的 API 响应。

让我们尝试一个空列表:

```
curl -d [] -H "Content-Type: application/json" -X POST http://localhost:8080/movies
```

在这个场景中，我们将得到一个 HTTP 状态 400 响应。这是因为输入不满足`@NotEmpty`约束。

```
Input movie list cannot be empty.
```

接下来，让我们尝试传递列表中的五个`Movie`对象:

```
curl -d [{"name":"Movie1"},{"name":"Movie2"},{"name":"Movie3"},{"name":"Movie4"},{"name":"Movie5"}] 
  -H "Content-Type: application/json" -X POST http://localhost:8080/movies
```

这也将导致 HTTP status 400 响应，因为我们没有通过`@MaxSizeConstraint` 约束:

```
The input list cannot contain more than 4 movies.
```

## 7.结论

在这篇简短的文章中，我们学习了如何在 Spring 中验证对象列表。

和往常一样，例子的完整源代码在 GitHub 上的[。](https://web.archive.org/web/20220628123722/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-4)