# 用 Java 下载电子邮件附件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-download-email-attachments>

## 1.概观

在本教程中，我们来看看如何使用 Java 下载电子邮件附件。为此，我们需要[JavaMail API](/web/20221207224415/https://www.baeldung.com/java-email)T3。JavaMail API 既可以作为 Maven 依赖项，也可以作为单独的 jar。

## 2.JavaMail API 概述

JavaMail API 用于编写、发送和接收来自 Gmail 等电子邮件服务器的电子邮件。它使用抽象类和接口为电子邮件系统提供了一个框架。该 API 支持大多数 RFC822 和 MIME 互联网消息协议，如 SMTP、POP、IMAP、MIME 和 NNTP。

## 3.JavaMail API 设置

我们需要在 Java 项目中添加 [javax.mail](https://web.archive.org/web/20221207224415/https://search.maven.org/search?q=g:com.sun.mail%20a:javax.mail) Maven 依赖项，以使用 JavaMail API:

```java
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId> 
    <version>1.6.2</version>
</dependency>
```

## 4.下载电子邮件附件

为了在 Java 中处理电子邮件，我们使用了`javax.mail`包中的`Message`类。`Message`实现了`javax.mail.Part`接口。

**`Part`接口有`BodyPart`和属性。带附件的内容是一个叫做`MultiPart`的`BodyPart`。**如果一封[电子邮件有任何附件](/web/20221207224415/https://www.baeldung.com/java-send-emails-attachments)，那么它的处置等同于`Part.ATTACHMENT`。如果没有附件，则处置为`null`。来自`Part`接口的`getDisposition`方法让我们得到处置。

我们通过一个简单的基于 Maven 的项目来理解下载电子邮件附件的工作原理。我们将集中精力下载电子邮件并将附件保存到磁盘上。

我们的项目有一个实用程序，处理下载电子邮件并保存到我们的磁盘。我们还显示了附件列表。

为了下载附件，我们首先检查内容类型是否包含多部分内容。如果是，我们可以进一步处理它，以检查零件是否有任何附件。为了检查内容类型，我们编写:

```java
if (contentType.contains("multipart")) {
    //send to the download utility...
}
```

如果我们有一个多部分，我们首先检查它是否属于类型`Part.ATTACHMENT`，如果是，我们使用`saveFile` 方法将文件保存到我们的目标文件夹。因此，在下载实用程序中，我们将检查:

```java
if (Part.ATTACHMENT.equalsIgnoreCase(part.getDisposition())) {
    String file = part.getFileName();
    part.saveFile(downloadDirectory + File.separator + part.getFileName());
    downloadedAttachments.add(file);
}
```

**因为我们使用的 JavaMail API 版本高于 1.4，所以我们可以从`Part`接口使用`saveFile`方法。****`saveFile`方法可以处理`File`对象或者`String`。我们在示例中使用了一个字符串。**这一步将附件保存到我们指定的文件夹中。我们还为展示维护了一个附件列表。

在 JavaMail API 版本之前，我们必须使用`FileStream`和`InputStream`一个字节一个字节地编写整个文件。在我们的例子中，我们使用 Pop3 服务器作为 Gmail 帐户。因此，要调用示例中的方法，我们需要一个有效的 Gmail 用户名和密码以及一个下载附件的文件夹。

让我们看看下载附件并将它们保存到磁盘的示例代码:

```java
public List<String> downloadAttachments(Message message) throws IOException, MessagingException {
    List<String> downloadedAttachments = new ArrayList<String>();
    Multipart multiPart = (Multipart) message.getContent();
    int numberOfParts = multiPart.getCount();
    for (int partCount = 0; partCount < numberOfParts; partCount++) {
        MimeBodyPart part = (MimeBodyPart) multiPart.getBodyPart(partCount);
        if (Part.ATTACHMENT.equalsIgnoreCase(part.getDisposition())) {
            String file = part.getFileName();
            part.saveFile(downloadDirectory + File.separator + part.getFileName());
            downloadedAttachments.add(file);
        }
    }
    return downloadedAttachments;
} 
```

## 5.结论

本文展示了如何使用本地 JavaMail 库下载电子邮件附件，从而用 Java 下载电子邮件。本教程的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221207224415/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)