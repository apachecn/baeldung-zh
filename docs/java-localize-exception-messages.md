# 用 Java 本地化异常消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-localize-exception-messages>

## 1.概观

Java 中的异常用来表示程序中出现了错误。除了抛出异常，我们甚至可以添加一条消息来提供额外的信息。

在本文中，我们将利用`getLocalizedMessage`方法以英语和法语提供异常消息。

## 2.资源包

我们需要一种方法来查找消息，使用`messageKey`来标识消息，使用 [`Locale`](/web/20221208143832/https://www.baeldung.com/java-8-localization#localization) 来标识哪个翻译将为`messageKey`提供值。我们将创建一个简单的类来抽象对我们的 [`ResourceBundle`](/web/20221208143832/https://www.baeldung.com/java-resourcebundle) 的访问，以检索英语和法语消息翻译:

```java
public class Messages {

    public static String getMessageForLocale(String messageKey, Locale locale) {
        return ResourceBundle.getBundle("messages", locale)
          .getString(messageKey);
    }

} 
```

我们的`Messages`类使用`ResourceBundle`将属性文件加载到我们的 bundle 中，bundle 位于我们的类路径的根位置。我们有两个文件，一个用于我们的英语信息，一个用于我们的法语信息:

```java
# messages.properties
message.exception = I am an exception.
```

```java
# messages_fr.properties
message.exception = Je suis une exception.
```

## 3.本地化异常类

我们的`Exception`子类将使用默认的 [`Locale`](/web/20221208143832/https://www.baeldung.com/java-8-localization#localization) 来决定对我们的消息使用哪种翻译。我们将使用`Locale#getDefault`得到默认的 [`Locale`](/web/20221208143832/https://www.baeldung.com/java-8-localization#localization) 。

**如果我们的应用程序运行在服务器上，我们将使用 HTTP 请求头来标识要使用的`Locale`,而不是设置默认值。**为此，我们将创建一个构造函数来接受一个`Locale.`

让我们创建我们的`Exception`子类。为此，我们可以扩展`RuntimeException`或`Exception`。让我们扩展`Exception`并覆盖`getLocalizedMessage`:

```java
public class LocalizedException extends Exception {

    private final String messageKey;
    private final Locale locale;

    public LocalizedException(String messageKey) {
        this(messageKey, Locale.getDefault());
    }

    public LocalizedException(String messageKey, Locale locale) {
        this.messageKey = messageKey;
        this.locale = locale;
    }

    public String getLocalizedMessage() {
        return Messages.getMessageForLocale(messageKey, locale);
    }
} 
```

## 4.把所有的放在一起

让我们创建一些单元测试来验证一切正常。我们将为英语和法语翻译创建测试，以验证在构建期间向异常传递自定义的`Locale`:

```java
@Test
public void givenUsEnglishProvidedLocale_whenLocalizingMessage_thenMessageComesFromDefaultMessage() {
    LocalizedException localizedException = new LocalizedException("message.exception", Locale.US);
    String usEnglishLocalizedExceptionMessage = localizedException.getLocalizedMessage();

    assertThat(usEnglishLocalizedExceptionMessage).isEqualTo("I am an exception.");
}

@Test
public void givenFranceFrenchProvidedLocale_whenLocalizingMessage_thenMessageComesFromFrenchTranslationMessages() {
    LocalizedException localizedException = new LocalizedException("message.exception", Locale.FRANCE);
    String franceFrenchLocalizedExceptionMessage = localizedException.getLocalizedMessage();

    assertThat(franceFrenchLocalizedExceptionMessage).isEqualTo("Je suis une exception.");
}
```

我们的异常也可以使用默认的`Locale`。让我们再创建两个测试来验证默认的`Locale`功能是否有效:

```java
@Test
public void givenUsEnglishDefaultLocale_whenLocalizingMessage_thenMessageComesFromDefaultMessages() {
    Locale.setDefault(Locale.US);

    LocalizedException localizedException = new LocalizedException("message.exception");
    String usEnglishLocalizedExceptionMessage = localizedException.getLocalizedMessage();

    assertThat(usEnglishLocalizedExceptionMessage).isEqualTo("I am an exception.");
}

@Test
public void givenFranceFrenchDefaultLocale_whenLocalizingMessage_thenMessageComesFromFrenchTranslationMessages() {
    Locale.setDefault(Locale.FRANCE);

    LocalizedException localizedException = new LocalizedException("message.exception");
    String franceFrenchLocalizedExceptionMessage = localizedException.getLocalizedMessage();

    assertThat(franceFrenchLocalizedExceptionMessage).isEqualTo("Je suis une exception.");
} 
```

## 5.警告

### 5.1.伐木可抛物

**我们需要记住我们用来将`Exception`实例发送到日志的日志框架。**

Log4J、Log4J2 和 Logback 使用`getMessage`来检索要写入日志附加器的消息。如果我们使用`java.util.logging`，内容来自`getLocalizedMessage`。

我们可能想考虑重写`getMessage`来调用`getLocalizedMessage`，这样我们就不必担心使用哪个日志实现了。

### 5.2.服务器端应用程序

当我们为客户端应用程序本地化我们的异常消息时，我们只需要担心一个系统的当前`Locale`。然而，**如果我们想在服务器端应用程序中本地化异常消息，我们应该记住，切换默认的`Locale`将影响我们的应用服务器中的所有请求。**

如果我们决定本地化异常消息，我们将在异常上创建一个构造函数来接受`Locale`。这将使我们能够在不更新默认`Locale`的情况下本地化我们的消息。

## 6.摘要

本地化异常消息相当简单。我们需要做的就是为我们的消息创建一个`ResourceBundle`，然后在我们的`Exception`子类中实现`getLocalizedMessage`。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)