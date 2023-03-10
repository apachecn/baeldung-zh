# Maven 编码指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-encoding>

## 1.概观

在本教程中，我们将学习如何在 [Maven](/web/20220815212340/https://www.baeldung.com/maven) 中设置字符编码。

我们将展示如何为一些常见的 Maven 插件设置编码。

此外，我们将了解如何在项目级别以及通过命令行设置编码。

## 2.什么是编码，我们为什么要关注编码？

世界上有许多不同的语言使用不同的字符。

一个名为 Unicode 的字符映射系统拥有超过 10 万个字符、符号，甚至表情符号。

为了不占用大量的内存，**我们使用一种叫做[的映射系统，一种编码](/web/20220815212340/https://www.baeldung.com/java-char-encoding)，将一个字符在比特和字节之间转换，并在屏幕上显示一个人类可读的字符。**

现在有很多编码系统。要读取一个文件，我们必须知道使用的是哪种编码系统。

### 2.1.如果我们不在 Maven 中声明编码会怎么样？

Maven 认为编码非常重要，如果我们不声明编码，它就会发出警告。

事实上，这个警告占据了 Apache Maven 站点上 [FAQ](https://web.archive.org/web/20220815212340/https://maven.apache.org/general.html#encoding-warning) 页面的第一位。

要看到这个警告，让我们在构建中添加几个插件。

首先，让我们添加 [`maven-resources-plugin`](https://web.archive.org/web/20220815212340/https://maven.apache.org/plugins/maven-resources-plugin/) ，它将资源复制到一个输出目录中:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.2.0</version>
</plugin>
```

我们还想编译我们的代码文件，所以让我们添加 [`maven-compiler-plugin`](https://web.archive.org/web/20220815212340/https://maven.apache.org/plugins/maven-compiler-plugin/) :

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
</plugin>
```

当我们在一个多模块项目中工作时，父 POM 可能已经为我们设置了编码。出于演示目的，让我们通过覆盖 encoding 属性来清除它(不要担心，我们稍后将回到这一点):

```java
<properties>
    <project.build.sourceEncoding></project.build.sourceEncoding>
</properties>
```

让我们使用标准的 Maven 命令运行插件:

```java
mvn clean install
```

像这样不设置我们的编码会破坏构建！我们将在日志中看到以下警告:

```java
[INFO] --- maven-resources-plugin:3.2.0:resources (default-resources) @ maven-properties ---
  [WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources, i.e. build is platform dependent!
```

警告声明**如果没有指定编码系统，Maven 将使用平台默认值。**

通常在 Windows 上，默认是 [Windows-1252](https://web.archive.org/web/20220815212340/https://en.wikipedia.org/wiki/Windows-1252) (又名 CP-1252，或 Cp1252)。

该默认值可能会根据本地环境而改变。我们将在下面看到如何从我们的构建中移除这种平台依赖性。

### 2.2.如果我们在 Maven 中声明了不正确的编码会怎么样？

maven 是一个需要能够读取源文件的构建工具。

为了读取源文件， **Maven 必须设置为使用与源文件相同的编码。**

Maven 还生成通常分发到另一台计算机的文件。因此，使用预期的编码编写输出文件非常重要。**非预期编码的输出文件可能无法在不同的系统上读取。**

为了展示这一点，让我们添加一个使用非 ASCII 字符的简单 Java 类:

```java
public class NonAsciiString {

    public static String getNonAsciiString() {

        String nonAsciiŞŧř = "ÜÝÞßàæç";
        return nonAsciiŞŧř;
    }
}
```

在我们的 POM 中，让我们将我们的构建设置为使用 [ASCII](https://web.archive.org/web/20220815212340/https://en.wikipedia.org/wiki/ASCII) 编码:

```java
<properties>
    <project.build.sourceEncoding>US-ASCII</project.build.sourceEncoding>
</properties>
```

使用`mvn clean install`运行它，我们看到我们得到了许多如下形式的构建错误:

```java
[ERROR] /Baeldung/tutorials/maven-modules/maven-properties/src/main/java/
com/baeldung/maven/properties/NonAsciiString.java:[15,31] unmappable character (0xC3) for encoding US-ASCII
```

我们看到这个是因为我们的文件包含非 ASCII 字符，所以不能通过 ASCII 编码读取。

在可能的情况下，保持简单并避免使用非 ASCII 字符是一个好主意。

在下一节中，我们将看到设置 Maven 使用 [UTF-8](https://web.archive.org/web/20220815212340/https://en.wikipedia.org/wiki/UTF-8) 编码来避免任何问题也是一个好主意。

## 3.我们如何在 Maven 配置中设置编码？

首先，让我们看看如何在插件级别设置编码。

然后我们将看到我们可以设置项目范围的属性。这意味着我们不需要在每个插件中声明一个编码。

### 3.1.我们如何在 Maven 插件中设置`encoding`参数？

大多数插件都有一个`encoding`参数，这使得它非常简单。

我们需要在`maven-resources-plugin`和`maven-compiler-plugin`中设置编码。我们可以简单地给每个 Maven 插件添加`encoding`参数:

```java
<configuration>
    <encoding>UTF-8</encoding>
</configuration>
```

让我们使用`mvn clean install`运行这段代码，看看日志记录:

```java
[INFO] --- maven-resources-plugin:3.2.0:resources (default-resources) @ maven-properties ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
```

我们可以看到这个插件现在正在使用 UTF-8，我们已经解决了上面的警告。

### 3.2.我们如何在 Maven 构建中设置项目范围的`encoding`参数？

记住为我们声明的每个插件设置一个编码是非常麻烦的。

幸运的是，大多数 Maven 插件使用相同的全局 Maven 属性作为它们的`encoding`参数的默认属性。

正如我们之前看到的，让我们从插件中移除`encoding`参数，改为设置:

```java
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties> 
```

运行我们的构建会产生与上面看到的相同的 UTF-8 日志记录行。

在多模块项目中，我们通常会在父 POM 中设置该属性。

**该属性将被设置的任何特定于插件的属性覆盖。**

重要的是要记住插件没有义务使用这个属性。例如，`maven-war-plugin`的早期版本(< 2.2)会忽略该属性。

### 3.3.我们如何为一个报告插件设置一个项目范围的`encoding`参数？

也许令人惊讶的是，**我们必须设置两个属性，以保证我们已经为所有情况设置了项目范围的编码。**

为了说明这一点，我们将使用`[properties-maven-plugin](https://web.archive.org/web/20220815212340/https://www.mojohaus.org/properties-maven-plugin/):`

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>properties-maven-plugin</artifactId>
    <version>1.1.0</version>
</plugin>
```

让我们也设置一个新的系统范围的属性为空:

```java
<project.reporting.outputEncoding></project.reporting.outputEncoding>
```

如果我们现在运行一个`mvn clean install`,我们的构建将会因为日志记录而失败:

```java
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-pmd-plugin:3.13.0:pmd (pmd) on project maven-properties: Execution pmd of goal 
  org.apache.maven.plugins:maven-pmd-plugin:3.13.0:pmd failed: org.apache.maven.reporting.MavenReportException: : UnsupportedEncodingException -> [Help 1]
```

即使我们已经设置了`project.build.sourceEncoding`，这个插件也使用了不同的属性。要理解这是为什么，我们必须了解[Maven 构建配置和 Maven 报告配置之间的区别。](/web/20220815212340/https://www.baeldung.com/maven-plugin-management#plugin-configuration)

插件既可以用于构建过程，也可以用于报告过程，后者使用单独的属性键。

这意味着仅仅设置`project.build.sourceEncoding`是不够的。我们还需要为报告流程添加以下属性:

```java
<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
```

建议在项目级设置这两个属性。

### 3.4.我们如何在命令行设置 Maven 编码？

我们能够通过命令行参数设置属性，而无需向 POM 文件添加任何配置。我们可能会这样做，因为我们没有 pom.xml 文件的写权限。

让我们运行以下代码来指定构建应该使用的编码:

```java
mvn clean install -Dproject.build.sourceEncoding=UTF-8 -Dproject.reporting.outputEncoding=UTF-8
```

命令行参数覆盖任何现有的配置。

因此，这允许我们成功地运行构建，即使我们删除了 pom.xml 文件中设置的任何编码属性。

## 4.在同一 Maven 项目中使用多种类型的编码

在整个项目中使用单一类型的编码是一个好主意。

然而，我们可能会被迫在同一个构建中处理多种类型的编码。例如，我们的资源文件可能有不同的编码系统，这可能超出了我们的控制。

我们有办法做到这一点吗？嗯，看情况吧。

我们看到我们可以在逐个插件的基础上设置`encoding`参数。因此，如果我们需要 CP-1252 中的代码，但希望在 UTF-8 中输出测试结果，那么我们可以这样做。

通过使用不同的执行，我们甚至能够在同一个插件中使用多种类型的编码。

特别是我们之前看到的`maven-resources-plugin`，内置了额外的功能。

我们前面看到了`encoding`参数。该插件还提供了一个`propertiesEncoding` 参数，允许属性文件以不同于其他资源的方式编码:

```java
<configuration>
    <encoding>UTF-8</encoding>
    <propertiesEncoding>ISO-8859-1</propertiesEncoding>
</configuration>
```

当使用`mvn clean install`运行构建时，会给出:

```java
[INFO] --- maven-resources-plugin:3.2.0:resources (default-resources) @ maven-properties ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Using 'ISO-8859-1' encoding to copy filtered properties files.
```

在研究插件如何使用编码时，参考 maven.apache.org 的技术文档总是值得的。

## 5.结论

在本文中，我们看到声明编码有助于**确保代码在任何环境中以相同的方式构建。**

我们看到我们可以在插件级别设置一个编码参数。

然后，我们了解到**我们可以在项目级别**设置两个属性。他们是`project.build.sourceEncoding`和`project.reporting.outputEncoding.`

我们还看到**可以通过命令行传递编码。**这允许我们在不编辑 Maven POM 文件的情况下设置编码类型。

最后，我们研究了如何在同一个项目中使用多种类型的编码。

与往常一样，GitHub 上的示例项目[可用。](https://web.archive.org/web/20220815212340/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-properties)