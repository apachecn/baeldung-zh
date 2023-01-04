# WebRTC 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/webrtc>

## 1。概述

当两个浏览器需要通信时，它们之间通常需要一个服务器来协调通信，在它们之间传递消息。但是中间有一个服务器会导致浏览器之间的通信延迟。

在本教程中，我们将了解 [WebRTC](https://web.archive.org/web/20220626194418/https://webrtc.org/) ，这是一个开源项目，**使浏览器和移动应用程序能够直接实时通信。**然后我们将通过编写一个简单的应用程序来看到它的实际应用，该应用程序创建一个对等连接来在两个 HTML 客户端之间共享数据。

我们将使用 HTML、JavaScript 和 WebSocket 库以及 web 浏览器中内置的 WebRTC 支持来构建客户端。此外，我们将使用 WebSocket 作为通信协议，与 Spring Boot 一起构建一个信令服务器。最后，我们将了解如何向该连接添加视频和音频流。

## 2。WebRTC 的基础和概念

让我们看看在没有 WebRTC 的典型场景下，两个浏览器是如何通信的。

假设我们有两个浏览器，`Browser 1`需要向`Browser 2`发送消息。`Browser 1`首先将其发送给`Server`:

[![](img/dbb87c00c98de5d94684778d5044abb3.png)](/web/20220626194418/https://www.baeldung.com/wp-content/uploads/2019/12/webrtc-2-1.png)

在`Server`收到消息后，它处理它，找到`Browser 2`，并向它发送消息:

[![](img/4d2e8b582132f5f1286b7409ca2285cc.png)](/web/20220626194418/https://www.baeldung.com/wp-content/uploads/2019/12/webrtc-2-2.png)

由于服务器必须在将消息发送到浏览器 2 之前对其进行处理，因此通信是实时进行的。当然，我们希望它是实时的。

WebRTC 通过在两个浏览器之间创建一个**直接通道解决了这个问题，不再需要服务器**:

[![](img/43a14b1e282c2ecad3890a3de1c0bba8.png)](/web/20220626194418/https://www.baeldung.com/wp-content/uploads/2019/12/webrtc-2-3.png)

因此，将消息从一个浏览器传递到另一个浏览器所需的时间大大减少，因为**消息现在直接从发送者路由到接收者**。它还消除了服务器带来的沉重负担和带宽，并使其在所涉及的客户机之间共享。

## 3.支持 WebRTC 和内置特性

WebRTC 受到 Chrome、Firefox、Opera 和 Microsoft Edge 等主流浏览器以及 Android 和 iOS 等平台的支持。

**WebRTC 不需要在我们的浏览器中安装任何外部插件，因为该解决方案是与浏览器捆绑在一起的。**

此外，在涉及视频和音频传输的典型实时应用中，我们必须高度依赖 C++库，并且必须处理许多问题，包括:

*   丢包隐藏
*   回波消除
*   带宽适应性
*   动态抖动缓冲
*   自动增益控制
*   噪音降低和抑制
*   图像“清洁”

但是 **WebRTC 在幕后处理所有这些问题**，使得客户端之间的实时通信更加简单。

## 4。点对点连接

与客户端-服务器通信不同，在 P2P(对等)连接中，没有一个对等体有到另一个对等体的直接地址，在客户端-服务器通信中，服务器有一个已知的地址，并且客户端已经知道要与之通信的服务器的地址。

要建立对等连接，只需几个步骤，客户端就可以:

*   让他们可以交流
*   相互识别并共享网络相关信息
*   共享并同意所涉及的数据、模式和协议的格式
*   共享日期

WebRTC 定义了一组 API 和方法来执行这些步骤。

对于客户端发现彼此，共享网络细节，然后共享数据的格式， **WebRTC 使用一种叫做`signaling`** 的机制。

## 5。信令

信令指的是网络发现、会话创建、管理会话和交换媒体能力元数据中涉及的过程。

这一点至关重要，因为客户需要事先相互了解才能开始沟通。

为了实现所有这些， **WebRTC 没有为信令指定标准，而是将其留给开发人员来实现。**因此，这为我们在一系列采用任何技术和支持协议的设备上使用 WebRTC 提供了灵活性。

### 5.1.构建信令服务器

对于信令服务器，我们将**使用 Spring Boot** 构建一个 WebSocket 服务器。我们可以从一个从 [Spring Initializr](https://web.archive.org/web/20220626194418/https://start.spring.io/) 生成的空 Spring Boot 项目开始。

为了在我们的实现中使用 WebSocket，让我们将依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
    <version>2.4.0</version>
</dependency>
```

我们总能从 [Maven Central](https://web.archive.org/web/20220626194418/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-websocket) 找到最新的版本来使用。

信令服务器的实现很简单——我们将创建一个端点，客户端应用程序可以使用它来注册 WebSocket 连接。

为了在 Spring Boot 做到这一点，让我们编写一个扩展了`WebSocketConfigurer`并覆盖了`registerWebSocketHandlers`方法的`@Configuration`类:

```
@Configuration
@EnableWebSocket
public class WebSocketConfiguration implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new SocketHandler(), "/socket")
          .setAllowedOrigins("*");
    }
}
```

请注意，我们已经将`/socket`标识为 URL，我们将从下一步构建的客户端注册该 URL。我们还传入了一个`SocketHandler`作为`addHandler`方法的参数——这实际上是我们接下来要创建的消息处理程序。

### 5.2.在信令服务器中创建消息处理程序

下一步是创建一个消息处理程序来处理我们将从多个客户端接收的 WebSocket 消息。

**这对于帮助不同客户端之间的元数据交换以建立直接的 WebRTC 连接是必不可少的**。

这里，为了简单起见，当我们从一个客户端接收到消息时，我们将把它发送给除它自己之外的所有其他客户端。

为此，我们可以从 Spring WebSocket 库中**扩展** **`TextWebSocketHandler`，并覆盖`handleTextMessage`和`afterConnectionEstablished`方法:**

```
@Component
public class SocketHandler extends TextWebSocketHandler {

    List<WebSocketSession>sessions = new CopyOnWriteArrayList<>();

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message)
      throws InterruptedException, IOException {
        for (WebSocketSession webSocketSession : sessions) {
            if (webSocketSession.isOpen() && !session.getId().equals(webSocketSession.getId())) {
                webSocketSession.sendMessage(message);
            }
        }
    }

    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        sessions.add(session);
    }
} 
```

正如我们在`afterConnectionEstablished `方法中看到的，我们将接收到的会话添加到会话列表中，这样我们就可以跟踪所有的客户端。

当我们从任何客户端收到消息时，正如在`handleTextMessage,` 中可以看到的，我们迭代列表中的所有客户端会话，并通过比较发送方的会话 id 和列表中的会话，将消息发送给除发送方之外的所有其他客户端。

## 6。交换元数据

在 P2P 连接中，客户端可能彼此非常不同。例如，Android 上的 Chrome 可以连接到 Mac 上的 Mozilla。

因此，这些设备的媒体功能可能会有很大差异。因此，对等体之间的握手必须就用于通信的媒体类型和编解码器达成一致。

在这个阶段， **WebRTC 使用 SDP(会话描述协议)来协商客户端之间的元数据。**

为了实现这一点，发起对等体创建一个 offer，该 offer 必须被另一个对等体设置为远程描述符。此外，另一个对等体随后生成一个应答，该应答被发起对等体作为远程描述符接受。

这个过程完成后，连接就建立了。

## 7。设置客户端

让我们创建我们的 WebRTC 客户机，这样它既可以充当发起对等点，也可以充当远程对等点。

我们将首先创建一个名为`index.html`的 HTML 文件和一个名为`client.js `的 JavaScript 文件，供`index.html `使用。

为了连接到我们的信令服务器，我们创建了一个 WebSocket 连接。假设我们构建的 Spring Boot 信令服务器运行在 [`http://localhost:8080`](https://web.archive.org/web/20220626194418/http://localhost:8080/) 上，我们可以创建连接:

