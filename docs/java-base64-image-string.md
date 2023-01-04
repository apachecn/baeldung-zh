# 图像到 Base64 字符串转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-base64-image-string>

## 1。概述

在这个快速教程中，我们将介绍如何将图像文件编码为 Base64 `String`，然后使用 Apache Common IO 和 Java 8 原生 Base64 特性对其进行解码以检索原始图像。

这个操作可以应用于任何二进制文件或二进制数组。当我们需要将 JSON 格式的二进制内容从移动应用程序传输到 REST 端点时，这很有用。

有关 Base64 转换的更多信息，请点击此处查看本文。

## 2。Maven 依赖关系

让我们将以下依赖项添加到`pom.xml`文件中:

```
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20221129015308/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-io%22%20AND%20a%3A%22commons-io%22) 上找到最新版本的 Apache Commons IO。

## 3。将图像文件转换为 Base64 `String`

首先，我们把文件内容读入一个字节数组，用 Java 8 `Base64`类进行编码:

```
byte[] fileContent = FileUtils.readFileToByteArray(new File(filePath));
String encodedString = Base64.getEncoder().encodeToString(fileContent);
```

`encodedString`是`A-Za-z0-9+/`集合中的一个`String`字符，解码器拒绝该集合之外的任何字符。

## 4。将 Base64 `String`转换为图像文件

现在我们有了一个 Base64 `String`，让我们将其解码回二进制内容并写入一个新文件:

```
byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
FileUtils.writeByteArrayToFile(new File(outputFileName), decodedBytes);
```

## 5。测试我们的代码

最后，我们可以通过读取一个文件，将其编码为 Base64 `String`，然后解码回一个新文件来验证代码是否正常工作:

```
public class FileToBase64StringConversionUnitTest {

    private String inputFilePath = "test_image.jpg";
    private String outputFilePath = "test_image_copy.jpg";

    @Test
    public void fileToBase64StringConversion() throws IOException {
        // load file from /src/test/resources
        ClassLoader classLoader = getClass().getClassLoader();
        File inputFile = new File(classLoader
          .getResource(inputFilePath)
          .getFile());

        byte[] fileContent = FileUtils.readFileToByteArray(inputFile);
        String encodedString = Base64
          .getEncoder()
          .encodeToString(fileContent);

        // create output file
        File outputFile = new File(inputFile
          .getParentFile()
          .getAbsolutePath() + File.pathSeparator + outputFilePath);

        // decode the string and write to file
        byte[] decodedBytes = Base64
          .getDecoder()
          .decode(encodedString);
        FileUtils.writeByteArrayToFile(outputFile, decodedBytes);

        assertTrue(FileUtils.contentEquals(inputFile, outputFile));
    }
}
```

## 6。结论

这篇中肯的文章解释了使用 Apache Common IO 和 Java 8 特性将任何文件的内容编码为 Base64 `String`，将 Base64 `String`解码为字节数组并保存到文件的基本原理。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221129015308/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)