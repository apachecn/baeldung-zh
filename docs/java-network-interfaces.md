# 在 Java 中使用网络接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-network-interfaces>

## 1。概述

在本文中，我们将关注网络接口以及如何在 Java 中以编程方式访问它们。

简单地说，**网络接口是设备与其任何网络连接**之间的互连点。

在日常语言中，我们称它们为网络接口卡(NIC)，但它们不一定都是硬件形式。

例如，我们在测试 web 和网络应用时经常使用的流行的 localhost IP `127.0.0.1`是环回接口——它不是一个直接的硬件接口。

当然，系统通常有多个活跃的网络连接，如有线以太网、WIFI、蓝牙等。

在 Java 中，我们可以用来与它们直接交互的主要 API 是`java.net.NetworkInterface`类。因此，为了快速开始，让我们导入完整的包:

```java
import java.net.*;
```

## 2。为什么要访问网络接口？

大多数 Java 程序可能不会与它们直接交互；然而，在一些特殊情况下，我们确实需要这种低级别的访问。

其中最突出的是一个系统有多个卡，你想有**的自由选择一个特定的接口使用一个插座与**。在这种情况下，我们通常知道名称，但不一定知道 IP 地址。

通常，当我们想要对特定的服务器地址进行套接字连接时:

```java
Socket socket = new Socket();
socket.connect(new InetSocketAddress(address, port));
```

这样，系统将选择一个合适的本地地址，绑定到它，并通过它的网络接口与服务器通信。然而，这种方法不允许我们自己选择。

这里我们将做一个假设；我们不知道地址，但我们知道名字。出于演示的目的，让我们假设我们希望通过环回接口进行连接，按照惯例，它的名称是`lo`，至少在 Linux 和 Windows 系统上是这样，在 OSX 上是`lo0`:

```java
NetworkInterface nif = NetworkInterface.getByName("lo");
Enumeration<InetAddress> nifAddresses = nif.getInetAddresses();

Socket socket = new Socket();
socket.bind(new InetSocketAddress(nifAddresses.nextElement(), 0));
socket.connect(new InetSocketAddress(address, port));
```

所以我们首先检索连接到`lo` 的网络接口，检索连接到它的地址，创建一个套接字，将它绑定到任何一个我们在编译时甚至不知道的枚举地址，然后连接。

一个`NetworkInterface`对象包含一个名称和一组分配给它的 IP 地址。因此绑定到这些地址中的任何一个都将保证通过这个接口的通信。

这并没有真正说明 API 有什么特别之处。我们知道，如果我们希望本地地址是 localhost，只要添加绑定代码，第一个代码片段就足够了。

此外，由于 localhost 有一个众所周知的地址，`127.0.0.1` ,我们不必真的经历所有这几个步骤，我们可以轻松地将套接字绑定到它。

然而，在您的例子中，`lo`可能代表其他接口，如蓝牙–`net1`、无线网络–`net0`或以太网–`eth0`。在这种情况下，您在编译时不知道 IP 地址。

## 3。正在检索网络接口

在这一节中，我们将探索用于检索可用接口的其他可用 API。在上一节中，我们只看到了其中一种方法；`getByName()`静态方法。

值得注意的是,`NetworkInterface`类没有任何公共构造函数，所以我们当然不能创建新的实例。相反，我们将使用可用的 API 来检索一个。

到目前为止，我们看到的 API 用于通过指定的名称搜索网络接口:

```java
@Test
public void givenName_whenReturnsNetworkInterface_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");

    assertNotNull(nif);
}
```

如果没有名称，则返回`null`:

```java
@Test
public void givenInExistentName_whenReturnsNull_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("inexistent_name");

    assertNull(nif);
}
```

第二个 API 是`getByInetAddress()`，它也要求我们提供一个已知的参数，这次我们可以提供 IP 地址:

```java
@Test
public void givenIP_whenReturnsNetworkInterface_thenCorrect() {
    byte[] ip = new byte[] { 127, 0, 0, 1 };

    NetworkInterface nif = NetworkInterface.getByInetAddress(
      InetAddress.getByAddress(ip));

    assertNotNull(nif);
}
```

或主机名称:

```java
@Test
public void givenHostName_whenReturnsNetworkInterface_thenCorrect()  {
    NetworkInterface nif = NetworkInterface.getByInetAddress(
      InetAddress.getByName("localhost"));

    assertNotNull(nif);
}
```

