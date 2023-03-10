# 阿帕奇骆驼简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-camel-intro>

## 1。概述

在本文中，我们将**介绍 Camel 并探索其核心概念之一——消息路由**。

我们将从介绍这些基本概念和术语开始，然后介绍定义路由的两个主要选项——Java DSL 和 Spring DSL。

我们还将通过一个例子来演示这些——通过定义一个路径，该路径使用一个文件夹中的文件并将它们移动到另一个文件夹中，同时为每个文件名预先给添加一个时间戳。

## 2。关于阿帕奇骆驼

Apache Camel 是一个开源集成框架，旨在使系统集成变得简单和容易。

它允许最终用户使用相同的 API 集成各种系统，提供对多种协议和数据类型的支持，同时可扩展并允许引入自定义协议。

## 3。Maven 依赖关系

为了使用 Camel，我们需要首先添加 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-core</artifactId>
    <version>2.18.0</version>
</dependency>
```

骆驼神器的最新版本可以在[这里](https://web.archive.org/web/20220706210227/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.camel%22)找到。

## 3。特定领域语言

路线和路由引擎是 Camel 的核心部分。路由包含不同系统之间集成的流程和逻辑。

为了更简单明了地定义路线，Camel 为 Java 或 Groovy 等编程语言提供了几种不同的领域特定语言(DSL)。另一方面，它也提供了用 Spring DSL 在 XML 中定义路由。

使用 Java DSL 还是 Spring DSL 主要是用户的偏好，因为大多数特性在两者中都可用。

Java DSL 提供了一些 Spring DSL 不支持的特性。然而，Spring DSL 有时更有益，因为 XML 可以在不需要重新编译代码的情况下进行更改。

## 4。术语和架构

现在让我们讨论基本的 Camel 术语和架构。

首先，我们来看看 Camel 的核心概念:

*   **消息**包含正在传输到路由的数据。每封邮件都有一个唯一的标识符，它由正文、标题和附件组成
*   **交换**是消息的容器，它是在路由过程中消费者收到消息时创建的。Exchange 允许系统之间不同类型的交互，它可以定义单向消息或请求-响应消息
*   **端点**是系统接收或发送消息的通道。它可以指 web 服务 URI、队列 URI、文件、电子邮件地址等
*   **组件**充当端点工厂。简而言之，组件使用相同的方法和语法提供了不同技术的接口。Camel 已经在它的 DSL 中为几乎所有可能的技术支持[许多组件](https://web.archive.org/web/20220706210227/https://camel.apache.org/components.html)，但是它也提供了编写定制组件的能力
*   **处理器**是一个简单的 Java 接口，用于向路由添加自定义集成逻辑。它包含一个用于在消费者收到的消息上执行定制业务逻辑的方法

在高层次上，Camel 的架构很简单。`CamelContext`代表 Camel 运行时系统，它连接了不同的概念，如路线、组件或端点。

在此之下，处理器处理端点之间的路由和转换，而端点集成不同的系统。

## 5。定义路线

可以用 Java DSL 或 Spring DSL 定义路由。

我们将通过定义一个路径来说明这两种风格，该路径使用一个文件夹中的文件，并将它们移动到另一个文件夹中，同时为每个文件名加上一个时间戳。

### 5.1。使用 Java DSL 进行路由

要用 Java DSL 定义路由，我们首先需要创建一个`DefaultCamelContext`实例。之后，我们需要扩展`RouteBuilder`类并实现包含路由流的`configure`方法:

```java
private static final long DURATION_MILIS = 10000;
private static final String SOURCE_FOLDER = "src/test/source-folder";
private static final String DESTINATION_FOLDER 
  = "src/test/destination-folder";

@Test
public void moveFolderContentJavaDSLTest() throws Exception {
    CamelContext camelContext = new DefaultCamelContext();
    camelContext.addRoutes(new RouteBuilder() {
      @Override
      public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true").process(
          new FileProcessor()).to("file://" + DESTINATION_FOLDER);
      }
    });
    camelContext.start();
    Thread.sleep(DURATION_MILIS);
    camelContext.stop();
}
```

`configure`方法可以这样理解:从源文件夹中读取文件，用`FileProcessor`处理它们，并将结果发送到目标文件夹。设置`delete=true`表示文件处理成功后将从源文件夹中删除。

为了启动 Camel，我们需要在`CamelContext`上调用`start`方法。调用`Thread.sleep`是为了让 Camel 有必要的时间将文件从一个文件夹移动到另一个文件夹。

`FileProcessor`实现`Processor`接口并包含单个`process`方法，该方法包含修改文件名的逻辑:

```java
public class FileProcessor implements Processor {
    public void process(Exchange exchange) throws Exception {
        String originalFileName = (String) exchange.getIn().getHeader(
          Exchange.FILE_NAME, String.class);

        Date date = new Date();
        SimpleDateFormat dateFormat = new SimpleDateFormat(
          "yyyy-MM-dd HH-mm-ss");
        String changedFileName = dateFormat.format(date) + originalFileName;
        exchange.getIn().setHeader(Exchange.FILE_NAME, changedFileName);
    }
}
```

为了检索文件名，我们必须从交换中检索传入的消息并访问其标题。类似地，要修改文件名，我们必须更新消息头。

### 5.2。使用 Spring DSL 进行路由

当使用 Spring DSL 定义路由时，我们使用一个 XML 文件来设置路由和处理器。这允许我们通过使用 Spring 来配置路由，而不使用任何代码，并最终给我们带来完全控制反转的好处。

这已经在现有文章中介绍过了，所以我们将集中讨论同时使用 Spring DSL 和 Java DSL，Java DSL 通常是定义路由的首选方式。

在这种安排中，CamelContext 是在 Spring XML 文件中使用 Camel 的自定义 XML 语法定义的，但没有像在使用 XML 的“纯”Spring DSL 的情况下那样定义路由:

```java
<bean id="fileRouter" class="com.baeldung.camel.file.FileRouter" />
<bean id="fileProcessor" 
  class="com.baeldung.camel.file.FileProcessor" />

<camelContext >
    <routeBuilder ref="fileRouter" />
</camelContext> 
```

这样，我们告诉 Camel 使用`FileRouter` 类，它保存了我们在 Java DSL 中的路由定义:

```java
public class FileRouter extends RouteBuilder {

    private static final String SOURCE_FOLDER = 
      "src/test/source-folder";
    private static final String DESTINATION_FOLDER = 
      "src/test/destination-folder";

    @Override
    public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true").process(
          new FileProcessor()).to("file://" + DESTINATION_FOLDER);
    }
}
```

为了测试这一点，我们必须创建一个`ClassPathXmlApplicationContext`的实例，它将在春天加载我们的`CamelContext`:

```java
@Test
public void moveFolderContentSpringDSLTest() throws InterruptedException {
    ClassPathXmlApplicationContext applicationContext = 
      new ClassPathXmlApplicationContext("camel-context.xml");
    Thread.sleep(DURATION_MILIS);
    applicationContext.close();
}
```

通过使用这种方法，我们获得了 Spring 提供的额外的灵活性和好处，以及通过使用 Java DSL 获得 Java 语言的所有可能性。

## 6。结论

在这篇简短的文章中，我们介绍了 Apache Camel，并展示了使用 Camel 完成集成任务的好处，比如将文件从一个文件夹路由到另一个文件夹。

在我们的例子中，我们看到 Camel 让您专注于业务逻辑，并减少了样板代码的数量。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220706210227/https://github.com/eugenp/tutorials/tree/master/spring-apache-camel)