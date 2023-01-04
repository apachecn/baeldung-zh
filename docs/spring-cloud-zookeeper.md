# 《春云动物园管理员》简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-zookeeper>

## 1。简介

在本文中，我们将了解 Zookeeper 以及它如何用于服务发现，服务发现是关于云中服务的集中知识。

Spring Cloud Zookeeper 通过自动配置和绑定到 Spring 环境，为 Spring Boot 应用程序提供了 [Apache Zookeeper](https://web.archive.org/web/20220630005618/https://zookeeper.apache.org/) 集成。

## 2。服务发现设置

我们将创建两个应用程序:

*   将提供服务的应用程序(本文中称为**服务提供商** `)`
*   将使用该服务的应用程序(称为**服务消费者**

Apache Zookeeper 将在我们的服务发现设置中充当协调者。Apache Zookeeper 安装说明可从以下[链接](https://web.archive.org/web/20220630005618/https://zookeeper.apache.org/doc/current/zookeeperStarted.html)获得。

## 3。服务提供商注册

我们将通过添加`spring-cloud-starter-zookeeper-discovery`依赖项并在主应用程序中使用注释`@EnableDiscoveryClient`来启用服务注册。

下面，我们将为返回“Hello World！”的服务一步步展示这个过程以响应 GET 请求。

### 3.1。Maven 依赖关系

首先，让我们将所需的 `[spring-cloud-starter-zookeeper-discovery](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-cloud-starter-zookeeper-discovery), [spring-web](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-web%22), [spring-cloud-dependencies](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20%20AND%20a%3A%22spring-cloud-dependencies%22)` 和`[spring-boot-starter](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter%22%20AND%20g%3A%22org.springframework.boot%22)` 依赖项添加到我们的`pom.xml`文件中:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
	<version>2.2.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
	<artifactId>spring-web</artifactId>
        <version>5.1.14.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
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
```

### 3.2。服务提供商注释

接下来，我们将用`@EnableDiscoveryClient`注释我们的主类。这将使`HelloWorld`应用发现感知:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class HelloWorldApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }
}
```

和一个简单的控制器:

```java
@GetMapping("/helloworld")
public String helloWorld() {
    return "Hello World!";
}
```

### 3.3。YAML 配置

现在让我们创建一个 YAML `Application.yml`文件，该文件将用于配置应用程序日志级别，并通知 Zookeeper 该应用程序支持发现。

向 Zookeeper 注册的应用程序的名称是最重要的。稍后在服务消费者中，`feign`客户机将在服务发现期间使用这个名称:

```java
spring:
  application:
    name: HelloWorld
  cloud:
    zookeeper:
      discovery:
        enabled: true
logging:
  level:
    org.apache.zookeeper.ClientCnxn: WARN
```

spring boot 应用程序在默认端口 2181 上寻找 zookeeper。如果 zookeeper 位于其他地方，则需要添加配置:

```java
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```

## 4。服务消费者

现在我们将创建一个 REST 服务消费者，并使用 Spring 网飞 Feign Client 注册它。

### 4.1。Maven 依赖关系

首先，让我们将所需的`[spring-cloud-starter-zookeeper-discovery](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-cloud-starter-zookeeper-discovery), [spring-web](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-web%22), [spring-cloud-dependencies](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20%20AND%20a%3A%22spring-cloud-dependencies%22), [spring-boot-starter-actuator](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-actuator%22%20AND%20g%3A%22org.springframework.boot%22)`和`[spring-cloud-starter-feign](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-cloud-starter-feign)`依赖项添加到我们的`pom.xml`文件中:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
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
```

### 4.2。服务消费者注释

与服务提供者一样，我们将使用`@EnableDiscoveryClient`对主类进行注释，使其具有发现意识:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GreetingApplication {

    public static void main(String[] args) {
        SpringApplication.run(GreetingApplication.class, args);
    }
} 
```

### 4.3。使用虚拟客户端发现服务

我们将使用网飞的一个项目，让你定义一个声明性的 REST 客户端。我们声明 URL 的外观，并由 feign 负责连接到 REST 服务。

`Feign Client`通过`[spring-cloud-starter-feign](https://web.archive.org/web/20220630005618/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-feign%22)`包导入。我们将用`@EnableFeignClients`注释一个`@Configuration`,以便在应用程序中使用它。

最后，我们用`@FeignClient(“service-name”)`注释一个接口，并将其自动连接到我们的应用程序中，以便我们以编程方式访问该服务。

在这里的注释`@FeignClient(name = “HelloWorld”)`中，我们引用我们之前创建的服务生产者的`service-name`。

```java
@Configuration
@EnableFeignClients
@EnableDiscoveryClient
public class HelloWorldClient {

    @Autowired
    private TheClient theClient;

    @FeignClient(name = "HelloWorld")
    interface TheClient {

        @RequestMapping(path = "/helloworld", method = RequestMethod.GET)
        @ResponseBody
	String helloWorld();
    }
    public String HelloWorld() {
        return theClient.HelloWorld();
    }
}
```

### 4.4。控制器类别

下面是一个简单的服务控制器类，它将调用我们的 feign client 类上的服务提供者函数，通过注入的接口`helloWorldClient`对象来消费服务(其细节通过服务发现来抽象),并在响应中显示它:

```java
@RestController
public class GreetingController {

    @Autowired
    private HelloWorldClient helloWorldClient;

    @GetMapping("/get-greeting")
    public String greeting() {
        return helloWorldClient.helloWorld();
    }
}
```

### 4.5。YAML 配置

接下来，我们创建一个 YAML 文件`Application.yml`,与之前使用的文件非常相似。它配置应用程序的日志级别:

```java
logging:
  level:
    org.apache.zookeeper.ClientCnxn: WARN
```

应用程序在默认端口`2181`上寻找 Zookeeper。如果 Zookeeper 位于其他地方，则需要添加配置:

```java
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```

## 5。测试设置

HelloWorld REST 服务在部署时向 Zookeeper 注册。然后，充当服务消费者的`Greeting`服务使用假客户端调用`HelloWorld`服务。

现在我们可以构建和运行这两个服务。

最后，我们将浏览器指向 [`http://localhost:8083/get-greeting`](https://web.archive.org/web/20220630005618/http://localhost:8080/get-greeting) ，它应该会显示:

```java
Hello World!
```

## 6。结论

在本文中，我们已经看到了如何使用`Spring Cloud Zookeeper`实现服务发现，并且我们在 Zookeeper 服务器中注册了一个名为`HelloWorld`的服务，由`Greeting`服务使用`Feign Client`来发现和消费，而不知道它的位置细节。

一如既往，本文的代码可以在 [GitHub](https://web.archive.org/web/20220630005618/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-zookeeper) 上获得。