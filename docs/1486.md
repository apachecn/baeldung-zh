# 用 Java 发送电子邮件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-email>

## 1。概述

在这个快速教程中，我们将看看使用核心 Java 邮件库发送带附件和不带附件的电子邮件。

## 2。项目设置和依赖关系

对于本文，我们将使用一个简单的基于 Maven 的项目，它依赖于 Java 邮件库:

```
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.5.0-b01</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220630130601/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax.mail%22%20AND%20a%3A%22mail%22)

## 3。发送纯文本和 HTML 电子邮件

首先，我们需要用我们的电子邮件服务提供商的凭证来配置库。然后，我们将创建一个`Session `,用于构造我们要发送的消息。

配置是通过 Java `Properties `对象进行的:

```
Properties prop = new Properties();
prop.put("mail.smtp.auth", true);
prop.put("mail.smtp.starttls.enable", "true");
prop.put("mail.smtp.host", "smtp.mailtrap.io");
prop.put("mail.smtp.port", "25");
prop.put("mail.smtp.ssl.trust", "smtp.mailtrap.io");
```

在上面的属性配置中，我们将电子邮件主机配置为 Mailtrap，并使用服务提供的端口。

现在，让我们进一步用我们的用户名和密码创建一个会话:

```
Session session = Session.getInstance(prop, new Authenticator() {
    @Override
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication(username, password);
    }
});
```

用户名和密码由邮件服务提供商连同主机和端口参数一起给出。

现在我们有了一个邮件`Session `对象，让我们创建一个`Mime` `Message `用于发送:

```
Message message = new MimeMessage(session);
message.setFrom(new InternetAddress("[[email protected]](/web/20220630130601/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
message.setRecipients(
  Message.RecipientType.TO, InternetAddress.parse("[[email protected]](/web/20220630130601/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
message.setSubject("Mail Subject");

String msg = "This is my first email using JavaMailer";

MimeBodyPart mimeBodyPart = new MimeBodyPart();
mimeBodyPart.setContent(msg, "text/html; charset=utf-8");

Multipart multipart = new MimeMultipart();
multipart.addBodyPart(mimeBodyPart);

message.setContent(multipart);

Transport.send(message);
```

在上面的代码片段中，我们首先创建了一个具有必要属性的`message `实例——to、from 和 subject。后跟一个编码为`text/html,` 的`mimeBodyPart`，因为我们的消息是 HTML 格式的。

我们做的下一件事是创建一个`MimeMultipart `对象的实例，我们可以用它来包装我们创建的`mimeBodyPart`。

最后，我们将`multipart `对象设置为`message`的内容，并使用`Transport `对象的`send()`来发送邮件。

**因此，我们可以说`mimeBodyPart`包含在`multipart `中，而`multipart `包含在`message. `中。因此，一个`multipart `可以包含不止一个`mimeBodyPart`** 。

这将是下一节的重点。

## 4。发送带附件的电子邮件

接下来，要发送附件，我们只需要创建另一个`MimeBodyPart`并将文件附加到它上面:

```
MimeBodyPart attachmentBodyPart = new MimeBodyPart();
attachmentBodyPart.attachFile(new File("path/to/file"));
```

然后，我们可以将新的身体部位添加到我们之前创建的`MimeMultipart`对象中:

```
multipart.addBodyPart(attachmentBodyPart);
```

这就是我们需要做的。

**我们再次将`multipart `实例设置为`message `对象的内容，最后我们将使用`send() `来发送邮件**。

## 5。格式化电子邮件文本

为了格式化和样式化我们的电子邮件文本，我们可以使用 HTML 和 CSS 标签。例如，如果我们希望我们的文本是粗体的，我们将实现`<b>` 标签`.`来给文本着色，我们可以使用`style` 标签`.` **如果我们希望有额外的属性，比如粗体，我们也可以将 HTML 标签与 CSS 标签结合起来。**

例如，我们可以创建一个包含红色粗体文本的`String`:

```
String msgStyled = "This is my <b style='color:red;'>bold-red email</b> using JavaMailer";
```

这个`String` 将保存我们要在邮件正文中发送的样式文本。

## 6。结论

总之，我们已经看到了如何使用原生 Java 邮件库发送带有附件的电子邮件。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220630130601/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)