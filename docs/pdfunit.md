# PDFUnit 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/pdfunit>

## 1。简介

在本文中，我们将探索用于测试 pdf 的 [PDFUnit](https://web.archive.org/web/20220524023022/http://www.pdfunit.com/) 库。

使用 PDFUnit 提供的强大 API，我们可以处理 pdf 并验证文本、图像、书签和许多其他东西。

我们最终可以使用 PDFUnit 编写非常复杂的测试用例，但是让我们从最常见的用例开始，这些用例将适用于您的大多数产品 pdf，并为进一步的开发提供一个良好的基础。

重要说明:PDFUnit 可免费用于评估目的，但不能用于商业用途。

## 2。安装和设置

当前版本的 PDFUnit (2016.05)在 Maven 中央存储库中不可用。因此，我们需要手动下载和安装 jar。请按照官方网站上的[说明进行手动安装。](https://web.archive.org/web/20220524023022/http://www.pdfunit.com/en/documentation/java/install_update/classpath.html)

## 3。页数

让我们从一个简单的例子开始，它简单地验证给定 PDF 文件中的页数:

```java
@Test
public void givenSinglePage_whenCheckForOnePage_thenSuccess() {

    String filename = getFilePath("sample.pdf");
    AssertThat.document(filename)
      .hasNumberOfPages(1);
}
```

`getFilePath()`是一个简单的方法，与 PDFUnit 无关，它只是将 PDF 文件的路径作为一个`String`返回。

所有的 PDFUnit 测试都从调用`AssertThat.document()`开始，它为测试准备文档。`hasNumberOfPages()`将一个`int`作为参数，指定 PDF 必须包含的页数。在我们的例子中，文件`sample.pdf`只包含一页，所以测试成功了。

如果实际页数与参数不匹配，则会引发异常。

让我们看一个如何测试抛出异常时的场景的例子:

```java
@Test(expected = PDFUnitValidationException.class)
public void givenMultiplePages_whenCheckForOnePage_thenException() {
    String filename = getFilePath("multiple_pages.pdf");
    AssertThat.document(filename)
      .hasNumberOfPages(1);
}
```

在这种情况下，文件`multiple_pages.pdf`包含多个页面。因此抛出了一个`PDFUnitValidationException`异常。

## 4。受密码保护的文件

处理受密码保护的文件再次变得非常简单。唯一的区别是在对`AssertThat.document()`的调用中，我们需要**传递第二个参数，也就是文件**的密码:

```java
@Test
public void givenPwdProtected_whenOpenWithPwd_thenSuccess() {
    String filename = getFilePath("password_protected.pdf");
    String userPassword = "pass1";

    AssertThat.document(filename, userPassword)
      .hasNumberOfPages(1);
}
```

## 5。文本比较

现在让我们将测试 PDF ( `sample.pdf`)与参考 PDF ( `sample_reference.pdf`)进行比较。如果被测文件的文本与参考文件相同，则测试成功:

```java
@Test
public void whenMatchWithReferenceFile_thenSuccess() {
    String testFileName = getFilePath("sample.pdf");
    String referenceFileName = getFilePath("sample_reference.pdf");

    AssertThat.document(testFileName)
      .and(referenceFileName)
      .haveSameText();
}
```

`haveSameText()`是完成两个文件之间文本比较的所有工作的方法。

如果我们不想比较两个文件之间的完整文本，而是想验证某个页面上特定文本的存在，那么`containing()`方法就很方便:

```java
@Test
public void whenPage2HasExpectedText_thenSuccess() {

    String filename = getFilePath("multiple_pages.pdf");
    String expectedText = "Chapter 1, content";

    AssertThat.document(filename)
      .restrictedTo(PagesToUse.getPage(2))
      .hasText()
      .containing(expectedText);
}
```

如果`multiple_pages.pdf`文件的第 2 页在页面的任何地方包含了`expectedText`，那么上面的测试就成功了。除了`expectedText`之外的任何其他文本的存在与否不影响结果。

现在让我们通过验证特定文本是否出现在页面的某个区域而不是整个页面来使测试更加严格。为此，我们需要理解`PageRegion`的概念。

`PageRegion`是在测试中的实际页面内的矩形子部分。`PageRegion`必须完全位于实际页面之下。如果`PageRegion`的任何部分落在实际页面之外，就会导致错误。

A `PageRegion`由四个要素定义:

1.  `leftX`–垂直线离开页面最左边垂直边缘的毫米数
2.  `upperY`–水平线距离页面最高水平边缘的毫米数
3.  `width`–以毫米为单位的区域宽度
4.  `height`–区域的高度，单位为毫米

为了更好地理解这个概念，让我们使用以下属性创建一个`PageRegion`:

1.  `leftX` = 20
2.  `upperY` = 10
3.  `width` = 150
4.  `height` = 50

以下是上述`PageRegion:`的近似图像表示

[![](img/7ef2a7b4eb33a98b3b26b5394065e00a.png)](/web/20220524023022/https://www.baeldung.com/wp-content/uploads/2017/07/PageRegion.png)

一旦概念清楚了，相应的测试用例就相对简单了:

```java
@Test
public void whenPageRegionHasExpectedtext_thenSuccess() {
    String filename = getFilePath("sample.pdf");
    int leftX = 20;
    int upperY = 10;
    int width = 150;
    int height = 50;
    PageRegion regionTitle = new PageRegion(leftX, upperY, width, height);

    AssertThat.document(filename)
      .restrictedTo(PagesToUse.getPage(1))
      .restrictedTo(regionTitle)
      .hasText()
      .containing("Adobe Acrobat PDF Files");
}
```

这里，我们在 PDF 文件的第 1 页中创建了一个`PageRegion`,并验证了该区域中的文本。

## 6。书签

让我们看几个与书签相关的测试案例:

```java
@Test
public void whenHasBookmarks_thenSuccess() {
    String filename = getFilePath("with_bookmarks.pdf");

    AssertThat.document(filename)
      .hasNumberOfBookmarks(5);
}
```

如果 PDF 文件正好有五个书签，测试将会成功。

也可以验证书签的标签:

```java
@Test
public void whenHasBookmarksWithLabel_thenSuccess() {
    String filename = getFilePath("with_bookmarks.pdf");

    AssertThat.document(filename)
      .hasBookmark()
      .withLabel("Chapter 2")
      .hasBookmark()
      .withLinkToPage(3);
}
```

这里我们检查给定的 PDF 是否有一个带有文本“第 2 章”的书签。它还验证是否有链接到第 3 页的书签。

## 7。图像

图像是 PDF 文档的另一个重要方面。再次单元测试 PDF 里面的图像非常容易:

```java
@Test
public void whenHas2DifferentImages_thenSuccess() {
    String filename = getFilePath("with_images.pdf");

    AssertThat.document(filename)
      .hasNumberOfDifferentImages(2);
}
```

该测试验证了在 PDF 中确实使用了两个不同的图像。不同图像的数量是指存储在 PDF 文档中的图像的实际数量。

然而，有可能在文档中存储了单个徽标图像，但是在文档的每一页上都显示了该图像。这是指可见图像的数量，可以多于不同图像的数量。

让我们看看如何验证可见图像:

```java
@Test
public void whenHas2VisibleImages_thenSuccess() {
    String filename = getFilePath("with_images.pdf");
    AssertThat.document(filename)
      .hasNumberOfVisibleImages(2);
}
```

PDFUnit 功能强大，可以逐字节比较图像内容。这也意味着 PDF 中的图像和参考图像必须完全相同。

由于字节比较，不同格式的图像如 BMP 和 PNG 被认为是不平等的:

```java
@Test
public void whenImageIsOnAnyPage_thenSuccess() {
    String filename = getFilePath("with_images.pdf");
    String imageFile = getFilePath("Superman.png");

    AssertThat.document(filename)
      .restrictedTo(AnyPage.getPreparedInstance())
      .hasImage()
      .matching(imageFile);
}
```

注意这里`AnyPage`的用法。我们没有将图像的出现限制在任何特定的页面上，而是限制在整个文档的任何页面上。

除了代表文件名的`String`之外，要比较的图像可以采用`BufferedImage`、`File`、`InputStream`或`URL`的形式。

## 8。嵌入文件

某些 PDF 文档带有嵌入的文件或附件。也有必要测试这些:

```java
@Test
public void whenHasEmbeddedFile_thenSuccess() {
    String filename = getFilePath("with_attachments.pdf");

    AssertThat.document(filename)
      .hasEmbeddedFile();
}
```

这将验证被测文档是否至少有一个嵌入文件。

我们也可以验证嵌入文件的名称:

```java
@Test
public void whenHasmultipleEmbeddedFiles_thenSuccess() {
    String filename = getFilePath("with_attachments.pdf");

    AssertThat.document(filename)
      .hasNumberOfEmbeddedFiles(4)
      .hasEmbeddedFile()
      .withName("complaintform1.xls")
      .hasEmbeddedFile()
      .withName("complaintform2.xls")
      .hasEmbeddedFile()
      .withName("complaintform3.xls");
}
```

**我们可以进一步验证嵌入文件的内容:**

```java
@Test
public void whenEmbeddedFileContentMatches_thenSuccess() {
    String filename = getFilePath("with_attachments.pdf");
    String embeddedFileName = getFilePath("complaintform1.xls");

    AssertThat.document(filename)
      .hasEmbeddedFile()
      .withContent(embeddedFileName);
}
```

本节中的所有示例都相对简单明了。

## 9。结论

在本教程中，我们看到了几个例子，涵盖了与 PDF 测试相关的最常见的用例。

然而，PDFUnit 还能做更多的事情；请务必访问[文档页面](https://web.archive.org/web/20220524023022/http://www.pdfunit.com/en/documentation/java/)了解更多信息。