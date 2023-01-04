# Apache HttpClient 连接管理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-connection-management>

## 1。概述

在本文中，我们将介绍 http client 4.0 中连接管理的基础知识。

我们将讨论如何使用`BasichttpClientConnectionManager`和`PoolingHttpClientConnectionManager`来加强 HTTP 连接的安全、符合协议和高效的使用。

## 2。`BasicHttpClientConnectionManager` 为低水平，单螺纹连接

从 HttpClient 4.3.3 开始，`BasicHttpClientConnectionManager`就是 HTTP 连接管理器最简单的实现。它用于创建和管理一次只能由一个线程使用的单个连接。

## 延伸阅读:

## [高级 Apache HttpClient 配置](/web/20220630140247/https://www.baeldung.com/httpclient-advanced-config)

高级用例的 HttpClient 配置。[阅读更多](/web/20220630140247/https://www.baeldung.com/httpclient-advanced-config)→

## [Apache HttpClient–发送自定义 Cookie](/web/20220630140247/https://www.baeldung.com/httpclient-cookies)

如何用 Apache http client 发送自定义 Cookie。[阅读更多](/web/20220630140247/https://www.baeldung.com/httpclient-cookies)→

## [Apache HttpClient with SSL](/web/20220630140247/https://www.baeldung.com/httpclient-ssl)

如何配置 http client with SSL 的例子。[阅读更多](/web/20220630140247/https://www.baeldung.com/httpclient-ssl) →
例题 2.1。**获取低层连接的连接请求(`HttpClientConnection` )**

```
BasicHttpClientConnectionManager connManager
 = new BasicHttpClientConnectionManager();
HttpRoute route = new HttpRoute(new HttpHost("www.baeldung.com", 80));
ConnectionRequest connRequest = connManager.requestConnection(route, null);
```

`requestConnection`方法从管理器获得一个连接池，供特定的`route`连接。`route`参数指定到目标主机或目标主机本身的“代理跳”路由。

直接使用`HttpClientConnection`执行请求是可能的，但是请记住，这种低级的方法冗长且难以管理。低级连接对于访问套接字和连接数据(比如超时和目标主机信息)很有用，但是对于标准执行来说，`HttpClient`是一个更容易使用的 API。

## 3。使用`PoolingHttpClientConnectionManager`获取和管理多线程连接池

`PoolingHttpClientConnectionManager` 将为我们使用的每个路由或目标主机创建和管理一个连接池。管理器可以打开的并发**连接**池的默认大小是每个路由或目标主机的 **2，总**打开连接的 **20。首先，让我们看看如何在一个简单的 HttpClient 上设置这个连接管理器:**

例 3.1。**在 HttpClient 上设置 PoolingHttpClientConnectionManager**

```
HttpClientConnectionManager poolingConnManager
  = new PoolingHttpClientConnectionManager();
CloseableHttpClient client
 = HttpClients.custom().setConnectionManager(poolingConnManager)
 .build();
client.execute(new HttpGet("/"));
assertTrue(poolingConnManager.getTotalStats().getLeased() == 1);
```

接下来，让我们看看在两个不同线程中运行的两个 HttpClients 如何使用同一个连接管理器:

例 3.2。**使用两个 HttpClients 分别连接到一个目标主机**

```
HttpGet get1 = new HttpGet("/");
HttpGet get2 = new HttpGet("http://google.com"); 
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager(); 
CloseableHttpClient client1 
  = HttpClients.custom().setConnectionManager(connManager).build();
CloseableHttpClient client2 
  = HttpClients.custom().setConnectionManager(connManager).build();

MultiHttpClientConnThread thread1
 = new MultiHttpClientConnThread(client1, get1); 
MultiHttpClientConnThread thread2
 = new MultiHttpClientConnThread(client2, get2); 
thread1.start();
thread2.start();
thread1.join();
thread2.join();
```

请注意，我们正在使用**一个非常简单的定制线程实现**——如下所示:

例 3.3。**自定义线程`Executing a` 获取请求**

```
public class MultiHttpClientConnThread extends Thread {
    private CloseableHttpClient client;
    private HttpGet get;

    // standard constructors
    public void run(){
        try {
            HttpResponse response = client.execute(get);  
            EntityUtils.consume(response.getEntity());
        } catch (ClientProtocolException ex) {    
        } catch (IOException ex) {
        }
    }
}
```

请注意 `**EntityUtils.consume(response.getEntity)**`调用——这是消费响应(实体)的全部内容所必需的，以便管理器能够**释放返回到池**的连接。

## 4。配置连接管理器

池连接管理器的缺省值选择得很好，但是——取决于您的用例——可能太小。那么，让我们来看看如何配置:

*   连接的总数
*   每个(任何)路由的最大连接数
*   每条特定路由的最大连接数

例 4.1。**增加可以打开和管理的连接数，使其超过默认限制**

```
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
connManager.setMaxTotal(5);
connManager.setDefaultMaxPerRoute(4);
HttpHost host = new HttpHost("www.baeldung.com", 80);
connManager.setMaxPerRoute(new HttpRoute(host), 5);
```

让我们回顾一下 API:

*   `setMaxTotal(int max)`:设置总打开连接数的最大值。
*   `setDefaultMaxPerRoute(int max)`:设置每条路由的最大并发连接数，默认为 2。
*   `setMaxPerRoute(int max)`:设置特定路由的并发连接总数，默认为 2。

因此，在不改变默认设置的情况下，**我们将很容易达到连接管理器**的极限——让我们看看是什么样子:

例 4.2。**使用线程执行连接**

```
HttpGet get = new HttpGet("http://www.baeldung.com");
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom().
    setConnectionManager(connManager).build();
MultiHttpClientConnThread thread1 
  = new MultiHttpClientConnThread(client, get);
MultiHttpClientConnThread thread2 
  = new MultiHttpClientConnThread(client, get);
MultiHttpClientConnThread thread3 
  = new MultiHttpClientConnThread(client, get);
thread1.start();
thread2.start();
thread3.start();
thread1.join();
thread2.join();
thread3.join();
```

正如我们已经讨论过的，默认情况下，**每个主机的连接限制是 2** 。因此，在这个例子中，我们试图让 3 个线程向同一个主机发出 **3 个请求，但是只有 2 个连接被并行分配。**

让我们看一下日志——我们有三个线程在运行，但只有两个租用的连接:

```
[Thread-0] INFO  o.b.h.c.MultiHttpClientConnThread
 - Before - Leased Connections = 0
[Thread-1] INFO  o.b.h.c.MultiHttpClientConnThread
 - Before - Leased Connections = 0
[Thread-2] INFO  o.b.h.c.MultiHttpClientConnThread
 - Before - Leased Connections = 0
[Thread-2] INFO  o.b.h.c.MultiHttpClientConnThread
 - After - Leased Connections = 2
[Thread-0] INFO  o.b.h.c.MultiHttpClientConnThread
 - After - Leased Connections = 2
```

## 5。连接保持活动策略

引用 HttpClient 4.3.3。引用:"如果 `Keep-Alive` 报头没有出现在响应中，`HttpClient`假定连接可以无限期地保持活动。"([参见 HttpClient 引用](https://web.archive.org/web/20220630140247/https://hc.apache.org/httpcomponents-client-4.5.x/current/tutorial/pdf/httpclient-tutorial.pdf))。

为了解决这个问题，并能够管理死连接，我们需要一个定制的策略实现，并将其构建到 `HttpClient`中。

例 5.1。**自定义保活策略**

```
ConnectionKeepAliveStrategy myStrategy = new ConnectionKeepAliveStrategy() {
    @Override
    public long getKeepAliveDuration(HttpResponse response, HttpContext context) {
        HeaderElementIterator it = new BasicHeaderElementIterator
            (response.headerIterator(HTTP.CONN_KEEP_ALIVE));
        while (it.hasNext()) {
            HeaderElement he = it.nextElement();
            String param = he.getName();
            String value = he.getValue();
            if (value != null && param.equalsIgnoreCase
               ("timeout")) {
                return Long.parseLong(value) * 1000;
            }
        }
        return 5 * 1000;
    }
};
```

该策略将首先尝试应用报头中声明的主机的`Keep-Alive`策略。如果响应报头中不存在该信息，它将保持活动连接 5 秒钟。

现在，让我们用这个定制策略创建一个客户端**:**

```
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom()
  .setKeepAliveStrategy(myStrategy)
  .setConnectionManager(connManager)
  .build();
```

## 6。连接持久性/重用

HTTP/1.1 规范规定，如果连接没有被关闭，就可以被重用，这就是所谓的连接持久性。

一旦管理器释放了一个连接，它就保持打开状态以供重用。当使用只能管理单个连接的`BasicHttpClientConnectionManager,`时，连接必须在被再次租回之前释放:

例 6.1。 **`BasicHttpClientConnectionManager`** **连接重用**

```
BasicHttpClientConnectionManager basicConnManager = 
    new BasicHttpClientConnectionManager();
HttpClientContext context = HttpClientContext.create();

// low level
HttpRoute route = new HttpRoute(new HttpHost("www.baeldung.com", 80));
ConnectionRequest connRequest = basicConnManager.requestConnection(route, null);
HttpClientConnection conn = connRequest.get(10, TimeUnit.SECONDS);
basicConnManager.connect(conn, route, 1000, context);
basicConnManager.routeComplete(conn, route, context);

HttpRequestExecutor exeRequest = new HttpRequestExecutor();
context.setTargetHost((new HttpHost("www.baeldung.com", 80)));
HttpGet get = new HttpGet("http://www.baeldung.com");
exeRequest.execute(get, conn, context);

basicConnManager.releaseConnection(conn, null, 1, TimeUnit.SECONDS);

// high level
CloseableHttpClient client = HttpClients.custom()
  .setConnectionManager(basicConnManager)
  .build();
client.execute(get);
```

让我们来看看会发生什么。

首先——注意，我们首先使用的是一个低级连接，这样我们就可以完全控制何时释放连接，然后是一个普通的带有 HttpClient 的高级连接。复杂的底层逻辑在这里不太相关——我们唯一关心的是`releaseConnection`调用。这将释放唯一可用的连接，并允许它被重用。

然后，客户端再次成功执行 GET 请求。如果我们跳过释放连接，我们将从 HttpClient 得到一个 IllegalStateException:

```
java.lang.IllegalStateException: Connection is still allocated
  at o.a.h.u.Asserts.check(Asserts.java:34)
  at o.a.h.i.c.BasicHttpClientConnectionManager.getConnection
    (BasicHttpClientConnectionManager.java:248)
```

注意，现有的连接并没有关闭，只是被释放，然后被第二个请求重用。

与上面的例子相反，`PoolingHttpClientConnectionManager`允许透明地重用连接，而不需要隐式地释放连接:

例 6.2。 ****`PoolingHttpClientConnectionManager` :** 重用线程连接**

```
HttpGet get = new HttpGet("http://echo.200please.com");
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
connManager.setDefaultMaxPerRoute(5);
connManager.setMaxTotal(5);
CloseableHttpClient client = HttpClients.custom()
  .setConnectionManager(connManager)
  .build();
MultiHttpClientConnThread[] threads 
  = new  MultiHttpClientConnThread[10];
for(int i = 0; i < threads.length; i++){
    threads[i] = new MultiHttpClientConnThread(client, get, connManager);
}
for (MultiHttpClientConnThread thread: threads) {
     thread.start();
}
for (MultiHttpClientConnThread thread: threads) {
     thread.join(1000);     
}
```

上面的例子有 10 个线程，执行 10 个请求，但只共享 5 个连接。

当然，这个例子依赖于服务器的`Keep-Alive`超时。为确保连接不会在重新使用前失效，建议使用`Keep-Alive`策略配置`client`(参见示例 5.1)。).

## 7 .**。配置超时–使用连接管理器的套接字超时**

在配置连接管理器时，唯一可以设置的超时是套接字超时:

例 7.1。**将套接字超时设置为 5 秒**

```
HttpRoute route = new HttpRoute(new HttpHost("www.baeldung.com", 80));
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
connManager.setSocketConfig(route.getTargetHost(),SocketConfig.custom().
    setSoTimeout(5000).build());
```

关于 HttpClient 中超时的更深入的讨论–[参见这个](/web/20220630140247/https://www.baeldung.com/httpclient-timeout "HttpClient Timeout")。

## 8。连接驱逐

连接驱逐用于**检测空闲和过期的连接并关闭它们**；有两种方法可以做到这一点。

1.  依靠`HttpClien` t 在执行请求之前检查连接是否失效。这是一个昂贵的选择，并不总是可靠的。
2.  创建一个监视器线程来关闭空闲和/或关闭的连接。

例 8.1。**设置`HttpClient`以检查陈旧连接**

```
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom().setDefaultRequestConfig(
    RequestConfig.custom().setStaleConnectionCheckEnabled(true).build()
).setConnectionManager(connManager).build();
```

例 8.2。**使用过时的连接监视器线程**

```
PoolingHttpClientConnectionManager connManager 
  = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom()
  .setConnectionManager(connManager).build();
IdleConnectionMonitorThread staleMonitor
 = new IdleConnectionMonitorThread(connManager);
staleMonitor.start();
staleMonitor.join(1000);
```

下面列出了 **`IdleConnectionMonitorThread`** 类:

```
public class IdleConnectionMonitorThread extends Thread {
    private final HttpClientConnectionManager connMgr;
    private volatile boolean shutdown;

    public IdleConnectionMonitorThread(
      PoolingHttpClientConnectionManager connMgr) {
        super();
        this.connMgr = connMgr;
    }
    @Override
    public void run() {
        try {
            while (!shutdown) {
                synchronized (this) {
                    wait(1000);
                    connMgr.closeExpiredConnections();
                    connMgr.closeIdleConnections(30, TimeUnit.SECONDS);
                }
            }
        } catch (InterruptedException ex) {
            shutdown();
        }
    }
    public void shutdown() {
        shutdown = true;
        synchronized (this) {
            notifyAll();
        }
    }
}
```

## 9。连接关闭

通过调用`shutdown`方法，连接可以正常关闭(尝试在关闭前刷新输出缓冲区)，也可以强制关闭(输出缓冲区不被刷新)。

要正确关闭连接，我们需要完成以下所有工作:

*   消耗并关闭响应(如果可关闭)

*   关闭客户端

*   关闭连接管理器

例 9.1。**关闭连接释放资源**

```
connManager = new PoolingHttpClientConnectionManager();
CloseableHttpClient client = HttpClients.custom()
  .setConnectionManager(connManager).build();
HttpGet get = new HttpGet("http://google.com");
CloseableHttpResponse response = client.execute(get);

EntityUtils.consume(response.getEntity());
response.close();
client.close();
connManager.close(); 
```

如果管理器在没有关闭连接的情况下关闭，所有连接都将被关闭，所有资源都将被释放。

请务必记住，这不会刷新现有连接可能正在进行的任何数据。

## 10。结论

在本文中，我们讨论了如何使用 HttpClient 的 HTTP 连接管理 API 来处理管理连接的整个过程——从打开和分配连接，通过管理多个代理对它们的并发使用，到最终关闭它们。

我们看到了`BasicHttpClientConnectionManager`如何成为处理单个连接的简单解决方案，以及它如何管理低级连接。我们还看到了如何将`PoolingHttpClientConnectionManager`与`HttpClient` API 结合起来，以提供高效且符合协议的 HTTP 连接使用。

本文中使用的代码可以在我们的 Github 上找到[。](https://web.archive.org/web/20220630140247/https://github.com/eugenp/tutorials/tree/master/apache-httpclient)**