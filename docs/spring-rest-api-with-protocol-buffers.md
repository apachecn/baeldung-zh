# 带协议缓冲区的 Spring REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-api-with-protocol-buffers>

## 1。概述

[Protocol Buffers](https://web.archive.org/web/20220707143825/https://developers.google.com/protocol-buffers/) 是一种语言和平台中立的机制，用于结构化数据的序列化和反序列化，它的创建者 Google 宣称它比其他类型的有效载荷(如 XML 和 JSON)更快、更小、更简单。

本教程将指导您设置 REST API，以利用这种基于二进制的消息结构。

## 2。协议缓冲区

本节给出了一些关于协议缓冲区的基本信息，以及它们是如何在 Java 生态系统中应用的。

### 2.1。协议缓冲器介绍

为了利用协议缓冲区，我们需要在`.proto`文件中定义消息结构。每个文件都是对可能从一个节点传输到另一个节点或存储在数据源中的数据的描述。这里有一个`.proto`文件的例子，它被命名为`baeldung.proto`，位于`src/main/resources`目录中。本教程稍后将使用该文件:

```java
syntax = "proto3";
package baeldung;
option java_package = "com.baeldung.protobuf";
option java_outer_classname = "BaeldungTraining";

message Course {
    int32 id = 1;
    string course_name = 2;
    repeated Student student = 3;
}
message Student {
    int32 id = 1;
    string first_name = 2;
    string last_name = 3;
    string email = 4;
    repeated PhoneNumber phone = 5;
    message PhoneNumber {
        string number = 1;
        PhoneType type = 2;
    }
    enum PhoneType {
        MOBILE = 0;
        LANDLINE = 1;
    }
}
```

在本教程中，**我们使用协议缓冲编译器和协议缓冲语言**的版本 3，因此`.proto`文件必须以`syntax = “proto3”`声明开始。如果正在使用编译器版本 2，则可以省略该声明。接下来是`package`声明，它是这个消息结构的名称空间，以避免与其他项目的命名冲突。

下面的两个声明只用于 Java:`java_package`选项指定我们生成的类所在的包，`java_outer_classname`选项表示包含这个`.proto`文件中定义的所有类型的类的名称。

下面的 2.3 小节将描述剩余的元素以及如何将它们编译成 Java 代码。

### 2.2。Java 协议缓冲区

在定义了消息结构之后，我们需要一个编译器来将这种语言无关的内容转换成 Java 代码。你可以按照[协议缓冲库](https://web.archive.org/web/20220707143825/https://github.com/google/protobuf)中的说明来获得合适的编译器版本。或者，您可以通过搜索 [`com.google.protobuf:protoc`](https://web.archive.org/web/20220707143825/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.protobuf%22%20AND%20a%3A%22protoc%22) 工件，从 Maven central repository 下载一个预构建的二进制编译器，然后选择一个适合您的平台的版本。

接下来，将编译器复制到项目的`src/main`目录中，并在命令行中执行以下命令:

```java
protoc --java_out=java resources/baeldung.proto
```

这将为`com.baeldung.protobuf`包中的`BaeldungTraining`类生成一个源文件，正如在`baeldung.proto`文件的`option`声明中所指定的。

除了编译器之外，还需要协议缓冲运行时。这可以通过向 Maven POM 文件添加以下依赖项来实现:

```java
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.0.0-beta-3</version>
</dependency>
```

我们可以使用运行时的另一个版本，只要它与编译器的版本相同。最新的，请查看[这个链接](https://web.archive.org/web/20220707143825/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.protobuf%22%20AND%20a%3A%22protobuf-java%22)。

### 2.3。编写消息描述

通过使用编译器，`.proto`文件中的消息被编译成静态嵌套 Java 类。在上面的例子中，`Course`和`Student`消息分别被转换为`Course`和`Student` Java 类。同时，消息的字段被编译成 JavaBeans 风格的 getterss 和 setterss，这些 getter 和 setter 位于这些生成的类型中。每个字段声明末尾的标记由一个等号和一个数字组成，是用于以二进制形式编码相关字段的唯一标记。

我们将遍历消息的类型化字段，看看它们是如何被转换成访问器方法的。

让我们从`Course`消息开始。它有两个简单的字段，包括`id`和`course_name`。它们的协议缓冲区类型`int32`和`string`被翻译成 Java `int`和`String`类型。以下是编译后它们相关的 getters(为简洁起见，省略了实现):

```java
public int getId();
public java.lang.String getCourseName();
```

请注意，类型化字段的名称应该是蛇形的(单个单词用下划线字符分隔)，以保持与其他语言的协作。编译器会根据 Java 约定将这些名称转换成 camel 大小写。

`Course`消息的最后一个字段`student`是`Student`复杂类型，这将在下面描述。该字段由关键字`repeated`前置，这意味着它可以重复任意次。编译器生成一些与`student`字段相关的方法，如下所示(没有实现):

```java
public java.util.List<com.baeldung.protobuf.BaeldungTraining.Student> getStudentList();
public int getStudentCount();
public com.baeldung.protobuf.BaeldungTraining.Student getStudent(int index);
```

现在我们将继续讨论`Student`消息，它被用作`Course`消息的`student`字段的复杂类型。它的简单字段包括`id`、`first_name`、`last_name`和`email`用于创建 Java 访问器方法:

```java
public int getId();
public java.lang.String getFirstName();
public java.lang.String getLastName();
public java.lang.String.getEmail();
```

最后一个字段`phone`属于`PhoneNumber`复杂类型。类似于`Course`消息的`student`字段，该字段是重复的，有几个相关的方法:

```java
public java.util.List<com.baeldung.protobuf.BaeldungTraining.Student.PhoneNumber> getPhoneList();
public int getPhoneCount();
public com.baeldung.protobuf.BaeldungTraining.Student.PhoneNumber getPhone(int index);
```

`PhoneNumber`消息被编译成`BaeldungTraining.Student.PhoneNumber`嵌套类型，有两个 getters 对应于消息的字段:

```java
public java.lang.String getNumber();
public com.baeldung.protobuf.BaeldungTraining.Student.PhoneType getType();
```

`PhoneNumber`消息的*类型*字段的复杂类型`PhoneType`，是一个枚举类型，将被转换成嵌套在`BaeldungTraining.Student`类中的 Java `enum`类型:

```java
public enum PhoneType implements com.google.protobuf.ProtocolMessageEnum {
    MOBILE(0),
    LANDLINE(1),
    UNRECOGNIZED(-1),
    ;
    // Other declarations
}
```

## 3。Spring REST API 中的 proto buf

本节将指导您使用 Spring Boot 设置 REST 服务。

### 3.1。Bean 声明

让我们从我们主要的`@SpringBootApplication`的定义开始:

```java
@SpringBootApplication
public class Application {
    @Bean
    ProtobufHttpMessageConverter protobufHttpMessageConverter() {
        return new ProtobufHttpMessageConverter();
    }

    @Bean
    public CourseRepository createTestCourses() {
        Map<Integer, Course> courses = new HashMap<>();
        Course course1 = Course.newBuilder()
          .setId(1)
          .setCourseName("REST with Spring")
          .addAllStudent(createTestStudents())
          .build();
        Course course2 = Course.newBuilder()
          .setId(2)
          .setCourseName("Learn Spring Security")
          .addAllStudent(new ArrayList<Student>())
          .build();
        courses.put(course1.getId(), course1);
        courses.put(course2.getId(), course2);
        return new CourseRepository(courses);
    }

    // Other declarations
}
```

`ProtobufHttpMessageConverter` bean 用于将由`@RequestMapping`带注释的方法返回的响应转换成协议缓冲区消息。

另一个 bean`CourseRepository`，包含我们的 API 的一些测试数据。

这里重要的是，我们使用的是**协议缓冲区特定的数据，而不是标准的 POJO**。

下面是`CourseRepository`的简单实现:

```java
public class CourseRepository {
    Map<Integer, Course> courses;

    public CourseRepository (Map<Integer, Course> courses) {
        this.courses = courses;
    }

    public Course getCourse(int id) {
        return courses.get(id);
    }
}
```

### 3.2。控制器配置

我们可以为测试 URL 定义`@Controller`类，如下所示:

```java
@RestController
public class CourseController {
    @Autowired
    CourseRepository courseRepo;

    @RequestMapping("/courses/{id}")
    Course customer(@PathVariable Integer id) {
        return courseRepo.getCourse(id);
    }
}
```

同样，这里重要的一点是，我们从控制器层返回的课程 DTO 不是标准的 POJO。这将是它在被传输回客户端之前被转换为协议缓冲区消息的触发器。

## 4。REST 客户端和测试

既然我们已经看了简单的 API 实现——现在让我们用两种方法来说明客户端上协议缓冲区消息的**反序列化。**

第一种利用带有预配置的`ProtobufHttpMessageConverter` bean 的`RestTemplate` API 来自动转换消息。

第二种是使用`protobuf-java-format`将协议缓冲区响应手动转换成 JSON 文档。

首先，我们需要设置集成测试的上下文，并指示 Spring Boot 通过如下声明一个测试类来查找`Application`类中的配置信息:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebIntegrationTest
public class ApplicationTest {
    // Other declarations
}
```

本节中的所有代码片段都将放在`ApplicationTest`类中。

### 4.1。预期响应

访问 REST 服务的第一步是确定请求 URL:

```java
private static final String COURSE1_URL = "http://localhost:8080/courses/1";
```

这个`COURSE1_URL`将用于从我们之前创建的 REST 服务中获取第一个测试双课程。将 GET 请求发送到上述 URL 后，使用以下断言验证相应的响应:

```java
private void assertResponse(String response) {
    assertThat(response, containsString("id"));
    assertThat(response, containsString("course_name"));
    assertThat(response, containsString("REST with Spring"));
    assertThat(response, containsString("student"));
    assertThat(response, containsString("first_name"));
    assertThat(response, containsString("last_name"));
    assertThat(response, containsString("email"));
    assertThat(response, containsString("[[email protected]](/web/20220707143825/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    assertThat(response, containsString("[[email protected]](/web/20220707143825/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    assertThat(response, containsString("[[email protected]](/web/20220707143825/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    assertThat(response, containsString("phone"));
    assertThat(response, containsString("number"));
    assertThat(response, containsString("type"));
}
```

在接下来的小节中，我们将在两个测试用例中使用这个助手方法。

### 4.2。用`RestTemplate`测试

下面是我们如何创建一个客户端，向指定的目的地发送一个 GET 请求，接收协议缓冲区消息形式的响应，并使用`RestTemplate` API 验证它:

```java
@Autowired
private RestTemplate restTemplate;

@Test
public void whenUsingRestTemplate_thenSucceed() {
    ResponseEntity<Course> course = restTemplate.getForEntity(COURSE1_URL, Course.class);
    assertResponse(course.toString());
}
```

为了让这个测试用例工作，我们需要在配置类中注册一个`RestTemplate`类型的 bean:

```java
@Bean
RestTemplate restTemplate(ProtobufHttpMessageConverter hmc) {
    return new RestTemplate(Arrays.asList(hmc));
}
```

还需要另一个`ProtobufHttpMessageConverter`类型的 bean 来自动转换接收到的协议缓冲区消息。该 bean 与第 3.1 小节中定义的 bean 相同。由于在本教程中客户机和服务器共享相同的应用程序上下文，我们可以在`Application`类中声明`RestTemplate` bean 并重用`ProtobufHttpMessageConverter` bean。

### 4.3。用`HttpClient`测试

使用`HttpClient` API 并手动转换协议缓冲区消息的第一步是向 Maven POM 文件添加以下两个依赖项:

```java
<dependency>
    <groupId>com.googlecode.protobuf-java-format</groupId>
    <artifactId>protobuf-java-format</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

对于这些依赖项的最新版本，请查看 Maven 中央存储库中的 [protobuf-java-format](https://web.archive.org/web/20220707143825/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.googlecode.protobuf-java-format%22%20AND%20a%3A%22protobuf-java-format%22) 和 [httpclient](https://web.archive.org/web/20220707143825/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.httpcomponents%22%20AND%20a%3A%22httpclient%22) 构件。

让我们继续创建一个客户机，执行一个 GET 请求，并使用给定的 URL 将相关的响应转换成一个`InputStream`实例:

```java
private InputStream executeHttpRequest(String url) throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    HttpGet request = new HttpGet(url);
    HttpResponse httpResponse = httpClient.execute(request);
    return httpResponse.getEntity().getContent();
}
```

现在，我们将以`InputStream`对象的形式将协议缓冲区消息转换成 JSON 文档:

```java
private String convertProtobufMessageStreamToJsonString(InputStream protobufStream) throws IOException {
    JsonFormat jsonFormat = new JsonFormat();
    Course course = Course.parseFrom(protobufStream);
    return jsonFormat.printToString(course);
}
```

下面是一个测试用例如何使用上面声明的私有 helper 方法并验证响应:

```java
@Test
public void whenUsingHttpClient_thenSucceed() throws IOException {
    InputStream responseStream = executeHttpRequest(COURSE1_URL);
    String jsonOutput = convertProtobufMessageStreamToJsonString(responseStream);
    assertResponse(jsonOutput);
}
```

### 4.4。JSON 中的响应

为了清楚起见，这里包括了我们在前面小节中描述的测试中收到的 JSON 形式的响应:

```java
id: 1
course_name: "REST with Spring"
student {
    id: 1
    first_name: "John"
    last_name: "Doe"
    email: "[[email protected]](/web/20220707143825/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    phone {
        number: "123456"
    }
}
student {
    id: 2
    first_name: "Richard"
    last_name: "Roe"
    email: "[[email protected]](/web/20220707143825/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    phone {
        number: "234567"
        type: LANDLINE
    }
}
student {
    id: 3
    first_name: "Jane"
    last_name: "Doe"
    email: "[[email protected]](/web/20220707143825/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    phone {
        number: "345678"
    }
    phone {
        number: "456789"
        type: LANDLINE
    }
}
```

## 5。结论

本教程快速介绍了协议缓冲区，并举例说明了使用 Spring 格式设置 REST API。然后，我们转向客户端支持和序列化-反序列化机制。

所有示例和代码片段的实现可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20220707143825/https://github.com/eugenp/tutorials/tree/master/spring-protobuf)