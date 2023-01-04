# 春云边车介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-sidecar-intro>

## 1.概观

Spring Cloud 带来了广泛的特性和库，如客户端负载平衡、服务注册/发现、并发控制和配置服务器。另一方面，在微服务领域，用不同的语言和框架编写多语言服务是一种常见的做法。那么，如果我们喜欢在整个生态系统中利用春云呢？春云网飞边车是这里的解决方案。

在本教程中，我们将通过工作示例了解更多关于 Spring Cloud Sidecar 的信息。

## 2.什么是春云边车？

云网飞 Sidecar 的灵感来自于[网飞普拉纳](https://web.archive.org/web/20221128051913/https://github.com/Netflix/Prana)，它可以作为一个实用程序来简化使用非 JVM 语言编写的服务的服务注册，并提高 Spring 云生态系统中端点的互操作性。

使用 Cloud Sidecar，可以在服务注册中心注册非 JVM 服务。此外，服务还可以使用服务发现来发现其他服务，甚至通过主机查找或 [Zuul 代理](/web/20221128051913/https://www.baeldung.com/spring-rest-with-zuul-proxy)来访问配置服务器。能够集成非 JVM 服务的唯一要求是有一个可用的标准健康检查端点。

## 3.示例应用程序

我们的示例用例由 3 个应用程序组成。为了展示云网飞 sidecar 的最佳性能，我们将在 NodeJS 中创建一个`/hello`端点，然后通过一个名为 Sidecar 的 Spring 应用程序将其暴露给我们的生态系统。我们还将开发另一个 Spring Boot 应用程序，使用服务发现和 Zuul 来回应`/hello`端点响应。

在这个项目中，我们的目标是涵盖请求的两个流程:

*   用户调用 echo Spring Boot 应用程序上的 echo 端点。echo 端点使用`DiscoveryClient`从 Eureka 查找 hello 服务 URL，即指向 NodeJS 服务的 URL。然后，echo 端点调用 NodeJS 应用程序上的 hello 端点
*   在 Zuul 代理的帮助下，用户直接从 echo 应用程序调用 hello 端点

### 3.1.NodeJS Hello 端点

让我们首先创建一个名为`hello.js`的 JS 文件。我们使用`express`来服务我们的 hello 请求。在我们的`hello.js`文件中，我们引入了三个端点——默认的“/”端点、`/hello`端点和`/health`端点，以满足 Spring Cloud Sidecar 的要求:

```java
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
    res.send('Hello World!')
})

app.get('/health', (req, res) => {
    res.send({ "status":"UP"})
})

app.get('/hello/:me', (req, res) => {
    res.send('Hello ' + req.params.me + '!')
})

app.listen(port, () => {
    console.log(`Hello app listening on port ${port}`)
})
```

接下来，我们将安装`express`:

```java
npm install express
```

最后，让我们开始我们的应用:

```java
node hello.js
```

随着应用程序的启动，让我们`curl`hello 端点:

```java
curl http://localhost:3000/hello/baeldung
Hello baeldung!
```

然后，我们测试健康端点:

```java
curl http://localhost:3000/health
status":"UP"}
```

因为我们已经为下一步准备好了节点应用程序，所以我们将对它进行 Springify。

### 3.2.边车应用

首先，我们需要[安装一台 Eureka 服务器](/web/20221128051913/https://www.baeldung.com/spring-cloud-netflix-eureka)。尤里卡服务器启动后，我们可以访问: [http://127.0.0.1:8761](https://web.archive.org/web/20221128051913/http://127.0.0.1:8761/)

让我们把 [`spring-cloud-netflix-sidecar`](https://web.archive.org/web/20221128051913/https://search.maven.org/artifact/org.springframework.cloud/spring-cloud-netflix-sidecar) 加为一个依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-sidecar</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

**需要注意的是，此时`spring-cloud-netflix-sidecar`的最新版本是`2.2.10.RELEASE`，并且只支持弹簧启动`2.3.12.RELEASE`。因此，目前最新版本的 Spring Boot 与网飞边车不兼容。**

然后让我们实现启用了 sidecar 的 Spring Boot 应用程序类:

```java
@SpringBootApplication
@EnableSidecar
public class SidecarApplication {
    public static void main(String[] args) {
        SpringApplication.run(SidecarApplication.class, args);
    }
}
```

下一步，我们必须设置连接到 Eureka 的属性。此外，我们使用 NodeJS hello 应用程序的端口和健康 URI 设置 sidecar 配置:

```java
server.port: 8084
spring:
  application:
    name: sidecar
eureka:
  instance:
    hostname: localhost
    leaseRenewalIntervalInSeconds: 1
    leaseExpirationDurationInSeconds: 2
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka
    healthcheck:
      enabled: true
sidecar:
  port: 3000
  health-uri: http://localhost:3000/health
```

现在我们可以开始我们的应用程序了。在我们的应用程序成功启动之后，Spring 在 Eureka 服务器中注册了一个名为“hello”的服务。

为了检查它是否工作，我们可以访问端点:[http://localhost:8084/hosts/sidecar](https://web.archive.org/web/20221128051913/http://localhost:8084/hosts/sidecar)。

`@EnableSidecar`不仅仅是在尤里卡注册侧服务的标志。这也导致添加了`@EnableCircuitBreaker`和`@EnableZuulProxy`，随后，我们的 Spring Boot 应用程序受益于 [Hystrix](/web/20221128051913/https://www.baeldung.com/introduction-to-hystrix) 和[Zuul](/web/20221128051913/https://www.baeldung.com/spring-rest-with-zuul-proxy)

现在，我们已经准备好了 Spring 应用程序，让我们进入下一步，看看我们的生态系统中服务之间的通信是如何工作的。

### 3.3.Echo 应用也说你好！

对于 echo 应用程序，我们将在服务发现的帮助下创建一个调用 NodeJS hello 端点的端点。此外，我们将启用 Zuul 代理来显示这两个服务之间通信的其他选项。

首先，让我们添加依赖项:

```java
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
     <version>2.2.10.RELEASE</version>
 </dependency>
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
     <version>2.2.10.RELEASE</version>
 </dependency>
```

**为了与 sidecar 应用程序保持一致，我们在 echo 应用程序中使用了相同版本的`2.2.10.RELEASE for`，两者都依赖于`[spring-cloud-starter-netflix-zuul](https://web.archive.org/web/20221128051913/https://search.maven.org/artifact/org.springframework.cloud/spring-cloud-starter-netflix-zuul)`和`[spring-cloud-starter-netflix-eureka-client](https://web.archive.org/web/20221128051913/https://search.maven.org/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client/2.2.10.RELEASE/jar)`。**

然后让我们创建 Spring Boot 主类并启用 Zuul 代理:

```java
@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy
public class EchoApplication {
    // ...
}
```

然后，我们像在上一节中一样配置 Eureka 客户端:

```java
server.port: 8085
spring:
  application:
    name: echo
eureka:
  instance:
    hostname: localhost
    leaseRenewalIntervalInSeconds: 1
    leaseExpirationDurationInSeconds: 2
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka
 ...
```

接下来，我们启动 echo 应用程序。启动之后，我们可以检查我们两个服务之间的互操作性。

要检查 sidecar 应用程序，让我们查询 echo 服务的元数据:

```java
curl http://localhost:8084/hosts/echo
```

然后，为了验证 echo 应用程序是否可以调用 sidecar 应用程序公开的 NodeJS 端点，让我们使用 Zuul 代理的魔力并卷曲这个 url:

```java
curl http://localhost:8085/sidecar/hello/baeldung
Hello baeldung!
```

既然我们已经验证了一切正常，那么让我们尝试另一种方法来调用 hello 端点。首先，我们将在 echo 应用程序中创建一个控制器并注入`DiscoveryClient.` ,然后我们添加一个`GET`端点，该端点使用`DiscoveryClient`查询 hello 服务并用`RestTemplate:`调用它

```java
@Autowired
DiscoveryClient discoveryClient;

@GetMapping("/hello/{me}")
public ResponseEntity<String> echo(@PathVariable("me") String me) {
    List<ServiceInstance> instances = discoveryClient.getInstances("sidecar");
    if (instances.isEmpty()) {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body("hello service is down");
    }
    String url = instances.get(0).getUri().toString();
    return ResponseEntity.ok(restTemplate.getForObject(url + "/hello/" + me, String.class));
}
```

让我们重新启动 echo 应用程序，并执行这个 curl 来验证从 echo 应用程序调用的 echo 端点:

```java
curl http://localhost:8085/hello/baeldung
Hello baeldung!
```

或者更有趣一点，从 sidecar 应用程序调用它:

```java
curl http://localhost:8084/echo/hello/baeldung
Hello baeldung!
```

## 4.结论

在本文中，我们了解了云网飞 Sidecar，并用 NodeJS 和两个 Spring 应用程序构建了一个工作样本，展示了它在 Spring 生态系统中的用法。

与往常一样，GitHub 上的[提供了示例的完整代码。](https://web.archive.org/web/20221128051913/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-netflix-sidecar)