```
var conn = new WebSocket('ws://localhost:8080/socket');
```

为了向信令服务器发送消息，我们将创建一个`send`方法，用于在接下来的步骤中传递消息:

```
function send(message) {
    conn.send(JSON.stringify(message));
}
```

## 8.设置一个简单的**`RTCDataChannel`**

在`client.js`中设置好客户端后，我们需要为 **`RTCPeerConnection`** 类创建一个对象:

```
configuration = null;
var peerConnection = new RTCPeerConnection(configuration);
```

在这个例子中，configuration 对象的目的是传入 STUN(NAT 的会话遍历实用程序)和 TURN(使用 NAT 周围的中继进行遍历)服务器以及我们将在本教程的后半部分讨论的其他配置。对于这个例子，传入`null`就足够了。

现在，我们可以创建一个`dataChannel`用于消息传递:

```
var dataChannel = peerConnection.createDataChannel("dataChannel", { reliable: true });
```

随后，我们可以为数据通道上的各种事件创建侦听器:

```
dataChannel.onerror = function(error) {
    console.log("Error:", error);
};
dataChannel.onclose = function() {
    console.log("Data channel is closed");
};
```

## 9。与 ICE 建立连接

建立 WebRTC 连接的下一步涉及 ICE(交互式连接建立)和 SDP 协议，其中对等体的会话描述在两个对等体上交换和接受。

