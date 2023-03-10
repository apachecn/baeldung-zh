# Maven 包装类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-packaging-types>

## 1.概观

打包类型是任何 Maven 项目的一个重要方面。它指定了项目产生的工件的类型。通常，一个构建会产生一个`jar`、`war`、`pom`或其他可执行文件。

Maven 提供了许多默认的打包类型，也提供了定义自定义打包类型的灵活性。

在本教程中，我们将深入研究 Maven 打包类型。首先，我们将看看 Maven 中的构建生命周期。然后，我们将讨论每种包装类型，它们代表什么，以及它们对项目生命周期的影响。最后，我们将看到如何定义一个定制的包装类型。

## 2.默认包装类型

[Maven](/web/20220726092743/https://www.baeldung.com/maven) 提供了许多默认的打包类型，包括`jar`、`war`、`ear`、`pom`、`rar`、`ejb`和`maven-plugin`。**每种打包类型都遵循由多个阶段组成的构建生命周期。**通常，每个阶段都是一系列目标，并执行特定的任务。

**不同的包装类型在特定阶段可能有不同的目标。**例如在`jar`打包类型的打包环节，执行`maven-jar-plugin`的 jar 目标。相反，对于一个`war`项目，`maven-war-plugin`的战争目标在同一个阶段执行。

### 2.1.`jar`

Java 归档——或`jar` ——是最流行的打包类型之一。这种打包类型的项目会生成一个扩展名为`.jar`的压缩 zip 文件。它可能包括纯 Java 类、接口、资源和元数据文件。

首先，让我们看看`jar`的一些默认的目标到构建阶段的绑定:

*   资源:`resources`
*   编译器:`compile`
*   资源:`testResources`
*   编译器:`testCompile`
*   万全之策:`test`
*   **罐子:`jar`**
*   安装:`install`
*   部署:`deploy`

现在，让我们定义一个`jar`项目的打包类型:

```java
<packaging>jar</packaging>
```

**如果没有指定，Maven 假定包装类型是`jar.`**

### 2.2.`war`

简而言之，web 应用程序档案——或`war`—包含与 web 应用程序相关的所有文件。它可能包括 Java servlets、JSP、HTML 页面、部署描述符和相关资源。总的来说，`war`与`jar`有相同的目标绑定，但是有一个例外——`war`的包阶段有不同的目标，即`war`。

毫无疑问，`jar`和`war`是 Java 社区中最流行的打包类型。这两个之间的详细[差异可能是一个有趣的阅读。](/web/20220726092743/https://www.baeldung.com/java-jar-war-packaging)

让我们定义一个 web 应用程序的打包类型:

```java
<packaging>war</packaging>
```

**其他包装类型`ejb`、`par`和`rar`也有类似的生命周期，但每个都有不同的包装目标。**

```java
ejb:ejb or par:par or rar:rar
```

### 2.3.`ear`

企业应用程序归档(或称`ear`)是一个包含 J2EE 应用程序的压缩文件。它由一个或多个模块组成，这些模块可以是 web 模块(打包成一个`war`文件)或 EJB 模块(打包成一个`jar`文件)，或者两者都是。

换句话说，`ear`是`jars`和`wars`的超集，需要一个应用服务器来运行应用，而`war`只需要一个 web 容器或 web 服务器来部署它。区分 web 服务器和应用服务器的方面，以及 Java 中那些[流行的服务器是什么](/web/20220726092743/https://www.baeldung.com/java-servers)，对于 Java 开发者来说是重要的概念。

让我们定义`ear`的默认目标绑定:

*   耳朵:`generate-application-xml`
*   资源:`resources`
*   **耳:`ear`**
*   安装:`install`
*   部署:`deploy`

下面是我们如何定义此类项目的打包类型:

```java
<packaging>ear</packaging>
```

### 2.4.`pom`

在所有包装类型中，`pom`是最简单的一种。它有助于创建聚合器和父项目。

**一个聚合器或[多模块项目](/web/20220726092743/https://www.baeldung.com/maven-multi-module)集合来自不同来源的子模块。**这些子模块是常规的 Maven 项目，遵循它们自己的构建生命周期。聚合器 POM 在`modules`元素下有所有子模块的引用。

**父项目允许您定义 POM 之间的继承关系。**父 POM 共享某些配置、插件和依赖项，以及它们的版本。父元素的大多数元素都被其子元素继承——例外情况包括`artifactId`、`name`和`prerequisites`。

因为没有要处理的资源，也没有要编译或测试的代码。因此，pom 项目的工件自己生成，而不是任何可执行文件。

让我们定义一个多模块项目的打包类型:

```java
<packaging>pom</packaging>
```

**这样的项目具有最简单的生命周期，只包含两个步骤:`install`和`deploy`。**

### 2.5.`maven-plugin`

Maven 提供了各种有用的插件。然而，可能会有默认插件不够用的情况。在这种情况下，该工具提供了根据项目需求[创建 maven-plugin](/web/20220726092743/https://www.baeldung.com/maven-plugin) 的灵活性。

要创建插件，请设置项目的打包类型:

```java
<packaging>maven-plugin</packaging>
```

`maven-plugin`的生命周期类似于`jar`的生命周期，但有两个例外:

*   插件: `descriptor`被绑定到生成资源阶段
*   插件: `addPluginArtifactMetadata`被添加到包阶段

对于这种类型的项目，需要一个`maven-plugin-api`依赖关系。

### 2.6.`ejb`

enterprise JavaBean s——或`[ejb](/web/20220726092743/https://www.baeldung.com/ejb-intro)`—有助于创建可伸缩的分布式服务器端应用程序。EJB 通常提供应用程序的业务逻辑。典型的 EJB 体系结构由三个组件组成:企业 Java bean(EJB)、EJB 容器和应用服务器。

现在，让我们定义 EJB 项目的包装类型:

```java
<packaging>ejb</packaging>
```

`ejb`包装类型也具有与`jar`包装相似的生命周期，但是具有不同的包装目标。这类项目的包目标是 ejb: `ejb`。

**打包类型为`ejb`的项目需要一个`maven-ejb-plugin`来执行生命周期目标。** Maven 为 EJB 2 号和 3 号提供支持。如果没有指定版本，则使用默认版本 2。

### 2.7.`rar`

资源适配器——或`rar`——是一个归档文件，用作将资源适配器部署到应用服务器的有效格式。基本上，它是一个连接 Java 应用程序和企业信息系统(EIS)的系统级驱动程序。

下面是资源适配器的打包类型声明:

```java
<packaging>rar</packaging>
```

每个资源适配器档案由两部分组成:一个包含源代码的`jar`文件和一个用作部署描述符的`ra.xml`。

同样，生命周期阶段与`jar`或`war`打包相同，只有一个例外:**`**package**` **阶段执行由** `**maven-rar-plugin**` **组成的`rar`目标，以打包文档** **。****

 **## 3.其他包装类型

到目前为止，我们已经研究了 Maven 默认提供的各种打包类型。现在，让我们想象我们希望我们的项目产生一个带有`.zip` 扩展名的工件。在这种情况下，默认的包装类型帮不了我们。

Maven 还通过插件提供了更多的打包类型。在这些插件的帮助下，我们可以定义一个定制的打包类型及其构建生命周期。这些类型包括:

*   `msi`
*   `rpm`
*   `tar`
*   `tar.bz2`
*   `tar.gz`
*   `tbz`
*   `zip`

要定义一个自定义类型，我们必须在它的生命周期`.` 中定义它的`packaging` `type` 和 `phases` 为此，在`src/main/resources/META-INF/plexus`目录下创建一个`components.xml`文件:

```java
<component>
   <role>org.apache.maven.lifecycle.mapping.LifecycleMapping</role>
   <role-hint>zip</role-hint>
   <implementation>org.apache.maven.lifecycle.mapping.DefaultLifecycleMapping</implementation>
   <configuration>
      <phases>
         <process-resources>org.apache.maven.plugins:maven-resources-plugin:resources</process-resources>
         <package>com.baeldung.maven.plugins:maven-zip-plugin:zip</package>
         <install>org.apache.maven.plugins:maven-install-plugin:install</install>
         <deploy>org.apache.maven.plugins:maven-deploy-plugin:deploy</deploy>
      </phases>
   </configuration>
</component>
```

直到现在，Maven 对我们的新包装类型及其生命周期一无所知。为了让它可见，让我们将插件添加到项目的`pom`文件中，并将`extensions`设置为`true`:

```java
<plugins>
    <plugin>
        <groupId>com.baeldung.maven.plugins</groupId>
        <artifactId>maven-zip-plugin</artifactId>
        <extensions>true</extensions>
    </plugin>
</plugins>
```

现在，该项目将可用于扫描，系统也将查看`plugins`和`components.xml`文件。

除了所有这些类型，Maven 还通过外部项目和插件提供了许多其他的打包类型。例如，`nar`(本机归档)、`swf`和`swc`是产生 Adobe Flash 和 Flex 内容的项目的打包类型。对于这样的项目，我们需要一个定义自定义打包的插件和一个包含插件的存储库。

## 4.结论

在本文中，我们研究了 Maven 中可用的各种打包类型。此外，我们熟悉了这些类型所代表的内容，以及它们在生命周期中的不同之处。最后，我们还学习了如何定义一个定制的打包类型和定制默认的构建生命周期。

Baeldung 上的所有代码示例都是使用 Maven 构建的。请务必查看我们的各种 Maven 配置[超过 0n GitHub](https://web.archive.org/web/20220726092743/https://github.com/eugenp/tutorials/tree/master/maven-modules) 。**