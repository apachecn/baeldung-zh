# Java 文件类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-io-file>

## 1.概观

在本教程中，我们将给出`File` 类的概述，它是`java.io` API 的一部分。**`File`类给了我们在文件系统**上处理文件和目录的能力。

## 2.创建一个`File`对象

`File `类有 4 个公共构造函数。根据开发人员的需要，可以创建不同类型的`File` 类实例。

*   `File(String pathname)` –创建代表给定`pathname`的实例
*   `File(String parent, String child)` –创建一个实例，该实例表示通过连接`parent`和`child`路径形成的路径
*   `File(File parent, String child)` –创建一个实例，其路径由另一个`File `实例代表的`parent`路径和`child`路径结合而成
*   `File(URI uri)`–创建代表给定统一资源标识符的实例

## 3.使用`File`类

`File`类有许多方法允许我们在文件系统上操作文件。我们将在此重点介绍其中一些。**需要注意的是`File`类不能修改或访问它所代表的文件的内容。**

### 3.1.创建和删除目录和文件

`File`类有创建和删除目录和文件的实例方法。目录和文件是分别使用`mkdir`和`createNewFile`方法创建的**。**

使用`delete`方法删除目录和文件**。所有这些方法都返回一个`boolean`值，当操作成功时为`true`，否则为`false`:**

```java
@Test
public void givenDir_whenMkdir_thenDirIsDeleted() {
    File directory = new File("dir");
    assertTrue(directory.mkdir());
    assertTrue(directory.delete());
}

@Test
public void givenFile_whenCreateNewFile_thenFileIsDeleted() {
    File file = new File("file.txt");
    try {
        assertTrue(file.createNewFile());
    } catch (IOException e) {
        fail("Could not create " + "file.txt");
    }
    assertTrue(file.delete());
}
```

在上面的片段中，我们还看到了**其他有用的方法**。

**`isDirectory`方法**可用于测试提供的名称所表示的文件是否为目录，而**方法`isFile`可用于测试提供的名称所表示的文件是否为文件。并且，我们可以使用**`exists `方法**来测试一个目录或文件是否已经存在于系统中。**

### 3.2.获取关于文件实例的元数据

`File `类有许多返回关于`File `实例的元数据的方法。让我们看看如何使用 **`getName, getParentFile,` 和`getPath`方法**:

```java
@Test
public void givenFile_whenCreateNewFile_thenMetadataIsCorrect() {

    String sep = File.separator;

    File parentDir = makeDir("filesDir");

    File child = new File(parentDir, "file.txt");
    try {
        child.createNewFile();
    } catch (IOException e) {
        fail("Could not create " + "file.txt");
    }

    assertEquals("file.txt", child.getName());
    assertEquals(parentDir.getName(), child.getParentFile().getName());
    assertEquals(parentDir.getPath() + sep + "file.txt", child.getPath());

    removeDir(parentDir);
}
```

这里，我们已经说明了如何验证在目录中创建的文件的元数据。我们还展示了如何找到文件的父文件以及该文件的相对路径。

### 3.3.设置文件和目录权限

`File `类有允许你设置文件或目录权限的方法。在这里，我们先来看看 **`setWritable`和`setReadable` 的方法**:

```java
@Test
public void givenReadOnlyFile_whenCreateNewFile_thenCantModFile() {
    File parentDir = makeDir("readDir");

    File child = new File(parentDir, "file.txt");
    try {
        child.createNewFile();
    } catch (IOException e) {
        fail("Could not create " + "file.txt");
    }
    child.setWritable(false);
    boolean writable = true;
    try (FileOutputStream fos = new FileOutputStream(child)) {
        fos.write("Hello World".getBytes()); // write operation
        fos.flush();
    } catch (IOException e) {
        writable = false;
    } finally {
        removeDir(parentDir);
    }
    assertFalse(writable);
}
```

在上面的代码中，我们在显式设置了阻止任何写入的权限后，尝试写入文件。我们用`setWritable` 方法来做这件事。**在不允许写入文件时试图写入文件会导致抛出`IOException` 。**

