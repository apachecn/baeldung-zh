# 用 Java 在特定目录下创建一个文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-create-file-in-directory>

## 1。概述

在这个快速教程中，我们将看看如何在[的特定目录](/web/20221205154006/https://www.baeldung.com/java-create-directory)中[创建一个文件](/web/20221205154006/https://www.baeldung.com/java-how-to-create-a-file)。

我们将看到绝对和相对文件路径之间的区别，我们将使用在几种主要操作系统上工作的路径。

## 2。绝对和相对文件路径

### 2.1。绝对路径

让我们从在目录中创建一个文件开始，通过**引用整个路径**，也称为绝对路径。为了演示，我们将使用用户`temp`目录的绝对路径，并将我们的文件添加到其中。

我们使用谷歌番石榴的一部分`Files.touch()`，作为创建一个空文件的简单方法:

```
File tempDirectory = new File(System.getProperty("java.io.tmpdir"));
File fileWithAbsolutePath = new File(tempDirectory.getAbsolutePath() + "/testFile.txt");

assertFalse(fileWithAbsolutePath.exists());

Files.touch(fileWithAbsolutePath);

assertTrue(fileWithAbsolutePath.exists());
```

### 2.2。相对路径

我们还可以在相对于另一个目录的目录**中创建一个文件。例如，让我们在用户`temp`目录中创建文件:**

```
File tempDirectory = new File(System.getProperty("java.io.tmpdir"));
File fileWithRelativePath = new File(tempDirectory, "newFile.txt");

assertFalse(fileWithRelativePath.exists());

Files.touch(fileWithRelativePath);

assertTrue(fileWithRelativePath.exists());
```

在上面的例子中，我们的新文件被添加到用户路径的`temp`目录中。

## 3。使用独立于平台的文件分隔符

为了构建文件路径，我们需要使用像 **/** 或 **\** 这样的分隔符。然而，**要使用的合适分隔符取决于您的操作系统**。幸运的是，有一个更简单的方法。我们可以用 Java 的`File.separator`代替分隔符。因此，Java 为我们选择了合适的分隔符。

让我们看一个用这种方法创建文件的例子:

```
File tempDirectory = new File(System.getProperty("java.io.tmpdir"));
File newFile = new File(tempDirectory.getAbsolutePath() + File.separator + "newFile.txt");

assertFalse(newFile.exists());

Files.touch(newFile);

assertTrue(newFile.exists());
```

使用`File.separator`，Java 知道如何基于底层文件系统构建路径。

## 4。结论

在本文中，我们探讨了绝对路径和相对路径之间的区别，以及如何创建适用于几种主要操作系统的文件路径。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221205154006/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2)