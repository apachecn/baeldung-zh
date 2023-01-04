# 阿帕奇骆驼和 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-camel-spring-boot>

## 1。概述

从本质上来说，Apache Camel 是一个集成引擎，简单地说，它可以用来促进各种技术之间的交互。

这些服务和技术之间的桥梁被称为`routes.`路由，在一个引擎上实现`CamelContext`，它们通过所谓的“交换消息”进行通信。

## 2。Maven 依赖关系

首先，我们需要包含 Spring Boot、Camel、Rest API 与 Swagger 和 JSON 的依赖关系:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-servlet-starter</artifactId>
        <version>3.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-jackson-starter</artifactId>
        <version>3.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-swagger-java-starter</artifactId>
        <version>3.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>3.15.0</version>
    </dependency>    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

Apache Camel 依赖项的最新版本可以在这里找到。

## 3。主类

让我们先创建一个 Spring Boot `Application`:

```java
@SpringBootApplication
@ComponentScan(basePackages="com.baeldung.camel")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 4。Spring Boot 骆驼配置

现在让我们用 Spring 配置我们的应用程序，从配置文件(属性)开始。

例如，让我们在`src/main/resources`中的`application.properties`文件上为我们的应用程序配置一个日志:

```java
logging.config=classpath:logback.xml
camel.springboot.name=MyCamel
server.address=0.0.0.0
management.address=0.0.0.0
management.port=8081
endpoints.enabled = true
endpoints.health.enabled = true
```

这个例子显示了一个`application.properties`文件，它也设置了一个回退配置的路径。通过将 IP 设置为“0.0.0.0”，我们完全限制了`admin`和`management`对 Spring Boot 提供的网络服务器的访问。此外，我们支持对我们的应用程序端点以及健康检查端点进行所需的网络访问。

另一个配置文件是`application.yml`。在其中，我们将添加一些属性来帮助我们将值注入我们的应用程序路线:

```java
server:
  port: 8080
camel:
  springboot:
    name: ServicesRest
management:
  port: 8081
endpoints:
  enabled: false
  health:
    enabled: true
quickstart:
  generateOrderPeriod: 10s
  processOrderPeriod: 30s
```

## 5 **。设置 Camel Servlet**

开始使用 Camel 的一种方法是将其注册为 servlet，这样它就可以拦截 HTTP 请求并将其重定向到我们的应用程序。

如前所述，从 Camel 的 2.18 及以下版本开始，我们可以利用我们的`application.yml`，为我们的最终 URL 创建一个参数。稍后它将被注入到我们的 Java 代码中:

```java
baeldung:
  api:
    path: '/camel'
```

回到我们的`Application`类，我们需要在上下文路径的根位置注册 Camel servlet，当应用程序启动时，它将从`application.yml`中的引用`baeldung.api.path` 注入:

```java
@Value("${baeldung.api.path}")
String contextPath;

@Bean
ServletRegistrationBean servletRegistrationBean() {
    ServletRegistrationBean servlet = new ServletRegistrationBean
      (new CamelHttpTransportServlet(), contextPath+"/*");
    servlet.setName("CamelServlet");
    return servlet;
}
```

从 Camel 的 2.19 版本开始，这种配置已经被删除，因为默认情况下`CamelServlet`被设置为`“/camel”`。

## 6。建设路线

让我们从扩展 Camel 的`RouteBuilder`类开始创建一个路由，并将其设置为`@Component`，这样组件扫描例程就可以在 web 服务器初始化期间找到它:

```java
@Component
class RestApi extends RouteBuilder {
    @Override
    public void configure() {
        CamelContext context = new DefaultCamelContext();

        restConfiguration()...
        rest("/api/")... 
        from("direct:remoteService")...
    }
}
```

在这个类中，我们覆盖了 Camel 的`RouteBuilder`类中的`configure()`方法。

**Camel 总是需要一个`CamelContext`实例**——保存传入和传出消息的核心组件。

在这个简单的例子中，`DefaultCamelContext`就足够了，因为它只是将消息和路由绑定到其中，就像我们将要创建的 REST 服务一样。

### 6.1。`restConfiguration()`路线

接下来，我们为我们计划在`restConfiguration()`方法中创建的端点创建一个 REST 声明:

```java
restConfiguration()
  .contextPath(contextPath) 
  .port(serverPort)
  .enableCORS(true)
  .apiContextPath("/api-doc")
  .apiProperty("api.title", "Test REST API")
  .apiProperty("api.version", "v1")
  .apiContextRouteId("doc-api")
  .component("servlet")
  .bindingMode(RestBindingMode.json)
```

这里，我们用来自 YAML 文件的注入属性注册上下文路径。同样的逻辑也适用于我们的应用程序的端口。CORS 已启用，允许跨站点使用此 web 服务。绑定模式允许并将参数转换到我们的 API。

接下来，我们将 Swagger 文档添加到我们之前设置的 URI、标题和版本中。当我们为 REST web 服务创建方法/端点时，Swagger 文档将自动更新。

这个 Swagger 上下文本身就是一条骆驼路线，在启动过程中我们可以在服务器日志中看到一些关于它的技术信息。我们的示例文档默认在`http://localhost:8080/camel/api-doc.`提供

### 6.2。`rest()`路线

