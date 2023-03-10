# 阿帕奇馆长简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-curator>

## 1。简介

Apache Curator 是 [Apache Zookeeper](https://web.archive.org/web/20220625222137/https://zookeeper.apache.org/) 的 Java 客户端，这是一个流行的分布式应用程序协调服务。

在本教程中，我们将介绍策展人提供的一些最相关的功能:

*   连接管理–管理连接和重试策略
*   异步——通过添加异步功能和使用 Java 8 lambdas 来增强现有客户端
*   配置管理–对系统进行集中配置
*   强类型模型–使用类型模型
*   配方——实现领导者选举、分布式锁或计数器

## 2。先决条件

首先，建议快速浏览一下 [Apache Zookeeper](https://web.archive.org/web/20220625222137/https://zookeeper.apache.org/) 及其特性。

对于本教程，我们假设已经有一个独立的 Zookeeper 实例在`127.0.0.1:2181`上运行；如果你刚刚入门，这里有关于如何安装和运行它的说明。

首先，我们需要将[策展人-x-异步](https://web.archive.org/web/20220625222137/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.curator%22%20AND%20a%3A%22curator-x-async%22)依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-x-async</artifactId>
    <version>4.0.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**最新版本的阿帕奇馆长 4。X.X 对 Zookeeper 3.5.X** 有很强的依赖性，该版本目前仍处于测试阶段。

因此，在本文中，我们将使用目前最新的稳定版 [Zookeeper 3.4.11](https://web.archive.org/web/20220625222137/https://zookeeper.apache.org/doc/r3.4.11/index.html) 来代替。

所以我们需要排除 Zookeeper 依赖项，并将 Zookeeper 版本的[依赖项添加到我们的`pom.xml`:](https://web.archive.org/web/20220625222137/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.zookeeper%22%20AND%20a%3A%22zookeeper%22)

```java
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
```

有关兼容性的更多信息，请参考[此链接](https://web.archive.org/web/20220625222137/https://curator.apache.org/zk-compatibility-34.html)。

## 3。连接管理

Apache Curator 的基本用例是连接到一个正在运行的 Apache Zookeeper 实例。

该工具提供了一个工厂来使用重试策略建立与 Zookeeper 的连接:

```java
int sleepMsBetweenRetries = 100;
int maxRetries = 3;
RetryPolicy retryPolicy = new RetryNTimes(
  maxRetries, sleepMsBetweenRetries);

CuratorFramework client = CuratorFrameworkFactory
  .newClient("127.0.0.1:2181", retryPolicy);
client.start();

assertThat(client.checkExists().forPath("/")).isNotNull();
```

在这个简单的示例中，我们将重试 3 次，并在重试之间等待 100 毫秒，以防出现连接问题。

一旦使用`CuratorFramework`客户端连接到 Zookeeper，我们现在可以浏览路径，获取/设置数据，并与服务器进行交互。

## 4。异步

**馆长异步模块使用[completion stage Java 8 API](https://web.archive.org/web/20220625222137/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletionStage.html)包装上述`CuratorFramework`客户端，提供非阻塞能力**。

让我们看看前面的例子是如何使用异步包装器的:

```java
int sleepMsBetweenRetries = 100;
int maxRetries = 3;
RetryPolicy retryPolicy 
  = new RetryNTimes(maxRetries, sleepMsBetweenRetries);

CuratorFramework client = CuratorFrameworkFactory
  .newClient("127.0.0.1:2181", retryPolicy);

client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);

AtomicBoolean exists = new AtomicBoolean(false);

async.checkExists()
  .forPath("/")
  .thenAcceptAsync(s -> exists.set(s != null));

await().until(() -> assertThat(exists.get()).isTrue());
```

现在，`checkExists()`操作工作在异步模式，不阻塞主线程。相反，我们也可以使用`thenAcceptAsync()`方法一个接一个地链接动作，该方法使用 [CompletionStage API](https://web.archive.org/web/20220625222137/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletionStage.html) 。

## 5。配置管理

在分布式环境中，最常见的挑战之一是管理许多应用程序之间的共享配置。我们可以使用 Zookeeper 作为数据存储来保存我们的配置。

让我们看一个使用 Apache Curator 获取和设置数据的例子:

```java
CuratorFramework client = newClient();
client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
String key = getKey();
String expected = "my_value";

client.create().forPath(key);

async.setData()
  .forPath(key, expected.getBytes());

AtomicBoolean isEquals = new AtomicBoolean();
async.getData()
  .forPath(key)
  .thenAccept(data -> isEquals.set(new String(data).equals(expected)));

await().until(() -> assertThat(isEquals.get()).isTrue());
```

在这个例子中，我们创建了节点路径，在 Zookeeper 中设置了数据，然后我们通过检查值是否相同来恢复它。`key`字段可以是类似于`/config/dev/my_key`的节点路径。

### 5.1。观察者

Zookeeper 中另一个有趣的特性是监视关键点或节点的能力。**它允许我们监听配置中的变化并更新我们的应用，而无需重新部署**。

让我们看看上面的例子在使用观察器时是什么样子的:

```java
CuratorFramework client = newClient()
client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
String key = getKey();
String expected = "my_value";

async.create().forPath(key);

List<String> changes = new ArrayList<>();

async.watched()
  .getData()
  .forPath(key)
  .event()
  .thenAccept(watchedEvent -> {
    try {
        changes.add(new String(client.getData()
          .forPath(watchedEvent.getPath())));
    } catch (Exception e) {
        // fail ...
    }});

// Set data value for our key
async.setData()
  .forPath(key, expected.getBytes());

await()
  .until(() -> assertThat(changes.size()).isEqualTo(1));
```

我们配置观察器，设置数据，然后确认被观察的事件被触发。我们可以一次观察一个节点或一组节点。

## 6。强类型模型

Zookeeper 主要处理字节数组，所以我们需要序列化和反序列化我们的数据。这使得我们可以灵活地处理任何可序列化的实例，但是这很难维护。

为了有所帮助，馆长添加了[类型化模型](https://web.archive.org/web/20220625222137/https://curator.apache.org/curator-x-async/modeled.html)的概念，**委托序列化/反序列化，并允许我们直接使用我们的类型**。让我们看看它是如何工作的。

首先，我们需要一个序列化框架。馆长建议使用杰克逊实现，所以让我们将杰克逊依赖关系添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

现在，让我们尝试保存我们的自定义类`HostConfig`:

```java
public class HostConfig {
    private String hostname;
    private int port;

    // getters and setters
}
```

我们需要提供从`HostConfig`类到路径的模型规范映射，并使用 Apache Curator 提供的模型化框架包装器:

```java
ModelSpec<HostConfig> mySpec = ModelSpec.builder(
  ZPath.parseWithIds("/config/dev"), 
  JacksonModelSerializer.build(HostConfig.class))
  .build();

CuratorFramework client = newClient();
client.start();

AsyncCuratorFramework async 
  = AsyncCuratorFramework.wrap(client);
ModeledFramework<HostConfig> modeledClient 
  = ModeledFramework.wrap(async, mySpec);

modeledClient.set(new HostConfig("host-name", 8080));

modeledClient.read()
  .whenComplete((value, e) -> {
     if (e != null) {
          fail("Cannot read host config", e);
     } else {
          assertThat(value).isNotNull();
          assertThat(value.getHostname()).isEqualTo("host-name");
          assertThat(value.getPort()).isEqualTo(8080);
     }
   });
```

读取路径`/config/dev`时的`whenComplete()`方法将返回 Zookeeper 中的`HostConfig`实例。

## 7。食谱

Zookeeper 提供了[这个指南](https://web.archive.org/web/20220625222137/https://zookeeper.apache.org/doc/current/recipes.html)来实现**高级解决方案或配方，如首领选举、分布式锁或共享计数器。**

Apache Curator 为这些食谱中的大部分提供了一个实现。要查看完整列表，请访问[馆长食谱文档](https://web.archive.org/web/20220625222137/https://curator.apache.org/curator-recipes/index.html)。

所有这些配方都在单独的模块中提供:

```java
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.1</version>
</dependency>
```

让我们马上开始，通过一些简单的例子来理解这些。

### 7.1。领袖选举

在分布式环境中，我们可能需要一个主节点或领导节点来协调复杂的任务。

这是[领袖选举配方](https://web.archive.org/web/20220625222137/https://curator.apache.org/curator-recipes/leader-election.html)在策展人中的用法:

```java
CuratorFramework client = newClient();
client.start();
LeaderSelector leaderSelector = new LeaderSelector(client, 
  "/mutex/select/leader/for/job/A", 
  new LeaderSelectorListener() {
      @Override
      public void stateChanged(
        CuratorFramework client, 
        ConnectionState newState) {
      }

      @Override
      public void takeLeadership(
        CuratorFramework client) throws Exception {
      }
  });

// join the members group
leaderSelector.start();

// wait until the job A is done among all members
leaderSelector.close();
```

当我们启动 leader 选择器时，我们的节点加入路径`/mutex/select/leader/for/job/A`中的成员组。一旦我们的节点成为领导者，将调用`takeLeadership`方法，我们作为领导者可以恢复工作。

### 7.2。共享锁

[共享锁方法](https://web.archive.org/web/20220625222137/https://curator.apache.org/curator-recipes/shared-lock.html)是关于拥有一个完全分布式的锁:

```java
CuratorFramework client = newClient();
client.start();
InterProcessSemaphoreMutex sharedLock = new InterProcessSemaphoreMutex(
  client, "/mutex/process/A");

sharedLock.acquire();

// do process A

sharedLock.release();
```

当我们获得锁时，Zookeeper 确保没有其他应用程序同时获得相同的锁。

### 7.3。计数器

[计数器配方](https://web.archive.org/web/20220625222137/https://curator.apache.org/curator-recipes/shared-counter.html)协调所有客户端之间的共享`Integer` :

```java
CuratorFramework client = newClient();
client.start();

SharedCount counter = new SharedCount(client, "/counters/A", 0);
counter.start();

counter.setCount(counter.getCount() + 1);

assertThat(counter.getCount()).isEqualTo(1);
```

在本例中，Zookeeper 将`Integer`值存储在路径`/counters/A`中，如果路径尚未创建，则将该值初始化为`0`。

## 8。结论

在本文中，我们看到了如何使用 Apache Curator 连接到 Apache Zookeeper，并利用其主要功能。

我们还介绍了策展人的几个主要食谱。

像往常一样，可以在 GitHub 上找到来源[。](https://web.archive.org/web/20220625222137/https://github.com/eugenp/tutorials/tree/master/apache-libraries)