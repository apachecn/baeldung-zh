# Java 测试中的 Docker 测试容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/docker-test-containers>

## 1.介绍

**在本教程中，我们将学习 Java `TestContainers`库。**它允许我们在测试中使用 Docker 容器。因此，我们可以编写依赖于外部资源的自包含集成测试。

我们可以在测试中使用任何具有 docker 图像的资源。例如，有数据库、web 浏览器、web 服务器和消息队列的图像。因此，我们可以在测试中将它们作为容器运行。

## 2.要求

`TestContainers` 库可以与 Java 8 及更高版本一起使用。此外，它与 JUnit 规则 API 兼容。

首先，让我们定义核心功能的 maven 依赖性:

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.11.4</version>
</dependency>
```

还有用于特殊容器的模块。在本教程中，我们将使用`PostgreSQL `和`Selenium. `

让我们添加相关的依赖项:

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql </artifactId>
    <version>1.11.4</version>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>selenium </artifactId>
    <version>1.11.4</version>
</dependency>
```

我们可以在 Maven Central 上找到最新版本。

另外，我们需要 Docker 来运行容器。参考[码头文件](https://web.archive.org/web/20221017045625/https://docs.docker.com/install/)获取安装说明。

确保您能够在您的测试环境中运行 Docker 容器。

## 3.使用

让我们配置一个通用容器规则:

```java
@ClassRule
public static GenericContainer simpleWebServer
 = new GenericContainer("alpine:3.2")
   .withExposedPorts(80)
   .withCommand("/bin/sh", "-c", "while true; do echo "
     + "\"HTTP/1.1 200 OK\n\nHello World!\" | nc -l -p 80; done");
```

我们通过指定 docker 图像名来构建一个`GenericContainer`测试规则。然后，我们用构建器方法对其进行配置:

*   我们使用`withExposedPorts `从容器中公开一个端口
*   `withCommand` 定义一个容器命令。它将在容器启动时执行。

规则被注释为`@ClassRule. `结果，它将在该类中的任何测试运行之前启动 Docker 容器`.` 容器将在所有方法被执行之后被销毁。

如果您应用`@Rule`注释，`GenericContainer` 规则将为每个测试方法启动一个新的容器。当测试方法完成时，它将停止容器。

**我们可以使用 IP 地址和端口与运行在容器中的进程进行通信**:

```java
@Test
public void givenSimpleWebServerContainer_whenGetReuqest_thenReturnsResponse()
  throws Exception {
    String address = "http://" 
      + simpleWebServer.getContainerIpAddress() 
      + ":" + simpleWebServer.getMappedPort(80);
    String response = simpleGetRequest(address);

    assertEquals(response, "Hello World!");
}
```

## 4.使用模式

有几个`usage modes` 测试容器。我们看到了一个运行`GenericContainer.`的例子

`TestContainers`库也有专门功能的规则定义。它们用于常见数据库的容器，如 MySQL、PostgreSQL 还有一些像 web 客户端。

尽管我们可以将它们作为通用容器运行，但是专门化提供了扩展的便利方法。

### 4.1.数据库

假设我们需要一个数据库服务器来进行数据访问层集成测试。借助 TestContainers 库，我们可以在容器中运行数据库。

例如，我们用`PostgreSQLContainer`规则启动一个 PostgreSQL 容器。然后，我们能够使用辅助方法。**这些是数据库连接的`getJdbcUrl, getUsername, getPassword` :**

```java
@Rule
public PostgreSQLContainer postgresContainer = new PostgreSQLContainer();

@Test
public void whenSelectQueryExecuted_thenResulstsReturned()
  throws Exception {
    String jdbcUrl = postgresContainer.getJdbcUrl();
    String username = postgresContainer.getUsername();
    String password = postgresContainer.getPassword();
    Connection conn = DriverManager
      .getConnection(jdbcUrl, username, password);
    ResultSet resultSet = 
      conn.createStatement().executeQuery("SELECT 1");
    resultSet.next();
    int result = resultSet.getInt(1);

    assertEquals(1, result);
}
```

也可以将 PostgreSQL 作为通用容器运行。但是配置连接会更加困难。

### 4.2.Web 驱动程序

另一个有用的场景是用 web 浏览器运行容器。`BrowserWebDriverContainer `规则允许运行`docker-selenium `容器中的`Chrome `和`Firefox `。然后，我们用`RemoteWebDriver. `来管理它们

这对于自动化 web 应用程序的 UI/验收测试非常有用:

```java
@Rule
public BrowserWebDriverContainer chrome = new BrowserWebDriverContainer()
  .withCapabilities(new ChromeOptions());
@Test
public void whenNavigatedToPage_thenHeadingIsInThePage() {
    RemoteWebDriver driver = chrome.getWebDriver();
    driver.get("http://example.com");
    String heading = driver.findElement(By.xpath("/html/body/div/h1"))
      .getText();

    assertEquals("Example Domain", heading);
}
```

### 4.3 .复合坞站

如果测试需要更复杂的服务，我们可以在一个`docker-compose`文件中指定它们:

```java
simpleWebServer:
  image: alpine:3.2
  command: ["/bin/sh", "-c", "while true; do echo 'HTTP/1.1 200 OK\n\nHello World!' | nc -l -p 80; done"]
```

然后，我们使用`DockerComposeContainer`规则。该规则将启动并运行合成文件中定义的服务。

**我们使用`getServiceHost`和`getServicePost`方法建立到服务的连接地址:**

```java
@ClassRule
public static DockerComposeContainer compose = 
  new DockerComposeContainer(
    new File("src/test/resources/test-compose.yml"))
      .withExposedService("simpleWebServer_1", 80);

@Test
public void givenSimpleWebServerContainer_whenGetReuqest_thenReturnsResponse()
  throws Exception {

    String address = "http://" + compose.getServiceHost("simpleWebServer_1", 80) + ":" + compose.getServicePort("simpleWebServer_1", 80);
    String response = simpleGetRequest(address);

    assertEquals(response, "Hello World");
}
```

## 5.结论

我们看到了如何使用图书馆。它简化了开发和运行集成测试。

我们对给定 docker 图像的容器使用了`GenericContainer `规则。然后，我们看了看`PostgreSQLContainer, BrowserWebDriverContainer` 和`DockerComposeContainer` 规则。它们为特定的用例提供了更多的功能。

最后，这里的代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221017045625/https://github.com/eugenp/tutorials/tree/master/testing-modules/test-containers)