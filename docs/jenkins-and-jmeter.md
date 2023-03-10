# 配置 Jenkins 运行并显示 JMeter 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/jenkins-and-jmeter>

## 1。概述

在本文中，我们将使用 [Jenkins](https://web.archive.org/web/20220524124829/https://jenkins.io/) 和 [Apache JMeter](https://web.archive.org/web/20220524124829/https://jmeter.apache.org/) 来配置一个连续交付管道。

我们将依赖于 JMeter 的文章作为一个很好的起点来首先理解 JMeter 的基础知识，因为它已经有一些我们可以运行的配置好的性能测试。并且，我们将使用该项目的构建输出来查看由 Jenkins [Performance](https://web.archive.org/web/20220524124829/https://plugins.jenkins.io/performance/) 插件生成的报告。

## 2。设置詹金斯

首先，我们需要下载最新稳定版本的[詹金斯](https://web.archive.org/web/20220524124829/https://jenkins.io/download/)，导航到我们的文件所在的文件夹，并使用`java -jar jenkins.war`命令运行它。

请记住，我们不能在没有初始用户设置的情况下使用 Jenkins。

## 3。安装性能插件

让我们安装`Performance`插件，这是运行和显示 JMeter 测试所必需的:

[![](img/ab3f97c9ee55f9162f5981a95f5ddb45.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/install-performance-plugin.png)

现在，我们需要记住重启实例。

## 4。与 Jenkins 一起运行 JMeter 测试

现在，让我们转到 Jenkins 主页，单击“创建新工作”，指定一个名称，选择`Freestyle project`，然后单击“确定”。

In the next step, on the `General` `Tab`, we can configure it with these general details:[![](img/cea26025f97f3d9225a6be4394bdc716.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/General_info_image.png)Next, let's set the repository URL and branches to build:[![](img/29f3e8cc270887e086cf3ff81cb03733.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Source_code_mangement.png)Now go to the `Build` `Tab` to specify how we'll build the project. Here instead of directly specified the Maven command to build the whole project, we can take another way to better have the control of our pipeline as the intent is just to build one module.

在`Execute shell` `Sub-tab`上，我们编写一个脚本，在存储库被克隆后执行必要的操作:

*   导航到所需的子模块
*   我们编辑了它
*   我们部署了它，知道它是一个基于 spring-boot 的项目
*   我们一直等到该应用程序在端口 8989 上可用
*   最后，我们只需指定用于性能测试的 JMeter 脚本的路径(位于`jmeter`模块的 resource 文件夹中)和同样位于 resource 文件夹中的结果文件的路径(`JMeter.jtl`

下面是相应的小型 shell 脚本:

```java
cd jmeter
./mvnw clean install -DskipTests
nohup ./mvnw spring-boot:run -Dserver.port=8989 &

while ! httping -qc1 http://localhost:8989 ; do sleep 1 ; done

jmeter -Jjmeter.save.saveservice.output_format=xml 
  -n -t src/main/resources/JMeter.jmx 
    -l src/main/resources/JMeter.jtl
```

如下图所示:

[![](img/cc2ddf9fd9b0085ca3f743f039774ee7.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Build_command.png)

从 GitHub 克隆项目后，我们编译它，在端口 8989 上打开并处理性能测试，我们需要使性能插件以用户友好的方式显示结果。

我们可以通过添加一个专用的`Post-build Actions`来实现。我们需要提供结果源文件并配置操作:

[![](img/c7ccaf302e747f670a1b52121e5a14e8.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Publish_performance_testresult_1.png)

我们选择带有后续配置的`Standard Mode`:

[![](img/3dfbca17f839e62db86263696cf45cbd.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Publish_performance_testresult_2.png)

让我们点击 Jenkins 仪表板左侧菜单上的`Save,`，点击按钮`Build Now` ，等待它完成我们在那里配置的一组操作。

完成后，我们将在控制台上看到项目的所有输出。最后我们会得到`Finished: SUCCESS` 或者`Finished: FAILURE`:

[![](img/14db0c66bb7bf0ee8737dddbde358c72.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Failed_build.png)

让我们通过左侧菜单进入`Performance Report` 区域。

这里我们将有包括当前版本在内的所有过去版本的报告，以查看性能方面的差异:

[![](img/b87891263c29c03280eb7e6ca96f63a3.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Performance_test_report.png)

让我们单击表格上方的指示，仅显示我们刚刚完成的最后一次构建的结果:

[![](img/744fb8f381874b336cb272437c553e84.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Performance_test_report_last.png)

从我们项目的仪表板中，我们可以得到`Performance Trend`，这是显示最后构建结果的其他图表:

[![](img/c6d0ca963ff324a0a9aaff124a9ada3e.png)](/web/20220524124829/https://www.baeldung.com/wp-content/uploads/2017/12/Performance_trend.png)

注意:对`Pipeline project`应用同样的东西也很简单:

1.  从仪表板创建另一个项目(条目)，并将其命名为`JMeter-pipeline`例如(`General info Tab`)
2.  选择`Pipeline` 作为项目类型
3.  在`Pipeline` `Tab`上，在定义上选择`Pipeline script`并勾选`Use Groovy Sandbox`
4.  在`script`区域，只需填写以下几行:

```java
node {
    stage 'Build, Test and Package'
    git 'https://github.com/eugenp/tutorials.git'

    dir('jmeter') {
        sh "./mvnw clean install -DskipTests"
        sh 'nohup ./mvnw spring-boot:run -Dserver.port=8989 &'
        sh "while ! httping -qc1
          http://localhost:8989 ; do sleep 1 ; done"

        sh "jmeter -Jjmeter.save.saveservice.output_format=xml
          -n -t src/main/resources/JMeter.jmx 
            -l src/main/resources/JMeter.jtl"
        step([$class: 'ArtifactArchiver', artifacts: 'JMeter.jtl'])
        sh "pid=\$(lsof -i:8989 -t); kill -TERM \$pid || kill -KILL \$pid"
    }
} 
```

该脚本从克隆项目开始，进入目标模块，编译并运行它，以确保可以在 http://localhost:8989 访问该应用程序

接下来，我们运行位于 resource 文件夹中的 JMeter 测试，将结果保存为构建输出，最后，应用程序关闭。

## 5。结论

在这篇简短的文章中，我们建立了一个简单的连续交付环境，以两种方式在`Jenkins`中运行和显示 Apache `JMeter`测试；第一个通过`Freestyle project`，第二个通过`Pipeline`。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524124829/https://github.com/eugenp/tutorials/tree/master/jmeter)