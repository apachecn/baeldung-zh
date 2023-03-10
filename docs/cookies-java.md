# Java HTTP cookie 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cookies-java>

## 1。概述

在本文中，我们将探讨 Java 网络编程的底层操作。我们将更深入地了解饼干。

Java 平台附带内置的网络支持，打包在`java.net`包中:

```java
import java.net.*;
```

## 2。HTTP cookie

每当客户端向服务器发送 HTTP 请求并收到响应时，服务器就会忘记这个客户端。下次客户端再次请求时，它将被视为一个全新的客户端。

然而，正如我们所知，cookies 使得在客户机和服务器之间建立会话成为可能，这样服务器就可以在多个请求响应对之间记住客户机。

从这一节开始，我们将学习如何在 Java 网络编程中使用 cookies 来增强我们的客户机-服务器通信。

`java.net`包中用于处理 cookies 的主类是`CookieHandler`。还有其他的帮助类和接口，如`CookieManager`、`CookiePolicy`、`CookieStore`和`HttpCookie`。

## 3。`CookieHandler`班

考虑这个场景；我们正在与位于`http://baeldung.com`的服务器或任何其他使用 HTTP 协议的 URL 进行通信，URL 对象将使用一个名为 HTTP 协议处理器的引擎。

这个 HTTP 协议处理程序检查系统中是否有默认的`CookieHandler`实例。如果有，它调用 it 来负责状态管理。

所以`CookieHandler`类的目的是提供一种回调机制，以利于 HTTP 协议处理程序。

`CookieHandler`是一个抽象类。它有一个静态的`getDefault()`方法，可以被调用来检索当前的
`CookieHandler`安装，或者我们可以调用`setDefault(CookieHandler)`来设置我们自己的安装。注意，调用`setDefault`会在系统范围内安装一个`CookieHandler`对象。

它还有`put(uri, responseHeaders)`用于将任何 cookie 保存到 cookie store。这些 cookies 是从给定 URI 的 HTTP 响应的响应头中检索的。每次收到响应时都会调用它。

一个相关的 API 方法—`get(uri,requestHeaders)`检索保存在给定 URI 下的 cookies，并将它们添加到`requetHeaders`。它在请求发出之前被调用。

这些方法都必须在一个具体的类`CookieHandler`中实现。在这一点上，`CookieManager`级值得我们关注。这个类为大多数常见用例提供了一个`CookieHandler`类的完整实现。

在接下来的两节中，我们将探索`CookieManager`类；首先是默认模式，然后是自定义模式。

## 4。`CookieManager`默认

为了有一个完整的 cookie 管理框架，我们需要有`CookiePolicy`和`CookieStore`的实现。

`CookiePolicy` 建立接受和拒绝 cookies 的规则。我们当然可以改变这些规则来适应我们的需要。

next-`CookieStore`顾名思义，它有保存和检索 cookies 的方法。当然，如果需要，我们也可以调整这里的存储机制。

我们先来看看默认值。创建默认值`CookieHandler`并将其设置为系统范围的默认值:

```java
CookieManager cm = new CookieManager();
CookieHandler.setDefault(cm);
```

我们应该注意到，默认的`CookieStore`将会有易失性内存，也就是说，它只存在于 JVM 的生命周期中。为了有一个更持久的 cookies 存储，我们必须定制它。

说到`CookiePolicy`，默认实现是`CookiePolicy.ACCEPT_ORIGINAL_SERVER`。这意味着如果响应是通过代理服务器接收的，那么 cookie 将被拒绝。

## 5。`CookieManager` 一种风俗

现在让我们通过提供我们自己的`CookiePolicy`或`CookieStore`(或者两者)的实例来定制默认的`CookieManager`。

### 5.1。`CookiePolicy`

`CookiePolicy`为了方便起见，提供了一些预定义的实现:

*   `CookiePolicy.ACCEPT_ORIGINAL_SERVER`–只能保存来自原始服务器的 cookies(默认实现)
*   `CookiePolicy.ACCEPT_ALL`–所有的 cookies 都可以保存，不管它们来自哪里
*   `CookiePolicy.ACCEPT_NONE`–不能保存任何 cookies

