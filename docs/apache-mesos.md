# 阿帕奇 Mesos 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-mesos>

## 1.概观

我们通常在同一个机器集群上部署各种应用程序。例如，如今在同一个集群中拥有像 [Apache Spark](/web/20220523231851/https://www.baeldung.com/apache-spark) 或 [Apache Flink](/web/20220523231851/https://www.baeldung.com/apache-flink) 这样的分布式处理引擎和像 [Apache Cassandra](/web/20220523231851/https://www.baeldung.com/cassandra-with-java) 这样的分布式数据库是很常见的。

Apache Mesos 是一个允许此类应用程序之间有效资源共享的平台。

在本文中，我们将首先讨论部署在同一个集群上的应用程序中的一些资源分配问题。稍后，我们将看到 Apache Mesos 如何在应用程序之间提供更好的资源利用率。

## 2.共享集群

许多应用程序需要共享一个集群。总的来说，有两种常见的方法:

*   对集群进行静态分区，并在每个分区上运行一个应用程序
*   将一组机器分配给应用程序

尽管这些方法允许应用程序彼此独立运行，但它没有实现高资源利用率。

例如，考虑一个应用程序，**只运行了很短一段时间，然后是一段时间的不活动。**现在，因为我们已经为这个应用程序分配了静态机器或分区，所以在非活动期间我们有**未利用的资源。**

我们可以通过将非活动期间的空闲资源重新分配给其他应用程序来优化资源利用率。

Apache Mesos 有助于应用程序之间的动态资源分配。

## 3\. Apache Mesos

对于我们上面讨论的两种集群共享方法，应用程序只知道它们正在运行的特定分区或机器的资源。但是，Apache Mesos 为应用程序提供了集群中所有资源的抽象视图。

我们很快就会看到，Mesos 充当机器和应用程序之间的接口。它为应用程序提供集群中所有机器上的**可用资源。** It **频繁地更新该信息，以包括由已经达到完成状态的应用**释放的资源。这使得应用程序能够做出关于在哪个机器上执行哪个任务的最佳决策。

为了理解 Mesos 是如何工作的，让我们来看看它的[架构](https://web.archive.org/web/20220523231851/https://mesos.apache.org/documentation/latest/architecture/):

[![](img/a07c81e9286bdec6bc9c15f8a2669e87.png)](/web/20220523231851/https://www.baeldung.com/wp-content/uploads/2019/07/Mesos-arch.jpg)

这张图片是 Mesos 官方文档的一部分([来源](https://web.archive.org/web/20220523231851/https://mesos.apache.org/assets/img/documentation/architecture3.jpg))。这里，`Hadoop`和`MPI`是共享集群的两个应用。

我们将在接下来的几节中讨论这里显示的每个组件。

### 3.1\. Mesos Master

Master 是这个设置中的核心组件，它存储集群中资源的当前状态。此外，它还通过传递资源和任务等信息，充当代理和应用程序之间的**协调器。**

由于 master 中的任何故障都会导致关于资源和任务的状态丢失，因此我们将其部署在高可用性配置中。如上图所示，Mesos 部署了备用主守护进程和一个主守护进程。这些守护进程依靠 Zookeeper 在出现故障时恢复状态。

### 3.2\. Mesos Agents

Mesos 集群必须在每台机器上运行一个代理。这些代理**定期向主代理**报告它们的资源，然后**接收应用程序调度运行的任务**。在计划任务完成或丢失后，此循环会重复。

在接下来的几节中，我们将看到应用程序如何在这些代理上调度和执行任务。

### 3.3.Mesos 框架

Mesos 允许应用程序实现一个抽象组件，该组件与主机交互以**接收集群**中的可用资源，此外**基于这些资源做出调度决策**。这些组件被称为框架。

Mesos 框架由两个子组件组成:

*   `Scheduler`–使应用程序能够根据所有代理上的可用资源来调度任务
*   `Executor`–在所有代理上运行，包含在该代理上执行任何计划任务所需的所有信息

整个过程描述如下:

[![](img/d890f66b5d03f8ba800cf135c097d36c.png)](/web/20220523231851/https://www.baeldung.com/wp-content/uploads/2019/07/Mesos-flow.jpg)

首先，代理向主代理报告它们的资源。此时，master 将这些资源提供给所有注册的调度程序。这个过程被称为资源提供，我们将在下一节详细讨论它。

然后，调度程序选择最佳代理，并通过主代理在其上执行各种任务。一旦执行器完成了分配的任务，代理就向主服务器重新发布它们的资源。Master 为集群中的所有框架重复这个资源共享过程。

Mesos 允许应用程序用各种编程语言实现他们的定制调度器和执行器 T2。一个 Java 实现的调度器必须**实现**T5`Scheduler `接口:

```java
public class HelloWorldScheduler implements Scheduler {

    @Override
    public void registered(SchedulerDriver schedulerDriver, Protos.FrameworkID frameworkID, 
      Protos.MasterInfo masterInfo) {
    }

    @Override
    public void reregistered(SchedulerDriver schedulerDriver, Protos.MasterInfo masterInfo) {
    }

    @Override
    public void resourceOffers(SchedulerDriver schedulerDriver, List<Offer> list) {
    }

    @Override
    public void offerRescinded(SchedulerDriver schedulerDriver, OfferID offerID) {
    }

    @Override
    public void statusUpdate(SchedulerDriver schedulerDriver, Protos.TaskStatus taskStatus) {
    }

    @Override
    public void frameworkMessage(SchedulerDriver schedulerDriver, Protos.ExecutorID executorID, 
      Protos.SlaveID slaveID, byte[] bytes) {
    }

    @Override
    public void disconnected(SchedulerDriver schedulerDriver) {
    }

    @Override
    public void slaveLost(SchedulerDriver schedulerDriver, Protos.SlaveID slaveID) {
    }

    @Override
    public void executorLost(SchedulerDriver schedulerDriver, Protos.ExecutorID executorID, 
      Protos.SlaveID slaveID, int i) {
    }

    @Override
    public void error(SchedulerDriver schedulerDriver, String s) {
    }
}
```

可以看出，它主要由**各种回调方法组成，特别是与主**的通信。

类似地，执行程序的实现必须实现`Executor `接口:

```java
public class HelloWorldExecutor implements Executor {
    @Override
    public void registered(ExecutorDriver driver, Protos.ExecutorInfo executorInfo, 
      Protos.FrameworkInfo frameworkInfo, Protos.SlaveInfo slaveInfo) {
    }

    @Override
    public void reregistered(ExecutorDriver driver, Protos.SlaveInfo slaveInfo) {
    }

    @Override
    public void disconnected(ExecutorDriver driver) {
    }

    @Override
    public void launchTask(ExecutorDriver driver, Protos.TaskInfo task) {
    }

    @Override
    public void killTask(ExecutorDriver driver, Protos.TaskID taskId) {
    }

    @Override
    public void frameworkMessage(ExecutorDriver driver, byte[] data) {
    }

    @Override
    public void shutdown(ExecutorDriver driver) {
    }
}
```

我们将在后面的小节中看到调度器和执行器的运行版本。

## 4.资源管理

### 4.1.资源报价

正如我们前面讨论的，代理向主服务器发布它们的资源信息。反过来，主服务器将这些资源提供给集群中运行的框架。这个过程被称为`resource offer.`

资源报价由两部分组成——资源和属性。

资源用于**发布代理机**的硬件信息，如内存、CPU、磁盘等。

每个代理都有五种预定义的资源:

*   `cpu`
*   `gpus`
*   `mem`
*   `disk`
*   `ports`

这些资源的值可以定义为以下三种类型之一:

*   `Scalar`–用于使用浮点数表示数字信息，以允许小数值，如 1.5G 内存
*   `Range`–用于表示标量值的范围–例如，端口范围
*   `Set`–用于表示多个文本值

默认情况下，Mesos 代理尝试从机器上检测这些资源。

但是，在某些情况下，我们可以在代理上配置自定义资源。这种自定义资源的值也应该是上面讨论的任何一种类型。

例如，我们可以用这些资源启动代理:

```java
--resources='cpus:24;gpus:2;mem:24576;disk:409600;ports:[21000-24000,30000-34000];bugs(debug_role):{a,b,c}'
```

可以看到，我们已经为代理配置了一些预定义的资源和一个名为`bugs `的自定义资源，它属于`set `类型。

除了资源之外，代理还可以向主服务器发布键值属性。这些属性在调度决策中充当代理和帮助框架的附加元数据。

一个有用的例子是**将代理添加到不同的机架或区域**，然后**在同一机架或区域**上调度各种任务，以实现数据局部性:

```java
--attributes='rack:abc;zone:west;os:centos5;level:10;keys:[1000-1500]'
```

与资源类似，属性值可以是标量、范围或文本类型。

### 4.2.资源角色

许多现代操作系统支持多用户。同样，Mesos 也支持同一个集群中的多个用户。这些用户被称为`roles`。我们可以将每个角色视为集群中的资源消费者。

由于这一点，Mesos 代理可以根据不同的分配策略在不同的角色下划分资源。此外，框架可以订阅集群中的这些角色，并对不同角色下的资源进行细粒度控制。

例如，考虑一个托管应用程序的**集群，这些应用程序为组织中的不同用户**提供服务。因此，通过**将资源划分为角色，每个应用程序都可以彼此独立工作**。

此外，框架可以使用这些角色来实现数据局部性。

例如，假设我们在集群中有两个名为`producer `和`consumer. `的应用程序，`producer `将数据写入一个持久卷，`consumer `随后可以读取该卷。我们可以通过与`producer.`共享卷来优化`consumer `应用

因为 Mesos 允许多个应用程序订阅同一个角色，所以我们可以将持久卷与资源角色相关联。此外，`producer `和`consumer `的框架都将订阅相同的资源角色。因此，`consumer `应用程序现在可以在与`producer `应用程序相同的卷上启动数据读取任务**。**

### 4.3.资源预留

现在的问题是，Mesos 如何将集群资源分配给不同的角色。Mesos 通过预留来分配资源。

有两种类型的预订:

*   静态保留
*   动态预订

静态预留类似于我们在前面讨论的代理启动时的资源分配:

```java
 --resources="cpus:4;mem:2048;cpus(baeldung):8;mem(baeldung):4096"
```

这里唯一的不同是，现在 Mesos 代理**为名为`baeldung`的角色保留了 8 个 CPU 和 4096m 内存。**

与静态预留不同，动态预留允许我们重新安排角色内的资源。Mesos 允许框架和集群操作者通过响应资源提供的框架消息或通过 [HTTP 端点](https://web.archive.org/web/20220523231851/https://mesos.apache.org/documentation/latest/reservation/#examples)动态地改变资源的分配。

Mesos 将没有任何角色的所有资源分配给名为(*)的默认角色。Master 向所有框架提供这样的资源，不管它们是否订阅了它。

### 4.4.资源权重和配额

通常，Mesos 主机使用公平策略提供资源。它使用加权主导资源公平性(wDRF)来识别缺少资源的角色。然后，主服务器向订阅了这些角色的框架提供更多的资源。

尽管应用程序之间公平共享资源是 Mesos 的一个重要特性，但并不总是必要的。假设一个集群托管着资源占用量低的应用程序和资源需求高的应用程序。在这种部署中，我们希望根据应用程序的性质来分配资源。

Mesos 允许框架通过订阅角色和为角色增加更高的权重值来要求更多的资源。因此，**如果有两个角色，一个权重为 1，另一个权重为 2，那么 Mesos 会将两倍的公平份额的资源分配给第二个角色。**

与资源类似，我们可以通过 [HTTP 端点](https://web.archive.org/web/20220523231851/https://mesos.apache.org/documentation/latest/weights/#operator-http-endpoint)配置权重。

除了确保资源公平地分配给一个有权重的角色，Mesos 还确保为一个角色分配最少的资源。

Mesos 允许我们向资源角色添加配额。配额指定**一个角色保证接收的最小资源量**。

## 5.实施框架

正如我们在前面讨论的，Mesos 允许应用程序以自己选择的语言提供框架实现。在 Java 中，使用 main 类实现框架——它充当框架过程的入口点——以及前面讨论的`Scheduler `和`Executor `的实现。

### 5.1.框架主类

在我们实现调度器和执行器之前，我们将首先实现框架的入口点:

*   向主服务器注册自己
*   向代理提供执行器运行时信息
*   启动调度程序

我们将首先为 Mesos 添加一个 [Maven 依赖关系](https://web.archive.org/web/20220523231851/https://search.maven.org/search?q=g:org.apache.mesos%20AND%20a:mesos&core=gav):

```java
<dependency>
    <groupId>org.apache.mesos</groupId>
    <artifactId>mesos</artifactId>
    <version>0.28.3</version>
</dependency>
```

接下来，我们将为我们的框架实现`HelloWorldMain `。我们要做的第一件事是在 Mesos 代理上启动 executor 进程:

```java
public static void main(String[] args) {

    String path = System.getProperty("user.dir")
      + "/target/libraries2-1.0.0-SNAPSHOT.jar";

    CommandInfo.URI uri = CommandInfo.URI.newBuilder().setValue(path).setExtract(false).build();

    String helloWorldCommand = "java -cp libraries2-1.0.0-SNAPSHOT.jar com.baeldung.mesos.executors.HelloWorldExecutor";
    CommandInfo commandInfoHelloWorld = CommandInfo.newBuilder()
      .setValue(helloWorldCommand)
      .addUris(uri)
      .build();

    ExecutorInfo executorHelloWorld = ExecutorInfo.newBuilder()
      .setExecutorId(Protos.ExecutorID.newBuilder()
      .setValue("HelloWorldExecutor"))
      .setCommand(commandInfoHelloWorld)
      .setName("Hello World (Java)")
      .setSource("java")
      .build();
}
```

这里，我们首先配置了 executor 二进制文件的位置。Mesos 代理将在框架注册时下载这个二进制文件。接下来，代理将运行给定的命令来启动 executor 进程。

接下来，我们将初始化我们的框架并启动调度程序:

```java
FrameworkInfo.Builder frameworkBuilder = FrameworkInfo.newBuilder()
  .setFailoverTimeout(120000)
  .setUser("")
  .setName("Hello World Framework (Java)");

frameworkBuilder.setPrincipal("test-framework-java");

MesosSchedulerDriver driver = new MesosSchedulerDriver(new HelloWorldScheduler(),
  frameworkBuilder.build(), args[0]);
```

最后，**我们将启动向主服务器注册的`MesosSchedulerDriver` 。为了成功注册，我们必须将 Master 的 IP 作为程序参数`args[0]`传递给这个 main 类:**

```java
int status = driver.run() == Protos.Status.DRIVER_STOPPED ? 0 : 1;

driver.stop();

System.exit(status);
```

在上面显示的类中，`CommandInfo, ExecutorInfo,`和`FrameworkInfo` 都是主框架和框架之间的 [protobuf 消息](/web/20220523231851/https://www.baeldung.com/google-protocol-buffer)的 Java 表示。

### 5.2.实施调度程序

从 Mesos 1.0 开始，**我们可以从任何 Java 应用程序调用 [HTTP 端点](https://web.archive.org/web/20220523231851/https://mesos.apache.org/documentation/latest/scheduler-http-api/)来发送和接收消息到 Mesos 主机**。这些消息中的一些包括例如框架注册、资源提供和提供拒绝。

对于 **Mesos 0.28 或者更早的版本，我们需要实现`Scheduler`接口**:

在大多数情况下，我们将只关注`Scheduler.`的`resourceOffers` 方法，让我们看看调度程序如何接收资源并基于它们初始化任务。

首先，我们将了解调度程序如何为任务分配资源:

```java
@Override
public void resourceOffers(SchedulerDriver schedulerDriver, List<Offer> list) {

    for (Offer offer : list) {
        List<TaskInfo> tasks = new ArrayList<TaskInfo>();
        Protos.TaskID taskId = Protos.TaskID.newBuilder()
          .setValue(Integer.toString(launchedTasks++)).build();

        System.out.println("Launching printHelloWorld " + taskId.getValue() + " Hello World Java");

        Protos.Resource.Builder cpus = Protos.Resource.newBuilder()
          .setName("cpus")
          .setType(Protos.Value.Type.SCALAR)
          .setScalar(Protos.Value.Scalar.newBuilder()
            .setValue(1));

        Protos.Resource.Builder mem = Protos.Resource.newBuilder()
          .setName("mem")
          .setType(Protos.Value.Type.SCALAR)
          .setScalar(Protos.Value.Scalar.newBuilder()
            .setValue(128));
```

这里，我们为我们的任务分配了 1 个 CPU 和 128M 内存。接下来，我们将使用`SchedulerDriver `在代理上启动任务:

```java
 TaskInfo printHelloWorld = TaskInfo.newBuilder()
          .setName("printHelloWorld " + taskId.getValue())
          .setTaskId(taskId)
          .setSlaveId(offer.getSlaveId())
          .addResources(cpus)
          .addResources(mem)
          .setExecutor(ExecutorInfo.newBuilder(helloWorldExecutor))
          .build();

        List<OfferID> offerIDS = new ArrayList<>();
        offerIDS.add(offer.getId());

        tasks.add(printHelloWorld);

        schedulerDriver.launchTasks(offerIDS, tasks);
    }
}
```

或者，`Scheduler`经常发现需要拒绝资源提议。例如，如果`Scheduler `由于缺乏资源而无法在代理上启动任务，它必须立即拒绝该提议:

```java
schedulerDriver.declineOffer(offer.getId());
```

### 5.3.实施`Executor`

正如我们前面讨论的，框架的执行器组件负责在 Mesos 代理上执行应用程序任务。

我们使用 HTTP 端点来实现 Mesos 1.0 中的`Scheduler`。同样，我们可以使用 [HTTP 端点](https://web.archive.org/web/20220523231851/https://mesos.apache.org/documentation/latest/executor-http-api/)作为执行器。

在前面的部分中，我们讨论了框架如何配置代理来启动 executor 进程:

```java
java -cp libraries2-1.0.0-SNAPSHOT.jar com.baeldung.mesos.executors.HelloWorldExecutor
```

值得注意的是，**这个命令认为`HelloWorldExecutor `是主类。**我们将实现这个`main`方法来**初始化`MesosExecutorDriver `** ，它连接到 Mesos 代理以接收任务并共享其他信息，如任务状态:

```java
public class HelloWorldExecutor implements Executor {
    public static void main(String[] args) {
        MesosExecutorDriver driver = new MesosExecutorDriver(new HelloWorldExecutor());
        System.exit(driver.run() == Protos.Status.DRIVER_STOPPED ? 0 : 1);
    }
}
```

现在要做的最后一件事是从框架接受任务，并在代理上启动它们。启动任何任务的信息都包含在`HelloWorldExecutor:`中

```java
public void launchTask(ExecutorDriver driver, TaskInfo task) {

    Protos.TaskStatus status = Protos.TaskStatus.newBuilder()
      .setTaskId(task.getTaskId())
      .setState(Protos.TaskState.TASK_RUNNING)
      .build();
    driver.sendStatusUpdate(status);

    System.out.println("Execute Task!!!");

    status = Protos.TaskStatus.newBuilder()
      .setTaskId(task.getTaskId())
      .setState(Protos.TaskState.TASK_FINISHED)
      .build();
    driver.sendStatusUpdate(status);
}
```

当然，这只是一个简单的实现，但它解释了执行者如何在每个阶段与主者共享任务状态，然后在发送完成状态之前执行任务。

在某些情况下，执行器还可以将数据发送回调度程序:

```java
String myStatus = "Hello Framework";
driver.sendFrameworkMessage(myStatus.getBytes());
```

## 6.结论

在本文中，我们简要讨论了在同一个集群中运行的应用程序之间的资源共享。我们还讨论了 Apache Mesos 如何通过集群资源(如 CPU 和内存)的抽象视图帮助应用程序实现最大利用率。

稍后，我们讨论了基于**各种公平策略和角色的**应用**之间的资源动态分配。** Mesos 允许应用程序根据集群中 Mesos 代理提供的资源做出**调度决策。**

最后，我们看到了 Mesos 框架在 Java 中的实现。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220523231851/https://github.com/eugenp/tutorials/tree/master/libraries-2)