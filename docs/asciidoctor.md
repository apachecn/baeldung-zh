# Java 中的 Asciidoctor 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/asciidoctor>

## 1。简介

在本文中，我们将快速介绍如何在 Java 中使用 Asciidoctor。我们将演示如何从 AsciiDoc 文档生成 HTML5 或 PDF。

## 2。什么是 AsciiDoc？

AsciiDoc 是一种文本文档格式。它可以用于**编写文档、书籍、网页、手册页和许多其他内容。**

由于可配置性很强，AsciiDoc 文档可以转换成许多其他格式，如 HTML、PDF、手册页、EPUB 等。

因为 AsciiDoc 语法非常基本，所以它在各种浏览器插件、编程语言插件和其他工具中得到了广泛的支持。

要了解关于该工具的更多信息，我们建议阅读官方文档，在那里您可以找到许多有用的资源，用于学习将 AsciiDoc 文档导出为其他格式的正确语法和方法。

## 3。什么是 Asciidoctor？

[Asciidoctor](https://web.archive.org/web/20221128103632/http://asciidoctor.org/) 是一个**文本处理器，用于将 AsciiDoc 文档**转换成 HTML、PDF 等格式。它是用 Ruby 写的，打包成一个 RubyGem。

如上所述，AsciiDoc 是一种非常流行的编写文档的格式，因此您可以很容易地在许多 GNU Linux 发行版(如 Ubuntu、Debian、Fedora 和 Arch)中找到作为标准包的 AsciiDoc。

因为我们想在 JVM 上使用 AsciidoctorJ，所以我们将讨论 AsciidoctorJ——它是带 Java 的 asciidorj。

## 4。依赖性

要在我们的应用程序中包含 AsciidoctorJ 包，需要下面的`pom.xml`条目:

```java
<dependency>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctorj</artifactId>
    <version>1.5.5</version>
</dependency>
<dependency>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctorj-pdf</artifactId>
    <version>1.5.0-alpha.15</version>
</dependency>
```

最新版本的库可以在[这里](https://web.archive.org/web/20221128103632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.asciidoctor%22%20AND%20a%3A%22asciidoctorj%22)和[这里找到。](https://web.archive.org/web/20221128103632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.asciidoctor%22%20AND%20a%3A%22asciidoctorj-pdf%22)

## 5。AsciidoctorJ API

AsciidoctorJ 的入口点是`Asciidoctor` Java 接口。

这些方法是:

*   `convert`–解析来自`String`或`Stream`的 AsciiDoc 文档，并将其转换为提供的格式类型
*   `convertFile`–从提供的`File`对象中解析 AsciiDoc 文档，并将其转换为提供的格式类型
*   `convertFiles`–与上一个相同，但该方法接受多个`File`对象
*   `convertDirectory`–解析所提供文件夹中的所有 AsciiDoc 文档，并将其转换为所提供的格式类型

### 5.1。代码中的 API 用法

要创建一个`Asciidoctor`实例，您需要从提供的工厂方法中检索实例:

```java
import static org.asciidoctor.Asciidoctor.Factory.create;
import org.asciidoctor.Asciidoctor;
..
//some code
..
Asciidoctor asciidoctor = create(); 
```

使用检索到的实例，我们可以非常容易地转换 AsciiDoc 文档:

```java
String output = asciidoctor
  .convert("Hello _Baeldung_!", new HashMap<String, Object>());
```

如果我们想从文件系统转换一个文本文档，我们将使用`convertFile`方法:

```java
String output = asciidoctor
  .convertFile(new File("baeldung.adoc"), new HashMap<String, Object>()); 
```

为了转换多个文件，`convertFiles`方法接受`List`对象作为第一个参数，并返回`String`对象的数组。
更有趣的是如何用 AsciidoctorJ 转换整个目录。

如上所述，要转换整个目录，我们应该调用`convertDirectory`方法。这将扫描所提供的路径，并搜索所有带有 AsciiDoc 扩展名(.adoc，。公元，。asciidoc，。asc)并转换它们。为了扫描所有文件，应该向该方法提供一个`DirectoryWalker`实例。

目前，Asciidoctor 提供了上述接口的两个内置实现:

*   `**AsciiDocDirectoryWalker**`–转换给定文件夹及其子文件夹的所有文件。忽略所有以“_”开头的文件
*   `**GlobDirectoryWalker**`–按照 glob 表达式转换给定文件夹的所有文件

```java
String[] result = asciidoctor.convertDirectory(
  new AsciiDocDirectoryWalker("src/asciidoc"),
  new HashMap<String, Object>()); 
```

同样，**我们可以用提供的`java.io.Reader`和`java.io.Writer`接口调用 convert 方法。** `Reader`接口用作源，`Writer`接口用于写入转换后的数据:

```java
FileReader reader = new FileReader(new File("sample.adoc"));
StringWriter writer = new StringWriter();

asciidoctor.convert(reader, writer, options().asMap());

StringBuffer htmlBuffer = writer.getBuffer();
```

### 5.2。PDF 生成

**要从 Asciidoc 文档生成 PDF 文件，我们需要在选项中指定生成文件的类型。**如果你更仔细地观察前面的例子，你会注意到任何 convert 方法的第二个参数是一个`Map`——它代表 options 对象。

我们将 in_place 选项设置为 true，这样我们的文件就会自动生成并保存到文件系统中:

```java
Map<String, Object> options = options()
  .inPlace(true)
  .backend("pdf")
  .asMap();

String outfile = asciidoctor.convertFile(new File("baeldung.adoc"), options);
```

## 6。Maven 插件

在上一节中，我们展示了如何使用您自己的 Java 实现直接生成 PDF 文件。在这一节中，我们将展示如何在 Maven 构建期间生成 PDF 文件。Gradle 和 Ant 也有类似的插件。

要在构建期间启用 PDF 生成，您需要将这个依赖项添加到您的`pom.xml:`

```java
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>1.5.5</version>
    <dependencies>
        <dependency>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctorj-pdf</artifactId>
            <version>1.5.0-alpha.15</version>
        </dependency>
    </dependencies>
</plugin>
```

Maven 插件依赖的最新版本可以在[这里](https://web.archive.org/web/20221128103632/https://search.maven.org/classic/#search%7Cga%7C1%7Casciidoctor-maven-plugin)找到。

### 6.1。用途

要在构建中使用插件，您必须在`pom.xml:`中定义它

```java
<plugin>
    <executions>
        <execution>
            <id>output-html</id> 
            <phase>generate-resources</phase> 
            <goals>
                <goal>process-asciidoc</goal> 
            </goals>
        </execution>
    </executions>
</plugin>
```

由于插件不会在任何特定的阶段运行，所以您必须设置您想要启动它的阶段。

与 Asciidoctorj 插件一样，我们也可以在这里使用各种选项来生成 PDF。

让我们快速浏览一下基本选项，您可以在[文档](https://web.archive.org/web/20221128103632/https://github.com/asciidoctor/asciidoctor-maven-plugin)中找到其他选项:

*   `**sourceDirectory**`–Asciidoc 文档所在的目录位置
*   `**outputDirectory**`–您想要存储生成的 PDF 文件的目录位置
*   `**backend**`–ascii doctor 输出的类型。用于 pdf 生成为 PDF 设置

这是一个如何在插件中定义基本选项的例子:

```java
<plugin>
    <configuration>
        <sourceDirectory>src/main/doc</sourceDirectory>
        <outputDirectory>target/docs</outputDirectory>
        <backend>pdf</backend>
    </configuration>
</plugin>
```

运行构建后，可以在指定的输出目录中找到 PDF 文件。

## 7。结论

尽管 AsciiDoc 非常容易使用和理解，但它是管理文档和其他文档的一个非常强大的工具。

在本文中，我们演示了一种从 AsciiDoc 文档生成 HTML 和 PDF 文件的简单方法。

代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128103632/https://github.com/eugenp/tutorials/tree/master/asciidoctor)