为了简单地改变当前的`CookiePolicy`而不实现我们自己的，我们在`CookieManager`实例上调用`setCookiePolicy`:

```java
CookieManager cm=new CookieManager();
cm.setCookiePolicy(CookiePolicy.ACCEPT_ALL);
```

但是我们可以做更多的定制。知道了`CookiePolicy.ACCEPT_ORIGINAL_SERVER`的行为，让我们假设我们信任一个特定的代理服务器，并且想要在原始服务器上接受来自它的 cookies。

我们必须实现`CookiePolicy`接口和`shouldAccept`方法；在这里，我们将通过添加所选代理服务器的域名来更改接受规则。

姑且称之为新政策——`ProxyAcceptCookiePolicy`。除了给定的代理地址之外，它基本上会拒绝来自其`shouldAccept`实现的任何其他代理服务器，然后调用`CookiePolicy.ACCEPT_ORIGINAL_SERVER`的`shouldAccept`方法来完成实现:

```java
public class ProxyAcceptCookiePolicy implements CookiePolicy {
    private String acceptedProxy;

    public boolean shouldAccept(URI uri, HttpCookie cookie) {
        String host = InetAddress.getByName(uri.getHost())
          .getCanonicalHostName();
        if (HttpCookie.domainMatches(acceptedProxy, host)) {
            return true;
        }

        return CookiePolicy.ACCEPT_ORIGINAL_SERVER
          .shouldAccept(uri, cookie);
    }

    // standard constructors
}
```

当我们创建一个`ProxyAcceptCookiePolicy`的实例时，除了原始服务器之外，我们还会传入一个我们想要接受 cookies 的域地址的字符串。

然后，我们将这个实例设置为`CookieManager`实例的 cookie 策略，然后将其设置为默认的 CookieHandler:

```java
CookieManager cm = new CookieManager();
cm.setCookiePolicy(new ProxyAcceptCookiePolicy("baeldung.com"));
CookieHandler.setDefault(cm);
```

这样，cookie 处理器将接受来自原始服务器的所有 cookie 以及来自`http://www.baeldung.com`的 cookie。

### 5.2。`CookieStore`

`CookieManager`为每个 HTTP 响应将 cookie 添加到`CookieStore`中，并为每个 HTTP 请求从`CookieStore`中检索 cookie。

默认的`CookieStore`实现没有持久性，当 JVM 重启时，它会丢失所有的数据。更像是电脑里的内存。

因此，如果我们希望我们的`CookieStore`实现像硬盘一样运行，并在 JVM 重启时保留 cookies，我们必须定制它的存储和检索机制。

需要注意的一点是，我们不能在创建后将一个`CookieStore`实例传递给`CookieManager`。我们唯一的选择是在创建`CookieManager`时传递它，或者通过调用 new `CookieManager().getCookieStore()`并补充它的行为来获得对默认实例的引用。

下面是`PersistentCookieStore`的实现:

```java
public class PersistentCookieStore implements CookieStore, Runnable {
    private CookieStore store;

    public PersistentCookieStore() {
        store = new CookieManager().getCookieStore();
        // deserialize cookies into store
        Runtime.getRuntime().addShutdownHook(new Thread(this));
    }

    @Override
    public void run() {
        // serialize cookies to persistent storage
    }

    @Override
    public void add(URI uri, HttpCookie cookie) {
        store.add(uri, cookie);

    }

    // delegate all implementations to store object like above
}
```

请注意，我们在构造函数中检索了对默认实现的引用。

我们实现了`runnable`,这样我们就可以添加一个在 JVM 关闭时运行的关闭挂钩。在`run`方法中，我们将所有的 cookies 保存在内存中。

我们可以将数据序列化到一个文件或任何合适的存储中。还要注意，在构造函数内部，我们首先将所有 cookies 从持久内存读入`CookieStore`。这两个简单的特性使得默认的`CookieStore`本质上是持久的(以一种简单的方式)。

## 6。结论

在本教程中，我们介绍了 HTTP cookies，并展示了如何以编程方式访问和操作它们。

文章的完整源代码和所有代码片段可以在 [GitHub 项目](https://web.archive.org/web/20220124014623/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)中找到。