信令服务器用于在对等体之间发送该信息。这涉及一系列步骤，其中客户端通过信令服务器交换连接元数据。

### 9.1。创建报价

首先，我们创建一个`offer`，并将其设置为`peerConnection`的本地描述。然后我们将`offer`发送给另一个对等体:

```
peerConnection.createOffer(function(offer) {
    send({
        event : "offer",
        data : offer
    });
    peerConnection.setLocalDescription(offer);
}, function(error) {
    // Handle error here
});
```

这里，`send`方法调用信令服务器来传递`offer`信息。

注意，我们可以用任何服务器端技术自由实现 send 方法的逻辑。

### 9.2。处理候选冰

其次，我们需要处理 ICE 候选人。 **WebRTC 使用 ICE(交互式连接建立)协议来发现对等点并建立连接。**

当我们在`peerConnection`上设置本地描述时，它会触发一个`icecandidate`事件。

该事件应该将候选对象传输到远程对等体，以便远程对等体可以将其添加到其远程候选对象集中。

为此，我们为`onicecandidate`事件创建一个监听器:

```
peerConnection.onicecandidate = function(event) {
    if (event.candidate) {
        send({
            event : "candidate",
            data : event.candidate
        });
    }
};
```

当所有的候选人都被收集时，`icecandidate`事件再次触发，候选人字符串为空。

我们还必须将这个候选对象传递给远程对等体。我们传递这个空的候选字符串来确保远程对等点知道所有的`icecandidate`对象都被收集了。

同样，同样的事件被再次触发，以指示候选冰收集完成，事件上的`candidate`对象的值被设置为`null`。这不需要传递给远程对等体。

### 9.3。接收 ICE 候选者

第三，我们需要处理另一个对等体发送的 ICE 候选。

**远程对等体在接收到此`candidate`后，应该将其添加到其候选池:**

```
peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
```

### 9.4。接受报价

此后，**当对方收到`offer`时，必须将其设置为远程描述`.`** 此外，还必须生成一个`answer`，发送给发起方:

```
peerConnection.setRemoteDescription(new RTCSessionDescription(offer));
peerConnection.createAnswer(function(answer) {
    peerConnection.setLocalDescription(answer);
        send({
            event : "answer",
            data : answer
        });
}, function(error) {
    // Handle error here
});
```

### 9.5。接收答案

最后，**发起对等体接收到`answer`并将其设置为** **远程描述**:

```
handleAnswer(answer){
    peerConnection.setRemoteDescription(new RTCSessionDescription(answer));
}
```

这样，WebRTC 建立了一个成功的连接。

现在，**我们可以直接在两个对等体之间发送和接收数据**，而不需要信令服务器。

## 10。发送消息

现在我们已经建立了连接，**我们可以使用** `**dataChannel**`的`send`方法在对等体之间发送消息:

```
dataChannel.send(“message”);
```

同样，为了在另一个对等体上接收消息，让我们为`onmessage`事件创建一个监听器:

```
dataChannel.onmessage = function(event) {
    console.log("Message:", event.data);
};
```

为了在数据通道上接收消息，我们还必须在`peerConnection`对象上添加一个回调:

```
peerConnection.ondatachannel = function (event) {
    dataChannel = event.channel;
};
```

通过这一步，我们已经创建了一个功能完整的 WebRTC 数据通道。我们现在可以在客户端之间发送和接收数据。此外，我们可以添加视频和音频通道。

## 11。添加视频和音频通道

