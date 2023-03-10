# Oracle Helidon 的微服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/microservices-oracle-helidon>

## 1。概述

[Helidon](https://web.archive.org/web/20221206211307/https://helidon.io/) 是 Oracle 最近开源的新 Java 微服务框架。它在 Oracle projects 内部以 J4C (Java for Cloud)的名称使用。

在本教程中，我们将涵盖框架的主要概念，然后我们将构建和运行基于 Helidon 的微服务。

## 2。编程模式

目前，**框架支持两种编写微服务的编程模型:Helidon SE 和 Helidon MP。**

虽然 Helidon SE 被设计为支持反应式编程模型的微框架，但另一方面，Helidon MP 是一个 Eclipse 微概要运行时，它允许 Jakarta EE 社区以可移植的方式运行微服务。

在这两种情况下，Helidon 微服务都是一个 Java SE 应用程序，它从 main 方法启动一个 tinny HTTP 服务器。

## 3 .helidon se〔t1〕

在本节中，我们将更详细地了解 Helidon SE 的主要组件:web 服务器、配置和安全性。

### 3.1。设置网络服务器

为了开始使用`WebServer API` `,`，我们需要将所需的 [Maven 依赖项](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-webserver)添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>io.helidon.webserver</groupId>
    <artifactId>helidon-webserver</artifactId>
    <version>0.10.4</version>
</dependency>
```

为了有一个简单的 web 应用程序，**我们可以使用下面的构建器方法之一:`WebServer.create(serverConfig, routing)`或者仅仅是`WebServer.create(routing)`** 。最后一种采用默认的服务器配置，允许服务器在随机端口上运行。

这是一个运行在预定义端口上的简单 Web 应用程序。我们还注册了一个简单的处理程序，它将对任何带有“/ `greet'`路径和`GET`方法的 HTTP 请求响应问候消息:

```java
public static void main(String... args) throws Exception {
    ServerConfiguration serverConfig = ServerConfiguration.builder()
      .port(9001).build();
    Routing routing = Routing.builder()
      .get("/greet", (request, response) -> response.send("Hello World !")).build();
    WebServer.create(serverConfig, routing)
      .start()
      .thenAccept(ws ->
          System.out.println("Server started at: http://localhost:" + ws.port())
      );
}
```

最后一行是启动服务器并等待服务 HTTP 请求。但是如果我们在 main 方法中运行这个示例代码，我们会得到错误:

```java
Exception in thread "main" java.lang.IllegalStateException: 
  No implementation found for SPI: io.helidon.webserver.spi.WebServerFactory
```

`WebServer`实际上是一个 SPI，我们需要提供一个运行时实现。目前， **Helidon 提供基于 [Netty](https://web.archive.org/web/20221206211307/https://netty.io/index.html) 内核的`NettyWebServer`实现**。

这里是这个实现的 [Maven 依赖关系](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-webserver-netty):

```java
<dependency>
    <groupId>io.helidon.webserver</groupId>
    <artifactId>helidon-webserver-netty</artifactId>
    <version>0.10.4</version>
    <scope>runtime</scope>
</dependency>
```

现在，我们可以运行主应用程序，并通过调用已配置的端点来检查它是否工作:

```java
http://localhost:9001/greet
```

在本例中，我们使用 builder 模式配置了端口和路径。

Helidon SE 还允许使用由`Config` API 提供配置数据的配置模式。这是下一节的主题。

### 3.2。`Config` API

**`Config`API 提供了从配置源**读取配置数据的工具。

Helidon SE 为许多配置源提供实现。默认实现由 [`helidon-config`](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-config) 提供，其中配置源是位于类路径下的`application.properties`文件:

```java
<dependency>
    <groupId>io.helidon.config</groupId>
    <artifactId>helidon-config</artifactId>
    <version>0.10.4</version>
</dependency>
```

要读取配置数据，我们只需使用默认的构建器，它默认从`application.properties:`获取配置数据

```java
Config config = Config.builder().build();
```

让我们在`src/main/resource`目录下创建一个`application.properties`文件，内容如下:

```java
server.port=9080
web.debug=true
web.page-size=15
user.home=C:/Users/app
```

**要读取这些值，我们可以使用`Config.get()`方法**，然后方便地转换成相应的 Java 类型:

```java
int port = config.get("server.port").asInt();
int pageSize = config.get("web.page-size").asInt();
boolean debug = config.get("web.debug").asBoolean();
String userHome = config.get("user.home").asString();
```

事实上，默认的构建器按照这个优先级顺序加载第一个找到的文件:`application.yaml, application.conf, application.json, and application.properties.`后三种格式需要一个额外的相关配置依赖项。例如，要使用 YAML 格式，我们需要添加相关的 YAML [配置](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-config-yaml)依赖项:

```java
<dependency>
    <groupId>io.helidon.config</groupId>
    <artifactId>helidon-config-yaml</artifactId>
    <version>0.10.4</version>
</dependency>
```

然后，我们添加一个`application.yml`:

```java
server:
  port: 9080  
web:
  debug: true
  page-size: 15
user:
  home: C:/Users/app
```

类似地，要使用 CONF，这是一种 JSON 简化格式，或 JSON 格式，我们需要添加 [helidon-config-hocon](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-config-hocon) 依赖项。

请注意，这些文件中的配置数据可以被环境变量和 Java 系统属性覆盖。

**我们还可以通过禁用环境变量和系统属性或者通过明确指定配置源来控制默认的构建器行为**:

```java
ConfigSource configSource = ConfigSources.classpath("application.yaml").build();
Config config = Config.builder()
  .disableSystemPropertiesSource()
  .disableEnvironmentVariablesSource()
  .sources(configSource)
  .build();
```

除了从类路径中读取配置数据，我们还可以使用两个外部源配置，即 git 和 etcd 配置。为此，我们需要 [helidon-config-git](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-config-git) 和 [helidon-git-etcd](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-config-etcd) 依赖项。

最后，如果所有这些配置源都不能满足我们的需求，Helidon 允许我们为我们的配置源提供一个实现。例如，我们可以提供一个可以从数据库中读取配置数据的实现。

### 3.3。`Routing` API

**`Routing`API 提供了将 HTTP 请求绑定到 Java 方法的机制。**我们可以通过使用请求方法和路径作为匹配标准或使用更多标准的`RequestPredicate`对象来实现这一点。

因此，要配置路由，我们可以只使用 HTTP 方法作为标准:

```java
Routing routing = Routing.builder()
  .get((request, response) -> {} );
```

或者我们可以将 HTTP 方法与请求路径结合起来:

```java
Routing routing = Routing.builder()
  .get("/path", (request, response) -> {} );
```

我们也可以使用`RequestPredicate`进行更多的控制。例如，我们可以检查现有的标题或内容类型:

```java
Routing routing = Routing.builder()
  .post("/save",
    RequestPredicate.whenRequest()
      .containsHeader("header1")
      .containsCookie("cookie1")
      .accepts(MediaType.APPLICATION_JSON)
      .containsQueryParameter("param1")
      .hasContentType("application/json")
      .thenApply((request, response) -> { })
      .otherwise((request, response) -> { }))
      .build();
```

到目前为止，我们已经提供了函数式的处理程序。我们也可以使用`Service`类，它允许以更复杂的方式编写处理程序。

所以，让我们首先为我们正在处理的对象创建一个模型，即`Book`类:

```java
public class Book {
    private String id;
    private String name;
    private String author;
    private Integer pages;
    // ...
}
```

**我们可以通过实现`Service.update()`方法为`Book`类创建 REST 服务。这允许配置相同资源的子路径:**

```java
public class BookResource implements Service {

    private BookManager bookManager = new BookManager();

    @Override
    public void update(Routing.Rules rules) {
        rules
          .get("/", this::books)
          .get("/{id}", this::bookById);
    }

    private void bookById(ServerRequest serverRequest, ServerResponse serverResponse) {
        String id = serverRequest.path().param("id");
        Book book = bookManager.get(id);
        JsonObject jsonObject = from(book);
        serverResponse.send(jsonObject);
    }

    private void books(ServerRequest serverRequest, ServerResponse serverResponse) {
        List<Book> books = bookManager.getAll();
        JsonArray jsonArray = from(books);
        serverResponse.send(jsonArray);
    }
    //...
}
```

我们还将媒体类型配置为 JSON，因此为此我们需要 [helidon-webserver-json](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-webserver-json) 依赖关系:

```java
<dependency>
    <groupId>io.helidon.webserver</groupId>
    <artifactId>helidon-webserver-json</artifactId>
    <version>0.10.4</version>
</dependency>
```

最后，**我们使用`Routing`构建器的`register()`方法将根路径绑定到资源。**在本例中，`Paths`所配置的服务都以根路径为前缀:

```java
Routing routing = Routing.builder()
  .register(JsonSupport.get())
  .register("/books", new BookResource())
  .build();
```

我们现在可以启动服务器并检查端点:

```java
http://localhost:9080/books
http://localhost:9080/books/0001-201810
```

### 3.4。安全性

在本节中，**我们将使用安全模块**来保护我们的资源。

让我们从声明所有必要的依赖项开始:

```java
<dependency>
    <groupId>io.helidon.security</groupId>
    <artifactId>helidon-security</artifactId>
    <version>0.10.4</version>
</dependency>
<dependency>
    <groupId>io.helidon.security</groupId>
    <artifactId>helidon-security-provider-http-auth</artifactId>
    <version>0.10.4</version>
</dependency>
<dependency>
    <groupId>io.helidon.security</groupId>
    <artifactId>helidon-security-integration-webserver</artifactId>
    <version>0.10.4</version>
</dependency>
```

[helidon-security](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-security) 、[heli don-security-provider-http-auth](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-security-provider-http-auth)和[heli don-security-integration-web server](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-security-integration-webserver)依赖项可从 Maven Central 获得。

安全模块为身份验证和授权提供了许多提供者。**对于这个例子，我们将使用 HTTP 基本认证提供者**,因为它相当简单，但是其他提供者的过程几乎是相同的。

**首先要做的是创建一个`Security`实例。为了简单起见，我们可以通过编程来实现**:

```java
Map<String, MyUser> users = //...
UserStore store = user -> Optional.ofNullable(users.get(user));

HttpBasicAuthProvider httpBasicAuthProvider = HttpBasicAuthProvider.builder()
  .realm("myRealm")
  .subjectType(SubjectType.USER)
  .userStore(store)
  .build();

Security security = Security.builder()
  .addAuthenticationProvider(httpBasicAuthProvider)
  .build();
```

或者我们可以使用配置方法。

在这种情况下，我们将在通过`Config` API 加载的`application.yml`文件中声明所有安全配置:

```java
#Config 4 Security ==> Mapped to Security Object
security:
  providers:
  - http-basic-auth:
      realm: "helidon"
      principal-type: USER # Can be USER or SERVICE, default is USER
      users:
      - login: "user"
        password: "user"
        roles: ["ROLE_USER"]
      - login: "admin"
        password: "admin"
        roles: ["ROLE_USER", "ROLE_ADMIN"]

  #Config 4 Security Web Server Integration ==> Mapped to WebSecurity Object
  web-server:
    securityDefaults:
      authenticate: true
    paths:
    - path: "/user"
      methods: ["get"]
      roles-allowed: ["ROLE_USER", "ROLE_ADMIN"]
    - path: "/admin"
      methods: ["get"]
      roles-allowed: ["ROLE_ADMIN"]
```

为了加载它，我们只需要创建一个`Config`对象，然后调用`Security.fromConfig()`方法:

```java
Config config = Config.create();
Security security = Security.fromConfig(config);
```

**一旦我们有了`Security`实例，我们首先需要使用`WebSecurity.from()`方法:**向`WebServer`注册它

```java
Routing routing = Routing.builder()
  .register(WebSecurity.from(security).securityDefaults(WebSecurity.authenticate()))
  .build();
```

我们还可以使用 config 方法直接创建一个`WebSecurity`实例，通过它我们可以加载安全性和 web 服务器配置:

```java
Routing routing = Routing.builder()        
  .register(WebSecurity.from(config))
  .build();
```

我们现在可以为`/user`和`/admin`路径添加一些处理程序，启动服务器并尝试访问它们:

```java
Routing routing = Routing.builder()
  .register(WebSecurity.from(config))
  .get("/user", (request, response) -> response.send("Hello, I'm Helidon SE"))
  .get("/admin", (request, response) -> response.send("Hello, I'm Helidon SE"))
  .build();
```

## 4。直升机 MP

Helidon MP 是 Eclipse MicroProfile 的一个实现，也为运行基于 MicroProfile 的微服务提供了一个运行时。

因为我们已经有了[一篇关于 Eclipse MicroProfile](/web/20221206211307/https://www.baeldung.com/eclipse-microprofile) 的文章，我们将检查源代码并修改它以在 Helidon MP 上运行。

检查完代码后，我们将删除所有依赖项和插件，并将 Helidon MP 依赖项添加到 POM 文件中:

```java
<dependency>
    <groupId>io.helidon.microprofile.bundles</groupId>
    <artifactId>helidon-microprofile-1.2</artifactId>
    <version>0.10.4</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-binding</artifactId>
    <version>2.26</version>
</dependency>
```

从 Maven Central 可以获得 [helidon-microprofile-1.2](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:helidon-microprofile-1.2) 和[jersey-media-JSON-binding](https://web.archive.org/web/20221206211307/https://search.maven.org/search?q=a:jersey-media-json-binding)依赖项。

接下来，**我们将在`src/main/resource/META-INF`目录下**添加`beans.xml`文件，内容如下:

```java
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
  http://xmlns.jcp.org/xml/ns/javaee/beans_2_0.xsd"
  version="2.0" bean-discovery-mode="annotated">
</beans>
```

在`LibraryApplication`类中，覆盖`getClasses()`方法，这样服务器就不会扫描资源:

```java
@Override
public Set<Class<?>> getClasses() {
    return CollectionsHelper.setOf(BookEndpoint.class);
}
```

最后，创建一个 main 方法并添加以下代码片段:

```java
public static void main(String... args) {
    Server server = Server.builder()
      .addApplication(LibraryApplication.class)
      .port(9080)
      .build();
    server.start();
}
```

仅此而已。我们现在可以调用所有的图书资源了。

## 5。结论

在本文中，我们探讨了 Helidon 的主要组件，还展示了如何设置 Helidon SE 和 MP。由于 Helidon MP 只是一个 Eclipse 微文件运行时，我们可以使用它运行任何现有的基于微文件的微服务。

和往常一样，上面所有例子的代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221206211307/https://github.com/eugenp/tutorials/tree/master/microservices-modules/helidon)