# 用 Java 发送带附件的电子邮件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-send-emails-attachments>

## 1.概观

在这个快速教程中，我们将学习如何使用`JavaMail` API 在 Java 中发送带有单个和多个附件的电子邮件。

## 2.项目设置

在本文中，我们将创建一个具有`[javax.mail](https://web.archive.org/web/20221208074836/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax.mail%22%20AND%20a%3A%22mail%22)`依赖关系的简单 Maven 项目:

```java
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.5.0-b01</version>
</dependency>
```

## 3.发送带附件的邮件

首先，我们[需要配置电子邮件服务提供商](/web/20221208074836/https://www.baeldung.com/java-email#sending-a-plain-text-and-an-html-email)的凭证。然后，通过提供电子邮件主机、端口、用户名和密码来创建`Session` 对象。所有这些细节都是由电子邮件主机服务提供的。我们可以为我们的代码使用任何假的 SMTP 测试服务器。

`**Session**` **对象将作为连接工厂来处理** `**JavaMail**` **的配置和认证。**

现在我们有了一个`Session`对象，让我们进一步创建`MimeMessage`和`MimeBodyPart`对象。我们使用这些对象来创建电子邮件:

```java
Message message = new MimeMessage(session); 
message.setFrom(new InternetAddress(from)); 
message.setRecipients(Message.RecipientType.TO, InternetAddress.parse(to)); 
message.setSubject("Test Mail Subject"); 

BodyPart messageBodyPart = new MimeBodyPart(); 
messageBodyPart.setText("Mail Body"); 
```

在上面的代码片段中，我们已经创建了具有所需细节的`MimeMessage`对象，例如 from、to 和 subject。然后，我们得到了一个带有电子邮件正文的`MimeBodyPart`对象。

现在，我们需要创建另一个`MimeBodyPart`来在邮件中添加附件:

```java
MimeBodyPart attachmentPart = new MimeBodyPart();
attachmentPart.attachFile(new File("path/to/file")); 
```

我们现在有两个`MimeBodyPart`对象用于一个邮件会话。所以我们需要创建一个`MimeMultipart`对象，然后将两个`MimeBodyPart`对象添加到其中:

```java
Multipart multipart = new MimeMultipart();
multipart.addBodyPart(messageBodyPart);
multipart.addBodyPart(attachmentPart); 
```

最后，`MimeMultiPart`被添加到`MimeMessage`对象中作为我们的邮件内容，并调用`Transport.send()`方法来发送消息:

```java
message.setContent(multipart);
Transport.send(message); 
```

**概括起来，** `**Message**` **包含** `**MimeMultiPart**` **其中又包含多个** `**MimeBodyPart(s)**`。这就是我们如何组装完整的电子邮件。

**此外，要发送多个附件，您只需添加另一个** `**MimeBodyPart**` **。**

## 4.结论

在本教程中，我们学习了如何用 Java 发送带有单个和多个附件的电子邮件。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208074836/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)