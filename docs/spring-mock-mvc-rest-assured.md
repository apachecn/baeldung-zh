# 对 Spring MockMvc 的可靠支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mock-mvc-rest-assured>

## 1.介绍

在本教程中，我们将学习如何使用`RestAssuredMockMvc`**测试我们的 Spring REST 控制器，这是一个构建在 Spring`MockMvc`之上的放心 API。**

首先，我们将检查不同的设置选项。然后，我们将深入研究如何编写单元测试和集成测试。

本教程使用了 [Spring MVC](/web/20221129010551/https://www.baeldung.com/spring-mvc) 、 [Spring MockMVC](/web/20221129010551/https://www.baeldung.com/integration-testing-in-spring) 和[放心](/web/20221129010551/https://www.baeldung.com/rest-assured-tutorial)，所以也请务必查看那些教程。

## 2.Maven 依赖性

在我们开始编写测试之前，我们需要将 [`io.rest-assured:spring-mock-mvc`模块](https://web.archive.org/web/20221129010551/https://search.maven.org/search?q=g:io.rest-assured%20AND%20a:spring-mock-mvc)导入到我们的 Maven `pom.xml`中:

```java
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>spring-mock-mvc</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
```

## 3.正在初始化`RestAssuredMockMvc`

接下来，我们需要在`standalone`或`web application context`模式下初始化`RestAssuredMockMvc,`DSL 的起始点。

在这两种模式下，我们既可以在每次测试时即时完成，也可以静态完成。让我们来看一些例子。

### 3.1.单独的

在独立模式下，我们**用一个或多个`[@Controller](https://web.archive.org/web/20221129010551/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)`或`[@ControllerAdvice](https://web.archive.org/web/20221129010551/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html)`注释类初始化`RestAssuredMockMvc`。**

如果我们只有几个测试，我们可以及时初始化`RestAssuredMockMvc`:

```java
@Test
public void whenGetCourse() {
    given()
      .standaloneSetup(new CourseController())
      //...
}
```

但是，如果我们有很多测试，静态地做一次会更容易:

```java
@Before
public void initialiseRestAssuredMockMvcStandalone() {
    RestAssuredMockMvc.standaloneSetup(new CourseController());
}
```

### 3.2.Web 应用程序上下文

在 web 应用程序上下文模式中，我们**用一个 Spring [`WebApplicationContext`](https://web.archive.org/web/20221129010551/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/WebApplicationContext.html) 的实例初始化`RestAssuredMockMvc`。**

类似于我们在独立模式设置中看到的，我们可以在每次测试时及时初始化`RestAssuredMockMvc`:

```java
@Autowired
private WebApplicationContext webApplicationContext;

@Test
public void whenGetCourse() {
    given()
      .webAppContextSetup(webApplicationContext)
      //...
}
```

或者，我们可以静态地做一次:

```java
@Autowired
private WebApplicationContext webApplicationContext;

@Before
public void initialiseRestAssuredMockMvcWebApplicationContext() {
    RestAssuredMockMvc.webAppContextSetup(webApplicationContext);
}
```

## 4.被测系统(SUT)

在我们开始几个示例测试之前，我们需要测试一些东西。让我们检查一下测试中的系统，从我们的`[@SpringBootApplication](https://web.archive.org/web/20221129010551/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/SpringBootApplication.html)`配置开始:

```java
@SpringBootApplication
class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

接下来，我们有一个简单的`[@RestController](https://web.archive.org/web/20221129010551/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html)`来展示我们的`Course`域:

```java
@RestController
@RequestMapping(path = "/courses")
public class CourseController {

    private final CourseService courseService;

    public CourseController(CourseService courseService) {
        this.courseService = courseService;
    }

    @GetMapping(produces = APPLICATION_JSON_UTF8_VALUE)
    public Collection<Course> getCourses() {
        return courseService.getCourses();
    }

    @GetMapping(path = "/{code}", produces = APPLICATION_JSON_UTF8_VALUE)
    public Course getCourse(@PathVariable String code) {
        return courseService.getCourse(code);
    }
}
```

```java
class Course {

    private String code;

    // usual contructors, getters and setters
}
```

最后但同样重要的是，我们的服务类和处理我们的`CourseNotFoundException`的`@ControllerAdvice`:

```java
@Service
class CourseService {

    private static final Map<String, Course> COURSE_MAP = new ConcurrentHashMap<>();

    static {
        Course wizardry = new Course("Wizardry");
        COURSE_MAP.put(wizardry.getCode(), wizardry);
    }

    Collection<Course> getCourses() {
        return COURSE_MAP.values();
    }

    Course getCourse(String code) {
        return Optional.ofNullable(COURSE_MAP.get(code)).orElseThrow(() -> 
          new CourseNotFoundException(code));
    }
}
```

```java
@ControllerAdvice(assignableTypes = CourseController.class)
public class CourseControllerExceptionHandler extends ResponseEntityExceptionHandler {

    @ResponseStatus(HttpStatus.NOT_FOUND)
    @ExceptionHandler(CourseNotFoundException.class)
    public void handleCourseNotFoundException(CourseNotFoundException cnfe) {
        //...
    }
}
```

```java
class CourseNotFoundException extends RuntimeException {

    CourseNotFoundException(String code) {
        super(code);
    }
}
```

现在我们有了一个要测试的系统，让我们来看看几个`RestAssuredMockMvc`测试。

## 5.放心地测试控制器单元

我们可以使用`RestAssuredMockMvc`和我们喜欢的测试工具 [JUnit](/web/20221129010551/https://www.baeldung.com/junit) 和 [Mockito](/web/20221129010551/https://www.baeldung.com/mockito-series) 来测试我们的`@RestController`。

首先，我们模仿并构建我们的 SUT，然后如上所述在独立模式下初始化`RestAssuredMockMvc`:

```java
@RunWith(MockitoJUnitRunner.class)
public class CourseControllerUnitTest {

    @Mock
    private CourseService courseService;
    @InjectMocks
    private CourseController courseController;
    @InjectMocks
    private CourseControllerExceptionHandler courseControllerExceptionHandler;

    @Before
    public void initialiseRestAssuredMockMvcStandalone() {
        RestAssuredMockMvc.standaloneSetup(courseController, courseControllerExceptionHandler);
    }
```

因为我们已经在我们的`@Before`方法中静态初始化了`RestAssuredMockMvc`，所以不需要在每个测试中初始化它。

独立模式非常适合单元测试，因为它只初始化我们提供的控制器，而不是整个应用上下文。这使得我们的测试速度很快。

现在，让我们来看一个测试示例:

```java
@Test
public void givenNoExistingCoursesWhenGetCoursesThenRespondWithStatusOkAndEmptyArray() {
    when(courseService.getCourses()).thenReturn(Collections.emptyList());

    given()
      .when()
        .get("/courses")
      .then()
        .log().ifValidationFails()
        .statusCode(OK.value())
        .contentType(JSON)
        .body(is(equalTo("[]")));
}
```

除了我们的`@RestController`之外，用我们的`@ControllerAdvice`初始化`RestAssuredMockMvc`也使我们能够测试我们的异常场景:

```java
@Test
public void givenNoMatchingCoursesWhenGetCoursesThenRespondWithStatusNotFound() {
    String nonMatchingCourseCode = "nonMatchingCourseCode";

    when(courseService.getCourse(nonMatchingCourseCode)).thenThrow(
      new CourseNotFoundException(nonMatchingCourseCode));

    given()
      .when()
        .get("/courses/" + nonMatchingCourseCode)
      .then()
        .log().ifValidationFails()
        .statusCode(NOT_FOUND.value());
}
```

如上所述，放心使用熟悉的给定时间场景格式来定义测试:

*   `given()` —指定 HTTP 请求的详细信息
*   `when()` —指定 HTTP 动词和路由
*   `then()` —验证 HTTP 响应

## 6.放心的 REST 控制器集成测试

我们还可以将`RestAssuredMockMvc`与 Spring 的测试工具一起用于我们的集成测试。

首先，我们用`@RunWith(SpringRunner.class)`和 [`@SpringBootTest(webEnvironment = RANDOM_PORT)`](https://web.archive.org/web/20221129010551/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html) 设置我们的测试类:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class CourseControllerIntegrationTest {
    //...
}
```

这将在随机端口上使用我们的`@SpringBootApplication`类中配置的应用程序上下文运行我们的测试。

接下来，我们注入我们的`WebApplicationContext`,并使用它初始化`RestAssuredMockMvc`,如上所示:

```java
@Autowired
private WebApplicationContext webApplicationContext;

@Before
public void initialiseRestAssuredMockMvcWebApplicationContext() {
    RestAssuredMockMvc.webAppContextSetup(webApplicationContext);
}
```

既然我们已经设置了测试类并且初始化了`RestAssuredMockMvc`,我们就可以开始编写测试了:

```java
@Test
public void givenNoMatchingCourseCodeWhenGetCourseThenRespondWithStatusNotFound() {
    String nonMatchingCourseCode = "nonMatchingCourseCode";

    given()
      .when()
        .get("/courses/" + nonMatchingCourseCode)
      .then()
        .log().ifValidationFails()
        .statusCode(NOT_FOUND.value());
}
```

记住，因为我们已经在我们的`@Before`方法中静态初始化了`RestAssuredMockMvc`，所以不需要在每个测试中初始化它。

要深入了解放心 API，请查看我们的[放心指南](/web/20221129010551/https://www.baeldung.com/rest-assured-tutorial)。

## 7.结论

在本教程中，我们看到了如何使用 REST-assured 的`spring-mock-mvc`模块来测试 Spring MVC 应用程序。

在**独立模式下初始化`RestAssuredMockMvc`对于单元测试**来说非常好，因为它只初始化提供的`Controller` s，保持我们的测试快速。

在 **web 应用程序上下文模式中初始化`RestAssuredMockMvc`对于集成测试**来说非常好，因为它使用了我们完整的`WebApplicationContext`。

和往常一样，你可以在 Github 上找到我们所有的样本代码[。](https://web.archive.org/web/20221129010551/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)