# NIO2 文件属性 API 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio2-file-attribute>

## 1。概述

在本文中，我们将探索 Java 7 NIO.2 文件系统 API 的高级特性之一——特别是文件属性 API。

如果你想先深入了解这些基础部分，我们之前已经介绍了 [`File`](/web/20220812054902/https://www.baeldung.com/java-nio-2-file-api) 和[`Path`API](/web/20220812054902/https://www.baeldung.com/java-nio-2-path)。

处理文件系统操作所需的所有文件都打包在`java.nio.file package`中:

```java
import java.nio.file.*;
```

## 2。档案基本属性

让我们从所有文件系统共有的基本属性的高级视图开始——由`BasicFileAttributeView`提供——它存储所有强制和可选的可见文件属性。

我们可以通过创建一个到 home 的路径并获取其基本属性视图来探索当前机器上用户 HOME 位置的基本属性:

```java
String HOME = System.getProperty("user.home");
Path home = Paths.get(HOME);
BasicFileAttributeView basicView = 
  Files.getFileAttributeView(home, BasicFileAttributeView.class);
```

完成上述步骤后，我们现在可以在一个批量操作中读取指向的路径的所有属性:

```java
BasicFileAttributes basicAttribs = basicView.readAttributes();
```

我们现在能够探索不同的公共属性，我们实际上可以在我们的应用程序中使用，特别是在条件语句中。

我们可以从文件的基本属性容器中查询文件的大小:

```java
@Test
public void givenPath_whenGetsFileSize_thenCorrect() {
    long size = basicAttribs.size();
    assertTrue(size > 0);
}
```

我们还可以检查它是否是一个目录:

```java
@Test
public void givenPath_whenChecksIfDirectory_thenCorrect() {
    boolean isDir = basicAttribs.isDirectory();
    assertTrue(isDir);
}
```

或者常规文件:

```java
@Test
public void givenPath_whenChecksIfFile_thenCorrect() {
    boolean isFile = basicAttribs.isRegularFile();
    assertFalse(isFile);
}
```

有了 Java NIO.2，我们现在能够处理文件系统中的符号链接或软链接。这些是我们通常称之为快捷方式的文件或目录。

要检查文件是否是符号链接:

```java
@Test
public void givenPath_whenChecksIfSymLink_thenCorrect() {
    boolean isSymLink = basicAttribs.isSymbolicLink();
    assertFalse(isSymLink);
}
```

在极少数情况下，我们可以调用`isOther` API 来检查文件是否属于常规文件、目录或符号链接的任何一个常见类别:

```java
@Test
public void givenPath_whenChecksIfOther_thenCorrect() {
    boolean isOther = basicAttribs.isOther();
    assertFalse(isOther);
}
```

要获取文件的创建时间:

```java
FileTime created = basicAttribs.creationTime();
```

要获取上次修改时间:

```java
FileTime modified = basicAttribs.lastModifiedTime();
```

要获得最后一次访问时间:

```java
FileTime accessed = basicAttribs.lastAccessTime();
```

以上所有例子都返回一个`FileTime`对象。这是一个比时间戳更有用的抽象。

例如，我们可以很容易地比较两个文件时间，以了解哪个事件在另一个之前或之后发生:

```java
@Test
public void givenFileTimes_whenComparesThem_ThenCorrect() {
    FileTime created = basicAttribs.creationTime();
    FileTime modified = basicAttribs.lastModifiedTime();
    FileTime accessed = basicAttribs.lastAccessTime();

    assertTrue(0 >= created.compareTo(accessed));
    assertTrue(0 <= modified.compareTo(created));
    assertTrue(0 == created.compareTo(created));
}
```

`compareTo` API 的工作方式与 Java 中的其他同类产品相同。如果被调用的对象小于参数，它将返回一个负值；在我们的例子中，和第一个断言一样，创建时间肯定在访问时间之前。

在第二个断言中，我们得到一个正整数值，因为修改只能在创建事件之后进行。最后，当比较的时间相等时，它返回 0。

当我们有一个 FileTime 对象时，我们可以根据需要将它转换成大多数其他单位；天、小时、分钟、秒、毫秒等等。我们通过调用适当的 API 来实现这一点:

```java
accessed.to(TimeUnit.SECONDS);
accessed.to(TimeUnit.HOURS);
accessed.toMillis();
```

我们也可以通过调用它的`toString` API 来打印人类可读形式的文件时间:

```java
accessed.toString();
```

它以 ISO 时间格式打印一些有用的东西:

```java
2016-11-24T07:52:53.376Z
```

我们还可以通过调用视图的 `setTimes(modified, accessed, created)` API 来改变视图的时间属性。我们在想要改变的地方传入新的`FileTime`对象，或者在不想改变的地方传入 null。

要将最后一次访问时间更改为未来一分钟，我们需要执行以下步骤:

```java
FileTime newAccessTime = FileTime.fromMillis(
  basicAttribs.lastAccessTime().toMillis() + 60000);
basicView.setTimes(null, newAccessTime , null);
```

从运行在机器上并使用文件系统的任何其他应用程序来看，这种更改将保留在实际文件中。

## 3。文件空间属性

当你在 Windows、Linux 或 Mac 上打开`my computer`时，你通常可以看到关于你的存储驱动器的空间信息的图形分析。

Java NIO.2 使得这种高级功能变得非常容易。它与底层文件系统交互来检索这些信息，而我们只需调用简单的 API。

我们可以使用`FileStore`类来检查存储驱动器，并获得重要信息，如它的大小、使用了多少空间以及还有多少空间未使用。

为了获得文件系统中任意文件位置的`FileStore`实例，我们使用了`Files`类的`getFileStore` API:

```java
Path file = Paths.get("file");
FileStore store = Files.getFileStore(file);
```

这个`FileStore`实例具体表示指定文件所在的文件存储，而不是文件本身。要获得总空间:

```java
long total = store.getTotalSpace();
```

要获得已用空间:

```java
long used = store.getTotalSpace() - store.getUnallocatedSpace();
```

我们不太可能遵循这种方法。

更常见的是，我们可能会获得关于所有文件存储的存储信息。为了在程序中模拟`my computer'`的图形驱动器空间信息，我们可以使用`FileSystem`类来枚举文件存储:

```java
Iterable<FileStore> fileStores = FileSystems.getDefault().getFileStores();
```

然后，我们可以对返回值进行循环，并对这些信息做我们需要做的任何事情，比如更新图形用户界面:

```java
for (FileStore fileStore : fileStores) {
    long totalSpace = fileStore.getTotalSpace();
    long unAllocated = fileStore.getUnallocatedSpace();
    long usable = fileStore.getUsableSpace();
}
```

注意，所有返回值都是以字节为单位的。我们可以转换成合适的单位，并使用基本算法计算其他信息，如已用空间。

**未分配空间和可用空间**的区别在于 JVM 的可访问性。

可用空间是 JVM 可用的空间，而未分配空间是底层文件系统可见的可用空间。因此，可用空间有时可能小于未分配的空间。

## 4。文件所有者属性

为了检查文件所有权信息，我们使用了`FileOwnerAttributeView`接口。它为我们提供了所有权信息的高级视图。

我们可以像这样创建一个`FileOwnerAttributeView` 对象:

```java
Path path = Paths.get(HOME);
FileOwnerAttributeView ownerView = Files.getFileAttributeView(
  attribPath, FileOwnerAttributeView.class);
```

从上面的视图中获取文件的所有者:

```java
UserPrincipal owner = ownerView.getOwner();
```

对于上面的对象，除了获取所有者的名字用于其他任意目的之外，我们真的没有什么可编程的:

```java
String ownerName = owner.toString();
```

## 5。用户定义的文件属性

有些情况下，文件系统中定义的文件属性不足以满足您的需求。如果您遇到这种情况，并且需要在文件上设置自己的属性，那么`UserDefinedFileAttributeView`接口将会派上用场:

```java
Path path = Paths.get("somefile");
UserDefinedFileAttributeView userDefView = Files.getFileAttributeView(
  attribPath, UserDefinedFileAttributeView.class);
```

要检索已经为上述视图表示的文件定义的用户定义属性列表:

```java
List<String> attribList = userDefView.list();
```

要在文件上设置用户定义的属性，我们使用以下习语:

```java
String name = "attrName";
String value = "attrValue";
userDefView.write(name, Charset.defaultCharset().encode(value));
```

当您需要访问用户定义的属性时，您可以循环查看视图返回的属性列表，并使用以下习语检查它们:

```java
ByteBuffer attrValue = ByteBuffer.allocate(userView.size(attrName));
userDefView.read(attribName, attribValue);
attrValue.flip();
String attrValue = Charset.defaultCharset().decode(attrValue).toString();
```

要从文件中删除用户定义的属性，我们只需调用视图的 delete API:

```java
userDefView.delete(attrName);
```

## 6。结论

在本文中，我们探讨了 Java 7 NIO.2 文件系统 API 中一些不常用的特性，特别是文件属性 API。

本文中使用的示例的完整源代码可以在 [Github 项目](https://web.archive.org/web/20220812054902/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio)中获得。