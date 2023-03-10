# 使用 Java 的 SSH 连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ssh-connection>

## 1.介绍

[SSH](/web/20221102033256/https://www.baeldung.com/cs/ssh-intro) ，也称为安全外壳或安全套接字外壳，是一种网络协议，允许[一台计算机通过不安全的网络安全地连接到另一台计算机](/web/20221102033256/https://www.baeldung.com/linux/secure-shell-ssh)。在本教程中，我们将展示如何**使用 JSch 和 Apache MINA SSHD 库**用 Java 建立到远程 SSH 服务器的连接。

在我们的示例中，我们将首先打开 SSH 连接，然后执行一个命令，读取输出并将其写入控制台，最后关闭 SSH 连接。我们将尽可能保持样本代码简单。

## 2.JSch

JSch 是 SSH2 的 Java 实现，它允许我们连接到 SSH 服务器并使用端口转发、X11 转发和文件传输。此外，它是在 BSD 风格许可下许可的，为我们提供了一种简单的方法来建立与 Java 的 SSH 连接。

首先，让我们将 [JSch Maven 依赖项](https://web.archive.org/web/20221102033256/https://search.maven.org/search?q=g:com.jcraft%20AND%20a:jsch)添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.55</version>
</dependency>
```

### 2.1.履行

**要使用 JSch 建立 SSH 连接，我们需要用户名、密码、主机 URL 和 SSH 端口**。默认的 SSH 端口是 22，但也有可能我们会将服务器配置为使用其他端口进行 SSH 连接:

```java
public static void listFolderStructure(String username, String password, 
  String host, int port, String command) throws Exception {

    Session session = null;
    ChannelExec channel = null;

    try {
        session = new JSch().getSession(username, host, port);
        session.setPassword(password);
        session.setConfig("StrictHostKeyChecking", "no");
        session.connect();

        channel = (ChannelExec) session.openChannel("exec");
        channel.setCommand(command);
        ByteArrayOutputStream responseStream = new ByteArrayOutputStream();
        channel.setOutputStream(responseStream);
        channel.connect();

        while (channel.isConnected()) {
            Thread.sleep(100);
        }

        String responseString = new String(responseStream.toByteArray());
        System.out.println(responseString);
    } finally {
        if (session != null) {
            session.disconnect();
        }
        if (channel != null) {
            channel.disconnect();
        }
    }
}
```

正如我们在代码中看到的，我们首先创建一个客户机会话，并将其配置为连接到我们的 SSH 服务器。然后，我们创建一个用于与 SSH 服务器通信的客户机通道，在这里我们提供一个通道类型——在本例中是,`exec,`,这意味着我们将向服务器传递 shell 命令。

此外，我们应该为我们的通道设置输出流，服务器响应将被写入其中。在我们使用`channel.connect()`方法建立连接之后，命令被传递，收到的响应被写入控制台。

让我们看看**如何使用 JSch 提供的不同配置参数**:

*   `StrictHostKeyChecking`–表示应用程序是否会检查主机公钥是否能在已知主机中找到。同样，可用的参数值有`ask`、`yes,`和`no`，其中`ask`为默认值。如果我们将这个属性设置为`yes`，JSch 将永远不会自动将主机密钥添加到`known_hosts`文件中，并且它将拒绝连接到主机密钥已经更改的主机。这迫使用户手动添加所有新主机。如果我们将其设置为`no`，JSch 将自动向已知主机列表中添加一个新的主机密钥
*   `compression.s2c`–指定是否对从服务器到客户端应用程序的数据流进行压缩。可用的值有`zlib`和`none`，其中第二个是默认值
*   `compression.c2s`–指定是否对客户端-服务器方向的数据流使用压缩。可用的值有`zlib`和`none`，其中第二个是默认值

重要的是**在与服务器的通信结束后关闭会话和 SFTP 通道，以避免内存泄漏**。

## 3.Apache MINA SSHD

Apache MINA SSHD 公司为基于 Java 的应用程序提供 SSH 支持。这个库基于 Apache MINA，这是一个可伸缩的高性能异步 IO 库。

让我们添加阿帕奇米娜 SSHD Maven 依赖关系:

```java
<dependency>
    <groupId>org.apache.sshd</groupId>
    <artifactId>sshd-core</artifactId>
    <version>2.5.1</version>
</dependency>
```

### 3.1.履行

让我们看看使用 Apache MINA SSHD 连接到 SSH 服务器的代码示例:

```java
public static void listFolderStructure(String username, String password, 
  String host, int port, long defaultTimeoutSeconds, String command) throws IOException {

    SshClient client = SshClient.setUpDefaultClient();
    client.start();

    try (ClientSession session = client.connect(username, host, port)
      .verify(defaultTimeoutSeconds, TimeUnit.SECONDS).getSession()) {
        session.addPasswordIdentity(password);
        session.auth().verify(defaultTimeoutSeconds, TimeUnit.SECONDS);

        try (ByteArrayOutputStream responseStream = new ByteArrayOutputStream(); 
          ClientChannel channel = session.createChannel(Channel.CHANNEL_SHELL)) {
            channel.setOut(responseStream);
            try {
                channel.open().verify(defaultTimeoutSeconds, TimeUnit.SECONDS);
                try (OutputStream pipedIn = channel.getInvertedIn()) {
                    pipedIn.write(command.getBytes());
                    pipedIn.flush();
                }

                channel.waitFor(EnumSet.of(ClientChannelEvent.CLOSED), 
                TimeUnit.SECONDS.toMillis(defaultTimeoutSeconds));
                String responseString = new String(responseStream.toByteArray());
                System.out.println(responseString);
            } finally {
                channel.close(false);
            }
        }
    } finally {
        client.stop();
    }
}
```

当使用 Apache MINA SSHD 时，我们有一个与 JSch 非常相似的事件序列。首先，我们使用`SshClient`类实例建立到 SSH 服务器的连接。如果我们用`SshClient.setupDefaultClient(),`初始化它，我们将能够使用具有适合大多数用例的默认配置的实例。这包括密码、压缩、MAC、密钥交换和签名。

之后，我们将创建`ClientChannel`并将`ByteArrayOutputStream`附加到它上面，以便我们将它用作响应流。正如我们所看到的，SSHD 要求每个操作都有规定的超时。它还允许我们使用`Channel.waitFor()`方法定义在命令传递后等待服务器响应的时间。

需要注意的是， **SSHD 会将完整的控制台输出写入响应流。JSch 只会用命令执行结果来做这件事。**

关于 Apache Mina SSHD 的完整文档可以在[项目的官方 GitHub 资源库](https://web.archive.org/web/20221102033256/https://github.com/apache/mina-sshd/tree/master/docs)中找到。

## 4.结论

本文展示了如何使用两个可用的 Java 库——JSch 和 Apache Mina SSHD——建立与 Java 的 SSH 连接。我们还展示了如何将命令传递给远程服务器并获得执行结果。另外，完整的代码样本可以在 GitHub 的[上找到。](https://web.archive.org/web/20221102033256/https://github.com/eugenp/tutorials/tree/master/libraries-security)