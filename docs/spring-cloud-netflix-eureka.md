# 网飞春云简介——尤里卡

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-netflix-eureka>

## 1。概述

在本教程中，我们将通过`Spring Cloud Netflix Eureka.`介绍`client-side service`发现

**`Client-side service discovery`允许服务在不硬编码主机名和端口的情况下相互查找和通信。**这种架构中唯一的“固定点”是每个服务必须注册的`service registry,`。

一个缺点是所有客户端都必须实现某种逻辑来与这个固定点进行交互。这假设在实际请求之前有一次额外的网络往返。

借助网飞尤里卡，每个客户端都可以同时充当服务器，将其状态复制到连接的对等端。换句话说，客户机在一个`service registry,`中检索所有连接的对等体的列表，并通过负载平衡算法向其他服务发出所有进一步的请求。

要被告知客户端的存在，它们必须向注册中心发送心跳信号。

为了实现本教程的目标，我们将实现三个`microservices`:

*   一个`service registry` ( `**Eureka Server**`)
*   一个`REST`服务，它在注册中心注册自己(`**Eureka Client**`)
*   一个 web 应用程序，它使用`REST`服务作为一个注册感知客户端(`Spring Cloud Netflix **Feign Client**` )

    ## 延伸阅读

    ## [网飞春云指南——海斯特里克斯](/web/20220807183451/https://www.baeldung.com/spring-cloud-netflix-hystrix)

    这篇文章展示了如何使用 Spring Cloud Hystrix 在应用程序逻辑中设置回退。[阅读更多](/web/20220807183451/https://www.baeldung.com/spring-cloud-netflix-hystrix) →

    ## [带祖尔语代理的弹簧座](/web/20220807183451/https://www.baeldung.com/spring-rest-with-zuul-proxy)

    探索 Spring REST API 的 Zuul 代理的使用，围绕 CORS 和浏览器的同源策略约束工作。[阅读更多](/web/20220807183451/https://www.baeldung.com/spring-rest-with-zuul-proxy) →

## 2。尤里卡服务器

为服务注册实现一个`Eureka Server`非常简单:

1.  将`[spring-cloud-starter-netflix-eureka-server](https://web.archive.org/web/20220807183451/https://search.maven.org/search?q=spring-cloud-starter-netflix-eureka-server)`添加到依赖项
2.  在 [`@SpringBootApplication`](/web/20220807183451/https://www.baeldung.com/spring-boot-application-configuration) 中启用尤里卡服务器，用`@EnableEurekaServer`对其进行注释
3.  配置一些属性

让我们一步一步来。

首先，我们将创建一个新的 Maven 项目，并将依赖项放入其中。注意，我们正在将`[spring-cloud-starter-parent](https://web.archive.org/web/20220807183451/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-parent%22)`导入到本教程中描述的所有项目中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-parent</artifactId>
            <version>Greenwich.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

我们可以在 Spring 的项目文档中查看最新的 Spring Cloud 版本。

然后我们将创建主应用程序类:

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

最后，我们将以`YAML`格式配置属性，因此`application.yml`将是我们的配置文件:

```java
server:
  port: 8761
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
```

**我们在这里配置一个应用程序端口；`Eureka `服务器的默认设置是`8761`。**我们告诉内置的`Eureka Client`不要向自己注册，因为我们的应用程序应该充当服务器。

现在我们将浏览器指向 [http://localhost:8761](https://web.archive.org/web/20220807183451/http://localhost:8761/) 来查看`Eureka`仪表板，稍后我们将在这里检查已注册的实例。

目前，我们可以看到一些基本指标，如状态和健康指标:

[![Screenshot_20160819_073151](img/e6156c6a4933c2066633d4db31c2af9a.png)](/web/20220807183451/https://www.baeldung.com/wp-content/uploads/2016/08/Screenshot_20160819_073151.png)

## 3。尤里卡客户端

为了让`@SpringBootApplication`具有发现意识，我们必须将`Spring Discovery Client` (例如， [`spring-cloud-starter-netflix-eureka-client`](https://web.archive.org/web/20220807183451/https://search.maven.org/search?q=spring-cloud-starter-netflix-eureka-client) )包含到我们的`classpath.` 中

然后我们需要用`@EnableDiscoveryClient` 或 `@EnableEurekaClient.` 注释`@Configuration`。注意，如果我们在类路径上有`spring-cloud-starter-netflix-eureka-client`依赖，那么这个注释是可选的。

后者明确告诉`Spring Boot`使用 Spring 网飞 Eureka 进行服务发现。为了给我们的客户端应用程序添加一些样本，我们还将在`pom.xml`中包含 [`spring-boot-starter-web`](https://web.archive.org/web/20220807183451/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22) 包，并实现一个`REST`控制器。

但是首先，我们将添加依赖项。同样，我们可以让`spring-cloud-starter-parent` 依赖关系来为我们计算工件版本:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

这里我们将实现主要的应用程序类:

```java
@SpringBootApplication
@RestController
public class EurekaClientApplication implements GreetingController {

    @Autowired
    @Lazy
    private EurekaClient eurekaClient;

    @Value("${spring.application.name}")
    private String appName;

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }

    @Override
    public String greeting() {
        return String.format(
          "Hello from '%s'!", eurekaClient.getApplication(appName).getName());
    }
}
```

和`GreetingController`界面:

```java
public interface GreetingController {
    @RequestMapping("/greeting")
    String greeting();
}
```

除了接口，我们还可以简单地在`EurekaClientApplication`类中声明映射。如果我们想在服务器和客户端之间共享它，这个接口会很有用。

接下来，我们必须设置一个带有已配置的`Spring`应用程序名称的`application.yml`，以便在已注册的应用程序列表中惟一地标识我们的客户端。

**我们可以让`Spring Boot`为我们选择一个随机端口，因为稍后我们将使用它的名称访问这个服务。**

最后，我们必须告诉我们的客户注册中心的位置:

```java
spring:
  application:
    name: spring-cloud-eureka-client
server:
  port: 0
eureka:
  client:
    serviceUrl:
      defaultZone: ${EUREKA_URI:http://localhost:8761/eureka}
  instance:
    preferIpAddress: true
```

我们决定以这种方式设置我们的 Eureka 客户机，因为这种服务以后应该很容易扩展。

现在我们将运行客户端，并再次将浏览器指向 [`http://localhost:8761`](https://web.archive.org/web/20220807183451/https://localhost:8761/) ,以查看它在 Eureka 仪表板上的注册状态。通过使用仪表板，我们可以做进一步的配置，比如出于管理目的将注册客户端的主页与仪表板链接起来。但是，配置选项超出了本文的范围:

[![Screenshot_20160819_101810](img/78ede6aba8887158a90700f022e04cd0.png)](/web/20220807183451/https://www.baeldung.com/wp-content/uploads/2016/08/Screenshot_20160819_101810.png)

## 4。假装客户端

为了用三个相关的微服务完成我们的项目，我们现在将使用`Spring Netflix Feign Client`实现一个使用`REST`的 web 应用程序。

**将`Feign`想象成一个发现感知 [`Spring` `RestTemplate`](/web/20220807183451/https://www.baeldung.com/rest-template) 使用接口与端点通信。这些接口将在运行时自动实现，它使用的是`service-names`，而不是`service-urls`。**

如果没有`Feign,`，我们将不得不把`EurekaClient`的一个实例自动连接到我们的控制器中，这样我们就可以通过`service-name`作为`Application`对象接收服务信息。

我们将使用这个`Application`来获取这个服务的所有实例的列表，选择一个合适的实例，然后使用这个`InstanceInfo`来获取主机名和端口。有了这个，我们可以对任何`http client:`做一个标准的请求

```java
@Autowired
private EurekaClient eurekaClient;

@RequestMapping("/get-greeting-no-feign")
public String greeting(Model model) {

    InstanceInfo service = eurekaClient
      .getApplication(spring-cloud-eureka-client)
      .getInstances()
      .get(0);

    String hostName = service.getHostName();
    int port = service.getPort();

    // ...
}
```

还可以使用一个`RestTemplate`来通过名称访问`Eureka`客户端服务，但是这个主题超出了本文的范围。

为了设置我们的`Feign Client`项目，我们将向它的`pom.xml`添加以下四个依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

`Feign Client`位于`[spring-cloud-starter-feign](https://web.archive.org/web/20220807183451/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-feign%22)`包装内。要启用它，我们必须用`@EnableFeignClients`注释一个`@Configuration`。要使用它，我们只需用`@FeignClient(“service-name”)`注释一个接口，并将其自动连接到控制器中。

创建这样的`Feign` `Clients`的一个好方法是用 [`@RequestMapping`](/web/20220807183451/https://www.baeldung.com/spring-requestmapping) 带注释的方法创建接口，并将它们放入一个单独的模块中。这样，它们可以在服务器和客户机之间共享。在服务器端，我们可以将它们实现为`@Controller`，在客户端，它们可以被扩展和注释为`@FeignClient`。

此外， [`spring-cloud-starter-eureka package`](https://web.archive.org/web/20220807183451/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22) 需要包含在项目中，并通过用`@EnableEurekaClient`注释主应用程序类来启用。

[`spring-boot-starter-web`](https://web.archive.org/web/20220807183451/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22) 和 [`spring-boot-starter-thymeleaf`](https://web.archive.org/web/20220807183451/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-thymeleaf%22) 依赖项用于[呈现包含从我们的`REST`服务获取的数据的视图](/web/20220807183451/https://www.baeldung.com/spring-mvc-form-tutorial)。

这将是我们的`Feign Client`界面:

```java
@FeignClient("spring-cloud-eureka-client")
public interface GreetingClient {
    @RequestMapping("/greeting")
    String greeting();
}
```

这里我们将实现主应用程序类，它同时充当控制器:

```java
@SpringBootApplication
@EnableFeignClients
@Controller
public class FeignClientApplication {
    @Autowired
    private GreetingClient greetingClient;

    public static void main(String[] args) {
        SpringApplication.run(FeignClientApplication.class, args);
    }

    @RequestMapping("/get-greeting")
    public String greeting(Model model) {
        model.addAttribute("greeting", greetingClient.greeting());
        return "greeting-view";
    }
} 
```

这将是我们视图的 HTML 模板:

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Greeting Page</title>
    </head>
    <body>
        <h2 th:text="${greeting}"/>
    </body>
</html>
```

`application.yml`配置文件与上一步几乎相同:

```java
spring:
  application:
    name: spring-cloud-eureka-feign-client
server:
  port: 8080
eureka:
  client:
    serviceUrl:
      defaultZone: ${EUREKA_URI:http://localhost:8761/eureka}
```

现在我们可以构建和运行这个服务了。最后，我们将浏览器指向 [`http://localhost:8080/get-greeting`](https://web.archive.org/web/20220807183451/http://localhost:8080/get-greeting) ，它应该会显示如下内容:

```java
Hello from SPRING-CLOUD-EUREKA-CLIENT!
```

## 5.`TransportException:`无法在任何已知的服务器上执行请求

在运行 Eureka 服务器时，我们经常会遇到如下异常:

```java
com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
```

基本上，这是由于`application.properties`或`application.yml`中的错误配置造成的。`Eureka`为客户端提供了两个可配置的属性:

*   `registerWithEureka:`如果我们将这个属性设置为`true,`，那么当服务器启动时，内置客户端将尝试向 Eureka 服务器注册自己。
*   `fetchRegistry:`如果我们将该属性配置为 true，内置客户端将尝试获取`Eureka`注册表。

现在**当我们启动 Eureka 服务器时，我们不想注册内置客户端来配置服务器**。

如果我们将上述属性标记为`true`(或者不要配置它们，因为它们默认为`true`)，那么在启动服务器时，内置客户端会尝试向`Eureka`服务器注册自己，并尝试获取注册表，但注册表还不可用。结果我们得到`TransportException`。

所以我们不应该在`Eureka`服务器应用程序中将这些属性配置为`true`。应该输入`application.yml`的正确设置是:

```java
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
```

## 6。结论

在本文中，我们学习了如何使用`Spring Netflix Eureka Server`实现服务注册中心，并向其注册一些`Eureka Clients`。

由于步骤 3 中的`Eureka Client`监听随机选择的端口，如果没有来自注册表的信息，它就不知道自己的位置。有了一个`Feign Client`和我们的注册中心，我们可以定位和使用`REST`服务，即使位置发生了变化。

最后，我们看到了在微服务架构中使用服务发现的总体情况。

像往常一样，我们可以在`[on GitHub](https://web.archive.org/web/20220807183451/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-eureka),` 中找到源代码，其中也包括一组与`Docker`相关的文件，可以使用 [`docker-compose`](/web/20220807183451/https://www.baeldung.com/dockerizing-spring-boot-application) 从我们的项目中创建容器。