# 通过亚马逊 S3 使用 JetS3t Java 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jets3t-amazon-s3>

## 1。概述

在本教程中，我们将使用 [JetS3t](https://web.archive.org/web/20220524003643/http://www.jets3t.org/) 库和[亚马逊 S3。](https://web.archive.org/web/20220524003643/https://aws.amazon.com/s3/)

简单地说，我们将创建桶，向其中写入数据，读回数据，复制数据，然后列出并删除它们。

## 2。JetS3t 设置

### 2.1。Maven 依赖关系

首先，我们需要将 NATS 库和 Apache HttpClient 添加到我们的 T0 中:

```java
<dependency>
    <groupId>org.lucee</groupId>
    <artifactId>jets3t</artifactId>
    <version>0.9.4.0006L</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.5</version>
</dependency> 
```

Maven Central 有[最新版本的 JetS3t 库](https://web.archive.org/web/20220524003643/https://search.maven.org/classic/#search%7Cga%7C1%7Cjets3t)和[最新版本的 HttpClient](https://web.archive.org/web/20220524003643/https://search.maven.org/classic/#artifactdetails%7Corg.apache.httpcomponents%7Chttpclient%7C4.5.5%7Cjar) 。JetS3t 的来源可以在[这里](https://web.archive.org/web/20220524003643/https://sourceforge.net/p/jets3t/wiki/Home/)找到。

我们将使用 [Apache Commons 编解码器](https://web.archive.org/web/20220524003643/https://commons.apache.org/proper/commons-codec/)进行我们的一项测试，所以我们也将它添加到我们的`pom.xml `中:

```java
<dependency>
    <groupId>org.lucee</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.10.L001</version>
</dependency> 
```

Maven Central 有最新版本[这里](https://web.archive.org/web/20220524003643/https://search.maven.org/classic/#artifactdetails%7Corg.lucee%7Ccommons-codec%7C1.10.L001%7Cbundle)。

### 2.2。亚马逊 AWS 键

我们需要 AWS 访问密钥来连接 S3 存储服务。在这里可以创建一个免费账户[。](https://web.archive.org/web/20220524003643/https://aws.amazon.com/free/?sc_ichannel=ha&sc_icampaign=signin_prospects&sc_isegment=en&sc_iplace=sign-in&sc_icontent=freetier&sc_segment=-1)

在我们拥有一个帐户后，我们需要创建一组[安全密钥。](https://web.archive.org/web/20220524003643/https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html)这里有关于用户和访问密钥的文档[。](https://web.archive.org/web/20220524003643/https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html)

JetS3t 使用 Apache Commons 日志，所以当我们想要打印我们正在做的事情的信息时，我们也会使用它。

## 3。连接到简单存储器

现在我们有了 AWS 访问密钥和秘密密钥，我们可以连接到 S3 存储。

### 3.1。连接到 AWS

首先，我们创建 AWS 凭证，然后使用它们连接到服务:

```java
AWSCredentials awsCredentials 
  = new AWSCredentials("access key", "secret key");
s3Service = new RestS3Service(awsCredentials); 
```

**`RestS3Service`是我们与亚马逊 S3 的联系。**它使用`HttpClient`通过 REST 与 S3 通信。

### 3.2。验证连接

我们可以通过列出存储桶来验证我们已经成功连接到服务:

```java
S3Bucket[] myBuckets = s3Service.listAllBuckets(); 
```

根据我们之前是否创建过桶，数组可能是空的，但是如果操作没有抛出异常，我们就有了一个有效的连接。

## 4。桶管理

通过连接到亚马逊 S3，我们可以创建存储桶来保存我们的数据。S3 是一个对象存储系统。数据作为对象上传并存储在存储桶中。

由于所有 S3 存储桶共享相同的全局名称空间，每个存储桶必须有唯一的名称。

### 4.1。创建存储桶

让我们试着创建一个名为"`mybucket`"的桶:

```java
S3Bucket bucket = s3Service.createBucket("mybucket"); 
```

这将失败，并出现异常:

```java
org.jets3t.service.S3ServiceException: Service Error Message.
  -- ResponseCode: 409, ResponseStatus: Conflict, XML Error Message:
  <!--?xml version="1.0" encoding="UTF-8"?-->
  <code>BucketAlreadyExists</code> The requested bucket name is not available. 
  The bucket namespace is shared by all users of the system.
  Please select a different name and try again.
  mybucket 07BE34FF3113ECCF 
at org.jets3t.service.S3Service.createBucket(S3Service.java:1586)
```

不出所料，“`mybucket`”这个名字已经被占用了。在本教程的其余部分，我们将编造我们的名字。

让我们用不同的名称再试一次:

```java
S3Bucket bucket = s3Service.createBucket("myuniquename");
log.info(bucket); 
```

使用一个惟一的名称，调用成功，我们会看到关于我们的存储桶的信息:

```java
[INFO] JetS3tClient - S3Bucket
[name=myuniquename,location=US,creationDate=Sat Mar 31 16:47:47 EDT 2018,owner=null] 
```

### 4.2。删除存储桶

删除一个 bucket 和创建它一样简单，除了一点；桶必须是空的才能被移走！

```java
s3Service.deleteBucket("myuniquename"); 
```

这将为非空的存储桶引发异常。

### 4.3。指定桶区域

可以在特定的数据中心创建存储桶。对于 JetS3t，默认为美国的北弗吉尼亚，即“美国东部 1 号”

我们可以通过指定不同的区域来覆盖它:

```java
S3Bucket euBucket 
  = s3Service.createBucket("eu-bucket", S3Bucket.LOCATION_EUROPE);
S3Bucket usWestBucket = s3Service
  .createBucket("us-west-bucket", S3Bucket.LOCATION_US_WEST);
S3Bucket asiaPacificBucket = s3Service
  .createBucket("asia-pacific-bucket", S3Bucket.LOCATION_ASIA_PACIFIC); 
```

JetS3t 有一个被定义为常量的区域列表。

## 5。上传、下载和删除数据

一旦我们有了一个桶，我们就可以向它添加对象。桶应该是持久的，并且对一个桶可以容纳的对象的大小或数量没有硬性限制。

通过创建`S3Objects.` **将数据上传到 S3 我们可以从`InputStream,`上传数据 a，但是 JetS3t 也为`Strings`和** `**Files**.`提供了方便的方法

### 5.1。`String`数据

先来看看`Strings`:

```java
S3Object stringObject = new S3Object("object name", "string object");
s3Service.putObject("myuniquebucket", stringObject); 
```

与桶类似，对象也有名字，**然而，对象名只存在于它们的桶中，所以我们不必担心它们是全局唯一的。**

我们通过向构造函数传递名称和数据来创建对象。然后我们用`putObject.`存储它

当我们用这个方法用 JetS3t 存储`Strings`时，它为我们设置了正确的内容类型。

让我们向 S3 查询有关我们对象的信息，并查看内容类型:

```java
StorageObject objectDetailsOnly 
  = s3Service.getObjectDetails("myuniquebucket", "my string");
log.info("Content type: " + objectDetailsOnly.getContentType() + " length: " 
  + objectDetailsOnly.getContentLength()); 
```

`ObjectDetailsOnly()`检索对象元数据，而不下载它。当我们记录内容类型时，我们看到:

```java
[INFO] JetS3tClient - Content type: text/plain; charset=utf-8 length: 9 
```

JetS3t 将数据识别为文本，并为我们设置了长度。

让我们下载数据，并将其与我们上传的数据进行比较:

```java
S3Object downloadObject = 
  s3Service.getObject("myuniquebucket, "string object");
String downloadString = new BufferedReader(new InputStreamReader(
  object.getDataInputStream())).lines().collect(Collectors.joining("\n"));

assertTrue("string object".equals(downloadString));
```

数据在我们用来上传它的同一个`S3Object`中被检索，可用字节在一个`DataInputStream.`中

### 5.2。文件数据

上传文件的过程类似于`Strings`:

```java
File file = new File("src/test/resources/test.jpg");
S3Object fileObject = new S3Object(file);
s3Service.putObject("myuniquebucket", fileObject); 
```

当`S3Objects`被传递一个`File` 时，它们的名字来源于它们包含的文件的基本名:

```java
[INFO] JetS3tClient - File object name is test.jpg
```

**JetS3t 取`File` 上传给我们。**它将尝试从类路径中加载一个 [mime.types 文件](https://web.archive.org/web/20220524003643/https://en.wikipedia.org/wiki/Media_type#mime.types)，并使用它来识别文件的类型和发送的内容类型。

如果我们检索文件上传的对象信息并获得我们看到的内容类型:

```java
[INFO] JetS3tClient - Content type:application/octet-stream
```

让我们将我们的文件下载到一个新文件中，并比较其中的内容:

```java
String getFileMD5(String filename) throws IOException {
    try (FileInputStream fis = new FileInputStream(new File(filename))) {
        return DigestUtils.md5Hex(fis);
    }
}

S3Object fileObject = s3Service.getObject("myuniquebucket", "test.jpg"); 
File newFile = new File("/tmp/newtest.jpg"); 
Files.copy(fileObject.getDataInputStream(), newFile.toPath(), 
  StandardCopyOption.REPLACE_EXISTING);
String origMD5 = getFileMD5("src/test/resources/test.jpg");
String newMD5 = getFileMD5("src/test/resources/newtest.jpg");
assertTrue(origMD5.equals(newMD5));
```

与`Strings`类似，我们下载了对象并使用`DataInputStream`创建了一个新文件。然后，我们为这两个文件计算了一个 MD5 散列，并对它们进行了比较。

### 5.3。流数据

**当我们上传除了`Strings`或`Files,`之外的对象时，我们有更多的工作要做:**

```java
ArrayList<Integer> numbers = new ArrayList<>();
// adding elements to the ArrayList

ByteArrayOutputStream bytes = new ByteArrayOutputStream();
ObjectOutputStream objectOutputStream = new ObjectOutputStream(bytes);
objectOutputStream.writeObject(numbers);

ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes.toByteArray());

S3Object streamObject = new S3Object("stream");
streamObject.setDataInputStream(byteArrayInputStream);
streamObject.setContentLength(byteArrayInputStream.available());
streamObject.setContentType("binary/octet-stream");

s3Service.putObject(BucketName, streamObject); 
```

我们需要在上传之前设置我们的内容类型和长度。

检索该流意味着颠倒该过程:

```java
S3Object newStreamObject = s3Service.getObject(BucketName, "stream");

ObjectInputStream objectInputStream = new ObjectInputStream(
  newStreamObject.getDataInputStream());
ArrayList<Integer> newNumbers = (ArrayList<Integer>) objectInputStream
  .readObject();

assertEquals(2, (int) newNumbers.get(0));
assertEquals(3, (int) newNumbers.get(1));
assertEquals(5, (int) newNumbers.get(2));
assertEquals(7, (int) newNumbers.get(3)); 
```

对于不同的数据类型，content type 属性可用于选择解码对象的不同方法。

## 6。复制、移动和重命名数据

### 6.1。复制对象

对象可以在 S3 内部被复制，而不需要检索它们。

让我们复制 5.2 节中的测试文件，并验证结果:

```java
S3Object targetObject = new S3Object("testcopy.jpg");
s3Service.copyObject(
  BucketName, "test.jpg", 
  "myuniquebucket", targetObject, false);
S3Object newFileObject = s3Service.getObject(
  "myuniquebucket", "testcopy.jpg");

File newFile = new File("src/test/resources/testcopy.jpg");
Files.copy(
  newFileObject.getDataInputStream(), 
  newFile.toPath(), 
  REPLACE_EXISTING);
String origMD5 = getFileMD5("src/test/resources/test.jpg");
String newMD5 = getFileMD5("src/test/resources/testcopy.jpg");

assertTrue(origMD5.equals(newMD5)); 
```

我们可以在同一个桶中或者在两个不同的桶之间复制对象。

如果最后一个参数为真，复制的对象将接收新的元数据。否则，它将保留源对象的元数据。

如果我们想修改元数据，我们可以将标志设置为 true:

```java
targetObject = new S3Object("testcopy.jpg");
targetObject.addMetadata("My_Custom_Field", "Hello, World!");
s3Service.copyObject(
  "myuniquebucket", "test.jpg", 
  "myuniquebucket", targetObject, true); 
```

### 6.2。移动物体

**可以将对象移动到同一区域的另一个 S3 桶中。**移动操作是复制然后删除的操作。

如果拷贝操作失败，源对象不会被删除。如果删除操作失败，该对象仍将存在于源位置和目标位置。

移动对象看起来类似于复制它:

```java
s3Service.moveObject(
  "myuniquebucket",
  "test.jpg",
  "myotheruniquebucket",
  new S3Object("spidey.jpg"),
  false); 
```

### 6.3。重命名对象

**JetS3t 有一个方便的重命名对象的方法。**要改变一个对象的名字，我们只需用一个新的`S3Object`来调用它:

```java
s3Service.renameObject(
  "myuniquebucket", "test.jpg", new S3Object("spidey.jpg")); 
```

## 7 .**。结论**

在本教程中，我们使用 JetS3t 连接到亚马逊 S3。我们创建和删除了存储桶。然后，我们将不同类型的数据添加到桶中，并检索数据。最后，我们复制并移动了我们的数据。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220524003643/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-s3)