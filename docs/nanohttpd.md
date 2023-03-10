# NanoHTTPD 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/nanohttpd>

## 1.介绍

NanoHTTPD 是一个用 Java 编写的开源轻量级 web 服务器。

在本教程中，我们将创建几个 REST APIs 来探索它的特性。

## 2.项目设置

让我们 将 [NanoHTTPD 核心依赖](https://web.archive.org/web/20220628131736/https://search.maven.org/search?q=a:nanohttpd%20AND%20g:org.nanohttpd)添加到我们的`pom.xml` :

```java
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd</artifactId>
    <version>2.3.1</version>
</dependency>
```

要创建一个简单的服务器，我们需要扩展`NanoHTTPD`并覆盖它的`serve`方法:

```java
public class App extends NanoHTTPD {
    public App() throws IOException {
        super(8080);
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    public static void main(String[] args ) throws IOException {
        new App();
    }

    @Override
    public Response serve(IHTTPSession session) {
        return newFixedLengthResponse("Hello world");
    }
}
```

我们将运行端口定义为`8080`，将服务器定义为守护进程(无读取超时)。

一旦我们启动应用程序，URL [`http://localhost:8080/`](https://web.archive.org/web/20220628131736/http://localhost:8080/) 将返回`Hello world`消息。我们使用`NanoHTTPD#newFixedLengthResponse`方法作为构建`NanoHTTPD.Response`对象的便捷方式。 

让我们用 [cURL](/web/20220628131736/https://www.baeldung.com/curl-rest) 来试试我们的项目:

```java
> curl 'http://localhost:8080/'
Hello world
```

## 3.REST API

在 HTTP 方法中，NanoHTTPD 允许 GET、POST、PUT、DELETE、HEAD、TRACE 和其他一些方法。

简单地说，我们可以通过 enum 方法找到受支持的 HTTP 动词。让我们看看这将如何发展。

### 3.1.HTTP GET

首先，我们来看看 GET。比方说，我们希望只有当应用程序收到 GET 请求时才返回内容。

与 [Java Servlet 容器](/web/20220628131736/https://www.baeldung.com/intro-to-servlets)不同，我们没有可用的`doGet `方法——相反，我们只是通过`getMethod`检查值:

```java
@Override
public Response serve(IHTTPSession session) {
    if (session.getMethod() == Method.GET) {
        String itemIdRequestParameter = session.getParameters().get("itemId").get(0);
        return newFixedLengthResponse("Requested itemId = " + itemIdRequestParameter);
    }
    return newFixedLengthResponse(Response.Status.NOT_FOUND, MIME_PLAINTEXT, 
        "The requested resource does not exist");
}
```

这很简单，对吧？让我们通过卷曲我们的新端点来运行一个快速测试，看看请求参数`itemId`是否被正确读取:

```java
> curl 'http://localhost:8080/?itemId=23Bk8'
Requested itemId = 23Bk8
```

### 3.2.HTTP 帖子

我们之前对 GET 作出了反应，并从 URL 中读取了一个参数。

为了介绍两种最流行的 HTTP 方法，现在我们应该处理一个 POST(从而读取请求体):

```java
@Override
public Response serve(IHTTPSession session) {
    if (session.getMethod() == Method.POST) {
        try {
            session.parseBody(new HashMap<>());
            String requestBody = session.getQueryParameterString();
            return newFixedLengthResponse("Request body = " + requestBody);
        } catch (IOException | ResponseException e) {
            // handle
        }
    }
    return newFixedLengthResponse(Response.Status.NOT_FOUND, MIME_PLAINTEXT, 
      "The requested resource does not exist");
}
```

Notice that before when we asked for the request body, **we first called the `parseBody` method.** That's because we wanted to load the request body for later retrieval.

我们将在我们的`cURL`命令中包含一个实体:

```java
> curl -X POST -d 'deliveryAddress=Washington nr 4&quantity;=5''http://localhost:8080/'
Request body = deliveryAddress=Washington nr 4&quantity;=5
```

其余的 HTTP 方法本质上非常相似，所以我们将跳过它们。

## 4.跨产地资源共享

Using [CORS](/web/20220628131736/https://www.baeldung.com/spring-cors), we enable cross-domain communication. The most common use case is AJAX calls from a different domain.The first approach that we can use is to enable CORS for all our APIs. Using the `–``-cors` argument, we’ll allow access to all domains. We can also define which domains we allow with `–cors=”http://dashboard.myApp.com http://admin.myapp.com”`.The second approach is to enable CORS for individual APIs. Let’s see how to use `addHeader` to achieve this:

```java
@Override 
public Response serve(IHTTPSession session) {
    Response response = newFixedLengthResponse("Hello world"); 
    response.addHeader("Access-Control-Allow-Origin", "*");
    return response;
}
```

现在当我们`cURL`时，我们将把我们的 CORS 头球拿回来:

```java
> curl -v 'http://localhost:8080'
HTTP/1.1 200 OK 
Content-Type: text/html
Date: Thu, 13 Jun 2019 03:58:14 GMT
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 11

Hello world
```

## 5.文件上传

NanoHTTPD 对于文件上传有一个单独的[依赖项，所以让我们将它添加到我们的项目:](https://web.archive.org/web/20220628131736/https://search.maven.org/search?q=a:nanohttpd-apache-fileupload)

```java
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd-apache-fileupload</artifactId>
    <version>2.3.1</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

请注意，还需要 [`servlet-api`依赖关系](https://web.archive.org/web/20220628131736/https://search.maven.org/search?q=a:javax.servlet-api%20AND%20g:javax.servlet)(否则我们会得到一个编译错误)。

NanoHTTPD 公开的是一个名为`NanoFileUpload`的类:

```java
@Override
public Response serve(IHTTPSession session) {
    try {
        List<FileItem> files
          = new NanoFileUpload(new DiskFileItemFactory()).parseRequest(session);
        int uploadedCount = 0;
        for (FileItem file : files) {
            try {
                String fileName = file.getName(); 
                byte[] fileContent = file.get(); 
                Files.write(Paths.get(fileName), fileContent);
                uploadedCount++;
            } catch (Exception exception) {
                // handle
            }
        }
        return newFixedLengthResponse(Response.Status.OK, MIME_PLAINTEXT, 
          "Uploaded files " + uploadedCount + " out of " + files.size());
    } catch (IOException | FileUploadException e) {
        throw new IllegalArgumentException("Could not handle files from API request", e);
    }
    return newFixedLengthResponse(
      Response.Status.BAD_REQUEST, MIME_PLAINTEXT, "Error when uploading");
}
```

嘿，让我们试试:

```java
> curl -F '[[email protected]](/web/20220628131736/https://www.baeldung.com/cdn-cgi/l/email-protection)/pathToFile.txt' 'http://localhost:8080'
Uploaded files: 1
```

## 6.多条路线

A `nanolet` is like a servlet but has a very low profile. We can use them to define many routes served by a single server (unlike previous examples with one route).

首先，让我们为`nanolets` 添加所需的[依赖项:](https://web.archive.org/web/20220628131736/https://search.maven.org/search?q=a:nanohttpd-nanolets)

```java
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd-nanolets</artifactId>
    <version>2.3.1</version>
</dependency>
```

现在我们将使用`RouterNanoHTTPD,` 来扩展我们的主类，定义我们的运行端口，并让服务器作为守护进程运行。

`addMappings`方法是我们定义处理程序的地方:

```java
public class MultipleRoutesExample extends RouterNanoHTTPD {
    public MultipleRoutesExample() throws IOException {
        super(8080);
        addMappings();
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    @Override
    public void addMappings() {
        // todo fill in the routes
    }
}
```

下一步是定义我们的`addMappings`方法。让我们定义几个处理程序。

第一个是 一个`IndexHandler`类到“/”路径。这个类带有 NanoHTTPD 库，默认情况下返回一条`Hello World`消息。当我们想要不同的响应时，我们可以覆盖`getText`方法:

```java
addRoute("/", IndexHandler.class); // inside addMappings method
```

为了测试我们的新路线，我们可以:

```java
> curl 'http://localhost:8080' 
<html><body><h2>Hello world!</h3></body></html>
```

其次，让我们创建一个新的`UserHandler` 类，它扩展了现有的`DefaultHandler.`类，它的路由将是/ `users`。在这里，我们处理了文本、MIME 类型和返回的状态代码:

```java
public static class UserHandler extends DefaultHandler {
    @Override
    public String getText() {
        return "UserA, UserB, UserC";
    }

    @Override
    public String getMimeType() {
        return MIME_PLAINTEXT;
    }

    @Override
    public Response.IStatus getStatus() {
        return Response.Status.OK;
    }
}
```

为了调用这条路线，我们将再次发出一个`cURL`命令:

```java
> curl -X POST 'http://localhost:8080/users' 
UserA, UserB, UserC
```

最后，我们可以用一个新的`StoreHandler`类来探索`GeneralHandler`。我们修改了返回的消息，以包含 URL 的`storeId`部分。

```java
public static class StoreHandler extends GeneralHandler {
    @Override
    public Response get(
      UriResource uriResource, Map<String, String> urlParams, IHTTPSession session) {
        return newFixedLengthResponse("Retrieving store for id = "
          + urlParams.get("storeId"));
    }
}
```

让我们检查一下我们的新 API:

```java
> curl 'http://localhost:8080/stores/123' 
Retrieving store for id = 123
```

## 7.HTTPS

为了使用 HTTPS，我们需要一个证书。请参考[我们关于 SSL](/web/20220628131736/https://www.baeldung.com/java-ssl) 的文章，了解更多深入信息。

我们可以使用类似于[的服务来加密](https://web.archive.org/web/20220628131736/https://letsencrypt.org/)，或者我们可以简单地生成一个自签名证书，如下所示:

```java
> keytool -genkey -keyalg RSA -alias selfsigned
  -keystore keystore.jks -storepass password -validity 360
  -keysize 2048 -ext SAN=DNS:localhost,IP:127.0.0.1  -validity 9999
```

接下来，我们将这个`keystore.jks` 复制到我们的类路径上的一个位置，比如 Maven 项目的`src/main/resources`文件夹。

之后，我们可以在对`NanoHTTPD#makeSSLSocketFactory`的调用中引用它:

```java
public class HttpsExample  extends NanoHTTPD {

    public HttpsExample() throws IOException {
        super(8080);
        makeSecure(NanoHTTPD.makeSSLSocketFactory(
          "/keystore.jks", "password".toCharArray()), null);
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    // main and serve methods
}
```

现在我们可以试试了。请注意— `insecure`参数的使用，因为默认情况下`cURL`无法验证我们的自签名证书:

```java
> curl --insecure 'https://localhost:8443'
HTTPS call is a success
```

## 8.WebSockets

NanoHTTPD 支持 [WebSockets](/web/20220628131736/https://www.baeldung.com/rest-vs-websockets) 。

让我们创建一个 WebSocket 的最简单的实现。为此，我们需要扩展`NanoWSD`类。我们还需要为 WebSocket: 添加 [`NanoHTTPD`依赖项](https://web.archive.org/web/20220628131736/https://search.maven.org/search?q=a:nanohttpd-websocket%20AND%20g:org.nanohttpd)

```java
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd-websocket</artifactId>
    <version>2.3.1</version>
</dependency>
```

对于我们的实现，我们将只回复一个简单的文本负载:

```java
public class WsdExample extends NanoWSD {
    public WsdExample() throws IOException {
        super(8080);
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    public static void main(String[] args) throws IOException {
        new WsdExample();
    }

    @Override
    protected WebSocket openWebSocket(IHTTPSession ihttpSession) {
        return new WsdSocket(ihttpSession);
    }

    private static class WsdSocket extends WebSocket {
        public WsdSocket(IHTTPSession handshakeRequest) {
            super(handshakeRequest);
        }

        //override onOpen, onClose, onPong and onException methods

        @Override
        protected void onMessage(WebSocketFrame webSocketFrame) {
            try {
                send(webSocketFrame.getTextPayload() + " to you");
            } catch (IOException e) {
                // handle
            }
        }
    }
}
```

这次我们不用`cURL`，而是用`[wscat](https://web.archive.org/web/20220628131736/https://github.com/websockets/wscat)`:

```java
> wscat -c localhost:8080
hello
hello to you
bye
bye to you
```

## 9.结论

总而言之，我们已经创建了一个使用 NanoHTTPD 库的项目。接下来，我们定义了 RESTful APIs 并探索了更多与 HTTP 相关的功能。最后，我们还实现了一个 WebSocket。

GitHub 上的[提供了所有这些代码片段的实现。](https://web.archive.org/web/20220628131736/https://github.com/eugenp/tutorials/tree/master/libraries-server)