# WebSocket 的 Java API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-websockets>

## 1。概述

WebSocket 通过提供双向、全双工、实时客户端/服务器通信，为服务器和 web 浏览器之间的高效通信限制提供了一种替代方案。服务器可以随时向客户端发送数据。因为它运行在 TCP 之上，所以它还提供了低延迟的低级通信，并减少了每个消息**的开销。**

在本文中，我们将通过创建一个类似聊天的应用程序来看看 WebSockets 的 Java API。

## 2。JSR 356

JSR 356 或 Java API for WebSocket 指定了一个 API，Java 开发人员可以使用该 API 将 WebSockets 集成到他们的应用程序中——包括服务器端和 Java 客户端。

这个 Java API 提供了服务器端和客户端组件:

*   **服务器**:一切都在`javax.websocket.server`包里。
*   **客户端**:包`javax.websocket`的内容，由客户端 API 和服务器与客户端的公共库组成。

## 3。使用 WebSockets 建立聊天

我们将构建一个非常简单的类似聊天的应用程序。任何用户都可以从任何浏览器打开聊天窗口，输入自己的名字，登录聊天窗口，开始与连接到聊天窗口的每个人交流。

我们首先将最新的依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>javax.websocket</groupId>
    <artifactId>javax.websocket-api</artifactId>
    <version>1.1</version>
</dependency>
```

最新版本可以在[这里](https://web.archive.org/web/20221021211339/https://search.maven.org/classic/#search%7Cga%7C1%7Cjavax.websocket-api)找到。

为了将 Java `Objects`转换成 JSON 表示，反之亦然，我们将使用 Gson:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.0</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221021211339/https://search.maven.org/classic/#search%7Cga%7C1%7Ccom.google.code.gson) 资源库中获得。

### 3.1。端点配置

配置端点有两种方式:`annotation-`基于和基于扩展。您可以扩展`javax.websocket.Endpoint`类或者使用专用的方法级注释。与编程模型相比，注释模型可以产生更简洁的代码，因此注释已经成为编码的常规选择。在这种情况下，WebSocket 端点生命周期事件由以下批注处理:

*   `@ServerEndpoint:`如果用`@ServerEndpoint,`修饰，容器确保类的可用性，作为一个`WebSocket`服务器监听一个特定的 URI 空间
*   `@ClientEndpoint`:用这个注释修饰的类被当作一个`WebSocket`客户端
*   `@OnOpen`:当一个新的`WebSocket`连接启动时，容器调用一个带有`@OnOpen`的 Java 方法
*   `@OnMessage`:当消息被发送到端点时，用`@OnMessage,`标注的 Java 方法从`WebSocket`容器接收信息
*   `@OnError`:当通信出现问题时，调用带有`@OnError`的方法
*   `@OnClose`:用于修饰一个 Java 方法，当`WebSocket`连接关闭时，容器调用该方法

### 3.2。编写服务器端点

我们通过用`@ServerEndpoint`对 Java 类`WebSocket`服务器端点进行注释来声明它。我们还指定部署端点的 URI。URI 是相对于服务器容器的根目录定义的，并且必须以正斜杠开头:

```java
@ServerEndpoint(value = "/chat/{username}")
public class ChatEndpoint {

    @OnOpen
    public void onOpen(Session session) throws IOException {
        // Get session and WebSocket connection
    }

    @OnMessage
    public void onMessage(Session session, Message message) throws IOException {
        // Handle new messages
    }

    @OnClose
    public void onClose(Session session) throws IOException {
        // WebSocket connection closes
    }

    @OnError
    public void onError(Session session, Throwable throwable) {
        // Do error handling here
    }
}
```

上面的代码是我们类似聊天的应用程序的服务器端点框架。如您所见，我们有 4 个注释映射到它们各自的方法。下面你可以看到这样的方法的实现:

```java
@ServerEndpoint(value="/chat/{username}")
public class ChatEndpoint {

    private Session session;
    private static Set<ChatEndpoint> chatEndpoints 
      = new CopyOnWriteArraySet<>();
    private static HashMap<String, String> users = new HashMap<>();

    @OnOpen
    public void onOpen(
      Session session, 
      @PathParam("username") String username) throws IOException {

        this.session = session;
        chatEndpoints.add(this);
        users.put(session.getId(), username);

        Message message = new Message();
        message.setFrom(username);
        message.setContent("Connected!");
        broadcast(message);
    }

    @OnMessage
    public void onMessage(Session session, Message message) 
      throws IOException {

        message.setFrom(users.get(session.getId()));
        broadcast(message);
    }

    @OnClose
    public void onClose(Session session) throws IOException {

        chatEndpoints.remove(this);
        Message message = new Message();
        message.setFrom(users.get(session.getId()));
        message.setContent("Disconnected!");
        broadcast(message);
    }

