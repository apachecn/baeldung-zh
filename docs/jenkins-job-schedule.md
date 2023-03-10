# 在詹金斯安排工作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-job-schedule>

## 1。简介

在本文中，我们将介绍 Jenkins 中调度作业的各种方法。

我们将从调度一个简单的作业开始，该作业执行像打印纯文本消息这样简单的事情。我们将把这个例子发展成调度一个作业，这个作业是由诸如 GitHub、Bitbucket 等配置管理库中的变化自动触发的。

## 2。初始设置

我们假设 **JDK 和 Maven 已经分别以 JDK9.0.1 和 Maven3.5.2** 的名字安装在`Global Tool Configuration`的 Jenkins 服务器上。

它还假设**我们已经获得了对配置管理库**的访问权，比如正确设置了 Maven 项目的 Bitbucket。

## 3。安排一个简单的任务

在作业配置页面中，让我们直接向下滚动到`Build Triggers`部分。因为我们打算创建一个简单的作业，所以让我们选中标记为`Build periodically`的复选框。只要我们选中此复选框，就会显示一个带有`Schedule`标签的文本框。

我们必须以一种 [Cron 兼容格式](/web/20221223152245/https://www.baeldung.com/cron-expressions) 提供价值。如果我们单击方框旁边的问号，页面上会显示大量信息。

让我们在这里键入`*/2 * * * *`，它代表两分钟的间隔:

[![build triggers section](img/9df010148d96f06bac94f79791946e7f.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/build-triggers-section.png)

当 tab 键离开文本框时，我们可以看到文本框正下方的信息。它告诉我们作业下一次运行的时间。

让我们保存作业—大约两分钟后，我们应该会看到作业第一次执行的状态:

[![job status](img/13322dacbe46b1ccf66100be9b9dadc5.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/job-status-1.png)

因为我们已经将作业配置为每两分钟运行一次，所以当我们等待一段时间后返回到作业仪表板时，应该会看到多个内部版本号。

## 4。创建轮询 SCM 的作业

让我们向前迈进一步，创建一个从配置管理库(比如 Bitbucket)中提取源代码并执行构建的作业。

让我们按照上一节中的说明创建一个新作业，并做一些修改。

在`Build Triggers`部分，**而不是选择`Build Periodically`，让我们选择`Poll SCM`T5。一旦我们这样做了，我们应该会看到一个带有标签`Schedule`的文本框。**

让我们在此框中键入`*/5 * * * *`,这意味着我们希望调度作业每 5 分钟运行一次:

[![poll scm](img/189784bb261b6eb824ddac31a638877c.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/poll-scm.png)

让我们向上滚动到`Source Code Management`部分。选择`Git`旁边的单选按钮后，会出现一个标记为`Repositories`的新部分。

**这是我们需要配置配置库细节的地方**。让我们在`Repository URL`文本字段中键入 SCM 存储库的 URL:

[![poll scm source code management empty](img/047a3098ceaa466fe84d3c72406357e9.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/poll-scm-source-code-management-empty.png)

我们还需要提供用户凭证，以便 Jenkins 可以访问存储库。

让我们单击凭证旁边的`Add`按钮，这将显示一个弹出屏幕来创建用户凭证。

让我们选择`Kind`作为`Username with Password`。我们必须在指定的文本字段中键入用户名和密码:

[![AddUser](img/dfb14746a446741a2ec523143676a225.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/AddUser-1.png)

点击`Add`按钮，我们将回到`Source Code Management`部分。

让我们在`Credentials`旁边的下拉列表中选择该用户凭证:

[![SourceCodeManagement](img/8833b9cb5d7aae6e9c22c998b7e4191d.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/SourceCodeManagement-2.png)

现在，我们需要做的最后一件事是设置构建脚本。

让我们向下滚动到`Build`部分，点击`Add build step`并选择`Execute Shell`。由于我们正在 SCM 资源库**中处理一个 Maven 项目，我们需要键入`mvn clean install`，它将执行一个 Maven 构建。**

让我们试着理解我们在这里做了什么。

我们已经创建了一个计划每 5 分钟运行一次的作业。**该作业已被配置为从给定位存储库**的主分支提取源代码。

它将使用提供的用户凭据登录到 Bitbucket。

**提取源代码后，作业将执行包含提供的 Maven 命令的脚本。**

现在，如果我们保存并等待大约五分钟，我们应该在作业仪表板的`Build History`部分看到构建执行。

`Console Output`应该显示 Maven 构建的输出。我们可以在控制台输出中看到，源代码已经从 Bitbucket 中取出，并且命令`mvn clean install`已经执行:

[![maven console output](img/54eeb3966aa3ab8d1b86c5fd4f6cbfa6.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/maven-console-output-1-1.png)

由于是 Maven 版本，**控制台输出可能会很长，这取决于下载的 Maven 依赖项的数量**。

但是在输出结束时，我们应该会看到`BUILD SUCCESS`消息。

## 5。创建使用管道作为脚本的作业

到目前为止，我们已经看到了如何创建在预定义的调度时间执行的作业。

现在，让我们创建一个不绑定到任何特定计划的作业。取而代之的是，我们将配置它在 SCM 库中有新的提交时自动触发。

回到 Jenkins 仪表盘，让我们点击`New Item`。这次，我们将选择`Pipeline`，而不是`Freestyle project`。我们把这个工作命名为`PipelineAsScriptJob.`

单击“确定”按钮后，我们将进入管道配置页面。该页面有几个部分，如“`General”`、`Build Triggers”`、`Advanced Project Options”`、`Pipeline”`。

让我们向下滚动到“`Build Triggers”`部分和`Build when a change is pushed to Bitbucket`旁边的选择复选框。**这个选项只有在我们安装了 Bitbucket 插件**后才可用:

[![pipeline script build triggers section](img/2e3571c96284d336abd01ba1e42192f0.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-script-build-triggers-section.png)

让我们向下滚动到`Pipeline`部分。我们需要在`Definition`旁边的下拉菜单中选择`Pipeline Script`。

该下拉框正下方的文本框正在等待脚本被放入。做这件事有两种方法。

我们既可以键入整个脚本，也可以使用 Jenkins 提供的工具`Pipeline Syntax` 。

让我们选择第二个选项:

[![pipeline syntax link](img/54b4328777b24b5f4854be0ad58bbe97.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-syntax-link.png)

点击`Pipeline Syntax`，如上图中突出显示的，一个新的选项卡在浏览器中打开。这是一个方便的工具，我们可以在这里指定我们想要执行的操作，**这个工具将为我们生成 Groovy** 中的脚本。然后，我们可以复制脚本并将其粘贴到管道配置中。

让我们在`Sample Step`下拉列表中选择`checkout: General SCM`。提供 SCM Repo URL 和用户凭证后，我们需要单击`Generate Pipeline Script`按钮。

这将在文本框中生成脚本:

[![pipeline script checkout syntax](img/f058e8eecbfdcf9e0b2b726e2dc437f7.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-script-checkout-syntax-1.png)

让我们复制这个脚本，并将其保存在某个地方供以后使用。

在同一页面上，让我们向上滚动并在`Sample Step`下拉列表中选择`withMaven: Provide Maven environment`。这里必须注意的是，**这个选项只有在安装了 Pipeline Maven 集成插件**的情况下才可用。

我们需要在相应的下拉框旁边选择 Maven 和 JDK 安装的名称。我们需要点击`Generate Pipeline Script`按钮。

这将在文本框中生成脚本:

[![pipeline script withMaven syntax](img/c7e7a5d23c3dceb2636ae98d72e0931e.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-script-withMaven-syntax.png)

让我们保存脚本。

我们还需要生成一些脚本。让我们在`Sample Step`下拉列表中选择`node: Allocate node`并在`Label`文本框中输入`master`然后点击`Generate Pipeline Script`:

[![pipeline node syntax](img/18dc17c445683c343129398aca26a5aa.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-node-syntax.png)

让我们保存脚本。

也是最后一个。

让我们在`Sample Step`下拉框中选择`stage: Stage`，在`Stage Name`文本框中输入`scm`，然后点击`Generate Pipeline Script`:

[![pipeline script stage syntax](img/5c24ba8a8a4eb2fd741b1d8899fd5871.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-script-stage-syntax-1.png)

还是省省吧。

是时候整理到目前为止生成的所有脚本并将它们缝合在一起了:

```java
node('master') {
    stage('scm') {
        checkout([$class: 'GitSCM', 
          branches: [[name: '*/master']], 
          doGenerateSubmoduleConfigurations: false, 
          extensions: [], 
          submoduleCfg: [], 
          userRemoteConfigs: [[credentialsId: 'e50f564f-fbb7-4660-9c95-52dc93fa26f7', 
          url: 'https://[[email protected]](/web/20221223152245/https://www.baeldung.com/cdn-cgi/l/email-protection)/projects/springpocrepo.git']]])
    }

    stage('build') {
        withMaven(jdk: 'JDK9.0.1', maven: 'Maven3.5.2') {
            sh 'mvn clean install'
        }
    }
}
```

上面脚本中的第一条语句`node(‘master')` ，**表示作业将在名为`master`** 的节点上执行，该节点是 Jenkins 服务器上的默认节点。

让我们将上面的脚本复制到`Pipeline`部分的文本框中:

[![pipeline script all together](img/72bcfbc37eb02196a70d495dc7178edb.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-script-all-together.png)

让我们保存这个配置。

现在，只剩下一件事了。我们需要在 Bitbucket 中配置一个 Webhook。每当 Bitbucket 中发生特定事件时，Webhook 负责向 Jenkins 服务器发送推送通知。

我们来看看如何配置。

让我们登录到 Bitbucket 并选择一个存储库。我们应该在左侧栏看到一个选项`Settings`。让我们点击它，我们应该在`WORKFLOW`部分看到一个选项`Webhooks`。让我们创建一个新的 Webhook。

我们需要在`Title`字段中为这个 webhook 提供一些名称。我们还需要在`URL`字段中为 webhook 提供一个 URL。这个 URL 需要指向一个由 Jenkins 提供的特定 REST API 端点，这个端点就是`bitbucket-hook`。

**必须注意，URL 必须以斜杠**结尾:

[![Webhook](img/442bdae8d03349688c3f49fffaea5b97.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/Webhook-1-1.png)

在配置上面的 Webhook 时，我们选择了选项`Repository push`。**这意味着每当推送发生时，Bitbucket 会向 Jenkins 服务器发送通知**。

这种行为是可以改变的；有几个不同的选项可供选择。

现在，让我们使用默认选项，即`Repository Push`。

**我们也可以在 Github 中设置 web hook**；[这里有一些关于如何配置的有用信息。](https://web.archive.org/web/20221223152245/https://docs.github.com/en/developers/webhooks-and-events/about-webhooks)

长话短说:我们已经用 Groovy 语言创建了一个管道脚本—**,它应该从所提供的 SCM 库的主分支**(使用所提供的用户凭证)中提取源代码，每当库中有推送时**,然后**在 Jenkins 服务器**上执行`mvn clean install`命令。**

必须注意的是，这个作业不会在任何特定的时间运行。取而代之的是，它将等待在配置管理库的主分支中有一个推送。

只要有新的推送事件，我们就会在 Jenkins 作业仪表板的`“Build History”`部分看到新的执行。

我们应该看到一个新的构建被启动，旁边有一个`pending`状态。

几秒钟后，这个构建将开始执行，我们将能够在`Console Output`中看到完整的日志。

**控制台输出中的第一条语句说“由 Bitbucket push by 启动..”–**这意味着当 Bitbucket 中发生推送时，会自动触发构建:

[![pipeline script console output](img/d481d14f3b1cea83c7db270257c4c020.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/pipeline-script-console-output-1-1.png)

如果一切顺利，构建应该会成功完成。

## 6。创建一个使用 Jenkinsfile 的作业

可以不在 Jenkins 管道中编写任何脚本，仍然实现由 Bitbucket Webhook 触发的构建执行。

为此，**我们必须在 Bitbucket 中创建一个新文件，并将其命名为`Jenkinsfile`** 。管道脚本需要稍微修改一下就转移到这个 Jenkinsfile。让我们来看看到底该怎么做。

我们将在 Jenkins 中创建一个新的管道，并将其命名为`PipelineWithJenkinsfile`。

在管道配置页面上，我们将选择`Pipeline`部分中`Definition`旁边的`Pipeline script from SCM`。我们将在`SCM`旁边看到一个带有不同选项的下拉菜单。让我们从下拉框中选择`Git`。

然后，我们必须提供 Bitbucket 存储库的 URL 和用户凭证。让我们确保`Script Path`旁边的文本字段**包含默认值，即`Jenkinsfile`** :

[![jenkinsfile pipeline config 2](img/289ddc80a25a50938334c730e8b2c8bd.png)](/web/20221223152245/https://www.baeldung.com/wp-content/uploads/2018/01/jenkinsfile-pipeline-config-2.png)

就 Jenkins 而言，这就是我们需要配置的全部内容。

但是，我们必须在存储库中创建 Jenkinsfile 因此，让我们创建一个新的文本文件，将其命名为 Jenkinsfile，并使用这个简单的 groovy 脚本:

```java
node('master') {
    stage('scm') {
        checkout scm
    }
    stage('build') {
        withMaven(jdk: 'JDK9.0.1', maven: 'Maven3.5.2') {
            sh 'mvn clean install'
        }
    }
}
```

该脚本与我们在前面部分创建的管道脚本几乎相同，只有一处修改。`stage(‘scm')`中的语句不需要 URL 和用户凭证信息。相反，它所需要的只是`checkout scm`。

仅此而已。**当我们将这个文件提交到 Bitbucket 时，它将触发 Jenkins** 中的构建——我们应该会看到构建在`Build History`中被触发。

这一节与前一节的唯一区别是**我们在 Bitbucket 存储库中定义了管道脚本。**

因此，**构建脚本是需要构建的项目的源代码的一部分**。我们并没有在詹金斯身上保留剧本。

相反，Jenkins 知道配置管理库的细节和脚本文件。**每当有推送到这个库的时候，`Jenkinsfile`中的脚本就会在詹金斯服务器**上执行。

## 7。结论

在本教程中，我们看到了如何使用各种策略在 Jenkins 中配置和调度作业。

我们还看到了如何在 Jenkins 中配置一个作业，以便根据在诸如 Bitbucket 之类的 SCM 存储库中执行的某些操作来自动触发它。