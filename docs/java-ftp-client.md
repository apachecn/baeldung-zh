# 用 Java 实现 FTP 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ftp-client>

## 1.概观

在本教程中，我们将看看如何利用 [Apache Commons Net](https://web.archive.org/web/20220905150932/https://commons.apache.org/proper/commons-net/) 库与外部 FTP 服务器进行交互。

## 2.设置

当使用用于与外部系统交互的库时，编写一些额外的集成测试通常是个好主意，以确保我们正确地使用了库。

现在，我们通常使用 Docker 来启动这些系统进行集成测试。然而，特别是在被动模式下使用时，如果我们想要利用动态端口映射(这对于能够在共享 CI 服务器上运行的测试通常是必要的)，FTP 服务器并不是在容器内透明运行的最容易的应用程序。

这就是为什么我们将使用 [MockFtpServer](https://web.archive.org/web/20220905150932/http://mockftpserver.sourceforge.net/index.html) 来代替，这是一个用 Java 编写的假/存根 FTP 服务器，它提供了一个扩展的 API 以便在 JUnit 测试中使用:

```java
<dependency>
    <groupId>commons-net</groupId>
    <artifactId>commons-net</artifactId>
    <version>3.6</version>
</dependency>
<dependency> 
    <groupId>org.mockftpserver</groupId> 
    <artifactId>MockFtpServer</artifactId> 
    <version>2.7.1</version> 
    <scope>test</scope> 
</dependency>
```

建议始终使用最新版本。那些可以在这里找到[，在这里](https://web.archive.org/web/20220905150932/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-net%22)找到[。](https://web.archive.org/web/20220905150932/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22MockFtpServer%22)

## 3.JDK 的 FTP 支持

**令人惊讶的是，在一些 JDK 版本中已经有了对 FTP 的基本支持，形式是`sun.net.www.protocol.ftp.FtpURLConnection`。**

然而，我们不应该直接使用这个类，而是可以使用 JDK 的 URL 类作为抽象。

这种 FTP 支持非常基本，但是利用`java.nio.file.Files,`的便利 API，对于简单的用例就足够了:

```java
@Test
public void givenRemoteFile_whenDownloading_thenItIsOnTheLocalFilesystem() throws IOException {
    String ftpUrl = String.format(
      "ftp://user:[[email protected]](/web/20220905150932/https://www.baeldung.com/cdn-cgi/l/email-protection):%d/foobar.txt", fakeFtpServer.getServerControlPort());

    URLConnection urlConnection = new URL(ftpUrl).openConnection();
    InputStream inputStream = urlConnection.getInputStream();
    Files.copy(inputStream, new File("downloaded_buz.txt").toPath());
    inputStream.close();

    assertThat(new File("downloaded_buz.txt")).exists();

    new File("downloaded_buz.txt").delete(); // cleanup
}
```

由于这个基本的 FTP 支持已经缺少了像文件列表这样的基本特性，我们将在下面的例子中使用 Apache Net Commons 库中的 FTP 支持。

## 4.连接

我们首先需要连接到 FTP 服务器。让我们从创建一个类`FtpClient.`开始

它将作为实际 Apache Commons Net FTP 客户端的抽象 API:

```java
class FtpClient {

    private String server;
    private int port;
    private String user;
    private String password;
    private FTPClient ftp;

    // constructor

    void open() throws IOException {
        ftp = new FTPClient();

        ftp.addProtocolCommandListener(new PrintCommandListener(new PrintWriter(System.out)));

        ftp.connect(server, port);
        int reply = ftp.getReplyCode();
        if (!FTPReply.isPositiveCompletion(reply)) {
            ftp.disconnect();
            throw new IOException("Exception in connecting to FTP Server");
        }

        ftp.login(user, password);
    }

    void close() throws IOException {
        ftp.disconnect();
    }
}
```

我们需要服务器地址和端口，以及用户名和密码。连接后，有必要实际检查回复代码，以确保连接成功。我们还添加了一个`PrintCommandListener`，用来打印我们通常在使用命令行工具连接到 FTP 服务器时看到的对 stdout 的响应。

由于我们的集成测试会有一些样板代码，比如启动/停止 MockFtpServer 和连接/断开我们的客户端，我们可以在`@Before`和`@After`方法中做这些事情:

```java
public class FtpClientIntegrationTest {

    private FakeFtpServer fakeFtpServer;

    private FtpClient ftpClient;

    @Before
    public void setup() throws IOException {
        fakeFtpServer = new FakeFtpServer();
        fakeFtpServer.addUserAccount(new UserAccount("user", "password", "/data"));

        FileSystem fileSystem = new UnixFakeFileSystem();
        fileSystem.add(new DirectoryEntry("/data"));
        fileSystem.add(new FileEntry("/data/foobar.txt", "abcdef 1234567890"));
        fakeFtpServer.setFileSystem(fileSystem);
        fakeFtpServer.setServerControlPort(0);

        fakeFtpServer.start();

        ftpClient = new FtpClient("localhost", fakeFtpServer.getServerControlPort(), "user", "password");
        ftpClient.open();
    }

    @After
    public void teardown() throws IOException {
        ftpClient.close();
        fakeFtpServer.stop();
    }
}
```

通过将模拟服务器控制端口设置为值 0，我们启动了模拟服务器和一个空闲的随机端口。

这就是为什么我们必须在服务器启动后使用`fakeFtpServer.getServerControlPort()`创建`FtpClient`时检索实际的端口。

## 5.列出文件

第一个实际的用例是列出文件。

让我们先从测试开始，TDD 式的:

```java
@Test
public void givenRemoteFile_whenListingRemoteFiles_thenItIsContainedInList() throws IOException {
    Collection<String> files = ftpClient.listFiles("");
    assertThat(files).contains("foobar.txt");
}
```

实现本身同样简单明了。为了使返回的数据结构简单一点，我们使用 Java 8 `Streams:`将返回的`FTPFile`数组转换成一个`Strings`列表

```java
Collection<String> listFiles(String path) throws IOException {
    FTPFile[] files = ftp.listFiles(path);
    return Arrays.stream(files)
      .map(FTPFile::getName)
      .collect(Collectors.toList());
}
```

## 6.下载

为了从 FTP 服务器下载文件，我们定义了一个 API。

这里我们定义了本地文件系统上的源文件和目标:

```java
@Test
public void givenRemoteFile_whenDownloading_thenItIsOnTheLocalFilesystem() throws IOException {
    ftpClient.downloadFile("/buz.txt", "downloaded_buz.txt");
    assertThat(new File("downloaded_buz.txt")).exists();
    new File("downloaded_buz.txt").delete(); // cleanup
}
```

Apache Net Commons FTP 客户端包含一个方便的 API，它将直接写入一个已定义的`OutputStream.`这意味着我们可以直接使用它:

```java
void downloadFile(String source, String destination) throws IOException {
    FileOutputStream out = new FileOutputStream(destination);
    ftp.retrieveFile(source, out);
}
```

## 7.上传

MockFtpServer 为访问其文件系统的内容提供了一些有用的方法。我们可以使用这个特性为上传功能编写一个简单的集成测试:

```java
@Test
public void givenLocalFile_whenUploadingIt_thenItExistsOnRemoteLocation() 
  throws URISyntaxException, IOException {

    File file = new File(getClass().getClassLoader().getResource("baz.txt").toURI());
    ftpClient.putFileToPath(file, "/buz.txt");
    assertThat(fakeFtpServer.getFileSystem().exists("/buz.txt")).isTrue();
}
```

上传文件在 API 方面与下载文件非常相似，但是我们不使用`OutputStream`，而是需要提供一个`InputStream`:

```java
void putFileToPath(File file, String path) throws IOException {
    ftp.storeFile(path, new FileInputStream(file));
}
```

## 8.结论

我们已经看到，结合使用 Java 和 Apache Net Commons，我们可以轻松地与外部 FTP 服务器进行交互，进行读写访问。

像往常一样，本文的完整代码可以在我们的 [GitHub 库](https://web.archive.org/web/20220905150932/https://github.com/eugenp/tutorials/tree/master/libraries-6)中找到。