    @OnError
    public void onError(Session session, Throwable throwable) {
        // Do error handling here
    }

    private static void broadcast(Message message) 
      throws IOException, EncodeException {

        chatEndpoints.forEach(endpoint -> {
            synchronized (endpoint) {
                try {
                    endpoint.session.getBasicRemote().
                      sendObject(message);
                } catch (IOException | EncodeException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

当一个新用户登录时(`@OnOpen`)立即被映射到一个活动用户的数据结构中。然后，使用`broadcast`方法创建一条消息并发送给所有端点。

每当任何连接的用户发送(`@OnMessage`)新消息时，也会使用这种方法——这是聊天的主要目的。

如果在某个时候出现错误，带有注释`@OnError`的方法会处理它。您可以使用此方法记录有关错误的信息并清除端点。

最后，当用户不再连接到聊天时，方法`@OnClose`清除端点，并向所有用户广播用户已经断开连接。

## 4。消息类型

WebSocket 规范支持两种在线数据格式——文本和二进制。API 支持这两种格式，增加了处理 Java 对象和健康检查消息(乒乓)的功能，如规范中所定义:

*   `Text`:任何文本数据(`java.lang.String`、原语或它们的等效包装类)
*   `Binary`:二进制数据(如音频、图像等。)由一个`java.nio.ByteBuffer`或一个`byte[]`(字节数组)表示
*   API 使得在你的代码中使用本地(Java 对象)表示成为可能，并使用定制的转换器(编码器/解码器)将它们转换成 WebSocket 协议允许的兼容的在线格式(文本、二进制)
*   `Ping-Pong`:`javax.websocket.PongMessage`是一个由 WebSocket 对等体响应健康检查(ping)请求而发送的确认

对于我们的应用程序，我们将使用`Java Objects.` 来创建编码和解码消息的类。

### 4.1。编码器

编码器接收一个 Java 对象，并产生一个适合作为消息传输的典型表示，如 JSON、XML 或二进制表示。可以通过实现`Encoder.Text<T>`或`Encoder.Binary<T>`接口来使用编码器。

在下面的代码中，我们定义了要编码的类`Message`，在方法`encode` 中，我们使用 Gson 将 Java 对象编码为 JSON:

```java
public class Message {
    private String from;
    private String to;
    private String content;

    //standard constructors, getters, setters
}
```

```java
public class MessageEncoder implements Encoder.Text<Message> {

    private static Gson gson = new Gson();

    @Override
    public String encode(Message message) throws EncodeException {
        return gson.toJson(message);
    }

    @Override
    public void init(EndpointConfig endpointConfig) {
        // Custom initialization logic
    }

    @Override
    public void destroy() {
        // Close resources
    }
}
```

### 4.2。解码器

解码器与编码器相反，用于将数据转换回 Java 对象。解码器可以使用`Decoder.Text<T>`或`Decoder.Binary<T>`接口实现。

正如我们在编码器中看到的，`decode` 方法是我们获取发送到端点的消息中检索到的 JSON，并使用 Gson 将其转换为名为`Message:`的 Java 类

```java
public class MessageDecoder implements Decoder.Text<Message> {

    private static Gson gson = new Gson();

    @Override
    public Message decode(String s) throws DecodeException {
        return gson.fromJson(s, Message.class);
    }

    @Override
    public boolean willDecode(String s) {
        return (s != null);
    }

    @Override
    public void init(EndpointConfig endpointConfig) {
        // Custom initialization logic
    }

    @Override
    public void destroy() {
        // Close resources
    }
}
```

### 4.3。在服务器端点设置编码器和解码器

让我们通过在类级注释`@ServerEndpoint`中添加为编码和解码数据而创建的类，将所有内容放在一起:

```java
@ServerEndpoint( 
  value="/chat/{username}", 
  decoders = MessageDecoder.class, 
  encoders = MessageEncoder.class )
```

每次消息被发送到端点时，它们将被自动转换成 JSON 或 Java 对象。

## 5。结论

在本文中，我们研究了什么是 WebSockets 的 Java API，以及它如何帮助我们构建像这种实时聊天这样的应用程序。

我们看到了创建端点的两种编程模型:注释和编程。我们使用应用程序的注释模型以及生命周期方法定义了一个端点。

此外，为了能够在服务器和客户机之间来回通信，我们需要编码器和解码器将 Java 对象转换成 JSON，反之亦然。

JSR 356 API 非常简单，基于注释的编程模型使得构建 WebSocket 应用程序非常容易。

要运行我们在示例中构建的应用程序，我们需要做的就是在 web 服务器中部署 war 文件，然后转到 URL: `http://localhost:8080/java-websocket/.` 您可以在这里找到到存储库[的链接。](https://web.archive.org/web/20221021211339/https://github.com/eugenp/tutorials/tree/master/java-websocket)