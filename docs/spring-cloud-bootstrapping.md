# spring Cloud–引导

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-bootstrapping>

## 1。概述

Spring Cloud 是一个用于构建健壮的云应用程序的框架。该框架为迁移到分布式环境时面临的许多常见问题提供了解决方案，从而简化了应用程序的开发。

运行微服务架构的应用旨在简化开发、部署和维护。应用程序的分解性质允许开发人员一次专注于一个问题。可以在不影响系统其他部分的情况下引入改进。

另一方面，当我们采用微服务方法时，会出现不同的挑战:

*   将配置外部化，这样就很灵活，不需要在更改时重新构建服务
*   服务发现
*   隐藏部署在不同主机上的服务的复杂性

在本文中，我们将构建五个微服务:配置服务器、发现服务器、网关服务器、图书服务，最后是评级服务。这五项微服务构成了一个坚实的基础应用，可以开始云开发并解决上述挑战。

## 2。配置服务器

在开发云应用程序时，一个问题是维护配置并将其分发到我们的服务。我们真的不想在横向扩展我们的服务之前花时间配置每个环境，或者冒着安全违规的风险将我们的配置嵌入到我们的应用程序中。

为了解决这个问题，我们将把所有的配置整合到一个 Git 存储库中，并把它连接到一个管理所有应用程序配置的应用程序。我们将建立一个非常简单的实现。

