# 蹦床——本地管理 Spring Boot 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/trampoline>

## 1。蹦床概述

从历史上看，理解我们的系统在运行时的状态的一个简单方法是在终端中手动运行它。最好的情况是，我们使用脚本来自动化一切。

当然，DevOps 运动改变了这一切，幸运的是，我们的行业已经超越了这种方式。 **[Trampoline](https://web.archive.org/web/20220921064804/https://github.com/ErnestOrt/Trampoline) 是 Java 生态系统中解决这个问题(针对 Unix 和 Windows 用户)的解决方案之一。**

该工具建立在 Spring Boot 之上，**旨在通过干净清新的用户界面来帮助 Spring Cloud 开发者的日常开发工作。**

以下是它的一些功能:

*   使用 Gradle 或 Maven 作为构建工具启动实例
*   管理 Spring Boot 实例
*   在启动阶段配置虚拟机参数
*   监控部署的实例:内存使用、日志和跟踪
*   向作者提供反馈

在这篇安静的文章中，我们将回顾蹦床渴望解决的问题，以及在实践中看看它。我们将进行一次有指导的旅行，包括注册一个新服务和启动该服务的一个实例。

[![TrampolineUI 2](img/633dfc6dec7a3eabcb579886121dea76.png)](/web/20220921064804/https://www.baeldung.com/wp-content/uploads/2017/09/TrampolineUI_2.png)

## 2。微服务:单一部署失效

正如我们所讨论的，使用单个部署单元部署应用程序的时代已经一去不复返了。

这有积极的后果，但不幸的是，也有消极的后果。虽然 Spring Boot 和 Spring Cloud 在这一过渡中提供了帮助，但我们需要注意一些副作用。

从单片到微服务的旅程极大地改善了开发人员构建应用的方式。

众所周知，打开一个有 30 个类的项目，在包之间有良好的结构，并有相应的单元测试，这与打开一个有大量类的庞大代码库是不同的，在那里事情很容易变得复杂。

不仅如此——可重用性、解耦以及关注点的分离也从这种发展中受益。虽然好处众所周知，但还是让我们列举一些吧:

*   单一责任原则——在可维护性和测试方面很重要
*   弹性——一项服务中的故障不会影响其他服务
*   高可扩展性——要求苛刻的服务可以部署在多个实例中

但是，当使用微服务架构时，我们必须面对一些权衡，特别是关于网络开销和部署。

然而，专注于部署，**我们失去了 monolith 的一个优势——单一部署**。为了在生产环境中解决这个问题，我们有一整套 CD 工具，这些工具将会有所帮助，并使我们的生活变得更加轻松。

## 3。蹦床:设置第一个服务

在这一部分，我们将在 Trampoline 中注册一个服务，并展示所有可用的特性。

### 3.1。下载最新版本

去蹦床库，在[发布部分](https://web.archive.org/web/20220921064804/https://github.com/ErnestOrt/Trampoline/releases)，我们将能够下载最新发布的版本。

然后，开始蹦床，例如使用`mvn spring-boot:run` `or` `./gradlew` (或 `gradle.bat` ) `bootRun`。

最后可以在 [http://localhost:8080](https://web.archive.org/web/20220921064804/http://localhost:8080/) 访问 UI。

### 3.2。注册服务

一旦我们有了蹦床和运行`,` 让我们去`Settings` 部分，在那里我们将能够注册我们的第一个服务。我们会在蹦床源代码中找到两个微服务的例子:`microservice-example-gradle`和`microservice-example-maven.`

注册服务需要以下信息:`name*`、`default port*`、`pom or build location*`、`build tool*`、`actuator prefix,`和`VM default arguments`。

如果我们决定使用 Maven 作为构建工具，首先我们必须设置我们的 Maven 位置。然而，如果我们决定使用 Gradle 包装器，它必须放在我们的`microservices`文件夹中。其他什么都不需要。

在本例中，我们将设置两者:

[![trampolinev3 3](img/fec1623eb7c9df5f24a200eb5dfd3775.png)](/web/20220921064804/https://www.baeldung.com/wp-content/uploads/2017/09/trampolinev3_3.png)

在任何时候，我们都可以通过点击`info`按钮查看服务信息，或者点击`trash`按钮删除服务信息。

最后，为了能够享受所有这些特性，唯一的要求是在我们的 Spring Boot 项目中包括`actuator starter`(参见示例片段)，以及通过众所周知的日志记录属性的`/logfile`端点:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 3.3。管理服务实例

现在，我们准备转到`Instances` 部分。在这里，我们将能够启动和停止服务实例，还可以监控它们的状态、跟踪、日志和内存消耗。

对于本教程，我们将启动之前注册的每个服务的一个实例:

[![trampoline instances](img/3f6c76db883f02e42d7cf5cb4d93cf3a.png)](/web/20220921064804/https://www.baeldung.com/wp-content/uploads/2017/09/trampoline-instances.png)

### 3.4。仪表板

最后，让我们快速浏览一下`Dashboard` 部分。在这里，我们可以可视化一些统计数据，如我们的计算机或注册或启动的服务的内存使用情况。

我们还可以看到在 settings 部分是否引入了必需的 Maven 信息:

[![trampoline dashboard](img/62a18604ed6976f817c98c6a6a1fc273.png)](/web/20220921064804/https://www.baeldung.com/wp-content/uploads/2017/09/trampoline_dashboard.png)

### 3.5。反馈

最后但同样重要的是，我们可以找到一个`Feedback`按钮，它重定向到 GitHub repo，在这里可以创建问题或提出疑问和改进。

[![trampoline feedback](img/ad611b291d5f2795686befc203d67ff5.png)](/web/20220921064804/https://www.baeldung.com/wp-content/uploads/2017/09/trampoline_feedback.png)

## 4。结论

在本教程的过程中，我们讨论了蹦床旨在解决的问题。

我们还展示了它的功能概述，以及一个关于如何注册服务和如何监控它的简短教程。

最后，请注意**这是一个开源项目**，欢迎您做出贡献。