现在，让我们从上面列出的`configure()`方法中实现`rest()`方法调用:

```java
rest("/api/")
  .id("api-route")
  .consumes("application/json")
  .post("/bean")
  .bindingMode(RestBindingMode.json_xml)
  .type(MyBean.class)
  .to("direct:remoteService");
```

对于熟悉 API 的人来说，这种方法非常简单。`id`是`CamelContext`内的路线标识。下一行定义了 MIME 类型。这里定义绑定模式是为了说明我们可以在`restConfiguration()`上设置一个模式。

`post()`方法向 API 添加一个操作，生成一个“`POST /bean`”端点，而`MyBean`(一个带有`Integer id`和`String name`的普通 Java bean)定义预期的参数。

类似地，HTTP 操作如 GET、PUT 和 DELETE 也可以以`get()`、`put()`、`delete()`的形式使用。

最后，`to()`方法创建了一个通向另一条路由的桥。在这里，它告诉 Camel 在它的上下文/引擎中搜索我们将要创建的另一条路线——这条路线由值/id“`direct: …`”命名和检测，匹配在`from()`方法中定义的路线。

### 6.3。`from()`路线与`transform()`

使用 Camel 时，路线接收参数，然后转换、变换和处理这些参数。之后，它将这些参数发送到另一个路由，该路由将结果转发到所需的输出(文件、数据库、SMTP 服务器或 REST API 响应)。

在本文中，我们只在我们正在覆盖的`configure()`方法中创建另一个路由。这将是我们最后一条`to()`路线的目的地路线:

```java
from("direct:remoteService")
  .routeId("direct-route")
  .tracing()
  .log(">>> ${body.id}")
  .log(">>> ${body.name}")
  .transform().simple("Hello ${in.body.name}")
  .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(200));
```

`from()`方法遵循相同的原则，并且有许多与`rest()`方法相同的方法，除了它使用 Camel 上下文消息。这就是参数“`direct-route`”的原因，它创建了一个到前述方法`rest().to()`的链接。

**许多其他的转换也是可用的**，包括提取为 Java 原语(或对象)并将其发送到持久层。请注意，路由总是从传入消息中读取，因此链式路由将忽略传出消息。

我们的例子已经准备好了，我们可以试一试:

*   运行提示命令:`mvn spring-boot:run`
*   用头参数`Content-Type: application/json`和有效载荷`{“id”: 1,”name”: “World”}`向`http://localhost:8080/camel/api/bean`发出 POST 请求
*   我们应该会收到返回代码 201 和响应:`Hello, World`

### 6.4。简单的脚本语言

该示例使用`tracing()`方法输出日志记录。注意，我们使用了`${}` 占位符；这些是属于 Camel 的脚本语言 SIMPLE 的一部分。它适用于通过路由交换的消息，如入站消息的正文。

在我们的示例中，我们使用 SIMPLE 将 Camel 消息体中的 bean 属性输出到日志中。

我们也可以用它来做简单的转换，就像用`transform()`方法展示的那样。

### 6.5。`from()`路线与`process()`

让我们做一些更有意义的事情，比如调用服务层返回处理过的数据。简单并不意味着繁重的数据处理，所以让我们用一个`process()`方法代替`transform()`:

```java
from("direct:remoteService")
  .routeId("direct-route")
  .tracing()
  .log(">>> ${body.id}")
  .log(">>> ${body.name}")
  .process(new Processor() {
      @Override
      public void process(Exchange exchange) throws Exception {
          MyBean bodyIn = (MyBean) exchange.getIn().getBody();
          ExampleServices.example(bodyIn);
          exchange.getIn().setBody(bodyIn);
      }
  })
  .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(200));
```

这允许我们将数据提取到一个 bean 中，与之前在`type()`方法中定义的 bean 相同，并在我们的`ExampleServices`层中处理它。

因为我们之前将`bindingMode()`设置为 JSON，所以响应已经是基于我们的 POJO 生成的正确的 JSON 格式。这意味着对于一个`ExampleServices`类:

```java
public class ExampleServices {
    public static void example(MyBean bodyIn) {
        bodyIn.setName( "Hello, " + bodyIn.getName() );
        bodyIn.setId(bodyIn.getId() * 10);
    }
}
```

相同的 HTTP 请求现在返回响应代码 201 和主体: `{“id”: 10,”name”: “Hello, World”}`。

## 7。结论

通过几行代码，我们创建了一个相对完整的应用程序。所有依赖项都是通过一个命令自动构建、管理和运行的。此外，我们可以创建将各种技术结合在一起的 API。

这种方法也是非常容器友好的，产生了一个非常精简的服务器环境，可以很容易地按需复制。额外的配置可能性可以很容易地合并到容器模板配置文件中。

这个 REST 例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220926191131/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-camel)

最后，除了`filter()`、 `process()`、 `transform()`和 `marshall()`API，Camel 中还提供了许多其他的集成模式和数据操作:

*   [骆驼整合模式](https://web.archive.org/web/20220926191131/https://camel.apache.org/enterprise-integration-patterns.html)
*   [Camel 用户指南](https://web.archive.org/web/20220926191131/https://camel.apache.org/user-guide.html)
*   [骆驼简单语言](https://web.archive.org/web/20220926191131/https://camel.apache.org/simple.html)