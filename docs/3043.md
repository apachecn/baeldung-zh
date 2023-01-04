# 比较 Java 中两个文件的内容

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compare-files>

## 1.概观

在本教程中，我们将回顾不同的方法来确定两个文件的内容是否相等。我们将使用核心 Java 流 I/O 库来读取文件的内容并实现基本的比较。

最后，我们将回顾 Apache Commons I/O 中提供的支持，以检查两个文件的内容是否相等。

## 2.逐字节比较

让我们从一个**简单的方法开始，从两个文件中读取字节，然后按顺序比较它们**。

为了加快读取文件的速度，我们将使用`BufferedInputStream`。正如我们将看到的，`BufferedInputStream `从底层的`InputStream`读取大块的字节到内部缓冲区。当客户端读取块中的所有字节时，缓冲区从流中读取另一个字节块。

显然，**使用`BufferedInputStream`比从底层流**中一次读取一个字节要快得多。

让我们编写一个使用`BufferedInputStream` s 来比较两个文件的方法:

```
public static long filesCompareByByte(Path path1, Path path2) throws IOException {
    try (BufferedInputStream fis1 = new BufferedInputStream(new FileInputStream(path1.toFile()));
         BufferedInputStream fis2 = new BufferedInputStream(new FileInputStream(path2.toFile()))) {

        int ch = 0;
        long pos = 1;
        while ((ch = fis1.read()) != -1) {
            if (ch != fis2.read()) {
                return pos;
            }
            pos++;
        }
        if (fis2.read() == -1) {
            return -1;
        }
        else {
            return pos;
        }
    }
}
```

我们使用`try-with-resources`语句来确保两个`BufferedInputStream`在语句结束时关闭。

通过`while`循环，我们读取第一个文件的每个字节，并将其与第二个文件的相应字节进行比较。如果我们发现不一致，我们返回不匹配的字节位置。否则，文件是相同的，方法返回-1L。

我们可以看到，如果文件大小不同，但较小文件的字节与较大文件的相应字节相匹配，那么它将返回较小文件的字节大小。

## 3.逐行比较

**为了比较文本文件，我们可以实现逐行读取文件并检查它们之间的相等性**。

让我们使用与`InputStreamBuffer`使用相同策略的`BufferedReader`，将数据块从文件复制到内部缓冲区以加速读取过程。

让我们回顾一下我们的实现:

```
public static long filesCompareByLine(Path path1, Path path2) throws IOException {
    try (BufferedReader bf1 = Files.newBufferedReader(path1);
         BufferedReader bf2 = Files.newBufferedReader(path2)) {

        long lineNumber = 1;
        String line1 = "", line2 = "";
        while ((line1 = bf1.readLine()) != null) {
            line2 = bf2.readLine();
            if (line2 == null || !line1.equals(line2)) {
                return lineNumber;
            }
            lineNumber++;
        }
        if (bf2.readLine() == null) {
            return -1;
        }
        else {
            return lineNumber;
        }
    }
} 
```

该代码遵循与上一个示例类似的策略。在`while`循环中，我们不是读取字节，而是读取每个文件的一行并检查是否相等。如果两个文件的所有行都相同，那么我们返回-1L，但是如果有差异，我们返回找到第一个不匹配的行号。

如果文件大小不同，但是较小的文件与较大文件的相应行匹配，那么它返回较小文件的行数。

## 4.与`Files::mismatch`比较

**Java 12 中增加的方法`Files::mismatch`，比较两个文件**的内容。如果文件相同，它返回-1L，否则，它返回第一个不匹配的位置(以字节为单位)。

这个方法**从文件的`InputStream`内部读取数据块，并使用 Java 9 中引入的`Arrays::mismatch`来比较它们**。

与我们的第一个例子一样，对于大小不同但小文件的内容与大文件中的相应内容相同的文件，它返回较小文件的大小(以字节为单位)。

要查看如何使用这种方法的示例，请参阅我们的文章，其中涵盖了 Java 12 的[新特性。](/web/20221124044937/https://www.baeldung.com/java-12-new-features)

## 5.使用内存映射文件

内存映射文件是一个内核对象，它将磁盘文件中的字节映射到计算机的内存地址空间。堆内存被避开了，因为 Java 代码操纵内存映射文件的内容，就像我们直接访问内存一样。

对于大文件，从内存映射文件中读写数据比使用标准的 Java I/O 库要快得多。重要的是，计算机有足够的内存来处理作业，以防止系统颠簸。

让我们编写一个非常简单的示例，展示如何使用内存映射文件比较两个文件的内容:

```
public static boolean compareByMemoryMappedFiles(Path path1, Path path2) throws IOException {
    try (RandomAccessFile randomAccessFile1 = new RandomAccessFile(path1.toFile(), "r"); 
         RandomAccessFile randomAccessFile2 = new RandomAccessFile(path2.toFile(), "r")) {

        FileChannel ch1 = randomAccessFile1.getChannel();
        FileChannel ch2 = randomAccessFile2.getChannel();
        if (ch1.size() != ch2.size()) {
            return false;
        }
        long size = ch1.size();
        MappedByteBuffer m1 = ch1.map(FileChannel.MapMode.READ_ONLY, 0L, size);
        MappedByteBuffer m2 = ch2.map(FileChannel.MapMode.READ_ONLY, 0L, size);

        return m1.equals(m2);
    }
}
```

如果文件的内容相同，该方法返回`true`，否则返回`false`。

我们使用`RamdomAccessFile`类打开文件，并访问它们各自的`FileChannel`以获得`MappedByteBuffer`。这是一个直接字节缓冲区，是文件的内存映射区域。在这个简单的实现中，我们使用它的`equals`方法在内存中一次性比较整个文件的字节。

## 6.使用 Apache Commons I/O

**方法`IOUtils::contentEquals`和`IOUtils::contentEqualsIgnoreEOL`比较两个文件的内容，判断是否相等**。两者的区别在于 **`contentEqualsIgnoreEOL`忽略换行符(\n)和回车符(\r)** 。这样做的动机是因为操作系统使用这些控制字符的不同组合来定义一个新行。

让我们看一个简单的例子来检查等式:

```
@Test
public void whenFilesIdentical_thenReturnTrue() throws IOException {
    Path path1 = Files.createTempFile("file1Test", ".txt");
    Path path2 = Files.createTempFile("file2Test", ".txt");

    InputStream inputStream1 = new FileInputStream(path1.toFile());
    InputStream inputStream2 = new FileInputStream(path2.toFile());

    Files.writeString(path1, "testing line 1" + System.lineSeparator() + "line 2");
    Files.writeString(path2, "testing line 1" + System.lineSeparator() + "line 2");

    assertTrue(IOUtils.contentEquals(inputStream1, inputStream2));
} 
```

如果我们想忽略换行符，但要检查内容是否相等:

```
@Test
public void whenFilesIdenticalIgnoreEOF_thenReturnTrue() throws IOException {
    Path path1 = Files.createTempFile("file1Test", ".txt");
    Path path2 = Files.createTempFile("file2Test", ".txt");

    Files.writeString(path1, "testing line 1 \n line 2");
    Files.writeString(path2, "testing line 1 \r\n line 2");

    Reader reader1 = new BufferedReader(new FileReader(path1.toFile()));
    Reader reader2 = new BufferedReader(new FileReader(path2.toFile()));

    assertTrue(IOUtils.contentEqualsIgnoreEOL(reader1, reader2));
} 
```

## 7.结论

在本文中，我们介绍了几种方法来实现两个文件内容的比较，以检查是否相等。

源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221124044937/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-12)