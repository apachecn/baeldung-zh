# Java 套接字指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/a-guide-to-java-sockets>

## 1。概述

术语`socket` *编程*是指编写在多台计算机上执行的程序，其中的设备都通过网络相互连接。

有两种通信协议我们可以用于 socket 编程:**用户数据报协议(UDP)和传输控制协议(TCP)** 。

两者之间的主要区别是 UDP 是无连接的，这意味着客户端和服务器之间没有会话，而 TCP 是面向连接的，这意味着必须首先在客户端和服务器之间建立独占连接才能进行通信。

本教程介绍了 TCP/IP 网络上的套接字编程，并演示了如何用 Java 编写客户机/服务器应用程序。UDP 不是主流协议，因此可能不会经常遇到。

## 2。项目设置

Java 提供了一组类和接口，负责客户端和服务器之间的底层通信细节。

这些主要包含在`java.net`包中，所以我们需要进行如下导入:

```java
import java.net.*;
```

我们还需要`java.io`包，它为我们提供了输入和输出流，以便在通信时写入和读取:

```java
import java.io.*;
```

为了简单起见，我们将在同一台计算机上运行我们的客户机和服务器程序。如果我们在不同的联网计算机上执行它们，唯一会改变的是 IP 地址。在这种情况下，我们将在`127.0.0.1`上使用`localhost`。

## 3。简单的例子

让我们用涉及客户机和服务器的最基本的例子来弄脏我们的手。这将是一个双向通信应用程序，其中客户端问候服务器，服务器响应。

我们将用下面的代码在名为`GreetServer.java`的类中创建服务器应用程序。

我们将包括`main`方法和全局变量，以引起对本文中我们将如何运行所有服务器的注意。对于本文的其余示例，我们将省略这种重复的代码:

```java
public class GreetServer {
    private ServerSocket serverSocket;
    private Socket clientSocket;
    private PrintWriter out;
    private BufferedReader in;

    public void start(int port) {
        serverSocket = new ServerSocket(port);
        clientSocket = serverSocket.accept();
        out = new PrintWriter(clientSocket.getOutputStream(), true);
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        String greeting = in.readLine();
            if ("hello server".equals(greeting)) {
                out.println("hello client");
            }
            else {
                out.println("unrecognised greeting");
            }
    }

    public void stop() {
        in.close();
        out.close();
        clientSocket.close();
        serverSocket.close();
    }
    public static void main(String[] args) {
        GreetServer server=new GreetServer();
        server.start(6666);
    }
}
```

我们还将使用以下代码创建一个名为`GreetClient.java`的客户端:

```java
public class GreetClient {
    private Socket clientSocket;
    private PrintWriter out;
    private BufferedReader in;

    public void startConnection(String ip, int port) {
        clientSocket = new Socket(ip, port);
        out = new PrintWriter(clientSocket.getOutputStream(), true);
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
    }

    public String sendMessage(String msg) {
        out.println(msg);
        String resp = in.readLine();
        return resp;
    }

    public void stopConnection() {
        in.close();
        out.close();
        clientSocket.close();
    }
}
```

现在让我们启动服务器。在我们的 IDE 中，我们只需将它作为 Java 应用程序运行即可。

然后，我们将使用单元测试向服务器发送问候，确认服务器发送问候作为响应:

```java
@Test
public void givenGreetingClient_whenServerRespondsWhenStarted_thenCorrect() {
    GreetClient client = new GreetClient();
    client.startConnection("127.0.0.1", 6666);
    String response = client.sendMessage("hello server");
    assertEquals("hello client", response);
}
```

这个例子让我们对本文后面的内容有所了解。因此，我们可能还没有完全理解这里发生了什么。

在接下来的部分中，我们将使用这个简单的例子来剖析**套接字通信**，并深入到更复杂的例子中。

## 4。插座如何工作

我们将使用上面的例子来逐步完成这一部分的不同部分。

