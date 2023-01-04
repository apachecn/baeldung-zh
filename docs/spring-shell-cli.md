# 带有弹簧外壳的 CLI

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-shell-cli>

## 1。概述

简单来说，Spring Shell [项目](https://web.archive.org/web/20220625231140/https://spring.io/projects/spring-shell)提供了一个交互式 Shell，用于处理命令和使用 Spring 编程模型构建全功能 CLI。

在本文中，我们将探索它的特性、关键类和注释，并实现几个定制命令和定制。

## 2。Maven 依赖关系

首先，我们需要将`spring-shell`依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.springframework.shell</groupId>
    <artifactId>spring-shell</artifactId>
    <version>1.2.0.RELEASE</version>
</dependency>
```

这个产品的最新版本可以在[这里](https://web.archive.org/web/20220625231140/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.shell%22%20AND%20a%3A%22spring-shell%22)找到。

## 3。访问外壳

在我们的应用程序中，有两种主要的方法来访问 shell。

第一个是在应用程序的入口点引导 shell，让用户输入命令:

```
public static void main(String[] args) throws IOException {
    Bootstrap.main(args);
}
```

第二个是获得一个`JLineShellComponent`并以编程方式执行命令:

```
Bootstrap bootstrap = new Bootstrap();
JLineShellComponent shell = bootstrap.getJLineShellComponent();
shell.executeCommand("help");
```

我们将使用第一种方法，因为它最适合本文中的例子，但是，在源代码中，您可以找到使用第二种形式的测试用例。

## 4。命令

shell 中已经有几个内置的命令，比如`clear`、`help`、`exit`等。，提供每个 CLI 的标准功能。

自定义命令可以通过在实现`CommandMarker`接口的 Spring 组件中添加标有`@CliCommand`注释的方法来公开。

该方法的每个参数都必须用一个`@CliOption`注释来标记，如果我们没有做到这一点，我们将在尝试执行命令时遇到几个错误。

### 4.1。向外壳添加命令

首先，我们需要让 shell 知道我们的命令在哪里。为此，它要求文件`META-INF/spring/spring-shell-plugin.xml`存在于我们的项目中，在那里，我们可以使用 Spring 的组件扫描功能:

```
<beans ... >
    <context:component-scan base-package="org.baeldung.shell.simple" />
</beans>
```

一旦组件被 Spring 注册和实例化，它们就被注册到 shell 解析器，并且它们的注释被处理。

让我们创建两个简单的命令，一个获取并显示 URL 的内容，另一个将这些内容保存到一个文件中:

```
@Component
public class SimpleCLI implements CommandMarker {

    @CliCommand(value = { "web-get", "wg" })
    public String webGet(
      @CliOption(key = "url") String url) {
        return getContentsOfUrlAsString(url);
    }

    @CliCommand(value = { "web-save", "ws" })
    public String webSave(
      @CliOption(key = "url") String url,
      @CliOption(key = { "out", "file" }) String file) {
        String contents = getContentsOfUrlAsString(url);
        try (PrintWriter out = new PrintWriter(file)) {
            out.write(contents);
        }
        return "Done.";
    }
}
```

注意，我们可以向`@CliCommand`和`@CliOption`的`value`和`key`属性分别传递多个字符串，这允许我们公开几个行为相同的命令和参数。

现在，让我们检查一下是否一切都按预期运行:

```
spring-shell>web-get --url https://www.google.com
<!doctype html ... 
spring-shell>web-save --url https://www.google.com --out contents.txt
Done.
```

### 4.2。命令的可用性

我们可以在返回`boolean`的方法上使用`@CliAvailabilityIndicator`注释，以在运行时改变命令是否应该暴露给 shell。

首先，让我们创建一个方法来修改`web-save`命令的可用性:

```
private boolean adminEnableExecuted = false;

@CliAvailabilityIndicator(value = "web-save")
public boolean isAdminEnabled() {
    return adminEnableExecuted;
}
```

现在，让我们创建一个命令来更改`adminEnableExecuted`变量:

```
@CliCommand(value = "admin-enable")
public String adminEnable() {
    adminEnableExecuted = true;
    return "Admin commands enabled.";
}
```

最后，我们来验证一下:

```
spring-shell>web-save --url https://www.google.com --out contents.txt
Command 'web-save --url https://www.google.com --out contents.txt'
  was found but is not currently available
  (type 'help' then ENTER to learn about this command)
spring-shell>admin-enable
Admin commands enabled.
spring-shell>web-save --url https://www.google.com --out contents.txt
Done.
```

### 4.3。必需的参数

默认情况下，所有命令参数都是可选的。然而，我们可以通过`@CliOption`注释的`mandatory`属性使它们成为必需的:

```
@CliOption(key = { "out", "file" }, mandatory = true)
```

现在，我们可以测试如果不引入它，会导致错误:

```
spring-shell>web-save --url https://www.google.com
You should specify option (--out) for this command
```

### 4.4。默认参数

一个空的`@CliOption`值使该参数成为默认值。在那里，我们将接收 shell 中引入的不属于任何命名参数的值:

```
@CliOption(key = { "", "url" })
```

现在，让我们检查它是否如预期的那样工作:

```
spring-shell>web-get https://www.google.com
<!doctype html ...
```

### 4.5。帮助用户

`@CliCommand`和`@CliOption`注释提供了一个`help`属性，允许我们在使用内置的`help`命令或跳转以获得自动完成时指导用户。

让我们修改我们的`web-get`来添加自定义帮助消息:

```
@CliCommand(
  // ...
  help = "Displays the contents of an URL")
public String webGet(
  @CliOption(
    // ...
    help = "URL whose contents will be displayed."
  ) String url) {
    // ...
}
```

现在，用户可以确切地知道我们的命令做了什么:

```
spring-shell>help web-get
Keyword:                    web-get
Keyword:                    wg
Description:                Displays the contents of a URL.
  Keyword:                  ** default **
  Keyword:                  url
    Help:                   URL whose contents will be displayed.
    Mandatory:              false
    Default if specified:   '__NULL__'
    Default if unspecified: '__NULL__'

* web-get - Displays the contents of a URL.
* wg - Displays the contents of a URL.
```

## 5。定制

有三种方法可以通过实现`BannerProvider`、`PromptProvider`和`HistoryFileNameProvider`接口来定制 shell，它们都已经提供了默认实现。

此外，我们需要使用`@Order`注释来允许我们的提供者优先于那些实现。

让我们创建一个新横幅，开始我们的定制:

```
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class SimpleBannerProvider extends DefaultBannerProvider {

    public String getBanner() {
        StringBuffer buf = new StringBuffer();
        buf.append("=======================================")
            .append(OsUtils.LINE_SEPARATOR);
        buf.append("*          Baeldung Shell             *")
            .append(OsUtils.LINE_SEPARATOR);
        buf.append("=======================================")
            .append(OsUtils.LINE_SEPARATOR);
        buf.append("Version:")
            .append(this.getVersion());
        return buf.toString();
    }

    public String getVersion() {
        return "1.0.1";
    }

    public String getWelcomeMessage() {
        return "Welcome to Baeldung CLI";
    }

    public String getProviderName() {
        return "Baeldung Banner";
    }
}
```

请注意，我们还可以更改版本号和欢迎消息。

现在，让我们更改提示:

```
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class SimplePromptProvider extends DefaultPromptProvider {

    public String getPrompt() {
        return "baeldung-shell";
    }

    public String getProviderName() {
        return "Baeldung Prompt";
    }
}
```

最后，让我们修改历史文件的名称:

```
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class SimpleHistoryFileNameProvider
  extends DefaultHistoryFileNameProvider {

    public String getHistoryFileName() {
        return "baeldung-shell.log";
    }

    public String getProviderName() {
        return "Baeldung History";
    }

}
```

历史文件将记录在 shell 中执行的所有命令，并将放在我们的应用程序旁边。

一切就绪后，我们可以调用我们的 shell 并查看它的运行情况:

```
=======================================
*          Baeldung Shell             *
=======================================
Version:1.0.1
Welcome to Baeldung CLI
baeldung-shell>
```

## 6。转换器

到目前为止，我们只使用简单类型作为命令的参数。常见类型如`Integer`、`Date`、`Enum`、`File`等。，已经注册了一个默认转换器。

通过实现`Converter`接口，我们还可以添加转换器来接收定制对象。

让我们创建一个可以将`String`转换成`URL`的转换器:

```
@Component
public class SimpleURLConverter implements Converter<URL> {

    public URL convertFromText(
      String value, Class<?> requiredType, String optionContext) {
        return new URL(value);
    }

    public boolean getAllPossibleValues(
      List<Completion> completions,
      Class<?> requiredType,
      String existingData,
      String optionContext,
      MethodTarget target) {
        return false;
    }

    public boolean supports(Class<?> requiredType, String optionContext) {
        return URL.class.isAssignableFrom(requiredType);
    }
}
```

最后，让我们修改我们的`web-get`和`web-save`命令:

```
public String webSave(... URL url) {
    // ...
}

public String webSave(... URL url) {
    // ...
}
```

您可能已经猜到了，这些命令的行为是一样的。

## 7。结论

在本文中，我们简要介绍了 Spring Shell 项目的核心特性。我们能够贡献我们的命令，并使用我们的提供者定制外壳，我们根据不同的运行时条件改变命令的可用性，并创建了一个简单的类型转换器。

本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625231140/https://github.com/eugenp/tutorials/tree/master/spring-shell)