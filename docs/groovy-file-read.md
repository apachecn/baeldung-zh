# 在 Groovy 中读取文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-file-read>

## 1.概观

在这个快速教程中，我们将探索在 [Groovy](/web/20220525131321/https://www.baeldung.com/groovy-language) 中读取文件的不同方式。

Groovy 提供了处理文件的便捷方式。我们将专注于`File`类，它有一些读取文件的帮助方法。

让我们在接下来的几节中逐一探讨它们。

## 2.逐行读取一个`File`

有许多像`readLine`和`eachLine`这样的[常规 IO 方法](https://web.archive.org/web/20220525131321/http://docs.groovy-lang.org/2.4.7/html/gapi/org/codehaus/groovy/runtime/IOGroovyMethods.html)可用于逐行读取文件。

### 2.1.使用`File.withReader`

先说 [`File`](https://web.archive.org/web/20220525131321/http://docs.groovy-lang.org/next/html/groovy-jdk/java/io/File.html) `.withReader`法。**它在封面下创建了一个新的`BufferedReader`，我们可以使用`readLine` 方法读取内容。**

例如，让我们逐行读取一个文件，并打印每一行。我们还将返回行数:

```java
int readFileLineByLine(String filePath) {
    File file = new File(filePath)
    def line, noOfLines = 0;
    file.withReader { reader ->
        while ((line = reader.readLine()) != null) {
            println "${line}"
            noOfLines++
        }
    }
    return noOfLines
}
```

让我们创建一个包含以下内容的纯文本文件`fileContent.txt`,并将其用于测试:

```java
Line 1 : Hello World!!!
Line 2 : This is a file content.
Line 3 : String content
```

让我们测试一下我们的实用方法:

```java
def 'Should return number of lines in File given filePath' () {
    given:
        def filePath = "src/main/resources/fileContent.txt"
    when:
        def noOfLines = readFile.readFileLineByLine(filePath)
    then:
        noOfLines
        noOfLines instanceof Integer
        assert noOfLines, 3
} 
```

**`withReader`方法也可以和 UTF-8 或 ASCII 这样的字符集参数一起使用来读取编码文件**。让我们看一个例子:

```java
new File("src/main/resources/utf8Content.html").withReader('UTF-8') { reader ->
def line
    while ((line = reader.readLine()) != null) { 
        println "${line}"
    }
}
```

### 2.2.使用`File.eachLine`

我们也可以使用`eachLine`方法:

```java
new File("src/main/resources/fileContent.txt").eachLine { line ->
    println line
} 
```

### 2.3.使用`File.newInputStream`和`InputStream.eachLine`

让我们看看如何使用`InputStream`和`eachLine`来读取文件:

```java
def is = new File("src/main/resources/fileContent.txt").newInputStream()
is.eachLine { 
    println it
}
is.close()
```

**当我们使用`newInputStream`方法时，我们必须处理关闭`InputStream`** 。

如果我们改为使用`withInputStream`方法，它将为我们处理关闭`InputStream`:

```java
new File("src/main/resources/fileContent.txt").withInputStream { stream ->
    stream.eachLine { line ->
        println line
    }
}
```

## 3.将一个`File`读入一个`List`

有时我们需要将文件的内容读入一系列行中。

### 3.1.使用`File.readLines`

为此，我们可以使用`readLines`方法，将文件读入到`Strings`的`List`中。

让我们快速查看一个读取文件内容并返回一个行列表的示例:

```java
List<String> readFileInList(String filePath) {
    File file = new File(filePath)
    def lines = file.readLines()
    return lines
}
```

让我们用`fileContent.txt`写一个快速测试:

```java
def 'Should return File Content in list of lines given filePath' () {
    given:
        def filePath = "src/main/resources/fileContent.txt"
    when:
        def lines = readFile.readFileInList(filePath)
    then:
        lines
        lines instanceof List<String>
        assert lines.size(), 3
}
```

### 3.2.使用`File.collect`

我们还可以使用`collect` API 将文件内容读入到`Strings`的`List`中:

```java
def list = new File("src/main/resources/fileContent.txt").collect {it} 
```

### 3.3.使用 `as`操作符

我们甚至可以利用`as`操作符将文件的内容读入一个`String`数组:

```java
def array = new File("src/main/resources/fileContent.txt") as String[]
```

## 4.将一个`File`读入一个单独的`String`

### 4.1.使用`File.text`

**我们可以简单地通过使用`File`类**的`text`属性将整个文件读入一个单独的`String`。

让我们来看一个例子:

```java
String readFileString(String filePath) {
    File file = new File(filePath)
    String fileContent = file.text
    return fileContent
} 
```

让我们用一个单元测试来验证这一点:

```java
def 'Should return file content in string given filePath' () {
    given:
        def filePath = "src/main/resources/fileContent.txt"
    when:
        def fileContent = readFile.readFileString(filePath)
    then:
        fileContent
        fileContent instanceof String
        fileContent.contains("""Line 1 : Hello World!!!
Line 2 : This is a file content.
Line 3 : String content""")
}
```

### 4.2.使用`File.getText`

如果我们使用`getTest(charset)` 方法，我们可以通过提供像 UTF-8 或 ASCII 这样的字符集参数将编码文件的内容读入到`String`中:

```java
String readFileStringWithCharset(String filePath) {
    File file = new File(filePath)
    String utf8Content = file.getText("UTF-8")
    return utf8Content
}
```

让我们为单元测试创建一个包含 UTF-8 内容的 HTML 文件，名为`utf8Content.html`:

```java
ᚠᛇᚻ᛫ᛒᛦᚦ᛫ᚠᚱᚩᚠᚢᚱ᛫ᚠᛁᚱᚪ᛫ᚷᛖᚻᚹᛦᛚᚳᚢᛗ
ᛋᚳᛖᚪᛚ᛫ᚦᛖᚪᚻ᛫ᛗᚪᚾᚾᚪ᛫ᚷᛖᚻᚹᛦᛚᚳ᛫ᛗᛁᚳᛚᚢᚾ᛫ᚻᛦᛏ᛫ᛞᚫᛚᚪᚾ
ᚷᛁᚠ᛫ᚻᛖ᛫ᚹᛁᛚᛖ᛫ᚠᚩᚱ᛫ᛞᚱᛁᚻᛏᚾᛖ᛫ᛞᚩᛗᛖᛋ᛫ᚻᛚᛇᛏᚪᚾ 
```

让我们看看单元测试:

```java
def 'Should return UTF-8 encoded file content in string given filePath' () {
    given:
        def filePath = "src/main/resources/utf8Content.html"
    when:
        def encodedContent = readFile.readFileStringWithCharset(filePath)
    then:
        encodedContent
        encodedContent instanceof String
}
```

## 5.用`File.bytes`读取二进制文件

Groovy 使得读取非文本或二进制文件变得很容易。**通过使用`bytes`属性，我们可以得到`File`的内容作为`byte`数组**:

```java
byte[] readBinaryFile(String filePath) {
    File file = new File(filePath)
    byte[] binaryContent = file.bytes
    return binaryContent
}
```

我们将使用一个 png 图像文件`sample.png`，包含以下单元测试内容:

[![](img/f98245f31495de47bc8340ce3db90537.png)](/web/20220525131321/https://www.baeldung.com/wp-content/uploads/2019/02/sample.png)

让我们看看单元测试:

```java
def 'Should return binary file content in byte array given filePath' () {
    given:
        def filePath = "src/main/resources/sample.png"
    when:
        def binaryContent = readFile.readBinaryFile(filePath)
    then:
        binaryContent
        binaryContent instanceof byte[]
        binaryContent.length == 329
}
```

## 6.结论

在这个快速教程中，我们已经看到了使用`File`类的各种方法以及`BufferedReader`和`InputStream`在 Groovy 中读取文件的不同方式。

这些实现和单元测试用例的完整源代码可以在 [GitHub](https://web.archive.org/web/20220525131321/https://github.com/eugenp/tutorials/tree/master/core-groovy) 项目中找到。