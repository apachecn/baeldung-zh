# 用 Eclipse MicroProfile 构建微服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-microprofile>

## 1。概述

在本文中，我们将着重于构建一个基于 Eclipse MicroProfile 的微服务。

我们将看看如何使用 JAX RS、CDI 和 JSON-P API 编写一个 RESTful web 应用程序。

## 2。微服务架构

简单来说，微服务是一种软件架构风格，它作为几个独立服务的集合，形成一个完整的系统。

每一个都专注于一个功能边界，并通过一个与语言无关的协议(比如 REST)与其他功能边界进行通信。

## 3。Eclipse 微文件

Eclipse MicroProfile 是一项旨在为微服务架构优化企业 Java 的计划。它基于 Jakarta EE WebProfile APIs 的一个子集，所以我们可以像构建 Jakarta EE 应用程序一样构建 MicroProfile 应用程序。

MicroProfile 的目标是定义构建微服务的标准 API，并在多个 MicroProfile 运行时交付可移植的应用程序。

## 4。Maven 依赖关系

构建 Eclipse MicroProfile 应用程序所需的所有依赖项都由这个 BOM(物料清单)依赖项提供:

```java
<dependency>
    <groupId>org.eclipse.microprofile</groupId>
    <artifactId>microprofile</artifactId>
    <version>1.2</version>
    <type>pom</type>
    <scope>provided</scope>
</dependency> 
```

范围被设置为`provided`，因为微文件运行时已经包括了 API 和实现。

## 5。表示模型

让我们从创建一个快速资源类开始:

```java
public class Book {
    private String id;
    private String name;
    private String author;
    private Integer pages;
    // ...
}
```

正如我们所见，这个`Book`类没有注释。

## 6。使用 CDI

简单来说，CDI 是一个提供依赖注入和生命周期管理的 API。它简化了企业 beans 在 Web 应用程序中的使用。

现在让我们创建一个 CDI 托管 bean 作为图书表示的存储:

```java
@ApplicationScoped
public class BookManager {

    private ConcurrentMap<String, Book> inMemoryStore
      = new ConcurrentHashMap<>();

    public String add(Book book) {
        // ...
    }

    public Book get(String id) {
        // ...
    }

    public List getAll() {
        // ...
    }
} 
```

我们用`@ApplicationScoped`注释这个类，因为我们只需要一个实例，它的状态由所有客户机共享。为此，我们使用了一个`ConcurrentMap`作为类型安全的内存数据存储。然后我们添加了`CRUD`操作的方法。

现在我们的 bean 已经准备好了 CDI，可以注入到 bean `BookEndpoint using`的`@Inject`注释中。

## 7 .**。JAX-遥感 API**

为了用 JAX-RS 创建一个 REST 应用程序，我们需要创建一个用`@ApplicationPath`注释的`Application`类和一个用`@Path.`注释的资源

### 7.1。JAX 的申请

JAX-RS 应用程序标识了我们在 Web 应用程序中公开资源的基础 URI。

让我们创建以下 JAX 遥感应用程序:

```java
@ApplicationPath("/library")
public class LibraryApplication extends Application {
}
```

在这个例子中，Web 应用程序中的所有 JAX 遥感资源类都与`LibraryApplication`相关联，使它们位于同一个`library`路径下，这就是`ApplicationPath annotation.`的值

这个带注释的类告诉 JAX RS 运行时它应该自动找到资源并公开它们。

### 7.2。JAX 终点

一个`Endpoint`类，也称为`Resource`类，应该定义一个资源，尽管许多相同的类型在技术上是可能的。

每个用`@Path`注释的 Java 类，或者至少有一个用 `@Path or @HttpMethod`注释的方法，都是一个端点。

现在，我们将创建一个公开该表示的 JAX-RS 端点:

