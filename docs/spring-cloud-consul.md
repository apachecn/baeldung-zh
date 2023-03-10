# 春云咨询快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-consul>

## 1。概述

Spring Cloud Consul 项目为 Spring Boot 应用程序提供了与 consult 的简单集成。

[Consul](https://web.archive.org/web/20220820044630/https://www.consul.io/intro/) 是一个工具，它提供了解决微服务架构中一些最常见挑战的组件:

*   服务发现–自动注册和注销服务实例的网络位置
*   健康检查–检测服务实例何时启动和运行
*   分布式配置——确保所有服务实例使用相同的配置

在本文中，我们将了解如何配置 Spring Boot 应用程序来使用这些特性。

## 2。先决条件

首先，建议快速浏览一下 [Consul](https://web.archive.org/web/20220820044630/https://www.consul.io/intro/) 及其所有功能。

在本文中，我们将使用运行在`localhost:8500`上的 Consul 代理。有关如何安装 Consul 和运行代理的更多详细信息，请参考此[链接](https://web.archive.org/web/20220820044630/https://learn.hashicorp.com/tutorials/consul/get-started-install)。

首先，我们需要将[spring-cloud-starter-consul-all](https://web.archive.org/web/20220820044630/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-cloud-starter-consul-all%22)依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-all</artifactId>
    <version>3.1.1</version>
</dependency>
```

## 3。服务发现

让我们编写第一个 Spring Boot 应用程序，并连接正在运行的 Consul 代理:

```java
@SpringBootApplication
public class ServiceDiscoveryApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(ServiceDiscoveryApplication.class)
          .web(true).run(args);
    }
}
```

**默认情况下，Spring Boot 会在`localhost:8500`尝试连接领事代理。**要使用其他设置，我们需要更新`application.yml`文件:

```java
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
```

然后，如果我们在浏览器中于`http://localhost:8500`访问 Consul 代理的站点，我们将看到我们的应用程序已经在 Consul 中用来自`“${spring.application.name}:${profiles separated by comma}:${server.port}”`的标识符正确注册了。

为了定制这个标识符，我们需要用另一个表达式更新属性`spring.cloud.discovery.instanceId`:

```java
spring:
  application:
    name: myApp
  cloud:
    consul:
      discovery:
        instanceId: ${spring.application.name}:${random.value}
```

如果我们再次运行这个应用程序，我们会看到它是用标识符`“MyApp”`加上一个随机值注册的。我们需要在本地机器上运行应用程序的多个实例。

最后，**要禁用服务发现，我们需要将属性`spring.cloud.consul.discovery.enabled`设置为`false`。**

### 3.1。查找服务

我们已经在 Consul 中注册了应用程序，但是客户端如何找到服务端点呢？我们需要一个 discovery 客户机服务来从 Consul 获得一个运行的、可用的服务。

**Spring 为这个**提供了一个`DiscoveryClient API`，我们可以用`@EnableDiscoveryClient`注释来启用它:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class DiscoveryClientApplication {
    // ...
}
```

然后，我们可以将`DiscoveryClient` bean 注入到我们的控制器中，并访问实例:

```java
@RestController
public class DiscoveryClientController {

    @Autowired
    private DiscoveryClient discoveryClient;

    public Optional<URI> serviceUrl() {
        return discoveryClient.getInstances("myApp")
          .stream()
          .findFirst() 
          .map(si -> si.getUri());
    }
}
```

最后，我们将定义应用程序端点:

```java
@GetMapping("/discoveryClient")
public String discoveryPing() throws RestClientException, 
  ServiceUnavailableException {
    URI service = serviceUrl()
      .map(s -> s.resolve("/ping"))
      .orElseThrow(ServiceUnavailableException::new);
    return restTemplate.getForEntity(service, String.class)
      .getBody();
}

@GetMapping("/ping")
public String ping() {
    return "pong";
}
```

`“myApp/ping”`路径是带有服务端点的 Spring 应用程序名称。Consul 将提供所有名为`“myApp”.`的可用应用程序

## 4。健康检查

Consul 定期检查服务端点的健康状况。

默认情况下， **Spring 实现健康端点返回`200 OK`，如果 app 启动**。如果我们想要定制端点，我们必须更新`application.yml:`

```java
spring:
  cloud:
    consul:
      discovery:
        healthCheckPath: /my-health-check
        healthCheckInterval: 20s
```

因此，Consul 将每 20 秒轮询一次`“/my-health-check”` 端点。

让我们定义我们的自定义健康检查服务来返回一个`FORBIDDEN`状态:

```java
@GetMapping("/my-health-check")
public ResponseEntity<String> myCustomCheck() {
    String message = "Testing my healh check function";
    return new ResponseEntity<>(message, HttpStatus.FORBIDDEN);
}
```

如果我们去 Consul 代理站点，我们会看到我们的应用程序正在失败。为了解决这个问题，`“/my-health-check”`服务应该返回 HTTP `200 OK`状态代码。

## 5。分布式配置

这个特性**允许在所有服务**之间同步配置。Consul 将监视任何配置更改，然后触发所有服务的更新。

首先，我们需要将[spring-cloud-starter-consul-config](https://web.archive.org/web/20220820044630/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-cloud-starter-consul-config%22)依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
    <version>3.1.1</version>
</dependency>
```

我们还需要将 Consul 和 Spring 应用程序名称的设置从`application.yml`文件移动到 Spring 首先加载的`bootstrap.yml`文件。

然后，我们需要启用 Spring Cloud Consul 配置:

```java
spring:
  application:
    name: myApp
  cloud:
    consul:
      host: localhost
      port: 8500
      config:
        enabled: true
```

Spring Cloud Consul Config 将在`“/config/myApp”`查找 Consul 中的属性。因此，如果我们有一个名为`“my.prop”`的属性，我们需要在 Consul 代理站点中创建这个属性。

我们可以通过转到`“KEY/VALUE”`部分来创建属性，然后在`“Create Key”`表单中输入`“/config/myApp/my/prop”`并输入`“Hello World”`作为值。最后，点击`“Create”`按钮。

请记住，如果我们使用的是 Spring 概要文件，我们需要在 Spring 应用程序名称旁边添加概要文件。例如，如果我们使用的是`dev`概要文件，那么 Consul 中的最终路径将是`“/config/myApp,dev”.`

现在，让我们看看注入了属性的控制器是什么样子的:

```java
@RestController
public class DistributedPropertiesController {

    @Value("${my.prop}")
    String value;

    @Autowired
    private MyProperties properties;

    @GetMapping("/getConfigFromValue")
    public String getConfigFromValue() {
        return value;
    }

    @GetMapping("/getConfigFromProperty")
    public String getConfigFromProperty() {
        return properties.getProp();
    }
}
```

和`MyProperties` 类:

```java
@RefreshScope
@Configuration
@ConfigurationProperties("my")
public class MyProperties {
    private String prop;

    // standard getter, setter
}
```

如果我们运行应用程序，字段`value`和`properties`具有来自 Consul 的相同的`“Hello World”`值。

### 5.1。更新配置

在不重启 Spring Boot 应用程序的情况下更新配置怎么样？

如果我们回到 Consul 代理站点，用另一个值如`“New Hello World”`更新属性`“/config/myApp/my/prop”`，那么字段`value`将不会改变，字段`properties`将如预期的那样被更新为`“New Hello World”` 。

这是因为字段`properties`是一个具有`@RefreshScope`注释的`MyProperties`类。**所有用`@RefreshScope`标注的 beans 在配置更改后都会被刷新。**

在现实生活中，我们不应该直接在 Consul 中拥有这些属性，而应该持久地将它们存储在某个地方。我们可以使用[配置服务器](/web/20220820044630/https://www.baeldung.com/spring-cloud-configuration)来做到这一点。

## 6。结论

在本文中，我们已经了解了如何设置我们的 Spring Boot 应用程序，以便与 Consul 一起进行服务发现、定制健康检查规则以及共享分布式配置。

我们还为客户端引入了许多调用这些注册服务的方法。

像往常一样，可以在 GitHub 上找到来源[。](https://web.archive.org/web/20220820044630/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-consul)