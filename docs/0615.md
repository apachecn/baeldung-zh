# Spring Boot 教程-引导一个简单的应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-start>

## 1。概述

Spring Boot 是 Spring 平台的一个自以为是的补充，专注于约定胜于配置——对于以最小的努力开始和创建独立的生产级应用程序非常有用。

本教程是 Boot 的起点，换句话说，是一种以简单方式开始使用基本 web 应用程序的方法。

我们将讨论一些核心配置、前端、快速数据操作和异常处理。

## 延伸阅读:

## [如何更改 Spring Boot 的默认端口](/web/20220923104110/https://www.baeldung.com/spring-boot-change-port)

Have a look at how you can change the default port in a Spring Boot application.[Read more](/web/20220923104110/https://www.baeldung.com/spring-boot-change-port) →

## [Spring Boot 新手入门](/web/20220923104110/https://www.baeldung.com/spring-boot-starters)

A quick overview of the most common Spring Boot Starters, along with examples on how to use them in a real-world project.[Read more](/web/20220923104110/https://www.baeldung.com/spring-boot-starters) →

## 2。设置

首先，让我们使用 [Spring Initializr](https://web.archive.org/web/20220923104110/https://start.spring.io/) 为我们的项目生成基础。

生成的项目依赖于引导父项目:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
    <relativePath />
</parent>
```

最初的依赖非常简单:

```
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
</dependency>
```

## 3。应用程序配置

接下来，我们将为我们的应用程序配置一个简单的`main`类:

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
} 
```

请注意**如何使用`@SpringBootApplication` 作为我们的主要应用程序配置类。**幕后，那相当于`@Configuration`、`@EnableAutoConfiguration,`和`@ComponentScan`合在一起。

最后，我们将定义一个简单的`application.properties` 文件，它现在只有一个属性:

```
server.port=8081 
```

`server.port`将服务器端口从默认的 8080 更改为 8081；当然，Spring Boot 还有更多[房产可供选择](https://web.archive.org/web/20220923104110/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)。

## 4。简单的 MVC 视图

现在让我们使用百里香叶添加一个简单的前端。

首先，我们需要将`spring-boot-starter-thymeleaf`依赖项添加到我们的`pom.xml`中:

```
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-thymeleaf</artifactId> 
</dependency> 
```

默认情况下会启用百里香。不需要额外的配置。

我们现在可以在我们的`application.properties`中配置它:

```
spring.thymeleaf.cache=false
spring.thymeleaf.enabled=true 
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

spring.application.name=Bootstrap Spring Boot 
```

接下来，我们将定义一个简单的[控制器](/web/20220923104110/https://www.baeldung.com/spring-controllers)和一个带有欢迎消息的基本主页:

```
@Controller
public class SimpleController {
    @Value("${spring.application.name}")
    String appName;

    @GetMapping("/")
    public String homePage(Model model) {
        model.addAttribute("appName", appName);
        return "home";
    }
} 
```

最后，这里是我们的`home.html`:

```
<html>
<head><title>Home Page</title></head>
<body>
<h1>Hello !</h1>
<p>Welcome to <span th:text="${appName}">Our App</span></p>
</body>
</html> 
```

请注意我们如何使用在属性中定义的属性，然后注入它，这样我们就可以在主页上显示它。

## 5。安全性

接下来，让我们通过首先包含安全启动程序来增加应用程序的安全性:

```
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-security</artifactId> 
</dependency> 
```

到目前为止，我们可以注意到一种模式:**大多数 Spring 库都可以通过使用简单的引导启动器轻松地导入到我们的项目中。**

一旦`spring-boot-starter-security`依赖项位于应用程序的类路径上，默认情况下所有端点都是安全的，使用基于 Spring Security 的内容协商策略的`httpBasic`或`formLogin`。

这就是为什么，如果我们在类路径上有 starter，我们通常应该定义我们自己的定制安全配置:

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .anyRequest()
            .permitAll()
            .and()
            .csrf()
            .disable();
        return http.build();
    }
} 
```

在我们的示例中，我们允许不受限制地访问所有端点。

当然，Spring 安全性是一个广泛的主题，不容易在几行配置中涵盖。所以，我们肯定鼓励[更深入地阅读主题](/web/20220923104110/https://www.baeldung.com/security-spring)。

## 6。简单持久性

让我们首先定义我们的数据模型，一个简单的`Book`实体:

```
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Column(nullable = false, unique = true)
    private String title;

    @Column(nullable = false)
    private String author;
}
```

和它的存储库，在这里很好地利用了 Spring 数据:

```
public interface BookRepository extends CrudRepository<Book, Long> {
    List<Book> findByTitle(String title);
}
```

最后，我们当然需要配置新的持久层:

```
@EnableJpaRepositories("com.baeldung.persistence.repo") 
@EntityScan("com.baeldung.persistence.model")
@SpringBootApplication 
public class Application {
   ...
}
```

请注意，我们使用了以下内容:

*   `@EnableJpaRepositories`扫描指定软件包中的存储库
*   `@EntityScan` 领取我们的 JPA 实体

为了简单起见，我们在这里使用 H2 内存数据库。这是为了让我们在运行项目时没有任何外部依赖。

一旦我们包含了 H2 依赖， **Spring Boot 会自动检测它并设置我们的持久性**,除了数据源属性之外，不需要额外的配置:

```
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:bootapp;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password= 
```

当然，像安全性一样，持久性是一个比这里的基本设置更广泛的主题，当然需要进一步探索。

## 7。Web 和控制器

接下来，让我们看看 web 层。我们将从设置一个简单的控制器`BookController`开始。

我们将实现基本的 CRUD 操作，通过一些简单的验证来公开`Book`资源:

```
@RestController
@RequestMapping("/api/books")
public class BookController {

    @Autowired
    private BookRepository bookRepository;

    @GetMapping
    public Iterable findAll() {
        return bookRepository.findAll();
    }

    @GetMapping("/title/{bookTitle}")
    public List findByTitle(@PathVariable String bookTitle) {
        return bookRepository.findByTitle(bookTitle);
    }

    @GetMapping("/{id}")
    public Book findOne(@PathVariable Long id) {
        return bookRepository.findById(id)
          .orElseThrow(BookNotFoundException::new);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Book create(@RequestBody Book book) {
        return bookRepository.save(book);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        bookRepository.findById(id)
          .orElseThrow(BookNotFoundException::new);
        bookRepository.deleteById(id);
    }

    @PutMapping("/{id}")
    public Book updateBook(@RequestBody Book book, @PathVariable Long id) {
        if (book.getId() != id) {
          throw new BookIdMismatchException();
        }
        bookRepository.findById(id)
          .orElseThrow(BookNotFoundException::new);
        return bookRepository.save(book);
    }
} 
```

考虑到应用程序的这一方面是一个 API，我们在这里使用了@ `RestController`注释——它相当于一个`@Controller`和`@ResponseBody`——这样每个方法都可以将返回的资源正确地编组到 HTTP 响应中。

注意，我们在这里将我们的`Book`实体公开为我们的外部资源。这对于这个简单的应用程序来说很好，但是在现实世界的应用程序中，我们可能会希望[将这两个概念](/web/20220923104110/https://www.baeldung.com/entity-to-and-from-dto-for-a-java-spring-application)分开。

## 8。错误处理

既然核心应用程序已经准备就绪，让我们关注一下**一个简单的集中式错误处理机制**使用`@ControllerAdvice`:

```
@ControllerAdvice
public class RestExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ BookNotFoundException.class })
    protected ResponseEntity<Object> handleNotFound(
      Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, "Book not found", 
          new HttpHeaders(), HttpStatus.NOT_FOUND, request);
    }

    @ExceptionHandler({ BookIdMismatchException.class, 
      ConstraintViolationException.class, 
      DataIntegrityViolationException.class })
    public ResponseEntity<Object> handleBadRequest(
      Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, ex.getLocalizedMessage(), 
          new HttpHeaders(), HttpStatus.BAD_REQUEST, request);
    }
} 
```

除了我们在这里处理的标准异常，我们还使用了一个自定义异常，`BookNotFoundException`:

```
public class BookNotFoundException extends RuntimeException {