```java
@Path("books")
@RequestScoped
public class BookEndpoint {

    @Inject
    private BookManager bookManager;

    @GET
    @Path("{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getBook(@PathParam("id") String id) {
        return Response.ok(bookManager.get(id)).build();
    }

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response getAllBooks() {
        return Response.ok(bookManager.getAll()).build();
    }

    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    public Response add(Book book) {
        String bookId = bookManager.add(book);
        return Response.created(
          UriBuilder.fromResource(this.getClass())
            .path(bookId).build())
            .build();
    }
} 
```

此时，我们可以在 web 应用程序中访问`/library/books`路径下的`BookEndpoint`资源。

### 7.3。JAX 的 JSON 媒体类型

JAX RS 支持许多媒体类型来与 REST 客户端通信，但是 **Eclipse MicroProfile 限制了 JSON** 的使用，因为它指定了 JSOP-P API 的使用。因此，我们需要用`@Consumes(MediaType.APPLICATION_JSON)`和@ `Produces(MediaType.APPLICATION_JSON).`来注释我们的方法

`@Consumes`注释限制了可接受的格式——在本例中，只接受 JSON 数据格式。HTTP 请求头`Content-Type`应该是`application/json`。

同样的想法隐藏在`@Produces`注释的背后。JAX RS 运行时应该将响应整理成 JSON 格式。请求 HTTP 头`Accept`应该是`application/json.`

## 8 个。JSON-p的缩写形式

JAX RS 运行时支持 JSON-P 开箱即用，这样我们就可以使用`JsonObject`作为方法输入参数或返回类型。

但是在现实世界中，我们经常使用 POJO 类。所以我们需要一种方法来做`JsonObject`和 POJO 之间的映射。这里是 JAX 的实体提供者开始行动的地方。

为了将 JSON 输入流封送到`Book` POJO，这就调用了一个带有类型`Book,`参数的资源方法，我们需要创建一个类`BookMessageBodyReader:`

```java
@Provider
@Consumes(MediaType.APPLICATION_JSON)
public class BookMessageBodyReader implements MessageBodyReader<Book> {

    @Override
    public boolean isReadable(
      Class<?> type, Type genericType, 
      Annotation[] annotations, 
      MediaType mediaType) {

        return type.equals(Book.class);
    }

    @Override
    public Book readFrom(
      Class type, Type genericType, 
      Annotation[] annotations,
      MediaType mediaType, 
      MultivaluedMap<String, String> httpHeaders, 
      InputStream entityStream) throws IOException, WebApplicationException {

        return BookMapper.map(entityStream);
    }
} 
```

我们用同样的过程将一个`Book`解组到 JSON 输出流，这是通过创建一个`BookMessageBodyWriter:`来调用一个返回类型为`Book,`的资源方法

```java
@Provider
@Produces(MediaType.APPLICATION_JSON)
public class BookMessageBodyWriter 
  implements MessageBodyWriter<Book> {

    @Override
    public boolean isWriteable(
      Class<?> type, Type genericType, 
      Annotation[] annotations, 
      MediaType mediaType) {

        return type.equals(Book.class);
    }

    // ...

    @Override
    public void writeTo(
      Book book, Class<?> type, 
      Type genericType, 
      Annotation[] annotations, 
      MediaType mediaType, 
      MultivaluedMap<String, Object> httpHeaders, 
      OutputStream entityStream) throws IOException, WebApplicationException {

        JsonWriter jsonWriter = Json.createWriter(entityStream);
        JsonObject jsonObject = BookMapper.map(book);
        jsonWriter.writeObject(jsonObject);
        jsonWriter.close();
    }
} 
```

由于`BookMessageBodyReader`和`BookMessageBodyWriter`用`@Provider`标注，它们被 JAX RS 运行时自动注册。

## 9。构建和运行应用程序

