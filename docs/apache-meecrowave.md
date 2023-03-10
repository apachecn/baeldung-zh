# 使用 Apache Meecrowave 构建微服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-meecrowave>

## 1。概述

在本教程中，我们将探索 Apache [Meecrowave](https://web.archive.org/web/20220626211508/https://openwebbeans.apache.org/meecrowave/) 框架的基本功能。

Meecrowave 是来自 Apache 的轻量级微服务框架，它与 CDI、JAX-RS 和 JSON API 配合得非常好。设置和部署非常简单。它还消除了部署 Tomcat、Glassfish、Wildfly 等大型应用服务器的麻烦。

## 2。Maven 依赖关系

要使用 Meecrowave，让我们在`pom.xml:`中定义依赖关系

```java
<dependency>
    <groupId>org.apache.meecrowave</groupId>
    <artifactId>meecrowave-core</artifactId>
    <version>1.2.1</version>
</dependency>
```

在 [Maven Central](https://web.archive.org/web/20220626211508/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.meecrowave%22%20AND%20a%3A%22meecrowave-core%22) 上查看最新版本。

## 3。启动一个简单的服务器

为了启动一个 Meecrowave 服务器，我们需要做的就是编写`main `方法，**创建一个`Meecrowave `实例并调用主`bake() `方法**:

```java
public static void main(String[] args) {
    try (Meecrowave meecrowave = new Meecrowave()) {
        meecrowave.bake().await();
    }
}
```

如果我们将应用程序打包成一个分发包，就不需要这个 main 方法；我们将在后面的部分中研究这个问题。从 IDE 测试应用程序时，main 类非常有用。

作为一个优势，在 IDE 中开发时，一旦我们使用 main 类运行应用程序，它会随着代码的更改自动重新加载，从而省去一次又一次重新启动服务器进行测试的麻烦。

注意，如果我们使用 Java 9，不要忘记给 VM 添加 javax `.xml.bind`模块:

```java
--add-module javax.xml.bind
```

以这种方式创建服务器将以默认配置启动它。我们可以使用`Meecrowave.Builder` 类`:`以编程方式更新默认配置

```java
Meecrowave.Builder builder = new Meecrowave.Builder();
builder.setHttpPort(8080);
builder.setScanningPackageIncludes("com.baeldung.meecrowave");
builder.setJaxrsMapping("/api/*");
builder.setJsonpPrettify(true);
```

并在烘烤服务器时使用这个`builder `实例:

```java
try (Meecrowave meecrowave = new Meecrowave(builder)) { 
    meecrowave.bake().await();
}
```

这里有更多可配置的属性。

## 4。休息终点

现在，一旦服务器准备就绪，让我们创建一些 REST 端点:

```java
@RequestScoped
@Path("article")
public class ArticleEndpoints {

    @GET
    public Response getArticle() {
        return Response.ok().entity(new Article("name", "author")).build();      
    }

    @POST 
    public Response createArticle(Article article) { 
        return Response.status(Status.CREATED).entity(article).build(); 
    }
}
```

注意，我们主要使用 **JAX-RS 注释来创建 REST 端点**。在这里阅读更多关于 JAX-RS [的信息](/web/20220626211508/https://www.baeldung.com/jax-rs-spec-and-implementations)。

在下一节中，我们将看到如何测试这些端点。

## 5.测试

用 Meecrowave 编写的 for REST API 编写单元测试用例就像编写带注释的 JUnit 测试用例一样简单。

让我们首先将测试依赖项添加到我们的`pom.xml `中:

```java
<dependency>
    <groupId>org.apache.meecrowave</groupId>
    <artifactId>meecrowave-junit</artifactId>
    <version>1.2.1</version>
    <scope>test</scope>
</dependency> 
```

要查看最新版本，请查看 Maven Central 。

同样，让我们添加 [OkHttp](/web/20220626211508/https://www.baeldung.com/guide-to-okhttp) 作为我们测试的 Http 客户端:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.10.0</version>
</dependency>
```

点击查看最新版本[。](https://web.archive.org/web/20220626211508/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22okhttp%22)

现在有了依赖关系，让我们继续编写测试:

```java
@RunWith(MonoMeecrowave.Runner.class)
public class ArticleEndpointsIntegrationTest {

    @ConfigurationInject
    private Meecrowave.Builder config;
    private static OkHttpClient client;

    @BeforeClass
    public static void setup() {
        client = new OkHttpClient();
    }

    @Test
    public void whenRetunedArticle_thenCorrect() {
        String base = "http://localhost:" + config.getHttpPort();

        Request request = new Request.Builder()
          .url(base + "/article")
          .build();
        Response response = client.newCall(request).execute();
        assertEquals(200, response.code());
    }
}
```

在编写测试用例时，用`MonoMeecrowave.Runner`类注释测试类，同时注入配置，以访问 Meecrowave 为测试服务器使用的随机端口

## 6.依赖注入

为了**将依赖注入到类**中，我们需要在特定的范围内注释那些类。

让我们以一个`ArticleService `类为例:

```java
@ApplicationScoped
public class ArticleService {
    public Article createArticle(Article article) {
        return article;
    }
}
```

现在让我们使用`javax.inject.Inject` 注释`:`将它注入到我们的`ArticleEndpoints` 实例中

```java
@Inject
ArticleService articleService;
```

## 7.打包应用程序

使用 Meecrowave Maven 插件，创建发行包变得非常简单:

```java
<build>
    ...
    <plugins>
        <plugin>
            <groupId>org.apache.meecrowave</groupId>
            <artifactId>meecrowave-maven-plugin</artifactId>
            <version>1.2.1</version>
        </plugin>
    </plugins>
</build>
```

一旦我们有了插件，让我们**使用 Maven 目标`meecrowave:bundle `来打包应用程序**。

打包后，它将在目标目录中创建一个 zip 文件:

```java
meecrowave-meecrowave-distribution.zip
```

这个 zip 文件包含部署应用程序所需的构件:

```java
|____meecrowave-distribution
| |____bin
| | |____meecrowave.sh
| |____logs
| | |____you_can_safely_delete.txt
| |____lib
| |____conf
| | |____log4j2.xml
| | |____meecrowave.properties
```

让我们导航到 bin 目录并启动应用程序:

```java
./meecrowave.sh start
```

要停止应用程序:

```java
./meecrowave.sh stop
```

## 8.结论

在本文中，我们学习了如何使用 Apache Meecrowave 来创建微服务。此外，我们还研究了应用程序的一些基本配置，并准备了一个分发包。

和往常一样，代码片段可以在 Github 项目中找到。