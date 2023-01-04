# Java 文件打开选项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-options>

## 1.概观

在本教程中，我们将关注 Java 中文件可用的标准打开选项。

我们将探索实现`OpenOption`接口并定义这些标准开放选项的`StandardOpenOption`枚举。

## 2.`OpenOption`参数

在 Java 中，我们可以使用 NIO2 API 来处理文件，它包含几个实用方法。其中一些方法使用可选的`OpenOption`参数来配置如何打开或创建文件。此外，如果没有设置，该参数将有一个默认值，对于这些方法中的每一个，该值可以是不同的。

`StandardOpenOption`枚举类型定义了标准选项并实现了`OpenOption`接口。

**下面是我们可以使用`StandardOpenOptions`枚举:**的支持选项列表

*   `WRITE`:打开文件进行写访问
*   `APPEND`:向文件追加一些数据
*   `TRUNCATE_EXISTING`:截断文件
*   `CREATE_NEW`:创建一个新文件，如果文件已经存在，抛出异常
*   `CREATE`:如果文件存在，则打开该文件；如果不存在，则创建一个新文件
*   `DELETE_ON_CLOSE`:关闭流后删除文件
*   `SPARSE`:新创建的文件将是稀疏的
*   `SYNC`:保持文件内容和元数据同步
*   `DSYNC`:仅保存同步文件的内容

在接下来的章节中，我们将看到如何使用这些选项的例子。

为了避免文件路径上的任何混乱，让我们获得用户主目录的句柄，这在所有操作系统中都是有效的:

```
private static String HOME = System.getProperty("user.home");
```

## 3.打开文件进行读写

首先，如果我们希望**创建一个新文件，如果它不存在，我们可以使用选项`CREATE`** :

```
@Test
public void givenExistingPath_whenCreateNewFile_thenCorrect() throws IOException {
    assertFalse(Files.exists(Paths.get(HOME, "newfile.txt")));
    Files.write(path, DUMMY_TEXT.getBytes(), StandardOpenOption.CREATE);
    assertTrue(Files.exists(path));
}
```

我们也可以使用选项 **`CREATE_NEW,` 来创建一个新文件，如果它不存在的话。** **但是，如果文件已经存在，就会抛出异常。**

其次，如果我们想让**打开文件进行读取，我们可以使用`newInputStream(Path, OpenOption.`** ..)方法。此方法打开文件进行读取，并返回输入流:

```
@Test
public void givenExistingPath_whenReadExistingFile_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, DUMMY_FILE_NAME);

    try (InputStream in = Files.newInputStream(path); BufferedReader reader = new BufferedReader(new InputStreamReader(in))) {
        String line;
        while ((line = reader.readLine()) != null) {
            assertThat(line, CoreMatchers.containsString(DUMMY_TEXT));
        }
    }
} 
```

注意**我们没有使用选项`READ`，因为它被方法`newInputStream`** 默认使用。

第三，**我们可以通过使用`newOutputStream(Path, OpenOption.`** 来创建一个文件，附加到一个文件，或者写入一个文件..)方法。这个方法打开或创建一个用于写的文件，并返回一个`OutputStream`。

如果我们不指定打开选项，API 将创建一个新文件，并且该文件不存在。但是，如果文件存在，它将被截断。这个选项类似于用`CREATE`和`TRUNCATE_EXISTING`选项调用方法。

让我们打开一个现有文件并添加一些数据:

```
@Test
public void givenExistingPath_whenWriteToExistingFile_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, DUMMY_FILE_NAME);

    try (OutputStream out = Files.newOutputStream(path, StandardOpenOption.APPEND, StandardOpenOption.WRITE)) {
        out.write(ANOTHER_DUMMY_TEXT.getBytes());
    }
}
```

## 4.创建一个`SPARSE`文件

我们可以告诉文件系统，新创建的文件应该是稀疏的(包含不会写入磁盘的空白空间的文件)。

为此，我们应该将选项`SPARSE`与选项`CREATE_NEW`一起使用。然而，**如果文件系统不支持稀疏文件**，该选项将被忽略。

让我们创建一个稀疏文件:

```
@Test
public void givenExistingPath_whenCreateSparseFile_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, "sparse.txt");
    Files.write(path, DUMMY_TEXT.getBytes(), StandardOpenOption.CREATE_NEW, StandardOpenOption.SPARSE);
}
```

## 5.保持文件同步

`StandardOpenOptions`枚举有`SYNC`和`DSYNC`选项。这些选项要求数据在存储中同步写入文件。换句话说，**这些将保证在系统崩溃的情况下数据不会丢失**。

让我们在文件中添加一些数据，并使用选项`SYNC`:

```
@Test
public void givenExistingPath_whenWriteAndSync_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, DUMMY_FILE_NAME);
    Files.write(path, ANOTHER_DUMMY_TEXT.getBytes(), StandardOpenOption.APPEND, StandardOpenOption.WRITE, StandardOpenOption.SYNC);
}
```

`SYNC`和`DSYNC`的区别在于 **`SYNC`** **在存储器中同步存储文件的内容和元数据**，而 **`DSYNC`在存储器中只同步存储文件的内容**。

## 6.关闭流后删除文件

enum 还提供了一个有用的选项，让我们能够在关闭流后销毁文件。如果我们想创建一个临时文件，这很有用。

让我们将一些数据添加到我们的文件中，并且**使用选项`DELETE_ON_CLOSE`** :

```
@Test
public void givenExistingPath_whenDeleteOnClose_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, EXISTING_FILE_NAME);
    assertTrue(Files.exists(path)); // file was already created and exists

    try (OutputStream out = Files.newOutputStream(path, StandardOpenOption.APPEND, 
      StandardOpenOption.WRITE, StandardOpenOption.DELETE_ON_CLOSE)) {
        out.write(ANOTHER_DUMMY_TEXT.getBytes());
    }

    assertFalse(Files.exists(path)); // file is deleted
}
```

## 7.结论

在本教程中，我们介绍了使用 Java 7 附带的新文件系统 API (NIO2)在 Java 中打开文件的可用选项。

像往常一样，教程中所有例子的源代码都可以在 Github 上找到[。](https://web.archive.org/web/20220524034652/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-3)