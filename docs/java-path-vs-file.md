# Java–路径与文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-path-vs-file>

## 1.概观

在 Java 中，`Path`和`File`是负责文件 I/O 操作的类。它们执行相同的功能，但属于不同的包。

在本教程中，我们将讨论这两个类之间的差异。我们将从快速的课堂回顾开始。然后，我们将谈论一些遗留的缺点。最后，我们将学习如何在两个 API 之间移植功能。

## 2.`java.io.File`阶级

从第一个版本开始，Java 就发布了自己的`java.io`包，其中包含了我们可能需要执行输入和输出操作的几乎所有类。[`File`类](/web/20220627102551/https://www.baeldung.com/java-io-file)是**文件和目录路径名**的抽象表示:

```java
File file = new File("baeldung/tutorial.txt");
```

`File` 类的实例是不可变的——一旦创建，由该对象表示的抽象路径名将永远不会改变。

## 3.`java.nio.file.Path`阶级

[`Path`类](/web/20220627102551/https://www.baeldung.com/java-nio-2-file-api)构成了 [NIO2](/web/20220627102551/https://www.baeldung.com/java-nio-2-file-api) 更新的一部分，该更新随版本 7 来到 Java。它提供了一个全新的 API 来处理 I/O。此外，像遗留的`File`类一样，`Path`也创建了**一个对象，可以用来在文件系统**中定位文件。

同样，它**可以执行所有可以用`File`类完成的操作**:

```java
Path path = Paths.get("baeldung/tutorial.txt");
```

我们使用静态的 [`java.nio.file.Paths.get()`](https://web.archive.org/web/20220627102551/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Paths.html#get(java.lang.String,java.lang.String...)) 方法创建一个`Path` 实例，而不是像我们使用`File` API 那样使用构造函数。

## 4.`File `阶级弊端

在简要回顾了这两个类之后，现在让我们来讨论这两个 API 并回答这个问题:如果它们提供相同的功能，**为什么 Oracle 决定引入一个新的 API，我应该使用哪一个？**

正如我们所知，`java` `.io`包是在 Java JDK 的第一个版本中交付的，允许我们执行 I/O 操作。从那以后，许多开发人员报告了它的许多缺点、缺失的功能以及一些功能上的问题。

### 4.1.错误处理

最常见的问题是糟糕的错误处理。许多方法**不会告诉我们遇到问题**的任何细节，甚至根本不会抛出异常。

假设我们有一个删除文件的简单程序:

```java
File file = new File("baeldung/tutorial.txt");
boolean result = file.delete();
```

这段代码成功编译并运行，没有任何错误。当然，我们有一个包含一个`false`值的`result`标志，但是我们不知道这个失败的原因。文件可能不存在，或者程序没有删除它的权限。

我们现在可以使用较新的 NIO2 API 重写相同的功能:

```java
Path path = Paths.get("baeldung/tutorial.txt");
Files.delete(path);
```

现在，编译器要求我们处理一个`IOException`。此外，抛出的异常有关于其失败的详细信息，例如，文件是否不存在。

### 4.2.元数据支持

`java.io`包中的`File`类缺乏元数据支持，这导致跨不同平台的 I/O 操作需要关于文件的元信息时出现问题。

元数据还可以包括权限、文件所有者和安全属性。由于这个原因，`File `类**根本不支持符号链接**，并且 **`rename()`方法在不同的平台上不一致**。

### 4.3.方法缩放和性能

还有一个性能问题，因为`File`类的方法不可伸缩。这会导致一些包含大量文件的目录出现问题。列出目录**的内容可能会导致挂起，导致内存资源问题**。最后，它可能导致拒绝服务。

由于这些缺点，Oracle 开发了改进的 NIO2 API。开发人员**应该尽可能使用这个新的`java.nio`包**而不是遗留类来开始新的项目。

## 5.映射功能

为了修复`java.io`包中的一些缺口，甲骨文准备了自己的[缺点总结](https://web.archive.org/web/20220627102551/https://docs.oracle.com/javase/tutorial/essential/io/legacy.html)，帮助开发者在 API 之间进行迁移。

NIO2 包提供了所有遗留功能，包括对上述缺点的改进。由于大量应用程序可能仍在使用这个遗留 API，Oracle 目前不打算在未来的版本中弃用或移除旧 API 。

在新的 API 中，**代替了实例方法，我们使用了来自`[java.nio.file.Files](https://web.archive.org/web/20220627102551/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html)`类的静态方法**。现在让我们快速比较一下这些 API。

### 5.1.`File`和`Path `实例

当然，主要的区别是包和类名:

```java
java.io.File file = new java.io.File("baeldung/tutorial.txt");
java.nio.file.Path path = java.nio.file.Paths.get("baeldung/tutorial.txt");
```

在这里，我们通过构造函数构建一个`File`对象，同时通过使用静态方法获得一个`Path`。我们还可以使用多个参数来解析复杂的路径:

```java
File file = new File("baeldung", "tutorial.txt");
Path path = Paths.get("baeldung", "tutorial.txt"); 
```

并且，我们可以通过链接`resolve()`方法获得相同的结果:

```java
Path path2 = Paths.get("baeldung").resolve("tutorial.txt");
```

此外，我们可以使用 [`toPath()`](https://web.archive.org/web/20220627102551/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#toPath()) 和 [`toFile()`](https://web.archive.org/web/20220627102551/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html#toFile()) 方法在 API 之间转换对象:

```java
Path pathFromFile = file.toPath();
File fileFromPath = path.toFile();
```

### 5.2.管理文件和目录

这两种 API 都提供了管理文件和目录的方法。我们将使用之前创建的实例对象来演示这一点。

**创建文件**，我们可以使用`createNewFile()`和`Files.createFile()`方法:

```java
boolean result = file.createNewFile();
Path newPath = Files.createFile(path);
```

**要创建一个目录**，我们需要使用`mkdir()`或`Files.createDirectory()`:

```java
boolean result = file.mkdir();
File newPath = Files.createDirectory(path);
```

这些方法**还有其他的变体，通过`mkdirs()`和`Files.createDirectories()`方法来包含所有不存在的子目录**:

```java
boolean result = file.mkdirs();
File newPath = Files.createDirectories(path);
```

当我们希望**重命名或移动一个文件**时，我们需要创建另一个实例对象并使用`renameTo()`或`Files.move()`:

```java
boolean result = file.renameTo(new File("baeldung/tutorial2.txt"));
Path newPath = Files.move(path, Paths.get("baeldung/tutorial2.txt"));
```

**为了执行删除操作**，我们使用`delete()`或`Files.delete()`:

```java
boolean result = file.delete();
Files.delete(Paths.get(path));
```

请注意，遗留方法在出现任何错误时都会返回一个结果设置为`false`的标志。NIO2 方法返回一个新的`Path `实例，除了删除操作，它在错误发生时抛出一个`IOException`。

### 5.3.读取元数据

我们还可以获得一些关于文件的基本信息，比如权限或类型。和以前一样，我们需要一个实例对象:

```java
// java.io API
boolean fileExists = file.exists();
boolean fileIsFile = file.isFile();
boolean fileIsDir = file.isDirectory();
boolean fileReadable = file.canRead();
boolean fileWritable = file.canWrite();
boolean fileExecutable = file.canExecute();
boolean fileHidden = file.isHidden();

// java.nio API
boolean pathExists = Files.exists(path);
boolean pathIsFile = Files.isRegularFile(path);
boolean pathIsDir = Files.isDirectory(path);
boolean pathReadable = Files.isReadable(path);
boolean pathWritable = Files.isWritable(path);
boolean pathExecutable = Files.isExecutable(path);
boolean pathHidden = Files.isHidden(path);
```

### 5.4.路径名方法

最后，让我们快速看一下`File`类中用于获取文件系统路径的[方法。请注意，与前面的示例不同，它们中的大多数都是直接在对象实例上执行的。](/web/20220627102551/https://www.baeldung.com/java-path)

**要获得绝对或规范路径**，我们可以使用:

```java
// java.io API
String absolutePathStr = file.getAbsolutePath();
String canonicalPathStr = file.getCanonicalPath();

// java.nio API
Path absolutePath = path.toAbsolutePath();
Path canonicalPath = path.toRealPath().normalize();
```

虽然`Path`对象是不可变的，但它返回一个新的实例。此外，NIO2 API 有`toRealPath()`和`normalize()`方法，我们可以用它们来消除冗余。

**到`URI`** 的转换可以通过使用`toUri()`方法来完成:

```java
URI fileUri = file.toURI();
URI pathUri = path.toUri();
```

同样，我们可以**列出目录内容**:

```java
// java.io API
String[] list = file.list();
File[] files = file.listFiles();

// java.nio API
DirectoryStream<Path> paths = Files.newDirectoryStream(path);
```

NIO2 API 返回自己的 [`DirectoryStream`](https://web.archive.org/web/20220627102551/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/DirectoryStream.html) 对象，实现了`Iterable`接口。

## 6.结论

从 Java 7 开始，开发人员现在可以在两个 API 之间选择来处理文件。在本文中，我们讨论了与`java.io.File`类相关的一些不同的缺点和问题。

为了解决这些问题，甲骨文决定推出 NIO 包，它带来了同样的功能，并有了巨大的改进。

然后，我们回顾了这两个 API。通过示例，我们了解了如何在它们之间迁移。我们还了解到， **`java.io.File`现在被视为遗产，不推荐用于新项目**。然而，没有计划弃用和删除它。

和往常一样，本文的所有代码片段都可以在 GitHub 上找到。