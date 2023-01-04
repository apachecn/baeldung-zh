# " HttpMessageNotWritableException:未找到类型为的返回值的转换器"

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-no-converter-found>

## 1.概观

在本教程中，我们将阐明 Spring 的`HttpMessageNotWritableException: “No converter found for return value of type”`异常。

首先，我们将解释异常的主要原因。然后，我们将更深入地研究如何使用真实世界的例子来产生它，最后如何修复它。

## 2.原因

通常，当 Spring 无法获取返回对象的属性时，会出现这种异常。

这个异常最典型的原因通常是**返回的对象没有任何针对其属性**的公共 getter 方法。

默认情况下，Spring Boot 依赖于 [Jackson 库](/web/20220626072337/https://www.baeldung.com/jackson)来完成序列化/反序列化请求和响应对象的所有繁重工作。

因此，我们异常的另一个**常见原因可能是丢失或使用了错误的 Jackson 依赖关系**。

简而言之，这种异常的一般准则是检查是否存在:

*   默认构造函数
*   吸气剂
*   杰克逊属地

请记住[异常类型](https://web.archive.org/web/20220626072337/https://github.com/spring-projects/spring-framework/commit/c32299f734521e907585de50d70a46dcd8018f13#diff-ea43e736983102ff93143d5a8e5a0e63837233bafa3a5f8bae78256211ed9113)已经从`java.lang.IllegalArgumentException` 变为`org.springframework.http.converter.HttpMessageNotWritableException.`

## 3.实际例子

现在，让我们看一个生成`org.springframework.http.converter.HttpMessageNotWritableException`的例子:“没有为类型的返回值找到转换器”。

为了展示一个真实的用例，我们将使用 [Spring Boot](/web/20220626072337/https://www.baeldung.com/spring-boot) 为学生管理构建一个基本的 REST API。

首先，让我们**创建我们的模型类`Student`，并假装忘记生成 getter 方法**:

```
public class Student {

    private int id;
    private String firstName;
    private String lastName;
    private String grade;

    public Student() {
    }

    public Student(int id, String firstName, String lastName, String grade) {
	this.id = id;
	this.firstName = firstName;
	this.lastName = lastName;
	this.grade = grade;
    }

    // Setters
}
```

其次，我们将创建一个[弹簧控制器](/web/20220626072337/https://www.baeldung.com/spring-controllers)，用一个单独的处理程序方法通过对象的`id`来检索一个`Student`对象:

```
@RestController
@RequestMapping(value = "/api")
public class StudentRestController {

    @GetMapping("/student/{id}")
    public ResponseEntity<Student> get(@PathVariable("id") int id) {
        // Custom logic
        return ResponseEntity.ok(new Student(id, "John", "Wiliams", "AA"));
     }
}
```

现在，如果我们使用 [CURL](/web/20220626072337/https://www.baeldung.com/curl-rest) 向`http://localhost:8080/api/student/1`发送请求:

```
curl http://localhost:8080/api/student/1
```

端点将发回以下响应:

```
{"timestamp":"2021-02-14T14:54:19.426+00:00","status":500,"error":"Internal Server Error","message":"","path":"/api/student/1"}
```

看着日志，太春扔出了`HttpMessageNotWritableException`:

```
[org.springframework.http.converter.HttpMessageNotWritableException: No converter found for return value of type: class com.baeldung.boot.noconverterfound.model.Student]
```

最后，让我们创建一个测试用例，看看当 getter 方法没有在`Student`类中定义时，Spring 的行为如何:

```
@RunWith(SpringRunner.class)
@WebMvcTest(StudentRestController.class)
public class NoConverterFoundIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenGettersNotDefined_thenThrowException() throws Exception {

        String url = "/api/student/1";

	this.mockMvc.perform(get(url))
	  .andExpect(status().isInternalServerError())
	  .andExpect(result -> assertThat(result.getResolvedException())
            .isInstanceOf(HttpMessageNotWritableException.class))
	  .andExpect(result -> assertThat(result.getResolvedException().getMessage())
	    .contains("No converter found for return value of type"));
    }
}
```

## 4.**解决方案**

防止异常的最常见的解决方案之一**是为我们想要在 JSON 中返回的每个对象的属性定义一个 getter 方法。**

因此，让我们在`Student`类中添加 getter 方法，并创建一个新的测试用例来验证是否一切都将按预期工作:

```
@Test
public void whenGettersAreDefined_thenReturnObject() throws Exception {

    String url = "/api/student/2";

    this.mockMvc.perform(get(url))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$.firstName").value("John"));
}
```

一个不明智的解决方案是将房产公之于众。然而，这不是 100%安全的方法，因为它违背了一些最佳实践。

## 5.结论

在这篇短文中，我们解释了导致 Spring 抛出`org.springframework.http.converter.HttpMessageNotWritableException: ” No converter found for return value of type”`的原因。

然后，我们讨论了如何产生异常以及如何在实践中解决它。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626072337/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2)