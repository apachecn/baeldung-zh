# Jooby 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jooby>

## 1。概述

`[Jooby](https://web.archive.org/web/20220524024843/http://jooby.org/)`是建立在最常用的`NIO web servers`之上的一个可扩展的快速微型网络框架。它非常简单和模块化，显然是为现代 web 架构设计的。它也支持`[Javascript](https://web.archive.org/web/20220524024843/https://www.jooby.org/docs/)`和`Kotlin`。

默认情况下，`Jooby`自带对`Netty, Jetty, and Undertow`的强大支持。

在本文中，我们将了解总体的`Jooby`项目结构，以及如何使用`Jooby`构建一个简单的 web 应用程序。

## 2。应用架构

一个简单的`Jooby`应用程序结构如下所示:

```java
├── public
|   └── welcome.html
├── conf
|   ├── application.conf
|   └── logback.xml
└── src
|   ├── main
|   |   └── java
|   |       └── com
|   |           └── baeldung
|   |               └── jooby
|   |                   └── App.java
|   └── test
|       └── java
|           └── com
|               └── baeldung
|                   └── jooby
|                       └── AppTest.java
├── pom.xml
```

这里要注意的一点是，在`public`目录中，我们可以放入 css/js/html 等静态文件。在`conf`目录中，我们可以放置应用程序需要的任何配置文件，如`logback.xml`或`application.conf`等。

## 3。Maven 依赖关系

我们可以通过在我们的`pom.xml:`中添加以下依赖项来创建一个简单的`Jooby`应用程序

```java
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-netty</artifactId>
    <version>1.1.3</version>
</dependency>
```

如果我们想选择`Jetty`或`Undertow`，我们可以使用以下依赖关系:

```java
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-jetty</artifactId>
    <version>1.1.3</version>
</dependency>
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-undertow</artifactId>
    <version>1.1.3</version>
</dependency>
```

