# Java 嵌入式 Jetty 服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jetty-embedded>

## 1.概观

在本文中，我们将关注 [`Jetty`](https://web.archive.org/web/20221219072217/https://www.eclipse.org/jetty/) 库。Jetty 提供了一个可以作为嵌入式容器运行的 web 服务器，并且可以很容易地与`javax.servlet` 库集成。

## 2。Maven 依赖关系

首先，我们将把 Maven 依赖项添加到 [jetty-server](https://web.archive.org/web/20221219072217/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.eclipse.jetty%22%20AND%20a%3A%22jetty-server%22) 和 [jetty-servlet](https://web.archive.org/web/20221219072217/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.eclipse.jetty%22%20AND%20a%3A%22jetty-servlet%22) 库中:

```java
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>9.4.3.v20170317</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-servlet</artifactId>
    <version>9.4.3.v20170317</version>
</dependency>
```

## 3。使用 Servlet 启动 Jetty 服务器

启动 Jetty 嵌入式容器很简单。我们需要实例化一个新的`Server` 对象，并将其设置为在给定的端口上启动:

```java
public class JettyServer {
    private Server server;

    public void start() throws Exception {
        server = new Server();
        ServerConnector connector = new ServerConnector(server);
        connector.setPort(8090);
        server.setConnectors(new Connector[] {connector});
}
```

假设我们想要创建一个端点，如果一切顺利，它将响应 HTTP 状态码 200 和一个简单的 JSON 有效负载。

我们将创建一个扩展`HttpServlet` 类的类来处理这样的请求；该类将是单线程的，并且在完成之前是阻塞的:

```java
public class BlockingServlet extends HttpServlet {

    protected void doGet(
      HttpServletRequest request, 
      HttpServletResponse response)
      throws ServletException, IOException {

        response.setContentType("application/json");
        response.setStatus(HttpServletResponse.SC_OK);
        response.getWriter().println("{ \"status\": \"ok\"}");
    }
}
```

接下来，我们需要使用`addServletWithMapping()`方法在`ServletHandler` 对象中注册`BlockingServlet` 类，并启动服务器:

```java
servletHandler.addServletWithMapping(BlockingServlet.class, "/status");
server.start();
```

如果我们希望测试我们的 Servlet 逻辑，我们需要使用之前创建的 `JettyServer` 类来启动我们的服务器，该类是测试设置中实际 Jetty 服务器实例的包装器:

```java
@Before
public void setup() throws Exception {
    jettyServer = new JettyServer();
    jettyServer.start();
}
```

一旦开始，我们将向`/status` 端点发送一个测试 HTTP 请求:

```java
String url = "http://localhost:8090/status";
HttpClient client = HttpClientBuilder.create().build();
HttpGet request = new HttpGet(url);

HttpResponse response = client.execute(request);

assertThat(response.getStatusLine().getStatusCode()).isEqualTo(200);
```

## 4。非阻塞 servlet

Jetty 对异步请求处理有很好的支持。

假设我们有一个巨大的 I/O 密集型资源，需要很长时间来加载，并在相当长的时间内阻塞执行线程。如果该线程可以被释放出来同时处理其他请求，而不是等待一些 I/O 资源，那就更好了。

为了用 Jetty 提供这样的逻辑，我们可以通过调用`HttpServletRequest.` 上的`startAsync()` 方法来创建一个使用`[AsyncContext](https://web.archive.org/web/20221219072217/https://docs.oracle.com/javaee/6/api/javax/servlet/AsyncContext.html)` 类的 servlet。这段代码不会阻塞正在执行的线程，但会在单独的线程中执行 I/O 操作，并在准备就绪时使用`AsyncContext.complete()` 方法返回结果:

```java
public class AsyncServlet extends HttpServlet {
    private static String HEAVY_RESOURCE 
      = "This is some heavy resource that will be served in an async way";

    protected void doGet(
      HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

        ByteBuffer content = ByteBuffer.wrap(
          HEAVY_RESOURCE.getBytes(StandardCharsets.UTF_8));

        AsyncContext async = request.startAsync();
        ServletOutputStream out = response.getOutputStream();
        out.setWriteListener(new WriteListener() {
            @Override
            public void onWritePossible() throws IOException {
                while (out.isReady()) {
                    if (!content.hasRemaining()) {
                        response.setStatus(200);
                        async.complete();
                        return;
                    }
                    out.write(content.get());
                }
            }

            @Override
            public void onError(Throwable t) {
                getServletContext().log("Async Error", t);
                async.complete();
            }
        });
    }
}
```

我们将`ByteBuffer`写入`OutputStream`，一旦整个缓冲区都被写入，我们就发出信号，通过调用`complete()` 方法将结果返回给客户端。

接下来，我们需要添加`AsyncServlet` 作为 Jetty servlet 映射:

```java
servletHandler.addServletWithMapping(
  AsyncServlet.class, "/heavy/async");
```

我们现在可以向`/heavy/async` 端点发送一个请求——该请求将由 Jetty 以异步方式处理:

```java
String url = "http://localhost:8090/heavy/async";
HttpClient client = HttpClientBuilder.create().build();
HttpGet request = new HttpGet(url);
HttpResponse response = client.execute(request);

assertThat(response.getStatusLine().getStatusCode())
  .isEqualTo(200);
String responseContent = IOUtils.toString(r
  esponse.getEntity().getContent(), StandardCharsets.UTF_8);
assertThat(responseContent).isEqualTo(
  "This is some heavy resource that will be served in an async way");
```

当我们的应用程序以异步方式处理请求时，我们应该显式地配置线程池。在下一节中，我们将配置`Jetty` 来使用自定义线程池。

## 5。码头配置

当我们在生产环境中运行 web 应用程序时，我们可能希望调整 Jetty 服务器处理请求的方式。这是通过定义线程池并将其应用于我们的 Jetty 服务器来实现的。

为此，我们可以设置三个配置设置:

*   `maxThreads`–指定 Jetty 可以在池中创建和使用的最大线程数
*   `minThreads –`设置 Jetty 将使用的池中线程的初始数量
*   `idleTimeout` –该值以毫秒为单位，定义线程在被停止并从线程池中删除之前可以空闲多长时间。池中剩余线程的数量永远不会低于`minThreads`设置

有了这些，我们可以通过将配置的线程池传递给`Server` 构造函数，以编程方式配置嵌入式`Jetty`服务器:

```java
int maxThreads = 100;
int minThreads = 10;
int idleTimeout = 120;

QueuedThreadPool threadPool = new QueuedThreadPool(maxThreads, minThreads, idleTimeout);

server = new Server(threadPool);
```

然后，当我们启动服务器时，它将使用特定线程池中的线程。

## 6。结论

在这个快速教程中，我们看到了如何将嵌入式服务器与 Jetty 集成，并测试了我们的 web 应用程序。

和往常一样，这段代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221219072217/https://github.com/eugenp/tutorials/tree/master/libraries-server)