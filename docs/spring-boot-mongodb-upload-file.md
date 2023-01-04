# 使用 MongoDB 和 Spring Boot 上传和检索文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-mongodb-upload-file>

## 1。概述

在本教程中，我们将讨论如何使用 MongoDB 和 Spring Boot 上传和检索文件。

**我们将使用 MongoDB `BSON`处理小文件，使用`GridFS`处理大文件。**

## 2。Maven 配置

首先，我们将把`[spring-boot-starter-data-mongodb](https://web.archive.org/web/20220626211356/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-mongodb)`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

此外，我们将需要`spring-boot-starter-web`和`spring-boot-starter-thymeleaf`依赖项来显示我们应用程序的用户界面。这些依赖也显示在我们的[Spring Boot 百里香叶](/web/20220626211356/https://www.baeldung.com/spring-boot-crud-thymeleaf)指南中。

在本教程中，我们使用的是 Spring Boot 版本 2.x

## 3。Spring Boot 房产

接下来，我们将配置必要的 Spring Boot 属性。

让我们从**MongoDB 属性**开始:

```java
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=springboot-mongo
```

**我们还将设置 Servlet 多部分属性，以允许上传大文件:**

```java
spring.servlet.multipart.max-file-size=256MB
spring.servlet.multipart.max-request-size=256MB
spring.servlet.multipart.enabled=true
```

## 4。上传小文件

现在，我们将讨论如何使用 MongoDB `BSON`上传和检索小文件(大小< 16MB)。

这里，我们有一个简单的`Document`类— `Photo.` **我们将把我们的图像文件存储在一个`BSON` `Binary`** :

```java
@Document(collection = "photos")
public class Photo {
    @Id
    private String id;

    private String title;

    private Binary image;
}
```

我们会有一个简单的`PhotoRepository`:

```java
public interface PhotoRepository extends MongoRepository<Photo, String> { }
```

现在，对于`PhotoService`，我们只有两种方法:

*   `addPhoto()` —将`Photo`上传到 MongoDB
*   `getPhoto()` —检索具有给定 id 的`Photo`

```java
@Service
public class PhotoService {

    @Autowired
    private PhotoRepository photoRepo;

    public String addPhoto(String title, MultipartFile file) throws IOException { 
        Photo photo = new Photo(title); 
        photo.setImage(
          new Binary(BsonBinarySubType.BINARY, file.getBytes())); 
        photo = photoRepo.insert(photo); return photo.getId(); 
    }

    public Photo getPhoto(String id) { 
        return photoRepo.findById(id).get(); 
    }
}
```

## 5。上传大文件

**现在，我们将使用`GridFS`来上传和检索大文件。**

首先，我们将定义一个简单的 DTO 来表示一个大文件:

```java
public class Video {
    private String title;
    private InputStream stream;
}
```

类似于`PhotoService`，我们将有一个带有两种方法的`VideoService`—`addVideo()`和`getVideo()`:

```java
@Service
public class VideoService {

    @Autowired
    private GridFsTemplate gridFsTemplate;

    @Autowired
    private GridFsOperations operations;

    public String addVideo(String title, MultipartFile file) throws IOException { 
        DBObject metaData = new BasicDBObject(); 
        metaData.put("type", "video"); 
        metaData.put("title", title); 
        ObjectId id = gridFsTemplate.store(
          file.getInputStream(), file.getName(), file.getContentType(), metaData); 
        return id.toString(); 
    }

    public Video getVideo(String id) throws IllegalStateException, IOException { 
        GridFSFile file = gridFsTemplate.findOne(new Query(Criteria.where("_id").is(id))); 
        Video video = new Video(); 
        video.setTitle(file.getMetadata().get("title").toString()); 
        video.setStream(operations.getResource(file).getInputStream());
        return video; 
    }
}
```

关于使用 Spring 的更多细节，请查看我们的 Spring Data MongoDB 文章中的 [GridFS。](/web/20220626211356/https://www.baeldung.com/spring-data-mongodb-gridfs)

## 6。控制器

现在，让我们来看看控制器— `PhotoController`和`VideoController`。

### 6.1。`PhotoController`

首先，**我们有`PhotoController,` ，它将使用我们的`PhotoService`来添加/获取照片**。

我们将定义`addPhoto()`方法来上传并创建一个新的`Photo`:

```java
@PostMapping("/photos/add")
public String addPhoto(@RequestParam("title") String title, 
  @RequestParam("image") MultipartFile image, Model model) 
  throws IOException {
    String id = photoService.addPhoto(title, image);
    return "redirect:/photos/" + id;
}
```

我们还需要`getPhoto()`来检索具有给定 id 的照片:

```java
@GetMapping("/photos/{id}")
public String getPhoto(@PathVariable String id, Model model) {
    Photo photo = photoService.getPhoto(id);
    model.addAttribute("title", photo.getTitle());
    model.addAttribute("image", 
      Base64.getEncoder().encodeToString(photo.getImage().getData()));
    return "photos";
}
```

**注意，由于我们将图像数据作为`byte[]`返回，我们将它转换为`Base64` `String`以在前端显示。**

### 6.2。`VideoController`

接下来，我们来看看我们的`VideoController`。

这将有一个类似的方法`addVideo()`，上传一个`Video`到我们的 MongoDB:

```java
@PostMapping("/videos/add")
public String addVideo(@RequestParam("title") String title, 
  @RequestParam("file") MultipartFile file, Model model) throws IOException {
    String id = videoService.addVideo(title, file);
    return "redirect:/videos/" + id;
}
```

这里我们用`getVideo()`来检索一个带有给定`id`的`Video`:

```java
@GetMapping("/videos/{id}")
public String getVideo(@PathVariable String id, Model model) throws Exception {
    Video video = videoService.getVideo(id);
    model.addAttribute("title", video.getTitle());
    model.addAttribute("url", "/videos/stream/" + id);
    return "videos";
}
```

我们还可以添加一个`streamVideo()`方法，该方法将从`Video` `InputStream`创建一个流 URL:

```java
@GetMapping("/videos/stream/{id}")
public void streamVideo(@PathVariable String id, HttpServletResponse response) throws Exception {
    Video video = videoService.getVideo(id);
    FileCopyUtils.copy(video.getStream(), response.getOutputStream());        
}
```

## 7。前端

最后，让我们看看我们的前端。
让我们从`uploadPhoto.html`开始，它提供了一个简单的上传图片的表单:

```java
<html>
<body>
<h1>Upload new Photo</h1>
<form method="POST" action="/photos/add" enctype="multipart/form-data">
    Title:<input type="text" name="title" />
    Image:<input type="file" name="image" accept="image/*" />
    <input type="submit" value="Upload" />
</form>
</body>
</html>
```

接下来，我们将添加`photos.html`视图来显示我们的照片:

```java
<html>
<body>
    <h1>View Photo</h1>
    Title: <span th:text="${title}">name</span>
    <img alt="sample" th:src="*{'data:image/png;base64,'+image}" />
</body>
</html>
```

类似地，我们用`uploadVideo.html`上传一个`Video`:

```java
<html>
<body>
<h1>Upload new Video</h1>
<form method="POST" action="/videos/add" enctype="multipart/form-data">
    Title:<input type="text" name="title" />
    Video:<input type="file" name="file" accept="video/*" />
    <input type="submit" value="Upload" />
</form>
</body>
</html>
```

和`videos.html`显示视频:

```java
<html>
<body>
    <h1>View Video</h1>
    Title: <span th:text="${title}">title</span>
    <video width="400" controls>
        <source th:src="${url}" />
    </video>
</body>
</html>
```

## 8。结论

在本文中，我们学习了如何使用 MongoDB 和 Spring Boot 上传和检索文件。我们使用了`BSON`和`GridFS`来上传和检索文件。

和往常一样，完整的源代码可以在 [GitHub 项目](https://web.archive.org/web/20220626211356/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb)中获得。