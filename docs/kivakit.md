# KivaKit 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kivakit>

## 1.概观

KivaKit 是一个模块化的 Java 应用程序框架，旨在使微服务和应用程序的开发更快更容易。KivaKit 从 2011 年开始在 [Telenav](https://web.archive.org/web/20220625162908/https://www.telenav.com/) 研发。现在可以在 [GitHub](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit) 上获得 Apache 许可的开源项目。

在本文中，我们将探讨 KivaKit 的设计，它是一组协同工作的“迷你框架”。此外，我们将了解每个小型框架的基本特性。

## 2.KivaKit 微型框架

在 [`kivakit`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit) 和 [`kivakit-extensions`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions) 库中，我们可以看到 KivaKit 1.0 包含了 54 个模块。我们会觉得这是压倒性的。然而，如果我们一步一步来，事情并没有那么糟糕。首先，我们可以挑选我们想要包含在项目中的内容。KivaKit 中的每个模块都可以独立使用。

一些 KivaKit 模块包含迷你框架。迷你框架是一种简单、抽象的设计，可以解决常见的问题。如果我们研究 KivaKit 的迷你框架，我们会发现它们具有简单明了、广泛适用的接口。因此，它们有点像 Legos。也就是说，它们是扣合在一起的简单部件。

在这里，我们可以看到 KivaKit 的微型框架以及它们之间的关系:

[![](img/7c2ffe4c0e39bb50e431681b84564e6b.png)](/web/20220625162908/https://www.baeldung.com/wp-content/uploads/2021/09/k1.svg)

  
| 微型框架 | 组件 | 描述 |
| --- | --- | --- |
| 应用 | [kiva kit-应用程序](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-application) | 应用程序和服务器的基本组件 |
| 命令行解析 | kivakit-command line | 使用转换和验证小型框架进行开关和参数解析 |
| 成分 | [kiva kit-组件](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-component) | 实现 KivaKit 组件的基本功能，包括应用程序 |
| 转换 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) | 实现健壮的模块化类型转换器的抽象 |
| 提取，血统 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) | 从数据源中提取对象 |
| 接口 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) | 用作框架间集成点的通用接口 |
| 记录 | [kivakit-kernel](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel)
kivakit-logs-* | 核心日志功能、日志服务提供者接口(SPI)和日志实现 |
| 信息发送 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) | 使组件能够传输和接收状态信息 |
| 混合蛋白 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) | 有状态[特征](https://web.archive.org/web/20220625162908/https://en.wikipedia.org/wiki/Trait_(computer_programming))的实现 |
| 资源 | [kiva kit-资源](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-resource)
[kiva kit-网络-*](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-network)
[kiva kit-文件系统-*](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions/tree/1.0.0/kivakit-filesystems) | 文件、文件夹和流式资源的抽象 |
| 服务定位器 | [kiva kit-配置](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-configuration) | 用于查找组件和设置信息的[服务定位器模式](https://web.archive.org/web/20220625162908/https://martinfowler.com/articles/injection.html)的实现 |
| 设置 | [kiva kit-配置](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-configuration) | 提供对组件配置信息的轻松访问 |
| 确认 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) | 检查对象一致性的基本功能 |

我们可以在`kivakit`存储库中找到这些框架。另一方面，我们会在 [`kivakit-extensions`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions) 中找到不太重要的模块，比如服务提供商。

### 2.1.信息发送

如上图所示，消息传递是集成的中心点。**在 KivaKit 中，消息传递使状态报告正式化**。作为 Java 程序员，我们习惯于记录状态信息。有时，我们也会看到这些信息被返回给调用者或者作为异常抛出。相比之下，KivaKit 中的状态信息包含在`[Message](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/messaging/Message.html)s.`中，我们可以编写组件[来广播](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/messaging/Broadcaster.html)这些消息。此外，我们可以编写组件让[监听](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/messaging/Listener.html)它们。

我们可以看到，这种设计允许组件一致地关注报告状态。对于一个组件来说，状态消息放在哪里并不重要。在一种情况下，我们可能会将消息发送给一个[记录器](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/logging/Logger.html)。在另一种情况下，我们可能会将它们包含在统计数据中。我们甚至可以向最终用户展示它们。组件不关心。它只是向任何可能感兴趣的人报告问题。

广播消息的 KivaKit 组件可以连接到一个或多个消息[中继器](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/messaging/Repeater.html)，形成一个监听器链:

[![](img/9bff03044b954bf7036e2455ae672d82.png)](/web/20220625162908/https://www.baeldung.com/wp-content/uploads/2021/09/k2.svg)

在 KivaKit 中，一个 [`Application`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.application/com/telenav/kivakit/application/Application.html) 通常是一个监听器链的末端。因此，`Application`类记录它收到的任何消息。在这个过程中，侦听器链中的组件可能对这些消息有其他用途。

### 2.2.混合蛋白

KivaKit 的另一个集成特性是 mixins 迷你框架。KivaKit mixins 允许基类通过接口继承“混合”到类型中。有时，mixins 被称为“[状态特征](https://web.archive.org/web/20220625162908/https://en.wikipedia.org/wiki/Trait_(computer_programming))”。

例如，KivaKit 中的 [`BaseComponent`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.component/com/telenav/kivakit/component/BaseComponent.html) 类提供了构建组件的基础。`BaseComponent`提供发送消息的便捷方式。此外，它还提供了对资源、设置和注册对象的轻松访问。

但是我们很快就遇到了这个设计的问题。我们知道，在 Java 中，一个已经有基类的类也不能扩展`BaseComponent`。KivaKit mixins 允许将`BaseComponent`功能添加到已经有基类的组件中。例如:

```
public class MyComponent extends MyBaseClass implements ComponentMixin { [...] }
```

我们在这里可以看到，接口 [`ComponentMixin`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.component/com/telenav/kivakit/component/ComponentMixin.html) 既扩展了 [`Mixin`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/language/mixin/Mixin.html) 又扩展了 [`Component`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.component/com/telenav/kivakit/component/Component.html) :

[![](img/33b711c2dcbedce56a46468bcc2c57d3.png)](/web/20220625162908/https://www.baeldung.com/wp-content/uploads/2021/09/k3.svg)

`Mixin` 接口首先向`ComponentMixin.` 提供了一个`state()`方法，该方法用于创建一个`BaseComponent` 并将其与实现`ComponentMixin`的对象关联起来。其次，`ComponentMixin`将`Component`接口的每个方法实现为 Java 默认方法。第三，每个默认方法以这种方式委托给相关的`BaseComponent.`，实现`ComponentMixin`提供了与扩展`BaseComponent`相同的方法。

### 2.3.服务定位器

**服务定位器类注册中心允许我们将组件连接在一起**。A `[Registry](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.configuration/com/telenav/kivakit/configuration/lookup/Registry.html)`提供了与依赖注入(DI)大致相同的功能。然而，它在一个重要方面不同于 DI 的典型用法。在服务定位器模式中，组件接触它们需要的接口。另一方面，DI 将接口推入组件。因此，服务定位器方法改进了封装。也缩小了参考的范围。例如，`Registry`可以在一个方法中灵活使用:

```
class MyData extends BaseComponent {

    [...]

    public void save() {
        var database = require(Database.class);
        database.save(this);
    }
}
```

`BaseComponent`。`require(Class)`方法在`Registry. `中查找对象，当我们的`save()`方法返回时，`database`引用离开作用域。这确保了外部代码无法获取我们的引用。

当我们的应用程序启动时，我们可以向其中一个`BaseComponent`注册服务对象。`registerObject()`方法。稍后，应用程序中其他地方的代码可以用`require(Class)`来查找它们。

### 2.4.资源和文件系统

**`kivakit-resource`模块提供了读写流资源和访问文件系统的抽象。**我们可以在这里看到 KivaKit 包含的一些更重要的资源类型:

*   文件(本地、邮政编码、S3 和 HDFS)
*   打包资源
*   网络协议(套接字、HTTP、HTTPS 和 FTP)
*   输入和输出流

我们从这种抽象中获得了两个有价值的好处。我们可以:

*   通过一致的、[面向对象的 API](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/lexakai/kivakit/kivakit-resource/documentation/diagrams/diagram-resource.svg) 访问任何流式资源
*   使用 [`Resource`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.resource/com/telenav/kivakit/resource/Resource.html) 和 [`WritableResource`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.resource/com/telenav/kivakit/resource/WritableResource.html) 接口允许使用未知的资源类型

### 2.5.成分

[`kivakit-component`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/blob/1.0.0/kivakit-component/README.md) 模块让我们可以随时访问常用功能。我们可以:

*   **发送和接收消息**
*   **访问包和打包的资源**
*   **注册和查找对象、组件和设置**

`Component`接口由`BaseComponent`和`ComponentMixin.`共同实现，因此，我们可以给任何对象添加“组件属性”。

### 2.6.记录

**由`KivaKit`组件组成的监听器链通常终止于一个`Logger.`**[`Logger`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/logging/Logger.html)将其接收到的消息写入一个或多个[`Log`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/logging/Log.html)。此外， [`kivakit-kernel`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) 模块为实现`Log` s 提供了一个服务提供者接口(SPI)。在这里我们可以看到 UML [中日志迷你框架的完整设计。](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/lexakai/kivakit/kivakit-kernel/documentation/diagrams/diagram-logging.svg)

使用日志 SPI， [`kivakit-extensions`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions) 库为我们提供了一些`Log`实现:

  
| 供应者 | 组件 |
| --- | --- |
| 控制台日志 | [kiva kit-内核](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) |
| 终身的 | [kivakit-logs-file](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions/tree/1.0.0/kivakit-logs/file) |
| 电子邮件日志 | [kivakit-logs-email](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions/tree/1.0.0/kivakit-logs/email) |

可以从命令行选择和配置一个或多个日志。这是通过定义 [`KIVAKIT_LOG`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/blob/1.0.0/kivakit-kernel/documentation/logging.md) 系统属性来实现的。

### 2.7.转换和验证

**`kivakit-kernel`模块包含用于类型转换和对象验证的迷你框架**。这些框架与 KivaKit 消息传递集成在一起。这使得他们能够一致地报告问题。它还简化了使用。要实现像 [StringConverter](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/data/conversion/string/StringConverter.html) 这样的类型[转换器](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/data/conversion/Converter.html)，我们需要编写转换代码。我们不需要担心异常、空字符串或空值。

[![](img/af4f4c760200550de3a2a1aaf4163665.png)](/web/20220625162908/https://www.baeldung.com/wp-content/uploads/2021/09/k4.svg)

我们可以在 KivaKit 的许多地方看到转换器的使用，包括:

*   开关和参数解析
*   从属性文件加载设置对象
*   将对象格式化为调试字符串
*   从 CSV 文件中读取对象

由 [`Validatable`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/data/validation/Validatable.html) s 广播的消息被验证微型框架捕获。随后，对它们进行分析，使我们能够方便地访问错误统计和验证问题。

[![](img/cfc7c7fcfb930ba4c72d9789afe5ec6a.png)](/web/20220625162908/https://www.baeldung.com/wp-content/uploads/2021/09/k5.svg)

### 2.8.应用程序、命令行和设置

**`kivakit-application`、`kivakit-configuration,`和`kivakit-commandline`模块为开发应用程序提供了一个简单、一致的模型。**

[`kivakit-application`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-application) 项目提供了 [`Application`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-application) 基类。一个`Application`是一个`[Component](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.component/com/telenav/kivakit/component/Component.html).`，它使用`[kivakit-configuration](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-configuration).`提供设置信息，另外，它使用 [`kivakit-commandline`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-commandline) 提供命令行解析。

`kivakit-configuration`项目使用 [`kivakit-resource`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-resource) 模块从`.properties`资源(以及将来的其他资源)加载设置信息。它使用 [`kivakit-kernel`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit/tree/1.0.0/kivakit-kernel) 转换器将这些资源中的属性转换为对象。然后用验证`mini-framework`对转换后的对象进行验证。

应用程序的命令行参数和开关由`kivakit-commandline`模块使用 KivaKit 转换器和验证器进行解析。我们可以看到应用程序的 [`ConsoleLog`](https://web.archive.org/web/20220625162908/https://www.kivakit.org/1.0.0/javadoc/kivakit/kivakit.kernel/com/telenav/kivakit/kernel/logging/logs/text/ConsoleLog.html) 中出现的问题。

[![](img/250e9d686439c24683badffe100c87f3.png)](/web/20220625162908/https://www.baeldung.com/wp-content/uploads/2021/09/k6.svg)

### 2.9.微服务

到目前为止，我们已经讨论了对任何应用程序都有用的 KivaKit 特性。此外，KivaKit 还在 [`kivakit-extensions`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions) 中提供了明确针对微服务的功能。我们来快速看一下`[kivakit-web](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions/tree/1.0.0/kivakit-web)`。

**`kivakit-web`项目包含快速开发一个简单的 REST 和微服务的 web 接口的模块。**`JettyServer`类为我们提供了一种最少麻烦地插入 servlets 和过滤器的方法。我们可以和`JettyServer`一起使用的插件包括:

  
| 插件 | 描述 |
| --- | --- |
| 球衣 | REST 应用程序支持 |
| JettySwagger | Swagger 自动休息文档 |
| 捷运站 | 支持 Apache Wicket web 框架 |

这些插件可以组合起来提供一个 RESTful 微服务，带有 Swagger 文档和 web 界面:

```
var application = new MyRestApplication();
listenTo(new JettyServer())
    .port(8080)
    .add("/*", new JettyWicket(MyWebApplication.class))
    .add("/open-api/*", new JettySwaggerOpenApi(application))
    .add("/docs/*", new JettySwaggerIndex(port))
    .add("/webapp/*", new JettySwaggerStaticResources())
    .add("/webjar/*", new JettySwaggerWebJar(application))
    .add("/*", new JettyJersey(application))
    .start();
```

KivaKit 1.1 将包含一个专用的微服务迷你框架。这将使我们更容易构建微服务。

## 3.文档和 Lexakai

KivaKit 的文档由 Lexakai 生成。 [Lexakai](https://web.archive.org/web/20220625162908/https://www.lexakai.org/) 创建 UML 图(需要时由注释引导)并更新 README.md markdown 文件。在每个项目的自述文件中，Lexakai 更新了标准的页眉和页脚。此外，它还维护生成的 UML 图和 Javadoc 文档的索引。Lexakai 是一个在 Apache 许可下发布的开源项目。

## 4.建筑 KivaKit

**KivaKit 针对 Java 11 或更高版本的虚拟机(但可以从 Java 8 源代码使用)**。我们可以在 [Maven Central](https://web.archive.org/web/20220625162908/https://search.maven.org/search?q=g:com.telenav.kivakit) 上找到 KivaKit 模块的所有工件。然而，我们可能想要修改 KivaKit 或者为开源项目做贡献。在这种情况下，我们需要构建它。

先来设置一下 Git， [Git 流](https://web.archive.org/web/20220625162908/https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)， [Java 16 JDK](https://web.archive.org/web/20220625162908/https://adoptopenjdk.net/?variant=openjdk16&jvmVariant=hotspot) ， [Maven 3.8.1 以上](https://web.archive.org/web/20220625162908/https://maven.apache.org/download.cgi)。

首先，我们将`kivakit`存储库克隆到我们的工作空间中:

```
mkdir ~/Workspace
cd ~/Workspace
git clone --branch develop https://github.com/Telenav/kivakit.git
```

接下来，我们将样本 bash 配置文件复制到我们的主文件夹:

```
cp kivakit/setup/profile ~/.profile
```

然后我们修改~/。概要文件指向我们的工作区，以及我们的 Java 和 Maven 安装:

```
export KIVAKIT_WORKSPACE=$HOME/Workspace 
export JAVA_HOME=/Library/Java/JavaVirtualMachines/temurin-16.jdk/Contents/Home 
export M2_HOME=$HOME/Developer/apache-maven-3.8.2
```

在我们的概要文件建立之后，我们确保我们正在运行`bash`(在 macOS 上，`zsh`现在是默认的):

```
chsh -s /bin/bash
```

最后，我们重新启动终端程序并执行命令:

```
$KIVAKIT_HOME/setup/setup.sh
```

安装脚本将克隆 [`kivakit-extensions`](https://web.archive.org/web/20220625162908/https://github.com/Telenav/kivakit-extensions) 和其他一些相关的存储库。之后，它将初始化 git-flow 并构建我们所有的 KivaKit 项目。

## 5.结论

在本文中，我们简要介绍了 KivaKit 的设计。我们还参观了它提供的一些更重要的功能。KivaKit 非常适合开发微服务。它被设计成在容易理解的、独立的片段中学习和使用。