# Java 中的广播和组播

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-broadcast-multicast>

## 1。简介

在本文中，我们将描述如何在 Java 中处理一对多(广播)和一对多(组播)通信。本文中概述的[广播和组播概念](/web/20220709122341/https://www.baeldung.com/cs/multicast-vs-broadcast-anycast-unicast)基于 UDP 协议。

我们先快速回顾一下数据报和广播，以及它是如何在 Java 中实现的。我们还研究了广播的缺点，并提出组播作为广播的替代方案。

最后，我们将讨论在 [IPv4 和 IPv6](/web/20220709122341/https://www.baeldung.com/cs/ipv4-vs-ipv6) 中对这两种寻址方法的支持。

## 2。数据报摘要

根据[数据报的官方定义](https://web.archive.org/web/20220709122341/https://docs.oracle.com/javase/tutorial/networking/datagrams/)，“数据报是通过网络发送的独立、自包含的消息，其到达、到达时间和内容是不确定的”。

在 Java 中，`java.net`包公开了可用于通过 UDP 协议进行通信的`DatagramPacket`和`DatagramSocket` 类。UDP 通常用于低延迟比有保证的传输更重要的情况，例如音频/视频流、网络发现等。

要了解更多关于 Java 中的 UDP 和数据报，请参考[Java 中的 UDP 指南](/web/20220709122341/https://www.baeldung.com/udp-in-java)。

## 3 。**广播**

广播是一种一对多类型的通信，即目的是将数据报发送到网络中的所有节点。**与点对点通信的情况不同，** **我们不必知道目标主机的 IP 地址**。而是使用广播地址。

根据 IPv4 协议，广播地址是一个逻辑地址，连接到网络的设备可以通过它接收数据包。在我们的例子中，我们使用一个特定的 IP 地址`255.255.255.255`，它是本地网络的广播地址。

根据定义，将本地网络连接到其他网络的路由器不会转发发送到此默认广播地址的数据包。稍后我们还将展示如何遍历所有的`NetworkInterfaces`，并将数据包发送到它们各自的广播地址。

首先，我们演示如何广播消息。为此，我们需要调用套接字上的`setBroadcast()`方法，让它知道要广播数据包:

```java
public class BroadcastingClient {
    private static DatagramSocket socket = null;

    public static void main((String[] args)) throws IOException {
        broadcast("Hello", InetAddress.getByName("255.255.255.255"));
    }

    public static void broadcast(
      String broadcastMessage, InetAddress address) throws IOException {
        socket = new DatagramSocket();
        socket.setBroadcast(true);

        byte[] buffer = broadcastMessage.getBytes();

        DatagramPacket packet 
          = new DatagramPacket(buffer, buffer.length, address, 4445);
        socket.send(packet);
        socket.close();
    }
}
```

下一个片段展示了如何遍历所有的`NetworkInterfaces`来找到它们的广播地址:

```java
List<InetAddress> listAllBroadcastAddresses() throws SocketException {
    List<InetAddress> broadcastList = new ArrayList<>();
    Enumeration<NetworkInterface> interfaces 
      = NetworkInterface.getNetworkInterfaces();
    while (interfaces.hasMoreElements()) {
        NetworkInterface networkInterface = interfaces.nextElement();

        if (networkInterface.isLoopback() || !networkInterface.isUp()) {
            continue;
        }

        networkInterface.getInterfaceAddresses().stream() 
          .map(a -> a.getBroadcast())
          .filter(Objects::nonNull)
          .forEach(broadcastList::add);
    }
    return broadcastList;
}
```

一旦我们有了广播地址列表，我们就可以对每个地址执行上面显示的`broadcast()`方法中的代码。

在接收方不需要特殊代码来接收广播消息。我们可以重用接收普通 UDP 数据报的相同代码。[Java 中的 UDP 指南](/web/20220709122341/https://www.baeldung.com/udp-in-java)包含了这个主题的更多细节。

## 4。多播

广播是低效的，因为分组被发送到网络中的所有节点，而不管它们是否对接收通信感兴趣。这可能是资源的浪费。

多播解决了这个问题，它只将数据包发送给那些感兴趣的用户。**多播基于组成员概念**，其中多播地址代表每个组。

在 IPv4 中，224.0.0.0 到 239.255.255.255 之间的任何地址都可以用作组播地址。只有那些订阅组的节点接收传送到该组的分组。

在 Java 中，`MulticastSocket`用于接收发送到组播 IP 的数据包。下面的例子演示了`MulticastSocket`的用法:

```java
public class MulticastReceiver extends Thread {
    protected MulticastSocket socket = null;
    protected byte[] buf = new byte[256];

    public void run() {
        socket = new MulticastSocket(4446);
        InetAddress group = InetAddress.getByName("230.0.0.0");
        socket.joinGroup(group);
        while (true) {
            DatagramPacket packet = new DatagramPacket(buf, buf.length);
            socket.receive(packet);
            String received = new String(
              packet.getData(), 0, packet.getLength());
            if ("end".equals(received)) {
                break;
            }
        }
        socket.leaveGroup(group);
        socket.close();
    }
}
```

将`MulticastSocket`绑定到端口后，我们调用`joinGroup()`方法，将组播 IP 作为参数。这是接收发布到该组的数据包所必需的。可以用`leaveGroup()`的方法离开群体。

以下示例显示了如何发布到多播 IP:

```java
public class MulticastPublisher {
    private DatagramSocket socket;
    private InetAddress group;
    private byte[] buf;

    public void multicast(
      String multicastMessage) throws IOException {
        socket = new DatagramSocket();
        group = InetAddress.getByName("230.0.0.0");
        buf = multicastMessage.getBytes();

        DatagramPacket packet 
          = new DatagramPacket(buf, buf.length, group, 4446);
        socket.send(packet);
        socket.close();
    }
}
```

## 5。广播和 IPv6

IPv4 支持三种类型的寻址:单播、广播和组播。理论上，广播是一种一对一的通信，即从一个设备发送的数据包有可能到达整个互联网。

由于显而易见的原因，这是不希望的，因此 IPv4 广播的范围被显著缩小。多播也是广播的一种更好的替代方式，但它出现的时间要晚得多，因此在采用上落后了。

在 IPv6 中，多播支持已成为强制性的，并且没有明确的广播概念。多播已经得到扩展和改进，因此所有广播功能现在都可以通过某种形式的多播来实现。

在 IPv6 中，地址最左边的位用于确定其类型。对于组播地址，前 8 位都是 1，即 FF00::/8。此外，位 113-116 表示地址的范围，它可以是以下 4 个中的任何一个:全局、站点本地、链路本地、节点本地。

除了单播和多播之外，IPv6 还支持任播，即数据包可以发送给组中的任何成员，但不需要发送给所有成员。

## 6。总结

在本文中，我们探讨了使用 UDP 协议的一对多和一对多通信类型的概念。我们看到了如何用 Java 实现这些概念的例子。

最后，我们还探讨了 IPv4 和 IPv6 支持。

Github 上的[提供了完整的示例代码。](https://web.archive.org/web/20220709122341/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)