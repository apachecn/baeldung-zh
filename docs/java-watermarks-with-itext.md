# 在 Java 中对 iText 使用水印

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-watermarks-with-itext>

## 1.概观

**[iText](/web/20221229072819/https://www.baeldung.com/java-pdf-creation) PDF 是一个用于创建和操作 PDF 文件** 的 Java 库。 **水印帮助保护机密信息** 。

在本教程中，我们将通过创建一个带水印的新 PDF 文件来探索 iText PDF 库。我们还将在现有的 PDF 文件中添加水印。

## 2.Maven 依赖性

在本教程中，我们将使用 Maven 来管理我们的依赖关系。我们将需要 [iText](https://web.archive.org/web/20221229072819/https://search.maven.org/artifact/com.itextpdf/itext7-core) 依赖来开始使用 iText PDF 库。同样，我们需要 [`AssertJ`](https://web.archive.org/web/20221229072819/https://search.maven.org/search?q=a:assertj-core) 依赖来进行测试。我们将把这两个依赖项添加到我们的`pom.xml` :

```java
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itext7-core</artifactId>
    <version>7.2.4</version>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.23.1</version>
    <scope>test</scope>
</dependency>
```

## 3.水印

水印有助于在文档或图像文件上覆盖或隐藏文字或徽标。 **对于版权保护、数字产品的营销、防止假冒** 等都至关重要。

在本教程中，我们将在生成的 PDF 中添加机密水印。水印将防止未经授权使用我们生成的 PDF:

[![watermark on PDF](img/34a88047d58ff3d0dc74d4604f708c6e.png)](/web/20221229072819/https://www.baeldung.com/wp-content/uploads/2022/12/watermarks.png)

## 4.用 iText 生成 PDF

在本文中，让我们策划一个故事，并使用 iText PDF 库将我们的故事转换为 PDF 格式。我们先写一个简单的程序， `StoryTime` 。首先，我们将声明两个 `String` 类型的变量。我们将把我们的故事存储在声明的变量中:

```java
public class StoryTime {
    String aliceStory = "I am ...";
    String paulStory = "I am Paul ..";
}
```

为了简单起见，我们将缩短`String`值。然后，让我们声明一个`String`类型的变量，它将存储我们生成的 PDF 的输出路径:

```java
public static final String OUTPUT_DIR = "output/alice.pdf";
```

最后，让我们创建一个包含程序逻辑的方法。我们将创建一个 `PdfWriter ` 的实例来指定我们的输出路径和名称。

接下来，我们将创建一个`PdfDocument`实例来处理我们的 PDF 文件。为了将我们的`String`值添加到 PDF 文档中，我们将创建一个新的 `Document` : 实例

```java
public void createPdf(String output) throws IOException {

    PdfWriter writer = new PdfWriter(output);
    PdfDocument pdf = new PdfDocument(writer);
    try (Document document = new Document(pdf, PageSize.A4, false)) {
        document.add(new Paragraph(aliceSpeech)
          .setFont(PdfFontFactory.createFont(StandardFonts.TIMES_ROMAN)));
        document.add(new Paragraph(paulSpeech)
          .setFont(PdfFontFactory.createFont(StandardFonts.TIMES_ROMAN)));
        document.close();
    }
}
```

我们的方法会生成一个新的 PDF 文件，并存储在 `OUTPUT_DIR` 中。

## 5.向生成的 PDF 添加水印

在上一节中，我们使用 iText PDF 库生成了一个 PDF 文件。**生成 PDF 首先有助于了解页面大小、旋转和页数。这有助于有效地添加水印** 。让我们给这个简单的程序添加更多的逻辑。我们的程序将添加水印生成的 PDF。

首先，让我们创建一个指定水印属性的方法。我们将设置`Font``fontSize``Opacity`我们的水印:

```java
public Paragraph createWatermarkParagraph(String watermark) throws IOException {

    PdfFont font = PdfFontFactory.createFont(StandardFonts.HELVETICA);
    Text text = new Text(watermark);
    text.setFont(font);
    text.setFontSize(56);
    text.setOpacity(0.5f);
    return new Paragraph(text);
}
```

接下来，让我们创建一个包含向 PDF 文档添加水印的逻辑的方法。该方法将以 `Document` 、 `Paragraph` 、 `offset` 为自变量。我们将计算放置水印段落的位置和旋转:

```java
public void addWatermarkToGeneratedPDF(Document document, int pageIndex, 
  Paragraph paragraph, float verticalOffset) {

    PdfPage pdfPage = document.getPdfDocument().getPage(pageIndex);
    PageSize pageSize = (PageSize) pdfPage.getPageSizeWithRotation();
    float x = (pageSize.getLeft() + pageSize.getRight()) / 2;
    float y = (pageSize.getTop() + pageSize.getBottom()) / 2;
    float xOffset = 100f / 2;
    float rotationInRadians = (float) (PI / 180 * 45f);
    document.showTextAligned(paragraph, x - xOffset, y + verticalOffset, 
      pageIndex, CENTER, TOP, rotationInRadians);
}
```

我们通过调用 `showTextAligned()` 方法将水印段落添加到我们的文档中。接下来，让我们编写一个生成新 PDF 并添加水印的方法。我们将调用`createWatermarkParagraph()`方法和`addWatermarkToGeneratedPDF()`方法:

```java
public void createNewPDF() throws IOException {

    StoryTime storyTime = new StoryTime();
    String waterMark = "CONFIDENTIAL";
    PdfWriter writer = new PdfWriter(storyTime.OUTPUT_FILE);
    PdfDocument pdf = new PdfDocument(writer);

    try (Document document = new Document(pdf)) {
        document.add(new Paragraph(storyTime.alice)
          .setFont(PdfFontFactory.createFont(StandardFonts.TIMES_ROMAN)));
        document.add(new Paragraph(storyTime.paul));
        Paragrapgh paragraph = storyTime.createWatermarkParagraph(waterMark);
        for (int i = 1; i <= document.getPdfDocument().getNumberOfPages(); i++) {
            storyTime.addWatermarkToGeneratedPDF(document, i, paragraph, 0f);
        }
    }
}
```

最后，让我们编写一个单元测试来验证水印的存在:

```java
@Test
public void givenNewTexts_whenGeneratingNewPDFWithIText() throws IOException {

    StoryTime storyTime = new StoryTime();
    String waterMark = "CONFIDENTIAL";
    LocationTextExtractionStrategy extStrategy = new LocationTextExtractionStrategy();
    try (PdfDocument pdfDocument = new PdfDocument(new PdfReader(storyTime.OUTPUT_FILE))) {
        for (int i = 1; i <= pdfDocument.getNumberOfPages(); i++) {
            String textFromPage = getTextFromPage(pdfDocument.getPage(i), extStrategy);
            assertThat(textFromPage).contains(waterMark);
        }
    }
}
```

我们的测试验证了生成的 PDF 中是否存在水印。

## 6.向现有 PDF 添加水印

**iText PDF 库使向现有 PDF 添加水印变得容易**。我们首先将 PDF 文档加载到我们的程序中。并使用 iText 库来操作我们现有的 PDF。

首先，我们需要创建一个添加水印段落的方法。因为我们在上一节中创建了一个，所以我们也可以在这里使用它。

接下来，我们将创建一个包含逻辑的方法，帮助我们在现有的 PDF 中添加水印。该方法将接受 `Document` ， `Paragraph` ， `PdfExtGState, pageIndex, and offSet` 作为参数。在该方法中，我们将创建一个新的 `PdfCanvas` 实例，将数据写入我们的 PDF 内容流。

然后，我们将计算我们的水印在 PDF 上的位置和旋转。我们将刷新文档并释放状态以提高性能:

```java
public void addWatermarkToExistingPDF(Document document, int pageIndex,
  Paragraph paragraph, PdfExtGState graphicState, float verticalOffset) {

    PdfDocument pdfDocument = document.getPdfDocument();
    PdfPage pdfPage = pdfDocument.getPage(pageIndex);
    PageSize pageSize = (PageSize) pdfPage.getPageSizeWithRotation();
    float x = (pageSize.getLeft() + pageSize.getRight()) / 2;
    float y = (pageSize.getTop() + pageSize.getBottom()) / 2;

    PdfCanvas over = new PdfCanvas(pdfDocument.getPage(pageIndex));
    over.saveState();
    over.setExtGState(graphicState);
    float xOffset = 14 / 2;
    float rotationInRadians = (float) (PI / 180 * 45f);

    document.showTextAligned(paragraph, x - xOffset, y + verticalOffset, 
      pageIndex, CENTER, TOP, rotationInRadians);
    document.flush();
    over.restoreState();
    over.release();
}
```

最后，让我们写一个方法来添加一个水印到一个现有的 PDF。我们将调用 `createWatermarkParagraph()` 来添加一个水印段落。同样，我们将调用 `addWatermarkToExistingPDF()` 来处理向页面添加水印的任务:

```java
public void addWatermarkToExistingPdf() throws IOException {

    StoryTime storyTime = new StoryTime();
    String outputPdf = "output/aliceNew.pdf";
    String watermark = "CONFIDENTIAL";

    try (PdfDocument pdfDocument = new PdfDocument(new PdfReader("output/alice.pdf"), 
      new PdfWriter(outputPdf))) {
        Document document = new Document(pdfDocument);
        Paragraph paragraph = storyTime.createWatermarkParagraph(watermark);
        PdfExtGState transparentGraphicState = new PdfExtGState().setFillOpacity(0.5f);
        for (int i = 1; i <= document.getPdfDocument().getNumberOfPages(); i++) {
            storyTime.addWatermarkToExistingPage(document, i, paragraph, 
              transparentGraphicState, 0f);
        }
    }
}
```

让我们编写一个单元测试来验证水印的存在:

```java
@Test
public void givenAnExistingPDF_whenManipulatedPDFWithITextmark() throws IOException {
    StoryTime storyTime = new StoryTime();
    String outputPdf = "output/aliceupdated.pdf";
    String watermark = "CONFIDENTIAL";

    LocationTextExtractionStrategy extStrategy 
      = new LocationTextExtractionStrategy();
    try (PdfDocument pdfDocument = new PdfDocument(new PdfReader(outputPdf))) {
        for (int i = 1; i <= pdfDocument.getNumberOfPages(); i++) {
            String textFromPage = getTextFromPage(pdfDocument.getPage(i), extStrategy);
            assertThat(textFromPage).contains(watermark);
        }
    }
}
```

我们的测试验证了现有 PDF 中水印的存在。

## 7.结论

在本教程中，我们通过生成一个新的 PDF 来探索 iText PDF 库。我们在生成的 PDF 中添加了水印，后来又在现有的 PDF 中添加了水印。iText 库在处理 pdf 时看起来很强大。完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221229072819/https://github.com/eugenp/tutorials/tree/master/libraries-files)