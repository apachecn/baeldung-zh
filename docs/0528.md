# 使用百里香叶生成 PDF 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-generate-pdf>

## 1.概观

在本教程中，我们将通过一个快速实用的例子来学习如何使用[百里香叶](/web/20220728105348/https://www.baeldung.com/thymeleaf-in-spring-mvc)作为模板引擎来生成 pdf。

## 2.Maven 依赖性

首先，让我们添加我们的[百里香叶](https://web.archive.org/web/20220728105348/https://search.maven.org/artifact/org.thymeleaf/thymeleaf)依赖项:

```
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

**百里香本身只是一个模板引擎，它不能自己生成 pdf。为此，我们将把`[flying-saucer-pdf](https://web.archive.org/web/20220728105348/https://search.maven.org/artifact/org.xhtmlrenderer/flying-saucer-pdf)`添加到我们的`pom.xml` :**

```
<dependency>
    <groupId>org.xhtmlrenderer</groupId>
    <artifactId>flying-saucer-pdf</artifactId>
    <version>9.1.20</version>
</dependency>
```

## 3.生成 pdf

接下来，让我们创建一个简单的百里香 HTML 模板—`thymeleaf_template.html`:

```
<html xmlns:th="http://www.thymeleaf.org">
  <body>
    <h3 style="text-align: center; color: green">
      <span th:text="'Welcome to ' + ${to} + '!'"></span>
    </h3>
  </body>
</html>
```

然后，我们将创建一个简单的函数——`parseThymeleafTemplate`——它将解析我们的模板并返回一个 HTML `String`:

```
private String parseThymeleafTemplate() {
    ClassLoaderTemplateResolver templateResolver = new ClassLoaderTemplateResolver();
    templateResolver.setSuffix(".html");
    templateResolver.setTemplateMode(TemplateMode.HTML);

    TemplateEngine templateEngine = new TemplateEngine();
    templateEngine.setTemplateResolver(templateResolver);

    Context context = new Context();
    context.setVariable("to", "Baeldung");

    return templateEngine.process("thymeleaf_template", context);
}
```

最后，让我们**实现一个简单的函数，它接收前面生成的 HTML 作为输入，并将一个 PDF 文件写入我们的主文件夹:**

```
public void generatePdfFromHtml(String html) {
    String outputFolder = System.getProperty("user.home") + File.separator + "thymeleaf.pdf";
    OutputStream outputStream = new FileOutputStream(outputFolder);

    ITextRenderer renderer = new ITextRenderer();
    renderer.setDocumentFromString(html);
    renderer.layout();
    renderer.createPDF(outputStream);

    outputStream.close();
}
```

运行我们的代码后，我们会注意到在我们用户的主目录中有一个名为`thymeleaf.pdf`的文件，看起来像这样: [![](img/f077609794cebc529bfcfeadd1705db1.png)](/web/20220728105348/https://www.baeldung.com/wp-content/uploads/2020/05/baeldung_pdf.png)

正如我们所看到的，文本是绿色的，并按照内联 CSS 中的定义居中对齐。这是一个非常强大的定制 pdf 的工具。

我们应该记住，百里香完全脱离了飞碟，这意味着我们可以使用任何其他模板引擎来创建 pdf，如 [Apache FreeMarker](/web/20220728105348/https://www.baeldung.com/freemarker-operations) 。

## 4.结论

在这个快速教程中，我们学习了如何使用百里香叶作为模板引擎轻松生成 pdf。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/pdf)