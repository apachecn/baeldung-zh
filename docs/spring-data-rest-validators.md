# Spring 数据休息验证器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-validators>

## 1。概述

本文涵盖了 Spring 数据休息验证器的基本介绍。如果您需要先复习 Spring Data REST 的基础知识，一定要访问[这篇文章](/web/20221023133213/https://www.baeldung.com/spring-data-rest-intro)来温习基础知识。

简单地说，使用 Spring Data REST，我们可以简单地通过 REST API 向数据库中添加一个新条目，但是我们当然也需要在实际保存数据之前确保数据是有效的。

这篇文章是现有文章的继续，我们将重用我们在那里建立的现有项目。

而且，如果你想首先**开始使用 Spring Data REST**——这里有一个很好的方法让你立即投入使用:

## 2。使用`Validators`

从 Spring 3 开始，该框架以`Validator`接口为特色——该接口可用于验证对象。

### 2.1。动机

在上一篇文章中，我们定义了具有两个属性的实体—`name`和`email`。

因此，要创建一个新资源，我们只需运行:

```java
curl -i -X POST -H "Content-Type:application/json" -d 
  '{ "name" : "Test", "email" : "[[email protected]](/web/20221023133213/https://www.baeldung.com/cdn-cgi/l/email-protection)" }' 
  http://localhost:8080/users
```

这个 POST 请求将把提供的 JSON 对象保存到我们的数据库中，操作将返回:

```java
{
  "name" : "Test",
  "email" : "[[email protected]](/web/20221023133213/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "_links" : {
    "self" : {
        "href" : "http://localhost:8080/users/1"
    },
    "websiteUser" : {
        "href" : "http://localhost:8080/users/1"
    }
  }
}
```

因为我们提供了有效的数据，所以预期会有积极的结果。但是，如果我们删除属性`name`，或者只是将值设置为空的*字符串*，会发生什么呢？

为了测试第一个场景，我们将运行之前修改过的命令，将空字符串设置为属性`name`的值:

```java
curl -i -X POST -H "Content-Type:application/json" -d 
  '{ "name" : "", "email" : "Baggins" }' http://localhost:8080/users
```

使用该命令，我们将得到以下响应:

```java
{
  "name" : "",
  "email" : "Baggins",
  "_links" : {
    "self" : {
        "href" : "http://localhost:8080/users/1"
    },
    "websiteUser" : {
        "href" : "http://localhost:8080/users/1"
    }
  }
}
```

对于第二个场景，我们将从请求中删除属性`name`:

```java
curl -i -X POST -H "Content-Type:application/json" -d 
  '{ "email" : "Baggins" }' http://localhost:8080/users
```

对于该命令，我们将得到以下响应:

```java
{
  "name" : null,
  "email" : "Baggins",
  "_links" : {
    "self" : {
        "href" : "http://localhost:8080/users/2"
    },
    "websiteUser" : {
        "href" : "http://localhost:8080/users/2"
    }
  }
}
```

正如我们所看到的，两个请求都没问题，我们可以用 201 状态代码和到我们的对象**的 API 链接来确认这一点。**

这种行为是不可接受的，因为我们希望避免向数据库中插入部分数据。

### 2.2。春季数据休息事件

每次调用 Spring Data REST API 时，Spring Data REST exporter 都会生成各种事件，如下所示:

*   `BeforeCreateEvent`
*   `AfterCreateEvent`
*   `BeforeSaveEvent`
*   `AfterSaveEvent`
*   `BeforeLinkSaveEvent`
*   `AfterLinkSaveEvent`
*   `BeforeDeleteEvent`
*   `AfterDeleteEvent`

由于所有事件都以相似的方式处理，我们将只展示如何处理在新对象保存到数据库之前生成的`beforeCreateEvent`。

### 2.3。`Validator`定义一个

为了创建我们自己的验证器，我们需要用`supports`和`validate`方法实现`org.springframework.validation.Validator`接口。

`Supports`检查验证器是否支持提供的请求，而`validate`方法验证请求中提供的数据。

让我们定义一个`WebsiteUserValidator` 类:

```java
public class WebsiteUserValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return WebsiteUser.class.equals(clazz);
    }

    @Override
    public void validate(Object obj, Errors errors) {
        WebsiteUser user = (WebsiteUser) obj;
        if (checkInputString(user.getName())) {
            errors.rejectValue("name", "name.empty");
        }

        if (checkInputString(user.getEmail())) {
            errors.rejectValue("email", "email.empty");
        }
    }

    private boolean checkInputString(String input) {
        return (input == null || input.trim().length() == 0);
    }
}
```

`Errors` object 是一个特殊的类，用于包含 *validate* 方法中提供的所有错误。在本文的后面，我们将展示如何使用包含在`Errors`对象中的消息。
要添加新的错误消息，我们必须调用`errors.rejectValue(nameOfField, errorMessage)`。

在我们定义了验证器之后，我们需要将它映射到一个特定的事件，该事件是在请求被接受之后生成的。

例如，在我们的例子中，`beforeCreateEvent`的生成是因为我们想在数据库中插入一个新对象。但是因为我们想要验证请求中的对象，我们需要首先定义我们的验证器。

这可以通过三种方式实现:

*   添加名为“`beforeCreateWebsiteUserValidator`”的`Component`标注。Spring Boot 将识别前缀`beforeCreate`，它决定了我们想要捕捉的事件，它还将识别来自`Component`名称的`WebsiteUser`类。

    ```java
    @Component("beforeCreateWebsiteUserValidator")
    public class WebsiteUserValidator implements Validator {
        ...
    }
    ```

*   在带有`@Bean`注释:

    ```java
    @Bean
    public WebsiteUserValidator beforeCreateWebsiteUserValidator() {
        return new WebsiteUserValidator();
    }
    ```

    的应用程序上下文中创建`Bean`
*   手动注册:

    ```java
    @SpringBootApplication
    public class SpringDataRestApplication implements RepositoryRestConfigurer {
        public static void main(String[] args) {
            SpringApplication.run(SpringDataRestApplication.class, args);
        }

        @Override
        public void configureValidatingRepositoryEventListener(
          ValidatingRepositoryEventListener v) {
            v.addValidator("beforeCreate", new WebsiteUserValidator());
        }
    }
    ```

    *   对于这种情况，在`WebsiteUserValidator`类上不需要任何注释。

### 2.4。事件发现错误

目前，Spring Data REST 中存在一个[bug——影响事件发现。](https://web.archive.org/web/20221023133213/https://jira.spring.io/browse/DATAREST-524)

如果我们调用生成`beforeCreate`事件的 POST 请求，我们的应用程序将不会调用 validator，因为由于这个 bug，事件将不会被发现。

解决这个问题的一个简单方法是将所有事件插入到 Spring Data REST `ValidatingRepositoryEventListener`类中:

```java
@Configuration
public class ValidatorEventRegister implements InitializingBean {

    @Autowired
    ValidatingRepositoryEventListener validatingRepositoryEventListener;

    @Autowired
    private Map<String, Validator> validators;

    @Override
    public void afterPropertiesSet() throws Exception {
        List<String> events = Arrays.asList("beforeCreate");
        for (Map.Entry<String, Validator> entry : validators.entrySet()) {
            events.stream()
              .filter(p -> entry.getKey().startsWith(p))
              .findFirst()
              .ifPresent(
                p -> validatingRepositoryEventListener
               .addValidator(p, entry.getValue()));
        }
    }
}
```

## 3。测试

在**第 2.1 节中。**我们展示了在没有验证器的情况下，我们可以将没有名称属性的对象添加到我们的数据库中，这不是我们想要的行为，因为我们不检查数据完整性。

如果我们想要添加没有`name`属性但有提供的验证器的相同对象，我们将得到这个错误:

```java
curl -i -X POST -H "Content-Type:application/json" -d 
  '{ "email" : "[[email protected]](/web/20221023133213/https://www.baeldung.com/cdn-cgi/l/email-protection)" }' http://localhost:8080/users
```

```java
{  
   "timestamp":1472510818701,
   "status":406,
   "error":"Not Acceptable",
   "exception":"org.springframework.data.rest.core.
    RepositoryConstraintViolationException",
   "message":"Validation failed",
   "path":"/users"
}
```

正如我们所看到的，检测到请求中缺少数据，并且没有将对象保存到数据库中。我们的请求返回了 500 HTTP 代码和内部错误消息。

错误消息没有提到我们请求中的任何问题。如果我们想让它更具信息性，我们将不得不修改响应对象。

在 Spring 文章的[异常处理中，我们展示了如何处理由框架生成的异常，因此在这一点上，这绝对是一篇好文章。](/web/20221023133213/https://www.baeldung.com/exception-handling-for-rest-with-spring)

由于我们的应用程序生成了一个`RepositoryConstraintViolationException`异常，我们将为这个特殊的异常创建一个处理程序，它将修改响应消息。

这是我们的班级:

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler extends
  ResponseEntityExceptionHandler {

    @ExceptionHandler({ RepositoryConstraintViolationException.class })
    public ResponseEntity<Object> handleAccessDeniedException(
      Exception ex, WebRequest request) {
          RepositoryConstraintViolationException nevEx = 
            (RepositoryConstraintViolationException) ex;

          String errors = nevEx.getErrors().getAllErrors().stream()
            .map(p -> p.toString()).collect(Collectors.joining("\n"));

          return new ResponseEntity<Object>(errors, new HttpHeaders(),
            HttpStatus.PARTIAL_CONTENT);
    }
}
```

有了这个自定义处理程序，我们的返回对象将拥有关于所有检测到的错误的信息。

## 4。结论

在本文中，我们展示了验证器对于每个 Spring Data REST API 都是必不可少的，它为数据插入提供了额外的安全层。

我们还展示了用注释创建新的验证器是多么简单。

一如既往，这个应用程序的代码可以在 [GitHub 项目](https://web.archive.org/web/20221023133213/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest-2)中找到。