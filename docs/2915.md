# 春季电子邮件指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-email>

## 1。概述

在本教程中，我们将介绍从普通的 Spring 应用程序和 Spring Boot 应用程序发送电子邮件的步骤。对于前者，我们将使用 [JavaMail](https://web.archive.org/web/20221005043342/https://java.net/projects/javamail/pages/Home) 库，而后者将使用`spring-boot-starter-mail`依赖项。

## 延伸阅读:

## [注册–通过电子邮件激活新账户](/web/20221005043342/https://www.baeldung.com/registration-verify-user-by-email)

Verify newly registered users by sending them a verification token via email before allowing them to log in - using Spring Security.[Read more](/web/20221005043342/https://www.baeldung.com/registration-verify-user-by-email) →

## [Spring Boot 执行器](/web/20221005043342/https://www.baeldung.com/spring-boot-actuators)

A quick intro to Spring Boot Actuators - using and extending the existing ones, configuration and rolling your own.[Read more](/web/20221005043342/https://www.baeldung.com/spring-boot-actuators) →

## 2。Maven 依赖关系

首先，我们需要将依赖项添加到我们的`pom.xml`中。

### 2.1。弹簧

下面是我们将添加到普通 Spring 框架中使用的内容:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20221005043342/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-context-support)

### 2.2。Spring Boot

对 Spring Boot 来说:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>2.5.6</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221005043342/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-mail%22) 资源库中获得。

## 3。邮件服务器属性

Spring 框架中支持 Java 邮件的接口和类组织如下:

1.  **`MailSender`界面**:顶层界面，提供发送简单邮件的基本功能
2.  **`JavaMailSender`接口**:上述`MailSender`的子接口。它支持 MIME 消息，通常与`MimeMessageHelper`类一起用于创建`MimeMessage`。建议在此接口上使用`MimeMessagePreparator`机制。
3.  **`JavaMailSenderImpl`类**提供了`JavaMailSender`接口的实现。它支持`MimeMessage`和`SimpleMailMessage`。
4.  **`SimpleMailMessage`类**:用于创建一个简单的邮件消息，包括发件人、收件人、抄送、主题和文本字段
5.  **`MimeMessagePreparator`接口**提供一个回调接口，用于准备 MIME 消息。
6.  **`MimeMessageHelper`类**:创建 MIME 消息的助手类。它支持 HTML 布局中的图像、典型的邮件附件和文本内容。

在接下来的部分中，我们将展示如何使用这些接口和类。

### 3.1。Spring 邮件服务器属性

例如，指定 SMTP 服务器所需的邮件属性可以使用`JavaMailSenderImpl`来定义。

对于 Gmail，可以按如下所示进行配置:

```
@Bean
public JavaMailSender getJavaMailSender() {
    JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
    mailSender.setHost("smtp.gmail.com");
    mailSender.setPort(587);

    mailSender.setUsername("[[email protected]](/web/20221005043342/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    mailSender.setPassword("password");

    Properties props = mailSender.getJavaMailProperties();
    props.put("mail.transport.protocol", "smtp");
    props.put("mail.smtp.auth", "true");
    props.put("mail.smtp.starttls.enable", "true");
    props.put("mail.debug", "true");

    return mailSender;
} 
```

### 3.2。Spring Boot 邮件服务器属性

一旦依赖关系就绪，下一步就是使用`spring.mail.*`名称空间在`application.properties`文件中指定邮件服务器属性。

我们可以这样指定 Gmail SMTP 服务器的属性:

```
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=<login user to smtp server>
spring.mail.password=<login password to smtp server>
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true 
```

一些 SMTP 服务器需要 TLS 连接，所以我们使用属性`spring.mail.properties.mail.smtp.starttls.enable`来启用 TLS 保护的连接。

#### 3.2.1。Gmail SMTP 属性

我们可以通过 Gmail SMTP 服务器发送电子邮件。查看[文档](https://web.archive.org/web/20221005043342/https://support.google.com/mail/answer/13273?hl=en&rd=2)以了解 Gmail 外发邮件 SMTP 服务器属性。

我们的`application.properties` 文件已经配置为使用 Gmail SMTP(参见上一节)。

请注意，我们帐户的密码不应该是普通密码，而是为我们的 Google 帐户生成的应用程序密码。点击此[链接](https://web.archive.org/web/20221005043342/https://support.google.com/accounts/answer/185833)查看详情并生成您的 Google App 密码。

#### 3.2.2。SES SMTP 属性

为了使用 Amazon SES 发送电子邮件，我们设置了我们的`application.properties`:

```
spring.mail.host=email-smtp.us-west-2.amazonaws.com
spring.mail.username=username
spring.mail.password=password
spring.mail.properties.mail.transport.protocol=smtp
spring.mail.properties.mail.smtp.port=25
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```

请注意，亚马逊要求我们在使用之前验证我们的凭据。点击[链接](https://web.archive.org/web/20221005043342/https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html)验证您的用户名和密码。

## 4。发送电子邮件

一旦依赖关系管理和配置就绪，我们就可以使用前面提到的`JavaMailSender`来发送电子邮件。

因为普通的 Spring 框架和它的引导版本都以相似的方式处理电子邮件的撰写和发送，所以在下面的小节中我们不必区分这两者。

### 4.1。发送简单的电子邮件

让我们首先撰写并发送一封不带任何附件的简单电子邮件:

```
@Component
public class EmailServiceImpl implements EmailService {

    @Autowired
    private JavaMailSender emailSender;

    public void sendSimpleMessage(
      String to, String subject, String text) {
        ...
        SimpleMailMessage message = new SimpleMailMessage(); 
        message.setFrom("[[email protected]](/web/20221005043342/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        message.setTo(to); 
        message.setSubject(subject); 
        message.setText(text);
        emailSender.send(message);
        ...
    }
}
```

注意，即使不强制提供`from`地址，许多 SMTP 服务器也会拒绝这样的消息。这就是为什么我们在我们的`EmailService`实现中使用[【电子邮件保护】](/web/20221005043342/https://www.baeldung.com/cdn-cgi/l/email-protection)电子邮件地址。

### 4.2。发送带附件的电子邮件

有时 Spring 的简单消息传递对于我们的用例来说是不够的。

例如，我们想发送一封附有发票的订单确认电子邮件。在这种情况下，我们应该使用来自`JavaMail` 库的`MIME` 多部分消息，而不是`SimpleMailMessage`。Spring 支持与`org.springframework.mail.javamail.MimeMessageHelper`类的`JavaMail`消息传递。

首先，我们将向`EmailServiceImpl` 添加一个方法来发送带有附件的电子邮件:

```
@Override
public void sendMessageWithAttachment(
  String to, String subject, String text, String pathToAttachment) {
    // ...

    MimeMessage message = emailSender.createMimeMessage();

    MimeMessageHelper helper = new MimeMessageHelper(message, true);

    helper.setFrom("[[email protected]](/web/20221005043342/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(text);

    FileSystemResource file 
      = new FileSystemResource(new File(pathToAttachment));
    helper.addAttachment("Invoice", file);

    emailSender.send(message);
    // ...
}
```

### 4.3。简单的电子邮件模板

`SimpleMailMessage`类支持文本格式化。

我们可以通过在配置中定义模板 bean 来创建电子邮件模板:

```
@Bean
public SimpleMailMessage templateSimpleMessage() {
    SimpleMailMessage message = new SimpleMailMessage();
    message.setText(
      "This is the test email template for your email:\n%s\n");
    return message;
}
```

现在我们可以使用这个 bean 作为电子邮件的模板，并且只需要向模板提供必要的参数:

```
@Autowired
public SimpleMailMessage template;
...
String text = String.format(template.getText(), templateArgs);  
sendSimpleMessage(to, subject, text);
```

## 5。处理发送错误

`JavaMail`提供`SendFailedException`来处理消息无法发送的情况。但是，当我们向错误的地址发送电子邮件时，我们可能不会得到这个异常。原因如下:

RFC 821 中的 SMTP 协议规范指定了 SMTP 服务器在试图向不正确的地址发送电子邮件时应该返回的 550 返回代码。但是大部分的公共 SMTP 服务器都不这么做。相反，他们会发送一封“投递失败”的电子邮件，或者根本不给任何反馈。

例如，Gmail SMTP 服务器发送一封“发送失败”的邮件。在我们的程序中没有例外。

所以，我们有几个选择来处理这个案子:

1.  接住`SendFailedException`，永远扔不掉。
2.  在一段时间内，检查我们的发件人邮箱中是否有“传递失败”的消息。这个不直截了当，时间段也不确定。
3.  如果我们的邮件服务器没有任何反馈，我们就什么也做不了。

## 6。结论

在这篇简短的文章中，我们展示了如何在 Spring Boot 应用程序中设置和发送电子邮件。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221005043342/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-2 "Spring MVC Email")