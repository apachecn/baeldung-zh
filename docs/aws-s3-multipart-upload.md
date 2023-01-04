# 用 Java 在亚马逊 S3 进行多部分上传

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-s3-multipart-upload>

## 1。概述

在本教程中，我们将了解如何使用 AWS Java SDK 在亚马逊 S3 中处理多部分上传。

简单地说，在多部分上传中，我们将内容分成更小的部分，然后分别上传每个部分。所有零件在收到时都已重新组装。

多部分上传具有以下优势:

*   更高的吞吐量–我们可以并行上传零件
*   更容易的错误恢复–我们只需要重新上传失败的部分
*   暂停和恢复上传–我们可以在任何时间点上传零件。整个过程可以暂停，其余部分可以稍后上传

请注意，当亚马逊 S3 使用多部分上传**时，除了最后一部分，每个部分的大小必须至少为 5 MB。**

## 2。Maven 依赖关系

在开始之前，我们需要在项目中添加 AWS SDK 依赖项:

```
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk</artifactId>
    <version>1.11.290</version>
</dependency>
```

要查看最新版本，请查看 [Maven Central](https://web.archive.org/web/20221208143837/https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk) 。

## 3.执行多部分上传

### 3.1。创建亚马逊 S3 客户端

首先，我们需要**创建一个访问亚马逊 S3 的客户端。**为此，我们将使用`AmazonS3ClientBuilder` :

```
AmazonS3 amazonS3 = AmazonS3ClientBuilder
  .standard()
  .withCredentials(new DefaultAWSCredentialsProviderChain())
  .withRegion(Regions.DEFAULT_REGION)
  .build();
```

这将创建一个使用默认凭据提供者链来访问 AWS 凭据的客户端。

关于默认凭证提供者链如何工作的更多细节，请参见[文档](https://web.archive.org/web/20221208143837/https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html#credentials-default)。如果您使用的是默认区域(`US West-2`)以外的区域，请确保用该自定义区域替换`Regions.DEFAULT_REGION` 。

### 3.2。创建用于管理上传的 TransferManager】

我们将使用`TransferManagerBuilder`来创建一个`TransferManager`实例。

这个类**提供了简单的 API 来管理亚马逊 S3** 的上传和下载，并管理所有相关的任务:

```
TransferManager tm = TransferManagerBuilder.standard()
  .withS3Client(amazonS3)
  .withMultipartUploadThreshold((long) (5 * 1024 * 1025))
  .build();
```

多部分上载阈值指定大小(以字节为单位),超过该值的上载将作为多部分上载执行。

亚马逊 S3 规定最小部分大小为 5 MB(对于除最后部分之外的部分)，因此我们使用 5 MB 作为多部分上传阈值。

### 3.3。上传对象

**要使用`TransferManager`上传对象，我们只需调用它的`upload()`函数**。这将并行上传零件:

```
String bucketName = "baeldung-bucket";
String keyName = "my-picture.jpg";
String file = new File("documents/my-picture.jpg");
Upload upload = tm.upload(bucketName, keyName, file);
```

`TransferManager.upload()`返回一个`Upload`对象。这可用于检查上传的状态和管理上传。我们将在下一节中这样做。

### 3.4。等待上传完成

**`TransferManager.upload()`是非阻塞函数**；当上传在后台运行时，它会立即返回。

我们可以**使用返回的`Upload`对象等待上传完成**，然后退出程序:

```
try {
    upload.waitForCompletion();
} catch (AmazonClientException e) {
    // ...
}
```

### 3.5。跟踪上传进度

跟踪上传的进度是一个很常见的需求；我们可以借助一个`P` `rogressListener`实例来实现:

```
ProgressListener progressListener = progressEvent -> System.out.println(
  "Transferred bytes: " + progressEvent.getBytesTransferred());
PutObjectRequest request = new PutObjectRequest(
  bucketName, keyName, file);
request.setGeneralProgressListener(progressListener);
Upload upload = tm.upload(request);
```

我们创建的`ProgressListener`将继续打印传输的字节数，直到上传完成。

### 3.6。控制上传并行度

默认情况下，`TransferManager`最多使用十个线程来执行多部分上传。

然而，我们可以通过在构建`TransferManager`时指定一个`ExecutorService`来控制这一点:

```
int maxUploadThreads = 5;
TransferManager tm = TransferManagerBuilder.standard()
  .withS3Client(amazonS3)
  .withMultipartUploadThreshold((long) (5 * 1024 * 1025))
  .withExecutorFactory(() -> Executors.newFixedThreadPool(maxUploadThreads))
  .build();
```

这里，我们使用一个 lambda 来创建`ExecutorFactory`的包装器实现，并将其传递给`withExecutorFactory()`函数。

## 4。结论

在这篇简短的文章中，我们学习了如何使用 AWS SDK for Java 执行多部分上传，并了解了如何控制上传的某些方面并跟踪其进度。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-s3)