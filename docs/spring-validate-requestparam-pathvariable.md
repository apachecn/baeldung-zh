# 在 Spring 中验证 RequestParams 和 PathVariables

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-validate-requestparam-pathvariable>

## 1.介绍

在本教程中，我们将看看如何在 Spring MVC 中验证 HTTP 请求参数和路径变量。

具体来说，我们将使用 [JSR 303 注释](https://web.archive.org/web/20220628065527/https://beanvalidation.org/1.0/spec/)来验证`String`和`Number `参数。

要探索其他类型的验证，请参考我们关于 [Java Bean 验证](/web/20220628065527/https://www.baeldung.com/javax-validation)和[方法约束](/web/20220628065527/https://www.baeldung.com/javax-validation-method-constraints)的教程，或者学习如何[创建自己的验证器](/web/20220628065527/https://www.baeldung.com/spring-mvc-custom-validator)。

## 2.配置

要使用 Java 验证 API，我们必须添加一个 JSR 303 实现，如`[hibernate-validator](https://web.archive.org/web/20220628065527/https://search.maven.org/search?q=a:hibernate-validator%20AND%20g:org.hibernate.validator)`:

```java
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.10.Final</version>
</dependency>
```

另外，**我们必须通过添加`@Validated` 注释**来启用控制器中请求参数和路径变量的验证:

```java
@RestController
@RequestMapping("/")
@Validated
public class Controller {
    // ...
}
```

需要注意的是，**启用参数验证还需要一个`MethodValidationPostProcessor` bean** 。如果我们使用的是 Spring Boot 应用程序，那么这个 bean 是自动配置的，因为我们的类路径上有`hibernate-validator`依赖项。

否则，在标准的 Spring 应用程序中，我们必须显式地添加这个 bean:

```java
@EnableWebMvc
@Configuration
@ComponentScan("com.baeldung.spring")
public class ClientWebConfigJava implements WebMvcConfigurer {
    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        return new MethodValidationPostProcessor();
    }
    // ...
}
```

默认情况下，Spring 中路径或请求验证期间的任何错误都会导致 HTTP 500 响应。在本教程中，我们使用一个定制的 [`ControllerAdvice`](/web/20220628065527/https://www.baeldung.com/exception-handling-for-rest-with-spring) 来以一种更可读的方式处理这种错误，并为任何错误的请求返回 HTTP 400。你可以在 [GitHub 找到这个解决方案的源代码。](https://web.archive.org/web/20220628065527/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml)

## 3.验证一个`RequestParam`

让我们考虑一个例子，其中我们将一个数字工作日作为请求参数传递给控制器方法:

```java
@GetMapping("/name-for-day")
public String getNameOfDayByNumber(@RequestParam Integer dayOfWeek) {
    // ...
}
```

我们的目标是确保`dayOfWeek`的值在 1 到 7 之间。为此，我们将使用`@Min`和`@Max`注释:

```java
@GetMapping("/name-for-day")
public String getNameOfDayByNumber(@RequestParam @Min(1) @Max(7) Integer dayOfWeek) {
    // ...
}
```

任何不符合这些条件的请求都将返回 HTTP 状态 400 和默认错误消息。

例如，如果我们调用 [http:// `localhost:8080/name-for-day?dayOfWeek=24`](https://web.archive.org/web/20220628065527/http://localhost:8080/name-for-day?dayOfWeek=24) ，响应消息将是:

```java
getNameOfDayByNumber.dayOfWeek: must be less than or equal to 7
```

我们可以通过添加自定义消息来更改默认消息:

```java
@Max(value = 1, message = “day number has to be less than or equal to 7”)
```

## 4.验证一个`PathVariable`

就像@ `RequestParam,`一样，我们可以使用`javax.validation.constraints`包中的任何注释来验证一个`@PathVariable`。

让我们考虑一个例子，其中我们验证一个字符串参数不为空，并且长度小于或等于 10:

```java
@GetMapping("/valid-name/{name}")
public void createUsername(@PathVariable("name") @NotBlank @Size(max = 10) String username) {
    // ...
}
```

例如，任何带有超过 10 个字符的`name`参数的请求都会导致 HTTP 400 错误，并显示一条消息:

```java
createUser.name:size must be between 0 and 10
```

通过设置`@Size`注释中的`message`参数，可以很容易地覆盖默认消息。

## 5.结论

在本文中，我们学习了如何在 Spring 应用程序中验证请求参数和路径变量。

像往常一样，所有的源代码都可以在 GitHub 上获得。