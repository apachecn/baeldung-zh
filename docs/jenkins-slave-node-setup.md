# 设置 Jenkins 从节点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-slave-node-setup>

## 1.概观

在本教程中，我们将回顾 Jenkins 架构中分布式构建的概念。此外，我们将了解如何配置 [Jenkins 主从架构](/web/20220625074856/https://www.baeldung.com/ops/jenkins-performance)。此外，我们将在 Jenkins 主节点上构建一个管道，以便在从节点上运行构建作业。

## 2.分布式构建

理想情况下，**我们安装标准 Jenkins 的机器将成为我们的 Jenkins master** 。在从节点机器上，我们将安装一个名为代理的运行时程序。安装代理将不是标准的 Jenkins 安装，但是这个代理将在 JVM 上运行。它有足够的能力在专用执行器中运行 Jenkins 的子任务或主任务:

[![Distributed Build](img/56c9284bf9be7829814c6f40c842a990.png)](/web/20220625074856/https://www.baeldung.com/wp-content/uploads/2021/07/Distributed-Build.jpg)

**我们可以有任意数量的代理节点或从节点**。此外，我们可以配置主节点来决定哪个任务或作业应该在哪个代理上运行，以及我们可以拥有多少个执行器代理。**主节点和 Jenkins 从节点之间的通信是双向的，通过 TCP/IP** 进行。

## 3.配置 Jenkins 主节点和从节点

标准的 Jenkins 安装包括 Jenkins master，在这个安装中，master 将管理我们构建系统的所有任务 。如果我们正在处理多个项目，我们可以在每个项目上运行多个任务。一些项目需要使用特定的节点，这就需要使用从节点。

Jenkins master 负责调度作业，分配从节点，并向从节点发送构建以供执行。它还将跟踪从节点的状态(离线或在线)，从从节点检索构建结果，并在终端输出上显示它们。在大多数安装中，会将多个从属节点分配给构建作业的任务。

在开始之前，**让我们仔细检查一下添加一个从节点**的所有先决条件:

*   [Jenkins 服务器](/web/20220625074856/https://www.baeldung.com/ops/jenkins-pipelines)已经启动并运行，可以使用了
*   用于从属节点配置的另一台服务器
*   Jenkins 服务器和从属服务器都连接到同一个网络

为了配置主服务器，我们将登录到 Jenkins 服务器，并按照下面的步骤操作。

首先，我们将转到**“管理詹金斯- >管理节点- >新节点”来创建一个新节点** : [![Manage nodes](img/0ed991914b2f46e21c62f0a50e94f017.png)](/web/20220625074856/https://www.baeldung.com/wp-content/uploads/2021/07/Manage-nodes.png)

在下一个屏幕上，我们**输入“节点名称”(slaveNode1)，选择“永久代理”，然后单击“确定”**:

[![New Slave Node](img/8db166a132666cbe84268d4f7105e1ed.png)](/web/20220625074856/https://www.baeldung.com/wp-content/uploads/2021/07/new-node-1.png)

点击“OK”后，**我们将被带到一个带有新表单的屏幕，在这里我们需要填写从节点的信息**。W 我们正在考虑将从节点运行在 Linux 操作系统上，因此启动方法被设置为“通过 ssh 启动代理” 。

同样，我们将添加相关的细节，比如名称、描述和执行者的数量。

**我们将通过按“保存”按钮**来保存我们的工作。名为“slaveNode1”的“标签”将帮助我们在这个从属节点上设置作业:

[![Slave Node Details](img/6ade3460eb26ef2805adc02b9d2a952e.png)](/web/20220625074856/https://www.baeldung.com/wp-content/uploads/2021/07/new-node-2.png)

## 4.在从属节点上构建项目

现在我们的主节点和从节点都准备好了，我们将讨论在从节点上构建项目的步骤。

为此，我们从点击仪表板左上角的“新项目”开始。

接下来，我们需要在“输入项目名称”字段中输入我们项目的名称，并选择“管道项目”，然后单击“确定”按钮。

在下一个屏幕上，我们将输入“描述”(可选)并导航到“管道”部分。确保“定义”字段选择了管道脚本选项。

之后，我们将以下声明性管道脚本复制并粘贴到“脚本”字段中:

```java
node('slaveNode1'){
    stage('Build') {
        sh '''echo build steps'''
    }
    stage('Test') {
        sh '''echo test steps'''
    }
} 
```

接下来，我们单击“保存”按钮。这将重定向到管道视图页面。

在左侧窗格中，我们单击“Build Now”按钮来执行管道。管道执行完成后，我们将看到管道视图:

[![Console Output](img/a7a0d38b351383d8c59be92abad6c436.png)](/web/20220625074856/https://www.baeldung.com/wp-content/uploads/2021/07/m-s-pipeline-console-output.png)

**我们可以通过点击构建号**在构建历史下验证已执行构建的历史。如上所示，当我们点击构建号并选择“控制台输出”时，我们可以看到管道在我们的`slaveNode1`机器上运行。

## 5.结论

在本教程中，我们已经介绍了 Jenkins 中的分布式构建，并且看到了 Jenkins 主节点如何调用从节点。此外，我们还学习了如何设置 Jenkins 主从配置。最后，我们对从节点配置进行了测试。