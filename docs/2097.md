# Apache CXF 对 RESTful Web 服务的支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-cxf-rest-api>

## 1。概述

本教程介绍了作为符合 JAX-RS 标准的框架的 Apache CXF，该标准定义了 Java 生态系统对表述性状态转移(REST)架构模式的支持。

具体来说，它一步一步地描述了如何构建和发布 RESTful web 服务，以及如何编写单元测试来验证服务。

这是 Apache CXF 系列的第三篇；[第一个](/web/20221208143856/https://www.baeldung.com/introduction-to-apache-cxf)关注 CXF 作为 JAX-WS 完全兼容的实现的使用。[的第二篇文章](/web/20221208143856/https://www.baeldung.com/apache-cxf-with-spring)提供了如何使用 CXF 和 Spring 的指南。

## 2。Maven 依赖关系

第一个需要依赖的是`org.apache.cxf:cxf-rt-frontend-` `jaxrs`。这个工件提供了 JAX-RS API 以及一个 CXF 实现:

```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxrs</artifactId>
    <version>3.1.7</version>
</dependency>
```

在本教程中，我们使用 CXF 创建一个`Server`端点来发布 web 服务，而不是使用 servlet 容器。因此，Maven POM 文件中需要包含以下依赖项:

```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http-jetty</artifactId>
    <version>3.1.7</version>
</dependency>
```

最后，让我们添加 HttpClient 库来促进单元测试:

```
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

[在这里](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-frontend-jaxrs%22)你可以找到最新版本的`cxf-rt-frontend-jaxrs`依赖。你可能还想参考[这个链接](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-transports-http-jetty%22)来获得`org.apache.cxf:cxf-rt-transports-http-jetty`工件的最新版本。最后，`httpclient`的最新版本可以在[这里](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.httpcomponents%22%20AND%20a%3A%22httpclient%22)找到。

## 3。资源类别和请求映射

让我们开始实现一个简单的例子；我们将用两个资源`Course` 和`Student.` 来设置我们的 REST API

我们将从简单的开始，然后逐步走向更复杂的例子。

### 3.1。资源

下面是`Student`资源类的定义:

```
@XmlRootElement(name = "Student")
public class Student {
    private int id;
    private String name;

    // standard getters and setters
    // standard equals and hashCode implementations

}
```

注意，我们使用了`@XmlRootElement`注释来告诉 JAXB 这个类的实例应该被封送到 XML。

接下来是`Course`资源类的定义:

```
@XmlRootElement(name = "Course")
public class Course {
    private int id;
    private String name;
    private List<Student> students = new ArrayList<>();

    private Student findById(int id) {
        for (Student student : students) {
            if (student.getId() == id) {
                return student;
            }
        }
        return null;
    }
```

```
 // standard getters and setters
    // standard equals and hasCode implementations

}
```

最后，让我们实现 `CourseRepository`——它是根资源，充当 web 服务资源的入口点:

```
@Path("course")
@Produces("text/xml")
public class CourseRepository {
    private Map<Integer, Course> courses = new HashMap<>();

    // request handling methods

    private Course findById(int id) {
        for (Map.Entry<Integer, Course> course : courses.entrySet()) {
            if (course.getKey() == id) {
                return course.getValue();
            }
        }
        return null;
    }
}
```

注意带有`@Path`注释的映射。这里的`CourseRepository`是根资源，所以它被映射来处理所有以`course`开头的 URL。

`@Produces`注释的值用于告诉服务器在将对象发送给客户机之前，将从该类中的方法返回的对象转换成 XML 文档。我们在这里使用 JAXB 作为缺省值，因为没有指定其他绑定机制。

### 3.2。简单的数据设置

因为这是一个简单的示例实现，所以我们使用内存中的数据，而不是成熟的持久解决方案。

记住这一点，让我们实现一些简单的设置逻辑，将一些数据填充到系统中:

```
{
    Student student1 = new Student();
    Student student2 = new Student();
    student1.setId(1);
    student1.setName("Student A");
    student2.setId(2);
    student2.setName("Student B");

    List<Student> course1Students = new ArrayList<>();
    course1Students.add(student1);
    course1Students.add(student2);

    Course course1 = new Course();
    Course course2 = new Course();
    course1.setId(1);
    course1.setName("REST with Spring");
    course1.setStudents(course1Students);
    course2.setId(2);
    course2.setName("Learn Spring Security");

    courses.put(1, course1);
    courses.put(2, course2);
}
```

这个类中处理 HTTP 请求的方法将在下一小节中介绍。

### 3.3。API–请求映射方法

现在，让我们来看看实际的 REST API 的实现。

我们将开始添加 API 操作——使用`@Path`注释——就在资源 POJOs 中。

理解这一点很重要，这与典型的 Spring 项目中的方法有很大的不同——在 Spring 项目中，API 操作将在控制器中定义，而不是在 POJO 本身上定义。

让我们从在`Course`类中定义的映射方法开始:

```
@GET
@Path("{studentId}")
public Student getStudent(@PathParam("studentId")int studentId) {
    return findById(studentId);
}
```

简单地说，该方法在处理`GET`请求时被调用，由`@GET`注释表示。

注意到从 HTTP 请求映射`studentId`路径参数的简单语法。

然后我们简单地使用`findById`助手方法来返回相应的`Student`实例。

下面的方法通过将接收到的`Student`对象添加到`students`列表来处理由`@POST`注释指示的`POST`请求:

```
@POST
@Path("")
public Response createStudent(Student student) {
    for (Student element : students) {
        if (element.getId() == student.getId() {
            return Response.status(Response.Status.CONFLICT).build();
        }
    }
    students.add(student);
    return Response.ok(student).build();
}
```

如果创建操作成功，它将返回一个`200 OK`响应，或者如果带有提交的`id`的对象已经存在，则返回`409 Conflict`。

还要注意，我们可以跳过`@Path`注释，因为它的值是一个空字符串。

最后一个方法处理`DELETE`请求。它从`students`列表中删除一个元素，该元素的`id`是接收到的路径参数，并返回一个状态为`OK` (200)的响应。如果没有与指定的`id`相关联的元素，这意味着没有要移除的内容，则该方法返回具有`Not Found` (404)状态的响应:

```
@DELETE
@Path("{studentId}")
public Response deleteStudent(@PathParam("studentId") int studentId) {
    Student student = findById(studentId);
    if (student == null) {
        return Response.status(Response.Status.NOT_FOUND).build();
    }
    students.remove(student);
    return Response.ok().build();
}
```

让我们继续来请求`CourseRepository`类的映射方法。

下面的`getCourse`方法返回一个`Course`对象，它是`courses`地图中一个条目的值，它的键是接收到的`GET`请求的`courseId`路径参数。在内部，该方法将路径参数分派给`findById`助手方法来完成它的工作。

```
@GET
@Path("courses/{courseId}")
public Course getCourse(@PathParam("courseId") int courseId) {
    return findById(courseId);
}
```

下面的方法更新了`courses`映射的现有条目，其中接收到的`PUT`请求的主体是条目值，而`courseId`参数是相关的键:

```
@PUT
@Path("courses/{courseId}")
public Response updateCourse(@PathParam("courseId") int courseId, Course course) {
    Course existingCourse = findById(courseId);        
    if (existingCourse == null) {
        return Response.status(Response.Status.NOT_FOUND).build();
    }
    if (existingCourse.equals(course)) {
        return Response.notModified().build();    
    }
    courses.put(courseId, course);
    return Response.ok().build();
}
```

如果更新成功，这个`updateCourse`方法返回一个带有`OK` (200)状态的响应，不做任何改变，如果现有的和上传的对象具有相同的字段值，则返回一个`Not Modified` (304)响应。如果在`courses`图中没有找到具有给定`id`的`Course`实例，该方法返回具有`Not Found` (404)状态的响应。

这个根资源类的第三个方法不直接处理任何 HTTP 请求。相反，它将请求委托给`Course`类，在那里请求由匹配的方法处理:

```
@Path("courses/{courseId}/students")
public Course pathToStudent(@PathParam("courseId") int courseId) {
    return findById(courseId);
}
```

我们已经展示了`Course`类中处理委托请求的方法。

## 4。`Server`终点

本节重点介绍 CXF 服务器的构造，该服务器用于发布 RESTful web 服务，其资源在前一节中有所描述。第一步是实例化一个`JAXRSServerFactoryBean`对象并设置根资源类:

```
JAXRSServerFactoryBean factoryBean = new JAXRSServerFactoryBean();
factoryBean.setResourceClasses(CourseRepository.class);
```

然后需要在工厂 bean 上设置一个资源提供者来管理根资源类的生命周期。我们使用默认的单一资源提供者，它为每个请求返回相同的资源实例:

```
factoryBean.setResourceProvider(
  new SingletonResourceProvider(new CourseRepository()));
```

我们还设置了一个地址来表示发布 web 服务的 URL:

```
factoryBean.setAddress("http://localhost:8080/");
```

现在`factoryBean`可以用来创建一个新的`server`，它将开始监听传入的连接:

```
Server server = factoryBean.create();
```

本节中的所有代码都应该包装在`main`方法中:

```
public class RestfulServer {
    public static void main(String args[]) throws Exception {
        // code snippets shown above
    }
}
```

第 6 节介绍了这个`main`方法的调用。

## 5。测试用例

本节描述了用于验证我们之前创建的 web 服务的测试用例。这些测试在响应四种最常用方法的 HTTP 请求后验证服务的资源状态，这四种方法是`GET`、`POST`、`PUT`和`DELETE`。

### 5.1。准备工作

首先，在测试类中声明了两个静态字段，名为`RestfulTest`:

```
private static String BASE_URL = "http://localhost:8080/baeldung/courses/";
private static CloseableHttpClient client;
```

在运行测试之前，我们创建一个`client`对象，用于与服务器通信，并在之后销毁它:

```
@BeforeClass
public static void createClient() {
    client = HttpClients.createDefault();
}

@AfterClass
public static void closeClient() throws IOException {
    client.close();
}
```

`client`实例现在可以被测试用例使用了。

### 5.2。`GET`请求

在 test 类中，我们定义了两种方法来向运行 web 服务的服务器发送`GET`请求。

第一种方法是在给定资源中的`id`的情况下获取一个`Course`实例:

```
private Course getCourse(int courseOrder) throws IOException {
    URL url = new URL(BASE_URL + courseOrder);
    InputStream input = url.openStream();
    Course course
      = JAXB.unmarshal(new InputStreamReader(input), Course.class);
    return course;
}
```

第二个是获得一个`Student`实例，给出资源中的课程和学生的`id`:

```
private Student getStudent(int courseOrder, int studentOrder)
  throws IOException {
    URL url = new URL(BASE_URL + courseOrder + "/students/" + studentOrder);
    InputStream input = url.openStream();
    Student student
      = JAXB.unmarshal(new InputStreamReader(input), Student.class);
    return student;
}
```

这些方法向服务资源发送 HTTP `GET`请求，然后将 XML 响应解组到相应类的实例。两者都用于在执行`POST`、`PUT`和`DELETE`请求后验证服务资源状态。

### 5.3。`POST`请求

这一小节以两个针对`POST`请求的测试案例为特色，说明了当上传的`Student`实例导致冲突时以及当它被成功创建时 web 服务的操作。

在第一个测试中，我们使用了一个从`conflict_student.xml`文件解组的`Student`对象，它位于类路径中，包含以下内容:

```
<Student>
    <id>2</id>
    <name>Student B</name>
</Student>
```

这就是内容被转换成`POST`请求体的方式:

```
HttpPost httpPost = new HttpPost(BASE_URL + "1/students");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("conflict_student.xml");
httpPost.setEntity(new InputStreamEntity(resourceStream));
```

设置`Content-Type`头是为了告诉服务器请求的内容类型是 XML:

```
httpPost.setHeader("Content-Type", "text/xml");
```

由于上传的`Student`对象已经存在于第一个`Course`实例中，我们预计创建会失败，并返回一个状态为`Conflict` (409)的响应。以下代码片段验证了预期:

```
HttpResponse response = client.execute(httpPost);
assertEquals(409, response.getStatusLine().getStatusCode());
```

在下一个测试中，我们从名为`created_student.xml`的文件中提取 HTTP 请求的主体，这个文件也在类路径中。以下是该文件的内容:

```
<Student>
    <id>3</id>
    <name>Student C</name>
</Student>
```

与前面的测试案例类似，我们构建并执行一个请求，然后验证一个新的实例是否成功创建:

```
HttpPost httpPost = new HttpPost(BASE_URL + "2/students");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("created_student.xml");
httpPost.setEntity(new InputStreamEntity(resourceStream));
httpPost.setHeader("Content-Type", "text/xml");

HttpResponse response = client.execute(httpPost);
assertEquals(200, response.getStatusLine().getStatusCode());
```

我们可以确认 web 服务资源的新状态:

```
Student student = getStudent(2, 3);
assertEquals(3, student.getId());
assertEquals("Student C", student.getName());
```

下面是对新的`Student`对象请求的 XML 响应:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Student>
    <id>3</id>
    <name>Student C</name>
</Student>
```

### 5.4。`PUT`请求

让我们从一个无效的更新请求开始，这里被更新的`Course`对象不存在。以下是用于替换 web 服务资源中不存在的`Course`对象的实例的内容:

```
<Course>
    <id>3</id>
    <name>Apache CXF Support for RESTful</name>
</Course>
```

该内容存储在类路径上名为`non_existent_course.xml`的文件中。它被提取出来，然后用下面的代码填充一个`PUT`请求的主体:

```
HttpPut httpPut = new HttpPut(BASE_URL + "3");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("non_existent_course.xml");
httpPut.setEntity(new InputStreamEntity(resourceStream));
```

设置`Content-Type`头是为了告诉服务器请求的内容类型是 XML:

```
httpPut.setHeader("Content-Type", "text/xml");
```

因为我们故意发送了一个无效的请求来更新一个不存在的对象，所以预期会收到一个`Not Found` (404)响应。响应得到验证:

```
HttpResponse response = client.execute(httpPut);
assertEquals(404, response.getStatusLine().getStatusCode());
```

在第二个针对`PUT`请求的测试案例中，我们提交了一个具有相同字段值的`Course`对象。因为在这种情况下什么都没有改变，我们期望返回一个状态为`Not Modified` (304)的响应。整个过程举例说明如下:

```
HttpPut httpPut = new HttpPut(BASE_URL + "1");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("unchanged_course.xml");
httpPut.setEntity(new InputStreamEntity(resourceStream));
httpPut.setHeader("Content-Type", "text/xml");

HttpResponse response = client.execute(httpPut);
assertEquals(304, response.getStatusLine().getStatusCode());
```

其中`unchanged_course.xml`是类路径上保存用于更新的信息的文件。以下是它的内容:

```
<Course>
    <id>1</id>
    <name>REST with Spring</name>
</Course>
```

在最后一个对`PUT`请求的演示中，我们执行了一个有效的更新。以下是`changed_course.xml`文件的内容，其内容用于更新 web 服务资源中的`Course`实例:

```
<Course>
    <id>2</id>
    <name>Apache CXF Support for RESTful</name>
</Course>
```

请求是这样构建和执行的:

```
HttpPut httpPut = new HttpPut(BASE_URL + "2");
InputStream resourceStream = this.getClass().getClassLoader()
  .getResourceAsStream("changed_course.xml");
httpPut.setEntity(new InputStreamEntity(resourceStream));
httpPut.setHeader("Content-Type", "text/xml");
```

让我们验证对服务器的`PUT`请求，并验证成功的上传:

```
HttpResponse response = client.execute(httpPut);
assertEquals(200, response.getStatusLine().getStatusCode());
```

让我们验证 web 服务资源的新状态:

```
Course course = getCourse(2);
assertEquals(2, course.getId());
assertEquals("Apache CXF Support for RESTful", course.getName());
```

下面的代码片段显示了发送对之前上传的`Course`对象的 GET 请求时 XML 响应的内容:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Course>
    <id>2</id>
    <name>Apache CXF Support for RESTful</name>
</Course>
```

### 5.5。`DELETE`请求

首先，让我们尝试删除一个不存在的`Student`实例。操作应该会失败，并且预期会有一个具有`Not Found` (404)状态的相应响应:

```
HttpDelete httpDelete = new HttpDelete(BASE_URL + "1/students/3");
HttpResponse response = client.execute(httpDelete);
assertEquals(404, response.getStatusLine().getStatusCode());
```

在针对`DELETE`请求的第二个测试用例中，我们创建、执行并验证一个请求:

```
HttpDelete httpDelete = new HttpDelete(BASE_URL + "1/students/1");
HttpResponse response = client.execute(httpDelete);
assertEquals(200, response.getStatusLine().getStatusCode());
```

我们使用以下代码片段验证 web 服务资源的新状态:

```
Course course = getCourse(1);
assertEquals(1, course.getStudents().size());
assertEquals(2, course.getStudents().get(0).getId());
assertEquals("Student B", course.getStudents().get(0).getName());
```

接下来，我们列出在请求 web 服务资源中的第一个`Course`对象之后收到的 XML 响应:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Course>
    <id>1</id>
    <name>REST with Spring</name>
    <students>
        <id>2</id>
        <name>Student B</name>
    </students>
</Course>
```

很明显，第一个`Student`已经成功移除。

## 6。测试执行

第 4 节描述了如何在`RestfulServer`类的`main`方法中创建和销毁一个`Server`实例。

启动并运行服务器的最后一步是调用那个`main`方法。为了实现这一点，在 Maven POM 文件中包含并配置了 Exec Maven 插件:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.0.0<version>
    <configuration>
        <mainClass>
          com.baeldung.cxf.jaxrs.implementation.RestfulServer
        </mainClass>
    </configuration>
</plugin>
```

这个插件的最新版本可以通过[这个链接](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.codehaus.mojo%22%20AND%20a%3A%22exec-maven-plugin%22)找到。

在编译和打包本教程中展示的工件的过程中，Maven Surefire 插件自动执行所有包含在名称以`Test`开头或结尾的类中的测试。如果是这种情况，插件应该被配置为排除那些测试:

```
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
    <excludes>
        <exclude>**/ServiceTest</exclude>
    </excludes>
    </configuration>
</plugin>
```

在上面的配置中， `ServiceTest`被排除，因为它是测试类的名称。您可以为该类选择任何名称，只要其中包含的测试在服务器准备好连接之前没有被 Maven Surefire 插件运行。

最新版本的 Maven Surefire 插件，请查看[这里](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22maven-surefire-plugin%22)。

现在您可以执行`exec:java`目标来启动 RESTful web 服务服务器，然后使用 IDE 运行上面的测试。同样，您可以通过在终端中执行命令 `mvn -Dtest=ServiceTest test`来开始测试。

## 7 .**。结论**

本教程演示了 Apache CXF 作为 JAX-RS 实现的使用。它演示了如何使用该框架来定义 RESTful web 服务的资源，以及如何创建用于发布服务的服务器。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。