    public BookNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
    // ...
} 
```

这让我们了解了这个全局异常处理机制的可能性。要查看完整的实现，请看深入教程。

注意，默认情况下，Spring Boot 还提供了一个`/error`映射。我们可以通过创建一个简单的`error.html`来自定义它的视图:

```
<html lang="en">
<head><title>Error Occurred</title></head>
<body>
    <h1>Error Occurred!</h1>    
    <b>[<span th:text="${status}">status</span>]
        <span th:text="${error}">error</span>
    </b>
    <p th:text="${message}">message</p>
</body>
</html>
```

像 Boot 中的大多数其他方面一样，我们可以用一个简单的属性来控制它:

```
server.error.path=/error2
```

## 9。测试

最后，让我们测试一下我们的新书 API。

我们可以利用 [`@SpringBootTest`](/web/20220923104110/https://www.baeldung.com/spring-boot-testing) 加载应用上下文，并验证运行应用时没有错误:

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringContextTest {

    @Test
    public void contextLoads() {
    }
}
```

接下来，让我们添加一个 JUnit 测试来验证对我们编写的 API 的调用，使用[放心](/web/20220923104110/https://www.baeldung.com/rest-assured-tutorial)。

首先，我们将添加 [`rest-assured`](https://web.archive.org/web/20220923104110/https://search.maven.org/artifact/io.rest-assured/rest-assured) 依赖项:

```
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <scope>test</scope>
</dependency>
```

现在我们可以添加测试:

```
public class SpringBootBootstrapLiveTest {

    private static final String API_ROOT
      = "http://localhost:8081/api/books";

    private Book createRandomBook() {
        Book book = new Book();
        book.setTitle(randomAlphabetic(10));
        book.setAuthor(randomAlphabetic(15));
        return book;
    }

    private String createBookAsUri(Book book) {
        Response response = RestAssured.given()
          .contentType(MediaType.APPLICATION_JSON_VALUE)
          .body(book)
          .post(API_ROOT);
        return API_ROOT + "/" + response.jsonPath().get("id");
    }
} 
```

首先，我们可以尝试使用不同的方法来查找书籍:

```
@Test
public void whenGetAllBooks_thenOK() {
    Response response = RestAssured.get(API_ROOT);

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
}

@Test
public void whenGetBooksByTitle_thenOK() {
    Book book = createRandomBook();
    createBookAsUri(book);
    Response response = RestAssured.get(
      API_ROOT + "/title/" + book.getTitle());

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertTrue(response.as(List.class)
      .size() > 0);
}
@Test
public void whenGetCreatedBookById_thenOK() {
    Book book = createRandomBook();
    String location = createBookAsUri(book);
    Response response = RestAssured.get(location);

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertEquals(book.getTitle(), response.jsonPath()
      .get("title"));
}

@Test
public void whenGetNotExistBookById_thenNotFound() {
    Response response = RestAssured.get(API_ROOT + "/" + randomNumeric(4));

    assertEquals(HttpStatus.NOT_FOUND.value(), response.getStatusCode());
} 
```

接下来，我们将测试创建一本新书:

```
@Test
public void whenCreateNewBook_thenCreated() {
    Book book = createRandomBook();
    Response response = RestAssured.given()
      .contentType(MediaType.APPLICATION_JSON_VALUE)
      .body(book)
      .post(API_ROOT);

    assertEquals(HttpStatus.CREATED.value(), response.getStatusCode());
}

@Test
public void whenInvalidBook_thenError() {
    Book book = createRandomBook();
    book.setAuthor(null);
    Response response = RestAssured.given()
      .contentType(MediaType.APPLICATION_JSON_VALUE)
      .body(book)
      .post(API_ROOT);

    assertEquals(HttpStatus.BAD_REQUEST.value(), response.getStatusCode());
} 
```

然后，我们将更新现有的图书:

```
@Test
public void whenUpdateCreatedBook_thenUpdated() {
    Book book = createRandomBook();
    String location = createBookAsUri(book);
    book.setId(Long.parseLong(location.split("api/books/")[1]));
    book.setAuthor("newAuthor");
    Response response = RestAssured.given()
      .contentType(MediaType.APPLICATION_JSON_VALUE)
      .body(book)
      .put(location);

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());

    response = RestAssured.get(location);

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertEquals("newAuthor", response.jsonPath()
      .get("author"));
} 
```

我们可以删除一本书:

```
@Test
public void whenDeleteCreatedBook_thenOk() {
    Book book = createRandomBook();
    String location = createBookAsUri(book);
    Response response = RestAssured.delete(location);

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());

    response = RestAssured.get(location);
    assertEquals(HttpStatus.NOT_FOUND.value(), response.getStatusCode());
} 
```

## 10。结论

这是对 Spring Boot 的快速而全面的介绍。

当然，我们在这里仅仅触及了皮毛。这个框架的内容比我们在一篇介绍文章中所能涵盖的要多得多。

这就是为什么我们在网站上有不止一篇关于靴子的文章。

和往常一样，我们这里的例子的完整源代码在 GitHub 上的[。](https://web.archive.org/web/20220923104110/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-bootstrap)