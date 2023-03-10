# 带 Java 的 AWS S3

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-s3-java>

## 1。简介

在本教程中，我们将学习如何通过 Java 编程与亚马逊 S3(简单存储服务)存储系统进行交互。

请记住，S3 的结构非常简单；每个 bucket 可以存储任意数量的对象，可以使用 SOAP 接口或 REST 风格的 API 来访问这些对象。

接下来，我们将使用 AWS SDK for Java 来创建、列出和删除 S3 存储桶。我们还将在这些桶中上传、列出、下载、复制、移动、重命名和删除对象。

## 2。Maven 依赖关系

在开始之前，我们需要在项目中声明 AWS SDK 依赖关系:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk</artifactId>
    <version>1.11.163</version>
</dependency>
```

要查看最新版本，可以查看 [Maven Central](https://web.archive.org/web/20221128105832/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22aws-java-sdk%22) 。

## 3。先决条件

要使用 AWS SDK，我们需要一些东西:

1.  AWS 帐户:我们需要一个亚马逊网络服务帐户。如果我们没有账户，我们可以继续[创建一个账户](https://web.archive.org/web/20221128105832/https://portal.aws.amazon.com/gp/aws/developer/registration/index.html)。
2.  **AWS 安全凭证:**这些是我们的访问密钥，允许我们对 AWS API 操作进行编程调用。我们可以通过两种方式获得这些凭证，要么从[安全凭证](https://web.archive.org/web/20221128105832/https://console.aws.amazon.com/iam/home#security_credential)页面的访问密钥部分使用 AWS root 帐户凭证，要么从 [IAM 控制台](https://web.archive.org/web/20221128105832/https://console.aws.amazon.com/iam/home)使用 IAM 用户凭证。
3.  **选择 AWS 区域:**我们还必须选择 AWS 区域，以便存储我们的亚马逊 S3 数据。请记住，S3 储物价格因地区而异。要了解更多细节，请前往[官方文档](https://web.archive.org/web/20221128105832/https://aws.amazon.com/s3/pricing/)。在本教程中，我们将使用美国东部(俄亥俄州，地区`us-east-2`)。

## 4。创建客户端连接

首先，我们需要创建一个客户端连接来访问亚马逊 S3 web 服务。为此，我们将使用`AmazonS3`接口:

```java
AWSCredentials credentials = new BasicAWSCredentials(
  "<AWS accesskey>", 
  "<AWS secretkey>"
); 
```

然后，我们将配置客户端:

```java
AmazonS3 s3client = AmazonS3ClientBuilder
  .standard()
  .withCredentials(new AWSStaticCredentialsProvider(credentials))
  .withRegion(Regions.US_EAST_2)
  .build();
```

## 5。亚马逊 S3 铲斗操作

### 5.1。创建存储桶

值得注意的是，bucket 名称空间由系统的所有用户共享。**因此，我们的 bucket 名称在亚马逊 S3** 的所有现有 bucket 名称中必须是唯一的(我们马上就会知道如何检查)。

此外，根据官方文件中的[规定，桶名必须符合以下要求:](https://web.archive.org/web/20221128105832/https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3.html#createBucket-java.lang.String-)

*   名称不应包含下划线
*   名称长度应该在 3 到 63 个字符之间
*   名字不应该以破折号结尾
*   名称不能包含相邻的句点
*   名称不能在句点旁边包含破折号(例如，“my-.bucket.com”和“my。-bucket "无效)
*   名称不能包含大写字符

现在让我们创建一个存储桶:

```java
String bucketName = "baeldung-bucket";

if(s3client.doesBucketExist(bucketName)) {
    LOG.info("Bucket name is not available."
      + " Try again with a different Bucket name.");
    return;
}

