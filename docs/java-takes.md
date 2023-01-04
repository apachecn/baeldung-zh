# 汇整简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-takes>

## 1.概观

Java 生态系统中有许多 web 框架，如 [Spring](/web/20221205131306/https://www.baeldung.com/spring-tutorial) 、 [Play](/web/20221205131306/https://www.baeldung.com/java-intro-to-the-play-framework) 和 [Grails](/web/20221205131306/https://www.baeldung.com/grails-gorm-tutorial) 。然而，它们都不能声称是完全不可变的和面向对象的。

在本教程中，我们将探索 Takes 框架，并使用路由、请求/响应处理和单元测试等常见功能创建一个简单的 web 应用程序。

## 2.接受

**Takes 是一个不可变的 Java 8 web 框架，既不使用`null`也不使用`public static`方法。**

此外，**框架不支持可变类、造型或反射**。因此，它是一个真正面向对象的框架。

汇整不需要用于设置的配置文件。除此之外，它还提供了 JSON/XML 响应和模板等内置特性。

## 3.设置

首先，我们将最新的`[takes](https://web.archive.org/web/20221205131306/https://search.maven.org/search?q=g:org.takes%20a:takes)` Maven 依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>org.takes</groupId>
    <artifactId>takes</artifactId>
    <version>1.19</version>
</dependency>
```

然后，让我们创建实现 [`Take`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/Take.html) 接口的`TakesHelloWorld`类:

```java
public class TakesHelloWorld implements Take {
    @Override
    public Response act(Request req) {
        return new RsText("Hello, world!");
    }
}
```

`Take`接口提供了框架的基本特性。每个`Take `作为一个**请求处理器，通过`act`方法**返回响应。

这里，我们使用了 [`RsText`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsText.html) 类来呈现纯文本`Hello, world!`作为响应，当请求被发送到`TakesHelloWorld`时。

接下来，我们将创建`TakesApp`类来启动 web 应用程序:

```java
public class TakesApp {
    public static void main(String... args) {
        new FtBasic(new TakesHelloWorld()).start(Exit.NEVER);
    }
}
```

这里，我们使用了提供了 [`Front`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/http/Front.html) 接口的基本实现的`[FtBasic](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/http/FtBasic.html)`类来启动 web 服务器，并将请求转发给`TakesHelloWorld` take。

Takes 通过使用 [`ServerSocket`](/web/20221205131306/https://www.baeldung.com/a-guide-to-java-sockets) 类实现自己的无状态 web 服务器。默认情况下，它在端口 80 上启动服务器。但是，我们可以在代码中定义端口:

```java
new FtBasic(new TakesHelloWorld(), 6060).start(Exit.NEVER);
```

或者，我们可以使用命令行参数`–port`**传递端口号。**

然后，让我们使用 Maven 命令编译这些类:

```java
mvn clean package
```

现在，我们准备在 IDE 中将`TakesApp`类作为一个简单的 Java 应用程序运行。

## 4.奔跑

我们也可以将我们的`TakesApp` 类作为一个单独的 web 服务器应用程序运行。

### 4.1.Java 命令行

首先，让我们编译我们的类:

```java
javac -cp "takes.jar:." com.baeldung.takes.*
```

然后，我们可以使用 Java 命令行运行应用程序:

```java
java -cp "takes.jar:." com.baeldung.takes.TakesApp --port=6060
```

### 4.2.专家

或者，我们可以使用**[`exec-maven-plugin`](https://web.archive.org/web/20221205131306/https://search.maven.org/search?q=g:org.codehaus.mojo%20a:exec-maven-plugin)插件通过 Maven** 来运行它:

```java
<profiles>
    <profile>
        <id>reload</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>exec-maven-plugin</artifactId>
                    <version>3.0.0</version>
                    <executions>
                        <execution>
                            <id>start-server</id>
                            <phase>pre-integration-test</phase>
                            <goals>
                                <goal>java</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <mainClass>com.baeldung.takes.TakesApp</mainClass>
                        <cleanupDaemonThreads>false</cleanupDaemonThreads>
                        <arguments>
                            <argument>--port=${port}</argument>
                        </arguments>
                   </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

现在，我们可以使用 Maven 命令运行我们的应用程序:

```java
mvn clean integration-test -Preload -Dport=6060
```

## 5.按指定路线发送

该框架提供了 [`TkFork`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fork/TkFork.html) 类来将请求路由到不同的 takes。

例如，让我们在应用程序中添加几条路线:

```java
public static void main(String... args) {
    new FtBasic(
        new TkFork(
            new FkRegex("/", new TakesHelloWorld()),
            new FkRegex("/contact", new TakesContact())
        ), 6060
    ).start(Exit.NEVER);
}
```

这里，我们使用了 [`FkRegex`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fork/FkRegex.html) 类来匹配请求路径。

## 6.请求处理

框架在`org.takes.rq`包中提供了一些[装饰类](/web/20221205131306/https://www.baeldung.com/java-decorator-pattern)来处理 HTTP 请求。

例如，我们可以使用 [`RqMethod`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqMethod.html) 接口来提取 HTTP 方法:

```java
public class TakesHelloWorld implements Take { 
    @Override
    public Response act(Request req) throws IOException {
        String requestMethod = new RqMethod.Base(req).method(); 
        return new RsText("Hello, world!"); 
    }
}
```

类似地， [`RqHeaders`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqHeaders.html) 接口可用于获取请求头:

```java
Iterable<String> requestHeaders = new RqHeaders.Base(req).head();
```

我们可以使用 [`RqPrint`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqPrint.html) 类来获得请求的体:

```java
String body = new RqPrint(req).printBody();
```

同样，我们可以使用 [`RqFormSmart`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/form/RqFormSmart.html) 类来访问表单参数:

```java
String username = new RqFormSmart(req).single("username");
```

## 7.响应处理

Takes 还提供了许多有用的装饰器来处理`org.takes.rs`包中的 HTTP 响应。

响应装饰器实现了 [`Response`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/Response.html) 接口的`head`和`body`方法。

例如， [`RsWithStatus`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsWithStatus.html) 类使用状态代码呈现响应:

```java
Response resp = new RsWithStatus(200);
```

可以使用`head`方法验证响应的输出:

```java
assertEquals("[HTTP/1.1 200 OK], ", resp.head().toString());
```

类似地， [`RsWithType`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsWithType.html) 类使用内容类型呈现响应:

```java
Response resp = new RsWithType(new RsEmpty(), "text/html");
```

这里， [`RsEmpty`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsEmpty.html) 类呈现空响应。

同样，我们可以使用 [`RsWithBody`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsWithBody.html) 类来呈现带有正文的响应。

因此，让我们创建`TakesContact`类并使用讨论过的装饰器来呈现响应:

```java
public class TakesContact implements Take {
    @Override
    public Response act(Request req) throws IOException {
        return new RsWithStatus(
          new RsWithType(
            new RsWithBody("Contact us at https://www.baeldung.com"), 
            "text/html"), 200);
    }
}
```

同样，我们可以使用 [`RsJson`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsJson.html) 类来呈现 JSON 响应:

```java
@Override 
public Response act(Request req) { 
    JsonStructure json = Json.createObjectBuilder() 
      .add("id", rs.getInt("id")) 
      .add("user", rs.getString("user")) 
      .build(); 
    return new RsJson(json); 
}
```

## 8.异常处理

**框架包含了 [`Fallback`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/Fallback.html) 接口来处理异常情况。**它还提供了一些处理回退场景的实现。

例如，让我们使用 [`TkFallback`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/TkFallback.html) 类来处理 HTTP 404 并向用户显示一条消息:

```java
public static void main(String... args) throws IOException, SQLException {
    new FtBasic(
        new TkFallback(
          new TkFork(
            new FkRegex("/", new TakesHelloWorld()),
            // ...
            ),
            new FbStatus(404, new RsText("Page Not Found"))), 6060
     ).start(Exit.NEVER);
}
```

这里，我们使用了 [`FbStatus`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/FbStatus.html) 类来处理已定义状态代码的回退。

类似地，我们可以使用 [`FbChain`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/FbChain.html) 类来定义一个组合后援:

```java
new TkFallback(
    new TkFork(
      // ...
      ),
    new FbChain(
      new FbStatus(404, new RsText("Page Not Found")),
      new FbStatus(405, new RsText("Method Not Allowed"))
      )
    ), 6060
).start(Exit.NEVER);
```

此外，我们可以实现`Fallback`接口来处理异常:

```java
new FbChain(
    new FbStatus(404, new RsText("Page Not Found")),
    new FbStatus(405, new RsText("Method Not Allowed")),
    new Fallback() {
        @Override
        public Opt<Response> route(RqFallback req) {
          return new Opt.Single<Response>(new RsText(req.throwable().getMessage()));
        }
    }
)
```

## 9.模板

让我们将 [Apache Velocity](/web/20221205131306/https://www.baeldung.com/apache-velocity) 与我们的 Takes web 应用程序集成，以提供一些模板功能。

首先，我们将添加 [`velocity-engine-core`](https://web.archive.org/web/20221205131306/https://search.maven.org/search?q=g:org.apache.velocity%20a:velocity-engine-core) Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.2</version>
</dependency>
```

然后，我们将使用`[RsVelocity](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsVelocity.html)`类来定义模板字符串和`act`方法中的绑定参数:

```java
public class TakesIndex implements Take {
    @Override
    public Response act(Request req) throws IOException {
        return new RsHtml(
            new RsVelocity("${username}", new RsVelocity.Pair("username", "Baeldung")));
        );
    }
}
```

这里，我们使用了 [`RsHtml`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsHtml.html) 类来呈现 HTML 响应。

此外，我们可以使用带有`RsVelocity`类的速度模板:

```java
new RsVelocity(this.getClass().getResource("/templates/index.vm"), 
    new RsVelocity.Pair("username", username))
);
```

## 10.单元测试

框架通过提供创建假请求的 [`RqFake`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqFake.html) 类来支持任何`Take`的单元测试:

例如，让我们使用 [JUnit](/web/20221205131306/https://www.baeldung.com/junit-5) 为我们的`TakesContact`类编写一个单元测试:

```java
String resp = new RsPrint(new TakesContact().act(new RqFake())).printBody();
assertEquals("Contact us at https://www.baeldung.com", resp); 
```

## 11.集成测试

我们可以使用 JUnit 和任何 HTTP 客户端测试整个应用程序。

框架提供了在随机端口上启动服务器的 [`FtRemote`](https://web.archive.org/web/20221205131306/https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/http/FtRemote.html) 类，并提供了对`Take`执行的远程控制。

例如，让我们编写一个集成测试并验证`TakesContact`类的响应:

```java
new FtRemote(new TakesContact()).exec(
    new FtRemote.Script() {
        @Override
        public void exec(URI home) throws IOException {
            HttpClient client = HttpClientBuilder.create().build();    
            HttpResponse response = client.execute(new HttpGet(home));
            int statusCode = response.getStatusLine().getStatusCode();
            HttpEntity entity = response.getEntity();
            String result = EntityUtils.toString(entity);

            assertEquals(200, statusCode);
            assertEquals("Contact us at https://www.baeldung.com", result);
        }
    });
```

这里，我们使用了 [Apache HttpClient](/web/20221205131306/https://www.baeldung.com/httpclient-status-code) 向服务器发出请求并验证响应。

## 12.结论

在本教程中，我们通过创建一个简单的 web 应用程序探索了 Takes 框架。

首先，我们看到了一种在 Maven 项目中建立框架并运行应用程序的快速方法。

然后，我们研究了一些常见的特性，比如路由、请求/响应处理和单元测试。

最后，我们探讨了框架对单元测试和集成测试的支持。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20221205131306/https://github.com/eugenp/tutorials/tree/master/libraries-3)