要了解更多细节并查看更复杂的示例，请查看我们的 [Spring Cloud 配置](/web/20220529012116/https://www.baeldung.com/spring-cloud-configuration)文章。

### 2.1。设置

导航到 `[https://start.spring.io](https://web.archive.org/web/20220529012116/https://start.spring.io/)`并选择 Maven 和 Spring Boot 2.2.x

将工件设置为“配置`“`”。在 dependencies 部分，搜索“config server”并添加该模块。然后按下`generate`按钮，我们就可以下载一个 zip 文件，里面有一个预先配置好的项目，并可以开始运行了。

或者，我们可以生成一个`Spring Boot`项目，并手动向 POM 文件添加一些依赖项。

这些依赖关系将在所有项目之间共享:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies> 

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

让我们为配置服务器添加一个依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

作为参考，我们可以在 Maven Central 上找到最新版本(`[spring-cloud-dependencies](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-dependencies%22), [test](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22), [config-server](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-config-server%22)`)。

### 2.2。弹簧配置

要启用配置服务器，我们必须向主应用程序类添加一些注释:

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigApplication {...}
```

将把我们的应用程序变成一个配置服务器。

### 2.3。属性

让我们在`src/main/resources`中添加`application.properties`:

```java
server.port=8081
spring.application.name=config

spring.cloud.config.server.git.uri=file://${user.home}/application-config
```

配置服务器最重要的设置是`git.uri`参数。这目前被设置为一个相对文件路径，通常解析为 Windows 上的`c:\Users\{username}\`或*nix 上的`/Users/{username}/`。这个属性指向一个 Git 存储库，所有其他应用程序的属性文件都存储在这个存储库中。如有必要，可以将其设置为绝对文件路径。

**提示**:在 windows 机器上以“file:///”作为值的前缀，在*nix 上则使用“file://”。

### 2.4。Git 存储库

导航到由`spring.cloud.config.server.git.uri` 定义的文件夹，并添加文件夹`application-config`。CD 放进那个文件夹，然后输入`git init`。这将初始化一个 Git 存储库，我们可以在其中存储文件并跟踪它们的更改。

### 2.5。运行

让我们运行配置服务器，并确保它正在工作。从命令行键入`mvn spring-boot:run`。这将启动服务器。

我们应该看到以下输出，表明服务器正在运行:

```java
Tomcat started on port(s): 8081 (http)
```

### 2.6。引导配置

在我们后续的服务器中，我们希望它们的应用程序属性由这个配置服务器管理。要做到这一点，我们实际上需要做一些先有鸡还是先有蛋的事情:**在每个应用程序中配置知道如何与服务器对话的属性。**

这是一个引导过程，**这些应用程序中的每一个都有一个名为`bootstrap.properties`的文件。**它将包含与`application.properties`类似的属性，但有所不同:

一个母弹簧`ApplicationContext` **首先加载`bootstrap.properties` 。**这很重要，这样配置服务器可以开始管理`application.properties`中的属性。正是这个特殊的`ApplicationContext `也将解密任何加密的应用程序属性。

**保持这些属性文件的不同是明智的。** `bootstrap.properties` 用于准备好配置服务器，`application.properties`用于特定于我们的应用程序的属性。不过，从技术上讲，将应用程序属性放在`bootstrap.properties`中是可能的。

最后，由于配置服务器正在管理我们的应用程序属性，有人可能想知道为什么要有一个`application.properties` ？答案是这些作为缺省值仍然有用，也许配置服务器没有这些缺省值。

## 3。发现

现在我们已经完成了配置，我们需要一种方法让我们所有的服务器能够找到彼此。我们将通过设置`Eureka`发现服务器来解决这个问题。因为我们的应用程序可以在任何 IP/端口组合上运行，所以我们需要一个中央地址注册中心来作为应用程序地址查找。

当提供新服务器时，它将与发现服务器通信并注册其地址，以便其他服务器可以与之通信。这样，其他应用程序可以在发出请求时使用这些信息。

要了解更多细节并了解更复杂的发现实现，请查看 [Spring Cloud Eureka 文章](/web/20220529012116/https://www.baeldung.com/spring-cloud-netflix-eureka)。

### 3.1。设置

我们将再次导航到 [start.spring.io](https://web.archive.org/web/20220529012116/https://start.spring.io/) 。将神器设置为“发现”。搜索“eureka server”并添加依赖项。搜索“配置客户端”并添加依赖项。最后，生成项目。

或者，我们可以创建一个`Spring Boot`项目，从配置服务器复制`POM`的内容，并交换这些依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

作为参考，我们会在 Maven Central ( `[config-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-server](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka-server%22)`)上找到这些包。

### 3.2。弹簧配置

让我们将 Java config 添加到主类中:

```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryApplication {...}
```

`@EnableEurekaServer`将使用`Netflix Eureka`将该服务器配置为发现服务器。`Spring Boot`将自动检测类路径上的配置依赖，并从配置服务器中查找配置。

### 3.3。属性

现在我们将添加两个属性文件:

首先，我们将`bootstrap.properties`添加到`src/main/resources`中:

```java
spring.cloud.config.name=discovery
spring.cloud.config.uri=http://localhost:8081
```

这些属性将允许发现服务器在启动时查询配置服务器。

其次，我们将`discovery.properties`添加到我们的 Git 存储库中

```java
spring.application.name=discovery
server.port=8082

eureka.instance.hostname=localhost

eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

文件名必须与`spring.application.name`属性匹配。

此外，我们告诉该服务器它正在默认区域中运行，这与配置客户端的区域设置相匹配。我们还告诉服务器不要向另一个发现实例注册。

在生产中，我们会有不止一个这样的设备，以便在出现故障时提供冗余，这种设置是正确的。

让我们将文件提交给 Git 存储库。否则，将无法检测到该文件。

### 3.4。向配置服务器添加依赖关系

将此依赖项添加到配置服务器 POM 文件中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

作为参考，我们可以在 Maven Central 上找到这个 bundle([`eureka-client`](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22))。

将这些属性添加到配置服务器的`src/main/resources`中的`application.properties`文件中:

```java
eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

### 3.5。运行

使用相同的命令`mvn spring-boot:run`启动发现服务器。命令行的输出应该包括:

```java
Fetching config from server at: http://localhost:8081
...
Tomcat started on port(s): 8082 (http)
```

停止并重新运行配置服务。如果一切正常，输出应该如下所示:

```java
DiscoveryClient_CONFIG/10.1.10.235:config:8081: registering service...
Tomcat started on port(s): 8081 (http)
DiscoveryClient_CONFIG/10.1.10.235:config:8081 - registration status: 204
```

## 4。网关

现在，我们已经解决了配置和发现问题，但客户端访问我们的所有应用程序仍有问题。

如果我们把一切都留在分布式系统中，那么我们将不得不管理复杂的 CORS 报头，以允许客户端上的跨源请求。我们可以通过创建网关服务器来解决这个问题。这将作为一个反向代理，将请求从客户端传送到我们的后端服务器。

网关服务器是微服务架构中的一个优秀应用，因为它允许所有响应来自单个主机。这将消除对 CORS 的需要，并给我们一个方便的地方来处理常见的问题，如认证。

### 4.1。设置

现在我们知道该怎么做了。导航到 [`https://start.spring.io`](https://web.archive.org/web/20220529012116/https://start.spring.io/) 。将工件设置为“网关”。搜索“zuul”并添加依赖项。搜索“配置客户端”并添加依赖项。搜索“尤里卡发现”并添加依赖项。最后，生成项目。

或者，我们可以用这些依赖项创建一个`Spring Boot`应用程序:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

作为参考，我们可以在 Maven Central ( `[config-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22), [zuul](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-zuul%22)`)上找到捆绑包。

### 4.2。弹簧配置

让我们将配置添加到主类中:

```java
@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
public class GatewayApplication {...}
```

### 4.3。属性

现在我们将添加两个属性文件:

`src/main/resources`中的`bootstrap.properties`:

```java
spring.cloud.config.name=gateway
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true

eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

在我们的 Git 存储库中

```java
spring.application.name=gateway
server.port=8080

eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5

zuul.routes.book-service.path=/book-service/**
zuul.routes.book-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.book-service.execution.isolation.thread.timeoutInMilliseconds=600000

zuul.routes.rating-service.path=/rating-service/**
zuul.routes.rating-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.rating-service.execution.isolation.thread.timeoutInMilliseconds=600000

zuul.routes.discovery.path=/discovery/**
zuul.routes.discovery.sensitive-headers=Set-Cookie,Authorization
zuul.routes.discovery.url=http://localhost:8082
hystrix.command.discovery.execution.isolation.thread.timeoutInMilliseconds=600000
```

`zuul.routes` 属性允许我们定义一个应用程序，根据 ant URL 匹配器路由某些请求。我们的属性告诉 Zuul 将来自`/book-service/**`的任何请求路由到具有`book-service`的`spring.application.name` 的应用程序。然后，Zuul 将使用应用程序名称从发现服务器中查找主机，并将请求转发到该服务器。

记住提交存储库中的更改！

### 4.4。运行

运行配置和发现应用程序，并等待配置应用程序向发现服务器注册。如果它们已经在运行，我们不必重新启动它们。完成后，运行网关服务器。网关服务器应该在端口 8080 上启动，并向发现服务器注册自己。控制台的输出应该包含:

```java
Fetching config from server at: http://10.1.10.235:8081/
...
DiscoveryClient_GATEWAY/10.1.10.235:gateway:8080: registering service...
DiscoveryClient_GATEWAY/10.1.10.235:gateway:8080 - registration status: 204
Tomcat started on port(s): 8080 (http)
```

一个容易犯的错误是在配置服务器向 Eureka 注册之前启动服务器。在这种情况下，我们将看到一个输出如下的日志:

```java
Fetching config from server at: http://localhost:8888
```

这是配置服务器的默认 URL 和端口，表明我们的发现服务在发出配置请求时没有地址。只需等待几秒钟，然后再试一次，一旦配置服务器注册到 Eureka，问题就会解决。

## 5。预订服务

在微服务架构中，我们可以自由开发尽可能多的应用来满足业务目标。工程师通常会根据领域来划分他们的服务。我们将遵循这种模式，创建一个图书服务来处理应用程序中图书的所有操作。

### 5.1。设置

再来一次。导航至`[https://start.spring.io](https://web.archive.org/web/20220529012116/https://start.spring.io/)`。将工件设置为“预订服务”。搜索“web”并添加依赖项。搜索“配置客户端”并添加依赖项。搜索“尤里卡发现”并添加依赖项。生成项目。

或者，将这些依赖项添加到项目中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

作为参考，我们可以在 Maven Central ( `[config-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22), [web](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)`)上找到捆绑包。

### 5.2。弹簧配置

让我们修改我们的主类:

```java
@SpringBootApplication
@EnableEurekaClient
@RestController
@RequestMapping("/books")
public class BookServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookServiceApplication.class, args);
    }

    private List<Book> bookList = Arrays.asList(
        new Book(1L, "Baeldung goes to the market", "Tim Schimandle"),
        new Book(2L, "Baeldung goes to the park", "Slavisa")
    );

    @GetMapping("")
    public List<Book> findAllBooks() {
        return bookList;
    }

    @GetMapping("/{bookId}")
    public Book findBook(@PathVariable Long bookId) {
        return bookList.stream().filter(b -> b.getId().equals(bookId)).findFirst().orElse(null);
    }
}
```

我们还添加了一个 REST 控制器和一个由属性文件设置的字段，以返回我们将在配置期间设置的值。

现在让我们添加 POJO 这本书:

```java
public class Book {
    private Long id;
    private String author;
    private String title;

    // standard getters and setters
}
```

### 5.3。属性

现在我们只需要添加两个属性文件:

`src/main/resources`中的`bootstrap.properties`:

```java
spring.cloud.config.name=book-service
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true

eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

在我们的 Git 存储库中:

```java
spring.application.name=book-service
server.port=8083

eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

让我们将变更提交给存储库。

### 5.4。运行

一旦所有其他应用程序启动，我们就可以启动图书服务。控制台输出应该如下所示:

```java
DiscoveryClient_BOOK-SERVICE/10.1.10.235:book-service:8083: registering service...
DiscoveryClient_BOOK-SERVICE/10.1.10.235:book-service:8083 - registration status: 204
Tomcat started on port(s): 8083 (http)
```

启动后，我们可以使用浏览器访问刚刚创建的端点。导航到 http://localhost:8080/book-service/books，我们得到一个 JSON 对象，其中包含我们在 out 控制器中添加的两本书。请注意，我们不是在端口 8083 上直接访问图书服务，而是通过网关服务器。

## 6。评级服务

像我们的图书服务一样，我们的评级服务将是一个域驱动的服务，它将处理与评级相关的操作。

### 6.1。设置

再来一次。导航到 [`https://start.spring.io`](https://web.archive.org/web/20220529012116/https://start.spring.io/) 。将神器设置为“评级-服务”。搜索“web”并添加依赖项。搜索“配置客户端”并添加依赖项。搜索`“`尤里卡发现`“`并添加依赖关系。然后，生成该项目。

或者，将这些依赖项添加到项目中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

作为参考，我们可以在 Maven Central ( `[config-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22), [eureka-client](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22), [web](https://web.archive.org/web/20220529012116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)`)上找到捆绑包。

### 6.2。弹簧配置

让我们修改我们的主类:

```java
@SpringBootApplication
@EnableEurekaClient
@RestController
@RequestMapping("/ratings")
public class RatingServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(RatingServiceApplication.class, args);
    }

    private List<Rating> ratingList = Arrays.asList(
        new Rating(1L, 1L, 2),
        new Rating(2L, 1L, 3),
        new Rating(3L, 2L, 4),
        new Rating(4L, 2L, 5)
    );

    @GetMapping("")
    public List<Rating> findRatingsByBookId(@RequestParam Long bookId) {
        return bookId == null || bookId.equals(0L) ? Collections.EMPTY_LIST : ratingList.stream().filter(r -> r.getBookId().equals(bookId)).collect(Collectors.toList());
    }

    @GetMapping("/all")
    public List<Rating> findAllRatings() {
        return ratingList;
    }
}
```

我们还添加了一个 REST 控制器和一个由属性文件设置的字段，以返回我们将在配置期间设置的值。

让我们添加评级 POJO:

```java
public class Rating {
    private Long id;
    private Long bookId;
    private int stars;

    //standard getters and setters
}
```

### 6.3。属性

现在我们只需要添加两个属性文件:

`src/main/resources`中的`bootstrap.properties`:

```java
spring.cloud.config.name=rating-service
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true

eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

在我们的 Git 存储库中:

```java
spring.application.name=rating-service
server.port=8084

eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

让我们将变更提交给存储库。

### 6.4。运行

一旦所有其他应用程序启动，我们就可以启动评级服务。控制台输出应该如下所示:

```java
DiscoveryClient_RATING-SERVICE/10.1.10.235:rating-service:8083: registering service...
DiscoveryClient_RATING-SERVICE/10.1.10.235:rating-service:8083 - registration status: 204
Tomcat started on port(s): 8084 (http)
```

启动后，我们可以使用浏览器访问刚刚创建的端点。导航到 [`http://localhost:8080/rating-service/ratings/all`](https://web.archive.org/web/20220529012116/http://localhost:8080/rating-service/ratings/all) ，我们将返回包含我们所有评级的 JSON。请注意，我们不是通过端口 8084 直接访问评级服务，而是通过网关服务器。

## 7。结论

现在，我们能够将 Spring Cloud 的各个部分连接成一个正常运行的微服务应用程序。这为我们开始构建更复杂的应用程序奠定了基础。

和往常一样，我们可以在 GitHub 上找到这段源代码。