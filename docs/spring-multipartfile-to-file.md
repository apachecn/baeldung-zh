# 将 Spring 多文件转换为文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-multipartfile-to-file>

## 1.概观

在这个快速教程中，我们将介绍**将弹簧 [`MultipartFile`](https://web.archive.org/web/20221208143841/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/multipart/MultipartFile.html) 转换为 [`File`](/web/20221208143841/https://www.baeldung.com/java-io-file)** 的各种方法。

## 2.`MultipartFile#getBytes`

**`MultipartFile`有一个`getBytes()`方法**返回文件内容的字节数组。我们可以用这个方法**把字节写到一个文件**:

```java
MultipartFile multipartFile = new MockMultipartFile("sourceFile.tmp", "Hello World".getBytes());

File file = new File("src/main/resources/targetFile.tmp");

try (OutputStream os = new FileOutputStream(file)) {
    os.write(multipartFile.getBytes());
}

assertThat(FileUtils.readFileToString(new File("src/main/resources/targetFile.tmp"), "UTF-8"))
  .isEqualTo("Hello World");
```

对于我们希望在写入磁盘之前对文件执行额外操作的实例**，方法`getBytes()`非常有用，比如计算文件散列。**

## 3.`MultipartFile#getInputStream`

接下来我们来看看 **`MultipartFile`的`getInputStream()`法**:

```java
MultipartFile multipartFile = new MockMultipartFile("sourceFile.tmp", "Hello World".getBytes());

InputStream initialStream = multipartFile.getInputStream();
byte[] buffer = new byte[initialStream.available()];
initialStream.read(buffer);

File targetFile = new File("src/main/resources/targetFile.tmp");

try (OutputStream outStream = new FileOutputStream(targetFile)) {
    outStream.write(buffer);
}

assertThat(FileUtils.readFileToString(new File("src/main/resources/targetFile.tmp"), "UTF-8"))
  .isEqualTo("Hello World");
```

这里我们使用`getInputStream()`方法来获取`InputStream`，从`InputStream,`中读取字节，并将它们存储在`byte[] buffer`中。然后我们创建一个`File`和`OutputStream`来写`buffer`的内容。

在**中，我们需要将`InputStream`包装在另一个** `**InputStream**,`中，比如说一个`GZipInputStream`中，如果上传的文件是 gzip 文件，那么`getInputStream()`方法是很有用的。

## 4.`MultipartFile#transferTo`

最后，我们来看看 **`MultipartFile`的`transferTo()`法**:

```java
MultipartFile multipartFile = new MockMultipartFile("sourceFile.tmp", "Hello World".getBytes());

File file = new File("src/main/resources/targetFile.tmp");

multipartFile.transferTo(file);

assertThat(FileUtils.readFileToString(new File("src/main/resources/targetFile.tmp"), "UTF-8"))
  .isEqualTo("Hello World");
```

使用`transferTo()`方法，我们只需创建想要写入字节的`File`，然后将该文件传递给`transferTo()`方法。

当`MultipartFile`只需要写入一个`File`时，`transferTo()`方法对**很有用。**

## 5.结论

在本教程中，我们探索了将弹簧`MultipartFile`转换为`File`的方法。

像往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java-2)