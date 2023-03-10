# Micronaut 框架简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/micronaut>

## 1。什么是 Micronaut

Micronaut 是一个基于 JVM 的框架，用于构建轻量级、模块化的应用程序。由 OCI 开发的，也是 Grails 的开发者， **Micronaut 是最新的框架，旨在使创建微服务变得简单快捷**。

虽然 Micronaut 包含一些与 Spring 等现有框架相似的特性，但它也有一些与众不同的新特性。由于支持 Java、Groovy 和 Kotlin，它提供了多种创建应用程序的方法。

## 2。主要特点

Micronaut 最令人兴奋的特性之一是它的编译时依赖注入机制。大多数框架使用反射和代理在运行时执行依赖注入。然而，Micronaut 在编译时构建其依赖注入数据。结果是更快的应用启动和更小的内存占用。

另一个特性是它对客户端和服务器的反应式编程的一流支持。由于 RxJava 和 Project Reactor 都受支持，开发人员可以选择特定的反应式实现。

Micronaut 还有几个特性使其成为开发云原生应用程序的优秀框架。**它支持多种服务发现工具，如 Eureka 和 Consul，还可以与不同的分布式跟踪系统配合使用，如 Zipkin 和 Jaeger。**

它还提供了对创建 AWS lambda 函数的支持，使创建无服务器应用程序变得容易。

## 3。入门

