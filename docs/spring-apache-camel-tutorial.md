# 使用 Apache Camel 和 Spring

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-apache-camel-tutorial>

## 1。概述

本文将演示如何配置和使用 Spring 的 Apache Camel。

Apache Camel 提供了相当多有用的组件，支持诸如 [JPA](https://web.archive.org/web/20220815030745/https://camel.apache.org/jpa.html) 、 [Hibernate](https://web.archive.org/web/20220815030745/https://people.apache.org/~dkulp/camel/hibernate.html) 、 [FTP](https://web.archive.org/web/20220815030745/https://camel.apache.org/ftp2.html) 、 [Apache-CXF](https://web.archive.org/web/20220815030745/https://camel.apache.org/cxf.html) 、 [AWS-S3](https://web.archive.org/web/20220815030745/https://camel.apache.org/components/3.12.x/aws2-s3-component.html) 等库，所有这些都有助于在两个不同系统之间集成数据。

例如，使用 Hibernate 和 Apache CXF 组件，您可以从数据库中提取数据，并通过 REST API 调用将其发送到另一个系统。

在本教程中，我们将查看一个简单的 Camel 示例——读取一个文件并将其内容转换为大写，然后再转换回小写。我们将使用 Camel 的[文件组件](https://web.archive.org/web/20220815030745/https://camel.apache.org/file2.html)和 Spring 4.2。

以下是该示例的全部细节:

1.  从源目录读取文件
2.  使用定制的[处理器](https://web.archive.org/web/20220815030745/https://camel.apache.org/processor.html)将文件内容转换为大写
3.  将转换后的输出写入目标目录
4.  使用 [Camel Translator](https://web.archive.org/web/20220815030745/https://camel.apache.org/message-translator.html) 将文件内容转换成小写
5.  将转换后的输出写入目标目录

## 2。添加依赖关系

要将 Apache Camel 与 Spring 一起使用，POM 文件中需要以下依赖项:

```java
<properties>
    <env.camel.version>2.16.1</env.camel.version>
    <env.spring.version>4.2.4.RELEASE</env.spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-core</artifactId>
        <version>${env.camel.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-spring</artifactId>
        <version>${env.camel.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-stream</artifactId>
        <version>${env.camel.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${env.spring.version}</version>
    </dependency>
</dependencies>
```

所以，我们有:

*   阿帕奇骆驼的主要依靠
*   `camel-spring`–使我们能够使用带弹簧的骆驼
*   `camel-stream`–一个可选的依赖项，可用于(例如)在路线运行时在控制台上显示一些消息
*   标准的 Spring 依赖，在我们的例子中是必需的，因为我们将在 Spring 上下文中运行 Camel 路线

## 3。春驼语境

首先，我们将创建 Spring 配置文件，稍后我们将在其中定义 Camel 路线。

请注意该文件如何包含所有必需的 Apache Camel 和 Spring 名称空间以及模式位置:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xmlns:camel="http://camel.apache.org/schema/spring"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd	
          http://camel.apache.org/schema/spring 
          http://camel.apache.org/schema/spring/camel-spring.xsd
          http://www.springframework.org/schema/util 
          http://www.springframework.org/schema/util/spring-util-4.2.xsd">

	<camelContext >
            <!-- Add routes here -->
	</camelContext>

</beans>
```

`<camelContext>`元素表示(不出所料)Camel 上下文，可以与 Spring 应用程序上下文相比较。现在您的上下文文件已经准备好开始定义骆驼路线了。

### 3.1。带定制处理器的骆驼路线

接下来，我们将编写第一个将文件内容转换为大写的路径。

我们需要定义路由将从中读取数据的源。这可以是数据库、文件、控制台或任何数量的其他来源。在我们的情况下，它将被归档。

然后，我们需要定义将从源中读取的数据的处理器。对于这个例子，我们将编写一个定制的处理器类。这个类将是一个 Spring bean，它将实现标准的 Camel 处理器接口。

一旦数据被处理，我们需要告诉路由到哪里直接处理的数据。同样，这可能是各种输出中的一种，如数据库、文件或控制台。在我们的例子中，我们将把它存储在一个文件中。

要设置这些步骤，包括输入、处理器和输出，请将以下路由添加到 Camel 上下文文件:

```java
<route>
    <from uri="file://data/input" /> <!-- INPUT -->
    <process ref="myFileProcessor" /> <!-- PROCESS -->
    <to uri="file://data/outputUpperCase" /> <!-- OUTPUT -->
</route>
```

此外，我们必须定义`myFileProcessor` bean:

```java
<bean id="myFileProcessor" class="org.apache.camel.processor.FileProcessor" />
```

### 3.2。定制大写处理器

现在我们需要创建我们在 bean 中定义的自定义文件处理器。它必须实现 Camel `Processor`接口，定义一个单独的`process`方法，该方法将一个`Exchange`对象作为其输入。该对象提供来自输入源的数据的详细信息。

我们的方法必须从`Exchange`中读取消息，将内容大写，然后将新内容设置回`Exchange` 对象:

```java
public class FileProcessor implements Processor {

    public void process(Exchange exchange) throws Exception {
        String originalFileContent = (String) exchange.getIn().getBody(String.class);
        String upperCaseFileContent = originalFileContent.toUpperCase();
        exchange.getIn().setBody(upperCaseFileContent);
    }
}
```

对于从源接收的每个输入，都将执行这个处理方法。

### 3.3。小写处理器

现在，我们将为我们的骆驼路线添加另一个输出。这一次，我们将把同一个输入文件的数据转换成小写。然而，这一次我们将不会使用定制的处理器；我们将使用 Apache Camel 的[消息翻译特性](https://web.archive.org/web/20220815030745/https://people.apache.org/~dkulp/camel/message-translator.html)。这是最新的骆驼路线:

```java
<route>
    <from uri="file://data/input" />
    <process ref="myFileProcessor" />
    <to uri="file://data/outputUppperCase" />
    <transform>
        <simple>${body.toLowerCase()}</simple>
    </transform>
    <to uri="file://data/outputLowerCase" />
</route>
```

## 4。运行应用程序

为了处理我们的路线，我们只需将 Camel 上下文文件加载到 Spring 应用程序上下文中:

```java
ClassPathXmlApplicationContext applicationContext = 
  new ClassPathXmlApplicationContext("camel-context.xml"); 
```

一旦路由成功运行，就会创建两个文件:一个包含大写内容，另一个包含小写内容。

## 5。结论

如果您正在进行集成工作，Apache Camel 绝对可以让事情变得更简单。该库提供了即插即用的组件，可以帮助您减少样板代码并专注于处理数据的主要逻辑。

如果你想详细地探索企业集成模式的概念，你应该看看格雷戈尔·霍佩和鲍比·伍尔夫写的《T2》这本书，他们非常清晰地定义了企业集成点。

本文中描述的例子可以在 GitHub 上的一个[项目中找到。](https://web.archive.org/web/20220815030745/https://github.com/eugenp/tutorials/tree/master/spring-apache-camel)