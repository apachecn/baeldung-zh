# 从詹金斯管道触发另一个任务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-pipeline-trigger-new-job>

## 1.概观

Jenkins 是一款自动化服务器，为开发人员提供自动构建、测试和部署应用的能力。Jenkins 服务器执行作业，这些作业可以手动或自动触发。此外，我们可以同时或按特定顺序运行这些作业。

在本教程中，我们将通过创建一个自由作业和一个[管道作业](https://web.archive.org/web/20230103152204/https://www.jenkins.io/pipeline/getting-started-pipelines/)来触发另一个作业。

## 2.作业设置

在 Jenkins 中，我们可以创建工作来运行代码覆盖测试、集成测试和部署应用程序。为了执行一组任务，Jenkins 提供了不同类型的作业。此外，根据任务的不同，我们可能需要触发一个内部作业。

这里，我们将构建两个作业，一个父管道作业和一个子自由式作业。**我们将从 Jenkins UI 手动运行父作业，而子作业将由父作业在内部触发。**

先看看孩子的工作。

### 2.1.创建自由式子作业

在开始之前，让我们先构建一个演示设置。我们将使用自由作业类型来创建子作业。自由式构建作业易于使用，并且有许多预制的选项。此外，要创建自由式作业，我们需要遵循以下步骤:

*   点击 Jenkins 仪表盘中的`New` `Item`

*   将“作业名称”设置为`childJob`

*   选择“作业类型”为“`Freestyle project”`

*   在构建步骤的执行 shell 中添加 `echo “childJob”`命令

*   保存作业

所有内容看起来都应该与此类似:

[![childFreestyleJob](img/86ce114bb5b266d12d3e67ec2990ef06.png)](/web/20230103152204/https://www.baeldung.com/wp-content/uploads/2022/12/Screenshot-2022-12-15-at-12.05.11-PM.png)

作为上述步骤的结果，我们现在能够创建一个简单的自由式作业，它可以独立运行，也可以由另一个作业触发。

### 2.2.创建管道父作业

Jenkins pipeline 是一系列相互关联的事件或工作，在我们的软件开发工作流程中产生持续交付的。在这里，我们将创建一个管道作业，该作业将在内部调用我们刚刚创建的`childJob`。让我们看看管道作业的 Groovy 脚本:

```java
pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                echo "parentJob"
            }
        }
        stage('triggerChildJob') {
            steps {
                build job: "childJob", wait: true
            }
        }
    }
}
```

上述脚本包含管道作业中的两个阶段`build`和`triggerChildJob`。第一级简单地执行`[echo](/web/20230103152204/https://www.baeldung.com/linux/printf-echo)`命令，而第二级将在内部触发`childJob`。

现在让我们使用上面的 Groovy 脚本创建一个管道作业:

*   点击 Jenkins 仪表盘中的`New` `Item`

*   将“作业名称”设置为`parentJob`

*   选择“作业类型”为`Pipeline project`

*   如上所述添加 Groovy 脚本并保存作业

因此:

[![](img/769da4ac969fa830cd99f2d982089532.png)](/web/20230103152204/https://www.baeldung.com/wp-content/uploads/2022/12/Screenshot-2022-12-15-at-11.43.37-AM.png)

我们现在可以从 Jenkins 管道作业中触发自由式作业。请注意，父作业构建将分阶段进行，如下图所示:

[![](img/3e1d8d5fda8dbaaffc073cbfe9aee152.png)](/web/20230103152204/https://www.baeldung.com/wp-content/uploads/2022/12/Screenshot-2022-12-15-at-11.48.13-AM.png)

在上面的输出中，我们可以看到阶段`build,`和`triggerChildJob` 都成功运行。

## 3.结论

在本文中，我们演示了如何在 Jenkins 中创建一个在内部触发另一个作业的作业。作为第一步，我们创建了一个样本 freestyle Jenkins 作业，然后创建了一个管道作业来内部触发它。

我们可以在 GitHub 上找到管道作业[的脚本。](https://web.archive.org/web/20230103152204/https://github.com/eugenp/tutorials/tree/master/jenkins-modules/jenkins-jobs)