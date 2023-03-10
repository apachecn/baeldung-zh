# 使用 JGroups 的可靠消息传递

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jgroups>

## 1。概述

JGroups 是一个用于可靠消息交换的 Java API。它有一个简单的界面，提供:

*   灵活的协议栈，包括 TCP 和 UDP
*   大型消息的分段和重组
*   可靠的单播和多播
*   故障探测
*   流控制

以及许多其他功能。

在本教程中，我们将创建一个简单的应用程序，用于在应用程序之间交换`String`消息，并在新应用程序加入网络时向它们提供共享状态。

## 2。设置

### 2.1。Maven 依赖关系

我们需要向我们的`pom.xml`添加一个依赖项:

```java
<dependency>
    <groupId>org.jgroups</groupId>
    <artifactId>jgroups</artifactId>
    <version>4.0.10.Final</version>
</dependency> 
```

最新版本的库可以在 [Maven Central 上查看。](https://web.archive.org/web/20221129004534/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jgroups%22%20AND%20a%3A%22jgroups%22)

### 2.2。联网

默认情况下，JGroups 将尝试使用 [IPV6](/web/20221129004534/https://www.baeldung.com/java-broadcast-multicast) 。根据我们的系统配置，这可能会导致应用程序无法通信。

为了避免这种情况，在这里运行我们的应用程序时，我们将设置`java.net.preferIPv4Stack`到`true`属性:

```java
java -Djava.net.preferIPv4Stack=true com.baeldung.jgroups.JGroupsMessenger 
```

## 3。JChannels

我们到 JGroups 网络的连接是一个`JChannel.`通道加入一个集群，发送和接收消息，以及关于网络状态的信息。

### 3.1。创建频道

我们创建一个带有配置文件路径的`JChannel`。如果我们省略文件名，它将在当前工作目录中寻找`udp.xml`。

我们将使用显式命名的配置文件创建一个通道:

```java
JChannel channel = new JChannel("src/main/resources/udp.xml"); 
```

JGroups 的配置可能非常复杂，但是默认的 UDP 和 TCP 配置对于大多数应用程序来说已经足够了。我们已经在代码中包含了 UDP 文件，并将在本教程中使用它。

有关配置传输的更多信息，请参见 JGroups 手册[此处](https://web.archive.org/web/20221129004534/http://jgroups.org/manual4/index.html#_transport_protocols)。

### 3.2。连接通道

创建通道后，我们需要加入集群。集群是一组交换消息的节点。

加入群集需要群集名称:

```java
channel.connect("Baeldung"); 
```

如果集群不存在，尝试加入集群的第一个节点将创建该集群。我们将在下面看到这个过程。

### 3.3。命名频道

节点通过名称来标识，以便对等点可以发送定向消息和接收关于谁进入和离开集群的通知。JGroups 将自动分配一个名称，或者我们可以设置自己的名称:

```java
channel.name("user1");
```

我们将在下面使用这些名称来跟踪节点何时进入和离开集群。

### 3.4。关闭一个通道

如果我们想让对等点及时收到我们已经退出的通知，通道清理是必不可少的。

我们用它的关闭方法关闭一个`JChannel`:

```java
channel.close()
```

## 4。集群视图变化

创建了 JChannel 之后，我们现在可以看到集群中对等体的状态，并与它们交换消息。

**JGroups 在`View`类中维护集群状态。**每个频道都有一个单独的`View`网络。当视图改变时，它通过`viewAccepted()`回调传递。

对于本教程，我们将扩展实现应用程序所需的所有接口方法的`ReceiverAdaptor` API 类。

这是实现回调的推荐方式。

让我们将`viewAccepted`添加到应用程序中:

```java
public void viewAccepted(View newView) {

    private View lastView;

    if (lastView == null) {
        System.out.println("Received initial view:");
        newView.forEach(System.out::println);
    } else {
        System.out.println("Received new view.");

        List<Address> newMembers = View.newMembers(lastView, newView);
        System.out.println("New members: ");
        newMembers.forEach(System.out::println);

        List<Address> exMembers = View.leftMembers(lastView, newView);
        System.out.println("Exited members:");
        exMembers.forEach(System.out::println);
    }
    lastView = newView;
} 
```

每个`View`包含一个`Address`对象的`List`，代表集群的每个成员。JGroups 提供了方便的方法来比较一个视图和另一个视图，我们用它来检测集群的新成员或退出成员。

## 5。发送消息

JGroups 中的消息处理非常简单。一个`Message`包含一个`byte`数组和对应于发送方和接收方的`Address`对象。

对于本教程，我们使用从命令行读取的`Strings` ,但是很容易看出应用程序如何交换其他数据类型。

### 5.1。广播消息

用目的地和字节数组创建一个`Message`;`JChannel`为我们设置发送者。**如果目标是`null`，*，*整个集群都会收到消息。**

我们将接受来自命令行的文本，并将其发送到集群:

```java
System.out.print("Enter a message: ");
String line = in.readLine().toLowerCase();
Message message = new Message(null, line.getBytes());
channel.send(message); 
```

如果我们运行程序的多个实例并发送这条消息(在我们实现下面的`receive()`方法之后)，所有的实例都会收到它，**包括发送者。**

### 5.2。阻止我们的消息

如果我们不想看到我们的消息，我们可以为此设置一个属性:

```java
channel.setDiscardOwnMessages(true); 
```

当我们运行前面的测试时，消息发送者没有收到它的广播消息。

### 5.3。直接消息

发送直接消息需要有效的`Address`。如果我们按名称引用节点，我们需要一种方法来查找`Address`。幸运的是，我们有 T2。

电流`View`始终可从`JChannel`获得:

```java
private Optional<address> getAddress(String name) { 
    View view = channel.view(); 
    return view.getMembers().stream()
      .filter(address -> name.equals(address.toString()))
      .findAny(); 
} 
```

`Address`名称可以通过类`toString()`方法获得，所以我们只需在集群成员的`List`中搜索我们想要的名称。

因此，我们可以从控制台接受一个名称，找到相关的目的地，并发送一条直接消息:

```java
Address destination = null;
System.out.print("Enter a destination: ");
String destinationName = in.readLine().toLowerCase();
destination = getAddress(destinationName)
  .orElseThrow(() -> new Exception("Destination not found"); 
Message message = new Message(destination, "Hi there!"); 
channel.send(message); 
```

## 6。接收消息

我们可以发送消息，现在让我们添加现在尝试接收它们。

让我们覆盖`ReceiverAdaptor's`空接收方法:

```java
public void receive(Message message) {
    String line = Message received from: " 
      + message.getSrc() 
      + " to: " + message.getDest() 
      + " -> " + message.getObject();
    System.out.println(line);
} 
```

因为我们知道消息包含一个`String`，我们可以安全地将`getObject()`传递给`System.out`。

## 7。状态交换

当节点进入网络时，它可能需要检索关于群集的状态信息。JGroups 为此提供了一种状态转移机制。

当一个节点加入集群时，它简单地调用`getState()`。集群通常从组中最老的成员——协调者那里检索状态。

让我们在应用程序中添加一个广播消息计数。我们将添加一个新的成员变量，并在`receive()`中递增它:

```java
private Integer messageCount = 0;

public void receive(Message message) {
    String line = "Message received from: " 
      + message.getSrc() 
      + " to: " + message.getDest() 
      + " -> " + message.getObject();
    System.out.println(line);

    if (message.getDest() == null) {
        messageCount++;
        System.out.println("Message count: " + messageCount);
    }
} 
```

我们检查一个`null`目的地，因为如果我们计算直接消息，每个节点将有不同的号码。

接下来，我们在`ReceiverAdaptor`中覆盖另外两个方法:

```java
public void setState(InputStream input) {
    try {
        messageCount = Util.objectFromStream(new DataInputStream(input));
    } catch (Exception e) {
        System.out.println("Error deserialing state!");
    }
    System.out.println(messageCount + " is the current messagecount.");
}

public void getState(OutputStream output) throws Exception {
    Util.objectToStream(messageCount, new DataOutputStream(output));
} 
```

与消息类似，JGroups 以一个数组`bytes`的形式传输状态。

JGroups 向协调器提供一个`InputStream`来写入状态，并向新节点提供一个`OutputStream`来读取状态。API 为序列化和反序列化数据提供了方便的类。

注意，在生产代码中，对状态信息的访问必须是线程安全的。

最后，在我们连接到集群之后，我们将对`getState()`的调用添加到我们的启动中:

```java
channel.connect(clusterName);
channel.getState(null, 0); 
```

`getState()`接受从其请求状态和超时(以毫秒为单位)的目的地。`null`目的地表示协调器，0 表示不超时。

当我们用一对节点运行这个应用程序并交换广播消息时，我们看到消息计数增加。

然后，如果我们添加第三个客户端，或者停止并启动其中一个客户端，我们将看到新连接的节点打印出正确的消息计数。

## 8。结论

在本教程中，我们使用 JGroups 创建了一个交换消息的应用程序。我们使用 API 来监控哪些节点连接和离开集群，并在新节点加入时将集群状态转移到新节点。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20221129004534/https://github.com/eugenp/tutorials/tree/master/messaging-modules/jgroups)