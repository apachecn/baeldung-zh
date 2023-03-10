# Spring Websockets 的@ SendToUser 注释的一个简单例子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-websockets-sendtouser>

## 1.概观

在这个快速教程中，我们将演示**如何使用 Spring WebSockets** 向特定会话或特定用户发送消息。

关于上述模块的介绍，请参考本文中的[。](/web/20220625221721/https://www.baeldung.com/websockets-spring)

## 2.WebSocket 配置

首先，我们需要**配置我们的消息代理和 WebSocket 应用程序端点**:

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig
  extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic/", "/queue/");
	config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
	registry.addEndpoint("/greeting");
    }	
}
```

通过`@EnableWebSocketMessageBroker`,我们**使用** `**STOMP**,`(代表面向流文本的消息传递协议)在 WebSocket 上实现了代理支持的消息传递。需要注意的是，这个注释需要和`@Configuration`一起使用。

扩展`AbstractWebSocketMessageBrokerConfigurer`不是强制性的，但是，举个简单的例子，定制导入的配置更容易。

在第一种方法中，我们设置了一个简单的基于内存的消息代理，在以`“/topic”`和`“/queue”`为前缀的目的地上将消息传送回客户端。

第二，我们在`“/greeting”`注册了 stomp 端点。

如果我们想要启用 SockJS，我们必须修改寄存器部分:

```java
registry.addEndpoint("/greeting").withSockJS();
```

## 3.通过拦截器获取会话 ID

获取会话 id 的一种方法是添加一个 Spring 拦截器，它将在握手期间被触发，并从请求数据中获取信息。

这个拦截器可以直接添加到`WebSocketConfig:`中

```java
@Override
public void registerStompEndpoints(StompEndpointRegistry registry) {

registry
  .addEndpoint("/greeting")
  .setHandshakeHandler(new DefaultHandshakeHandler() {

      public boolean beforeHandshake(
        ServerHttpRequest request, 
        ServerHttpResponse response, 
        WebSocketHandler wsHandler,
        Map attributes) throws Exception {

            if (request instanceof ServletServerHttpRequest) {
                ServletServerHttpRequest servletRequest
                 = (ServletServerHttpRequest) request;
                HttpSession session = servletRequest
                  .getServletRequest().getSession();
                attributes.put("sessionId", session.getId());
            }
                return true;
        }}).withSockJS();
    }
```

## 4\. WebSocket Endpoint

从 Spring 5.0.5.RELEASE 开始，由于 **`@SendToUser`注释的改进，不需要做任何定制，允许我们通过`/user/{sessionId}/…`而不是`/user/{user}/…`向用户目的地**发送消息。

这意味着注释依赖于输入消息的会话 id，有效地向会话私有的目的地发送回复:

```java
@Controller
public class WebSocketController {

    @Autowired
    private SimpMessageSendingOperations messagingTemplate;

    private Gson gson = new Gson();

    @MessageMapping("/message")
    @SendToUser("/queue/reply")
    public String processMessageFromClient(
      @Payload String message, 
      Principal principal) throws Exception {
	return gson
          .fromJson(message, Map.class)
          .get("name").toString();
    }

    @MessageExceptionHandler
    @SendToUser("/queue/errors")
    public String handleException(Throwable exception) {
        return exception.getMessage();
    }
}
```

需要注意的是，`@SendToUser`表示一个消息处理方法的返回值应该作为一个`Message`发送到指定的**目的地，前面加上`/user/{username}`**。

## 5.WebSocket 客户端

```java
function connect() {
    var socket = new WebSocket('ws://localhost:8080/greeting');
    ws = Stomp.over(socket);

    ws.connect({}, function(frame) {
        ws.subscribe("/user/queue/errors", function(message) {
            alert("Error " + message.body);
        });

        ws.subscribe("/user/queue/reply", function(message) {
            alert("Message " + message.body);
        });
    }, function(error) {
        alert("STOMP error " + error);
    });
}

function disconnect() {
    if (ws != null) {
        ws.close();
    }
    setConnected(false);
    console.log("Disconnected");
}
```

为`WebSocketConfiguration`中的映射创建一个指向`/greeting`的新`WebSocket`。

当我们为客户端订阅“`/user/queue/errors`”和“`/user/queue/reply`”时，我们使用上一节中的注释信息。

我们可以看到，`@SendToUser`指向了`queue/errors`，但是消息会被发送到`/user/queue/errors`。

## 6.结论

在本文中，我们探索了一种使用 Spring WebSocket 直接向用户或会话 id 发送消息的方法

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220625221721/https://github.com/eugenp/tutorials/tree/master/spring-websockets)