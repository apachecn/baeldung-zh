# Java 和 Zookeeper 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-zookeeper>

## 1。概述

Apache ZooKeeper **是一个分布式协调服务**，它简化了分布式应用程序的开发。它被 Apache Hadoop、HBase 和 [others](https://web.archive.org/web/20221109150443/https://cwiki.apache.org/confluence/display/ZOOKEEPER/PoweredBy) 等项目用于不同的用例，如领导者选举、配置管理、节点协调、服务器租赁管理等。

ZooKeeper 集群中的节点将它们的数据存储在一个共享的层次名称空间中，该名称空间类似于一个标准文件系统或一个树形数据结构。

在本文中，我们将探索如何使用 Apache Zookeeper 的 Java API 来存储、更新和删除 Zookeeper 中存储的信息。

## 2.**设置**

Apache ZooKeeper Java 库的最新版本可以在这里找到:

```java
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
```

## 3。动物园管理员数据模型–ZNode

ZooKeeper 有一个分层的名称空间，很像一个分布式文件系统，它存储协调数据，如状态信息、协调信息、位置信息等。这些信息存储在不同的节点上。

动物园管理员树中的每个节点都被称为 ZNode。

每个 ZNode 为任何数据或 ACL 更改维护版本号和时间戳。此外，这允许 ZooKeeper 验证缓存并协调更新。

## 4。安装

### 4.1。安装

最新的 ZooKeeper 版本可以从[这里](https://web.archive.org/web/20221109150443/https://www.apache.org/dyn/closer.cgi/zookeeper/)下载。在此之前，我们需要确保我们满足这里描述的[的系统需求](https://web.archive.org/web/20221109150443/https://zookeeper.apache.org/doc/r3.3.5/zookeeperAdmin.html#sc_systemReq)。

### 4.2。独立模式

对于本文，我们将以独立模式运行 ZooKeeper，因为它需要最少的配置。在此遵循文档[中描述的步骤。](https://web.archive.org/web/20221109150443/https://zookeeper.apache.org/doc/r3.3.5/zookeeperStarted.html#sc_InstallingSingleMode)

注意:在独立模式下，没有复制，所以如果 ZooKeeper 进程失败，服务将关闭。

## 5。ZooKeeper CLI 示例

我们现在将使用 ZooKeeper 命令行界面(CLI)与 ZooKeeper 进行交互:

```java
bin/zkCli.sh -server 127.0.0.1:2181
```

上述命令在本地启动一个独立实例。现在让我们看看如何创建一个 ZNode 并在 ZooKeeper 中存储信息:

```java
[zk: localhost:2181(CONNECTED) 0] create /MyFirstZNode ZNodeVal
Created /FirstZnode
```

我们刚刚在 ZooKeeper 分层名称空间的根位置创建了一个 ZNode `‘MyFirstZNode’`,并将`‘ZNodeVal’`写入其中。

因为我们没有传递任何标志，所以创建的 ZNode 将是持久的。

现在让我们发出一个`‘get'`命令来获取数据以及与 ZNode 相关联的元数据:

```java
[zk: localhost:2181(CONNECTED) 1] get /FirstZnode

“Myfirstzookeeper-app”
cZxid = 0x7f
ctime = Sun Feb 18 16:15:47 IST 2018
mZxid = 0x7f
mtime = Sun Feb 18 16:15:47 IST 2018
pZxid = 0x7f
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 22
numChildren = 0
```

我们可以使用`set`操作更新现有`ZNode`的数据。

例如:

```java
set /MyFirstZNode ZNodeValUpdated
```

这将把`MyFirstZNode`处的数据从`ZNodeVal`更新到`ZNodeValUpdated.`

## 6。 ZooKeeper Java API 示例

现在让我们看看 Zookeeper Java API，创建一个节点，更新节点并检索一些数据。

### 6.1。Java 包

ZooKeeper Java 绑定主要由两个 Java 包组成:

1.  `org.apache.zookeeper` **:** 定义了 ZooKeeper 客户端库的主类以及 ZooKeeper 事件类型和状态的许多静态定义
2.  `org.apache.zookeeper.data`:它定义了与 ZNodes 相关的特征，比如访问控制列表(ACL)、id、stats 等等

还有 ZooKeeper Java APIs 用于服务器实现，如`org.apache.zookeeper.server`、`org.apache.zookeeper.server.quorum`和`org.apache.zookeeper.server.upgrade`。

然而，它们超出了本文的范围。

### 6.2。连接到 ZooKeeper 实例

现在让我们创建`ZKConnection`类，它将用于连接和断开已经运行的 ZooKeeper:

```java
public class ZKConnection {
    private ZooKeeper zoo;
    CountDownLatch connectionLatch = new CountDownLatch(1);

    // ...

    public ZooKeeper connect(String host) 
      throws IOException, 
      InterruptedException {
        zoo = new ZooKeeper(host, 2000, new Watcher() {
            public void process(WatchedEvent we) {
                if (we.getState() == KeeperState.SyncConnected) {
                    connectionLatch.countDown();
                }
            }
        });

        connectionLatch.await();
        return zoo;
    }

    public void close() throws InterruptedException {
        zoo.close();
    }
}
```

要使用 ZooKeeper 服务，应用程序必须首先实例化一个 **`ZooKeeper`类的对象，这是 ZooKeeper 客户端库**的主类。

在`connect`方法中，我们实例化了`ZooKeeper`类的一个实例。此外，我们还注册了一个回调方法来处理来自 ZooKeeper 的用于连接接受的`WatchedEvent`，并相应地使用`CountDownLatch`的`countdown`方法来完成`connect`方法。

一旦建立了与服务器的连接，就会为客户机分配一个会话 ID。为了保持会话有效，客户端应该定期向服务器发送心跳。

客户端应用程序可以调用 ZooKeeper APIs，只要它的会话 ID 保持有效。

### 6.3。客户端操作

我们现在将创建一个`ZKManager` 接口，它公开不同的操作，比如创建一个 ZNode 并保存一些数据，获取和更新 ZNode 数据:

```java
public interface ZKManager {
    public void create(String path, byte[] data)
      throws KeeperException, InterruptedException;
    public Object getZNodeData(String path, boolean watchFlag);
    public void update(String path, byte[] data) 
      throws KeeperException, InterruptedException;
}
```

现在让我们看看上面接口的实现:

```java
public class ZKManagerImpl implements ZKManager {
    private static ZooKeeper zkeeper;
    private static ZKConnection zkConnection;

    public ZKManagerImpl() {
        initialize();
    }

    private void initialize() {
        zkConnection = new ZKConnection();
        zkeeper = zkConnection.connect("localhost");
    }

    public void closeConnection() {
        zkConnection.close();
    }

    public void create(String path, byte[] data) 
      throws KeeperException, 
      InterruptedException {

        zkeeper.create(
          path, 
          data, 
          ZooDefs.Ids.OPEN_ACL_UNSAFE, 
          CreateMode.PERSISTENT);
    }

    public Object getZNodeData(String path, boolean watchFlag) 
      throws KeeperException, 
      InterruptedException {

        byte[] b = null;
        b = zkeeper.getData(path, null, null);
        return new String(b, "UTF-8");
    }

    public void update(String path, byte[] data) throws KeeperException, 
      InterruptedException {
        int version = zkeeper.exists(path, true).getVersion();
        zkeeper.setData(path, data, version);
    }
}
```

在上面的代码中，`connect`和`disconnect`调用被委托给先前创建的`ZKConnection`类。我们的`create`方法用于从字节数组数据在给定路径创建一个 ZNode。仅出于演示目的，我们保持 ACL 完全开放。

一旦创建，ZNode 是持久的，当客户端断开连接时不会被删除。

在我们的`getZNodeData`方法中，从 ZooKeeper 获取 ZNode 数据的逻辑非常简单。最后，使用`update`方法，我们检查给定路径上是否存在 ZNode，如果存在就获取它。

除此之外，为了更新数据，我们首先检查 ZNode 是否存在，并获取当前版本。然后，我们以 ZNode 的路径、数据和当前版本作为参数调用`setData`方法。只有当传递的版本与最新版本匹配时，ZooKeeper 才会更新数据。

## 7。结论

在开发分布式应用程序时，Apache ZooKeeper 作为一个分布式协调服务发挥着关键作用。专门用于存储共享配置、选举主节点等用例。

ZooKeeper 还提供了一个优雅的基于 Java 的 API，用于客户端应用程序代码，与 ZooKeeper ZNodes 进行无缝通信。

和往常一样，本教程的所有资料都可以在 Github 上找到[。](https://web.archive.org/web/20221109150443/https://github.com/eugenp/tutorials/tree/master/apache-libraries)