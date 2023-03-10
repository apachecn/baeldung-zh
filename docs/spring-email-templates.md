# 在 Spring 中使用百里香叶和 FreeMarker 电子邮件模板

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-email-templates>

## 1.概观

在我们的[上一篇文章](/web/20220728105348/https://www.baeldung.com/spring-email)中，我们看到了如何使用 Spring 编写和发送文本邮件。

但是也有可能**使用 [Spring 模板引擎](/web/20220728105348/https://www.baeldung.com/spring-template-engines)来编写带有动态内容的漂亮的 HTML 电子邮件**。

在本教程中，我们将学习如何使用其中最著名的: **[百里香](https://web.archive.org/web/20220728105348/https://www.thymeleaf.org/)和 [FreeMarker](https://web.archive.org/web/20220728105348/https://freemarker.apache.org/)** 。

## 2.Spring HTML 邮件

先从[春季邮件教程](/web/20220728105348/https://www.baeldung.com/spring-email)说起。

首先，我们将向`EmailServiceImpl`类添加一个方法来发送带有 HTML 正文的电子邮件:

```java
private void sendHtmlMessage(String to, String subject, String htmlBody) throws MessagingException {
    MimeMessage message = emailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(htmlBody, true);
    emailSender.send(message);
}
```

我们使用 **`MimeMessageHelper` 来填充消息**。重要的部分是传递给`setText`方法的`true`值:它指定了 HTML 内容类型。

现在让我们看看如何使用百里香和 FreeMarker 模板来构建这个`htmlBody`。

## 3.百里香叶构型

先说配置。我们可以在一个名为`EmailConfiguration`的类中隔离这一点。

首先，我们应该**提供一个模板解析器来定位模板文件目录**。

### 3.1.模板作为类路径资源

**模板文件可以放在 JAR 文件**中，这是维护模板和它们的输入数据之间的内聚性的最简单的方法。

为了从 JAR 中定位模板，我们使用了`ClassLoaderTemplateResolver`。我们的模板在`main/resources/mail-templates`目录中，所以我们设置了相对于`resource`目录的`Prefix`属性:

```java
@Bean
public ITemplateResolver thymeleafTemplateResolver() {
    ClassLoaderTemplateResolver templateResolver = new ClassLoaderTemplateResolver();
    templateResolver.setPrefix("mail-templates/");
    templateResolver.setSuffix(".html");
    templateResolver.setTemplateMode("HTML");
    templateResolver.setCharacterEncoding("UTF-8");
    return templateResolver;
}
```

### 3.2.来自外部目录的模板

在其他情况下，**我们可能想要修改模板，而不必重新构建和部署**。为此，我们可以将模板放在文件系统上。

在`application.properties`中配置这个路径可能是有用的，这样我们可以为每个部署修改它。可以使用`@Value`注释来访问该属性:

```java
@Value("${spring.mail.templates.path}")
private String mailTemplatesPath;
```

然后我们将这个值传递给一个`FileTemplateResolver`，代替我们的`thymeleafTemplateResolver`方法中的`ClassLoaderTemplateResolver` :

```java
FileTemplateResolver templateResolver = new FileTemplateResolver();
templateResolver.setPrefix(mailTemplatesPath);
```

### 3.3.配置百里香引擎

最后一步是为百里香引擎创建工厂方法。我们需要告诉引擎我们选择了哪个`TemplateResolver`,我们可以通过 bean factory 方法的一个参数注入它:

```java
@Bean
public SpringTemplateEngine thymeleafTemplateEngine(ITemplateResolver templateResolver) {
    SpringTemplateEngine templateEngine = new SpringTemplateEngine();
    templateEngine.setTemplateResolver(templateResolver);
    templateEngine.setTemplateEngineMessageSource(emailMessageSource());
    return templateEngine;
}
```

这里，我们之前创建的解析器被 Spring 自动注入到模板引擎工厂方法中。

## 4.自由标记配置

与百里香相同，在`EmailConfiguration`类中，我们将为 FreeMarker 模板`(.ftl`:
配置**模板解析器，这次，**模板的位置将在`FreeMarkerConfigurer` bean** 中配置。**

### 4.1.类路径中的模板

在这里，我们有与百里香相同的选择。让我们将模板配置为类路径资源:

```java
@Bean 
public FreeMarkerConfigurer freemarkerClassLoaderConfig() {
    Configuration configuration = new Configuration(Configuration.VERSION_2_3_27);
    TemplateLoader templateLoader = new ClassTemplateLoader(this.getClass(), "/mail-templates");
    configuration.setTemplateLoader(templateLoader);
    FreeMarkerConfigurer freeMarkerConfigurer = new FreeMarkerConfigurer();
    freeMarkerConfigurer.setConfiguration(configuration);
    return freeMarkerConfigurer; 
}
```

### 4.2.文件系统上的模板

要从文件系统中的另一个路径配置模板，我们需要替换`TemplateLoader`实例:

```java
TemplateLoader templateLoader = new FileTemplateLoader(new File(mailTemplatesPath));
```

## 5.用百里香叶和游离标记定位

为了用百里香叶管理翻译，我们可以**为引擎**指定一个`MessageSource`实例:

```java
@Bean
public ResourceBundleMessageSource emailMessageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasename("mailMessages");
    return messageSource;
}
```

```java
@Bean
public SpringTemplateEngine thymeleafTemplateEngine() {
   ...
   templateEngine.setTemplateEngineMessageSource(emailMessageSource());
   ...
}
```

然后，我们将为我们支持的每个语言环境创建资源包:

```java
src/main/resources/mailMessages_xx_YY.properties
```

由于 **FreeMarker 通过[复制模板](https://web.archive.org/web/20220728105348/https://freemarker.apache.org/docs/ref_directive_include.html#ref_directive_include_localized)** 提出本地化，我们不必在那里配置消息源。

## 6.百里香叶模板含量

接下来，我们来看看`template-thymeleaf.html`文件:

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
    <p th:text="#{greetings(${recipientName})}"></p>
    <p th:text="${text}"></p>
    <p th:text="#{regards}"></p>
    <p>
      <em th:text="#{signature(${senderName})}"></em> <br />
    </p>
  </body>
</html>
```

可以看到，我们使用了百里香符号，也就是说， **`${…}`表示变量，`#{…}`表示本地化字符串**。

由于模板引擎配置正确，使用起来非常简单:我们只需创建一个包含模板变量(在这里作为`Map`传递)的 **`Context`对象。**

然后，我们将把它和模板名一起传递给`process`方法:

```java
@Autowired
private SpringTemplateEngine thymeleafTemplateEngine;

@Override
public void sendMessageUsingThymeleafTemplate(
    String to, String subject, Map<String, Object> templateModel)
        throws MessagingException {

    Context thymeleafContext = new Context();
    thymeleafContext.setVariables(templateModel);
    String htmlBody = thymeleafTemplateEngine.process("template-thymeleaf.html", thymeleafContext);

    sendHtmlMessage(to, subject, htmlBody);
}
```

现在，让我们看看如何用 FreeMarker 做同样的事情。

## 7.FreeMarker 模板内容

可以看出，FreeMarker 的语法更简单，但是它也不管理本地化的字符串。所以，这是英文版:

```java
<!DOCTYPE html>
<html>
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
      <p>Hi ${recipientName}</p>
      <p>${text}</p>
      <p>Regards,</p>
      <p>
        <em>${senderName} at Baeldung</em> <br />
      </p>
    </body>
</html>
```

然后，我们应该使用 **`FreeMarkerConfigurer`类来获取模板文件，最后，`FreeMarkerTemplateUtils`从我们的`Map`** 中注入数据:

```java
@Autowired
private FreeMarkerConfigurer freemarkerConfigurer;

@Override
public void sendMessageUsingFreemarkerTemplate(
    String to, String subject, Map<String, Object> templateModel)
        throws IOException, TemplateException, MessagingException {

    Template freemarkerTemplate = freemarkerConfigurer.getConfiguration()
      .getTemplate("template-freemarker.ftl");
    String htmlBody = FreeMarkerTemplateUtils.processTemplateIntoString(freemarkerTemplate, templateModel);

    sendHtmlMessage(to, subject, htmlBody);
}
```

为了更进一步，我们将看到如何添加一个标志到我们的电子邮件签名。

## 8.嵌入图像的电子邮件

因为在 HTML 邮件中包含图片是很常见的，我们将会看到如何使用一个 [CID 附件](https://web.archive.org/web/20220728105348/https://blog.mailtrap.io/embedding-images-in-html-email-have-the-rules-changed/#CID_attachments_or_embedding_an_image_using_MIME_object)来做到这一点。

第一个变化与`sendHtmlMessage`方法有关。**我们必须将`MimeMessageHelper`设置为多部分**，方法是将`true`传递给构造函数的第二个参数:

```java
MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
```

然后，我们必须获取图像文件作为资源。我们可以为此使用`@Value`注释:

```java
@Value("classpath:/mail-logo.png")
Resource resourceFile;
```

请注意，`mail-logo.png`文件位于`src/main/resources`目录中。

回到`sendHtmlMessage`方法，我们将**添加`resourceFile`作为内嵌附件**，以便能够用 CID 引用它:

```java
helper.addInline("attachment.png", resourceFile);
```

最后，必须使用 CID 符号从百里香和 FreeMarker 电子邮件中引用**图像:**

```java
<img src="cid:attachment.png" />
```

## 9.结论

在本文中，我们看到了如何发送百里香和 FreeMarker 电子邮件，包括丰富的 HTML 内容。

总结一下，大部分作品都和春天有关；因此，**对于发送电子邮件**这样的简单需求，这两者的使用非常相似。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-2)