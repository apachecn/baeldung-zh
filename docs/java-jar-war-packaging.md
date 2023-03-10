# 罐子包装和战争包装的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jar-war-packaging>

## 1。概述

在这个快速教程中，我们将关注 Java 中 JAR 和 WAR 打包的区别。

首先，我们将分别定义每个打包选项。之后，我们将总结他们的不同之处。

## 2。罐子包装

简单地说，JAR——或 Java Archive——是一种包文件格式。JAR 文件的扩展名为`.jar`，可能包含**库、资源和元数据文件。**

本质上，它是一个压缩文件，包含压缩版本的`.class`文件和编译后的 Java 库和应用程序的资源。

例如，下面是一个简单的 JAR 文件结构:

```java
META-INF/
    MANIFEST.MF
com/
    baeldung/
        MyApplication.class 
```

[`META-INF/MANIFEST.MF`文件](/web/20220626202815/https://www.baeldung.com/java-jar-executable-manifest-main-class)可以包含关于存储在档案中的文件的附加元数据。

我们可以使用`jar`命令或者像 [Maven](/web/20220626202815/https://www.baeldung.com/executable-jar-with-maven) 这样的工具[创建一个 JAR](/web/20220626202815/https://www.baeldung.com/java-create-jar) 文件。

## 3。战争包装

WAR 代表 Web 应用程序档案或 Web 应用程序资源。这些归档文件的扩展名为`.war`,用于打包 web 应用程序,我们可以在任何 Servlet/JSP 容器上部署这些应用程序。

下面是一个典型 WAR 文件结构的布局示例:

```java
META-INF/
    MANIFEST.MF
WEB-INF/
    web.xml
    jsp/
        helloWorld.jsp
    classes/
        static/
        templates/
        application.properties
    lib/
        // *.jar files as libs
```

在里面，它有一个`META-INF`目录，保存着`MANIFEST.MF`中关于 web 存档的有用信息。`META-INF`目录是私有的，不能从外部访问。

另一方面，它还包含了包含所有静态 web 资源的`WEB-INF`公共目录，包括 HTML 页面、图像和 JS 文件。此外，它包含了 `web.xml`文件、servlet 类和库。

我们可以使用与构建 JAR 相同的工具和命令来构建一个`.war`档案。

## 4。主要差异

那么，这两种归档类型之间的主要区别是什么？

第一个也是最明显的区别是**文件扩展名**。jar 的扩展名为`.jar`，而 WAR 文件的扩展名为`.war`。

第二个主要区别是**它们的目的和运行方式**。JAR 文件允许我们打包多个文件，以便将其用作库、插件或任何类型的应用程序。另一方面，WAR 文件仅用于 web 应用程序。

**档案馆的结构也不同。**我们可以创建任何所需结构的罐子。相比之下，WAR 有一个预定义的结构，包含`WEB-INF`和`META-INF`目录。

最后，如果我们将 JAR 构建为一个[可执行的 JAR](/web/20220626202815/https://www.baeldung.com/executable-jar-with-maven) ，而不使用额外的软件，我们就可以**从命令行**运行 JAR。或者，我们可以把它当作一个图书馆。相比之下，我们**需要一个服务器来执行一场战争**。

## 5。结论

在这篇简短的文章中，我们比较了`.jar`和`.war` Java 打包类型。在这样做的时候，我们还注意到，虽然两者都使用相同的 ZIP 文件格式，但是有几个重要的区别。