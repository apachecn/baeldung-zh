# Jenkins 架构和性能改进指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-performance>

## 1.概观

在本教程中，我们将回顾詹金斯架构的基础。此外，我们将了解如何配置 Jenkins 来提高性能。此外，我们将讨论手动重启或关闭 Jenkins 的选项。

## 2。詹金斯架构

一台 Jenkins 服务器无法满足某些需求。首先，我们可能需要多个不同的环境来测试我们的构建。单个 Jenkins 服务器无法做到这一点。其次，如果定期生产更大更重的项目，单个 Jenkins 服务器将不堪重负。

Jenkins 分布式体系结构就是为了满足上述需求而创建的。此外， **Jenkins 使用主从架构**管理分布式构建。[TCP/IP 协议](/web/20220523154418/https://www.baeldung.com/cs/udp-vs-tcp)用于本设计中主从机之间的通信。

### 2.1.詹金斯大师

Jenkins 主服务器负责调度作业、分配从服务器，并向从服务器发送构建以执行作业。它还将跟踪从属服务器的状态(离线或在线)，并从从属服务器检索构建结果响应，并将它们显示在控制台输出上。

### 2.2.詹金斯奴隶

它运行在远程服务器上。**Jenkins 服务器遵循 Jenkins 主服务器的请求，并与所有操作系统兼容**。主设备分派的构建任务由从设备执行。此外，该项目可以配置为选择特定的从机。

### 2.3.分布式主从结构

让我们用一个例子来看看 Jenkins 架构。下图描述了一个主设备和三个 Jenkins 从设备:

[![](img/698e40cd4e54cf7a468ea6f9c185f92c.png)](/web/20220523154418/https://www.baeldung.com/wp-content/uploads/2021/06/Jenkins-Architecture-1.svg)

让我们看看如何利用 Jenkins 在各种系统中进行测试，如 Ubuntu、Windows 或 Mac:

[![](img/54412e3bca397cba5b042441aa47b2be.png)](/web/20220523154418/https://www.baeldung.com/wp-content/uploads/2021/06/code-commit.svg)

在该图中，考虑了以下项目:

*   Jenkins 将定期检查 GIT 存储库中源代码的任何变化
*   每个 Jenkins 构建都需要自己的测试环境，这不能在单个服务器上创建。詹金斯通过根据需要雇佣各种奴隶来实现这一目标
*   詹金斯主人将向这些奴隶传达测试请求，以及测试报告

## 3。詹金斯命令行界面

Jenkins 有一个命令行界面，用户和管理员可以使用它从脚本或 shell 环境访问 Jenkins。SSH、Jenkins CLI 客户端或 Jenkins 附带的 JAR 文件可以使用这个命令行界面。

为此，我们必须首先从位于 URL `/jnlpJars/jenkins-cli.jar`的 Jenkins 控制器下载`jenkins-cli.jar`，实际上是`JENKINS_URL/jnlpJars/jenkins-cli.jar,`，然后如下运行它:

```java
java -jar jenkins-cli.jar -s http://localhost:8080/ -webSocket help
```

通过从“管理 Jenkins”页面的“工具和操作”部分选择“Jenkins CLI ”,可以使用此选项:

[![](img/19ca9acca58ec93d6e13b4eed3f9b375.png)](/web/20220523154418/https://www.baeldung.com/wp-content/uploads/2021/06/Jenkins-CLI-Commands.png)

可用命令列表显示在该区域中。我们可以使用命令行工具通过这些命令来访问各种功能。

## 4.手动重启 Jenkins

如果我们希望手动重启或关闭 Jenkins，只需遵循以下步骤:

### 4.1。重启

我们可以使用 Jenkins Rest API 执行重启。这将强制重新启动该过程，而不等待现有作业完成:

```java
http://(jenkins_url)/restart 
```

我们可以使用 Jenkins Rest API 来执行`safeRestart`。这允许我们完成任何现有的任务:

```java
http://(jenkins_url)/safeRestart
```

如果我们将它作为`rpm`或`deb`包安装，下面的命令将起作用:

```java
service jenkins restart 
```

### 4.2。Ubuntu

我们还可以使用`apt-get/dpkg`来安装以下内容:

```java
sudo /etc/init.d/jenkins restart
Usage: /etc/init.d/jenkins {start|stop|status|restart|force-reload} 
```

### 4.3。安全关闭詹金斯

如果我们希望安全地关闭 Jenkins，我们可以使用 Jenkins Rest API 执行退出:

```java
http://(jenkins_url)/exit
```

我们可以使用 Jenkins Rest API 执行 kill 来终止我们的所有进程:

```java
http://(jenkins_url)/kill
```

## 5。提升詹金斯的表现

滞后或缓慢的响应是 Jenkins 用户的典型抱怨，并且有许多报告的故障。缓慢的 CI 系统不方便，因为它们降低了开发速度并浪费了时间。通过一些简单的建议，我们可以提高这些系统的性能。

在接下来的部分中，我们将讨论一些改进 Jenkins 的建议，让我们的工程师脸上露出微笑。

### 5.1.最小化主节点上的生成

**主节点是应用程序实际执行的地方；这是詹金斯的大脑，不像奴隶，它是不可替代的**。因此，我们希望让我们的 Jenkins 主服务器尽可能“不工作”，使用 CPU 和 RAM 来调度和触发从服务器上的构建。我们可以通过将作业限制在一个节点标签上来实现这一点，比如`SlaveNode`。

为管道作业分配节点时，使用标签，例如:

```java
stage("stage 1"){
    node("SlaveNode"){
        sh "echo \"Hello ${params.NAME}\" "
    }
} 
```

在这些情况下，任务和节点块将只在带有`SlaveNode`标签的从机上运行。

### 5.2.不要保留太多构建历史

在配置作业时，我们可以指定要在文件系统上保留多少次构建以及保留多长时间。当我们在短时间内触发一个作业的许多构建时，这个称为丢弃旧构建的特性就变得很有用。

我们已经看到了历史限制设置得太高，导致保存了过多构建的例子。此外，在这种情况下，Jenkins 不得不加载大量旧版本。

### 5.3。清除旧的詹金斯数据

继续前面关于构建数据的建议，另一个需要了解的关键要素是旧的数据管理功能。我们可能知道，Jenkins 管理作业并在文件系统上存储数据。当我们升级核心、安装或更新插件时，数据格式可能会改变。

在这种情况下，Jenkins 将旧的数据格式保存到文件系统，并将新的格式加载到内存中。如果我们需要回滚升级，这是非常有益的，但有时会有太多的数据被加载到 RAM 中。缓慢的 UI 响应甚至内存不足的问题都是内存使用率高的迹象。建议打开之前的数据管理页面，以避免这种情况:

[![](img/02b5b3b8bd41776dfa3afe51b5b3ee88.png)](/web/20220523154418/https://www.baeldung.com/wp-content/uploads/2021/06/Manage-old-data.png)

### 5.4。定义正确的堆大小

当前许多 Java 应用程序都使用最大堆大小选项。在定义堆大小时，有一个重要的 JVM 方面需要注意。`[UseCompressedOops](/web/20220523154418/https://www.baeldung.com/jvm-compressed-oops)` 是这个特性的名字，它只在我们大多数人使用的 64 位平台上工作。它将对象的指针从 64 位减少到 32 位，节省了大量内存。

默认情况下，该标志在大小不超过 32GB(稍小)的堆上启用，在大于 32GB 的堆上停止工作。堆应该扩展到 48GB，以补偿损失的容量。因此，在定义堆大小时，建议将其保持在 32GB 以下。

我们可以使用下面的命令(`jinfo`包含在 JDK 中)来查看是否设置了标志:

```java
jinfo -flag UseCompressedOops <pid>
```

### 5.5。调整垃圾收集器

[垃圾收集器](/web/20220523154418/https://www.baeldung.com/jvm-garbage-collectors)是一个在后台运行的内存管理系统。

它的主要目标是在堆中找到未使用的对象，并释放它们所包含的内存。Java 应用程序可能会因为一些 GC 操作而停止运行(还记得 UI 冻结吗？).如果我们的应用程序有一个巨大的堆(超过 4GB)，这是最有可能发生的。为了减少这些情况下的延迟时间，需要进行 GC 优化。在多个 Jenkins 设置中应对这些挑战后，我们提出了以下建议:

*   启用 [G1GC](/web/20220523154418/https://www.baeldung.com/jvm-garbage-collectors) —最新的 GC 实现(默认为 JDK9)
*   启用 [GC 日志记录](/web/20220523154418/https://www.baeldung.com/java-gc-logging-to-file)–这将有助于未来的监控和调优
*   如果需要，用额外的标志配置 GC
*   继续监控

## 6。结论

在这个快速教程中，我们首先讨论了 Jenkins 中的分布式主从架构。之后，我们看了几个选项，手动启动，停止和重启詹金斯。最后，我们研究了 Jenkins 中的不同配置以提高性能。

如果我们花一些时间在 Jenkins 上并遵循这些指导方针，我们将能够利用它的许多有用的功能，同时避免潜在的危险。