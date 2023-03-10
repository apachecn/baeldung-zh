# gRPC 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/grpc-introduction>

## 1。简介

gRPC 是一个高性能、开源的 RPC 框架，最初由 Google 开发。它有助于消除样板代码，并有助于在数据中心内和跨数据中心连接多语言服务。

## 2。概述

该框架基于远程过程调用的客户机-服务器模型。客户端应用程序可以直接调用服务器应用程序上的方法，就像它是本地对象一样。

本文将按照以下步骤使用 gRPC 创建一个典型的客户机-服务器应用程序:

1.  在一个`.proto`文件中定义一个服务
2.  使用协议缓冲编译器生成服务器和客户端代码
3.  创建服务器应用程序，实现生成的服务接口并生成 gRPC 服务器
4.  创建客户端应用程序，使用生成的存根进行 RPC 调用

让我们定义一个简单的`HelloService`，它返回问候来交换名字和姓氏。

## 3。Maven 依赖关系

让我们添加 [grpc-netty](https://web.archive.org/web/20220626205430/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22grpc-netty%22) 、 [grpc-protobuf](https://web.archive.org/web/20220626205430/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22grpc-protobuf%22) 和 [grpc-stub](https://web.archive.org/web/20220626205430/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22grpc-stub%22) 依赖项:

```java
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty</artifactId>
    <version>1.16.1</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>1.16.1</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>1.16.1</version>
</dependency> 
```

## 4。定义服务

我们从定义服务开始，**指定可以远程调用的方法以及它们的参数和返回类型**。

这是使用[协议缓冲区](/web/20220626205430/https://www.baeldung.com/google-protocol-buffer)在`.proto`文件中完成的。它们也用于描述有效载荷消息的结构。

### 4.1。基本配置

让我们为我们的示例`HelloService`创建一个`HelloService.proto`文件。我们从添加一些基本配置细节开始:

```java
syntax = "proto3";
option java_multiple_files = true;
package org.baeldung.grpc;
```

第一行告诉编译器在这个文件中使用了什么语法。默认情况下，编译器在一个 Java 文件中生成所有的 Java 代码。第二行覆盖了这个设置，所有内容都将在单独的文件中生成。

最后，我们指定要用于生成的 Java 类的包。

### 4.2。定义消息结构

接下来，我们定义消息:

```java
message HelloRequest {
    string firstName = 1;
    string lastName = 2;
}
```

这定义了请求有效负载。在这里，进入消息的每个属性都与其类型一起定义。

需要为每个属性分配一个唯一的编号，称为标签。**该标签被协议缓冲区用来表示属性，而不是使用属性名。**

因此，不像 JSON 那样每次都传递属性名`firstName`，protocol buffer 将使用数字 1 来表示`firstName`。响应负载定义类似于请求。

请注意，我们可以在多种消息类型中使用相同的标记:

```java
message HelloResponse {
    string greeting = 1;
}
```

### 4.3。定义服务合同

最后，让我们来定义服务契约。对于我们的`HelloService`,我们定义一个`hello()`操作:

```java
service HelloService {
    rpc hello(HelloRequest) returns (HelloResponse);
}
```

`hello()`操作接受一元请求并返回一元响应。gRPC 还通过在请求和响应前添加关键字`stream`来支持流。

## 5。生成代码

现在我们将`HelloService.proto`文件传递给协议缓冲编译器`protoc`来生成 Java 文件。有多种方法可以触发这种情况。

### 5.1。使用协议缓冲编译器

首先，我们需要协议缓冲编译器。我们可以从许多可用的预编译二进制文件中选择[这里](https://web.archive.org/web/20220626205430/https://github.com/protocolbuffers/protobuf/releases)。

此外，我们需要获得 gRPC Java Codegen 插件。

最后，我们可以使用下面的命令来生成代码:

```java
protoc --plugin=protoc-gen-grpc-java=$PATH_TO_PLUGIN -I=$SRC_DIR 
  --java_out=$DST_DIR --grpc-java_out=$DST_DIR $SRC_DIR/HelloService.proto
```

### 5.2。使用 Maven 插件

作为开发人员，您会希望代码生成与您的构建系统紧密集成。gRPC 为 Maven 构建系统提供了一个 [`protobuf-maven-plugin`](https://web.archive.org/web/20220626205430/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.xolstice.maven.plugins%22%20AND%20a%3A%22protobuf-maven-plugin%22) :

```java
<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <artifactId>os-maven-plugin</artifactId>
      <version>1.6.1</version>
    </extension>
  </extensions>
  <plugins>
    <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.6.1</version>
      <configuration>
        <protocArtifact>
          com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}
        </protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>
          io.grpc:protoc-gen-grpc-java:1.4.0:exe:${os.detected.classifier}
        </pluginArtifact>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>compile-custom</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

[os-maven-plugin](https://web.archive.org/web/20220626205430/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22kr.motd.maven%22%20AND%20a%3A%22os-maven-plugin%22) 扩展/插件生成各种有用的平台相关项目属性，如`${os.detected.classifier}`

## 6。创建服务器

无论您使用哪种方法生成代码，都将生成以下密钥文件:

*   `HelloRequest.java –` 包含了`HelloRequest`的类型定义
*   `HelloResponse.java **–**` 这包含了`HelleResponse`类型定义
*   这包含了抽象类`HelloServiceImplBase`,它提供了我们在服务接口中定义的所有操作的实现

### 6.1。覆盖服务基类

抽象类`HelloServiceImplBase`的**默认实现是抛出运行时异常** `io.grpc.StatusRuntimeException`，表示该方法未实现。

我们将扩展这个类并覆盖我们的服务定义中提到的`hello()`方法:

```java
public class HelloServiceImpl extends HelloServiceImplBase {

    @Override
    public void hello(
      HelloRequest request, StreamObserver<HelloResponse> responseObserver) {

        String greeting = new StringBuilder()
          .append("Hello, ")
          .append(request.getFirstName())
          .append(" ")
          .append(request.getLastName())
          .toString();

        HelloResponse response = HelloResponse.newBuilder()
          .setGreeting(greeting)
          .build();

        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

如果我们将`hello()`的签名与我们在`HellService.proto`文件中写的签名进行比较，我们会注意到它没有返回`HelloResponse`。相反，它将第二个参数作为`StreamObserver<HelloResponse>`，这是一个响应观察器，是服务器调用其响应的回调。

通过这种方式**，客户端可以选择进行阻塞呼叫还是非阻塞呼叫**。

gRPC 使用构建器来创建对象。我们使用`HelloResponse.newBuilder()`并设置问候文本来构建一个`HelloResponse`对象。我们将该对象设置为 responseObserver 的`onNext()`方法，以将其发送给客户端。

最后，我们需要调用`onCompleted()`来指定我们已经完成了对 RPC 的处理，否则连接将被挂起，客户端将只是等待更多信息的到来。

### 6.2。运行 Grpc 服务器

接下来，我们需要启动 gRPC 服务器来监听传入的请求:

```java
public class GrpcServer {
    public static void main(String[] args) {
        Server server = ServerBuilder
          .forPort(8080)
          .addService(new HelloServiceImpl()).build();

        server.start();
        server.awaitTermination();
    }
}
```

这里，我们再次使用构建器在端口 8080 上创建一个 gRPC 服务器，并添加我们定义的`HelloServiceImpl`服务。`start()`会启动服务器。在我们的例子中，我们将调用`awaitTermination()`来保持服务器在前台运行，阻止提示。

## 7 .**。创建客户端**

gRPC 提供了一个通道构造，它抽象出底层细节，如连接、连接池、负载平衡等。

我们将使用`ManagedChannelBuilder`创建一个频道。这里，我们指定服务器地址和端口。

我们将使用没有任何加密的纯文本:

```java
public class GrpcClient {
    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080)
          .usePlaintext()
          .build();

        HelloServiceGrpc.HelloServiceBlockingStub stub 
          = HelloServiceGrpc.newBlockingStub(channel);

        HelloResponse helloResponse = stub.hello(HelloRequest.newBuilder()
          .setFirstName("Baeldung")
          .setLastName("gRPC")
          .build());

        channel.shutdown();
    }
}
```

接下来，我们需要创建一个存根，我们将使用它对`hello()`进行实际的远程调用。**存根是客户端与服务器交互的主要方式。**当使用自动生成的存根时，存根类将有包装通道的构造函数。

这里我们使用一个阻塞/同步存根，以便 RPC 调用等待服务器响应，并返回一个响应或引发一个异常。gRPC 还提供了另外两种类型的存根，它们有助于非阻塞/异步调用。

最后，是时候进行`hello()` RPC 调用了。这里我们通过`HelloRequest`。我们可以使用自动生成的设置器来设置`HelloRequest`对象的`firstName`、`lastName`属性。

我们取回从服务器返回的`HelloResponse`对象。

## 8。结论

在本教程中，我们看到了如何使用 gRPC 来简化两个服务之间的通信开发，方法是专注于定义服务并让 gRPC 处理所有的样板代码。

像往常一样，你会在 GitHub 上找到这些资源[。](https://web.archive.org/web/20220626205430/https://github.com/eugenp/tutorials/tree/master/grpc)