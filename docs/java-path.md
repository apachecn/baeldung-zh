# 比较 Java 中的 getPath()、getAbsolutePath()和 getCanonicalPath()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-path>

## 1。概述

`java.io.File`类有三种方法——`getPath()`、`getAbsolutePath()`和`getCanonicalPath()`——来获取文件系统路径。

在本文中，我们将快速查看它们之间的差异，并讨论一个用例，在这个用例中，您可以选择使用其中一个而不是其他。

## 2。方法定义和示例

让我们先来看一下这三种方法的定义，以及基于用户主目录中存在以下目录结构的示例:

```java
|-- baeldung
    |-- baeldung.txt
    |-- foo
    |   |-- foo-one.txt
    |   \-- foo-two.txt
    \-- bar
        |-- bar-one.txt
        |-- bar-two.txt
        \-- baz
            |-- baz-one.txt
            \-- baz-two.txt
```

### 2.1。`getPath()`

简单地说，`getPath()`返回文件抽象路径名的`String`表示。这实际上是**传递给`File`构造函数**的路径名。

因此，如果`File`对象是使用相对路径创建的，那么从`getPath()`方法返回的值也将是相对路径。

如果我们从`{user.home}/baeldung`目录中调用以下代码:

```java
File file = new File("foo/foo-one.txt");
String path = file.getPath();
```

`path`变量的值为:

```java
foo/foo-one.txt  // on Unix systems
foo\foo-one.txt  // on Windows systems
```

请注意，对于 Windows 系统，名称分隔符已从传递给构造函数的正斜杠(/)字符更改为反斜杠(\)字符。这是因为**返回的`String`总是使用平台的默认名称分隔符**。

### 2.2。`getAbsolutePath()`

`getAbsolutePath()`方法在解析当前用户目录的路径后返回**文件的路径名——这称为绝对路径名。因此，对于我们之前的示例，`file.getAbsolutePath()`将返回:**

```java
/home/username/baeldung/foo/foo-one.txt     // on Unix systems
C:\Users\username\baeldung\foo\foo-one.txt  // on Windows systems
```

此方法仅解析相对路径的当前目录。速记表示(如"`.”`和"`..”`)不会被进一步解析。因此，当我们从`{user.home}/baeldung:`目录中执行下面的代码时

```java
File file = new File("bar/baz/../bar-one.txt");
String path = file.getAbsolutePath();
```

变量`path`的值将是:

```java
/home/username/baeldung/bar/baz/../bar-one.txt      // on Unix systems
C:\Users\username\baeldung\bar\baz\..\bar-one.txt   // on Windows systems
```

### 2.3。`getCanonicalPath()`

`getCanonicalPath()`方法更进一步，**根据目录结构解析绝对路径名以及缩写或冗余名称，如“`.`”和“`..`”**。它还**在 Unix 系统上解析符号链接**，在 Windows 系统上**将驱动器号转换成标准大小写**。

因此对于前面的例子，`getCanonicalPath()`方法将返回:

```java
/home/username/baeldung/bar/bar-one.txt     // on Unix systems
C:\Users\username\baeldung\bar\bar-one.txt  // on Windows systems
```

我们再举一个例子。给定当前目录为使用参数`new File(“bar/baz/./baz-one.txt”)`创建的`${user.home}/baeldung`和`File`对象，`getCanonicalPath()`的输出将是:

```java
/home/username/baeldung/bar/baz/baz-one.txt     // on Unix systems
C:\Users\username\baeldung\bar\baz\baz-one.txt  // on Windows Systems
```

值得一提的是，文件系统中的单个文件可以有无限多的绝对路径，因为有无限多种方式可以使用简写表示。然而，**规范路径将总是唯一的**，因为所有这样的表示都被解析。

与后两种方法不同，`getCanonicalPath()`可能会抛出`IOException`,因为它需要文件系统查询。

例如，在 Windows 系统上，如果我们用一个非法字符创建一个`File`对象，解析规范路径将抛出一个`IOException`:

```java
new File("*").getCanonicalPath();
```

## 3。用例

假设我们正在编写一个方法，它接受一个`File`对象作为参数，并将它的[完全限定名](https://web.archive.org/web/20220626202045/https://en.wikipedia.org/wiki/Fully_qualified_name#Filenames_and_paths)保存到数据库中。我们不知道该路径是相对的还是包含人手不足。在这种情况下，我们可能要使用`getCanonicalPath()`。

然而，因为`getCanonicalPath()`读取文件系统，所以会有性能损失。如果我们确定没有多余的名称或符号链接，并且驱动器号大小写是标准化的(如果使用 Windows 操作系统)，那么我们应该更喜欢使用`getAbsoultePath()`。

## 4。结论

在这个快速教程中，我们讨论了获取文件系统路径的三种`File`方法之间的区别。我们还展示了一个用例，其中一种方法可能优于另一种方法。

演示本文示例的测试类可以在 GitHub 的[中找到。](https://web.archive.org/web/20220626202045/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)