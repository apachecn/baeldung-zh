# NIO2 FileVisitor 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio2-file-visitor>

## 1。概述

在本文中，我们将探索 NIO2 的一个有趣特性——`FileVisitor`接口。

所有操作系统和一些第三方应用程序都有文件搜索功能，用户可以在其中定义搜索标准。

这个接口是我们在 Java 应用程序中实现这种功能所需要的。如果你需要搜索所有的`.mp3`文件，找到并删除`.class`文件或者找到所有上个月没有被访问过的文件，那么这个界面就是你需要的。

我们实现该功能所需的所有类都打包在一个包中:

```
import java.nio.file.*;
```

## 2。如何运作

使用`FileVisitor`接口，您可以遍历文件树到任何深度，并对在任何分支上找到的文件或目录执行任何操作。

`FileVisitor`接口的典型实现如下所示:

```
public class FileVisitorImpl implements FileVisitor<Path> {

    @Override
    public FileVisitResult preVisitDirectory(
      Path dir, BasicFileAttributes attrs) {
        return null;
    }

    @Override
    public FileVisitResult visitFile(
      Path file, BasicFileAttributes attrs) {
        return null;
    }

    @Override
    public FileVisitResult visitFileFailed(
      Path file, IOException exc) {       
        return null;
    }

    @Override
    public FileVisitResult postVisitDirectory(
      Path dir, IOException exc) {    
        return null;
    }
}
```

四个接口方法允许我们在遍历过程中的关键点指定所需的行为:在访问目录之前、访问文件时，或者分别在发生故障时和访问目录之后。

每个阶段的返回值都是类型`FileVisitResult`的，并且控制遍历的流程。也许你想遍历文件树寻找一个特定的目录，并在找到时终止进程，或者你想跳过特定的目录或文件。

`FileVisitResult`是`FileVisitor`接口方法的四个可能返回值的枚举:

*   **`FileVisitResult.CONTINUE`**–表示文件树遍历应该在返回它的方法退出后继续
*   **`FileVisitResult.TERMINATE`**–停止文件树遍历，不再访问任何目录或文件
*   **`FileVisitResult.SKIP_SUBTREE`**–这个结果只有在从`preVisitDirectory` API 返回时才有意义，在其他地方，它像`CONTINUE`一样工作。它表示应该跳过当前目录及其所有子目录
*   **`FileVisitResult.SKIP_SIBLINGS`**–表示遍历应该继续，而不访问当前文件或目录的同级。如果在`preVisitDirectory`阶段被调用，那么甚至当前目录都会被跳过，并且`postVisitDirectory`不会被调用

最后，必须有一种方法来触发遍历过程，也许是在定义了搜索条件之后，当用户从图形用户界面单击`search`按钮时。这是最简单的部分。

我们只需调用`Files`类的静态`walkFileTree` API，并向其传递一个代表遍历起点的`Path`类的实例，然后传递一个`FileVisitor`类的实例:

```
Path startingDir = Paths.get("pathToDir");
FileVisitorImpl visitor = new FileVisitorImpl();
Files.walkFileTree(startingDir, visitor);
```

## 3。文件搜索示例

在本节中，我们将使用`FileVisitor`接口实现一个文件搜索应用程序。我们希望让用户可以指定完整的文件名和扩展名以及要查找的起始目录。

当我们找到文件时，我们在屏幕上打印一条成功消息，当搜索了整个文件树而没有找到文件时，我们也打印一条适当的失败消息。

### 3.1。主类

我们将这个类称为`FileSearchExample.java`:

```
public class FileSearchExample implements FileVisitor<Path> {
    private String fileName;
    private Path startDir;

    // standard constructors
}
```

我们还没有实现接口方法。注意，我们已经创建了一个构造函数，它接受要搜索的文件名和开始搜索的路径。我们将仅使用起始路径作为基本情况来推断文件尚未找到。