最简单的入门方法是使用 [SDKMAN](https://web.archive.org/web/20220625080557/https://sdkman.io/install) :

```java
> sdk install micronaut 1.0.0.RC2
```

这将安装我们构建、测试和部署 Micronaut 应用程序所需的所有二进制文件。它还提供了 Micronaut CLI 工具，让我们可以轻松地开始新项目。

二进制工件也可以在 [Sonatype](https://web.archive.org/web/20220625080557/https://oss.sonatype.org/content/groups/public/io/micronaut/) 和 [GitHub](https://web.archive.org/web/20220625080557/https://github.com/micronaut-projects/micronaut-core/releases) 上获得。

在接下来的几节中，我们将看看这个框架的一些特性。

## 4。依赖注入

如前所述，Micronaut 在编译时处理依赖注入，这与大多数 IoC 容器不同。

然而，它仍然**完全支持 JSR-330 注释**,所以使用 beans 类似于其他 IoC 框架。

为了将 bean 自动连接到我们的代码中，我们使用了`@Inject:`

```java
@Inject
private EmployeeService service;
```

`@Inject`注释就像`@Autowired`一样工作，可以用于字段、方法、构造函数和参数。

默认情况下，所有 beans 的作用范围都是原型。我们可以使用`@Singleton. `快速创建单例 bean。如果多个类实现了同一个 bean 接口，可以使用`@Primary`来解除它们之间的冲突:

```java
@Primary
@Singleton
public class BlueCar implements Car {}
```

当 beans 可选时，可以使用`@Requires`注释，或者只在满足某些条件时执行自动连接。

在这方面，它的表现很像 Spring Boot `@Conditional`的注解:

```java
@Singleton
@Requires(beans = DataSource.class)
@Requires(property = "enabled")
@Requires(missingBeans = EmployeeService)
@Requires(sdk = Sdk.JAVA, value = "1.8")
public class JdbcEmployeeService implements EmployeeService {}
```

## 5.构建 HTTP 服务器

现在让我们来看看如何创建一个简单的 HTTP 服务器应用程序。首先，我们将使用 SDKMAN 创建一个项目:

```java
> mn create-app hello-world-server -build maven
```

这将在名为`hello-world-server.`的目录中使用 Maven 创建一个新的 Java 项目。在这个目录中，我们将找到我们的主应用程序源代码、Maven POM 文件和项目的其他支持文件。

非常简单的默认应用程序:

```java
public class ServerApplication {
    public static void main(String[] args) {
        Micronaut.run(ServerApplication.class);
    }
}
```

### 5.1。阻止 HTTP

这个应用程序本身不会做太多事情。让我们添加一个有两个端点的控制器。两者都将返回问候，但是一个将使用`GET` HTTP 动词，另一个将使用`POST:`

```java
@Controller("/greet")
public class GreetController {

    @Inject
    private GreetingService greetingService;

    @Get("/{name}")
    public String greet(String name) {
        return greetingService.getGreeting() + name;
    }

    @Post(value = "/{name}", consumes = MediaType.TEXT_PLAIN)
    public String setGreeting(@Body String name) {
        return greetingService.getGreeting() + name;
    }
}
```

### 5.2。无功 IO

默认情况下，Micronaut 将使用传统的阻塞 I/O 来实现这些端点。然而，**我们可以通过将返回类型更改为任何反应式非阻塞类型**来快速实现非阻塞端点。

例如，对于 RxJava，我们可以使用`Observable`。同样，当使用 Reactor 时，我们可以返回`Mono`或`Flux`数据类型:

```java
@Get("/{name}")
public Mono<String> greet(String name) {
    return Mono.just(greetingService.getGreeting() + name);
}
```

对于阻塞和非阻塞端点，Netty 是用于处理 HTTP 请求的底层服务器。

通常，请求是在启动时创建的主 I/O 线程池中处理的，这使它们成为阻塞。

然而，当从控制器端点返回非阻塞数据类型时，Micronaut 使用 Netty 事件循环线程，使整个请求成为非阻塞的。

## 6.构建 HTTP 客户端

现在让我们构建一个客户端来使用我们刚刚创建的端点。Micronaut 提供了两种创建 HTTP 客户端的方法:

*   声明式 HTTP 客户端
*   编程式 HTTP 客户端

### 6.1。声明式 HTTP 客户端

创建的第一个也是最快的方法是使用声明性方法:

```java
@Client("/greet")
public interface GreetingClient {
    @Get("/{name}")
    String greet(String name);
}
```

注意**我们没有实现任何代码来调用我们的服务**。相反，Micronaut 知道如何从我们提供的方法签名和注释中调用服务。

为了测试这个客户机，我们可以创建一个 JUnit 测试，它使用嵌入式服务器 API 来运行我们服务器的嵌入式实例:

```java
public class GreetingClientTest {
    private EmbeddedServer server;
    private GreetingClient client;

    @Before
    public void setup() {
        server = ApplicationContext.run(EmbeddedServer.class);
        client = server.getApplicationContext().getBean(GreetingClient.class);
    }

    @After
    public void cleanup() {
        server.stop();
    }

    @Test
    public void testGreeting() {
        assertEquals(client.greet("Mike"), "Hello Mike");
    }
}
```

### 6.2。编程式 HTTP 客户端

如果我们需要对其行为和实现进行更多的控制，我们也可以选择编写一个更传统的客户端:

```java
@Singleton
public class ConcreteGreetingClient {
   private RxHttpClient httpClient;

   public ConcreteGreetingClient(@Client("/") RxHttpClient httpClient) {
      this.httpClient = httpClient;
   }

   public String greet(String name) {
      HttpRequest<String> req = HttpRequest.GET("/greet/" + name);
      return httpClient.retrieve(req).blockingFirst();
   }

   public Single<String> greetAsync(String name) {
      HttpRequest<String> req = HttpRequest.GET("/async/greet/" + name);
      return httpClient.retrieve(req).first("An error as occurred");
   }
}
```

默认的 HTTP 客户端使用 RxJava，因此可以轻松地处理阻塞或非阻塞调用。

## 7.Micronaut CLI

当我们使用 Micronaut CLI 工具来创建我们的示例项目时，我们已经看到了它的作用。

在我们的例子中，我们创建了一个独立的应用程序，但是它还有其他一些功能。

### 7.1。联盟项目

在 Micronaut 中，联合只是一组位于同一目录下的独立应用程序。通过使用联盟，我们可以轻松地一起管理它们，并确保它们获得相同的默认值和设置。

当我们使用 CLI 工具生成一个联盟时，它使用与`create-app`命令相同的参数。它将创建一个顶级的项目结构，每个独立的应用程序都将在它的子目录中创建。

### 7.2。功能

**创建独立应用或联盟时，我们可以决定我们的应用需要哪些功能**。这有助于确保最小的依赖集包含在项目中。

我们使用`-features argument`指定特性，并提供一个以逗号分隔的特性名称列表。

通过运行以下命令，我们可以找到可用功能的列表:

```java
> mn profile-info service

Provided Features:
--------------------
* annotation-api - Adds Java annotation API
* config-consul - Adds support for Distributed Configuration with Consul
* discovery-consul - Adds support for Service Discovery with Consul
* discovery-eureka - Adds support for Service Discovery with Eureka
* groovy - Creates a Groovy application
[...] More features available
```

### 7.3。现有项目

**我们还可以使用 CLI 工具来修改现有项目。**使我们能够创建 beans、客户机、控制器等等。当我们从现有项目内部运行`mn`命令时，我们将有一组新的命令可用:

```java
> mn help
| Command Name         Command Description
-----------------------------------------------
create-bean            Creates a singleton bean
create-client          Creates a client interface
create-controller      Creates a controller and associated test
create-job             Creates a job with scheduled method
```

## 8.结论

在对 Micronaut 的简要介绍中，我们看到了构建阻塞和非阻塞 HTTP 服务器和客户机是多么容易。此外，我们还探讨了它的 CLI 的一些特性。

但这只是它提供的功能的一小部分。还完全支持无服务器功能、服务发现、分布式跟踪、监控和度量、分布式配置等等。

虽然它的许多特性都是从 Grails 和 Spring 等现有框架中派生出来的，但它也有许多独特的特性，有助于它脱颖而出。

和往常一样，我们可以在我们的 [GitHub repo](https://web.archive.org/web/20220625080557/https://github.com/eugenp/tutorials/tree/master/micronaut) 中找到上面的示例代码。