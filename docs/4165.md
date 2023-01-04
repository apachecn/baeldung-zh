# 用 Java 创建一个目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-create-directory>

## 1.概观

用 Java 创建一个目录非常简单。该语言为我们提供了两种方法，允许我们创建单个目录或多个嵌套目录—`mkdir()`和`mkdirs()`。

在本教程中，我们将了解它们的行为。

## 2.创建一个目录

让我们从创建一个目录开始。

出于我们的目的，我们将使用用户`temp`目录。我们可以用`System.getProperty(“java.io.tmpdir”)`查一下。

我们将把这个路径传递给一个 Java `File`对象，它将代表我们的临时目录:

```
private static final File TEMP_DIRECTORY = new File(System.getProperty("java.io.tmpdir"));
```

现在让我们在其中创建一个新目录。**我们将通过在一个新的`File`对象上调用`File::mkdir`方法来实现这一点，该对象表示要创建的目录:**

```
File newDirectory = new File(TEMP_DIRECTORY, "new_directory");
assertFalse(newDirectory.exists());
assertTrue(newDirectory.mkdir());
```

**为了确保我们的目录还不存在，我们首先使用了`exists()`方法。**

然后我们调用了`mkdir()`方法，它告诉我们目录创建是否成功。如果目录已经存在，该方法将返回`false`。

如果我们再打同样的电话:

```
assertTrue(newDirectory.exists());
assertFalse(newDirectory.mkdir());
```

然后，如我们所料，该方法在第二次调用时返回`false`。

**并且，`mkdir()`方法不仅在目录已经存在** [时返回`false`，在其他一些情况下也返回](https://web.archive.org/web/20221115015550/https://twitter.com/steveloughran/status/1087427627869261825)。例如，一个文件可能以我们想要创建的目录的名称存在。或者我们可能缺少创建该目录的权限。

考虑到这一点，我们必须找到一种方法来确保我们的目录最终存在，要么是我们创建的，要么是它已经存在了。为此，[我们可以使用`isDirectory()`方法](https://web.archive.org/web/20221115015550/https://twitter.com/steveloughran/status/1087428893882175490):

```
newDirectory.mkdir() || newDirectory.isDirectory()
```

这样，我们可以确保我们需要的目录在那里。

## 3.创建多个嵌套目录

到目前为止，我们所看到的在单个目录上运行良好，但是如果我们想要创建多个嵌套的目录，会发生什么呢？

在下面的例子中，我们将看到`File::mkdir`对此不起作用:

```
File newDirectory = new File(TEMP_DIRECTORY, "new_directory");
File nestedDirectory = new File(newDirectory, "nested_directory");
assertFalse(newDirectory.exists());
assertFalse(nestedDirectory.exists());
assertFalse(nestedDirectory.mkdir());
```

由于`new_directory`不存在，`mkdir` 不会创建底层的`nested_directory`。

**然而，`File`类为我们提供了另一种方法来实现这一点——`mkdirs()`。**这个方法将像`mkdir()`一样工作，但是也会创建所有不存在的父目录。

在我们之前的例子中，这意味着不仅要创建`nested_directory`，还要创建`new_directory.`

注意，到目前为止我们使用的是`File(File, String)`构造函数，但是**我们也可以使用`File(String)`构造函数，并使用`File.separator`** 来传递文件的完整路径，以分隔路径的不同部分:

```
File newDirectory = new File(System.getProperty("java.io.tmpdir") + File.separator + "new_directory");
File nestedDirectory = new File(newDirectory, "nested_directory");
assertFalse(newDirectory.exists());
assertFalse(nestedDirectory.exists());
assertTrue(nestedDirectories.mkdirs());
```

如我们所见，目录按预期创建。**此外，该方法仅在创建了至少一个目录时才返回`true`。至于`mkdir()`方法，它将在其他情况下返回`false`。**

因此，这意味着在存在父目录的目录上使用的`mkdirs()`方法将与`mkdir()`方法工作相同。

## 4.结论

在本文中，我们看到了两种允许我们用 Java 创建目录的方法。第一个是`mkdir()`，目标是创建一个目录，前提是它的父目录已经存在。第二个是`mkdirs()`，它能够创建一个目录及其不存在的父目录。

这篇文章的代码可以在我们的 GitHub 上找到[。](https://web.archive.org/web/20221115015550/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-2)