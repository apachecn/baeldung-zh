# Java 中的 PDF 转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/pdf-conversions-java>

## 1。简介

在这篇简短的文章中，我们将重点关注在 Java 中进行 PDF 文件和其他格式之间的编程**转换。**

更具体地说，我们将描述如何使用多个 Java 开源库将 pdf 保存为图像文件，如 PNG 或 JPEG，将 pdf 转换为 Microsoft Word 文档，导出为 HTML，以及提取文本。

## 2。Maven 依赖关系

我们要看的第一个库是 **[Pdf2Dom](https://web.archive.org/web/20220922043838/http://cssbox.sourceforge.net/pdf2dom/documentation.php)** 。让我们从需要添加到项目中的 Maven 依赖项开始:

```
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox-tools</artifactId>
    <version>2.0.25</version>
</dependency>
<dependency>
    <groupId>net.sf.cssbox</groupId>
    <artifactId>pdf2dom</artifactId>
    <version>2.0.1</version>
</dependency>
```

我们将使用第一个依赖项来加载选定的 PDF 文件。第二个依赖项负责转换本身。最新版本可以在这里找到: [pdfbox-tools](https://web.archive.org/web/20220922043838/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22pdfbox-tools%22) 和 [pdf2dom](https://web.archive.org/web/20220922043838/https://search.maven.org/classic/#search%7Cga%7C1%7Cpdf2dom) 。

此外，我们将使用 **[iText](https://web.archive.org/web/20220922043838/http://itextpdf.com/)** 从 PDF 文件中提取文本，并使用 **[POI](https://web.archive.org/web/20220922043838/https://poi.apache.org/)** 创建。`docx`文档。

让我们看看我们需要包含在项目中的 Maven 依赖项:

```
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.10</version>
</dependency>
<dependency>
    <groupId>com.itextpdf.tool</groupId>
    <artifactId>xmlworker</artifactId>
    <version>5.5.10</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.15</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-scratchpad</artifactId>
    <version>3.15</version>
</dependency>
```

iText 的最新版本可以在[这里](https://web.archive.org/web/20220922043838/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.itextpdf%22)找到，你也可以在这里寻找 Apache POI [。](https://web.archive.org/web/20220922043838/https://search.maven.org/classic/#search%7Cga%7C1%7Cpoi)

## 3。PDF 和 HTML 转换

为了处理 HTML 文件，我们将使用**[Pdf 2 DOM](https://web.archive.org/web/20220922043838/http://cssbox.sourceforge.net/pdf2dom/documentation.php)**——一个将文档转换成 HTML DOM 表示的 PDF 解析器。然后，可以将获得的 DOM 树序列化为 HTML 文件或进一步处理。

要将 PDF 转换成 HTML，我们需要使用 XMLWorker，这是由 **[iText](https://web.archive.org/web/20220922043838/http://itextpdf.com/)** 提供的库。

### 3.1。PDF 转 HTML

让我们来看一个从 PDF 到 HTML 的简单转换:

```
private void generateHTMLFromPDF(String filename) {
    PDDocument pdf = PDDocument.load(new File(filename));
    Writer output = new PrintWriter("src/output/pdf.html", "utf-8");
    new PDFDomTree().writeText(pdf, output);

    output.close();
} 
```

在上面的代码片段中，我们使用 PDFBox 中的 load API 加载 PDF 文件。加载 PDF 后，我们使用解析器解析文件并写入由`java.io.Writer.`指定的输出

注意**将 PDF 转换成 HTML 从来不是 100%的像素到像素的结果。**结果取决于特定 PDF 文件的复杂性和结构。

### 3.2。HTML 到 PDF

现在，让我们看看从 HTML 到 PDF 的转换:

```
private static void generatePDFFromHTML(String filename) {
    Document document = new Document();
    PdfWriter writer = PdfWriter.getInstance(document,
      new FileOutputStream("src/output/html.pdf"));
    document.open();
    XMLWorkerHelper.getInstance().parseXHtml(writer, document,
      new FileInputStream(filename));
    document.close();
}
```

注意**将 HTML 转换成 PDF，你需要确保 HTML 有正确开始和结束的所有标签，否则 PDF 不会被创建。**这种方法的积极方面是，创建的 PDF 与 HTML 文件完全一样。

## 4. **PDF 到图像的转换**

有许多方法可以将 PDF 文件转换为图像。其中最流行的解决方案被命名为 **[Apache PDFBox](https://web.archive.org/web/20220922043838/https://pdfbox.apache.org/)** 。这个库是一个用于处理 PDF 文档的开源 Java 工具。对于图像到 PDF 的转换，我们将再次使用 **[iText](https://web.archive.org/web/20220922043838/http://itextpdf.com/)** 。

### 4.1。PDF 转图像

要开始将 pdf 转换为图像，我们需要使用上一节提到的依赖关系—`pdfbox-tools`。

让我们看一下代码示例:

```
private void generateImageFromPDF(String filename, String extension) {
    PDDocument document = PDDocument.load(new File(filename));
    PDFRenderer pdfRenderer = new PDFRenderer(document);
    for (int page = 0; page < document.getNumberOfPages(); ++page) {
        BufferedImage bim = pdfRenderer.renderImageWithDPI(
          page, 300, ImageType.RGB);
        ImageIOUtil.writeImage(
          bim, String.format("src/output/pdf-%d.%s", page + 1, extension), 300);
    }
    document.close();
}
```

上述代码中很少有重要的部分。我们需要使用`PDFRenderer`，以便将 PDF 渲染为`BufferedImage`。此外，PDF 文件的每一页都需要单独呈现。

最后，我们使用 Apache PDFBox 工具中的`ImageIOUtil`，用我们指定的扩展名编写一个图像。可能的文件格式是`jpeg, jpg, gif, tiff` 或`png.`

**请注意，Apache PDFBox 是一个高级工具**–我们可以从头开始创建自己的 PDF 文件，在 PDF 文件中填写表格，对 PDF 文件进行签名和/或加密。

### 4.2。图像到 PDF

让我们看一下代码示例:

```
private static void generatePDFFromImage(String filename, String extension) {
    Document document = new Document();
    String input = filename + "." + extension;
    String output = "src/output/" + extension + ".pdf";
    FileOutputStream fos = new FileOutputStream(output);

    PdfWriter writer = PdfWriter.getInstance(document, fos);
    writer.open();
    document.open();
    document.add(Image.getInstance((new URL(input))));
    document.close();
    writer.close();
}
```

**请注意，我们可以提供一个文件形式的图像，或从 URL 加载，如上面的例子所示。**此外，我们可以使用的输出文件的扩展名是`jpeg, jpg, gif, tiff` 或`png.`

## 5. **PDF 到文本的转换**

为了从 PDF 文件中提取原始文本，我们还将再次使用 **[Apache PDFBox](https://web.archive.org/web/20220922043838/https://pdfbox.apache.org/)** 。对于文本到 PDF 的转换，我们将使用 **[iText](https://web.archive.org/web/20220922043838/http://itextpdf.com/)** 。

### 5.1。PDF 转文本

我们创建了一个名为`generateTxtFromPDF(…)`的方法，并将它分为三个主要部分:PDF 文件的加载、文本的提取和最终文件的创建。

让我们从装载零件开始:

```
File f = new File(filename);
String parsedText;
PDFParser parser = new PDFParser(new RandomAccessFile(f, "r"));
parser.parse();
```

为了读取 PDF 文件，我们使用`PDFParser`，带有“r”(读取)选项。此外，我们需要使用`parser.parse()`方法，这将导致 PDF 被解析为一个流，并填充到`COSDocument`对象中。

让我们来看看提取文本部分:

```
COSDocument cosDoc = parser.getDocument();
PDFTextStripper pdfStripper = new PDFTextStripper();
PDDocument pdDoc = new PDDocument(cosDoc);
parsedText = pdfStripper.getText(pdDoc);
```

在第一行中，我们将把`COSDocument`保存在`cosDoc`变量中。然后它将被用来构建`PDocument`，这是 PDF 文档的内存表示。最后，我们将使用`PDFTextStripper`返回文档的原始文本。在所有这些操作之后，我们将需要使用`close()`方法来关闭所有被使用的流`.`

在最后一部分中，我们将使用简单的 Java `PrintWriter`将文本保存到新创建的文件中:

```
PrintWriter pw = new PrintWriter("src/output/pdf.txt");
pw.print(parsedText);
pw.close();
```

请注意，您不能在纯文本文件中保留格式，因为它只包含文本。

### 5.2。文本到 PDF

将文本文件转换成 PDF 有点棘手。为了保持文件格式，你需要应用额外的规则。

在下面的例子中，我们没有考虑文件的格式。

首先，我们需要定义 PDF 文件的大小、版本和输出文件。让我们看一下代码示例:

```
Document pdfDoc = new Document(PageSize.A4);
PdfWriter.getInstance(pdfDoc, new FileOutputStream("src/output/txt.pdf"))
  .setPdfVersion(PdfWriter.PDF_VERSION_1_7);
pdfDoc.open();
```

在下一步中，我们将定义字体以及用于生成新段落的命令:

```
Font myfont = new Font();
myfont.setStyle(Font.NORMAL);
myfont.setSize(11);
pdfDoc.add(new Paragraph("\n"));
```

最后，我们将在新创建的 PDF 文件中添加段落:

```
BufferedReader br = new BufferedReader(new FileReader(filename));
String strLine;
while ((strLine = br.readLine()) != null) {
    Paragraph para = new Paragraph(strLine + "\n", myfont);
    para.setAlignment(Element.ALIGN_JUSTIFIED);
    pdfDoc.add(para);
}	
pdfDoc.close();
br.close();
```

## 6. **PDF 到 Docx 的转换**

从 Word 文档创建 PDF 文件并不容易，我们不会在这里讨论这个主题。我们推荐第三方库来做，像 **[jWordConvert](https://web.archive.org/web/20220922043838/https://www.qoppa.com/wordconvert/)** 。

要从 PDF 创建 Microsoft Word 文件，我们需要两个库。这两个库都是开源的。第一个是 **[iText](https://web.archive.org/web/20220922043838/http://itextpdf.com/)** ，用于从 PDF 文件中提取文本。第二个是 **[POI](https://web.archive.org/web/20220922043838/https://poi.apache.org/)** ，用于创建。`docx`文档。

让我们来看看 PDF 加载部分的代码片段:

```
XWPFDocument doc = new XWPFDocument();
String pdf = filename;
PdfReader reader = new PdfReader(pdf);
PdfReaderContentParser parser = new PdfReaderContentParser(reader); 
```

加载 PDF 后，我们需要在循环中分别读取和呈现每个页面，然后写入输出文件:

```
for (int i = 1; i <= reader.getNumberOfPages(); i++) {
    TextExtractionStrategy strategy =
      parser.processContent(i, new SimpleTextExtractionStrategy());
    String text = strategy.getResultantText();
    XWPFParagraph p = doc.createParagraph();
    XWPFRun run = p.createRun();
    run.setText(text);
    run.addBreak(BreakType.PAGE);
}
FileOutputStream out = new FileOutputStream("src/output/pdf.docx");
doc.write(out);
// Close all open files
```

请注意，使用`SimpleTextExtractionStrategy()`提取策略，我们将丢失所有格式规则。为了修复它，使用这里描述的提取策略，实现一个更复杂的解决方案。

## 7. **PDF 到 X 商业图书馆**

在前面的章节中，我们描述了开源库。值得注意的图书馆很少，但它们是有报酬的:

*   [jPDFImages](https://web.archive.org/web/20220922043838/https://www.qoppa.com/pdfimages/)–jPDFImages 可以从 PDF 文档中的页面创建图像，并将其导出为 JPEG、TIFF 或 PNG 图像。
*   [JPEDAL](https://web.archive.org/web/20220922043838/https://www.idrsolutions.com/jpedal/)–JPEDAL 是一个积极开发的非常强大的原生 Java PDF 库 SDK，用于打印、查看和转换文件
*   这是另一个 Web/HTML 到 PDF 和 PDF 到 Web/HTML 的转换库，具有高级 GUI

## 8.**结论**

在本文中，我们讨论了将 PDF 文件转换成各种格式的方法。

本教程的完整实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目。为了进行测试，只需运行示例并在`output`文件夹中查看结果。