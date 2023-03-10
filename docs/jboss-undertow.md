# JBoss Undertow 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jboss-undertow>

## 1。概述

**[under flow](https://web.archive.org/web/20220630010348/http://undertow.io/)是来自`JBoss`的一个极其轻量级和高性能的 web 服务器。**支持带`NIO`的阻塞和非阻塞 API。

由于是用 Java 写的，所以可以在任何基于 JVM 的应用中以嵌入式方式使用，甚至 JBoss 的`[WilfFly](https://web.archive.org/web/20220630010348/http://wildfly.org/)`服务器内部也用`Undertow`来提高服务器的性能。

在本教程中，我们将展示逆流的特点以及如何使用它。

## 2。为什么要逆流？

*   轻量级:`Undertow`在 1MB 以下是非常轻量级的。在嵌入式模式下，它在运行时只使用 4MB 的堆空间
*   Servlet 3.1:完全支持`Servlet 3.1`
*   Web Socket:支持 Web Socket 功能(包括`JSR-356`)
*   持久连接:默认情况下，`Undertow`通过添加`keep-alive`响应头来包含 HTTP 持久连接。它帮助支持持久连接的客户端通过重用连接细节来优化性能

## 3。使用回流

让我们通过创建一个简单的 web 服务器来开始使用`Undertow`。

### 3.1。Maven 依赖关系

要使用`Undertow`，我们需要将以下依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>io.undertow</groupId>
    <artifactId>undertow-servlet</artifactId>
    <version>1.4.18.Final</version>
</dependency>
```

为了构建可运行的 jar，我们还需要添加 [maven-shade-plugin](https://web.archive.org/web/20220630010348/https://maven.apache.org/plugins/maven-shade-plugin/) 。这就是为什么我们还需要添加以下配置:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

最新版本的`Undertow`可以在[中央 Maven 资源库](https://web.archive.org/web/20220630010348/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.undertow%22)中获得。

### 3.2。简单服务器

通过下面的代码片段，我们可以使用 Undertow 的 [`Builder`](https://web.archive.org/web/20220630010348/http://undertow.io/javadoc/1.4.x/io/undertow/Undertow.Builder.html) API 创建一个简单的 web 服务器:

```java
public class SimpleServer {
    public static void main(String[] args) {
        Undertow server = Undertow.builder().addHttpListener(8080, 
          "localhost").setHandler(exchange -> {
            exchange.getResponseHeaders()
            .put(Headers.CONTENT_TYPE, "text/plain");
          exchange.getResponseSender().send("Hello Baeldung");
        }).build();
        server.start();
    }
}
```

这里，我们使用了`Builder` API 将`8080`端口绑定到这个服务器。另外，请注意，我们使用了 lambda 表达式来使用处理程序。

我们也可以使用下面的代码片段来做同样的事情，而不使用 lambda 表达式:

```java
Undertow server = Undertow.builder().addHttpListener(8080, "localhost")
  .setHandler(new HttpHandler() {
      @Override
      public void handleRequest(HttpServerExchange exchange) 
        throws Exception {
          exchange.getResponseHeaders().put(
            Headers.CONTENT_TYPE, "text/plain");
          exchange.getResponseSender().send("Hello Baeldung");
      }
  }).build();
```

这里需要注意的是 [**`HttpHandler`**](https://web.archive.org/web/20220630010348/http://undertow.io/javadoc/1.4.x/io/undertow/server/HttpHandler.html) API 的用法。根据我们的需求定制一个`Undertow`应用程序，这是最重要的插件。

在这种情况下，我们添加了一个定制的处理程序，它将为每个请求添加`Content-Type: text/plain`响应头。

类似地，如果我们想在每个响应中返回一些默认文本，我们可以使用下面的代码片段:

```java
exchange.getResponseSender()
  .send("Hello Baeldung");
```

### 3.3。安全访问

在大多数情况下，我们不允许所有用户访问我们的服务器。通常，具有有效凭据的用户可以获得访问权限。我们可以用`Undertow`实现相同的机制。

为了实现它，我们需要创建一个身份管理器来检查每个请求的真实性。

为此，我们可以使用 Undertow 的`[**IdentityManager**](https://web.archive.org/web/20220630010348/http://undertow.io/javadoc/1.4.x/io/undertow/security/idm/IdentityManager.html)`:

```java
public class CustomIdentityManager implements IdentityManager {
    private Map<String, char[]> users;

    // standard constructors

    @Override
    public Account verify(Account account) {
        return account;
    }

    @Override
    public Account verify(Credential credential) {
        return null;
    }

    @Override
    public Account verify(String id, Credential credential) {
        Account account = getAccount(id);
        if (account != null && verifyCredential(account, credential)) {
            return account;
        }
        return null;
    }
}
```

创建 identity manager 后，我们需要创建一个保存用户凭据的领域:

```java
private static HttpHandler addSecurity(
  HttpHandler toWrap, 
  IdentityManager identityManager) {

    HttpHandler handler = toWrap;
    handler = new AuthenticationCallHandler(handler);
    handler = new AuthenticationConstraintHandler(handler);
    List<AuthenticationMechanism> mechanisms = Collections.singletonList(
      new BasicAuthenticationMechanism("Baeldung_Realm"));
    handler = new AuthenticationMechanismsHandler(handler, mechanisms);
    handler = new SecurityInitialHandler(
      AuthenticationMode.PRO_ACTIVE, identityManager, handler);
    return handler;
}
```

这里，我们使用了 [`**AuthenticationMode**`](https://web.archive.org/web/20220630010348/http://undertow.io/javadoc/1.4.x/io/undertow/security/api/AuthenticationMode.html) 作为`PRO_ACTIVE`，这意味着每个到达该服务器的请求都将被传递到定义的认证机制，以急切地执行认证。

**如果我们将`AuthenticationMode`定义为`CONSTRAINT_DRIVEN`，那么只有那些请求将通过已定义的认证机制，在该机制中，授权认证的约束被触发。**

现在，我们只需要在服务器启动之前将该领域和 identity manager 映射到服务器:

```java
public static void main(String[] args) {
    Map<String, char[]> users = new HashMap<>(2);
    users.put("root", "password".toCharArray());
    users.put("admin", "password".toCharArray());

    IdentityManager idm = new CustomIdentityManager(users);

    Undertow server = Undertow.builder().addHttpListener(8080, "localhost")
      .setHandler(addSecurity(e -> setExchange(e), idm)).build();

    server.start();
}

private static void setExchange(HttpServerExchange exchange) {
    SecurityContext context = exchange.getSecurityContext();
    exchange.getResponseSender().send("Hello " + 
      context.getAuthenticatedAccount().getPrincipal().getName(),
      IoCallback.END_EXCHANGE);
}
```

这里，我们已经创建了两个带有凭证的用户实例。一旦服务器启动，要访问它，我们需要使用这两个凭证中的任何一个。

### 3.4。网络套接字

用`UnderTow's [**WebSocketHttpExchange**](https://web.archive.org/web/20220630010348/http://undertow.io/javadoc/1.4.x/io/undertow/websockets/spi/WebSocketHttpExchange.html)` API 创建 web 套接字交换通道很简单。

例如，我们可以用下面的代码片段在路径`baeldungApp`上打开一个套接字通信通道:

```java
public static void main(String[] args) {
    Undertow server = Undertow.builder().addHttpListener(8080, "localhost")
      .setHandler(path().addPrefixPath("/baeldungApp", websocket(
        (exchange, channel) -> {
          channel.getReceiveSetter().set(getListener());
          channel.resumeReceives();
      })).addPrefixPath("/", resource(new ClassPathResourceManager(
        SocketServer.class.getClassLoader(),
        SocketServer.class.getPackage())).addWelcomeFiles("index.html")))
        .build();

    server.start();
}

private static AbstractReceiveListener getListener() {
    return new AbstractReceiveListener() {
        @Override
        protected void onFullTextMessage(WebSocketChannel channel, 
          BufferedTextMessage message) {
            String messageData = message.getData();
            for (WebSocketChannel session : channel.getPeerConnections()) {
                WebSockets.sendText(messageData, session, null);
            }
        }
    };
}
```

我们可以创建一个名为`index.html`的 HTML 页面，并使用 JavaScript 的`[WebSocket](https://web.archive.org/web/20220630010348/https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)` API 连接到这个通道。

### 3.5。文件服务器

使用`Undertow`，我们还可以创建一个文件服务器，它可以显示目录内容并直接从目录中提供文件:

```java
public static void main( String[] args ) {
    Undertow server = Undertow.builder().addHttpListener(8080, "localhost")
        .setHandler(resource(new PathResourceManager(
          Paths.get(System.getProperty("user.home")), 100 ))
        .setDirectoryListingEnabled( true ))
        .build();
    server.start();
}
```

我们不需要创建任何 UI 内容来显示目录内容。现成的`Undertow` 为此显示功能提供了一个页面。

## 4。Spring Boot 插件

除了[`Tomcat`](https://web.archive.org/web/20220630010348/https://tomcat.apache.org/)`[Jetty](https://web.archive.org/web/20220630010348/https://www.eclipse.org/jetty/),``Spring Boot`支持`UnderTow`作为嵌入式 servlet 容器。要使用`Undertow`，我们需要在`pom.xml:`中添加以下依赖项

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
    <version>1.5.6.RELEASE</version>
</dependency>
```

最新版本的`Spring Boot Undertow plugin`可以在[中央 Maven 资源库](https://web.archive.org/web/20220630010348/https://search.maven.org/classic/#search|gav|1|g%3A"org.springframework.boot" AND a%3A"spring-boot-starter-undertow")中获得。

## 5。结论

在本文中，我们学习了`Undertow`以及如何用它创建不同类型的服务器。

像往常一样，完整的源代码可以在 GitHub 上获得[。](https://web.archive.org/web/20220630010348/https://github.com/eugenp/tutorials/tree/master/undertow)