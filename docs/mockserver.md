# MockServer 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockserver>

## 1。概述

[MockServer](https://web.archive.org/web/20220524125229/http://www.mock-server.com/) 是一个模仿/存根外部 HTTP APIs 的工具。

## 2。Maven 依赖关系

为了在我们的应用程序中使用`MockServer`,我们需要添加两个依赖项:

```java
<dependency>
    <groupId>org.mock-server</groupId>
    <artifactId>mockserver-netty</artifactId>
    <version>3.10.8</version>
</dependency>
<dependency>
    <groupId>org.mock-server</groupId>
    <artifactId>mockserver-client-java</artifactId>
    <version>3.10.8</version>
</dependency>
```

依赖关系的最新版本可以作为 [mockserver-netty](https://web.archive.org/web/20220524125229/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.mock-server%22%20%20a%3A%22mockserver-netty%22) 和 [mockserver-client 获得。](https://web.archive.org/web/20220524125229/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.mock-server%22%20%20a%3A%22mockserver-client-java%22)

## 3。`MockServer`功能

简而言之，该工具可以:

*   生成并返回固定响应
*   将请求转发到另一台服务器
*   执行回调
*   核实请求

## 4。如何运行`MockServer`

我们可以用几种不同的方式启动服务器，让我们来探索其中的一些方法。

### 4.1。通过 Maven 插件启动

这将在`process-test-class`阶段启动服务器，并在`verify`阶段停止:

```java
<plugin>
    <groupId>org.mock-server</groupId>
    <artifactId>mockserver-maven-plugin</artifactId>
    <version>3.10.8</version>
    <configuration>
        <serverPort>1080</serverPort>
        <proxyPort>1090</proxyPort>
        <logLevel>DEBUG</logLevel>
        <initializationClass>org.mockserver.maven.ExampleInitializationClass</initializationClass>
    </configuration>
    <executions>
        <execution>
            <id>process-test-classes</id>
            <phase>process-test-classes</phase>
            <goals>
                <goal>start</goal>
            </goals>
        </execution>
        <execution>
            <id>verify</id>
            <phase>verify</phase>
            <goals>
                <goal>stop</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 4.2。通过 Java API 启动

我们可以使用`startClientAndServer()` Java API 来启动服务器。通常，我们会在运行所有测试之前启动服务器:

```java
public class TestMockServer {

    private ClientAndServer mockServer;

    @BeforeClass
    public void startServer() {
        mockServer = startClientAndServer(1080);
    }

    @AfterClass 
    public void stopServer() { 
        mockServer.stop();
    }

    // ...
}
```

## 5。模拟客户端

`MockServerClient` API 用于提供连接到`MockServer.`的能力，它模拟来自服务器的请求和相应的响应。

它支持多种操作:

### 5.1。用模拟回答创造期望

期望是一种机制，我们通过它来模拟来自客户机的请求和来自模拟服务器的结果响应。

为了创建一个期望，我们需要定义一个请求匹配器和一个应该返回的响应。

可以使用以下方式匹配请求:

*   路径–URL 路径
*   查询字符串–URL 参数
*   标题–请求标题
*   cookie–客户端 cookie
*   body——用 XPATH、JSON、JSON 模式、正则表达式、完全匹配的纯文本或主体参数发布请求主体

以上所有参数都可以使用纯文本或正则表达式来指定。

响应动作将包含:

*   状态代码–有效的 HTTP 状态代码，例如 200、400 等。
*   body–它是包含任何内容的字节序列
*   标题–带有名称和一个或多个值的响应标题
*   cookie–带有名称和一个或多个值的响应 cookie

让我们看看如何**创造期望**:

```java
public class TestMockServer {
    private void createExpectationForInvalidAuth() {
        new MockServerClient("127.0.0.1", 1080)
          .when(
            request()
              .withMethod("POST")
              .withPath("/validate")
              .withHeader("\"Content-type\", \"application/json\"")
              .withBody(exact("{username: 'foo', password: 'bar'}")),
              exactly(1))
                .respond(
                  response()
                    .withStatusCode(401)
                    .withHeaders(
                      new Header("Content-Type", "application/json; charset=utf-8"),
                      new Header("Cache-Control", "public, max-age=86400"))
                    .withBody("{ message: 'incorrect username and password combination' }")
                    .withDelay(TimeUnit.SECONDS,1)
                );
    }
    // ...
}
```

这里，我们向服务器发送一个`POST`请求。并且我们已经指定了我们需要使用`exactly(1)` 调用发出这个请求的次数。

收到这个请求后，我们用状态代码、标题和响应体等字段模拟了一个响应。

### 5.2。转发请求

可以设置期望值来转发请求。一些参数可以描述前进动作:

*   **主机**–要转发到的主机，例如 www.baeldung.com
*   **端口**–请求转发到的端口，默认端口为 80
*   **方案**–使用 HTTP 或 HTTPS 等协议

让我们看一个转发请求的例子:

```java
private void createExpectationForForward(){
    new MockServerClient("127.0.0.1", 1080)
      .when(
        request()
          .withMethod("GET")
          .withPath("/index.html"),
          exactly(1))
        .forward(
          forward()
            .withHost("www.mock-server.com")
            .withPort(80)
            .withScheme(HttpForward.Scheme.HTTP)
           );
}
```

在这种情况下，我们模拟了一个请求，该请求将准确地命中模拟服务器，然后转发到另一个服务器。外部的`forward()` 方法指定转发动作，内部的`forward()` 方法调用帮助构造 URL 并转发请求。

### 5.3。执行回调

**可以将服务器设置为在接收到特定请求时执行回调。**回调动作可以定义实现`org.mockserver.mock.action.ExpectationCallback` 接口的回调类。它应该有默认的构造函数，并且应该在类路径中。

让我们看一个关于回调的期望的例子:

```java
private void createExpectationForCallBack() {
    mockServer
      .when(
        request().withPath("/callback"))
        .callback(
          callback()
            .withCallbackClass("com.baeldung.mock.server.TestExpectationCallback")
        );
}
```

这里，外部的`callback()` 指定回调动作，内部的`callback()` 方法指定回调方法类的实例。

在这种情况下，当 MockServer 接收到带有`/callback,` 的请求时，将执行在指定类中实现的回调句柄方法:

```java
public class TestExpectationCallback implements ExpectationCallback {

    public HttpResponse handle(HttpRequest httpRequest) {
        if (httpRequest.getPath().getValue().endsWith("/callback")) {
            return httpResponse;
        } else {
            return notFoundResponse();
        }
    }

    public static HttpResponse httpResponse = response()
      .withStatusCode(200);
}
```

### 5.4。验证请求

能够检查被测系统是否发送了请求:

```java
private void verifyPostRequest() {
    new MockServerClient("localhost", 1080).verify(
      request()
        .withMethod("POST")
        .withPath("/validate")
        .withBody(exact("{username: 'foo', password: 'bar'}")),
        VerificationTimes.exactly(1)
    );
}
```

这里，`org.mockserver.verify.VerificationTimes`类用于指定模拟服务器应该匹配请求的次数。

## 6。结论

在这篇简短的文章中，我们探索了 MockServer 的不同功能。我们还探索了所提供的不同 API，以及如何用它来测试复杂的系统。

和往常一样，本文的完整代码可以在 GitHub 上找到。