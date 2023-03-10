# Groovy 中的模板引擎

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-template-engines>

## 1.概观

在这篇介绍性教程中，我们将探索 [Groovy](/web/20220630131159/https://www.baeldung.com/groovy-language) 中模板引擎的概念。

在 Groovy 中，我们可以使用`[GString](/web/20220630131159/https://www.baeldung.com/groovy-strings)` [s](/web/20220630131159/https://www.baeldung.com/groovy-strings) 轻松生成动态文本。然而，模板引擎提供了使用静态模板处理动态文本的更好方法。

这些模板便于为各种通知(如短信和电子邮件)定义静态模板。

## 2.Groovy 的`TemplateEngine`是什么？

**Groovy 的`TemplateEngine`是一个包含`createTemplate`方法的抽象类。** 

Groovy 中所有可用的模板框架引擎都扩展了`TemplateEngine`并实现了`createTemplate.`此外，每个引擎都返回了 `Template`接口对象。

**`Template`接口有一个方法`make`，它采用一个映射来绑定变量。**因此，它必须由每个模板框架来实现。

让我们讨论 Groovy 中所有可用模板框架的功能和行为。

## 3.`SimpleTemplateEngine`

`SimpleTemplateEngine`使用`String`插值和 scriptlets 生成动态文本。这个引擎对于简单的通知非常有用，比如 SMS 和简单的文本邮件。

例如:

```java
def smsTemplate = 'Dear <% print user %>, Thanks for reading our Article. ${signature}'
def bindMap = [user: "Norman", signature: "Baeldung"]
def smsText = new SimpleTemplateEngine().createTemplate(smsTemplate).make(bindMap)

assert smsText.toString() == "Dear Norman, Thanks for reading our Article. Baeldung"
```

## 4.`StreamingTemplateEngine`

一般来说，`StreamingTemplateEngine`的工作方式与`SimpleTemplateEngine.`相似，但是在内部，它使用 [`Writable`](https://web.archive.org/web/20220630131159/http://docs.groovy-lang.org/latest/html/api/groovy/lang/Writable.html) 闭包来生成模板。

出于同样的原因，它在处理较大的字符串(> 64K)时也有好处。因此，它比`SimpleTemplateEngine.`更有效

让我们写一个简单的例子，使用静态模板生成动态电子邮件内容。

首先，我们将创建一个静态的`articleEmail`模板:

```java
Dear <% out << (user) %>,
Please read the requested article below.
<% out << (articleText) %>
From,
<% out << (signature) %>
```

在这里，我们将`<% %>` script let 用于动态文本，将`out`用于编写器。

现在，我们将使用`StreamingTemplateEngine`生成电子邮件的内容:

```java
def articleEmailTemplate = new File('src/main/resources/articleEmail.template')
def bindMap = [user: "Norman", signature: "Baeldung"]

bindMap.articleText = """1\. Overview
This is a tutorial article on Template Engines...""" //can be a string larger than 64k

def articleEmailText = new StreamingTemplateEngine().createTemplate(articleEmailTemplate).make(bindMap)

assert articleEmailText.toString() == """Dear Norman,
Please read the requested article below.
1\. Overview
This is a tutorial article on Template Engines...
From,
Baeldung"""
```

## 5.`GStringTemplateEngine`

顾名思义，`GStringTemplateEngine` 使用`GString`从静态模板生成动态文本。

首先，让我们使用`GString`编写一个简单的`email`模板:

```java
Dear $user,
Thanks for subscribing our services.
${signature}
```

现在，我们将使用`GStringTemplateEngine`来创建动态内容:

```java
def emailTemplate = new File('src/main/resources/email.template')
def emailText = new GStringTemplateEngine().createTemplate(emailTemplate).make(bindMap) 
```

## 6.`XmlTemplateEngine`

当我们想要创建动态 XML 输出时,`XmlTemplateEngine`非常有用。它需要 XML schema 作为输入，并允许两个特殊的标签，`<gsp:scriptlet>`注入脚本，`<gsp:expression>` 注入表达式。

例如，让我们将已经讨论过的`email`模板转换成 XML:

```java
def emailXmlTemplate = '''
<xs xmlns:gsp='groovy-server-pages'>
    <gsp:scriptlet>def emailContent = "Thanks for subscribing our services."</gsp:scriptlet>
    <email>
        <greet>Dear ${user}</greet>
        <content><gsp:expression>emailContent</gsp:expression></content>
        <signature>${signature}</signature>
    </email>
</xs>'''

def emailXml = new XmlTemplateEngine().createTemplate(emailXmlTemplate).make(bindMap)
```

因此，`emailXml`将呈现 XML，内容将是:

```java
<xs>
  <email>
    <greet>
      Dear Norman
    </greet>
    <content>
      Thanks for subscribing our services.
    </content>
    <signature>
      Baeldung
    </signature>
  </email>
</xs> 
```

有趣的是，模板框架对 XML 输出进行了缩进和美化。

## 7.`MarkupTemplateEngine`

这个模板框架是生成 HTML 和其他标记语言的完整包。

此外，它使用特定于领域的语言来处理模板，并且是 Groovy 中所有可用模板框架中优化程度最高的。

### 7.1.超文本标记语言

让我们写一个简单的例子来为已经讨论过的`email`模板呈现 HTML:

```java
def emailHtmlTemplate = """
html {
    head {
        title('Service Subscription Email')
    }
    body {
        p('Dear Norman')
        p('Thanks for subscribing our services.')
        p('Baeldung')
    }
}"""
def emailHtml = new MarkupTemplateEngine().createTemplate(emailHtmlTemplate).make()
```

因此，`emailHtml`的内容将是:

```java
<html><head><title>Service Subscription Email</title></head>
<body><p>Dear Norman</p><p>Thanks for subscribing our services.</p><p>Baeldung</p></body></html>
```

### 7.2.可扩展标记语言

同样，我们可以呈现 XML:

```java
def emailXmlTemplate = """
xmlDeclaration()  
    xs{
        email {
            greet('Dear Norman')
            content('Thanks for subscribing our services.')
            signature('Baeldung')
        }  
    }"""
def emailXml = new MarkupTemplateEngine().createTemplate(emailXmlTemplate).make()
```

因此，`emailXml`的内容将是:

```java
<?xml version='1.0'?>
<xs><email><greet>Dear Norman</greet><content>Thanks for subscribing our services.</content>
<signature>Baeldung</signature></email></xs>
```

### 7.3.`TemplateConfiguration`

注意，和 like `XmlTemplateEngine`不同，这个框架的模板输出不是自己缩进美化的。

对于这样的配置，我们将使用`TemplateConfiguration` 类:

```java
TemplateConfiguration config = new TemplateConfiguration()
config.autoIndent = true
config.autoEscape = true
config.autoNewLine = true

def templateEngine = new MarkupTemplateEngine(config)
```

### 7.4.国际化

另外，`TemplateConfiguration` 的`locale`属性可用于启用对国际化的支持。

首先，我们将创建一个静态模板文件`email.tpl`，并将已经讨论过的`emailHtmlTemplate`字符串复制到其中。这将被视为默认模板。

同样，我们将创建基于地区的模板文件，如日语的`email_ja_JP.tpl`，法语的 email_fr_FR.tpl 等。

最后，我们需要做的就是在`TemplateConfiguration`对象中设置`locale`:

```java
config.locale = Locale.JAPAN
```

因此，将选择相应的基于地区的模板。

## 8.结论

在本文中，我们看到了 Groovy 中可用的各种模板框架。

我们可以利用这些方便的模板引擎，使用静态模板生成动态文本。因此，它们有助于动态生成各种通知或屏幕消息和错误。

像往常一样，本教程的代码实现可以在 [GitHub](https://web.archive.org/web/20220630131159/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2) 项目中获得。