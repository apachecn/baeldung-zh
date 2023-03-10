# S3 春季云自动气象站

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-aws-s3>

在这篇简短的文章中，我们将探讨 Spring 云平台中提供的 AWS 支持，重点关注 S3。

Content Series:[This article is part of a series:](javascript:void(0);)• Spring Cloud AWS – S3 (current article)[• Spring Cloud AWS – EC2](/web/20220626081329/https://www.baeldung.com/spring-cloud-aws-ec2)
[• Spring Cloud AWS – RDS](/web/20220626081329/https://www.baeldung.com/spring-cloud-aws-rds)
[• Spring Cloud AWS – Messaging Support](/web/20220626081329/https://www.baeldung.com/spring-cloud-aws-messaging)

## 1。简单 S3 下载

让我们从轻松访问存储在 S3 上的文件开始:

```java
@Autowired
ResourceLoader resourceLoader;

public void downloadS3Object(String s3Url) throws IOException {
    Resource resource = resourceLoader.getResource(s3Url);
    File downloadedS3Object = new File(resource.getFilename());

    try (InputStream inputStream = resource.getInputStream()) {
        Files.copy(inputStream, downloadedS3Object.toPath(), 
          StandardCopyOption.REPLACE_EXISTING);
    }
}
```

## 2。简单的 S3 上传

我们还可以上传文件:

```java
public void uploadFileToS3(File file, String s3Url) throws IOException {
    WritableResource resource = (WritableResource) resourceLoader
      .getResource(s3Url);

    try (OutputStream outputStream = resource.getOutputStream()) {
        Files.copy(file.toPath(), outputStream);
    }
}
```

## 3。S3 网址结构

使用以下格式表示`s3Url`:

```java
s3://<bucket>/<object>
```

例如，如果文件`bar.zip`在`my-s3-bucket`桶上的文件夹`foo`中，那么 URL 将是:

```java
s3://my-s3-bucket/foo/bar.zip
```

此外，我们还可以使用`ResourcePatternResolver`和 Ant 模式匹配一次下载多个对象:

```java
private ResourcePatternResolver resourcePatternResolver;

@Autowired
public void setupResolver(ApplicationContext applicationContext, AmazonS3 amazonS3) {
    this.resourcePatternResolver = 
      new PathMatchingSimpleStorageResourcePatternResolver(amazonS3, applicationContext);
 }

public void downloadMultipleS3Objects(String s3Url) throws IOException {
    Resource[] allFileMatchingPatten = this.resourcePatternResolver
      .getResources(s3Url);
        // ...
    }
}
```

URL 可以包含通配符，而不是确切的名称。

例如，`s3://my-s3-bucket/**/a*.txt URL` 将在`my-s3-bucket`的任何文件夹中递归查找名称以`a`开头的所有文本文件。

注意，bean`ResourceLoader`和`ResourcePatternResolver`是在应用程序启动时使用 Spring Boot 的自动配置特性创建的。

## 4。结论

我们完成了——这是一个关于使用 Spring Cloud AWS 访问 S3 的快速而中肯的介绍。

在本系列的下一篇文章中，我们将探索框架的 EC2 支持。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626081329/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws)

Next **»**[Spring Cloud AWS – EC2](/web/20220626081329/https://www.baeldung.com/spring-cloud-aws-ec2)