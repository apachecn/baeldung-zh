# Java RMI 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-rmi>

## 1。概述

当两个 JVM 需要通信时，Java RMI 是我们必须实现的一个选项。在本文中，我们将引导一个简单的例子来展示 Java RMI 技术。

## 2。创建服务器

创建 RMI 服务器需要两个步骤:

1.  创建一个定义客户机/服务器契约的接口。
2.  创建该接口的实现。

### 2.1。定义合同

首先，让我们为远程对象创建接口。该接口扩展了`java.rmi.Remote` 标记接口。

另外，接口中声明的每个方法都会抛出`java.rmi.` `RemoteException`:

```java
public interface MessengerService extends Remote {
    String sendMessage(String clientMessage) throws RemoteException;
}
```

不过，请注意，RMI 支持方法签名的完整 Java 规范，只要 Java 类型实现了`java.io.Serializabl` `e`。

我们将在以后的章节中看到客户机和服务器将如何使用这个接口。

对于服务器，我们将创建实现，通常称为`Remote Object`。

对于客户端，**RMI 库将动态创建一个名为`Stub`的实现。**

### 2.2。实施

此外，让我们实现远程接口，也称为`Remote Object`:

```java
public class MessengerServiceImpl implements MessengerService { 

    @Override 
    public String sendMessage(String clientMessage) { 
        return "Client Message".equals(clientMessage) ? "Server Message" : null;
    }

    public String unexposedMethod() { /* code */ }
}
```

注意，我们在方法定义中省略了`throws` `RemoteException`子句。

我们的远程对象抛出一个`RemoteException`是不常见的，因为这个异常通常是为 RMI 库保留的，以便向客户端发出通信错误。

省略它还有一个好处，就是使我们的实现与 RMI 无关。

此外，在远程对象中定义的任何额外的方法，而不是在接口中定义的，对于客户端来说是不可见的。

## 3。注册服务

一旦我们创建了远程实现，我们需要将远程对象绑定到 RMI 注册中心。

### 3.1。创建存根

首先，我们需要创建远程对象的存根:

```java
MessengerService server = new MessengerServiceImpl();
MessengerService stub = (MessengerService) UnicastRemoteObject
  .exportObject((MessengerService) server, 0);
```

我们使用静态的`UnicastRemoteObject.exportObject`方法来创建我们的存根实现。**存根就是通过底层 RMI 协议与服务器通信的魔力所在。**

`exportObject`的第一个参数是远程服务器对象。

第二个参数是端口，`exportObject`用于将远程对象导出到注册表。

给一个零值表示我们不关心哪个端口`exportObject` 被使用，这是典型的，因此是动态选择的。

**不幸的是，不带端口号的`exportObject()`方法已被弃用。**

### 3.2。创建注册表

我们可以在服务器本地建立一个注册中心，或者作为一个独立的服务。

为简单起见，我们将创建一个本地服务器:

```java
Registry registry = LocateRegistry.createRegistry(1099);
```

这将创建一个注册表，存根可以被服务器绑定到该注册表，并被客户端发现。

此外，我们使用了`createRegistry` 方法，因为我们在服务器本地创建注册表。

默认情况下，RMI 注册表在端口 1099 上运行。相反，也可以在`createRegistry` 工厂方法中指定不同的端口。

但是在独立的情况下，我们会调用`getRegistry`，将主机名和端口号作为参数传递。

### 3.3。装订存根

因此，让我们将存根绑定到注册表。RMI 注册表是一个命名工具，像 JNDI 等。我们可以遵循类似的模式，将存根绑定到一个唯一的键:

```java
registry.rebind("MessengerService", stub); 
```

因此，远程对象现在对任何能够找到注册表的客户机都是可用的。

## 4。创建客户端

最后，让我们编写客户端来调用远程方法。

为此，我们将首先定位 RMI 注册表。此外，我们将使用有界惟一键来查找远程对象存根。

最后，我们将调用`sendMessage` 方法:

```java
Registry registry = LocateRegistry.getRegistry();
MessengerService server = (MessengerService) registry
  .lookup("MessengerService");
String responseMessage = server.sendMessage("Client Message");
String expectedMessage = "Server Message";

assertEquals(expectedMessage, responseMessage);
```

因为我们在本地机器和默认端口 1099 上运行 RMI 注册表，所以我们不向`getRegistry`传递任何参数。

实际上，如果注册表位于不同的主机或不同的端口上，我们可以提供这些参数。

一旦我们使用注册表查找存根对象，我们就可以调用远程服务器上的方法。

## 5。结论

在本教程中，我们简要介绍了 Java RMI，以及它如何成为客户机-服务器应用程序的基础。请继续关注关于 RMI 一些独特特性的其他帖子！

这个教程的源代码可以在 GitHub 的[上面找到。](https://web.archive.org/web/20221128051459/https://github.com/eugenp/tutorials/tree/master/java-rmi)