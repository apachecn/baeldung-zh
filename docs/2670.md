# 如何在 Java 中获取文件的文件扩展名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-extension>

## 1.概观

在这个快速教程中，我们将展示如何在 Java 中以编程方式获得文件扩展名。我们将集中讨论解决这个问题的三种主要方法。

在我们的实现中，将返回最后一个'`.'` 后面的字符。

因此，举个简单的例子，如果我们的文件名是`jarvis.txt` ，那么它将返回`String` " `txt”` 作为文件的扩展名。

## 2.获取文件扩展名

对于每种方法，我们将学习如何实现它，并跟踪两种特殊情况下会发生什么:

*   当文件名没有扩展名时，如`makefile`文件
*   如果文件名仅由扩展名组成，如`.gitignore`或`.DS_Store.`

### 2.1.简单的`String`处理方法

通过这种方法，我们将使用一种简单的`String`处理方法来查找扩展:

```
public Optional<String> getExtensionByStringHandling(String filename) {
    return Optional.ofNullable(filename)
      .filter(f -> f.contains("."))
      .map(f -> f.substring(filename.lastIndexOf(".") + 1));
} 
```

此方法将检查点“.”出现在给定的文件名中。

如果它存在，那么它将找到点“.”的最后位置并返回其后的字符，即最后一个点“.”后的字符称为文件扩展名。

**特殊情况:**

1.  **没有扩展名**–这个方法将返回一个空的`String`
2.  **仅扩展名**–该方法将返回点号后的`String`，例如`“gitignore”`

### 2.2.来自 Apache Commons IO

在第二种方法中，我们将使用 Apache Commons IO 库提供的实用程序类来找到扩展:

```
public String getExtensionByApacheCommonLib(String filename) {
    return FilenameUtils.getExtension(filename);
}
```

这里，除了文件名，我们还可以指定一个文件的完整路径 `e.g.`“`C:/baeldung/com/demo.java`”。

方法`getExtension(String)`将检查给定的`filename`是否为空。

**如果`filename`为空或 null，`getExtension(String filename)`将返回它被给定的实例。否则，它返回文件名的扩展名。**

为此，它使用方法`indexOfExtension(String)`，该方法又使用`lastIndexof(char)`来查找最后一次出现的“.”。这些方法都是由`FilenameUtils`提供的。

这个方法还通过使用另一个方法`indexOfLastSeparator(String),`来检查最后一个点之后是否有目录分隔符，这个方法将处理 Unix 或 Windows 格式的文件。

**特殊情况:**

1.  **无扩展名**–该方法将返回一个空字符串。
2.  **仅扩展名**–该方法将返回点号后的`String`，例如`“gitignore”`

### 2.3。使用番石榴库

在最后一种方法中，我们将使用 Guava 库来查找扩展。

要添加一个番石榴库，我们可以将下面的依赖项添加到我们的`pom.xml:`

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

对于最新的依赖关系，我们可以查看 [Maven Central](https://web.archive.org/web/20220812132830/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 。

添加库之后，我们可以简单地使用它的`getFileExtension`方法:

```
public String getExtensionByGuava(String filename) {
    return Files.getFileExtension(filename);
} 
```

方法`getFileExtension(String)`将首先检查给定的`filename`是否为空。

如果`filename`不为空，那么它将通过将给定的`filename`转换成抽象路径名来创建一个`File`实例，并对其调用`File's` `getName()`方法，这将返回由该抽象路径名表示的文件名，或者如果给定的`filename`为空，则返回空字符串。

基于这个返回值，它获取最后一次出现“.”的索引通过使用`String`类内置方法`lastIndexOf(char)`。

**特殊情况:**

1.  没有扩展名——这个方法将返回一个空的`String`
2.  仅扩展名–该方法将返回点号后的`String`，例如`“gitignore”`

## 3.结论

**当在 Apache `Commons`和`Guava`之间选择时，虽然这两个库都有一些共同的特性，而且它们也有一些功能是它们的替代品所没有的。**

这意味着，如果某项功能是必需的，就选择具备该功能的。否则，如果需要更多的自定义场景，请选择最能满足您需求的场景，并随意用您自己的实现来包装它，以获得想要的结果。

另外，在 Github 上查看本文[中的所有例子。](https://web.archive.org/web/20220812132830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)