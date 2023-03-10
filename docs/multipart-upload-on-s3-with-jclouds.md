# 使用 jclouds 在 S3 上进行多部分上传

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/2013/04/04/multipart-upload-on-s3-with-jclouds>

## 1。目标

在之前关于 S3 上传的文章中，我们看到了如何使用 jclouds 的通用 Blob APIs 上传内容到 S3。在本文中，我们将使用 jclouds 的 **S3 特定异步 API 来上传内容，并利用 S3 提供的多部分上传功能[。](https://web.archive.org/web/20220120012756/http://aws.typepad.com/aws/2010/11/amazon-s3-multipart-upload.html "Amazon S3: Multipart Upload")**

## 2。准备工作

### 2.1。设置自定义 API

上传过程的第一部分是创建 jclouds API——这是亚马逊 S3 的自定义 API:

```java
public AWSS3AsyncClient s3AsyncClient() {
   String identity = ...
   String credentials = ...

   BlobStoreContext context = ContextBuilder.newBuilder("aws-s3").
      credentials(identity, credentials).buildView(BlobStoreContext.class);

   RestContext<AWSS3Client, AWSS3AsyncClient> providerContext = context.unwrap();
   return providerContext.getAsyncApi();
}
```

### 2.2。确定内容的部分数量

亚马逊 S3 上传的每个部分有 5 MB 的限制。因此，我们需要做的第一件事是确定我们可以将内容分成的正确部分数量，这样我们就不会有低于 5 MB 限制的部分:

```java
public static int getMaximumNumberOfParts(byte[] byteArray) {
   int numberOfParts= byteArray.length / fiveMB; // 5*1024*1024
   if (numberOfParts== 0) {
      return 1;
   }
   return numberOfParts;
}
```

### 2.3。将内容分成几部分

我们要把字节数组分成一定数量的部分:

```java
public static List<byte[]> breakByteArrayIntoParts(byte[] byteArray, int maxNumberOfParts) {
   List<byte[]> parts = Lists.<byte[]> newArrayListWithCapacity(maxNumberOfParts);
   int fullSize = byteArray.length;
   long dimensionOfPart = fullSize / maxNumberOfParts;
   for (int i = 0; i < maxNumberOfParts; i++) {
      int previousSplitPoint = (int) (dimensionOfPart * i);
      int splitPoint = (int) (dimensionOfPart * (i + 1));
      if (i == (maxNumberOfParts - 1)) {
         splitPoint = fullSize;
      }
      byte[] partBytes = Arrays.copyOfRange(byteArray, previousSplitPoint, splitPoint);
      parts.add(partBytes);
   }

   return parts;
}
```

我们将**测试**将字节数组分成几部分的逻辑——我们将生成一些字节，分割字节数组，使用番石榴重新组合在一起，并且**验证**我们得到了原始的:

```java
@Test
public void given16MByteArray_whenFileBytesAreSplitInto3_thenTheSplitIsCorrect() {
   byte[] byteArray = randomByteData(16);

   int maximumNumberOfParts = S3Util.getMaximumNumberOfParts(byteArray);
   List<byte[]> fileParts = S3Util.breakByteArrayIntoParts(byteArray, maximumNumberOfParts);

   assertThat(fileParts.get(0).length + fileParts.get(1).length + fileParts.get(2).length, 
      equalTo(byteArray.length));
   byte[] unmultiplexed = Bytes.concat(fileParts.get(0), fileParts.get(1), fileParts.get(2));
   assertThat(byteArray, equalTo(unmultiplexed));
}
```

为了生成数据，我们只需使用来自`Random`的支持:

```java
byte[] randomByteData(int mb) {
   byte[] randomBytes = new byte[mb * 1024 * 1024];
   new Random().nextBytes(randomBytes);
   return randomBytes;
}
```

### 2.4。创建有效载荷

既然我们已经为我们的内容确定了正确的部分数量，并且我们成功地将内容分成了多个部分，我们需要**为 jclouds API 生成有效负载对象**:

```java
public static List<Payload> createPayloadsOutOfParts(Iterable<byte[]> fileParts) {
   List<Payload> payloads = Lists.newArrayList();
   for (byte[] filePart : fileParts) {
      byte[] partMd5Bytes = Hashing.md5().hashBytes(filePart).asBytes();
      Payload partPayload = Payloads.newByteArrayPayload(filePart);
      partPayload.getContentMetadata().setContentLength((long) filePart.length);
      partPayload.getContentMetadata().setContentMD5(partMd5Bytes);
      payloads.add(partPayload);
   }
   return payloads;
}
```

## 3。上传

上传流程是一个灵活的多步骤流程，这意味着:

*   上传可以在获得所有数据之前**开始——数据可以在进入时上传**
*   数据以**块**的形式上传——如果这些操作中的一个失败了，它可以被简单地恢复
*   块可以并行上传**——这可以大大提高上传速度，尤其是在大文件的情况下**

 **### 3.1。开始上传操作

上传操作的第一步是**启动流程**。这个对 S3 的请求必须包含标准的 HTTP 头——`Content`—`MD5`头尤其需要计算。我们将在这里使用 Guava 哈希函数支持:

```java
Hashing.md5().hashBytes(byteArray).asBytes();
```

这是整个字节数组的 **md5 散列**，还不是部分的。

为了**启动上传**，以及与 S3 的所有进一步交互，我们将使用 AWS S3 async client——我们之前创建的异步 API:

```java
ObjectMetadata metadata = ObjectMetadataBuilder.create().key(key).contentMD5(md5Bytes).build();
String uploadId = s3AsyncApi.initiateMultipartUpload(container, metadata).get();
```

`**key**`是分配给对象的句柄——这需要是客户端指定的唯一标识符。

还要注意的是，尽管我们使用的是异步版本的 API，**我们为这个操作的结果阻塞了**——这是因为我们需要初始化的结果才能继续。

操作的结果是由 S3 返回的**上传 id**-这将在上传的整个生命周期中标识上传，并将出现在所有后续的上传操作中。

### 3.2。上传零件

下一步是**上传零件**。我们这里的目标是并行发送这些请求**，因为上传部分操作代表了大部分的上传过程:**

```java
List<ListenableFuture<String>> ongoingOperations = Lists.newArrayList();
for (int partNumber = 0; partNumber < filePartsAsByteArrays.size(); partNumber++) {
   ListenableFuture<String> future = s3AsyncApi.uploadPart(
      container, key, partNumber + 1, uploadId, payloads.get(partNumber));
   ongoingOperations.add(future);
}
```

零件号需要连续，但发送请求的顺序无关紧要。

在提交了所有上传部分请求之后，我们需要**等待它们的响应**，以便我们可以收集每个部分的单独 ETag 值:

```java
Function<ListenableFuture<String>, String> getEtagFromOp = 
  new Function<ListenableFuture<String>, String>() {
   public String apply(ListenableFuture<String> ongoingOperation) {
      try {
         return ongoingOperation.get();
      } catch (InterruptedException | ExecutionException e) {
         throw new IllegalStateException(e);
      }
   }
};
List<String> etagsOfParts = Lists.transform(ongoingOperations, getEtagFromOp);
```

无论出于何种原因，如果其中一个上传部分操作失败，**可以重试该操作**，直到成功。上面的逻辑不包含重试机制，但是构建它应该足够简单。

### 3.3。完成上传操作

上传过程的最后一步是**完成多部分操作**。S3 API 需要来自之前上传的部分的响应作为一个`Map`，我们现在可以很容易地从上面获得的 ETags 列表中创建它:

```java
Map<Integer, String> parts = Maps.newHashMap();
for (int i = 0; i < etagsOfParts.size(); i++) {
   parts.put(i + 1, etagsOfParts.get(i));
}
```

最后，发送完整的请求:

```java
s3AsyncApi.completeMultipartUpload(container, key, uploadId, parts).get();
```

这将返回已完成对象的最终 ETag，并完成整个上传过程。

## 4。结论

在本文中，我们使用定制的 S3 jclouds API 构建了一个支持多部分、完全并行的 S3 上传操作。这个操作已经可以直接使用了，但是可以用一些方法对它进行改进。

首先，应该在上传操作周围添加**重试逻辑**，以更好地处理失败。

接下来，对于非常大的文件，即使该机制并行发送所有上传多部分请求，**节流机制**仍然应该限制发送的并行请求的数量。这既是为了避免带宽成为瓶颈，也是为了确保亚马逊自己不会将上传过程标记为超过每秒请求的允许限制—[番石榴速率限制器](https://web.archive.org/web/20220120012756/https://github.com/google/guava/blob/master/guava/src/com/google/common/util/concurrent/RateLimiter.java "Guava RateLimiter")可能非常适合这一点。****