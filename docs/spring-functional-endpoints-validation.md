# Spring 5 中功能端点的验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-functional-endpoints-validation>

## 1.概观

为我们的 API 实现输入验证通常是有用的，这样可以避免以后在处理数据时出现意外错误。

不幸的是，在 Spring 5 中，没有办法像我们在基于注释的端点上那样在功能端点上自动运行验证。我们必须手动管理它们。

尽管如此，我们可以利用 Spring 提供的一些有用的工具来简单明了地验证我们的资源是有效的。

## 2.使用 Spring 验证

在开始实际验证之前，让我们先用一个工作功能端点来配置我们的项目。

想象我们有以下的`RouterFunction`:

```java
@Bean
public RouterFunction<ServerResponse> functionalRoute(
  FunctionalHandler handler) {
    return RouterFunctions.route(
      RequestPredicates.POST("/functional-endpoint"),
      handler::handleRequest);
}
```

该路由器使用下列控制器类提供的处理程序功能:

```java
@Component
public class FunctionalHandler {

    public Mono<ServerResponse> handleRequest(ServerRequest request) {
        Mono<String> responseBody = request
          .bodyToMono(CustomRequestEntity.class)
          .map(cre -> String.format(
            "Hi, %s [%s]!", cre.getName(), cre.getCode()));

        return ServerResponse.ok()
          .contentType(MediaType.APPLICATION_JSON)
          .body(responseBody, String.class);
    }
}
```

正如我们所看到的，我们在这个功能端点中所做的只是格式化和检索我们在请求体中接收到的信息，该请求体被构造为一个`CustomRequestEntity`对象:

```java
public class CustomRequestEntity {

    private String name;
    private String code;

    // ... Constructors, Getters and Setters ...

}
```

这样做很好，但是让我们想象一下，我们现在需要检查我们的输入是否符合一些给定的约束，例如，没有一个字段可以为空，并且代码应该超过 6 位。

我们需要找到一种方法来有效地做出这些断言，如果可能的话，从我们的业务逻辑中分离出来。

### 2.1.实现验证器

