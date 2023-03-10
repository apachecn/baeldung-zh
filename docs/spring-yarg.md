# 春天雅阁集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-yarg>

## 1。概述

另一个报告生成器(YARG)是由 Haulmont 开发的 Java 开源报告库。它允许以最常见的格式(`.doc, .docs, .xls, .xlsx, .html, .ftl, .csv`)或定制文本格式创建模板，并用 SQL、Groovy 或 JSON 加载的数据填充模板。

在本文中，我们将演示如何使用 Spring `@RestController`输出带有 JSON 加载数据的`.docx`文档。

## 2.树立榜样

为了开始使用 YARG，我们需要将以下依赖项添加到我们的`pom:`

```java
<repositories>
    <repository>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
        <id>bintray-cuba-platform-main</id>
        <name>bintray</name>
        <url>http://dl.bintray.com/cuba-platform/main</url>
    </repository>
</repositories>
...
<dependency> 
    <groupId>com.haulmont.yarg</groupId> 
    <artifactId>yarg</artifactId> 
    <version>2.0.4</version> 
</dependency>
```

接下来，**我们需要一个数据模板**；我们将使用一个简单的`Letter.docx`:

```java
${Main.title}

Hello ${Main.name},

${Main.content} 
```

注意 YARG 是如何使用标记/模板语言的——它允许在不同的部分插入内容。这些部分根据它们所属的数据组进行划分。

在这个例子中，我们有一个包含字母的`title`、`name`和`content`的“主”组。

**这些组在 YARG 被称为`ReportBand`** ，它们对于区分不同类型的数据非常有用。

## 3.将春天与 YARG 融为一体

使用报告生成器的最好方法之一是创建一个可以为我们返回文档的服务。

因此，我们将使用 Spring 并实现一个简单的`@RestController`，它将负责读取模板、获取 JSON、将它加载到文档中并返回一个格式化的`.docx.`

首先，让我们创建一个`DocumentController`:

```java
@RestController
public class DocumentController {

    @GetMapping("/generate/doc")
    public void generateDocument(HttpServletResponse response)
      throws IOException {
    }
} 
```

这将公开文档作为服务的创建。

现在我们将为模板添加加载逻辑:

```java
ReportBuilder reportBuilder = new ReportBuilder();
ReportTemplateBuilder reportTemplateBuilder = new ReportTemplateBuilder()
  .documentPath("./src/main/resources/Letter.docx")
  .documentName("Letter.docx")
  .outputType(ReportOutputType.docx)
  .readFileFromPath();
reportBuilder.template(reportTemplateBuilder.build()); 
```

`ReportBuilder`类将负责创建报告，对模板和数据进行分组。`ReportTemplateBuilder`通过指定文档的路径、名称和输出类型来加载我们之前定义的`Letter.` docx 模板。

然后我们将**将加载的模板**添加到报告构建器中。

现在我们需要定义将要插入到文档中的数据，这将是一个包含以下内容的`Data.json` 文件:

```java
{
    "main": {
        "title" : "INTRODUCTION TO YARG",
        "name" : "Baeldung",
        "content" : "This is the content of the letter, can be anything we like."
    }
}
```

这是一个简单的 JSON 结构，带有一个“main”对象，包含模板所需的标题、名称和内容。

现在，让我们继续将数据加载到我们的`ReportBuilder`:

```java
BandBuilder bandBuilder = new BandBuilder();
String json = FileUtils.readFileToString(
  new File("./src/main/resources/Data.json"));
ReportBand main = bandBuilder.name("Main")
  .query("Main", "parameter=param1 $.main", "json")
  .build();
reportBuilder.band(main);
Report report = reportBuilder.build();
```

这里我们定义了一个`BandBuilder`来创建一个`ReportBand`，这是 YARG 为我们之前在模板文档中定义的数据组使用的抽象。

我们可以看到，我们用完全相同的部分“Main”定义了名称**，然后我们使用查询方法来查找“Main”部分，并声明一个参数，该参数将用于查找填充模板所需的数据。**

值得注意的是，YARG 使用 [JsonPath](https://web.archive.org/web/20220524022231/https://github.com/json-path/JsonPath) 来遍历 JSON，这就是为什么我们看到这个“$。主”语法。

接下来，让我们在查询中指定数据的格式为“json”，然后将波段添加到报告中**，最后构建它**。

最后一步是定义`Reporting`对象，它负责将数据插入模板并生成最终文档:

```java
Reporting reporting = new Reporting();
reporting.setFormatterFactory(new DefaultFormatterFactory());
reporting.setLoaderFactory(
  new DefaultLoaderFactory().setJsonDataLoader(new JsonDataLoader()));
response.setContentType(
 "application/vnd.openxmlformats-officedocument.wordprocessingml.document");
reporting.runReport(
  new RunParams(report).param("param1", json),
  response.getOutputStream());
```

我们使用一个`DefaultFormatterFactory`来支持本文开头列出的常见格式。之后，我们设置负责解析 JSON 的`JsonDataLoader`。

在最后一步中，我们为。docx 格式并运行报告。这将连接 JSON 数据并将其插入模板，将结果输出到响应输出流中。

现在我们可以访问`/generate/doc` URL 来下载文档，我们将在生成的。docx:

[![](img/01eaee08b652c0a22d9f8c32f42f4fa0.png)](/web/20220524022231/https://www.baeldung.com/wp-content/uploads/2017/09/doc.png)

## 4。结论

在本文中，我们展示了如何轻松地将 YARG 与 Spring 集成，并使用其强大的 API 以简单的方式创建文档。

我们使用 JSON 作为数据输入，但是也支持 Groovy 和 SQL。

如果你想了解更多，你可以在这里找到文档。

和往常一样，你可以在 GitHub 上找到完整的例子[。](https://web.archive.org/web/20220524022231/https://github.com/eugenp/tutorials/tree/master/libraries-4)