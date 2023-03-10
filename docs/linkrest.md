# LinkRest 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/linkrest>

## 1。概述

[`LinkRest`](https://web.archive.org/web/20220627081048/https://github.com/nhl/link-rest) 是一个构建数据驱动的 REST web 服务的开源框架。它建立在 `JAX-RS`和`Apache Cayenne ORM`之上，使用基于 HTTP/JSON 的消息协议。

基本上，这个框架旨在提供一种在 web 上公开我们的数据存储的简单方法。

在接下来的小节中，我们将看看如何使用`LinkRest`构建 REST web 服务来访问数据模型。

## 2。`Maven`属地

要开始使用这个库，首先我们需要添加 [`link-rest`](https://web.archive.org/web/20220627081048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22link-rest%22) 依赖项:

```java
<dependency>
    <groupId>com.nhl.link.rest</groupId>
    <artifactId>link-rest</artifactId>
    <version>2.9</version>
</dependency>
```

这也带来了`cayenne-server`神器。

此外，我们将使用`Jersey`作为`JAX-RS`实现，因此我们需要添加`[jersey-container-servlet](https://web.archive.org/web/20220627081048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jersey-container-servlet%22)`依赖项，以及用于序列化 JSON 响应的 [`jersey-media-moxy`](https://web.archive.org/web/20220627081048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jersey-media-moxy%22) :

```java
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet</artifactId>
    <version>2.25.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-moxy</artifactId>
    <version>2.25.1</version>
</dependency>
```

对于我们的例子，我们将使用内存中的`H2`数据库，因为它更容易设置；因此，我们还要加上 [`h2`](https://web.archive.org/web/20220627081048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) :

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

## `3\. Cayenne`数据模型

我们将使用的数据模型包含一个代表一对多关系的`Department`和一个`Employee`实体:

[![tables](img/fb853f5de90bf00e85496183623e7378.png)](/web/20220627081048/https://www.baeldung.com/wp-content/uploads/2017/09/tables.png)

如前所述， **`LinkRest`处理使用`Apache Cayenne ORM`** 生成的数据对象。使用`Cayenne`不是本文的主题，所以要了解更多信息，请查看 [`Apache Cayenne`文档](https://web.archive.org/web/20220627081048/https://cayenne.apache.org/)。

我们将把`Cayenne`项目保存在一个`cayenne-linkrest-project.xml`文件中。

运行`cayenne-maven-plugin`之后，这将生成两个`_Department`和`_Employee`抽象类——它们将扩展`CayenneDataObject`类，以及从它们派生的两个具体类`Department`和`Employee`。

后面这些类是我们可以定制并使用`LinkRest`的类。

## 4。`LinkRest`应用启动

在下一节中，我们将编写和测试 REST 端点，因此为了能够运行它们，我们需要设置我们的运行时。

因为我们使用`Jersey`作为`JAX-RS`的实现，所以让我们添加一个扩展`ResourceConfig`的类，并指定包含定义 REST 端点的类的包:

```java
@ApplicationPath("/linkrest")
public class LinkRestApplication extends ResourceConfig {

    public LinkRestApplication() {
        packages("com.baeldung.linkrest.apis");

        // load linkrest runtime
    }
}
```

在同一个构造函数中，我们需要构建并注册`LinkRestRuntime`到`Jersey`容器。这个类是基于第一次装载`CayenneRuntime`:

```java
ServerRuntime cayenneRuntime = ServerRuntime.builder()
  .addConfig("cayenne-linkrest-project.xml")
  .build();
LinkRestRuntime lrRuntime = LinkRestBuilder.build(cayenneRuntime);
super.register(lrRuntime);
```

最后，我们需要将类添加到 `web.xml`:

```java
<servlet>
    <servlet-name>linkrest</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
        <init-param>
            <param-name>javax.ws.rs.Application</param-name>
            <param-value>com.baeldung.LinkRestApplication</param-value>
        </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>linkrest</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>
```

## 5。剩余资源

现在我们已经有了模型类，我们可以开始编写 REST 资源了。

使用标准的`JAX-RS`注释创建 **REST 端点，而使用`LinkRest`类构建响应。**

我们的例子将包括编写一系列使用不同 HTTP 方法访问`/department` URL 的 CRUD 端点。

首先，让我们创建`DepartmentResource`类，它被映射到`/department`:

```java
@Path("department")
@Produces(MediaType.APPLICATION_JSON)
public class DepartmentResource {

    @Context
    private Configuration config;

    // ...
}
```

`LinkRest`类需要一个`JAX-RS Configuration`类的实例，它是使用`Context`注释注入的，也是由`JAX-RS`提供的。

接下来，让我们继续编写访问`Department`对象的每个端点。

### 5.1。使用 POST 创建实体

为了创建一个实体，`LinkRest`类提供了返回一个`UpdateBuilder`对象的`create()`方法:

```java
@POST
public SimpleResponse create(String data) {
    return LinkRest.create(Department.class, config).sync(data);
}
```

数据参数可以是代表一个`Department`的单个 JSON 对象，也可以是一个对象数组。使用`sync()`方法将该参数发送给`UpdateBuilder`以创建一个或多个对象并将记录插入数据库，之后该方法返回一个`SimpleResponse`。

该库为响应定义了 3 种附加格式:

*   `DataResponse<T>`–代表`T`集合的响应
*   `MetadataResponse<T>`–包含关于类型的元数据信息
*   `SimpleResponse`–包含两个`success`和`message`属性的对象

接下来，让我们使用`curl`向数据库添加一条`Department`记录:

```java
curl -i -X POST -H "Content-Type:application/json" 
  -d "{"name":"IT"}" http://localhost:8080/linkrest/department
```

因此，该命令返回状态`201 Created`和一个`success`属性:

```java
{"success":true}
```

我们还可以通过发送一个 JSON 数组来创建多个对象:

```java
curl -i -X POST -H "Content-Type:application/json" 
  -d "[{"name":"HR"},{"name":"Marketing"}]" 
  http://localhost:8080/linkrest/department
```

### 5.2。使用 GET 读取实体

查询对象的主要方法是来自`LinkRest`类的`select()`方法。这返回了一个`SelectBuilder`对象，我们可以用它来链接附加的查询或过滤方法。

让我们在`DepartmentResource`类中创建一个端点，它返回数据库中的所有`Department`对象:

```java
@GET
public DataResponse<Department> getAll(@Context UriInfo uriInfo) {
    return LinkRest.select(Department.class, config).uri(uriInfo).get();
}
```

`uri()`调用为`SelectBuilder`设置请求信息，而 get()返回包装为`DataResponse<Department>`对象的`Departments`集合。

让我们看看在使用此端点之前我们添加的部门:

```java
curl -i -X GET http://localhost:8080/linkrest/department
```

响应采用 JSON 对象的形式，带有一个`data`数组和一个`total`属性:

```java
{"data":[
  {"id":200,"name":"IT"},
  {"id":201,"name":"Marketing"},
  {"id":202,"name":"HR"}
], 
"total":3}
```

或者，为了检索对象的集合，我们也可以通过使用 `getOne()`而不是`get()`来检索单个对象。

让我们添加一个映射到`/department/{departmentId}`的端点，它返回一个具有给定 id 的对象。为此，我们将使用`byId()`方法过滤记录:

```java
@GET
@Path("{id}")
public DataResponse<Department> getOne(@PathParam("id") int id, 
  @Context UriInfo uriInfo) {
    return LinkRest.select(Department.class, config)
      .byId(id).uri(uriInfo).getOne();
}
```

然后，我们可以向这个 URL 发送一个 GET 请求:

```java
curl -i -X GET http://localhost:8080/linkrest/department/200
```

结果是一个包含一个元素的`data`数组:

```java
{"data":[{"id":200,"name":"IT"}],"total":1}
```

### 5.3。使用 PUT 更新实体

为了更新记录，我们可以使用`update()`或`createOrUpdate()`方法。后者将更新记录(如果存在),或者创建记录(如果不存在):

```java
@PUT
public SimpleResponse createOrUpdate(String data) {
    return LinkRest.createOrUpdate(Department.class, config).sync(data);
}
```

与前面的部分类似，`data`参数可以是单个对象或一组对象。

让我们更新之前添加的一个部门:

```java
curl -i -X PUT -H "Content-Type:application/json" 
  -d "{"id":202,"name":"Human Resources"}" 
  http://localhost:8080/linkrest/department
```

这将返回一个带有成功或错误消息的 JSON 对象。之后，我们可以验证 ID 为 202 的部门名称是否被更改:

```java
curl -i -X GET http://localhost:8080/linkrest/department/202
```

果然，这个命令返回了具有新名称的对象:

```java
{"data":[
  {"id":202,"name":"Human Resources"}
],
"total":1}
```

### 5.4。使用`DELETE` 删除实体

并且，要删除一个对象，我们可以调用创建一个`DeleteBuilder`的`delete()`方法，然后使用`id()`方法指定我们想要删除的对象的主键:

```java
@DELETE
@Path("{id}")
public SimpleResponse delete(@PathParam("id") int id) {
    return LinkRest.delete(Department.class, config).id(id).delete();
}
```

然后我们可以使用`curl`调用这个端点:

```java
curl -i -X DELETE http://localhost:8080/linkrest/department/202
```

### 5.5。处理实体间的关系

`LinkRest`还包含了使处理对象间关系更容易的方法。

由于`Department`与`Employee`有一对多的关系，让我们添加一个访问`EmployeeSubResource`类的`/department/{departmentId}/employees`端点:

```java
@Path("{id}/employees")
public EmployeeSubResource getEmployees(
  @PathParam("id") int id, @Context UriInfo uriInfo) {
    return new EmployeeSubResource(id);
}
```

`EmployeeSubResource`类对应于一个部门，因此它将有一个设置部门 id 的构造函数，以及`Configuration`实例:

```java
@Produces(MediaType.APPLICATION_JSON)
public class EmployeeSubResource {
    private Configuration config;

    private int departmentId;

    public EmployeeSubResource(int departmentId, Configuration configuration) {
        this.departmentId = departmentId;
        this.config = config;
    }

    public EmployeeSubResource() {
    }
}
```

请注意，默认构造函数是将对象序列化为 JSON 对象所必需的。

接下来，让我们定义一个从一个部门检索所有雇员的端点:

```java
@GET
public DataResponse<Employee> getAll(@Context UriInfo uriInfo) {
    return LinkRest.select(Employee.class, config)
      .toManyParent(Department.class, departmentId, Department.EMPLOYEES)
      .uri(uriInfo).get();
}
```

在这个例子中，我们使用了`SelectBuilder`的`toManyParent()`方法，只查询具有给定父对象的对象。

可以用类似的方式创建 POST、PUT、DELETE 方法的端点。

要向一个部门添加雇员，我们可以用 POST 方法调用`departments/{departmentId}/employees`端点:

```java
curl -i -X POST -H "Content-Type:application/json" 
  -d "{"name":"John"}" http://localhost:8080/linkrest/department/200/employees
```

然后，让我们发送一个 GET 请求来查看该部门的雇员:

```java
curl -i -X GET "http://localhost:8080/linkrest/department/200/employees
```

这将返回一个带有数据数组的 JSON 对象:

```java
{"data":[{"id":200,"name":"John"}],"total":1}
```

## 6。使用请求参数定制响应

`LinkRest`通过向请求添加特定的参数，提供了一种定制响应的简单方法。这些可用于过滤、排序、分页或限制结果集的属性集。

### 6.1。过滤

我们可以使用`cayenneExp`参数根据属性值过滤结果。顾名思义，这遵循了 [`Cayenne`表情](https://web.archive.org/web/20220627081048/https://cayenne.apache.org/docs/4.0/cayenne-guide/expressions.html)的格式。

让我们发送一个请求，只返回名称为“IT”的部门:

```java
curl -i -X GET http://localhost:8080/linkrest/department?cayenneExp=name='IT'
```

### 6.2。分类

为对一组结果进行排序而添加的参数是`sort`和`dir`。第一个参数指定排序所依据的属性，第二个参数指定排序的方向。

让我们看看按名称排序的所有部门:

```java
curl -i -X GET "http://localhost:8080/linkrest/department?sort=name&dir;=ASC"
```

### 6.3。分页

该库通过添加`start`和`limit`参数来支持分页:

```java
curl -i -X GET "http://localhost:8080/linkrest/department?start=0&limit;=2
```

### 6.4。选择属性

使用`include`和`exclude`参数，我们可以控制在结果中返回哪些属性或关系。

例如，让我们发送一个只显示部门名称的请求:

```java
curl -i -X GET "http://localhost:8080/linkrest/department?include=name
```

为了只显示姓名和部门雇员的姓名，我们可以使用两次`include`属性:

```java
curl -i -X GET "http://localhost:8080/linkrest/department?include=name&include;=employees.name
```

## 7 .**。结论**

在本文中，我们展示了如何使用`LinkRest`框架通过 REST 端点快速公开数据模型。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627081048/https://github.com/eugenp/tutorials/tree/master/linkrest)