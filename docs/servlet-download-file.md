# 在 Servlet 中下载文件的示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/servlet-download-file>

## 1.概观

web 应用程序的一个共同特征是能够下载文件。

在本教程中，**我们将介绍一个创建可下载文件并从 Java Servlet 应用程序**提供该文件的简单示例。

我们使用的文件将来自 webapp 资源。

## 2.Maven 依赖性

如果使用 Jakarta EE，那么我们就不需要添加任何依赖项。然而，如果我们使用 Java SE，我们将需要 javax.servlet-api 依赖关系:

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency> 
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20221127162926/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax.servlet%22%20AND%20a%3A%22javax.servlet-api%22)找到。

## 3.小型应用程序

让我们先看一下代码，然后看看发生了什么:

```java
@WebServlet("/download")
public class DownloadServlet extends HttpServlet {
    private final int ARBITARY_SIZE = 1048;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
      throws ServletException, IOException {

        resp.setContentType("text/plain");
        resp.setHeader("Content-disposition", "attachment; filename=sample.txt");

        try(InputStream in = req.getServletContext().getResourceAsStream("/WEB-INF/sample.txt");
          OutputStream out = resp.getOutputStream()) {

            byte[] buffer = new byte[ARBITARY_SIZE];

            int numBytesRead;
            while ((numBytesRead = in.read(buffer)) > 0) {
                out.write(buffer, 0, numBytesRead);
            }
        }
    }
}
```

### 3.1.请求端点

`@WebServlet(“/download”)`注释标记了`DownloadServlet`类来服务指向`“/download” `端点的请求。

或者，我们可以通过在 web.xml 文件中描述映射来实现这一点。

### 3.2.响应`Content-Type`

`HttpServletResponse` 对象有一个名为`setContentType`的方法，我们可以用它来设置 HTTP 响应的`Content-Type`头。

`Content-Type`是表头属性的历史名称。另一个名字是 MIME 类型(多用途互联网邮件扩展)。我们现在简单地将该值称为媒体类型。

该值可以是“应用程序/pdf”、“文本/普通文本”、“文本/html”、“图像/jpg”等。，官方名单由互联网号码分配机构(IANA)维护，可在此找到[。](https://web.archive.org/web/20221127162926/https://www.iana.org/assignments/media-types/media-types.xhtml#application)

对于我们的例子，我们使用一个简单的文本文件。**文本文件的`Content-Type`是“文本/普通”。**

### 3.3。`Content-Disposition`回应

在响应对象中设置`Content-Disposition `头告诉浏览器如何处理它正在访问的文件。

浏览器将`Content-Disposition`的使用理解为一种约定，但它实际上并不是 HTTP 标准的一部分。W3 有一个关于使用`Content-Disposition`的备忘录，可以在这里阅读。

**响应正文的`Content-Disposition`值可以是“inline”(用于显示网页内容)或“attachment”(用于下载文件)。**

如果未指定，默认`Content-Disposition`为“内联”。

使用可选的头参数，我们可以指定文件名“sample.txt”。

一些浏览器会使用给定的文件名立即下载文件，而另一些会显示包含我们预定义值的下载对话框。

所采取的具体操作将取决于浏览器。

### 3.4.从文件读取并写入输出流

在剩余的代码行中，我们从请求中获取`ServletContext`，并使用它来获取位于“/WEB-INF/sample.txt”的文件。

**使用`HttpServletResponse` # `getOutputStream()`，我们然后从资源的输入流中读取并写入响应的`OutputStream`。**

我们使用的字节数组的大小是任意的。我们可以根据将数据从`InputStream`传递到`OutputStream`所合理分配的内存量来决定大小；数字越小，循环越多；数字越大，内存使用率越高。

这个循环一直持续到`numByteRead`为 0，因为这表示文件结束。

### 3.5.关闭并冲洗

实例在使用后必须关闭，以释放它当前持有的任何资源。`Writer`还必须刷新实例，以便将任何剩余的缓冲字节写入其目的地。

使用一个`try-with-resources`语句，应用程序将自动`close`任何被定义为`try`语句一部分的`AutoCloseable`实例。点击阅读更多关于“尝试资源”[的信息。](/web/20221127162926/http://www.baeldung.com/java-try-with-resources)

我们使用这两种方法来释放内存，确保我们准备的数据从我们的应用程序中发送出去。

### 3.6.下载文件

一切就绪后，我们现在准备运行我们的 Servlet。

现在，当我们访问相对端点`“/download”`时，我们的浏览器将尝试下载文件“simple.txt”。

## 4.结论

从 Servlet 下载文件变得非常简单。使用流允许我们以字节的形式传递数据，媒体类型通知客户端浏览器预期的数据类型。

**由浏览器决定如何处理响应，但是，我们可以用`Content-Disposition`头给出一些指导。**

本文中的所有代码都可以在 GitHub 的[中找到。](https://web.archive.org/web/20221127162926/https://github.com/eugenp/tutorials/tree/master/web-modules/javax-servlets)