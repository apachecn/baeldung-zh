# 问题指南 Spring Web 库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/problem-spring-web>

## 1.概观

在本教程中，我们将探索**如何使用[问题 Spring Web 库](https://web.archive.org/web/20220816214725/https://github.com/zalando/problem-spring-web)产生 `application/problem+json`响应** **。这个库帮助我们避免与错误处理相关的重复性任务。**

通过将问题 Spring Web 集成到我们的 Spring Boot 应用程序中，我们可以**简化我们在项目中处理异常的方式，并相应地生成响应**。

## 2.问题库

问题是一个小库，目的是标准化基于 Java 的 Rest APIs 向消费者表达错误的方式。

一个`Problem `是我们想要通知的任何错误的抽象。它包含了关于错误的有用信息。让我们看看`Problem`响应的默认表示:

```
{
  "title": "Not Found",
  "status": 404
}
```

在这种情况下，状态代码和标题足以描述错误。但是，我们也可以添加一个对它的详细描述:

```
{
  "title": "Service Unavailable",
  "status": 503,
  "detail": "Database not reachable"
}
```

我们还可以创建定制的`Problem`对象来适应我们的需求:

```
Problem.builder()
  .withType(URI.create("https://example.org/out-of-stock"))
  .withTitle("Out of Stock")
  .withStatus(BAD_REQUEST)
  .withDetail("Item B00027Y5QG is no longer available")
  .with("product", "B00027Y5QG")
  .build();
```

在本教程中，我们将关注 Spring Boot 项目的问题库实现。

## 3.Spring Web 安装有问题

因为这是一个基于 Maven 的项目，所以让我们将`[problem-spring-web](https://web.archive.org/web/20220816214725/https://search.maven.org/search?q=g:%22org.zalando%22%20AND%20a:%22problem-spring-web%22)`依赖项添加到`pom.xml`中:

```
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>problem-spring-web</artifactId>
    <version>0.23.0</version>
</dependency>
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.0</version> 
</dependency>
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.4.0</version>  
</dependency>
```

我们还需要 `[spring-boot-starter-web](https://web.archive.org/web/20220816214725/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web)`和`[spring-boot-starter-security](https://web.archive.org/web/20220816214725/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-security)` 依赖项。从`problem-spring-web`0 . 23 . 0 版本开始需要 Spring 安全。

## 4.基本配置

作为我们的第一步，我们需要禁用白色标签错误页面，以便我们能够看到我们的自定义错误表示:

```
@EnableAutoConfiguration(exclude = ErrorMvcAutoConfiguration.class)
```

现在，让我们在`ObjectMapper` bean 中注册一些必需的组件:

```
@Bean
public ObjectMapper objectMapper() {
    return new ObjectMapper().registerModules(
      new ProblemModule(),
      new ConstraintViolationProblemModule());
} 
```

之后，我们需要将以下属性添加到`application.properties`文件中:

```
spring.resources.add-mappings=false
spring.mvc.throw-exception-if-no-handler-found=true
spring.http.encoding.force=true
```

最后，我们需要实现`ProblemHandling`接口:

```
@ControllerAdvice
public class ExceptionHandler implements ProblemHandling {}
```

## 5.高级配置

除了基本配置之外，我们还可以配置我们的项目来处理与安全相关的问题。第一步是创建一个配置类来支持库与 Spring Security 的集成:

```
@Configuration
@EnableWebSecurity
@Import(SecurityProblemSupport.class)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private SecurityProblemSupport problemSupport;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // Other security-related configuration
        http.exceptionHandling()
          .authenticationEntryPoint(problemSupport)
          .accessDeniedHandler(problemSupport);
    }
}
```

最后，我们需要为与安全相关的异常创建一个异常处理程序:

```
@ControllerAdvice
public class SecurityExceptionHandler implements SecurityAdviceTrait {}
```

## 6.休息控制器

配置完应用程序后，我们就可以创建 RESTful 控制器了:

```
@RestController
@RequestMapping("/tasks")
public class ProblemDemoController {

    private static final Map<Long, Task> MY_TASKS;

    static {
        MY_TASKS = new HashMap<>();
        MY_TASKS.put(1L, new Task(1L, "My first task"));
        MY_TASKS.put(2L, new Task(2L, "My second task"));
    }

    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public List<Task> getTasks() {
        return new ArrayList<>(MY_TASKS.values());
    }

    @GetMapping(value = "/{id}",
      produces = MediaType.APPLICATION_JSON_VALUE)
    public Task getTasks(@PathVariable("id") Long taskId) {
        if (MY_TASKS.containsKey(taskId)) {
            return MY_TASKS.get(taskId);
        } else {
            throw new TaskNotFoundProblem(taskId);
        }
    }

    @PutMapping("/{id}")
    public void updateTask(@PathVariable("id") Long id) {
        throw new UnsupportedOperationException();
    }

    @DeleteMapping("/{id}")
    public void deleteTask(@PathVariable("id") Long id) {
        throw new AccessDeniedException("You can't delete this task");
    }

}
```

在这个控制器中，我们有意抛出一些异常。那些异常将被自动转换成`Problem`对象，以产生一个包含失败细节的`application/problem+json`响应。

现在，让我们讨论一下内置的通知特征，以及如何创建一个定制的`Problem`实现。

## 7.内在的建议特征

建议特征是一个小的异常处理程序，它捕捉异常并返回正确的问题对象。

常见异常有内置的建议特征。因此，我们可以通过抛出异常来使用它们:

```
throw new UnsupportedOperationException();
```

结果，我们会得到这样的回应:

```
{
    "title": "Not Implemented",
    "status": 501
}
```

因为我们也配置了与 Spring Security 的集成，所以我们能够抛出与安全相关的异常:

```
throw new AccessDeniedException("You can't delete this task");
```

并得到适当的回应:

```
{
    "title": "Forbidden",
    "status": 403,
    "detail": "You can't delete this task"
}
```

## 8.创建自定义`Problem`

可以创建一个`Problem`的定制实现。我们只需要扩展`AbstractThrowableProblem`类:

```
public class TaskNotFoundProblem extends AbstractThrowableProblem {

    private static final URI TYPE
      = URI.create("https://example.org/not-found");

    public TaskNotFoundProblem(Long taskId) {
        super(
          TYPE,
          "Not found",
          Status.NOT_FOUND,
          String.format("Task '%s' not found", taskId));
    }

}
```

而且我们可以抛出我们的自定义问题如下:

```
if (MY_TASKS.containsKey(taskId)) {
    return MY_TASKS.get(taskId);
} else {
    throw new TaskNotFoundProblem(taskId);
}
```

抛出`TaskNotFoundProblem`问题的结果是，我们会得到:

```
{
    "type": "https://example.org/not-found",
    "title": "Not found",
    "status": 404,
    "detail": "Task '3' not found"
}
```

## 9.处理堆栈跟踪

如果我们想在响应中包含堆栈跟踪，我们需要相应地配置我们的`ProblemModule`:

```
ObjectMapper mapper = new ObjectMapper()
  .registerModule(new ProblemModule().withStackTraces());
```

默认情况下，原因的因果链是禁用的，但我们可以通过覆盖以下行为轻松启用它:

```
@ControllerAdvice
class ExceptionHandling implements ProblemHandling {

    @Override
    public boolean isCausalChainsEnabled() {
        return true;
    }

}
```

启用这两个特性后，我们将得到与此类似的响应:

```
{
  "title": "Internal Server Error",
  "status": 500,
  "detail": "Illegal State",
  "stacktrace": [
    "org.example.ExampleRestController
      .newIllegalState(ExampleRestController.java:96)",
    "org.example.ExampleRestController
      .nestedThrowable(ExampleRestController.java:91)"
  ],
  "cause": {
    "title": "Internal Server Error",
    "status": 500,
    "detail": "Illegal Argument",
    "stacktrace": [
      "org.example.ExampleRestController
        .newIllegalArgument(ExampleRestController.java:100)",
      "org.example.ExampleRestController
        .nestedThrowable(ExampleRestController.java:88)"
    ],
    "cause": {
      // ....
    }
  }
}
```

## 10.结论

在本文中，我们探讨了如何使用 Problem Spring Web library 通过使用`application/problem+json`响应来创建包含错误细节的响应。我们还学习了如何在我们的 Spring Boot 应用程序中配置库，并创建一个`Problem`对象的定制实现。

该指南的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行它。