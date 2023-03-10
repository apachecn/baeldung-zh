# Java 中的压缩和解压缩

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compress-and-uncompress>

## 1。概述

在这个快速教程中，我们将学习如何将一个文件压缩到一个归档文件中，以及如何解压缩这个归档文件，所有这些都使用 Java 提供的核心库。

这些核心库是`**java.util.zip**`包的一部分，在这里我们可以找到所有与压缩和解压缩相关的工具。

## 2。压缩一个文件

首先，让我们来看一个简单的操作，压缩一个文件。

对于我们的例子，我们将把名为`test1.txt` 的文件压缩到名为`compressed.zip`的归档文件中。

当然，我们将首先从磁盘访问该文件:

```java
public class ZipFile {
    public static void main(String[] args) throws IOException {
        String sourceFile = "test1.txt";
        FileOutputStream fos = new FileOutputStream("compressed.zip");
        ZipOutputStream zipOut = new ZipOutputStream(fos);
        File fileToZip = new File(sourceFile);
        FileInputStream fis = new FileInputStream(fileToZip);
        ZipEntry zipEntry = new ZipEntry(fileToZip.getName());
        zipOut.putNextEntry(zipEntry);
        byte[] bytes = new byte[1024];
        int length;
        while((length = fis.read(bytes)) >= 0) {
            zipOut.write(bytes, 0, length);
        }
        zipOut.close();
        fis.close();
        fos.close();
    }
}
```

## 3。压缩多个文件

接下来，让我们看看如何将多个文件压缩成一个 zip 文件。我们将把`test1.txt`和`test2.txt`压缩成`multiCompressed.zip`:

```java
public class ZipMultipleFiles {
    public static void main(String[] args) throws IOException {
        List<String> srcFiles = Arrays.asList("test1.txt", "test2.txt");
        FileOutputStream fos = new FileOutputStream("multiCompressed.zip");
        ZipOutputStream zipOut = new ZipOutputStream(fos);
        for (String srcFile : srcFiles) {
            File fileToZip = new File(srcFile);
            FileInputStream fis = new FileInputStream(fileToZip);
            ZipEntry zipEntry = new ZipEntry(fileToZip.getName());
            zipOut.putNextEntry(zipEntry);

            byte[] bytes = new byte[1024];
            int length;
            while((length = fis.read(bytes)) >= 0) {
                zipOut.write(bytes, 0, length);
            }
            fis.close();
        }
        zipOut.close();
        fos.close();
    }
}
```

## 4。压缩一个目录

现在让我们讨论如何压缩整个目录。我们将把`zipTest`压缩成`dirCompressed.zip`:

```java
public class ZipDirectory {
    public static void main(String[] args) throws IOException {
        String sourceFile = "zipTest";
        FileOutputStream fos = new FileOutputStream("dirCompressed.zip");
        ZipOutputStream zipOut = new ZipOutputStream(fos);
        File fileToZip = new File(sourceFile);

        zipFile(fileToZip, fileToZip.getName(), zipOut);
        zipOut.close();
        fos.close();
    }

    private static void zipFile(File fileToZip, String fileName, ZipOutputStream zipOut) throws IOException {
        if (fileToZip.isHidden()) {
            return;
        }
        if (fileToZip.isDirectory()) {
            if (fileName.endsWith("/")) {
                zipOut.putNextEntry(new ZipEntry(fileName));
                zipOut.closeEntry();
            } else {
                zipOut.putNextEntry(new ZipEntry(fileName + "/"));
                zipOut.closeEntry();
            }
            File[] children = fileToZip.listFiles();
            for (File childFile : children) {
                zipFile(childFile, fileName + "/" + childFile.getName(), zipOut);
            }
            return;
        }
        FileInputStream fis = new FileInputStream(fileToZip);
        ZipEntry zipEntry = new ZipEntry(fileName);
        zipOut.putNextEntry(zipEntry);
        byte[] bytes = new byte[1024];
        int length;
        while ((length = fis.read(bytes)) >= 0) {
            zipOut.write(bytes, 0, length);
        }
        fis.close();
    }
}
```

请注意:

*   为了压缩子目录，我们递归地遍历它们。
*   每当我们找到一个目录时，我们就把它的名字附加到后代`ZipEntry`的名字上来保存层次结构。
*   我们还为每个空目录创建一个目录条目。

## 5。解压缩档案文件

现在让我们解压缩一个归档文件并提取其内容。

对于这个例子，我们将把`compressed.zip`解压到一个名为`unzipTest:`的新文件夹中

```java
public class UnzipFile {
    public static void main(String[] args) throws IOException {
        String fileZip = "src/main/resources/unzipTest/compressed.zip";
        File destDir = new File("src/main/resources/unzipTest");
        byte[] buffer = new byte[1024];
        ZipInputStream zis = new ZipInputStream(new FileInputStream(fileZip));
        ZipEntry zipEntry = zis.getNextEntry();
        while (zipEntry != null) {
           // ...
        }
        zis.closeEntry();
        zis.close();
    }
}
```

在`while`循环中，**我们将遍历每个`ZipEntry`，首先检查它是否是一个目录**。如果是，那么我们将使用`mkdirs()`方法创建目录；否则，我们将继续创建文件:

```java
while (zipEntry != null) {
     File newFile = newFile(destDir, zipEntry);
     if (zipEntry.isDirectory()) {
         if (!newFile.isDirectory() && !newFile.mkdirs()) {
             throw new IOException("Failed to create directory " + newFile);
         }
     } else {
         // fix for Windows-created archives
         File parent = newFile.getParentFile();
         if (!parent.isDirectory() && !parent.mkdirs()) {
             throw new IOException("Failed to create directory " + parent);
         }

         // write file content
         FileOutputStream fos = new FileOutputStream(newFile);
         int len;
         while ((len = zis.read(buffer)) > 0) {
             fos.write(buffer, 0, len);
         }
         fos.close();
     }
 zipEntry = zis.getNextEntry();
}
```

这里需要注意的是，在`else`分支上，我们还检查文件的父目录是否存在。这对于在 Windows 上创建的归档文件是必要的，因为根目录在 zip 文件中没有相应的条目。

在`newFile()`方法中可以看到另一个关键点:

```java
public static File newFile(File destinationDir, ZipEntry zipEntry) throws IOException {
    File destFile = new File(destinationDir, zipEntry.getName());

    String destDirPath = destinationDir.getCanonicalPath();
    String destFilePath = destFile.getCanonicalPath();

    if (!destFilePath.startsWith(destDirPath + File.separator)) {
        throw new IOException("Entry is outside of the target dir: " + zipEntry.getName());
    }

    return destFile;
}
```

这种方法可以防止将文件写入目标文件夹之外的文件系统。这个漏洞被称为 Zip Slip，我们可以[在这里](https://web.archive.org/web/20220817154022/https://snyk.io/research/zip-slip-vulnerability)了解更多信息。

## 6。结论

在本文中，我们展示了如何使用 Java 库来压缩和解压缩文件。

这些例子的实现可以在 GitHub 的[中找到。](https://web.archive.org/web/20220817154022/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)