根据定义，`socket`是网络上不同计算机上运行的两个程序之间双向通信链路的一个端点。[一个套接字绑定到一个端口号](/web/20221003154930/https://www.baeldung.com/cs/port-vs-socket)，这样传输层就可以识别数据要发送到的应用程序。

### 4.1。服务器

通常，服务器运行在网络上的特定计算机上，并有一个绑定到特定端口号的套接字。在我们的例子中，我们将使用同一台计算机作为客户机，并在端口`6666`上启动服务器:

```java
ServerSocket serverSocket = new ServerSocket(6666);
```

服务器只是等待，监听客户机发出连接请求的套接字。这发生在下一步:

```java
Socket clientSocket = serverSocket.accept();
```

当服务器代码遇到`accept`方法时，它会阻塞，直到客户端向它发出连接请求。

如果一切顺利，服务器`accepts`连接。接受之后，服务器获得一个新的套接字`clientSocket`，绑定到同一个本地端口`6666`，并将其远程端点设置为客户端的地址和端口。

此时，新的`Socket`对象将服务器与客户机直接连接起来。然后，我们可以访问输出和输入流，分别向客户端写入消息和从客户端接收消息:

```java
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
```

现在，服务器能够与客户机无休止地交换消息，直到套接字关闭其流。

然而，在我们的示例中，服务器只能在关闭连接之前发送问候响应。这意味着如果我们再次运行测试，服务器将拒绝连接。

为了保持通信的连续性，我们必须在一个`while`循环中读取输入流，并且只有当客户端发送终止请求时才退出。我们将在下一节中看到这一点。

对于每个新的客户机，服务器需要一个由`accept`调用返回的新套接字。我们使用`serverSocket` 继续监听连接请求，同时满足连接客户端的需求。我们在第一个例子中还没有考虑到这一点。

### 4.2。客户端

客户端必须知道运行服务器的机器的主机名或 IP，以及服务器监听的端口号。

要发出连接请求，客户端会尝试在服务器的机器和端口上与服务器会合:

```java
Socket clientSocket = new Socket("127.0.0.1", 6666);
```

客户机还需要向服务器表明自己的身份，因此它绑定到系统分配的本地端口号，它将在连接过程中使用这个端口号。我们自己不处理这件事。

上面的构造函数只在服务器有连接的时候创建一个新的套接字；否则，我们会得到一个连接被拒绝的异常。成功创建后，我们可以从它那里获得输入和输出流，以便与服务器通信:

```java
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
```

客户端的输入流连接到服务器的输出流，就像服务器的输入流连接到客户端的输出流一样。

## 5。持续通信

我们当前的服务器会一直阻塞，直到有客户端连接到它，然后再次阻塞，以侦听来自客户端的消息。在单个消息之后，它关闭了连接，因为我们没有处理连续性。

因此，它只对 ping 请求有帮助。但是想象一下，我们想要实现一个聊天服务器；服务器和客户机之间持续的来回通信肯定是必需的。

我们将不得不创建一个 while 循环来持续观察服务器的输入流以获取输入消息。

因此，让我们创建一个名为`EchoServer.java,`的新服务器，它的唯一目的是回显它从客户端接收到的任何消息:

```java
public class EchoServer {
    public void start(int port) {
        serverSocket = new ServerSocket(port);
        clientSocket = serverSocket.accept();
        out = new PrintWriter(clientSocket.getOutputStream(), true);
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));

        String inputLine;
        while ((inputLine = in.readLine()) != null) {
        if (".".equals(inputLine)) {
            out.println("good bye");
            break;
         }
         out.println(inputLine);
    }
}
```

注意，我们添加了一个终止条件，当我们收到一个句点字符时，while 循环退出。

我们将使用 main 方法开始`EchoServer`，就像我们对`GreetServer`所做的一样。这一次，我们从另一个港口开始，比如`4444,`，以避免混淆。

`EchoClient`类似于`GreetClient`，所以我们可以复制代码。为了清楚起见，我们将它们分开。

在一个不同的测试类中，我们将创建一个测试来显示对`EchoServer`的多个请求将在服务器不关闭套接字的情况下得到服务。只要我们从同一个客户端发送请求，这就是真的。

处理多个客户端是不同的情况，我们将在随后的部分中看到。

现在让我们创建一个`setup`方法来启动与服务器的连接:

```java
@Before
public void setup() {
    client = new EchoClient();
    client.startConnection("127.0.0.1", 4444);
}
```

我们还将创建一个`tearDown`方法来释放我们所有的资源。对于我们使用网络资源的每种情况，这都是最佳实践:

```java
@After
public void tearDown() {
    client.stopConnection();
}
```

然后，我们将使用几个请求来测试我们的 echo 服务器:

```java
@Test
public void givenClient_whenServerEchosMessage_thenCorrect() {
    String resp1 = client.sendMessage("hello");
    String resp2 = client.sendMessage("world");
    String resp3 = client.sendMessage("!");
    String resp4 = client.sendMessage(".");

    assertEquals("hello", resp1);
    assertEquals("world", resp2);
    assertEquals("!", resp3);
    assertEquals("good bye", resp4);
}
```

与最初的例子相比，这是一个改进，在最初的例子中，在服务器关闭我们的连接之前，我们只通信一次。**现在我们发送一个终止信号，告诉服务器我们完成了会话**。

## 6。多客户端服务器

尽管前面的例子比第一个有所改进，但它仍然不是一个很好的解决方案。一台服务器必须有能力同时为许多客户机和许多请求提供服务。

我们将在这一部分讨论如何处理多个客户。

我们将在这里看到的另一个特性是，同一个客户端可以断开连接并重新连接，而不会出现连接被拒绝异常或在服务器上重置连接。我们以前无法做到这一点。

这意味着我们的服务器对于来自多个客户端的多个请求将变得更加健壮和有弹性。

为此，我们将为每个新客户端创建一个新的套接字，并在不同的线程上为该客户端的请求提供服务。同时接受服务的客户端数量将等于正在运行的线程数量。

主线程在侦听新连接时将运行 while 循环。

现在让我们来看看实际情况。我们将在其中创建另一个名为`EchoMultiServer.java.`的服务器，我们将创建一个处理程序线程类来管理每个客户端在其套接字上的通信:

```java
public class EchoMultiServer {
    private ServerSocket serverSocket;

    public void start(int port) {
        serverSocket = new ServerSocket(port);
        while (true)
            new EchoClientHandler(serverSocket.accept()).start();
    }

    public void stop() {
        serverSocket.close();
    }

    private static class EchoClientHandler extends Thread {
        private Socket clientSocket;
        private PrintWriter out;
        private BufferedReader in;

        public EchoClientHandler(Socket socket) {
            this.clientSocket = socket;
        }

        public void run() {
            out = new PrintWriter(clientSocket.getOutputStream(), true);
            in = new BufferedReader(
              new InputStreamReader(clientSocket.getInputStream()));

            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                if (".".equals(inputLine)) {
                    out.println("bye");
                    break;
                }
                out.println(inputLine);
            }

            in.close();
            out.close();
            clientSocket.close();
    }
}
```

注意，我们现在在一个`while`循环中调用`accept`。每次执行`while`循环时，它都会阻塞`accept`调用，直到有新的客户端连接。然后，为这个客户端创建处理程序线程`EchoClientHandler`。

线程内部发生的事情与我们只处理一个客户端的`EchoServer,`相同。`EchoMultiServer`将这项工作委托给`EchoClientHandler`，这样它就可以在`while`循环中继续监听更多的客户端。

我们仍将使用`EchoClient`来测试服务器。这一次，我们将创建多个客户机，每个客户机从服务器发送和接收多条消息。

让我们使用端口`5555`上的 main 方法启动我们的服务器。

为了清楚起见，我们仍然将测试放在一个新的套件中:

```java
@Test
public void givenClient1_whenServerResponds_thenCorrect() {
    EchoClient client1 = new EchoClient();
    client1.startConnection("127.0.0.1", 5555);
    String msg1 = client1.sendMessage("hello");
    String msg2 = client1.sendMessage("world");
    String terminate = client1.sendMessage(".");

    assertEquals(msg1, "hello");
    assertEquals(msg2, "world");
    assertEquals(terminate, "bye");
}

@Test
public void givenClient2_whenServerResponds_thenCorrect() {
    EchoClient client2 = new EchoClient();
    client2.startConnection("127.0.0.1", 5555);
    String msg1 = client2.sendMessage("hello");
    String msg2 = client2.sendMessage("world");
    String terminate = client2.sendMessage(".");

    assertEquals(msg1, "hello");
    assertEquals(msg2, "world");
    assertEquals(terminate, "bye");
}
```

我们可以创建尽可能多的测试用例，每一个都产生一个新的客户端，服务器将为它们提供服务。

## 7。结论

在本文中，我们关注于**TCP/IP 上套接字编程的介绍，**并用 Java 编写了一个简单的客户机/服务器应用程序。

本文的完整源代码可以在 [GitHub](https://web.archive.org/web/20221003154930/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking) 项目中找到。