在下面的小节中，我们将实现每个接口方法，并讨论它在这个特定的示例应用程序中的作用。

### 3.2。`preVisitDirectory` API

让我们从实现`preVisitDirectory` API 开始:

```
@Override
public FileVisitResult preVisitDirectory(
  Path dir, BasicFileAttributes attrs) {
    return CONTINUE;
}
```

如前所述，每当进程遇到树中的新目录时，都会调用这个 API。根据我们的决定，它的返回值决定了接下来会发生什么。在这一点上，我们将跳过特定的目录，并从搜索样本空间中删除它们。

让我们选择不歧视任何目录，只在所有的目录中搜索。

### 3.3。`visitFile` API

接下来，我们将实现`visitFile` API。这是主要活动发生的地方。每次遇到文件时都会调用这个 API。我们利用这一点来检查文件属性，并与我们的标准进行比较，然后返回适当的结果:

```
@Override
public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
    String fileName = file.getFileName().toString();
    if (FILE_NAME.equals(fileName)) {
        System.out.println("File found: " + file.toString());
        return TERMINATE;
    }
    return CONTINUE;
}
```

在我们的例子中，我们只检查被访问文件的名称，以了解它是否是用户正在搜索的文件。如果名称匹配，我们打印一条成功消息并终止该过程。

然而，这里有很多事情可以做，尤其是在阅读了`File Attributes`部分之后。您可以检查创建时间、最后修改时间或最后访问时间或几个在`attrs`参数中可用的属性，并据此做出决定。

### 3.4。`visitFileFailed` API

接下来，我们将实现`visitFileFailed` API。当 JVM 无法访问某个特定文件时，就会调用这个 API。可能它已被另一个应用程序锁定，或者可能只是权限问题:

```
@Override
public FileVisitResult visitFileFailed(Path file, IOException exc) {
    System.out.println("Failed to access file: " + file.toString());
    return CONTINUE;
}
```

我们简单地记录一条失败消息，并继续遍历目录树的其余部分。在图形应用程序中，您可以选择询问用户是否继续使用对话框，或者只是将消息记录在某个地方，然后编写一份报告供以后使用。

### 3.5。`postVisitDirectory` API

最后，我们将实现`postVisitDirectory` API。每次完全遍历目录时，都会调用此 API:

```
@Override
public FileVisitResult postVisitDirectory(Path dir, IOException exc){
    boolean finishedSearch = Files.isSameFile(dir, START_DIR);
    if (finishedSearch) {
        System.out.println("File:" + FILE_NAME + " not found");
        return TERMINATE;
    }
    return CONTINUE;
}
```

我们使用`Files.isSameFile` API 来检查刚刚被遍历的目录是否是我们开始遍历的目录。如果返回值是`true`，那意味着搜索已经完成，文件还没有找到。因此，我们用一条失败消息来终止这个过程。

然而，如果返回值是`false`，这意味着我们刚刚遍历完一个子目录，仍然有可能在其他子目录中找到该文件。所以我们继续遍历。

我们现在可以添加我们的主方法来执行`FileSearchExample`应用程序:

```
public static void main(String[] args) {
    Path startingDir = Paths.get("C:/Users/user/Desktop");
    String fileToSearch = "hibernate-guide.txt"
    FileSearchExample crawler = new FileSearchExample(
      fileToSearch, startingDir);
    Files.walkFileTree(startingDir, crawler);
}
```

您可以通过改变`startingDir`和`fileToSearch`变量的值来试验这个例子。当`fileToSearch`存在于`startingDir`或其任何子目录中时，你会得到一个成功信息，否则，会得到一个失败信息。

## 4。结论

在本文中，我们探索了 Java 7 NIO.2 文件系统 API 中一些不常用的特性，尤其是`FileVisitor`接口。我们还设法完成了构建一个文件搜索应用程序来演示其功能的步骤。

本文中使用的示例的完整源代码可以在 [Github 项目](https://web.archive.org/web/20221012141740/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)中获得。