或者，如果您特别关注本地主机:

```java
@Test
public void givenLocalHost_whenReturnsNetworkInterface_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByInetAddress(
      InetAddress.getLocalHost());

    assertNotNull(nif);
}
```

另一种方法也是显式使用环回接口:

```java
@Test
public void givenLoopBack_whenReturnsNetworkInterface_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByInetAddress(
      InetAddress.getLoopbackAddress());

    assertNotNull(nif);
}
```

第三种方法是通过索引获取网络接口，这是从 Java 7 开始才有的:

```java
NetworkInterface nif = NetworkInterface.getByIndex(int index);
```

最后一种方法涉及到使用`getNetworkInterfaces` API。它返回系统中所有可用网络接口的`Enumeration`。我们需要在一个循环中检索返回的对象，标准的习语使用一个`List`:

```java
Enumeration<NetworkInterface> nets = NetworkInterface.getNetworkInterfaces();

for (NetworkInterface nif: Collections.list(nets)) {
    //do something with the network interface
}
```

## 4。网络接口参数

在检索其对象后，我们可以从其中获得许多有价值的信息。其中最有用的是分配给它的 IP 地址列表。

我们可以使用两个 API 来获取 IP 地址。第一个 API 是`getInetAddresses()`。它返回一个`InetAddress`实例的`Enumeration`，我们可以按照自己认为合适的方式进行处理:

```java
@Test
public void givenInterface_whenReturnsInetAddresses_thenCorrect()  {
    NetworkInterface nif = NetworkInterface.getByName("lo");
    Enumeration<InetAddress> addressEnum = nif.getInetAddresses();
    InetAddress address = addressEnum.nextElement();

    assertEquals("127.0.0.1", address.getHostAddress());
}
```

第二个 API 是`getInterfaceAddresses()`。它返回比`InetAddress`实例更强大的`InterfaceAddress`实例的`List`。例如，除了 IP 地址之外，您可能还对广播地址感兴趣:

```java
@Test
public void givenInterface_whenReturnsInterfaceAddresses_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");
    List<InterfaceAddress> addressEnum = nif.getInterfaceAddresses();
    InterfaceAddress address = addressEnum.get(0);

    InetAddress localAddress=address.getAddress();
    InetAddress broadCastAddress = address.getBroadcast();

    assertEquals("127.0.0.1", localAddress.getHostAddress());
    assertEquals("127.255.255.255",broadCastAddress.getHostAddress());
}
```

除了分配给接口的名称和 IP 地址之外，我们还可以访问有关接口的网络参数。要检查它是否启动并运行:

```java
@Test
public void givenInterface_whenChecksIfUp_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");

    assertTrue(nif.isUp());
}
```

要检查它是否是环回接口:

```java
@Test
public void givenInterface_whenChecksIfLoopback_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");

    assertTrue(nif.isLoopback());
}
```

要检查它是否代表点对点网络连接:

```java
@Test
public void givenInterface_whenChecksIfPointToPoint_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");

    assertFalse(nif.isPointToPoint());
}
```

或者如果是虚拟界面:

```java
@Test
public void givenInterface_whenChecksIfVirtual_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");
    assertFalse(nif.isVirtual());
}
```

要检查是否支持多播，请执行以下操作:

```java
@Test
public void givenInterface_whenChecksMulticastSupport_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");

    assertTrue(nif.supportsMulticast());
}
```

或者检索其物理地址，通常称为 MAC 地址:

```java
@Test
public void givenInterface_whenGetsMacAddress_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("lo");
    byte[] bytes = nif.getHardwareAddress();

    assertNotNull(bytes);
}
```

另一个参数是最大传输单位，它定义了可以通过该接口传输的最大数据包大小:

```java
@Test
public void givenInterface_whenGetsMTU_thenCorrect() {
    NetworkInterface nif = NetworkInterface.getByName("net0");
    int mtu = nif.getMTU();

    assertEquals(1500, mtu);
}
```

## 5。结论

在本文中，我们展示了网络接口，如何通过编程访问它们，以及为什么我们需要访问它们。

本文中使用的完整源代码和示例可以在 [Github 项目](https://web.archive.org/web/20220124000203/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)中获得。