微文件应用程序是可移植的，并且应该在任何兼容的微文件运行时中运行。我们将解释如何在 [Open Liberty](https://web.archive.org/web/20221129021345/https://openliberty.io/) 中构建和运行我们的应用程序，但是我们可以使用任何兼容的 Eclipse MicroProfile。

我们通过配置文件`server.xml`配置 Open Liberty 运行时:

```java
<server description="OpenLiberty MicroProfile server">
    <featureManager>
        <feature>jaxrs-2.0</feature>
        <feature>cdi-1.2</feature>
        <feature>jsonp-1.0</feature>
    </featureManager>
    <httpEndpoint httpPort="${default.http.port}" httpsPort="${default.https.port}"
      id="defaultHttpEndpoint" host="*"/>
    <applicationManager autoExpand="true"/>
    <webApplication context-root="${app.context.root}" location="${app.location}"/>
</server>
```

让我们将插件`liberty-maven-plugin`添加到 pom.xml 中:

```java
<?xml version="1.0" encoding="UTF-8"?>
<plugin>
    <groupId>net.wasdev.wlp.maven.plugins</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>2.1.2</version>
    <configuration>
        <assemblyArtifact>
            <groupId>io.openliberty</groupId>
            <artifactId>openliberty-runtime</artifactId>
            <version>17.0.0.4</version>
            <type>zip</type>
        </assemblyArtifact>
        <configFile>${basedir}/src/main/liberty/config/server.xml</configFile>
        <packageFile>${package.file}</packageFile>
        <include>${packaging.type}</include>
        <looseApplication>false</looseApplication>
        <installAppPackages>project</installAppPackages>
        <bootstrapProperties>
            <app.context.root>/</app.context.root>
            <app.location>${project.artifactId}-${project.version}.war</app.location>
            <default.http.port>9080</default.http.port>
            <default.https.port>9443</default.https.port>
        </bootstrapProperties>
    </configuration>
    <executions>
        <execution>
            <id>install-server</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>install-server</goal>
                <goal>create-server</goal>
                <goal>install-feature</goal>
            </goals>
        </execution>
        <execution>
            <id>package-server-with-apps</id>
            <phase>package</phase>
            <goals>
                <goal>install-apps</goal>
                <goal>package-server</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

这个插件可以通过一组属性进行配置:

```java
<properties>
    <!--...-->
    <app.name>library</app.name>
    <package.file>${project.build.directory}/${app.name}-service.jar</package.file>
    <packaging.type>runnable</packaging.type>
</properties>
```

上面的 exec 目标生成一个可执行的 jar 文件，这样我们的应用程序将成为一个独立的微服务，可以独立部署和运行。我们还可以将其部署为 Docker 映像。

要创建可执行 jar，请运行以下命令:

```java
mvn package 
```

为了运行我们的微服务，我们使用以下命令:

```java
java -jar target/library-service.jar
```

这将启动 Open Liberty 运行时并部署我们的服务。我们可以访问我们的端点，并通过以下 URL 获取所有书籍:

```java
curl http://localhost:9080/library/books
```

结果是一个 JSON:

```java
[
  {
    "id": "0001-201802",
    "isbn": "1",
    "name": "Building Microservice With Eclipse MicroProfile",
    "author": "baeldung",
    "pages": 420
  }
] 
```

要获得一本书，我们需要以下 URL:

```java
curl http://localhost:9080/library/books/0001-201802
```

结果是 JSON:

```java
{
    "id": "0001-201802",
    "isbn": "1",
    "name": "Building Microservice With Eclipse MicroProfile",
    "author": "baeldung",
    "pages": 420
}
```

现在，我们将通过与 API 交互来添加一本新书:

```java
curl 
  -H "Content-Type: application/json" 
  -X POST 
  -d '{"isbn": "22", "name": "Gradle in Action","author": "baeldung","pages": 420}' 
  http://localhost:9080/library/books 
```

正如我们所看到的，响应的状态是 201，表明图书已经成功创建，而`Location`是我们可以访问它的 URI:

```java
< HTTP/1.1 201 Created
< Location: http://localhost:9080/library/books/0009-201802
```

## 10。结论

本文展示了如何基于 Eclipse MicroProfile 构建一个简单的微服务，讨论了 JAX RS、JSON-P 和 CDI。

代码可以在 Github 上的[处获得；这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221129021345/https://github.com/eugenp/tutorials/tree/master/microservices-modules/microprofile)