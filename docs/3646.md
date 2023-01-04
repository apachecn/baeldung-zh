# 存储由数据库索引的文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-db-storing-files>

## 1.概观

当我们构建某种内容管理解决方案时，我们需要解决两个问题。我们需要一个地方来存储文件本身，我们需要某种数据库来索引它们。

可以将文件的内容存储在数据库本身中，或者我们可以将内容存储在其他地方，并用数据库对其进行索引。

在本文中，我们将用一个基本的图像归档应用程序来说明这两种方法。我们还将为上传和下载实现 REST APIs。

## 2.用例

我们的图像存档应用程序将允许我们**上传和下载 JPEG 图像**。

当我们上传图片时，应用程序将为它创建一个唯一的标识符。然后我们可以用这个标识符来下载它。

我们将使用一个关系数据库，用 [Spring Data JPA](/web/20221129124459/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 和 [Hibernate](/web/20221129124459/https://www.baeldung.com/tag/hibernate/) 。

## 3.数据库存储

让我们从我们的数据库开始。

### 3.1.图像实体

首先，让我们创建我们的`Image`实体:

```
@Entity
class Image {

    @Id
    @GeneratedValue
    Long id;

    @Lob
    byte[] content;

    String name;
    // Getters and Setters
}
```

`id`字段用`@GeneratedValue`标注。这意味着数据库将为我们添加的每条记录创建一个唯一的标识符。通过用这些值索引图像，我们不需要担心相同图像的多次上传会相互冲突。

其次，我们有了[冬眠`@Lob`注解](/web/20221129124459/https://www.baeldung.com/hibernate-lob)。这就是我们如何告诉 JPA 我们**打算存储一个潜在的大二进制文件**。

### 3.2.图像存储库

接下来，我们需要一个**存储库来连接数据库**。

我们就用春天 [`JpaRepository`](/web/20221129124459/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) :

```
@Repository
interface ImageDbRepository extends JpaRepository<Image, Long> {}
```

现在我们准备保存我们的图像。我们只需要一种方法将它们上传到我们的应用程序中。

### 4.休息控制器

我们将使用一个 [`MultipartFile`来上传](/web/20221129124459/https://www.baeldung.com/spring-file-upload)我们的图片。上传将返回我们稍后可以用来下载图像的`imageId`。

### 4.1.图像上传

让我们从创建支持上传的`ImageController` 开始:

```
@RestController
class ImageController {

    @Autowired
    ImageDbRepository imageDbRepository;

    @PostMapping
    Long uploadImage(@RequestParam MultipartFile multipartImage) throws Exception {
        Image dbImage = new Image();
        dbImage.setName(multipartImage.getName());
        dbImage.setContent(multipartImage.getBytes());

        return imageDbRepository.save(dbImage)
            .getId();
    }
}
```

`MultipartFile`对象包含文件的内容和原始名称。我们用它来构造我们的`Image`对象，并存储在数据库中。

该控制器返回生成的 id 作为其响应的主体。

### 4.2.图像下载

现在，让我们添加一个下载路径`:`

```
@GetMapping(value = "/image/{imageId}", produces = MediaType.IMAGE_JPEG_VALUE)
Resource downloadImage(@PathVariable Long imageId) {
    byte[] image = imageRepository.findById(imageId)
      .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND))
      .getContent();

    return new ByteArrayResource(image);
}
```

`imageId`路径变量包含上传时生成的 id。如果提供了一个无效的 id，那么我们将使用`ResponseStatusException`返回一个 HTTP 响应代码 404(未找到)。否则，我们将把存储的文件字节包装在一个`ByteArrayResource`中，这样就可以下载它们。

## 5.数据库映像存档测试

现在我们准备测试我们的图像存档。

首先，让我们构建我们的应用程序:

```
mvn package
```

其次，让我们开始吧:

```
java -jar target/image-archive-0.0.1-SNAPSHOT.jar
```

### 5.1.图像上传测试

在我们的应用程序运行之后，我们将**使用 [`curl`命令行工具](/web/20221129124459/https://www.baeldung.com/curl-rest)上传我们的图像**:

```
curl -H "Content-Type: multipart/form-data" \
  -F "[[email protected]](/web/20221129124459/https://www.baeldung.com/cdn-cgi/l/email-protection)" http://localhost:8080/image
```

由于上传服务**的响应是`imageId`** `,`，这是我们的第一个请求，输出将是:

```
1
```

### 5.2.图像下载测试

然后我们可以下载我们的图像:

```
curl -v http://localhost:8080/image/1 -o image.jpeg
```

`-o image.jpeg`选项将创建一个名为`image.jpeg`的文件，并将响应内容存储在其中:

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /image/1 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 
< Accept-Ranges: bytes
< Content-Type: image/jpeg
< Content-Length: 9291
```

我们得到了一个 HTTP/1.1 200，这意味着我们的下载是成功的。

我们也可以点击 [`http://localhost:8080/image/1`](https://web.archive.org/web/20221129124459/http://localhost:8080/image/1) 在浏览器中下载图片。

## 6.分离内容和位置

到目前为止，我们能够在数据库中上传和下载图像。

另一个好的选择是将文件内容上传到不同的位置。然后我们**在 DB** 中只保存 **的文件系统`location`。**

为此，我们需要向我们的`Image`实体添加一个新字段:

```
String location;
```

这将包含某个外部存储器中文件的逻辑路径。在我们的例子中，**它将是服务器文件系统上的路径。**

然而，我们同样可以将这种想法应用于不同的商店。例如，我们可以使用云存储-[谷歌云存储](/web/20221129124459/https://www.baeldung.com/java-google-cloud-storage)或[亚马逊 S3](/web/20221129124459/https://www.baeldung.com/aws-s3-java) 。该位置也可以使用 URI 格式，例如`s3://somebucket/path/to/file`。

我们的上传服务不是将文件的字节写入数据库，而是将文件存储在适当的服务中——在本例中是文件系统——然后将文件的位置放入数据库。

## 7.文件系统存储

让我们在解决方案中添加将图像存储在文件系统中的功能。

### 7.1.保存在文件系统中

首先，我们需要将图像保存到文件系统:

```
@Repository
class FileSystemRepository {

    String RESOURCES_DIR = FileSystemRepository.class.getResource("/")
        .getPath();

    String save(byte[] content, String imageName) throws Exception {
        Path newFile = Paths.get(RESOURCES_DIR + new Date().getTime() + "-" + imageName);
        Files.createDirectories(newFile.getParent());

        Files.write(newFile, content);

        return newFile.toAbsolutePath()
            .toString();
    }
}
```

一个重要的注意事项——我们需要确保**我们的每个图像在上传时都有一个唯一的`location` 定义的服务器端**。否则，我们的上传可能会互相覆盖。

同样的规则也适用于任何云存储，我们应该创建唯一的键。在本例中，我们将以毫秒为单位将当前日期添加到图像名称中:

```
/workspace/archive-achive/target/classes/1602949218879-baeldung.jpeg
```

### 7.2.从文件系统中检索

现在让我们实现代码来从文件系统中获取我们的映像:

```
FileSystemResource findInFileSystem(String location) {
    try {
        return new FileSystemResource(Paths.get(location));
    } catch (Exception e) {
        // Handle access or file not found problems.
        throw new RuntimeException();
    }
}
```

这里我们**使用它的`location`** 寻找图像。然后我们返回一个`FileSystemResource`。

此外，我们正在捕捉读取文件时可能发生的任何异常。我们可能还希望抛出带有特定 HTTP 状态的异常。

### 7.3.数据流和 Spring 资源

我们的`findInFileSystem`方法返回一个`[FileSystemResource](https://web.archive.org/web/20221129124459/https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/core/io/FileSystemResource.html),` 一个 [Spring 的`Resource`](/web/20221129124459/https://www.baeldung.com/spring-classpath-file-access) 接口的实现。

只有当我们使用它的时候，它才会开始读取我们的文件。在我们的例子中，它是通过`RestController`发送给客户机的。此外，它将把文件内容从文件系统流式传输给用户，**让我们不用把所有的字节都加载到内存中**。

这种方法是将文件流式传输到客户端的一个很好的通用解决方案。如果我们使用云存储而不是文件系统，我们可以为另一个资源的实现替换`FileSystemResource`，比如`[InputStreamResource](https://web.archive.org/web/20221129124459/https://docs.spring.io/spring-framework/docs/3.2.4.RELEASE_to_4.0.0.M3/Spring%20Framework%203.2.4.RELEASE/org/springframework/core/io/InputStreamResource.html)`或 [`ByteArrayResource`](https://web.archive.org/web/20221129124459/https://docs.spring.io/spring-framework/docs/3.0.6.RELEASE_to_3.1.0.BUILD-SNAPSHOT/3.0.6.RELEASE/org/springframework/core/io/ByteArrayResource.html) 。

## 8.连接文件内容和位置

现在我们有了我们的`FileSystemRepository,` ,我们需要把它和我们的`ImageDbRepository.`联系起来

### 8.1.保存在数据库和文件系统中

让我们创建一个`FileLocationService`，从我们的保存流开始:

```
@Service
class FileLocationService {

    @Autowired
    FileSystemRepository fileSystemRepository;
    @Autowired
    ImageDbRepository imageDbRepository;

    Long save(byte[] bytes, String imageName) throws Exception {
        String location = fileSystemRepository.save(bytes, imageName);

        return imageDbRepository.save(new Image(imageName, location))
            .getId();
    }
}
```

首先，我们**将映像保存在文件系统**中。然后我们**将包含其`location`的记录保存在数据库**中。

### 8.2.从数据库和文件系统中检索

现在，让我们创建一个方法来使用它的`id`找到我们的图像:

```
FileSystemResource find(Long imageId) {
    Image image = imageDbRepository.findById(imageId)
      .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));

    return fileSystemRepository.findInFileSystem(image.getLocation());
} 
```

首先，我们**在数据库**中寻找我们的图像。然后我们得到它的位置，**从文件系统**中获取它。

如果我们在数据库中没有找到`imageId`，我们将使用 ResponseStatusException 返回一个 HTTP Not Found 响应`.`

## 9.文件系统上传和下载

最后，让我们创建`FileSystemImageController:`

```
@RestController
@RequestMapping("file-system")
class FileSystemImageController {

    @Autowired
    FileLocationService fileLocationService;

    @PostMapping("/image")
    Long uploadImage(@RequestParam MultipartFile image) throws Exception {
        return fileLocationService.save(image.getBytes(), image.getOriginalFilename());
    }

    @GetMapping(value = "/image/{imageId}", produces = MediaType.IMAGE_JPEG_VALUE)
    FileSystemResource downloadImage(@PathVariable Long imageId) throws Exception {
        return fileLocationService.find(imageId);
    }
}
```

首先，我们让新路径从“/ `file-system`”开始。

然后我们创建了类似于`ImageController`中的上传路径，但是没有`dbImage`对象。

最后，我们有自己的下载路径，它使用`FileLocationService`来查找图像，并将`FileSystemResource`作为 HTTP 响应返回。

## 10.文件系统映像存档测试

现在，我们可以像测试数据库版本一样测试我们的文件系统版本，尽管路径现在以“`file-system`”开头:

```
curl -H "Content-Type: multipart/form-data" \
  -F "[[email protected]](/web/20221129124459/https://www.baeldung.com/cdn-cgi/l/email-protection)" http://localhost:8080/file-system/image

1
```

然后我们下载:

```
curl -v http://localhost:8080/file-system/image/1 -o image.jpeg
```

## 11.结论

在本文中，我们学习了如何在数据库中保存文件信息，将文件内容保存在同一行或外部位置。

我们还使用多部分上传构建并测试了一个 REST API，并使用`Resource`提供了一个下载特性，允许将文件流式传输给调用者。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221129124459/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2)