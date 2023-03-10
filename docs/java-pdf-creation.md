# 用 Java 创建 PDF 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pdf-creation>

## 1。简介

在这个快速教程中，我们将重点介绍如何基于 iText 和 PdfBox 库从头开始创建 PDF 文档。

## 2。Maven 依赖关系

首先，我们需要在项目中包含以下 Maven 依赖项:

```java
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.10</version>
</dependency>
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>2.0.4</version>
</dependency>
```

这些库的最新版本可以在这里找到: [iText](https://web.archive.org/web/20221017042642/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22itextpdf%22) 和 [PdfBox](https://web.archive.org/web/20221017042642/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22pdfbox%22) 。

重要的是要知道 iText 可以在开源的 AGPL 许可下使用，也可以在商业许可下使用。如果我们购买了商业许可证，我们可以保留我们的源代码，允许我们保留我们的知识产权。如果我们使用 AGPL 版本，我们需要免费发布我们的源代码。我们可以通过这个[链接](https://web.archive.org/web/20221017042642/https://itextpdf.com/en/blog/technical-notes/how-do-i-make-sure-my-software-complies-agpl-how-can-i-use-itext-free)来确定如何确保我们的软件符合 AGPL。

我们还需要添加一个额外的依赖项，以防我们需要加密我们的文件。Bouncy Castle 提供程序包包含加密算法的实现，两个库都需要它:

```java
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.56</version>
</dependency> 
```

最新版本的库可以在这里找到:[充气城堡提供商](https://web.archive.org/web/20221017042642/https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on/1.56)。

## 3。概述

iText 和 PdfBox 都是 Java 库，我们用它们来创建和操作 pdf 文件。尽管库的最终输出是相同的，但是它们以不同的方式运行。让我们仔细看看它们。

## 4。在 IText 中创建 Pdf

### 4.1。在 Pdf 中插入文本

让我们看看如何将带有“Hello World”文本的新文件插入 pdf 文件:

```java
Document document = new Document();
PdfWriter.getInstance(document, new FileOutputStream("iTextHelloWorld.pdf"));

document.open();
Font font = FontFactory.getFont(FontFactory.COURIER, 16, BaseColor.BLACK);
Chunk chunk = new Chunk("Hello World", font);

document.add(chunk);
document.close();
```

**使用 iText 库创建 pdf 是基于操作在`Document`** 中实现`Elements`接口的对象(在版本 5.5.10 中有 45 个这样的实现)。

我们可以添加到文档中并使用的最小元素是`Chunk`，它基本上是一个应用了字体的字符串。

此外，我们可以将`Chunk`与其他元素结合，如`Paragraphs`、`Section,`等。，产生好看的文档。

### 4.2。插入图像

**iText 库提供了一种向文档添加图像的简单方法。**我们只需要创建一个`Image`实例，并将其添加到`Document:`中

```java
Path path = Paths.get(ClassLoader.getSystemResource("Java_logo.png").toURI());

Document document = new Document();
PdfWriter.getInstance(document, new FileOutputStream("iTextImageExample.pdf"));
document.open();
Image img = Image.getInstance(path.toAbsolutePath().toString());
document.add(img);

document.close();
```

### 4.3。插入表格

如果我们想在 pdf 中添加表格，可能会遇到一个问题。幸运的是， **iText 提供了这种现成的功能。**

首先，我们需要创建一个 *PdfTable* 对象，并在构造函数中为我们的表提供一些列。

然后，我们可以简单地通过在新创建的表对象上调用 *addCell* 方法来添加新的单元格。只要定义了所有必要的单元格，iText 就会创建表格行。这意味着，一旦我们创建了一个有三列的表格，并向其中添加了八个单元格，将只显示两行，每行三个单元格。

让我们看看这个例子:

```java
Document document = new Document();
PdfWriter.getInstance(document, new FileOutputStream("iTextTable.pdf"));

document.open();

PdfPTable table = new PdfPTable(3);
addTableHeader(table);
addRows(table);
addCustomRows(table);

document.add(table);
document.close();
```

现在我们将创建一个三列三行的新表格。我们将第一行视为一个背景颜色和边框宽度发生变化的表头:

```java
private void addTableHeader(PdfPTable table) {
    Stream.of("column header 1", "column header 2", "column header 3")
      .forEach(columnTitle -> {
        PdfPCell header = new PdfPCell();
        header.setBackgroundColor(BaseColor.LIGHT_GRAY);
        header.setBorderWidth(2);
        header.setPhrase(new Phrase(columnTitle));
        table.addCell(header);
    });
}
```

第二行将由三个单元格组成，只有文本，没有额外的格式:

```java
private void addRows(PdfPTable table) {
    table.addCell("row 1, col 1");
    table.addCell("row 1, col 2");
    table.addCell("row 1, col 3");
}
```

我们也可以在单元格中包含图像。此外，我们可以单独设置每个单元格的格式。

在本例中，我们应用水平和垂直对齐调整:

```java
private void addCustomRows(PdfPTable table) 
  throws URISyntaxException, BadElementException, IOException {
    Path path = Paths.get(ClassLoader.getSystemResource("Java_logo.png").toURI());
    Image img = Image.getInstance(path.toAbsolutePath().toString());
    img.scalePercent(10);

    PdfPCell imageCell = new PdfPCell(img);
    table.addCell(imageCell);

    PdfPCell horizontalAlignCell = new PdfPCell(new Phrase("row 2, col 2"));
    horizontalAlignCell.setHorizontalAlignment(Element.ALIGN_CENTER);
    table.addCell(horizontalAlignCell);

    PdfPCell verticalAlignCell = new PdfPCell(new Phrase("row 2, col 3"));
    verticalAlignCell.setVerticalAlignment(Element.ALIGN_BOTTOM);
    table.addCell(verticalAlignCell);
}
```

### 4.4。文件加密

为了使用 iText 库应用权限，我们需要已经创建了 pdf 文档。在我们的例子中，我们将使用之前生成的`iTextHelloWorld.pdf`文件。

一旦我们使用`PdfReader`加载了文件，我们需要创建一个`PdfStamper,`，我们将使用它向文件应用额外的内容，比如元数据、加密等。：

```java
PdfReader pdfReader = new PdfReader("HelloWorld.pdf");
PdfStamper pdfStamper 
  = new PdfStamper(pdfReader, new FileOutputStream("encryptedPdf.pdf"));

pdfStamper.setEncryption(
  "userpass".getBytes(),
  ".getBytes(),
  0,
  PdfWriter.ENCRYPTION_AES_256
);

pdfStamper.close();
```

在我们的示例中，**我们用两个密码对文件进行了加密:**用户密码(“userpass”)，用户只有只读权限，不能打印；所有者密码(“ownerpass”)，用作主密钥，允许用户完全访问 pdf。

如果我们希望允许用户打印 pdf，那么我们可以传递:

```java
PdfWriter.ALLOW_PRINTING
```

当然，我们也可以混合不同的权限，比如:

```java
PdfWriter.ALLOW_PRINTING | PdfWriter.ALLOW_COPY
```

请记住，当使用 iText 设置访问权限时，我们还创建了一个临时 pdf，应该删除它。如果我们不删除它，任何人都可以访问它。

## 5。在 PdfBox 中创建 Pdf

### 5.1。在 Pdf 中插入文本

与`iText`、**相反，`PdfBox`库提供了一个基于流操作的 API。没有像`Chunk` / `Paragraph,`这样的职业。`PDDocument`类是内存中的 Pdf 表示，用户通过操作`PDPageContentStream`类来写入数据。**

让我们看一下代码示例:

```java
PDDocument document = new PDDocument();
PDPage page = new PDPage();
document.addPage(page);

PDPageContentStream contentStream = new PDPageContentStream(document, page);

contentStream.setFont(PDType1Font.COURIER, 12);
contentStream.beginText();
contentStream.showText("Hello World");
contentStream.endText();
contentStream.close();

document.save("pdfBoxHelloWorld.pdf");
document.close();
```

### 5.2。插入图像

插入图像也很简单。

我们需要加载一个文件并创建一个`PDImageXObject`，随后在文档上绘制它(需要提供精确的 x，y 坐标):

```java
PDDocument document = new PDDocument();
PDPage page = new PDPage();
document.addPage(page);

Path path = Paths.get(ClassLoader.getSystemResource("Java_logo.png").toURI());
PDPageContentStream contentStream = new PDPageContentStream(document, page);
PDImageXObject image 
  = PDImageXObject.createFromFile(path.toAbsolutePath().toString(), document);
contentStream.drawImage(image, 0, 0);
contentStream.close();

document.save("pdfBoxImage.pdf");
document.close(); 
```

### 5.3。插入表格

不幸的是，`PdfBox`没有提供任何允许我们创建表格的现成方法。在这种情况下，我们能做的是手动绘制，逐字绘制每条线，直到我们的绘图类似于我们想要的表格。

### 5.4。文件加密

**`PdfBox`库为用户提供了加密和调整文件权限的能力。**与`iText`相比，它不需要我们使用已经存在的文件，因为我们只需要使用`PDDocument`。Pdf 文件权限由`AccessPermission`类处理，在这里我们可以设置用户是否能够修改、提取内容或打印文件。

随后，我们创建一个`StandardProtectionPolicy`对象，为文档添加基于密码的保护。我们可以指定两种类型的密码。用户密码允许用户使用应用的访问权限打开文件，所有者密码对文件没有限制:

```java
PDDocument document = new PDDocument();
PDPage page = new PDPage();
document.addPage(page);

AccessPermission accessPermission = new AccessPermission();
accessPermission.setCanPrint(false);
accessPermission.setCanModify(false);

StandardProtectionPolicy standardProtectionPolicy 
  = new StandardProtectionPolicy("ownerpass", "userpass", accessPermission);
document.protect(standardProtectionPolicy);
document.save("pdfBoxEncryption.pdf");
document.close(); 
```

我们的例子表明，如果用户提供了用户密码，文件就不能被修改或打印。

## 6。结论

在本文中，我们学习了如何在两个流行的 Java 库中创建 pdf 文件。

本文的完整示例可以在 GitHub 上基于 Maven 的项目[中找到。](https://web.archive.org/web/20221017042642/https://github.com/eugenp/tutorials/tree/master/pdf)