s3client.createBucket(bucketName);
```

这里我们使用的是在上一步中创建的`s3client`。在创建 bucket 之前，我们必须使用`doesBucketExist()` 方法检查我们的 bucket 名称是否可用。如果名称可用，那么我们将使用`createBucket()` 方法。

### 5.2。列表存储桶

既然我们已经创建了一些 bucket，那么让我们使用`listBuckets()`方法打印一个在我们的 S3 环境中可用的所有 bucket 的列表。该方法将返回所有存储桶的列表:

```java
List<Bucket> buckets = s3client.listBuckets();
for(Bucket bucket : buckets) {
    System.out.println(bucket.getName());
}
```

这将列出我们的 S3 环境中存在的所有存储桶:

```java
baeldung-bucket
baeldung-bucket-test2
elasticbeanstalk-us-east-2
```

### 5.3。删除存储桶

在删除之前，确保我们的存储桶是空的是很重要的。否则会抛出异常。另外，请注意，只有存储桶的所有者可以删除它，而不管它的权限(访问控制策略):

```java
try {
    s3client.deleteBucket("baeldung-bucket-test2");
} catch (AmazonServiceException e) {
    System.err.println("e.getErrorMessage());
    return;
}
```

## 6。亚马逊 S3 对象操作

亚马逊 S3 存储桶中的文件或数据集合被称为对象。我们可以对对象进行上传、列表、下载、复制、移动、重命名和删除等操作。

### 6.1。上传对象

上传对象是一个非常简单的过程。我们将使用`putObject()` 方法，它接受三个参数:

1.  `bucketName`:我们要上传对象的桶名
2.  `key`:这是文件的完整路径
3.  `file`:包含待上传数据的实际文件

```java
s3client.putObject(
  bucketName, 
  "Document/hello.txt", 
  new File("/Users/user/Document/hello.txt")
);
```

### 6.2。列表对象

我们将使用`listObjects()` 方法列出我们的 S3 桶中所有可用的对象:

```java
ObjectListing objectListing = s3client.listObjects(bucketName);
for(S3ObjectSummary os : objectListing.getObjectSummaries()) {
    LOG.info(os.getKey());
}
```

调用`s3client` 对象的`listObjects()`方法将产生`ObjectListing` 对象，该对象可用于获取指定桶中所有对象摘要的列表。我们只是在这里打印密钥，但也有一些其他选项可用，如大小、所有者、上次修改时间、存储类别等。

这将打印我们的桶中所有对象的列表:

```java
Document/hello.txt
```

### 6.3。下载对象

为了下载一个对象，我们将首先在将返回一个`S3Object`对象的`s3client,` 上使用`getObject()` 方法。一旦我们得到这个，我们将调用它的`getObjectContent()` 来得到一个`S3ObjectInputStream` 对象，它的行为就像一个传统的 Java `InputStream:`

```java
S3Object s3object = s3client.getObject(bucketName, "picture/pic.png");
S3ObjectInputStream inputStream = s3object.getObjectContent();
FileUtils.copyInputStreamToFile(inputStream, new File("/Users/user/Desktop/hello.txt"));
```

这里我们使用 Apache Commons 的`FileUtils.copyInputStreamToFile()`方法。我们也可以访问[这篇文章](/web/20221128105832/https://www.baeldung.com/convert-input-stream-to-a-file)，探索将`InputStream`转换为`File.`的其他方法

### 6.4。复制、重命名和移动对象

我们可以通过调用接受四个参数的`s3client,` 上的`copyObject()` 方法来复制一个对象:

1.  源时段名称
2.  源时段中的对象关键字
3.  目标存储桶名称(可以与源相同)
4.  目标存储桶中的对象关键字

```java
s3client.copyObject(
  "baeldung-bucket", 
  "picture/pic.png", 
  "baeldung-bucket2", 
  "document/picture.png"
);
```

注意:我们可以结合使用`copyObject()`方法和`deleteObject()`来执行移动和重命名任务。这将包括首先复制对象，然后将其从旧位置删除。

### 6.5。删除一个对象

要删除一个对象，我们将调用`s3client` 上的`deleteObject()` 方法，并传递桶名和对象键:

```java
s3client.deleteObject("baeldung-bucket","picture/pic.png");
```

### 6.6。删除多个对象

为了一次删除多个对象，我们将首先创建`DeleteObjectsRequest` 对象，并将 bucket 名称传递给它的构造函数。然后，我们将传递一个包含所有要删除的对象键的数组。

一旦我们有了这个`DeleteObjectsRequest` 对象，我们可以将它作为参数传递给我们的`s3client` 的`deleteObjects()` 方法。如果成功，它将删除我们提供的所有对象:

```java
String objkeyArr[] = {
  "document/hello.txt", 
  "document/pic.png"
};

DeleteObjectsRequest delObjReq = new DeleteObjectsRequest("baeldung-bucket")
  .withKeys(objkeyArr);
s3client.deleteObjects(delObjReq);
```

## 7。结论

在本文中，我们关注了与亚马逊 S3 web 服务交互的基础，包括桶和对象级别。

和往常一样，这篇文章的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20221128105832/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-s3)