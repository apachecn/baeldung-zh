# Ratpack 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ratpack>

## 1。概述

[Ratpack](https://web.archive.org/web/20220628094108/https://ratpack.io/) 是一组基于`JVM`的库，为现代高性能实时应用而构建。它建立在嵌入式`Netty`事件驱动的网络引擎之上，完全符合反应式设计模式。

在本文中，我们将学习如何使用 Ratpack，并使用它构建一个小应用程序。

## 2。为什么是 Ratpack？

Ratpack 的主要优势:

*   它非常轻便、快速且可扩展
*   比 DropWizard 等其他框架占用内存少；一个有趣的基准对比结果可以在[这里](https://web.archive.org/web/20220628094108/https://phillbarber.blogspot.in/2016/01/choosing-between-ratpack-and-dropwizard.html)找到
*   因为 Ratpack 构建在`Netty`之上，所以它本质上完全是事件驱动的和非阻塞的
*   它支持`Guice`依赖管理
*   与`Spring` `Boot`非常相似，Ratpack 有自己的测试库来快速设置测试用例

## 3。创建应用程序

为了理解 Ratpack 是如何工作的，让我们从用它创建一个小应用程序开始。

### 3.1。Maven 依赖关系

首先，让我们将以下依赖项添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.4.5</version>
</dependency>
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-test</artifactId>
    <version>1.4.5</version>
</dependency>
```

你可以在 [Maven](https://web.archive.org/web/20220628094108/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-core%22) [Central](https://web.archive.org/web/20220628094108/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-core%22) 上查看最新版本。

注意，虽然我们使用 Maven 作为我们的构建系统，但是根据 [Ratpack 推荐](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/quick-start.html#using_the_gradle_plugins)，最好使用`Gradle`作为构建工具，因为 Ratpack 通过 [Ratpack 的 Gradle 插件](https://web.archive.org/web/20220628094108/https://plugins.gradle.org/search?term=ratpack)提供了一流的 Gradle 支持。

我们可以使用下面的构建 Gradle 脚本:

```java
buildscript {
    repositories {
      jcenter()
    }
    dependencies {
      classpath "io.ratpack:ratpack-gradle:1.4.5"
    }
}

apply plugin: "io.ratpack.ratpack-java"
repositories {
    jcenter()
}
dependencies {
    testCompile 'junit:junit:4.11'
    runtime "org.slf4j:slf4j-simple:1.7.21"
}
test {
    testLogging {
      events 'started', 'passed'
    }
} 
```

### 3.2。构建应用程序

配置好构建管理后，我们需要创建一个类来启动嵌入式`Netty`服务器，并构建一个简单的上下文来处理默认请求:

```java
public class Application {

    public static void main(String[] args) throws Exception {
        RatpackServer.start(server -> server.handlers(chain -> chain
          .get(ctx -> ctx.render("Welcome to Baeldung ratpack!!!"))));
    }
}
```

正如我们所看到的，通过使用`RatpackServer`我们现在可以启动服务器(默认端口 5050)。`handlers()`方法接受一个函数，该函数接收一个[链](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/api/ratpack/handling/Chain.html)对象，该对象映射所有相应的传入请求。这个“处理程序链 API”用于构建响应处理策略。

如果我们运行这个代码片段并点击浏览器 http://localhost:5050，“欢迎使用 Baeldung ratpack！！!"应该显示出来。

类似地，我们可以映射一个 HTTP POST 请求。

### 3.3。处理 URL 路径参数

在下一个例子中，我们需要在应用程序中捕获一些 URL 路径参数。在 Ratpack 中，我们使用 [PathTokens](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/api/ratpack/path/PathTokens.html) 来捕获它们:

```java
RatpackServer.start(server -> server
  .handlers(chain -> chain
  .get(":name", ctx -> ctx.render("Hello " 
  + ctx.getPathTokens().get("name") + " !!!"))));
```

这里，我们映射了`name` URL 参数。每当像`http://localhost:5050/John`这样的请求到来时，响应将是“你好，约翰！！!"。

### 3.4。带/不带过滤器的请求/响应报头修改

有时，我们需要根据需要修改内联 HTTP 响应头。Ratpack 有[个可变头](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/api/ratpack/http/MutableHeaders.html)来定制输出响应。

例如，我们需要修改响应中的以下头:`Access-Control-Allow-Origin`、`Accept-Language`和`Accept-Charset`:

```java
RatpackServer.start(server -> server.handlers(chain -> chain.all(ctx -> {
    MutableHeaders headers = ctx.getResponse().getHeaders();
    headers.set("Access-Control-Allow-Origin", "*");
    headers.set("Accept-Language", "en-us");
    headers.set("Accept-Charset", "UTF-8");
    ctx.next();
}).get(":name", ctx -> ctx
    .render("Hello " + ctx.getPathTokens().get("name") + "!!!"))));
```

通过使用`MutableHeaders` ，我们设置了三个标题，并将它们推入`Chain`。

同样，我们也可以检查传入的请求头:

```java
ctx.getRequest().getHeaders().get("//TODO")
```

同样可以通过创建过滤器来实现。Ratpack 有一个`[Handler](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/api/ratpack/handling/Handler.html)` 接口`,`，可以实现它来创建一个过滤器。它只有一个方法`handle(),`，以电流`Context`为参数:

```java
public class RequestValidatorFilter implements Handler {

    @Override
    public void handle(Context ctx) throws Exception {
        MutableHeaders headers = ctx.getResponse().getHeaders();
        headers.set("Access-Control-Allow-Origin", "*");
        ctx.next();
    }
}
```

我们可以按以下方式使用此过滤器:

```java
RatpackServer.start(
    server -> server.handlers(chain -> chain
      .all(new RequestValidatorFilter())
      .get(ctx -> ctx.render("Welcome to baeldung ratpack!!!"))));
}
```

### 3.5。JSON 解析器

Ratpack 内部使用`faster-jackson`进行 JSON 解析。我们可以使用[杰克逊](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/api/ratpack/jackson/Jackson.html)模块来解析 JSON 的任何对象。

让我们创建一个简单的 POJO 类，它将用于解析:

```java
public class Employee {

    private Long id;
    private String title;
    private String name;

    // getters and setters 

}
```

这里，我们创建了一个名为`Employee`的简单 POJO 类，它有三个参数:`id, title`和`name`。现在，我们将使用这个`Employee`对象转换成 JSON，并在点击某个 URL 时返回相同的内容:

```java
List<Employee> employees = new ArrayList<Employee>();
employees.add(new Employee(1L, "Mr", "John Doe"));
employees.add(new Employee(2L, "Mr", "White Snow"));

RatpackServer.start(
    server -> server.handlers(chain -> chain
      .get("data/employees",
      ctx -> ctx.render(Jackson.json(employees)))));
```

正如我们所见，我们手动将两个`Employee`对象添加到一个列表中，并使用`Jackson`模块将它们解析为 JSON。一旦点击了`/data/employees` URL，就会返回 JSON 对象。

这里要注意的一点是**我们根本没有使用`ObjectMapper`**，因为 Ratpack 的 Jackson 模块会在运行中完成必要的工作。

### 3.6。内存数据库

Ratpack 为内存数据库提供了一流的支持。它使用 [HikariCP](https://web.archive.org/web/20220628094108/https://github.com/brettwooldridge/HikariCP) 进行 JDBC 连接池。为了使用它，我们需要在`pom.xml`中添加 Ratpack 的 [HikariCP 模块依赖](https://web.archive.org/web/20220628094108/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22ratpack-hikari%22):

```java
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-hikari</artifactId>
    <version>1.4.5</version>
</dependency>
```

如果我们使用`Gradle`，同样需要添加到 Gradle 构建文件中:

```java
compile ratpack.dependency('hikari')
```

现在，我们需要创建一个包含表 DDL 语句的 SQL 文件，以便在服务器启动并运行后立即创建表。我们将在`src/main/resources`目录中创建`DDL.sql`文件，并向其中添加一些 DDL 语句。

因为我们使用的是 H2 数据库，所以我们也必须为其添加依赖项。

现在，通过使用 HikariModule，我们可以在运行时初始化数据库:

```java
RatpackServer.start(
    server -> server.registry(Guice.registry(bindings -> 
      bindings.module(HikariModule.class, config -> {
          config.setDataSourceClassName("org.h2.jdbcx.JdbcDataSource");
          config.addDataSourceProperty("URL",
          "jdbc:h2:mem:baeldung;INIT=RUNSCRIPT FROM 'classpath:/DDL.sql'");
      }))).handlers(...));
```

## 4。测试

如前所述，Ratpack 对 jUnit 测试用例有一流的支持。通过使用[mainclassapplicationundest](https://web.archive.org/web/20220628094108/https://ratpack.io/manual/current/api/ratpack/test/MainClassApplicationUnderTest.html)，我们可以轻松地创建测试用例并测试端点:

```java
@RunWith(JUnit4.class)
public class ApplicationTest {

    MainClassApplicationUnderTest appUnderTest
      = new MainClassApplicationUnderTest(Application.class);

    @Test
    public void givenDefaultUrl_getStaticText() {
        assertEquals("Welcome to baeldung ratpack!!!", 
          appUnderTest.getHttpClient().getText("/"));
    }

    @Test
    public void givenDynamicUrl_getDynamicText() {
        assertEquals("Hello dummybot!!!", 
          appUnderTest.getHttpClient().getText("/dummybot"));
    }

    @Test
    public void givenUrl_getListOfEmployee() 
      throws JsonProcessingException {

        List<Employee> employees = new ArrayList<Employee>();
        ObjectMapper mapper = new ObjectMapper();
        employees.add(new Employee(1L, "Mr", "John Doe"));
        employees.add(new Employee(2L, "Mr", "White Snow"));

        assertEquals(mapper.writeValueAsString(employees), 
          appUnderTest.getHttpClient().getText("/data/employees"));
    }

    @After
    public void shutdown() {
        appUnderTest.close();
    }

}
```

请注意，我们需要通过调用`close()`方法来手动终止正在运行的`MainClassApplicationUnderTest`实例，因为它可能会不必要地阻塞 JVM 资源。这就是为什么我们使用`@After`注释在测试用例执行后强制终止实例。

## 5。结论

在本文中，我们看到了使用 Ratpack 的简单性。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628094108/https://github.com/eugenp/tutorials/tree/master/ratpack)