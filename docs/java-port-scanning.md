# 用 Java 进行端口扫描

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-port-scanning>

## 1.概观

用 Java 进行端口扫描是一种枚举目标机器的开放或活动端口的方法。目标主要是列出开放的端口，以便了解当前正在运行的应用程序和服务。

在本教程中，**我们将解释如何用 Java** 开发一个简单的端口扫描应用程序，我们可以用它来扫描主机的开放端口。

## 2.什么是计算机端口？

计算机端口是一个逻辑实体，它可以将特定的服务与连接相关联。此外，端口由从 1 到 65535 的整数标识。按照惯例，前 1024 个是为标准服务保留的，例如:

*   端口 20: FTP
*   端口 23: Telnet
*   端口 25: SMTP
*   端口 80: HTTP

端口扫描器的想法是创建一个`TCP` S `ocket`并尝试连接到一个特定的端口。如果连接成功建立，那么我们将把这个端口标记为开放，如果没有，我们将把它标记为关闭。

但是，在 65535 个端口中的每个端口上建立连接可能需要每个端口 200 毫秒。这听起来时间很短，但总的来说，**逐个扫描一台主机的所有端口将需要相当长的时间**。

为了解决性能问题，**我们将使用[多线程](/web/20221208143856/https://www.baeldung.com/cs/multithreaded-algorithms)方法**。与尝试顺序连接到每个端口相比，这可以大大加快进程。

## 3.履行

为了实现我们的程序，我们创建了一个函数`portScan()`，它有两个参数作为输入:

*   `ip`:要扫描的`IP`地址；它相当于本地主机的 127.0.0.1
*   `nbrPortMaxToScan`:扫描的最大端口数；如果我们要扫描所有端口，这个数字相当于 65535

### 3.1 实施

让我们看看我们的`portScan()`方法是什么样子的:

```java
public void runPortScan(String ip, int nbrPortMaxToScan) throws IOException {
        ConcurrentLinkedQueue openPorts = new ConcurrentLinkedQueue<>();
        ExecutorService executorService = Executors.newFixedThreadPool(poolSize);
        AtomicInteger port = new AtomicInteger(0);
        while (port.get() < nbrPortMaxToScan) {
                final int currentPort = port.getAndIncrement();
                executorService.submit(() -> {
                        try {
                                Socket socket = new Socket();
                                socket.connect(new InetSocketAddress(ip, currentPort), timeOut);
                                socket.close();
                                openPorts.add(currentPort);
                                System.out.println(ip + " ,port open: " + currentPort);
                        } catch (IOException e) {
                                System.err.println(e);
                        }
                });
        }
        executorService.shutdown();
        try {
                executorService.awaitTermination(10, TimeUnit.MINUTES);
        } catch (InterruptedException e) {
                throw new RuntimeException(e);
        }
        List openPortList = new ArrayList<>();
        System.out.println("openPortsQueue: " + openPorts.size());
        while (!openPorts.isEmpty()) {
                openPortList.add(openPorts.poll());
        }
        openPortList.forEach(p -> System.out.println("port " + p + " is open"));
}
```

我们的方法返回一个带有所有开放端口的`List`。为此，我们创建一个新的`Socket` 对象，用作两台主机之间的连接器。如果连接成功建立，那么我们假设端口是开放的，在这种情况下，我们继续下一行。另一方面，如果连接失败，那么我们假设端口关闭，抛出一个 [`SocketTimeoutException`](/web/20221208143856/https://www.baeldung.com/java-socket-connection-read-timeout) ，我们被抛出异常`catch`块。

### 3.2 多线程

为了优化扫描目标机器的所有 65535 个端口所需的时间，我们将同时运行我们的方法。我们使用 [`ExecutorService`](/web/20221208143856/https://www.baeldung.com/java-executor-service-tutorial) ，它封装了一个线程池和一个待执行任务队列。池中的所有线程仍在运行。

该服务检查队列中是否有要处理的任务，如果有，它将撤回并执行该任务。一旦任务被执行，线程再次等待服务从队列中为它分配一个新任务。

此外，我们使用带有 10 个`Thread`的`FixedThreadPool`，这意味着程序将并行运行最多 10 个线程。我们可以根据我们的机器配置和容量来调整这个池的大小。

## 4.结论

在这个快速教程中，我们解释了**如何使用`Socket` s 和多线程方法用 Java** 开发一个简单的端口扫描应用程序。

和往常一样，代码片段可以在 GitHub 上[找到。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)