你可以在[中央 Maven 资源库](https://web.archive.org/web/20220524024843/https://search.maven.org/classic/#search%7Cga%7C1%7Cjooby)中查看最新版本的`Jooby`项目。

**`Jooby`也有专门的 Maven 原型。**我们可以用它来创建一个带有所有必要的预建依赖项的样本项目。

我们可以使用以下脚本来生成示例项目:

```java
mvn archetype:generate -B -DgroupId=com.baeldung.jooby -DartifactId=jooby 
-Dversion=1.0 -DarchetypeArtifactId=jooby-archetype 
-DarchetypeGroupId=org.jooby -DarchetypeVersion=1.1.3
```

## 4。构建应用程序

### 4.1。启动服务器

要启动嵌入式服务器，我们需要使用以下代码片段:

```java
public class App extends Jooby {
    public static void main(String[] args) {
        run(App::new, args);
    }
}
```

一旦启动，服务器将在`default port` `8080`上运行。

我们还可以用自定义端口和自定义`HTTPS`端口配置后端服务器:

```java
{
    port( 8081 );
    securePort( 8443 );
}
```

### 4.2。实施路由器

在`Jooby`中创建基于路径的路由器非常容易。例如，我们可以通过以下方式为路径'【T1]'创建一个路由器:

```java
{
    get( "/login", () -> "Hello from Baeldung");
}
```

同样，如果我们想处理其他`HTTP`方法，比如 POST、PUT 等，我们可以使用下面的代码片段:

```java
{
    post( "/save", req -> {
        Mutant token = req.param( "token" );
        return token.intValue();
    });
}
```

这里，我们从请求中获取请求参数名称标记。默认情况下，所有请求参数都被类型转换为`Jooby`的 **`[Mutant](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/Mutant.html)`** 数据类型。根据期望，我们可以将其转换成任何支持的原始数据类型。

我们可以通过以下方式检查任何 url 参数:

```java
{
    get( "/user/{id}", req -> "Hello user : " + req.param("id").value() );
    get( "/user/:id", req -> "Hello user: " + req.param("id").value() );
}
```

我们可以使用以上任何一种。也可以找到以固定内容开始的参数。例如，我们可以通过以下方式找到一个以'`uid:'` 开头的 URL 参数:

```java
{
    get( "/uid:{id}", req -> "Hello User with id : uid" + 
        req.param("id").value());
}
```

### 4.3。实现 MVC 模式控制器

对于一个企业应用程序来说，`Jooby`附带了一个 MVC API，就像任何其他 MVC 框架一样，比如 Spring MVC。

例如，我们可以处理一个名为'`/hello`'的路径:

```java
@Path("/hello")
public class GetController {
    @GET
    public String hello() {
        return "Hello Baeldung";
    }
}
```

同样，我们可以创建一个处理程序来处理其他带有`[@POST](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/mvc/POST.html), [@PUT,](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/mvc/PUT.html) [@DELETE](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/mvc/DELETE.html)`的 HTTP 方法，等等。注释。

### 4.4。处理静态内容

提供任何静态内容，如 HTML、Javascript、CSS、图像等。，我们需要将这些文件放在`public`目录中。

放置后，我们可以从路由器将任何 url 映射到这些资源:

```java
{
    assets( "/employee" , "form.html" );
}
```

### 4.5。处理表单

`Jooby's` [`Request`](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/Request.html) 界面默认处理任何表单对象而不使用任何手动类型转换。

假设我们需要通过表单提交员工的详细信息。第一步，我们需要创建一个用于保存数据的`Employee` bean 对象:

```java
public class Employee {
    String id;
    String name;
    String email;

    // standard constructors, getters and setters
}
```

现在，我们需要创建一个页面来创建表单:

```java
<form enctype="application/x-www-form-urlencoded" action="/submitForm" 
    method="post">
    <input name="id" />
    <input name="name" />
    <input name="email" />
    <input type="submit" value="Submit"/>
</form>
```

接下来，我们将创建一个 post 处理程序来处理这个表单并获取提交的数据:

```java
post( "/submitForm", req -> {
    Employee employee = req.params(Employee.class);
    // ...
    return "empoyee data saved successfullly";
});
```

这里要注意的一点是，我们必须需要将表单`enctype`声明为`application/x-www-form-urlencoded`来支持动态表单绑定。

通过`[Request.file(String filename)](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/Request.html#file-java.lang.String-)`,我们可以检索上传的文件:

```java
post( "/upload", req -> {
    Upload upload = req.file("file");
    // ...
    upload.close();
});
```

### 4.6。实施过滤器

现成的`, Jooby`提供了定义全局过滤器和基于路径的过滤器的灵活性。

在`Jooby`中实现过滤器有点棘手，因为**我们需要配置 URL 路径两次，一次用于过滤器，一次用于处理程序。**

例如，如果我们必须为名为'【T0]的 URL 路径实现一个过滤器，我们需要在这个路径中显式地实现过滤器:

```java
get( "/filter", ( req, resp, chain ) -> {
    // ...
    chain.next( req, resp );
});
```

语法非常类似于`Servlet`过滤器。通过调用 `[Response.send(Result result)](https://web.archive.org/web/20220524024843/https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/Response.html#send-org.jooby.Result-)`方法，可以在过滤器中限制请求并发回响应。

一旦实现了过滤器，我们需要实现请求处理程序:

```java
get("/filter", (req, resp) -> {
    resp.send("filter response");
});
```

### 4.7。会话

`Jooby`提供了两种类型的会话实现；内存中和基于 cookie。

实现内存中的会话管理非常简单。我们可以选择任何带有`Jooby` 的高吞吐量会话存储，比如`EhCache, Guava, HazleCast, Cassandra, Couchbase, Redis, MongoDB,` 和 `Memcached.`

例如，要实现基于 Redis 的会话存储，我们需要添加以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.jooby</groupId>
    <artifactId>jooby-jedis</artifactId>
    <version>1.1.3</version>
</dependency>
```

现在，我们可以使用下面的代码片段来启用会话管理:

```java
{
    use(new Redis());
    session(RedisSessionStore.class);

    get( "/session" , req -> {
        Session session = req.session();
        session.set("token", "value");
        return session.get("token").value();
    });
}
```

这里要注意的一点是，我们可以将`Redis` url 配置为`application.conf`中的`‘db'`属性。

为了启用基于 cookie 的会话管理，我们需要声明`cookieSession()`。**如果选择基于 cookie 的方法，我们必须需要在`application.conf`文件中声明`application.secret`属性。**由于每个 cookie 都将使用这个密钥进行签名，所以使用长的随机字符串片段作为密钥总是明智的。

在内存和基于 cookie 的方法中，我们必须在`application.conf`文件中声明必要的配置参数，否则应用程序将在启动时抛出一个`[IllegalStateException](https://web.archive.org/web/20220524024843/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IllegalStateException.html)`。

## 5。测试

测试 MVC 路由确实很容易，因为路由被绑定到某个类的策略上。这使得针对所有路由运行单元测试变得容易。

例如，我们可以快速为默认 URL 创建一个测试用例:

```java
public class AppTest {

    @ClassRule
    public static JoobyRule app = new JoobyRule(new App());

    @Test
    public void given_defaultUrl_expect_fixedString() {

        get("/").then().assertThat().body(equalTo("Hello World!"))
          .statusCode(200).contentType("text/html;charset=UTF-8");
    }
}
```

这里要注意的一点是，使用`@ClassRule`注释将为所有测试用例只创建一个服务器实例。如果我们需要为每个测试用例构建单独的服务器实例，我们必须使用不带 static 修饰符的`@Rule`注释。

我们也可以使用`Jooby's [MockRouter](https://web.archive.org/web/20220524024843/https://github.com/jooby-project/jooby/blob/2.x/modules/jooby-test/src/main/java/io/jooby/MockRouter.java)`以同样的方式测试路径:

```java
@Test
public void given_defaultUrl_with_mockrouter_expect_fixedString() 
  throws Throwable {

    String result = new MockRouter(new App()).get("/");

    assertEquals("Hello World!", result);
}
```

## 6。结论

在本教程中，我们探索了`Jooby`项目及其基本功能。

像往常一样，完整的源代码可以在 GitHub 上获得[。](https://web.archive.org/web/20220524024843/https://github.com/eugenp/tutorials/tree/master/jooby)