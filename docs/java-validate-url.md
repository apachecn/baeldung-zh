# 在 Java 中验证 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-validate-url>

## 1.概观

**[URL](/web/20221220190132/https://www.baeldung.com/java-url) 代表统一资源定位器，是网络上唯一资源的地址。**

在本教程中，我们将讨论使用 Java 验证 URL。在现代 web 开发中，通过应用程序读取、写入或访问 URL 是非常常见的。因此，成功的验证可确保 URL 有效且合规。

有不同的库用于验证 URL。我们将讨论两个类——`[java.net.Url](https://web.archive.org/web/20221220190132/https://developer.classpath.org/doc/java/net/URL.html)` 来自 JDK，而`[org.apache.commons.validator.routines.UrlValidator](https://web.archive.org/web/20221220190132/https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html)` 来自 Apache Commons 库。

## 2.使用 JDK 验证 URL

让我们看看如何使用类`java.net.URL`来验证 URL:

```java
boolean isValidURL(String url) throws MalformedURLException, URISyntaxException {
    try {
        new URL(url).toURI();
        return true;
    } catch (MalformedURLException e) {
        return false;
    } catch (URISyntaxException e) {
        return false;
    }
}
```

在上面的方法中，*新建 URL(网址)。toURI()；*尝试使用`String`参数创建一个`URI`。如果传递的`String`不符合 URL 语法，库就会抛出一个`Exception`。

当内置的`URL`类在输入的`String`对象中发现格式错误的语法时，它抛出一个`MalformedURLException`。当`String`的格式不符合时，内置类抛出一个`URISyntaxException.`

现在，让我们通过一个小测试来验证我们的方法是否有效:

```java
assertTrue(isValidURL("http://baeldung.com/"));
assertFalse(isValidURL("https://www.baeldung.com/ java-%%$^&& iuyi"));
```

我们必须了解 [URL 和 URI](/web/20221220190132/https://www.baeldung.com/java-url-vs-uri) 的区别。 **`toURI()` 方法在这里很重要，因为它确保任何符合 [RC 2396](https://web.archive.org/web/20221220190132/https://datatracker.ietf.org/doc/html/rfc2396) 的 URL 字符串都被转换为`URL`。然而，如果我们使用`new URL(String value),`，它不能确保创建的 URL 是完全兼容的。**

让我们看一个例子，如果我们只使用`new URL(String url)`，许多不符合的 URL 将通过验证:

```java
boolean isValidUrl(String url) throws MalformedURLException {
    try {
        // it will check only for scheme and not null input 
        new URL(url);
        return true;
    } catch (MalformedURLException e) {
        return false;
    }
} 
```

让我们看看上面的方法如何通过一些测试来验证不同的 URL:

```java
assertTrue(isValidUrl("http://baeldung.com/"));
assertTrue(isValidUrl("https://www.baeldung.com/ java-%%$^&& iuyi")); 
assertFalse(isValidUrl(""));
```

**在上面的方法中，`[new URL(url)](https://web.archive.org/web/20221220190132/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/net/URL.html#%3Cinit%3E(java.lang.String)) `只检查有效的协议和空字符串作为输入。**因此，如果协议是正确的，它将返回一个`URL`对象，即使它不符合 RC 2396。

因此，**我们必须使用`new [URL(url).toURI()](https://web.archive.org/web/20221220190132/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/net/URL.html#toURI())`来确保 URL 是有效的并且符合**。

## 3.使用 Apache Commons 验证 URL

我们需要将`[commons-validator](https://web.archive.org/web/20221220190132/https://mvnrepository.com/artifact/commons-validator/commons-validator)`依赖项导入到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>commons-validator</groupId>
    <artifactId>commons-validator</artifactId>
    <version>1.7</version>
</dependency>
```

让我们使用这个库中的`UrlValidator` 类来验证:

```java
boolean isValidURL(String url) throws MalformedURLException {
    UrlValidator validator = new UrlValidator();
    return validator.isValid(url);
}
```

在上面的方法中，我们**创建一个 *URLValidator* ，然后使用 *isValid()* 方法检查`String`参数的 URL 有效性。**

让我们看看上面的方法对于不同的输入是如何表现的:

```java
assertFalse(isValidURL("https://www.baeldung.com/ java-%%$^&& iuyi"));
assertTrue(isValidURL("http://baeldung.com/"));
```

*URLValidator* 允许我们微调验证 URL 字符串的条件。例如，如果我们使用重载的构造函数`UrlValidator(String[] schemes)`，它只为提供的方案列表(`http`、`https`、`ftp`等)验证 URL。).

同样，还有其他一些标志——`ALLOW_2_SLASHES`、`NO_FRAGMENT,`和`ALLOW_ALL_SCHEMES` ，可以根据需要进行设置。我们可以在[官方文档](https://web.archive.org/web/20221220190132/https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html)中找到该库提供的所有选项的详细信息。

## 4.结论

在本文中，我们学习了两种不同的验证 URL 的方法。我们还讨论了`URL(String url)` 和 `URL.toURI()`的区别。

如果我们只需要验证协议和非空字符串，那么我们可以使用`URL(String url)` 构造函数`.` ,然而，当我们需要验证并通过一致性检查时，我们需要使用`URL(url).to URI().`

此外，如果我们添加 Apache Commons 依赖项，我们可以使用 *URLValidator* 类来执行验证，该类中还有其他选项可用于微调验证规则。

与往常一样，本文中示例的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221220190132/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-4)