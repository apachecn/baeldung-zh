# 获取关于 Java 中 PDF 的信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pdf-info>

## 1.概观

在本教程中，我们将了解使用 Java 中的 [iText](https://web.archive.org/web/20221120022800/https://itextpdf.com/) 和 [PDFBox](https://web.archive.org/web/20221120022800/https://pdfbox.apache.org/) 库获取 PDF 文件信息的不同方法。

## 2.使用 iText 库

iText 是一个用于[创建](/web/20221120022800/https://www.baeldung.com/java-pdf-creation)和操作 PDF 文档的库。此外，它还提供了一种获取文档信息的简单方法。

### 2.1.Maven 依赖性

让我们首先在我们的`pom.xml`中声明 [`itextpdf`](https://web.archive.org/web/20221120022800/https://search.maven.org/search?q=g:com.itextpdf%20AND%20a:itextpdf) 依赖项:

```java
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.13.3</version>
</dependency>
```

### 2.2.获取页数

让我们用返回 PDF 文档页数的`getNumberOfPages()`方法创建一个`PdfInfoIText`类:

```java
public class PdfInfoIText {

    public static int getNumberOfPages(final String pdfFile) throws IOException {
        PdfReader reader = new PdfReader(pdfFile);
        int pages = reader.getNumberOfPages();
        reader.close();
        return pages;
    }
}
```

在我们的例子中，首先，**我们使用`PdfReader`类从`File`对象**加载 PDF。之后，我们使用`getNumberOfPages()`方法。最后，我们关闭`PdfReader`对象。让我们为它声明一个测试用例:

```java
@Test
public void givenPdf_whenGetNumberOfPages_thenOK() throws IOException {
    Assert.assertEquals(4, PdfInfoIText.getNumberOfPages(PDF_FILE));
}
```

在我们的测试案例中，我们验证存储在 test `resources`文件夹中的给定 PDF 文件的页数。

### 2.3.获取 PDF 元数据

现在让我们看看如何获取文档的元数据。**我们将使用`getInfo()`方法**。这个方法可以获得文件的信息，比如标题、作者、创建日期、创建者、制作者等等。让我们将`getInfo()`方法添加到我们的`PdfInfoIText`类中:

```java
public static Map<String, String> getInfo(final String pdfFile) throws IOException {
    PdfReader reader = new PdfReader(pdfFile);
    Map<String, String> info = reader.getInfo();
    reader.close();
    return info;
}
```

现在，让我们编写一个测试用例来获取文档的创建者和生产者:

```java
@Test
public void givenPdf_whenGetInfo_thenOK() throws IOException {
    Map<String, String> info = PdfInfoIText.getInfo(PDF_FILE);
    Assert.assertEquals("LibreOffice 4.2", info.get("Producer"));
    Assert.assertEquals("Writer", info.get("Creator"));
}
```

### 2.4.了解 PDF 密码保护

我们现在想知道**文档**是否有密码保护。为此，让我们将`isEncrypted()`方法添加到`PdfInfoIText`类中:

```java
public static boolean isPasswordRequired(final String pdfFile) throws IOException {
    PdfReader reader = new PdfReader(pdfFile);
    boolean isEncrypted = reader.isEncrypted();
    reader.close();
    return isEncrypted;
}
```

现在，让我们创建一个测试用例来看看这个方法是如何工作的:

```java
@Test
public void givenPdf_whenIsPasswordRequired_thenOK() throws IOException {
    Assert.assertFalse(PdfInfoIText.isPasswordRequired(PDF_FILE));
}
```

在下一节中，我们将使用 PDFBox 库做同样的工作。

## 3.使用 PDFBox 库

获取 PDF 文件信息的另一种方法是使用 Apache PDFBox 库。

### 3.1.Maven 依赖性

我们需要在我们的项目中包含 [`pdfbox`](https://web.archive.org/web/20221120022800/https://search.maven.org/search?q=g:org.apache.pdfbox%20AND%20a:pdfbox) Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>3.0.0-RC1</version>
</dependency>
```

### 3.2.获取页数

PDFBox 库提供了处理 PDF 文档的能力。**为了获得页数，我们简单地使用`Loader`类及其`loadPDF()`方法从`File`对象中加载文档。之后，我们使用`PDDocument`类**的 `getNumberOfPages()`方法:

```java
public class PdfInfoPdfBox {

    public static int getNumberOfPages(final String pdfFile) throws IOException {
        File file = new File(pdfFile);
        PDDocument document = Loader.loadPDF(file);
        int pages = document.getNumberOfPages();
        document.close();
        return pages;
    }
}
```

让我们为它创建一个测试用例:

```java
@Test
public void givenPdf_whenGetNumberOfPages_thenOK() throws IOException {
    Assert.assertEquals(4, PdfInfoPdfBox.getNumberOfPages(PDF_FILE));
}
```

### 3.3.获取 PDF 元数据

获取 PDF 元数据也很简单。**我们需要使用`getDocumentInformation()`方法。这个方法将文档元数据(比如文档的作者或创建日期)作为一个`PDDocumentInformation`对象**返回:

```java
public static PDDocumentInformation getInfo(final String pdfFile) throws IOException {
    File file = new File(pdfFile);
    PDDocument document = Loader.loadPDF(file);
    PDDocumentInformation info = document.getDocumentInformation();
    document.close();
    return info;
}
```

让我们为它编写一个测试用例:

```java
@Test
public void givenPdf_whenGetInfo_thenOK() throws IOException {
    PDDocumentInformation info = PdfInfoPdfBox.getInfo(PDF_FILE);
    Assert.assertEquals("LibreOffice 4.2", info.getProducer());
    Assert.assertEquals("Writer", info.getCreator());
}
```

在这个测试用例中，我们只是验证文档的生产者和创建者。

### 3.4.了解 PDF 密码保护

我们可以使用`PDDocument`类的`isEncrypted()`方法检查 PDF 是否有密码保护:

```java
public static boolean isPasswordRequired(final String pdfFile) throws IOException {
    File file = new File(pdfFile);
    PDDocument document = Loader.loadPDF(file);
    boolean isEncrypted = document.isEncrypted();
    document.close();
    return isEncrypted;
}
```

让我们创建一个验证密码保护的测试案例:

```java
@Test
public void givenPdf_whenIsPasswordRequired_thenOK() throws IOException {
    Assert.assertFalse(PdfInfoPdfBox.isPasswordRequired(PDF_FILE));
}
```

## 4.结论

在本文中，我们学习了如何使用两个流行的 Java 库获取关于 PDF 文件的信息。GitHub 上的[提供了本文所示代码的工作版本。](https://web.archive.org/web/20221120022800/https://github.com/eugenp/tutorials/tree/master/pdf-2)