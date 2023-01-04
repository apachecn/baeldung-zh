# 为 Apache HttpClient 启用日志记录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-httpclient-enable-logging>

## 1.概观

在本教程中，我们将展示如何在 [Apache 的 http client](/web/20220813062922/https://www.baeldung.com/httpclient-guide)T3 中**启用登录。此外，我们将解释如何在库中实现日志记录。之后，我们将展示如何启用不同级别的日志记录。**

## 2.日志实现

HttpClient 库提供了高效、最新、功能丰富的 HTTP 协议实现客户端站点。

**的确作为一个库，HttpClient 并不强制日志实现**。出于这个目的，4.5 版本提供了带有 [Commons Logging](https://web.archive.org/web/20220813062922/https://commons.apache.org/proper/commons-logging/) 的日志。类似地，最新版本 5.1 使用了由 [SLF4J](https://web.archive.org/web/20220813062922/https://baeldung-cn.com/slf4j-with-log4j2-logback) 提供的日志门面。两个版本都使用层次结构模式来匹配记录器和它们的配置。

得益于此，可以为单个类或与相同功能相关的所有类设置日志程序。

## 3.日志类型

让我们看看由库定义的日志级别。我们可以区分 3 种类型的日志:

*   上下文日志记录–记录有关 HttpClient 的所有内部操作的信息。它还包含电线和标题日志。
*   有线日志记录–仅记录与服务器之间传输的数据
*   标头记录–仅记录 HTTP 标头

在 4.5 版本中，对应的**包是`org.apache.http.impl.client` 和 `org.apache.http.wire, org.apache.http.headers.` T3**

相应地，在 5.1 版本中，有 **`org.apache.hc.client5.http`、`org.apache.hc.client5.http.wire` 和`org.apache.hc.client5.http.headers.`**

## 4.Log4j 配置

让我们来看看如何登录到这两个版本。我们的目标是在两个版本中实现相同的灵活性。在 4.1 版本中，我们会将日志重定向到 SLF4j。因此，可以使用不同的日志框架。

### 4.1.4.5 版配置

让我们添加 [httpclient](https://web.archive.org/web/20220813062922/https://search.maven.org/artifact/org.apache.httpcomponents/httpclient) 依赖项:

```java
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.8</version>
    <exclusions>
        <exclusion>
            <artifactId>commons-logging</artifactId>
            <groupId>commons-logging</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

我们将使用 [jul-to-slf4j](https://web.archive.org/web/20220813062922/https://search.maven.org/artifact/org.slf4j/jul-to-slf4j) 将日志重定向到 slf4j。因此我们排除了`commons-logging`。接下来，让我们为朱莉和 SLF4J 之间的桥添加一个依赖项:

```java
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
    <version>1.7.26</version>
</dependency>
```

因为 SLF4J 只是一个门面，我们需要一个绑定。在我们的例子中，我们将使用[回退](https://web.archive.org/web/20220813062922/https://search.maven.org/artifact/ch.qos.logback/logback-classic):

```java
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

现在让我们创建`ApacheHttpClientUnitTest` 类:

```java
public class ApacheHttpClientUnitTest {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    public static final String DUMMY_URL = "https://postman-echo.com/get";

    @Test
    public void whenUseApacheHttpClient_thenCorrect() throws IOException {
        HttpGet request = new HttpGet(DUMMY_URL);

        try (CloseableHttpClient client = HttpClients.createDefault(); CloseableHttpResponse response = client.execute(request)) {
            HttpEntity entity = response.getEntity();
            logger.debug("Response -> {}",  EntityUtils.toString(entity));
        }
    }
}
```

该测试获取一个虚拟网页，并将内容打印到日志中。

现在让我们用我们的`logback.xml`文件定义一个日志配置:

```java
<configuration debug="false">
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%date [%level] %logger - %msg %n</pattern>
        </encoder>
    </appender>

    <logger name="com.baeldung.httpclient.readresponsebodystring" level="debug"/>
    <logger name="org.apache.http" level="debug"/>

    <root level="WARN">
        <appender-ref ref="stdout"/>
    </root>
</configuration>
```

运行我们的测试后，可以在控制台中找到所有 HttpClient 的日志:

```java
...
2021-06-19 22:24:45,378 [DEBUG] org.apache.http.impl.execchain.MainClientExec - Executing request GET /get HTTP/1.1 
2021-06-19 22:24:45,378 [DEBUG] org.apache.http.impl.execchain.MainClientExec - Target auth state: UNCHALLENGED 
2021-06-19 22:24:45,379 [DEBUG] org.apache.http.impl.execchain.MainClientExec - Proxy auth state: UNCHALLENGED 
2021-06-19 22:24:45,382 [DEBUG] org.apache.http.headers - http-outgoing-0 >> GET /get HTTP/1.1 
...
```

### 4.2.5.1 版配置

现在让我们来看看更高的版本。**它包含重新设计的日志记录。因此，它利用了 SLF4J，而不是公共日志。**因此，logger facade 的绑定是唯一附加的依赖项。因此，我们将使用第一个例子中的`logback-classic` 。

让我们添加 [httpclient5](https://web.archive.org/web/20220813062922/https://search.maven.org/artifact/org.apache.httpcomponents.client5/httpclient5) 依赖项:

```java
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.1</version>
</dependency>
```

让我们添加一个与上例相似的测试:

```java
public class ApacheHttpClient5UnitTest {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    public static final String DUMMY_URL = "https://postman-echo.com/get";

    @Test
    public void whenUseApacheHttpClient_thenCorrect() throws IOException, ParseException {
        HttpGet request = new HttpGet(DUMMY_URL);

        try (CloseableHttpClient client = HttpClients.createDefault(); CloseableHttpResponse response = client.execute(request)) {
            HttpEntity entity = response.getEntity();
            logger.debug("Response -> {}", EntityUtils.toString(entity));
        }
    }
}
```

接下来，我们需要向 `logback.xml`文件添加一个记录器:

```java
<configuration debug="false">
...
    <logger name="org.apache.hc.client5.http" level="debug"/>
...
</configuration>
```

让我们运行测试类`ApacheHttpClient5UnitTest` 并检查输出。它类似于旧版本:

```java
...
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.classic.InternalHttpClient - ep-0000000000 endpoint connected 
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.classic.MainClientExec - ex-0000000001 executing GET /get HTTP/1.1 
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.classic.InternalHttpClient - ep-0000000000 start execution ex-0000000001 
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.io.PoolingHttpClientConnectionManager - ep-0000000000 executing exchange ex-0000000001 over http-outgoing-0 
2021-06-19 22:27:16,960 [DEBUG] org.apache.hc.client5.http.headers - http-outgoing-0 >> GET /get HTTP/1.1 
...
```

## 5.结论

这篇关于如何为 Apache 的 HttpClient 配置日志记录的简短教程到此结束。首先，我们解释了日志是如何在库中实现的。其次，我们在两个版本中配置日志记录，并执行简单的测试用例来显示输出。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220813062922/https://github.com/eugenp/tutorials/tree/master/apache-httpclient-2)