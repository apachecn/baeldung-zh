# Groovy 中的 I/O 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-io>

## 1.介绍

尽管在 [Groovy](/web/20220524024917/https://www.baeldung.com/groovy-language) 中，我们可以像在 Java 中一样处理 I/O，但是 **Groovy 用许多助手方法扩展了 Java 的 I/O 功能。**

在本教程中，我们将看看如何通过 Groovy 的`File` 扩展方法读写文件、遍历文件系统以及序列化数据和对象。

在适用的地方，我们将链接到我们的相关 Java 文章，以便于与 Java 对等物进行比较。

## 2.读取文件

Groovy 以`eachLine`方法的形式为[读取文件](/web/20220524024917/https://www.baeldung.com/groovy-file-read)添加了方便的功能，获取`BufferedReader`和`InputStream`的方法，以及用一行代码获取所有文件数据的方法。

Java 7 和 Java 8 对 Java 中的[读取文件有类似的支持。](/web/20220524024917/https://www.baeldung.com/reading-file-in-java)

### 2.1.用`eachLine`阅读

在处理文本文件时，我们经常需要阅读每一行并对其进行处理。 **Groovy 用 `eachLine`方法**为`java.io.File` 提供了一个方便的扩展:

```java
def lines = []

new File('src/main/resources/ioInput.txt').eachLine { line ->
    lines.add(line)
}
```

提供给`eachLine`的闭包也有一个有用的可选行号。让我们使用行号从文件中只获取特定的行:

```java
def lineNoRange = 2..4
def lines = []

new File('src/main/resources/ioInput.txt').eachLine { line, lineNo ->
    if (lineNoRange.contains(lineNo)) {
        lines.add(line)
    }
}
```

**默认情况下，行号从 1 开始。**我们可以通过将一个值作为第一个参数传递给`eachLine`方法来提供该值作为第一个行号。

让我们从零开始我们的行号:

```java
new File('src/main/resources/ioInput.txt').eachLine(0, { line, lineNo ->
    if (lineNoRange.contains(lineNo)) {
        lines.add(line)
    }
})
```

如果在`eachLine, ` **中抛出异常，Groovy 会确保文件资源被关闭**。很像 Java 中的`try-with-resources`或`try-finally`。

### 2.2.用阅读器阅读

我们也可以很容易地从一个 Groovy `File`对象中获得一个`BufferedReader`。我们可以使用`withReader`来获取文件对象的`BufferedReader`，并将其传递给闭包:

```java
def actualCount = 0
new File('src/main/resources/ioInput.txt').withReader { reader ->
    while(reader.readLine()) {
        actualCount++
    }
}
```

与`eachLine`一样，`withReader`方法会在抛出异常时自动关闭资源。

有时，我们可能希望让`BufferedReader`对象可用。例如，我们可能计划调用一个将一作为参数的方法。为此，我们可以使用`newReader`方法:

```java
def outputPath = 'src/main/resources/ioOut.txt'
def reader = new File('src/main/resources/ioInput.txt').newReader()
new File(outputPath).append(reader)
reader.close()
```

与我们到目前为止看到的其他方法不同，**当我们以这种方式获得一个`Buffered` `Reader`时，我们负责关闭`BufferedReader`资源。**

### 2.3.用`InputStream` s 阅读

与`withReader`和`newReader`类似， **Groovy 也提供了轻松使用`InputStream` s** 的方法。虽然我们可以读取带有`InputStream`的文本，Groovy 甚至为其添加了功能，但是`InputStream`最常用于二进制数据。

让我们使用`withInputStream`向闭包传递一个`InputStream`并读入字节:

```java
byte[] data = []
new File("src/main/resources/binaryExample.jpg").withInputStream { stream ->
    data = stream.getBytes()
}
```

如果我们需要一个`InputStream`对象，我们可以使用`newInputStream`得到一个:

```java
def outputPath = 'src/main/resources/binaryOut.jpg'
def is = new File('src/main/resources/binaryExample.jpg').newInputStream()
new File(outputPath).append(is)
is.close()
```

与`BufferedReader`一样，当我们使用`newInputStream,`时，我们需要自己关闭`InputStream`资源，而不是在使用`withInputStream`时。

### 2.4.阅读其他方式

让我们通过查看 Groovy 在一条语句中获取所有文件数据的一些方法来结束阅读主题。

如果我们希望我们的文件行在一个`List`中，我们可以使用`collect`和一个传递给闭包的迭代器`it`:

```java
def actualList = new File('src/main/resources/ioInput.txt').collect {it}
```

要将文件中的行放入一个数组`Strings`，我们可以使用`as String[]`:

```java
def actualArray = new File('src/main/resources/ioInput.txt') as String[]
```

对于短文件，我们可以使用`text`获得一个`String`中的全部内容:

```java
def actualString = new File('src/main/resources/ioInput.txt').text
```

当处理二进制文件时，有一个`bytes`方法:

```java
def contents = new File('src/main/resources/binaryExample.jpg').bytes
```

## 3.写文件

在我们开始[写入文件](/web/20220524024917/https://www.baeldung.com/java-write-to-file)之前，让我们设置一下将要输出的文本:

```java
def outputLines = [
    'Line one of output example',
    'Line two of output example',
    'Line three of output example'
]
```

### 3.1.用 Writer 写作

与读取文件一样，**我们也可以很容易地从`File`对象**中获取`BufferedWriter`。

让我们使用`withWriter`来获取一个`BufferedWriter`并将其传递给一个闭包:

```java
def outputFileName = 'src/main/resources/ioOutput.txt'
new File(outputFileName).withWriter { writer ->
    outputLines.each { line ->
        writer.writeLine line
    }
}
```

如果出现异常，使用`withReader`将关闭资源。

Groovy 还有一个获取`BufferedWriter`对象的方法。让我们用`newWriter`得到一个`BufferedWriter`:

```java
def outputFileName = 'src/main/resources/ioOutput.txt'
def writer = new File(outputFileName).newWriter()
outputLines.forEach {line ->
    writer.writeLine line
}
writer.flush()
writer.close()
```

当我们使用`newWriter`时，我们负责刷新和关闭我们的`BufferedWriter`对象。

### 3.2.使用输出流写入

如果我们正在写出二进制数据，**我们可以使用`withOutputStream`或`newOutputStream`** 得到一个`OutputStream`。

让我们使用`withOutputStream`将一些字节写入文件:

```java
byte[] outBytes = [44, 88, 22]
new File(outputFileName).withOutputStream { stream ->
    stream.write(outBytes)
}
```

让我们用`newOutputStream`得到一个`OutputStream`对象，并用它来写一些字节:

```java
byte[] outBytes = [44, 88, 22]
def os = new File(outputFileName).newOutputStream()
os.write(outBytes)
os.close()
```

类似于`InputStream`、`BufferedReader`、`BufferedWriter`，当我们使用`newOutputStream`时，我们自己负责关闭`OutputStream`。

### 3.3.使用<

由于将文本写入文件是如此普遍，所以`<<`操作符直接提供了这个特性。

让我们使用`<<`操作符来编写一些简单的文本行:

```java
def ln = System.getProperty('line.separator')
def outputFileName = 'src/main/resources/ioOutput.txt'
new File(outputFileName) << "Line one of output example${ln}" + 
  "Line two of output example${ln}Line three of output example"
```

### 3.4.用字节写入二进制数据

**我们在文章前面看到，我们可以简单地通过访问`bytes`字段从二进制文件中获取所有字节。**

让我们用同样的方式写二进制数据:

```java
def outputFileName = 'src/main/resources/ioBinaryOutput.bin'
def outputFile = new File(outputFileName)
byte[] outBytes = [44, 88, 22]
outputFile.bytes = outBytes
```

## 4.遍历文件树

Groovy 也为我们提供了使用文件树的简单方法。在本节中，我们将使用`eachFile`、`eachDir`及其变体和`traverse`方法来实现。

### 4.1.用`eachFile`列出文件

让我们使用`eachFile`列出一个目录中的所有文件和目录:

```java
new File('src/main/resources').eachFile { file ->
    println file.name
}
```

处理文件时的另一个常见场景是需要根据文件名过滤文件。我们只列出以“io”开头，以”结尾的文件。txt "使用`eachFileMatch` 和一个正则表达式:

```java
new File('src/main/resources').eachFileMatch(~/io.*\.txt/) { file ->
    println file.name
}
```

**`eachFile`和`eachFileMatch`方法只列出顶层目录的内容。Groovy 还允许我们通过向方法传递一个`FileType`来限制`eachFile`方法返回什么。选项有`ANY`、`FILES`和`DIRECTORIES`。**

让我们使用`eachFileRecurse`递归地列出所有文件，并为其提供一个`FILES`的`FileType`:

```java
new File('src/main').eachFileRecurse(FileType.FILES) { file ->
    println "$file.parent $file.name"
}
```

如果我们给方法提供一个文件的路径而不是一个目录，`eachFile`方法就会抛出一个`IllegalArgumentException`。

**Groovy 还提供了只处理目录的`eachDir`方法。我们可以使用`eachDir`及其变体来完成与使用`eachFile`和`DIRECTORIES`的`FileType`相同的事情。**

让我们用`eachFileRecurse`递归地列出目录:

```java
new File('src/main').eachFileRecurse(FileType.DIRECTORIES) { file ->
    println "$file.parent $file.name"
}
```

现在，让我们对`eachDirRecurse`做同样的事情:

```java
new File('src/main').eachDirRecurse { dir ->
    println "$dir.parent $dir.name"
}
```

### 4.2.用遍历列出文件

**对于[更复杂的目录遍历](/web/20220524024917/https://www.baeldung.com/java-nio2-file-visitor)用例，我们可以使用`traverse`方法。**它的功能与`eachFileRecurse`相似，但是提供了返回`FileVisitResult`对象来控制处理的能力。

让我们在我们的 *src/main* 目录下使用`traverse`,并跳过处理`groovy`目录下的树:

```java
new File('src/main').traverse { file ->
   if (file.directory && file.name == 'groovy') {
        FileVisitResult.SKIP_SUBTREE
    } else {
        println "$file.parent - $file.name"
    }
}
```

## 5.使用数据和对象

### 5.1.序列化原语

在 Java 中，我们可以使用`DataInputStream`和`DataOutputStream`来[序列化原始数据字段](/web/20220524024917/https://www.baeldung.com/java-serialization)。Groovy 在这里也添加了有用的扩展。

让我们建立一些原始数据:

```java
String message = 'This is a serialized string'
int length = message.length()
boolean valid = true
```

现在，让我们使用`withDataOutputStream`将数据序列化到一个文件中:

```java
new File('src/main/resources/ioData.txt').withDataOutputStream { out ->
    out.writeUTF(message)
    out.writeInt(length)
    out.writeBoolean(valid)
}
```

并使用`withDataInputStream`将其读回:

```java
String loadedMessage = ""
int loadedLength
boolean loadedValid

new File('src/main/resources/ioData.txt').withDataInputStream { is ->
    loadedMessage = is.readUTF()
    loadedLength = is.readInt()
    loadedValid = is.readBoolean()
}
```

类似于其他的`with*`方法，`withDataOutputStream`和`withDataInputStream`将流传递给闭包，并确保它正确关闭。

### 5.2.序列化对象

**Groovy 还构建在 Java 的`ObjectInputStream`和`ObjectOutputStream`之上，允许我们轻松地序列化实现`Serializable`** 的对象。

让我们首先定义一个实现`Serializable`的类:

```java
class Task implements Serializable {
    String description
    Date startDate
    Date dueDate
    int status
}
```

现在让我们创建一个可以序列化到文件中的`Task`实例:

```java
Task task = new Task(description:'Take out the trash', startDate:new Date(), status:0)
```

有了我们的`Task`对象，让我们使用`withObjectOutputStream`将它序列化为一个文件:

```java
new File('src/main/resources/ioSerializedObject.txt').withObjectOutputStream { out ->
    out.writeObject(task)
}
```

最后，让我们用`withObjectInputStream`来读一下我们的`Task`:

```java
Task taskRead

new File('src/main/resources/ioSerializedObject.txt').withObjectInputStream { is ->
    taskRead = is.readObject()
}
```

我们使用的方法`withObjectOutputStream`和`withObjectInputStream`将流传递给闭包，并适当地处理关闭资源，就像其他`with*`方法一样。

## 6.结论

在本文中，我们探索了 Groovy 添加到现有 Java 文件 I/O 类中的功能。我们使用这个功能来读写文件、使用目录结构以及序列化数据和对象。

我们只涉及了一些 helper 方法，所以有必要深入研究一下 Groovy 的文档,看看它还为 Java 的 I/O 功能增加了什么。

GitHub 上的[提供了示例代码。](https://web.archive.org/web/20220524024917/https://github.com/eugenp/tutorials/tree/master/core-groovy)