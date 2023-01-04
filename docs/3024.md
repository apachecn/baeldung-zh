# 了解 JAR 清单文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jar-manifest>

## 1。简介

Java 档案(JAR)由其清单文件描述。本文探索了它的许多功能，包括添加属性、使 JAR 可执行以及嵌入版本信息。

不过，让我们先快速回顾一下什么是清单文件。

## 2。清单文件

清单文件名为`MANIFEST.MF`，位于 JAR 中的`META-INF`目录下。它仅仅是**一个键和值对的列表，被称为`headers`或`attributes`，被分组为几个部分。**

这些`headers`提供元数据，帮助我们描述 JAR 的各个方面，比如包的版本、要执行的应用程序类、类路径、签名材料等等。

## 3。添加清单文件

### 3.1。默认清单

每当我们[创建一个 JAR](/web/20221018153013/https://www.baeldung.com/java-create-jar) 时，就会自动添加一个清单文件。

例如，如果我们在 OpenJDK 11 中构建一个 JAR:

```
jar cf MyJar.jar classes/
```

它生成一个非常简单的清单文件:

```
Manifest-Version: 1.0
Created-By: 11.0.3 (AdoptOpenJDK)
```

### 3.2。自定义清单

或者，我们可以指定自己的清单文件。

例如，假设我们有一个名为`manifest.txt`的定制清单文件:

```
Built-By: baeldung
```

我们可以包含这个文件，当我们使用`m` 选项时，`jar` 将**将其与默认的清单文件**合并:

```
jar cfm MyJar.jar manifest.txt classes/
```

然后，生成的清单文件是:

```
Manifest-Version: 1.0
Built-By: baeldung
Created-By: 11.0.3 (AdoptOpenJDK)
```

### 3.3 版。肚子

现在，默认清单文件**的内容根据我们使用的工具而改变。**

例如，Maven 添加了一些额外的头:

```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven 3.3.9
Built-By: baeldung
Build-Jdk: 11.0.3
```

我们实际上可以在 pom 中定制这些头。

比方说，我们想指出 JAR 是由谁创建的，以及包:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <archive>
            <manifest>
                <packageName>com.baeldung.java</packageName>
            </manifest>
            <manifestEntries>
                <Created-By>baeldung</Created-By>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

这将生成一个带有定制的`package`和`created-by`头的清单文件:

```
Manifest-Version: 1.0
Build-Jdk-Spec: 11
Package: com.baeldung.java
Created-By: baeldung 
```

参考[Maven JAR 插件文档](https://web.archive.org/web/20221018153013/https://maven.apache.org/plugins/maven-jar-plugin/)获取选项的完整列表。

## 4。标题

标题必须遵循特定的格式，并由新行分隔:

```
key1: value1
Key2: value2
```

**有效的标题在冒号和值**之间必须有一个空格。另外重要的一点是**在文件**的末尾必须有新的一行。否则，将忽略最后一个标头。

让我们看看来自[规范](https://web.archive.org/web/20221018153013/https://docs.oracle.com/en/java/javase/11/docs/specs/jar/jar.html#jar-manifest)的一些标准头和一些常见的定制头。

### 4.1。主集管

主标题通常提供一般信息:

*   `Manifest-Version`:规格的版本
*   `Created-By`:创建清单文件的工具版本和厂商
*   `Multi-Release`:如果`true`，那么这是一个[多释放震击器](/web/20221018153013/https://www.baeldung.com/java-multi-release-jar)
*   `Built-By`:这个自定义头给出了创建清单文件的用户的名字

### 4.2。入口点和类路径

如果我们的 JAR 包含一个可运行的应用程序，那么我们可以指定入口点。同样，我们可以提供`classpath`。通过这样做，我们避免了在想要运行它时必须指定它。

*   `Main-Class`:带有 main 方法的类的包和名称(编号类扩展)
*   `Class-Path`:以空格分隔的库或资源的相对路径列表

例如，如果我们的应用程序入口点在`Application.class`中，并且它使用库和资源，那么我们可以添加所需的头:

```
Main-Class: com.baeldung.Application
Class-Path: core.jar lib/ properties/
```

类路径包括`core.jar`以及`lib`和`properties`目录中的所有文件。**这些资产是相对于 JAR 的执行位置加载的，而不是从 JAR 本身加载的**。换句话说，它们必须存在于罐子之外。

### 4.3。包装版本和密封

这些标准头描述了 JAR 中的包。

*   `Name`:包装
*   `Implementation-Build-Date`:实现的构建日期
*   `Implementation-Title`:实现的标题
*   `Implementation-Vendor`:实施的厂商
*   `Implementation-Version`:实现版本
*   `Specification-Title`:规范的标题
*   `Specification-Vendor`:规格的卖方
*   `Specification-Version`:规格版本
*   `Sealed`:如果为真，那么这个包的所有类都来自同一个 JAR(默认为假)

例如，我们在 MySQL 驱动程序连接器/J [JAR](https://web.archive.org/web/20221018153013/https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.16/mysql-connector-java-8.0.16.jar) 中找到这些清单头。它们描述了 JAR 符合的 JDBC 规范的版本，以及驱动程序本身的版本:

```
Specification-Title: JDBC
Specification-Version: 4.2
Specification-Vendor: Oracle Corporation
Implementation-Title: MySQL Connector/J
Implementation-Version: 8.0.16
Implementation-Vendor: Oracle
```

### 4.4。已签名的 Jar

我们可以对我们的 JAR 进行数字签名，以增加额外的安全性和验证。虽然这个过程超出了本文的范围，**这样做是向清单文件**添加显示每个签名类及其编码签名的标准头。更多细节请参见 [JAR 签名](https://web.archive.org/web/20221018153013/https://docs.oracle.com/en/java/javase/11/docs/specs/jar/jar.html#signed-jar-file)文档。

### 4.5。OSGI

常见的还有 OSGI 包的定制[头:](https://web.archive.org/web/20221018153013/http://docs.osgi.org/reference/bundle-headers.html)

*   `Bundle-Name` : title
*   `Bundle-SymbolicName`:唯一的标识符
*   `Bundle-Version`:版本
*   `Import-Package`:捆绑包所依赖的包和版本
*   `Export-Package`:可供使用的捆绑包和版本

参见我们的[OSGI 简介](/web/20221018153013/https://www.baeldung.com/osgi)文章，了解更多关于 OSGI 包的信息。

## 5。章节

清单文件中有两种类型的部分，main 和 per-entry。出现在主要部分的标题适用于 JAR 中的所有内容。而出现在每个条目部分的**头只适用于指定的包或类**。

此外，每个条目部分中出现的标题会覆盖主部分中的相同标题。每个条目部分包含关于包版本和密封以及数字签名的信息是很常见的。

让我们看一个简单的每个条目部分的例子:

```
Implementation-Title: baeldung-examples 
Implementation-Version: 1.0.1
Implementation-Vendor: Baeldung
Sealed: true

Name: com/baeldung/utils/
Sealed: false
```

顶部的主要部分密封了我们罐子中的所有包。但是，包`com.baeldung.utils`是由每个条目部分解封的。

## 6。结论

本文概述了如何向 JAR 添加清单文件，如何使用节和一些常见的头。清单文件的结构允许我们提供标准信息，比如版本信息。

然而，它的灵活性允许我们定义任何我们认为相关的信息来描述我们的 jar 的内容。