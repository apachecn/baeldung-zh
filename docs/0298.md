# Jenkins 2 简介和管道的力量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-pipelines>

## 1。概述

在本文中，我们将通过一个使用 [Jenkins](https://web.archive.org/web/20220524124219/https://jenkins.io/) 的连续交付的例子来展示管道的用法。

我们将为我们的示例项目构建一个简单但非常有用的管道:

*   汇编
*   简单的静态分析(与编译并行)
*   单元测试
*   集成测试(与单元测试并行)
*   部署

## 2。设置詹金斯

首先，我们需要下载 [Jenkins](https://web.archive.org/web/20220524124219/https://jenkins.io/download/) 的最新稳定版本(撰写本文时为 2.73.3)。

让我们导航到我们的文件所在的文件夹，并使用`java -jar jenkins.war`命令运行它。**请记住，没有初始用户设置，我们无法使用 Jenkins。**

使用管理员生成的初始密码解锁 Jenkins 后，我们必须填写第一个管理员用户的个人资料信息，并确保安装所有推荐的插件。

[![](img/c9a167ba511f4d85b189b0dbbffb2966.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins1.png)

现在我们有一个新安装的 Jenkins 可以使用了。

[![](img/3c83b61220bce51f653118b7474b83af.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins2.png)

Jenkins 的所有可用版本都可以在这里找到。

## 3。管道

Jenkins 2 附带了一个很棒的特性叫做`Pipelines`，当我们需要为一个项目定义一个持续集成环境的时候，这个特性是非常可扩展的。

**管道是使用代码**定义一些 Jenkins 步骤的另一种方式，并且自动化部署软件的过程。

它使用具有两种不同语法的[领域特定语言(DSL)](https://web.archive.org/web/20220524124219/https://jenkins.io/doc/book/pipeline/syntax/) :

*   声明管道
*   脚本化管道

在我们的例子中，我们将使用**和`Scripted Pipeline`，它们遵循用 Groovy** 构建的更强制性的编程模型。

让我们来看看`Pipeline`插件的一些特征:

*   管道被写入文本文件，并被视为代码；这意味着可以将它们添加到版本控制中，并在以后进行修改
*   它们将在 Jenkins 服务器重启后保留
*   我们可以选择暂停管道
*   它们支持复杂的需求，例如并行执行工作
*   管道插件也可以扩展或与其他插件集成

换句话说，建立一个管道项目意味着编写一个脚本，该脚本将顺序地应用我们想要完成的过程的一些步骤。

为了开始使用管道，我们必须安装[管道](https://web.archive.org/web/20220524124219/https://plugins.jenkins.io/workflow-aggregator/)插件，它允许组合简单和复杂的自动化。

我们可以选择拥有一个[管道阶段视图](https://web.archive.org/web/20220524124219/https://plugins.jenkins.io/pipeline-stage-view/),这样当我们运行一个构建时，我们将会看到我们已经配置的所有阶段。

## 4。一个简单的例子

对于我们的例子，我们将使用一个小的 Spring Boot 应用程序。然后我们将创建一个管道来克隆项目，构建它并运行几个测试，然后运行应用程序。

让我们安装`**[Checkstyle](https://web.archive.org/web/20220524124219/https://www.jenkins.io/doc/pipeline/steps/checkstyle/)**,` [**静态`Analysis`收集器**](https://web.archive.org/web/20220524124219/https://www.jenkins.io/doc/pipeline/steps/analysis-collector/) 和`[**JUnit**](https://web.archive.org/web/20220524124219/https://plugins.jenkins.io/junit/)`插件，它们分别用于收集`Checkstyle`结果，构建测试报告的组合分析图，并显示成功执行和失败的测试。

首先，让我们在这里了解一下 Checkstyle 的原因:它是一个开发工具，可以帮助程序员按照公认的、众所周知的标准编写更好的 Java 代码。

静态分析收集器是一个附加组件，它收集不同的分析输出，并在一个组合的趋势图中打印结果。此外，该插件提供健康报告，并基于这些分组结果构建稳定性。

最后，`JUnit`插件提供了一个发布器，它使用构建期间生成的 XML 测试报告，并输出与项目测试相关的详细且有意义的信息。

我们还将在应用程序的`pom.xml:`中配置`Checkstyle`

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>2.17</version>
</plugin>
```

## 5。创建管道脚本

首先，我们需要创建一个新的詹金斯职位。让我们确保在点击确定按钮之前选择`Pipeline`作为类型，如该屏幕截图所述:

[![](img/56c686b2fa58d874a3b0634c20485985.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins3.png)

下一个屏幕允许我们填写 Jenkins 工作的不同步骤的更多细节，例如`description`、`triggers`和一些`advanced project options:`

[![](img/cd984ee2445b516c948cd66987971a2d.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins4.png)

让我们点击`Pipeline`选项卡，深入了解这类工作的主要和最重要的部分。

然后，对于定义，选择`Pipeline script`并勾选`Use Groovy Sandbox.`

以下是 Unix 环境的工作脚本:

```
node {
    stage 'Clone the project'
    git 'https://github.com/eugenp/tutorials.git'

    dir('spring-jenkins-pipeline') {
        stage("Compilation and Analysis") {
            parallel 'Compilation': {
                sh "./mvnw clean install -DskipTests"
            }, 'Static Analysis': {
                stage("Checkstyle") {
                    sh "./mvnw checkstyle:checkstyle"

                    step([$class: 'CheckStylePublisher',
                      canRunOnFailed: true,
                      defaultEncoding: '',
                      healthy: '100',
                      pattern: '**/target/checkstyle-result.xml',
                      unHealthy: '90',
                      useStableBuildAsReference: true
                    ])
                }
            }
        }

        stage("Tests and Deployment") {
            parallel 'Unit tests': {
                stage("Runing unit tests") {
                    try {
                        sh "./mvnw test -Punit"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults: 
                          '**/target/surefire-reports/TEST-*UnitTest.xml'])
                        throw err
                    }
                   step([$class: 'JUnitResultArchiver', testResults: 
                     '**/target/surefire-reports/TEST-*UnitTest.xml'])
                }
            }, 'Integration tests': {
                stage("Runing integration tests") {
                    try {
                        sh "./mvnw test -Pintegration"
                    } catch(err) {
                        step([$class: 'JUnitResultArchiver', testResults: 
                          '**/target/surefire-reports/TEST-' 
                            + '*IntegrationTest.xml'])
                        throw err
                    }
                    step([$class: 'JUnitResultArchiver', testResults: 
                      '**/target/surefire-reports/TEST-' 
                        + '*IntegrationTest.xml'])
                }
            }

            stage("Staging") {
                sh "pid=\$(lsof -i:8989 -t); kill -TERM \$pid " 
                  + "|| kill -KILL \$pid"
                withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
                    sh 'nohup ./mvnw spring-boot:run -Dserver.port=8989 &'
                }   
            }
        }
    }
}
```

首先，我们从 GitHub 克隆存储库，然后将目录更改为我们的项目，名为`spring-jenkins-pipeline.`

接下来，我们编译项目，并以并行的方式应用`Checkstyle`分析。

以下步骤代表单元测试和集成测试的并行执行，然后是应用程序的部署。

**并行性用于优化流水线，使作业运行得更快。**在 Jenkins 中，同时运行一些独立的动作是一种最佳实践，这可能会花费很多时间。

例如，在一个现实世界的项目中，我们通常有许多单元和集成测试，这些测试可能需要更长的时间。

请注意，如果任何测试失败，构建也将被标记为失败，部署将不会发生。

此外，我们使用 `JENKINS_NODE_COOKIE`来防止应用程序在管道到达终点时立即关闭。

要查看在其他不同系统上运行的更通用的脚本，请查看 [GitHub 库](https://web.archive.org/web/20220524124219/https://github.com/eugenp/tutorials/blob/master/spring-jenkins-pipeline/scripted-pipeline-unix-nonunix)。

## 6。分析报告

创建作业后，我们将保存我们的脚本，并在 Jenkins 仪表板的项目主页上点击`Build Now`。

以下是构件的概述:

[![](img/8aa0cda3e6e4fbdf47763d4d266823b1.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins5.png)

再往下一点，我们将看到管道的阶段视图，以及每个阶段的结果:

[![](img/9c83bf6e674d617ca55cd622acbcfbfd.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins6.png)

将鼠标悬停在阶段单元格上并单击`Logs`按钮以查看该步骤中打印的日志消息时，可以访问每个输出。

我们还可以找到代码分析的更多细节。让我们从右侧菜单的`Build History`中点击想要的构建，然后点击`Checkstyle Warnings.`

这里我们看到 60 个高优先级警告，可通过单击浏览:

[![](img/0d0407119aeefe9e69cac9a24bfb9923.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins7.png)

`Details`选项卡显示了一些信息，这些信息突出显示了警告，并使开发人员能够理解警告背后的原因。

同样，点击`Test Result`链接，可获得完整的测试报告。让我们看看`com.baeldung`包的结果:

[![](img/54f946321316810782b9ff536d4e0a06.png)](/web/20220524124219/https://www.baeldung.com/wp-content/uploads/2017/12/jenkins8.png)

在这里，我们可以看到每个测试文件及其持续时间和状态。

## 7 .**。结论**

在本文中，我们建立了一个简单的连续交付环境，通过一个`Pipeline`任务在 Jenkins 中运行并显示静态代码分析和测试报告。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524124219/https://github.com/eugenp/tutorials/tree/master/spring-jenkins-pipeline)