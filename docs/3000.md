# Apache commons me(Apache 公用程式)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-io>

## 1。概述

创建 Apache Commons 项目是为了向开发人员提供一组他们可以在日常代码中使用的公共库。

在本教程中，我们将探索 Commons IO 模块的一些关键实用程序类及其最著名的功能。

## 2。Maven 依赖关系

为了使用这个库，让我们在`pom.xml`中包含以下 Maven 依赖项:

```
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

该库的最新版本可以在 [Maven Central](https://web.archive.org/web/20220626120155/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22) 中找到。

## 3。实用程序类别

简单地说，实用程序类提供了一组静态方法，这些方法可以用来对文件执行常见的任务。

### 3.1。`FileUtils`

这个类提供了对文件的不同操作，比如打开、读取、复制和移动。

让我们看看**如何使用`FileUtils`读取或复制文件**:

```
File file = FileUtils.getFile(getClass().getClassLoader()
  .getResource("fileTest.txt")
  .getPath());
File tempDir = FileUtils.getTempDirectory();
FileUtils.copyFileToDirectory(file, tempDir);
File newTempFile = FileUtils.getFile(tempDir, file.getName());
String data = FileUtils.readFileToString(newTempFile,
  Charset.defaultCharset());
```

### 3.2。`FilenameUtils`

这个实用程序提供了一种**操作系统不可知的方式来执行文件名**上的公共函数。让我们看看我们可以利用的一些不同方法:

```
String fullPath = FilenameUtils.getFullPath(path);
String extension = FilenameUtils.getExtension(path);
String baseName = FilenameUtils.getBaseName(path);
```

### 3.3。`FileSystemUtils`

我们可以使用`FileSystemUtils`到**检查给定卷或驱动器上的可用空间**:

```
long freeSpace = FileSystemUtils.freeSpaceKb("/");
```

## 4。输入和输出

这个包为使用输入和输出流的**提供了几个实现。**

我们将重点关注`TeeInputStream`和`TeeOutputSteam`。单词“`Tee`”(来源于字母“`T`”)通常用于描述一个输入将被分成两个不同的输出。

让我们看一个例子，演示我们如何将一个输入流写入两个不同的输出流:

```
String str = "Hello World.";
ByteArrayInputStream inputStream = new ByteArrayInputStream(str.getBytes());
ByteArrayOutputStream outputStream1 = new ByteArrayOutputStream();
ByteArrayOutputStream outputStream2 = new ByteArrayOutputStream();

FilterOutputStream teeOutputStream
  = new TeeOutputStream(outputStream1, outputStream2);
new TeeInputStream(inputStream, teeOutputStream, true)
  .read(new byte[str.length()]);

assertEquals(str, String.valueOf(outputStream1));
assertEquals(str, String.valueOf(outputStream2));
```

## 5。过滤器

Commons IO 包括一系列有用的文件过滤器。当开发人员想要从一个异构的文件列表中**缩小到一个特定的想要的文件列表**时，这些可以派上用场。

该库还支持对给定文件列表的`AND`和`OR`逻辑操作。因此，我们可以混合搭配这些滤镜来获得想要的效果。

让我们来看一个例子，它使用`WildcardFileFilter`和`SuffixFileFilter` 来检索名称中带有“`ple`”并带有“`txt`”后缀的文件。注意，我们使用`ANDFileFilter`来包装上面的过滤器:

```
@Test
public void whenGetFilewith_ANDFileFilter_thenFind_sample_txt()
  throws IOException {

    String path = getClass().getClassLoader()
      .getResource("fileTest.txt")
      .getPath();
    File dir = FileUtils.getFile(FilenameUtils.getFullPath(path));

    assertEquals("sample.txt",
      dir.list(new AndFileFilter(
        new WildcardFileFilter("*ple*", IOCase.INSENSITIVE),
        new SuffixFileFilter("txt")))[0]);
}
```

## 6。比较器

`Comparator`包为**文件**提供了不同类型的比较。这里我们将探讨两种不同的比较器。

### 6.1。`PathFileComparator`

`PathFileComparator`类可用于按照路径对文件列表或数组进行排序，可以区分大小写，也可以不区分大小写，或者根据系统区分大小写。让我们看看如何使用这个实用程序对资源目录中的文件路径进行排序:

```
@Test
public void whenSortDirWithPathFileComparator_thenFirstFile_aaatxt() 
  throws IOException {

    PathFileComparator pathFileComparator = new PathFileComparator(
      IOCase.INSENSITIVE);
    String path = FilenameUtils.getFullPath(getClass()
      .getClassLoader()
      .getResource("fileTest.txt")
      .getPath());
    File dir = new File(path);
    File[] files = dir.listFiles();

    pathFileComparator.sort(files);

    assertEquals("aaa.txt", files[0].getName());
}
```

注意，我们已经使用了`IOCase.INSENSITIVE`配置。`PathFileComparator` 还提供了许多**单例实例，它们具有不同的区分大小写和反向排序选项**。

这些静态字段包括`PATH_COMPARATOR, PATH_INSENSITIVE_COMPARATOR, PATH_INSENSITIVE_REVERSE, PATH_SYSTEM_COMPARATOR,` 等等。

### 6.2。`SizeFileComparator`

`SizeFileComparator`顾名思义是用来**比较两个文件**的大小(长度)的。如果第一个文件的大小小于第二个文件的大小，它将返回负整数值。如果文件大小相等，则返回零，如果第一个文件的大小大于第二个文件的大小，则返回正值。

让我们编写一个单元测试来演示文件大小的比较:

```
@Test
public void whenSizeFileComparator_thenLargerFile_large()
  throws IOException {

    SizeFileComparator sizeFileComparator = new SizeFileComparator();
    File largerFile = FileUtils.getFile(getClass().getClassLoader()
      .getResource("fileTest.txt")
      .getPath());
    File smallerFile = FileUtils.getFile(getClass().getClassLoader()
      .getResource("sample.txt")
      .getPath());

    int i = sizeFileComparator.compare(largerFile, smallerFile);

    Assert.assertTrue(i > 0);
}
```

## 7。文件监视器

Commons IO monitor 包提供了**跟踪文件或目录**变化的能力。让我们看一个简单的例子，看看如何将`FileAlterationMonitor`与`FileAlterationObserver`和`FileAlterationListener` 一起使用来监控一个文件或文件夹。

当`FileAlterationMonitor` 启动时，我们将开始接收正在被监控的目录上的文件更改通知 *:*

```
FileAlterationObserver observer = new FileAlterationObserver(folder);
FileAlterationMonitor monitor = new FileAlterationMonitor(5000);

FileAlterationListener fal = new FileAlterationListenerAdaptor() {

    @Override
    public void onFileCreate(File file) {
        // on create action
    }

    @Override
    public void onFileDelete(File file) {
        // on delete action
    }
};

observer.addListener(fal);
monitor.addObserver(observer);
monitor.start();
```

## 8。结论

本文介绍了 Commons IO 包的一些常用组件。然而，该软件包也提供了许多其他功能。更多详情请参考 [API 文档](https://web.archive.org/web/20220626120155/https://commons.apache.org/proper/commons-io/javadocs/api-2.5/index.html)。

本例中使用的代码可以在 [GitHub 项目](https://web.archive.org/web/20220626120155/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-io)中找到。