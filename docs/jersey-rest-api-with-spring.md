# 带运动衫和弹簧的 REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-rest-api-with-spring>

## 1。概述

Jersey 是一个开发 RESTful Web 服务的开源框架。它是 JAX 遥感系统的参考实施。

在本文中，**我们将探索使用 Jersey 2** 创建 RESTful Web 服务。此外，我们将使用 Spring 的依赖注入(DI)和 Java 配置。

## 2。Maven 依赖关系

让我们从向`pom.xml`添加依赖项开始:

```java
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet</artifactId>
    <version>2.26</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.26</version>
</dependency>
```

此外，对于 Spring 集成，我们必须添加`jersey-spring4`依赖项:

```java
<dependency>
    <groupId>org.glassfish.jersey.ext</groupId>
    <artifactId>jersey-spring4</artifactId>
    <version>2.26</version>
</dependency>
```

这些依赖项的最新版本可以在 [jersey-container-servlet](https://web.archive.org/web/20220913185122/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.containers%22%20AND%20a%3A%22jersey-container-servlet%22) 、[jersey-media-JSON-Jackson](https://web.archive.org/web/20220913185122/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.media%22%20AND%20a%3A%22jersey-media-json-jackson%22)和 [jersey-spring4](https://web.archive.org/web/20220913185122/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jersey-spring4%22) 获得。

## 3。网络配置

**接下来，我们需要设置一个 web 项目来做 Servlet 配置。**为此，我们将使用 Spring 的`WebApplicationInitializer`:

```java
@Order(Ordered.HIGHEST_PRECEDENCE)
public class ApplicationInitializer 
  implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) 
      throws ServletException {

        AnnotationConfigWebApplicationContext context 
          = new AnnotationConfigWebApplicationContext();

        servletContext.addListener(new ContextLoaderListener(context));
        servletContext.setInitParameter(
          "contextConfigLocation", "com.baeldung.server");
    }
}
```

这里，我们添加了`@Order(Ordered.HIGHEST_PRECEDENCE)` 注释，以确保我们的初始化器在 Jersey-Spring 默认初始化器之前执行。

## 4。使用新泽西 JAX RS 的服务

### 4.1。资源表示类

让我们使用一个示例资源表示类:

```java
@XmlRootElement
public class Employee {
    private int id;
    private String firstName;

    // standard getters and setters
}
```

注意，像`@XmlRootElement`这样的 JAXB 注释只有在需要 XML 支持时才是必需的(除了 JSON 之外)。

### 4.2。服务实施

现在让我们看看如何使用 JAX-RS 注释来创建 RESTful web 服务:

```java
@Path("/employees")
public class EmployeeResource {

    @Autowired
    private EmployeeRepository employeeRepository;

    @GET
    @Path("/{id}")
    @Produces({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    public Employee getEmployee(@PathParam("id") int id) {
        return employeeRepository.getEmployee(id);
    }

    @POST
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    public Response addEmployee(
      Employee employee, @Context UriInfo uriInfo) {

        employeeRepository.addEmployee(new Employee(employee.getId(), 
          employee.getFirstName(), employee.getLastName(), 
          employee.getAge()));

        return Response.status(Response.Status.CREATED.getStatusCode())
          .header(
            "Location", 
            String.format("%s/%s",uriInfo.getAbsolutePath().toString(), 
            employee.getId())).build();
    }
}
```

**`@Path`注释提供了服务的相对 URI 路径。**我们也可以在 URI 语法中嵌入变量，如`{id}` 变量所示。然后，变量将在运行时被替换。要获得变量的值，我们可以使用`@PathParam`注释。

***@GET* 、 *@PUT* 、@POST、 *@DELETE* 和 *@HEAD* 定义了请求**的 HTTP 方法，这些方法将被带注释的方法处理。

**`@Produces`注释定义了端点的响应类型** (MIME 媒体类型)。在我们的例子中，我们将它配置为根据 HTTP 头`Accept` ( `application/json`或`application/xml`)的值返回 JSON 或 XML。

**另一方面， `the @Consumes`注解定义了服务可以使用的 MIME 媒体类型。**在我们的例子中，服务可以使用 JSON 或 XML，这取决于 HTTP 头`Content-Type` ( `application/json`或`application/xml`)。

`@Context`注释用于将信息注入类字段、bean 属性或方法参数。在我们的例子中，我们用它来注入`UriInfo`。我们也可以用它来注射`ServletConfig`、`ServletContext`、`HttpServletRequest`和`HttpServletResponse.`

## 5。使用`ExceptionMapper`

`ExceptionMapper`允许我们拦截异常并向客户端返回适当的 HTTP 响应代码。在下面的示例中，如果引发了`EmployeeNotFound`异常，则返回 HTTP 响应代码 404:

```java
@Provider
public class NotFoundExceptionHandler 
  implements ExceptionMapper<EmployeeNotFound> {

    public Response toResponse(EmployeeNotFound ex) {
        return Response.status(Response.Status.NOT_FOUND).build();
    }
}
```

## 6。管理资源类

最后，**让我们将所有服务实现类和异常映射器连接到一个应用程序路径:**

```java
@ApplicationPath("/resources")
public class RestConfig extends Application {
    public Set<Class<?>> getClasses() {
        return new HashSet<Class<?>>(
          Arrays.asList(
            EmployeeResource.class, 
            NotFoundExceptionHandler.class, 
            AlreadyExistsExceptionHandler.class));
    }
}
```

## 7 .**。API 测试**

现在让我们用一些实时测试来测试 API:

```java
public class JerseyApiLiveTest {

    private static final String SERVICE_URL
      = "http://localhost:8082/spring-jersey/resources/employees";

    @Test
    public void givenGetAllEmployees_whenCorrectRequest_thenResponseCodeSuccess() 
      throws ClientProtocolException, IOException {

        HttpUriRequest request = new HttpGet(SERVICE_URL);

        HttpResponse httpResponse = HttpClientBuilder
          .create()
          .build()
          .execute(request);

        assertEquals(httpResponse
          .getStatusLine()
          .getStatusCode(), HttpStatus.SC_OK);
    }
}
```

## 8。结论

在本文中，我们介绍了 Jersey 框架并开发了一个简单的 API。我们使用 Spring 来实现依赖注入特性。我们也看到了`ExceptionMapper`的用法。

和往常一样，完整的源代码可以在 Github 项目中获得。