**正如在[这个 Spring 参考文档](https://web.archive.org/web/20221208143919/https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/core.html#validator)中所解释的，我们可以使用 Spring 的`Validator`接口来评估我们资源的价值**:

```java
public class CustomRequestEntityValidator 
  implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return CustomRequestEntity.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(
          errors, "name", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(
          errors, "code", "field.required");
        CustomRequestEntity request = (CustomRequestEntity) target;
        if (request.getCode() != null && request.getCode().trim().length() < 6) {
            errors.rejectValue(
              "code",
              "field.min.length",
              new Object[] { Integer.valueOf(6) },
              "The code must be at least [6] characters in length.");
        }
    }
}
```

我们不会详细讨论`Validator` 是如何工作的。当验证一个对象时，知道所有的错误都被收集就足够了——**一个空的错误收集意味着这个对象遵守我们所有的约束**。

所以现在我们已经有了我们的`Validator`,我们必须在实际执行我们的业务逻辑之前显式地调用它的`validate `。

### 2.2.执行验证

首先，我们可以认为在我们的情况下使用`[HandlerFilterFunction](/web/20221208143919/https://www.baeldung.com/spring-webflux-filters)`是合适的。

但是我们必须记住，在那些过滤器中——和在处理程序中一样——我们处理[异步结构](/web/20221208143919/https://www.baeldung.com/spring-webflux)—比如`Mono`和`Flux`。

这意味着我们可以访问`Publisher` (`Mono`或`Flux`对象)，但不能访问它最终提供的数据。

因此，我们能做的最好的事情就是当我们在处理函数中实际处理它的时候验证主体。

让我们继续修改我们的处理程序方法，包括验证逻辑:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    Validator validator = new CustomRequestEntityValidator();
    Mono<String> responseBody = request
      .bodyToMono(CustomRequestEntity.class)
      .map(body -> {
        Errors errors = new BeanPropertyBindingResult(
          body,
          CustomRequestEntity.class.getName());
        validator.validate(body, errors);

        if (errors == null || errors.getAllErrors().isEmpty()) {
            return String.format("Hi, %s [%s]!", body.getName(), body.getCode());
        } else {
            throw new ResponseStatusException(
              HttpStatus.BAD_REQUEST,
              errors.getAllErrors().toString());
        }
    });
    return ServerResponse.ok()
      .contentType(MediaType.APPLICATION_JSON)
      .body(responseBody, String.class);
}
```

简而言之，如果请求的主体不符合我们的限制，我们的服务现在将检索一个'`Bad Request`'响应。

我们能说我们达到了目标吗？嗯，我们快到了。我们正在进行验证，但是这种方法有很多缺点。

我们将验证与业务逻辑混合在一起，更糟糕的是，我们将不得不在任何想要进行输入验证的处理程序中重复上面的代码。

让我们努力改善这一点。

## 3.干法工作

为了创建一个更简洁的解决方案，我们将从声明一个抽象类开始，这个抽象类包含处理请求的基本过程。

所有需要输入验证的处理程序都将扩展这个抽象类，以便重用它的主方案，因此遵循 DRY(不要重复自己)原则。

我们将使用泛型，以使它足够灵活来支持任何主体类型及其各自的验证器:

```java
public abstract class AbstractValidationHandler<T, U extends Validator> {

    private final Class<T> validationClass;

    private final U validator;

    protected AbstractValidationHandler(Class<T> clazz, U validator) {
        this.validationClass = clazz;
        this.validator = validator;
    }

    public final Mono<ServerResponse> handleRequest(final ServerRequest request) {
        // ...here we will validate and process the request...
    }
}
```

现在让我们用标准过程来编码我们的`handleRequest`方法:

```java
public Mono<ServerResponse> handleRequest(final ServerRequest request) {
    return request.bodyToMono(this.validationClass)
      .flatMap(body -> {
        Errors errors = new BeanPropertyBindingResult(
          body,
          this.validationClass.getName());
        this.validator.validate(body, errors);

        if (errors == null || errors.getAllErrors().isEmpty()) {
            return processBody(body, request);
        } else {
            return onValidationErrors(errors, body, request);
        }
    });
}
```

正如我们所看到的，我们正在使用两个我们还没有创建的方法。

让我们先定义当我们有验证错误时调用的那个:

```java
protected Mono<ServerResponse> onValidationErrors(
  Errors errors,
  T invalidBody,
  ServerRequest request) {
    throw new ResponseStatusException(
      HttpStatus.BAD_REQUEST,
      errors.getAllErrors().toString());
}
```

虽然这只是一个默认的实现，但是它可以很容易地被子类覆盖。

**最后，我们将设置未定义的`processBody`方法——我们将让子类来决定在这种情况下如何进行**:

```java
abstract protected Mono<ServerResponse> processBody(
  T validBody,
  ServerRequest originalRequest);
```

这门课要分析几个方面。

首先，通过使用泛型，子实现将不得不显式声明它们所期望的内容类型以及将用于评估它的验证器。

这也使得我们的结构健壮，因为它限制了我们方法的签名。

在运行时，构造函数将分配实际的验证器对象和用于转换请求体的类。

我们可以在这里看一下完整的类[。](https://web.archive.org/web/20221208143919/https://github.com/eugenp/tutorials/blob/master/spring-reactive-modules/spring-5-reactive-2/src/main/java/com/baeldung/validations/functional/handlers/AbstractValidationHandler.java)

现在让我们看看如何从这种结构中获益。

### 3.1.调整我们的处理程序

显然，我们要做的第一件事是从这个抽象类中扩展我们的处理程序。

通过这样做，**我们将被迫使用父类的构造函数，并定义如何在`processBody` 方法**中处理我们的请求:

```java
@Component
public class FunctionalHandler
  extends AbstractValidationHandler<CustomRequestEntity, CustomRequestEntityValidator> {

    private CustomRequestEntityValidationHandler() {
        super(CustomRequestEntity.class, new CustomRequestEntityValidator());
    }

    @Override
    protected Mono<ServerResponse> processBody(
      CustomRequestEntity validBody,
      ServerRequest originalRequest) {
        String responseBody = String.format(
          "Hi, %s [%s]!",
          validBody.getName(),
          validBody.getCode());
        return ServerResponse.ok()
          .contentType(MediaType.APPLICATION_JSON)
          .body(Mono.just(responseBody), String.class);
    }
}
```

正如我们可以理解的，我们的子处理程序现在比我们在上一节中获得的要简单得多，因为它避免了对资源的实际验证。

## 4.支持 Bean 验证 API 注释

**通过这种方法，我们还可以利用`javax.validation`包提供的强大的 [Bean 验证的注释](/web/20221208143919/https://www.baeldung.com/javax-validation#validation)。**

例如，让我们定义一个带有注释字段的新实体:

```java
public class AnnotatedRequestEntity {

    @NotNull
    private String user;

    @NotNull
    @Size(min = 4, max = 7)
    private String password;

    // ... Constructors, Getters and Setters ...
}
```

**我们现在可以简单地创建一个新的处理器，注入由`LocalValidatorFactoryBean` bean** 提供的默认弹簧`Validator`:

```java
public class AnnotatedRequestEntityValidationHandler
  extends AbstractValidationHandler<AnnotatedRequestEntity, Validator> {

    private AnnotatedRequestEntityValidationHandler(@Autowired Validator validator) {
        super(AnnotatedRequestEntity.class, validator);
    }

    @Override
    protected Mono<ServerResponse> processBody(
      AnnotatedRequestEntity validBody,
      ServerRequest originalRequest) {

        // ...

    }
}
```

我们必须记住，如果上下文中存在其他的`Validator` beans，我们可能必须使用`@Primary`注释显式声明这个 bean:

```java
@Bean
@Primary
public Validator springValidator() {
    return new LocalValidatorFactoryBean();
}
```

## 5.结论

总之，在这篇文章中，我们学习了如何在 Spring 5 功能端点中验证输入数据。

我们创建了一个很好的方法，通过避免将其逻辑与业务逻辑混合来优雅地处理验证。

当然，建议的解决方案可能不适合任何场景。我们将不得不分析我们的情况，并可能调整结构以适应我们的需要。

如果我们想看完整的工作示例，我们可以在 GitHub repo 中找到它。