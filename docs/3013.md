# 使用 OpenPDF 将 HTML 转换为 PDF

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-html-to-pdf>

## 1.概观

在这个快速教程中，**我们将看看如何在 Java 中使用 OpenPDF 以编程方式将 HTML 文件转换成 PDF 格式**。

## 2.OpenPDF

OpenPDF 是一个免费的 Java 库，用于在 LGPL 和 MPL 许可下创建和编辑 PDF 文件。这是 iText 程序的一个分支。事实上，在版本 5 之前，使用 OpenPDF 生成 PDF 的代码与 iText API 几乎相同。这是一个维护良好的用 Java 制作 pdf 的解决方案。

## 3.使用飞碟转换

飞碟是一个 Java 库，它允许我们使用 CSS 2.1 来呈现格式良好的 XML(或 XHTML)以进行样式和格式设置，生成 PDF、图片和 swing 面板输出。

### 3.1. **Maven 依赖关系**

我们将从 Maven 依赖项开始:

```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.13.1</version>
</dependency>
<dependency>
    <groupId>org.xhtmlrenderer</groupId>
    <artifactId>flying-saucer-pdf-openpdf</artifactId>
    <version>9.1.20</version>
</dependency> 
```

我们将使用库 [`jsoup`](https://web.archive.org/web/20220928075047/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jsoup%22%20AND%20a%3A%22jsoup%22) 来解析 HTML 文件、输入流、URL 甚至字符串。它提供了 DOM(文档对象模型)遍历功能、CSS 和类似 jQuery 的选择器来从 HTML 中提取数据。

[`**flying-saucer-pdf-openpdf**`](https://web.archive.org/web/20220928075047/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.xhtmlrenderer%22%20AND%20a%3A%22flying-saucer-pdf-openpdf%22) **库接受 HTML 文件的 XML 表示作为输入，应用 CSS 格式和样式**，并输出 PDF。

### 3.2.HTML 到 PDF

在本教程中，我们将尝试涵盖在 HTML 到 PDF 转换中可能遇到的简单实例，例如 HTML 中的图像和样式，使用飞碟和 OpenPDF。我们还将讨论如何定制代码来接受外部样式、图像和字体。

让我们来看看我们的样本 HTML 代码:

```
<html>
    <head>
        <style>
            .center_div {
                border: 1px solid gray;
                margin-left: auto;
                margin-right: auto;
                width: 90%;
                background-color: #d0f0f6;
                text-align: left;
                padding: 8px;
            }
        </style>
        <link href="style.css" rel="stylesheet">
    </head>
    <body>
        <div class="center_div">
            <h1>Hello Baeldung!</h1>
            <img src="Java_logo.png">
            <div class="myclass">
                <p>This is the tutorial to convert html to pdf.</p>
            </div>
        </div>
    </body>
</html>
```

要将 HTML 转换为 PDF，我们将首先从定义的位置读取 HTML 文件:

```
File inputHTML = new File(HTML);
```

下一步，我们将使用 [`jsoup`](/web/20220928075047/https://www.baeldung.com/java-with-jsoup) 将上述 HTML 文件转换为`jsoup` `Document`来呈现 XHTML。

下面给出的是 XHTML 输出:

```
Document document = Jsoup.parse(inputHTML, "UTF-8");
document.outputSettings().syntax(Document.OutputSettings.Syntax.xml);
return document;
```

现在，作为最后一步，让我们从上一步生成的 XHTML 文档创建一个 PDF。`ITextRenderer`将获取这个 XHTML 文档并创建一个输出 PDF 文件。注意**我们将代码包装在`[try-with-resources](/web/20220928075047/https://www.baeldung.com/java-try-with-resources)`块中，以确保输出流是关闭的** :

```
try (OutputStream outputStream = new FileOutputStream(outputPdf)) {
    ITextRenderer renderer = new ITextRenderer();
    SharedContext sharedContext = renderer.getSharedContext();
    sharedContext.setPrint(true);
    sharedContext.setInteractive(false);
    renderer.setDocumentFromString(xhtml.html());
    renderer.layout();
    renderer.createPDF(outputStream);
}
```

### 3.3.定制外部样式

我们可以将 HTML 输入文档中使用的额外字体注册到`ITextRenderer`,这样它就可以在生成 PDF 时包含这些字体:

```
renderer.getFontResolver().addFont(getClass().getClassLoader().getResource("fonts/PRISTINA.ttf").toString(), true);
```

`ITextRenderer`可能需要注册相关 URL 以访问外部样式:

```
String baseUrl = FileSystems.getDefault()
  .getPath("src/main/resources/")
  .toUri().toURL().toString();
renderer.setDocumentFromString(xhtml, baseUrl); 
```

我们可以通过实现`ReplacedElementFactory` : 来定制图像相关的属性

```
public ReplacedElement createReplacedElement(LayoutContext lc, BlockBox box, UserAgentCallback uac, int cssWidth, int cssHeight) {
    Element e = box.getElement();
    String nodeName = e.getNodeName();
    if (nodeName.equals("img")) {
        String imagePath = e.getAttribute("src");
        try {
            InputStream input = new FileInputStream("src/main/resources/"+imagePath);
            byte[] bytes = IOUtils.toByteArray(input);
            Image image = Image.getInstance(bytes);
            FSImage fsImage = new ITextFSImage(image);
            if (cssWidth != -1 || cssHeight != -1) {
                fsImage.scale(cssWidth, cssHeight);
            } else {
                fsImage.scale(2000, 1000);
            }
            return new ITextImageElement(fsImage);
        } catch (Exception e1) {
            e1.printStackTrace();
        }
    }
    return null;
}
```

注意:上面的代码将基本路径作为图像路径的前缀，并在没有提供默认图像大小的情况下设置默认图像大小。

然后，我们可以将自定义的`ReplacedElementFactory` 添加到 `SharedContext`中:

```
sharedContext.setReplacedElementFactory(new CustomElementFactoryImpl()); 
```

## 4.使用 Open HTML 转换

Open HTML to PDF 是一个 Java 库，它使用 CSS 2.1(以及更高版本的标准)输出格式良好的 XML/XHTML(甚至一些 HTML5)到 PDF 或图片，用于布局和格式。

### 4.1.Maven 依赖性

除了上面显示的`jsoup`库之外，我们还需要添加几个打开的 HTML 到 PDF 库到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>com.openhtmltopdf</groupId>
    <artifactId>openhtmltopdf-core</artifactId>
    <version>1.0.6</version>
</dependency>
<dependency>
    <groupId>com.openhtmltopdf</groupId>
    <artifactId>openhtmltopdf-pdfbox</artifactId>
    <version>1.0.6</version>
</dependency>
```

**库 [`openhtmltopdf-core`](https://web.archive.org/web/20220928075047/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.openhtmltopdf%22%20AND%20a%3A%22openhtmltopdf-core%22) 呈现格式良好的 XML/XHTML，`[openhtmltopdf-pdfbox](https://web.archive.org/web/20220928075047/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.openhtmltopdf%22%20AND%20a%3A%22openhtmltopdf-pdfbox%22)`从呈现的 XHTML 表示生成 PDF 文档**。

### 4.2.HTML 到 PDF

在这个程序中，要使用 Open HTML 将 HTML 转换成 PDF，我们将使用 3.2 节中提到的相同的 HTML。我们将首先把 HTML 文件转换成一个`jsoup` `Document`，就像我们在前面的例子中展示的那样。

在最后一步中，为了从 XHTML 文档创建一个 PDF， **`PdfRendererBuilder`将获取这个 XHTML 文档并创建一个 PDF 作为输出文件**。同样，我们使用`try-with-resources`来包装我们的逻辑:

```
try (OutputStream os = new FileOutputStream(outputPdf)) {
    PdfRendererBuilder builder = new PdfRendererBuilder();
    builder.withUri(outputPdf);
    builder.toStream(os);
    builder.withW3cDocument(new W3CDom().fromJsoup(doc), "/");
    builder.run();
}
```

### 4.3.定制外部样式

我们可以将 HTML 输入文档中使用的额外字体注册到`PdfRendererBuilder`,这样它就可以将它们包含在 PDF 中:

```
builder.useFont(new File(getClass().getClassLoader().getResource("fonts/PRISTINA.ttf").getFile()), "PRISTINA");
```

`PdfRendererBuilder`库也可能需要注册相对 URL 来访问外部样式，类似于我们之前的例子:

```
String baseUrl = FileSystems.getDefault()
  .getPath("src/main/resources/")
  .toUri().toURL().toString();
builder.withW3cDocument(new W3CDom().fromJsoup(doc), baseUrl);
```

## 5.结论

在本文中，我们学习了如何使用飞碟将 HTML 转换为 PDF 并打开 HTML。我们还讨论了如何注册外部字体、样式和定制。

按照惯例，本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220928075047/https://github.com/eugenp/tutorials/tree/master/pdf)