接下来，在对文件设置了阻止任何读取的权限后，我们尝试读取文件。**使用`setReadable`方法阻止读取:**

```java
@Test
public void givenWriteOnlyFile_whenCreateNewFile_thenCantReadFile() {
    File parentDir = makeDir("writeDir");

    File child = new File(parentDir, "file.txt");
    try {
        child.createNewFile();
    } catch (IOException e) {
        fail("Could not create " + "file.txt");
    }
    child.setReadable(false);
    boolean readable = true;
    try (FileInputStream fis = new FileInputStream(child)) {
        fis.read(); // read operation
    } catch (IOException e) {
        readable = false;
    } finally {
        removeDir(parentDir);
    }
    assertFalse(readable);
}
```

同样，**JVM 将抛出一个`IOException` 来尝试读取一个不允许读取的文件**。

### 3.4.列出目录中的文件

`File `类的方法允许我们列出目录中包含的文件。同样，也可以列出目录。这里我们先来看看 **`list`和`list(FilenameFilter)`的方法**:

```java
@Test
public void givenFilesInDir_whenCreateNewFile_thenCanListFiles() {
    File parentDir = makeDir("filtersDir");

    String[] files = {"file1.csv", "file2.txt"};
    for (String file : files) {
        try {
            new File(parentDir, file).createNewFile();
        } catch (IOException e) {
            fail("Could not create " + file);
        }
    }

    //normal listing
    assertEquals(2, parentDir.list().length);

    //filtered listing
    FilenameFilter csvFilter = (dir, ext) -> ext.endsWith(".csv");
    assertEquals(1, parentDir.list(csvFilter).length);

    removeDir(parentDir);
}
```

我们创建了一个目录，并向其中添加了两个文件——一个扩展名为`csv`,另一个扩展名为`txt`。当列出目录中的所有文件时，我们得到了预期的两个文件。当我们通过过滤扩展名为`csv`的文件来过滤列表时，我们只得到一个返回的文件。

### 3.5.重命名文件和目录

`File`类具有使用`renameTo`方法重命名文件和目录**的功能:**

```java
@Test
public void givenDir_whenMkdir_thenCanRenameDir() {

    File source = makeDir("source");
    File destination = makeDir("destination");
    boolean renamed = source.renameTo(destination);

    if (renamed) {
        assertFalse(source.isDirectory());
        assertTrue(destination.isDirectory());

        removeDir(destination);
    }
}
```

在上面的示例中，我们创建了两个目录—源目录和目标目录。然后我们使用`renameTo`方法将源目录重命名为目标目录。同样可以用来重命名文件而不是目录。

### 3.6.获取磁盘空间信息

`File `类还允许我们获取磁盘空间信息。让我们来看一下`getFreeSpace`方法的**演示:**

```java
@Test
public void givenDataWritten_whenWrite_thenFreeSpaceReduces() {

    String home = System.getProperty("user.home");
    String sep = File.separator;
    File testDir = makeDir(home + sep + "test");
    File sample = new File(testDir, "sample.txt");

    long freeSpaceBefore = testDir.getFreeSpace();
    try {
        writeSampleDataToFile(sample);
    } catch (IOException e) {
        fail("Could not write to " + "sample.txt");
    }

    long freeSpaceAfter = testDir.getFreeSpace();
    assertTrue(freeSpaceAfter < freeSpaceBefore);

    removeDir(testDir);
}
```

在本例中，我们在用户的主目录中创建了一个目录，然后在其中创建了一个文件。然后，我们检查在用一些文本填充这个文件之后，主目录分区上的可用空间是否发生了变化。**其他给出磁盘空间信息的方法有`getTotalSpace`和** `**getUsableSpace**` **。**

## 4.结论

在本教程中，我们已经展示了`File`类为处理文件系统中的文件和目录提供的一些功能。。

和往常一样，这个例子的完整源代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221127062543/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)