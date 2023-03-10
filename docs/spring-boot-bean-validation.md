# 在 Spring Boot 验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-bean-validation>

## 1。概述

当涉及到验证用户输入时， [Spring Boot](/web/20221003130442/https://www.baeldung.com/spring-boot) 提供了对这一常见但关键的任务的强大支持。

尽管 Spring Boot 支持与定制验证器的无缝集成，但是**执行验证的事实标准是[Hibernate Validator](https://web.archive.org/web/20221003130442/http://hibernate.org/validator/)**,[Bean 验证框架的参考实现。](/web/20221003130442/https://www.baeldung.com/javax-validation)

在本教程中，**我们将看看如何在 Spring Boot** 中验证域对象。

## 延伸阅读:

## [Spring Boot 的自定义验证消息来源](/web/20221003130442/https://www.baeldung.com/spring-custom-validation-message-source)

Learn how to register a custom MessageSource for validation messages in Spring Boot.[Read more](/web/20221003130442/https://www.baeldung.com/spring-custom-validation-message-source) →

## [Bean 验证中@NotNull、@NotEmpty 和@NotBlank 约束之间的差异](/web/20221003130442/https://www.baeldung.com/java-bean-validation-not-null-empty-blank)

Learn the semantics of the @NotNull, @NotEmpty, and @NotBlank bean validation annotations in Java and how they differ.[Read more](/web/20221003130442/https://www.baeldung.com/java-bean-validation-not-null-empty-blank) →

## 2。美芬依赖

在这种情况下，我们将学习如何通过构建一个基本的 REST 控制器来验证 Spring Boot **中的域对象。**

控制器将首先获取一个域对象，然后用 Hibernate Validator 验证它，最后将它持久化到内存中的 H2 数据库中。

该项目的依赖项相当标准:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency> 
<dependency> 
    <groupId>com.h2database</groupId> 
    <artifactId>h2</artifactId>
    <version>1.4.197</version> 
    <scope>runtime</scope>
</dependency>
```

如上所示，我们在`pom.xml`文件中包含了`[spring-boot-starter-web](https://web.archive.org/web/20221003130442/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web)`,因为我们需要它来创建 REST 控制器。此外，让我们确保在 Maven Central 上查看最新版本的`[spring-boot-starter-jpa](https://web.archive.org/web/20221003130442/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa)`和 [H2 数据库](https://web.archive.org/web/20221003130442/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2)。

从 Boot 2.3 开始，我们还需要显式添加 [`spring-boot-starter-validation`](https://web.archive.org/web/20221003130442/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-validation) 依赖项:

```java
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-validation</artifactId> 
</dependency>
```

## 3。一个简单的域类

我们项目的依赖项已经就绪，接下来我们需要定义一个示例 JPA 实体类，它的角色仅仅是建模用户。

让我们来看看这个类:

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @NotBlank(message = "Name is mandatory")
    private String name;

    @NotBlank(message = "Email is mandatory")
    private String email;

    // standard constructors / setters / getters / toString

}
```

我们的`User`实体类的实现确实非常贫乏，但是它简单地展示了如何使用 Bean 验证的约束来约束`name`和`email`字段。

为了简单起见，我们只使用 [`@NotBlank`](/web/20221003130442/https://www.baeldung.com/java-bean-validation-not-null-empty-blank) 约束来约束目标字段。此外，我们用`message`属性指定了错误消息。

因此，当 Spring Boot 验证类实例时，受约束的字段**必须不为空，并且它们的修整长度必须大于零**。

此外， [Bean 验证](/web/20221003130442/https://www.baeldung.com/javax-validation)除了`@NotBlank.` 之外还提供了许多其他方便的约束，这允许我们对受约束的类应用和组合不同的验证规则。更多信息，请阅读[官方 bean 验证文档](https://web.archive.org/web/20221003130442/https://beanvalidation.org/2.0/)。

由于我们将使用 [Spring Data JPA](/web/20221003130442/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 将用户保存到内存中的 H2 数据库，我们还需要定义一个简单的存储库接口，以便在`User`对象上拥有基本的 CRUD 功能:

```java
@Repository
public interface UserRepository extends CrudRepository<User, Long> {}
```

## 4。实施休息控制器

当然，我们需要实现一个层，允许我们获取分配给我们的`User`对象的约束字段的值。

因此，我们可以验证它们，并根据验证结果执行一些进一步的任务。

Spring Boot 通过实现一个 REST 控制器使这个看似复杂的过程变得非常简单。

让我们看看 REST 控制器的实现:

```java
@RestController
public class UserController {

    @PostMapping("/users")
    ResponseEntity<String> addUser(@Valid @RequestBody User user) {
        // persisting the user
        return ResponseEntity.ok("User is valid");
    }

    // standard constructors / other methods

} 
```

在 [Spring REST 上下文](/web/20221003130442/https://www.baeldung.com/rest-with-spring-series)中，`addUser()`方法的实现是相当标准的。

当然，最相关的部分是使用了`@Valid`注释。

**当 Spring Boot 找到用`@Valid`标注的参数时，它会自动启动默认的 JSR 380 实现——Hibernate Validator——并验证参数。**

当目标参数未能通过验证时，Spring Boot 抛出一个 [`MethodArgumentNotValidException`](https://web.archive.org/web/20221003130442/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/MethodArgumentNotValidException.html) 异常。

## 5。`@ExceptionHandler`注解

虽然让 Spring Boot 自动验证传递给`addUser()`方法的`User`对象非常方便，但是这个过程缺少的方面是我们如何处理验证结果。

[`@ExceptionHandler`](https://web.archive.org/web/20221003130442/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html) 注释**允许我们通过一个方法处理特定类型的异常。**

因此，我们可以用它来处理验证错误:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(MethodArgumentNotValidException.class)
public Map<String, String> handleValidationExceptions(
  MethodArgumentNotValidException ex) {
    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getAllErrors().forEach((error) -> {
        String fieldName = ((FieldError) error).getField();
        String errorMessage = error.getDefaultMessage();
        errors.put(fieldName, errorMessage);
    });
    return errors;
}
```

我们将`MethodArgumentNotValidException`异常指定为[要处理的异常](/web/20221003130442/https://www.baeldung.com/exception-handling-for-rest-with-spring)。因此，当指定的`User`对象无效时，Spring Boot 将调用这个方法**。**

该方法将每个无效字段的名称和验证后错误消息存储在一个`Map.`中，然后将`Map`作为 JSON 表示发送回客户端，以供进一步处理。

简单地说，REST 控制器允许我们轻松地处理对不同端点的请求，验证`User`对象，并以 JSON 格式发送响应。

该设计足够灵活，可以通过几个 web 层处理控制器响应，从模板引擎如[百里香](/web/20221003130442/https://www.baeldung.com/spring-boot-crud-thymeleaf)，到全功能 JavaScript 框架如 [Angular](https://web.archive.org/web/20221003130442/https://angular.io/) 。

## 6。测试剩余控制器

我们可以通过[集成测试](/web/20221003130442/https://www.baeldung.com/spring-boot-testing)轻松测试 REST 控制器的功能。

让我们开始模仿/自动连接`UserRepository`接口实现，以及`UserController`实例和`[MockMvc](https://web.archive.org/web/20221003130442/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html)`对象:

```java
@RunWith(SpringRunner.class) 
@WebMvcTest
@AutoConfigureMockMvc
public class UserControllerIntegrationTest {

    @MockBean
    private UserRepository userRepository;

    @Autowired
    UserController userController;

    @Autowired
    private MockMvc mockMvc;

    //...

} 
```

因为我们只测试 web 层，所以我们使用 [`@WebMvcTest`](https://web.archive.org/web/20221003130442/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html) 注释。它允许我们使用由 [`MockMvcRequestBuilders`](https://web.archive.org/web/20221003130442/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockMvcRequestBuilders.html) 和`[MockMvcResultMatchers](https://web.archive.org/web/20221003130442/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/result/MockMvcResultMatchers.html)`类实现的一组静态方法轻松测试请求和响应。

现在让我们用请求体中传递的有效和无效的`User`对象来测试`addUser()`方法:

```java
@Test
public void whenPostRequestToUsersAndValidUser_thenCorrectResponse() throws Exception {
    MediaType textPlainUtf8 = new MediaType(MediaType.TEXT_PLAIN, Charset.forName("UTF-8"));
    String user = "{\"name\": \"bob\", \"email\" : \"[[email protected]](/web/20221003130442/https://www.baeldung.com/cdn-cgi/l/email-protection)\"}";
    mockMvc.perform(MockMvcRequestBuilders.post("/users")
      .content(user)
      .contentType(MediaType.APPLICATION_JSON_UTF8))
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.content()
        .contentType(textPlainUtf8));
}

@Test
public void whenPostRequestToUsersAndInValidUser_thenCorrectResponse() throws Exception {
    String user = "{\"name\": \"\", \"email\" : \"[[email protected]](/web/20221003130442/https://www.baeldung.com/cdn-cgi/l/email-protection)\"}";
    mockMvc.perform(MockMvcRequestBuilders.post("/users")
      .content(user)
      .contentType(MediaType.APPLICATION_JSON_UTF8))
      .andExpect(MockMvcResultMatchers.status().isBadRequest())
      .andExpect(MockMvcResultMatchers.jsonPath("$.name", Is.is("Name is mandatory")))
      .andExpect(MockMvcResultMatchers.content()
        .contentType(MediaType.APPLICATION_JSON_UTF8));
    }
} 
```

此外，我们可以使用免费的 API 生命周期测试应用程序来测试 REST 控制器 API **，比如[邮递员](https://web.archive.org/web/20221003130442/https://www.getpostman.com/)。**

## 7。运行示例应用程序

最后，我们可以用标准的`main()`方法运行我们的示例项目:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CommandLineRunner run(UserRepository userRepository) throws Exception {
        return (String[] args) -> {
            User user1 = new User("Bob", "[[email protected]](/web/20221003130442/https://www.baeldung.com/cdn-cgi/l/email-protection)");
            User user2 = new User("Jenny", "[[email protected]](/web/20221003130442/https://www.baeldung.com/cdn-cgi/l/email-protection)");
            userRepository.save(user1);
            userRepository.save(user2);
            userRepository.findAll().forEach(System.out::println);
        };
    }
} 
```

正如所料，我们应该在控制台中看到几个`User`对象被打印出来。

带有有效的`User`对象的对[http://localhost:8080/users](https://web.archive.org/web/20221003130442/http://localhost:8080/users)端点的 POST 请求将返回`String`“用户有效”。

同样，带有不带`name`和`email`值的`User` 对象的 POST 请求将返回以下响应:

```java
{
  "name":"Name is mandatory",
  "email":"Email is mandatory"
}
```

## 8。结论

在本文中，**我们学习了在 Spring Boot** 执行验证的基础知识。

像往常一样，本文中展示的所有例子都可以在 [GitHub](https://web.archive.org/web/20221003130442/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-validation) 上找到。