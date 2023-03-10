# 加特林简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-gatling>

## 1。概述

Gatling 是一个**负载测试工具**，它提供了对`HTTP`协议的出色支持——这使得它成为负载测试任何`HTTP`服务器的一个非常好的选择。

这个快速指南将向你展示如何**设置一个简单的场景**来对`HTTP`服务器进行负载测试。

加特林**的模拟脚本是用`Scala`** 编写的，但是不要担心——这个工具会通过一个 GUI 来帮助我们记录场景。一旦我们完成了场景的记录，GUI 就会创建代表模拟的`Scala`脚本。

运行模拟后，我们有一个现成的 **`HTML`报告**。

最后但同样重要的是，加特林的架构**是异步的**。这种架构允许我们将虚拟用户实现为消息，而不是专用线程，这使得它们非常便宜。因此，运行数千个并发虚拟用户不成问题。

同样值得注意的是，核心引擎实际上是**协议不可知的**，所以实现对其他协议的支持是完全可能的。例如，加特林目前也提供`JMS`支持。

## 2。使用原型创建项目

一个l 虽然我们可以得到加特林捆绑作为`.zip`我们选择使用加特林的`Maven Archetype`。这允许我们集成加特林，并将其运行到一个`IDE`中，并使得在版本控制系统中维护项目变得容易。小心，因为加特林**需要 JDK8** 。

在命令行中，键入:

```java
mvn archetype:generate
```

然后，出现提示时:

```java
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains):
```

类型:

```java
gatling
```

然后，您应该会看到:

```java
Choose archetype:
1: remote -> 
  io.gatling.highcharts:gatling-highcharts-maven-archetype (gatling-highcharts-maven-archetype)
```

类型`:`

```java
1
```

选择原型，然后选择要使用的版本(选择最新版本)。

在确认原型创建之前，为类选择 `groupId`、`artifactId`、`version`和`package`名称。

最后将原型导入 IDE——例如导入 [Scala IDE](https://web.archive.org/web/20220521212150/https://github.com/scala-ide/scala-ide) (基于 Eclipse)或者导入 [IntelliJ IDEA](https://web.archive.org/web/20220521212150/https://www.jetbrains.com/idea/) 。

## 3。定义一个场景

在启动记录器之前，我们需要**定义一个场景**。它将代表用户浏览 web 应用程序时的真实情况。

在本教程中，我们将使用 Gatling 团队提供的应用程序作为示例，该应用程序位于 URL[http://computer-database . Gatling . io](https://web.archive.org/web/20220521212150/http://computer-database.gatling.io/)。

我们的简单场景可能是:

*   用户到达应用程序。
*   用户搜索“amstrad”。
*   用户打开一个相关的模型。
*   用户返回主页。
*   用户遍历页面。

## 4。配置记录仪

首先从 IDE 中启动`Recorder`类。启动后，GUI 允许您配置如何记录请求和响应。选择以下选项:

*   `8000` 作为监听端口
*   `org.baeldung.simulation` 包装
*   `RecordedSimulation`班级名称
*   `Follow Redirects?`已检查
*   `Automatic Referers?`已检查
*   `Black list first`选择过滤策略
*   黑名单过滤器中的`.*\.css`、`.*\.js`和`.*\.ico`

[![setting](img/a52132c2f14ce0ae1ea964191516c15a.png)](/web/20220521212150/https://www.baeldung.com/wp-content/uploads/2016/06/setting-1024x576.png)

现在我们必须配置我们的浏览器，以使用在配置过程中选择的已定义端口(`8000`)。这是我们的浏览器必须连接的端口，以便`Recorder`能够捕获我们的导航。

以下是如何使用 Firefox，打开浏览器高级设置，然后进入网络面板并更新连接设置:

[![proxy settings](img/8670212dbcf29dc560b4433436c2ba5a.png)](/web/20220521212150/https://www.baeldung.com/wp-content/uploads/2016/06/settings.png)

## 5。录制场景

现在一切都配置好了，我们可以记录我们上面定义的场景。步骤如下:

1.  点击“开始”按钮开始录制
2.  进入网址:[http://computer-database . Gatling . io](https://web.archive.org/web/20220521212150/http://computer-database.gatling.io/)
3.  搜索名称中带有“amstrad”的型号
4.  选择“Amstrad CPC 6128”
5.  返回主页
6.  点击`Next`按钮，在模型页面中迭代几次
7.  点击“停止并保存”按钮

仿真将在配置期间定义的包`org.baeldung`中以名称`RecordedSimulation.scala`生成

## 6。用 Maven 运行模拟

要运行我们记录的模拟，我们需要更新我们的`pom.xml`:

```java
<plugin>
    <groupId>io.gatling</groupId>
    <artifactId>gatling-maven-plugin</artifactId>
    <version>2.2.4</version>
    <executions>
        <execution>
            <phase>test</phase>
            <goals><goal>execute</goal></goals>
            <configuration> 
                <disableCompiler>true</disableCompiler> 
            </configuration>
        </execution>
    </executions>
</plugin>
```

这让我们在测试阶段执行模拟。要开始测试，只需运行:

```java
mvn test
```

模拟完成后，控制台将显示 HTML 报告的路径。

注意:使用配置`<disableCompiler>true</disableCompiler>`是因为我们将使用 Scala 和 maven，这个标志将确保我们不会两次编译我们的模拟。更多细节请见[加特林文档](https://web.archive.org/web/20220521212150/https://gatling.io/docs/current/extensions/maven_plugin/#coexisting-with-scala-maven-plugin)。

## 7 .**。查看结果**

如果我们在建议的位置打开`index.html`，报告如下所示:

[![reports](img/a1c1ac7142596bb62dfa5fd3b9c3cf0d.png)](/web/20220521212150/https://www.baeldung.com/wp-content/uploads/2016/06/reports-1024x486.png)

## 8。结论

在本教程中，我们已经探索了 使用 Gatling 对 HTTP 服务器进行负载测试。这些工具允许我们在 GUI 界面 的帮助下，基于定义的场景 记录模拟。录制完成后，我们可以开始测试。测试报告将以 HTML 简历的形式提交。

为了构建我们的示例，我们选择使用 maven 原型。这有助于我们 将加特林集成并运行到一个 IDE 中，使得在版本控制系统 中维护项目变得容易。

示例代码可以在 [GitHub 项目](https://web.archive.org/web/20220521212150/https://github.com/eugenp/tutorials/tree/master/testing-modules/gatling)中找到。