当 WebRTC 建立 P2P 连接时，我们可以很容易地直接传输音频和视频流。

### 11.1。获取媒体流

首先，我们需要从浏览器获取媒体流。WebRTC 为此提供了一个 API:

```
const constraints = {
    video: true,audio : true
};
navigator.mediaDevices.getUserMedia(constraints).
  then(function(stream) { /* use the stream */ })
    .catch(function(err) { /* handle the error */ });
```

**我们可以使用约束对象指定视频的帧速率、宽度和高度。**

约束对象还允许指定移动设备中使用的摄像机:

```
var constraints = {
    video : {
        frameRate : {
            ideal : 10,
            max : 15
        },
        width : 1280,
        height : 720,
        facingMode : "user"
    }
};
```

此外，如果我们想要启用背面摄像头，可以将`facingMode`的值设置为`“environment”`而不是`“user”`。

### 11.2。发送流

其次，我们必须将流添加到 WebRTC 对等连接对象中:

```
peerConnection.addStream(stream);
```

**将流添加到对等连接会触发已连接对等端上的`addstream` 事件。**

### 11.3。接收流

第三，**为了在远程对等体上接收流，我们可以创建一个监听器**。

让我们将这个流设置为一个 HTML 视频元素:

```
peerConnection.onaddstream = function(event) {
    videoElement.srcObject = event.stream;
};
```

## 12。NAT 问题

在现实世界中，防火墙和 NAT(网络地址遍历)设备将我们的设备连接到公共互联网。

**NAT 为设备提供了一个在本地网络中使用的 IP 地址。因此，该地址在本地网络之外是不可访问的。**没有公共地址，对等方无法与我们交流。

为了解决这个问题，WebRTC 使用了两种机制:

1.  使目瞪口呆
2.  转动

## 13。使用 **击晕**

STUN 是解决这个问题的最简单的方法。在向对等方共享网络信息之前，客户端向 STUN 服务器发出请求。STUN 服务器的职责是返回接收请求的 IP 地址。

因此，通过查询 STUN 服务器，我们获得了自己面向公众的 IP 地址。然后，我们将这个 IP 和端口信息共享给我们想要连接的对等体。其他对等体可以做同样的事情来共享他们面向公众的 IP。

要使用 STUN 服务器，我们可以简单地在配置对象中传递 URL 来创建`RTCPeerConnection`对象:

```
var configuration = {
    "iceServers" : [ {
        "url" : "stun:stun2.1.google.com:19302"
    } ]
}; 
```

## 14。利用T2 转

相反，TURN 是在 WebRTC 无法建立 P2P 连接时使用的一种回退机制。TURN 服务器的作用是在对等体之间直接中继数据。在这种情况下，实际的数据流流经 TURN 服务器。使用默认实现，TURN 服务器也充当 STUN 服务器。

TURN 服务器是公开可用的，即使它们位于防火墙或代理之后，客户端也可以访问它们。

但是，使用 TURN 服务器并不是真正的 P2P 连接，因为存在中间服务器。

**注意:TURN 是我们无法建立 P2P 连接时的最后手段。**当数据流经 TURN 服务器时，它需要大量的带宽，在这种情况下，我们没有使用 P2P。

与 STUN 类似，我们可以在同一个配置对象中提供 TURN 服务器 URL:

```
{
  'iceServers': [
    {
      'urls': 'stun:stun.l.google.com:19302'
    },
    {
      'urls': 'turn:10.158.29.39:3478?transport=udp',
      'credential': 'XXXXXXXXXXXXX',
      'username': 'XXXXXXXXXXXXXXX'
    },
    {
      'urls': 'turn:10.158.29.39:3478?transport=tcp',
      'credential': 'XXXXXXXXXXXXX',
      'username': 'XXXXXXXXXXXXXXX'
    }
  ]
}
```

## 15。结论

在本教程中，我们讨论了什么是 WebRTC 项目，并介绍了它的基本概念。然后，我们构建了一个简单的应用程序，在两个 HTML 客户机之间共享数据。

我们还讨论了创建和建立 WebRTC 连接的步骤。

此外，我们研究了使用 STUN 和 TURN 服务器作为 WebRTC 失败时的后备机制。

你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20220626194418/https://github.com/eugenp/tutorials/tree/master/webrtc)