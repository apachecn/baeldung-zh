# 用 jclouds 库上传 S3

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2013/04/01/upload-on-s3-with-jclouds>

在 Java 世界中，有几种将内容上传到 S3 桶的好方法——在本文中，我们将看看 jclouds 库为此提供了什么。

为了使用 jclouds——特别是本文中讨论的 API，应该将这个简单的 Maven 依赖项添加到项目的 pom 中:

```
<dependency>
   <groupId>org.jclouds</groupId>
   <artifactId>jclouds-allblobstore</artifactId>
   <version>1.5.10</version>
</dependency>
```

## 1。上传到亚马逊 S3

为了访问这些 API，第一步是创建一个`BlobStoreContext`:

```
BlobStoreContext context = 
  ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
    .buildView(BlobStoreContext.class);
```

这代表了一般键值存储服务的入口点，例如亚马逊 S3，但不限于此。

对于更具体的仅 S3 实现，可以类似地创建上下文:

```
BlobStoreContext context = 
  ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
    .buildView(S3BlobStoreContext.class);
```

更具体地说:

```
BlobStoreContext context = 
  ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
    .buildView(AWSS3BlobStoreContext.class);
```

当不再需要认证上下文时，**关闭**它需要释放所有与其相关的资源——线程和连接。

## 2。jclouds 的四个 S3 API

jclouds 库提供了四种不同的 API 向 S3 bucket 上传内容，从简单但不灵活到复杂而强大，所有这些都是通过`BlobStoreContext`获得的。先说最简单的。

### 2.1。通过`Map` API 上传

使用 jclouds 与 S3 桶进行交互的最简单方法是将该桶表示为一个地图。API 是从上下文中获得的:

```
InputStreamMap bucket = context.createInputStreamMap("bucketName");
```

然后，上传一个简单的 HTML 文件:

```
bucket.putString("index1.html", "<html><body>hello world1</body></html>");
```

`InputStreamMap` API 公开了其他几种类型的 PUT 操作——文件、原始字节——包括单次和批量。

一个简单的集成测试可以作为一个例子:

```
@Test
public void whenFileIsUploadedToS3WithMapApi_thenNoExceptions() {
   BlobStoreContext context = 
      ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
         .buildView(AWSS3BlobStoreContext.class);

   InputStreamMap bucket = context.createInputStreamMap("bucketName");  

   bucket.putString("index1.html", "<html><body>hello world1</body></html>");
   context.close();
}
```

### 2.2。通过`BlobMap` 上传

使用简单地图 API 很简单，但最终会受到限制——例如，无法传入关于上传内容的元数据。当需要更多的灵活性和定制时，这种通过地图向 S3 上传数据的简化方法已经不够了。

我们要看的下一个 API 是 Blob Map API，它是从上下文中获得的:

```
BlobMap bucket = context.createBlobMap("bucketName");
```

API 允许客户端访问更多的底层细节，如`Content`—`Length`、`Content-Type`、`Content-Encoding`、`eTag` hash 等；要在桶中上传新内容:

```
Blob blob = bucket.blobBuilder().name("index2.html").
   payload("<html><body>hello world2</body></html>").
      contentType("text/html").calculateMD5().build();
```

API 还允许在创建请求上设置各种有效负载。

通过 Blob Map API 向 S3 上传一个基本 HTML 文件的简单集成测试:

```
@Test
public void whenFileIsUploadedToS3WithBlobMap_thenNoExceptions() throws IOException {
   BlobStoreContext context = 
      ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
         .buildView(AWSS3BlobStoreContext.class);

   BlobMap bucket = context.createBlobMap("bucketName");

   Blob blob = bucket.blobBuilder().name("index2.html").
      payload("<html><body>hello world2</body></html>").
         contentType("text/html").calculateMD5().build();
   bucket.put(blob.getMetadata().getName(), blob);

   context.close();
}
```

### 2.3。通过`BlobStore` 上传

以前的 API 无法使用**多部分上传**来上传内容——这使得它们不适合处理大文件。这个限制将由我们接下来要看的 API 解决——同步 BlobStore API。

这是从上下文中获得的:

```
BlobStore blobStore = context.getBlobStore();
```

要使用多部分支持并将文件上传到 S3:

```
Blob blob = blobStore.blobBuilder("index3.html").
   payload("<html><body>hello world3</body></html>").contentType("text/html").build();
blobStore.putBlob("bucketName", blob, PutOptions.Builder.multipart());
```

payload builder 与`BlobMap` API 使用的是同一个构建器，所以在指定关于 blob 的低级元数据信息时，这里也有同样的灵活性。不同之处在于 API 的 PUT 操作支持的`PutOptions`——即**多部分支持**。

先前的集成测试现在启用了多部分:

```
@Test
public void whenFileIsUploadedToS3WithBlobStore_thenNoExceptions() {
   BlobStoreContext context = 
      ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
         .buildView(AWSS3BlobStoreContext.class);

   BlobStore blobStore = context.getBlobStore();

   Blob blob = blobStore.blobBuilder("index3.html").
      payload("<html><body>hello world3</body></html>").contentType("text/html").build();
   blobStore.putBlob("bucketName", blob, PutOptions.Builder.multipart());
   context.close();
}
```

### 2.4。通过`AsyncBlobStore` 上传

虽然之前的 BlobStore API 是同步的，但是还有一个用于`BlobStore`–`AsyncBlobStore`的**异步 API** 。API 同样从上下文中获得:

```
AsyncBlobStore blobStore = context.getAsyncBlobStore();
```

两者之间唯一的区别是异步 API 为 **`PUT`异步操作**返回`ListenableFuture`:

```
Blob blob = blobStore.blobBuilder("index4.html").
   .payload("<html><body>hello world4</body></html>").build();
blobStore.putBlob("bucketName", blob)<strong>.get()</strong>;
```

显示此操作的集成测试类似于同步测试:

```
@Test
public void whenFileIsUploadedToS3WithBlobStore_thenNoExceptions() {
   BlobStoreContext context = 
      ContextBuilder.newBuilder("aws-s3").credentials(identity, credentials)
         .buildView(AWSS3BlobStoreContext.class);

   BlobStore blobStore = context.getBlobStore();

   Blob blob = blobStore.blobBuilder("index4.html").
      payload("<html><body>hello world4</body></html>").contentType("text/html").build();
   Future<String> putOp = blobStore.putBlob("bucketName", blob, PutOptions.Builder.multipart());
   putOp.get();
   context.close();
}
```

## 3。结论

在本文中，我们分析了 jclouds 库提供的用于向亚马逊 S3 上传内容的四个 API。这四个 API 是**通用的**，它们也可以与其他键值存储服务协同工作——比如微软 Azure Storage。

在下一篇文章中，我们将看看 jclouds 中可用的亚马逊特定 S3 API——`AWSS3Client`。我们将实现上传大文件的操作，动态计算任何给定文件的最佳部分数量，并并行执行所有部分的上传。