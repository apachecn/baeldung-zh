# 如何创建受密码保护的 Zip 文件并在 Java 中解压缩

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-password-protected-zip-unzip>

## 1.概观

在之前的教程中，我们展示了如何借助`java.util.zip`包在 Java 中[压缩和解压缩。但是我们没有任何标准的 Java 库来创建受密码保护的 zip 文件。](/web/20220625071042/https://www.baeldung.com/java-compress-and-uncompress)

在本教程中，我们将学习如何**创建受密码保护的 zip 文件，并用 [Zip4j](https://web.archive.org/web/20220625071042/https://github.com/srikanth-lingala/zip4j) 库解压它们**。这是针对 zip 文件的最全面的 Java 库。

## 2.属国

让我们首先将 [`zip4j`](https://web.archive.org/web/20220625071042/https://search.maven.org/search?q=g:%20net.lingala.zip4j%20a:zip4j) 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>net.lingala.zip4j</groupId>
    <artifactId>zip4j</artifactId>
    <version>2.9.0</version>
</dependency>
```

## 3.压缩文件

首先，**我们将使用 *ZipFile addFile()* 方法**将一个名为`aFile.txt`的文件压缩到一个名为`compressed.zip`的受密码保护的档案中:

```java
ZipParameters zipParameters = new ZipParameters();
zipParameters.setEncryptFiles(true);
zipParameters.setCompressionLevel(CompressionLevel.HIGHER);
zipParameters.setEncryptionMethod(EncryptionMethod.AES);

ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.addFile(new File("aFile.txt"), zipParameters);
```

`setCompressionLevel`行是可选的。我们可以选择从`FASTEST`到`ULTRA`(默认为`NORMAL`)。

在本例中，我们使用了 AES 加密。如果我们想使用 Zip 标准加密，我们只需将`AES`替换为`ZIP_STANDARD`。

注意**如果文件“aFile.txt”在磁盘上不存在，该方法会抛出一个异常:`“net.lingala.zip4j.exception.ZipException: File does not exist: …”`**

要解决这个问题，我们必须确保该文件是手动创建的并放在项目文件夹中，或者我们必须从 Java 创建它:

```java
File fileToAdd = new File("aFile.txt");
if (!fileToAdd.exists()) {
    fileToAdd.createNewFile();
}
```

同样，在我们完成新的`ZipFile`，**之后，关闭资源**是一个很好的实践

```java
zipFile.close(); 
```

## 4.压缩多个文件

让我们稍微修改一下代码，这样我们就可以一次压缩多个文件:

```java
ZipParameters zipParameters = new ZipParameters();
zipParameters.setEncryptFiles(true);
zipParameters.setEncryptionMethod(EncryptionMethod.AES);

List<File> filesToAdd = Arrays.asList(
  new File("aFile.txt"),
  new File("bFile.txt")
);

ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.addFiles(filesToAdd, zipParameters);
```

不使用`addFile`方法，**我们使用`addFiles()`并传入一个文件的`List`。**

## 5.压缩目录

我们可以简单地通过用`addFolder`替换`addFile`方法来压缩文件夹:

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.addFolder(new File("/users/folder_to_add"), zipParameters);
```

## 6.创建分割 Zip 文件

我们可以通过使用`createSplitZipFile`和`createSplitZipFileFromFolder`方法，在文件大小超过特定限制时**将 zip 文件分割成几个文件:**

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
int splitLength = 1024 * 1024 * 10; //10MB
zipFile.createSplitZipFile(Arrays.asList(new File("aFile.txt")), zipParameters, true, splitLength);
```

```java
zipFile.createSplitZipFileFromFolder(new File("/users/folder_to_add"), zipParameters, true, splitLength);
```

`splitLength`的单位是字节。

## 7.提取所有文件

提取文件也一样简单。我们可以用`extractAll()`方法从我们的`compressed.zip`中提取所有文件:

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.extractAll("/destination_directory");
```

## 8.提取单个文件

如果我们只想从`compressed.zip`中提取一个文件，我们可以使用`extractFile()`方法:

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.extractFile("aFile.txt", "/destination_directory");
```

## 9.结论

总之，我们已经学习了**如何创建受密码保护的 zip 文件，并使用 Zip4j 库在 Java 中解压它们**。

这些例子的实现可以在 GitHub 的[中找到。](https://web.archive.org/web/20220625071042/https://github.com/eugenp/tutorials/tree/master/libraries-io)