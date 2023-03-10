# 获取 Spring Boot 的运行端口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-running-port>

## 1.概观

Spring Boot 应用程序嵌入了一个 web 服务器，有时，我们可能希望在运行时发现 HTTP 端口。

在本教程中，我们将介绍如何在 Spring Boot 应用程序中以编程方式获取 HTTP 端口。

## 2.介绍

### 2.1.我们的 Spring Boot 应用程序

我们将创建一个简单的 Spring Boot 应用程序示例，快速展示在运行时发现 HTTP 端口的方法:

```java
@SpringBootApplication
public class GetServerPortApplication {
    public static void main(String[] args) {
        SpringApplication.run(GetServerPortApplication.class, args);
    }
} 
```

### 2.2.设置端口的两种情况

通常，[配置 Spring Boot 应用程序的 HTTP 端口](/web/20221208143917/https://www.baeldung.com/spring-boot-change-port#properties)的最直接方式是在配置文件`application.properties`或`application.yml`中定义端口。

例如，在`application.properties`文件中，我们可以将`7777`设置为应用程序运行的端口:

```java
server.port=7777 
```

或者，**不是定义一个固定的端口，我们可以通过将“`0`”设置为“`server.port`”属性**的值，让 Spring Boot 应用程序在一个随机的端口上运行:

```java
server.port=0 
```

接下来，让我们浏览这两个场景，并讨论在运行时以编程方式获取端口的不同方法。

在本教程中，我们将在单元测试中发现服务器端口。

## 3.在运行时获取固定端口

让我们创建一个属性文件`application-fixedport.properties`并在其中定义一个固定端口`7777` :

```java
server.port=7777
```

接下来，我们将尝试在单元测试类中获取端口:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = GetServerPortApplication.class,
  webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
@ActiveProfiles("fixedport")
public class GetServerFixedPortUnitTest {
    private final static int EXPECTED_PORT = 7777;
    ....
} 
```

在我们看到测试方法之前，让我们快速看一下测试类的注释:

*   `@RunWith(SpringRunner.class)`–这将通过弹簧`TestContext`加入 JUnit 测试
*   `@SpringBootTest( … SpringBootTest.WebEnvironment.DEFINED_PORT)`–在`SpringBootTest`中，我们将把`DEFINED_PORT`用于嵌入式 web 服务器
*   `@ActiveProfiles(“fixedport”)`–有了这个注释，我们启用了[弹簧轮廓](/web/20221208143917/https://www.baeldung.com/spring-profiles)`fixedport`，这样我们的`application-fixedport.properties`将被加载

### 3.1.使用`@Value(“${server.port}”)`注释

由于`application-fixedport.properties`文件将被加载，我们可以使用 [`@Value`注释](/web/20221208143917/https://www.baeldung.com/spring-value-annotation)来获得`server.port`属性:

```java
@Value("${server.port}")
private int serverPort;

@Test
public void givenFixedPortAsServerPort_whenReadServerPort_thenGetThePort() {
    assertEquals(EXPECTED_PORT, serverPort);
} 
```

### 3.2.使用`ServerProperties`类

`[ServerProperties](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/web/ServerProperties.html)`保存嵌入式 web 服务器的属性，比如端口、地址和服务器头。

我们可以注入一个`ServerProperties`组件并从中获取端口:

```java
@Autowired
private ServerProperties serverProperties;

@Test
public void givenFixedPortAsServerPort_whenReadServerProps_thenGetThePort() {
    int port = serverProperties.getPort();

    assertEquals(EXPECTED_PORT, port);
}
```

到目前为止，我们已经学习了两种在运行时获得固定端口的方法。接下来，让我们看看如何在随机端口场景中发现端口。

## 4.运行时获取随机端口

这一次，让我们创建另一个属性文件`application-randomport.properties`:

```java
server.port=0
```

如上面的代码所示，我们允许 Spring Boot 在 web 服务器启动时随机选择一个空闲端口。

同样，让我们创建另一个单元测试类:

```java
....
@ActiveProfiles("randomport")
public class GetServerRandomPortUnitTest {
...
} 
```

这里，我们需要激活"`randomport` " Spring profile 来加载相应的属性文件。

我们已经学习了两种在运行时发现固定端口的方法。然而，他们不能帮助我们得到随机端口:

```java
@Value("${server.port}")
private int randomServerPort;

@Test
public void given0AsServerPort_whenReadServerPort_thenGet0() {
    assertEquals(0, randomServerPort);
}

@Autowired
private ServerProperties serverProperties;

@Test
public void given0AsServerPort_whenReadServerProps_thenGet0() {
    int port = serverProperties.getPort();

    assertEquals(0, port);
} 
```

如两种测试方法所示，**`@Value(“${server.port}”)`和`serverProperties.getPort()`都报告端口为“0”。**显然，这不是我们期待的正确港口。

### 4.1.使用`ServletWebServerApplicationContext`

如果嵌入式 web 服务器启动，Spring Boot 会启动一个 [`ServletWebServerApplicationContext`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/servlet/context/ServletWebServerApplicationContext.html) 。

因此，我们可以从 context 对象中获取`[WebServer](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/server/WebServer.html)` 来获取服务器信息或操纵服务器:

```java
@Autowired
private ServletWebServerApplicationContext webServerAppCtxt;

@Test
public void given0AsServerPort_whenReadWebAppCtxt_thenGetThePort() {
    int port = webServerAppCtxt.getWebServer().getPort();

    assertTrue(port > 1023);
} 
```

在上面的测试中，我们检查端口是否大于 1023。这是因为 0-1023 是系统端口。

### 4.2.处理`ServletWebServerInitializedEvent`

一个 Spring 应用程序可以发布各种[事件](/web/20221208143917/https://www.baeldung.com/spring-events)和`[EventListeners](/web/20221208143917/https://www.baeldung.com/spring-events#annotation-driven)`处理这些事件。

当嵌入式 web 服务器启动时，将发布一个`[ServletWebServerInitializedEvent](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/servlet/context/ServletWebServerInitializedEvent.html) `。此事件包含有关 web 服务器的信息。

因此，我们可以创建一个`EventListener `来从这个事件中获取端口:

```java
@Service
public class ServerPortService {
    private int port;

    public int getPort() {
        return port;
    }

    @EventListener
    public void onApplicationEvent(final ServletWebServerInitializedEvent event) {
        port = event.getWebServer().getPort();
    }
}
```

我们可以将服务组件注入到我们的测试类中，以快速获得随机端口:

```java
@Autowired
private ServerPortService serverPortService;

@Test
public void given0AsServerPort_whenReadFromListener_thenGetThePort() {
    int port = serverPortService.getPort();

    assertTrue(port > 1023);
} 
```

## 5.结论

通常，我们在属性文件或 YAML 文件中配置 Spring Boot 应用程序的服务器端口，我们可以设置固定或随机端口。

在本文中，我们讨论了在运行时获得固定和随机端口的不同方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-environment)