# 在 Java 中获取文件的 Mime 类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-mime-type>

## 1。概述

在本教程中，我们将看看获取文件 MIME 类型的各种策略。我们将研究在任何适用的地方扩展策略可用的 MIME 类型的方法。

我们还将指出我们应该在哪些方面支持一种策略。

## 2。使用 Java 7

让我们从 Java 7 开始——它提供了解析 MIME 类型的方法 [`Files.probeContentType(path)`](https://web.archive.org/web/20220913091603/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#probeContentType(java.nio.file.Path)) :

```java
@Test
public void whenUsingJava7_thenSuccess() {
    Path path = new File("product.png").toPath();
    String mimeType = Files.probeContentType(path);

    assertEquals(mimeType, "image/png");
} 
```

这个方法利用已安装的`FileTypeDetector` 实现来探测 MIME 类型。它调用每个实现的`probeContentType` 来解析类型。

现在，如果文件被任何实现识别，则返回内容类型。然而，如果没有发生这种情况，系统默认的文件类型检测器被调用。

然而，默认实现是特定于操作系统的，根据我们使用的操作系统，可能会失败。

除此之外，还需要注意的是，如果文件不在文件系统中，这个策略将会失败。此外，如果文件没有扩展名，将会导致失败。

## 3。使用`URLConnection`

提供了几个 API 来检测文件的 MIME 类型。让我们简单地探讨一下它们。

### 3.1。使用`getContentType()`

我们可以使用`URLConnection`的`getContentType()`方法来检索文件的 MIME 类型:

```java
@Test
public void whenUsingGetContentType_thenSuccess(){
    File file = new File("product.png");
    URLConnection connection = file.toURL().openConnection();
    String mimeType = connection.getContentType();

    assertEquals(mimeType, "image/png");
}
```

然而，这种方法的一个主要缺点是**非常慢**。

### 3.2。使用`guessContentTypeFromName()`

接下来，让我们看看如何利用`guessContentTypeFromName()`来达到这个目的:

```java
@Test
public void whenUsingGuessContentTypeFromName_thenSuccess(){
    File file = new File("product.png");
    String mimeType = URLConnection.guessContentTypeFromName(file.getName());

    assertEquals(mimeType, "image/png");
}
```

这个方法利用内部的`FileNameMap`到**从扩展**中解析 MIME 类型。

我们也可以选择使用`guessContentTypeFromStream()`来代替，它使用输入流的前几个字符来确定类型。

### 3.3。使用`getFileNameMap` ()

使用`URLConnection`获得 MIME 类型的一个更快的方法是使用`getFileNameMap()`方法:

```java
@Test
public void whenUsingGetFileNameMap_thenSuccess(){
    File file = new File("product.png");
    FileNameMap fileNameMap = URLConnection.getFileNameMap();
    String mimeType = fileNameMap.getContentTypeFor(file.getName());

    assertEquals(mimeType, "image/png");
}
```

该方法返回所有`URLConnection.` 实例使用的 MIME 类型表，然后使用该表解析输入文件类型。

到了`URLConnection`，内置的 MIME 类型表就非常有限了。

默认情况下，**类使用`JRE_HOME/lib`中的`content-types.properties`** 文件。**然而，我们可以扩展它，通过使用`content.types.user.table `属性:**指定一个用户特定的表

```java
System.setProperty("content.types.user.table","<path-to-file>"); 
```

## 4。使用`MimeTypesFileTypeMap`

`MimeTypesFileTypeMap`使用文件扩展名解析 MIME 类型。这个类是 Java 6 附带的，因此当我们使用 JDK 1.6 时非常方便。

现在让我们看看如何使用它:

```java
@Test
public void whenUsingMimeTypesFileTypeMap_thenSuccess() {
    File file = new File("product.png");
    MimetypesFileTypeMap fileTypeMap = new MimetypesFileTypeMap();
    String mimeType = fileTypeMap.getContentType(file.getName());

    assertEquals(mimeType, "image/png");
}
```

这里，我们可以将文件名或`File`实例本身作为参数传递给函数。但是，将`File`实例作为参数的函数在内部调用接受文件名作为参数的重载方法。

**在内部，这个方法查找一个名为`mime.types`的文件来获取类型解析。值得注意的是，该方法按照特定的顺序搜索文件:**

1.  以编程方式向`MimetypesFileTypeMap`实例添加条目
2.  。`mime.types`在用户的主目录中
3.  `<java.home>/lib/mime.types`
4.  名为`META-INF/mime.types`的资源
5.  名为`META-INF/mimetypes.default`的资源(通常只能在`activation.jar`文件中找到)

但是，如果没有找到文件，它将返回`application/octet-stream`作为响应。

## 5。使用`jMimeMagic`

jMimeMagic 是一个受限许可的库，我们可以用它来获取文件的 MIME 类型。

让我们从配置 Maven 依赖项开始:

```java
<dependency>
    <groupId>net.sf.jmimemagic</groupId>
    <artifactId>jmimemagic</artifactId>
    <version>0.1.5</version>
</dependency>
```

我们可以在 Maven Central 上找到这个库的最新版本。

接下来，我们将探索如何使用该库:

```java
@Test    
public void whenUsingJmimeMagic_thenSuccess() {
    File file = new File("product.png");
    Magic magic = new Magic();
    MagicMatch match = magic.getMagicMatch(file, false);

    assertEquals(match.getMimeType(), "image/png");
}
```

这个库可以处理数据流，因此不需要文件存在于文件系统中。

## 6。使用阿帕奇 Tika

Apache Tika 是一个工具集，可以从各种文件中检测和提取元数据和文本。它有一个丰富而强大的 API，并带有我们可以利用的 [tika-core](https://web.archive.org/web/20220913091603/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.tika%22%20AND%20a%3A%22tika-core%22) ，用于检测文件的 MIME 类型。

让我们从配置 Maven 依赖性开始:

```java
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>1.18</version>
</dependency>
```

接下来，我们将利用`detect()` 方法来解析类型:

```java
@Test
public void whenUsingTika_thenSuccess() {
    File file = new File("product.png");
    Tika tika = new Tika();
    String mimeType = tika.detect(file);

    assertEquals(mimeType, "image/png");
}
```

该库依赖流前缀中的神奇标记来进行类型解析。

## 7 .**。结论**

在本文中，我们研究了获取文件 MIME 类型的各种策略。此外，我们也分析了这些方法的利弊。我们还指出了我们应该选择一种策略而不是另一种策略的场景。

一如既往，本文中使用的完整源代码可以从 GitHub 的[处获得。](https://web.archive.org/web/20220913091603/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)