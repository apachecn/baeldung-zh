# 使用 Apache Thrift

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-thrift>

## 1。概述

在本文中，我们将发现如何借助名为 [Apache Thrift](https://web.archive.org/web/20220625173214/https://thrift.apache.org/) 的 RPC 框架开发跨平台的客户机-服务器应用程序。

我们将涵盖:

*   用 IDL 定义数据类型和服务接口
*   安装库并生成不同语言的源代码
*   用特定语言实现定义的接口
*   实施客户机/服务器软件

如果你想直接看例子，直接看第 5 部分。

## 2。阿帕奇节俭

Apache Thrift 最初是由脸书开发团队开发的，目前由 Apache 维护。

与管理跨平台对象序列化/反序列化过程的[协议缓冲区](/web/20220625173214/https://www.baeldung.com/spring-rest-api-with-protocol-buffers)相比， **Thrift 主要关注系统组件之间的通信层。**

Thrift 使用一种特殊的接口描述语言(IDL)来定义数据类型和服务接口，这些数据类型和服务接口被存储为`.thrift`文件，以后作为编译器的输入，用于生成通过不同编程语言进行通信的客户端和服务器软件的源代码。

要在您的项目中使用 Apache Thrift，请添加这个 Maven 依赖项:

```
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.10.0</version>
</dependency>
```

你可以在 [Maven 资源库](https://web.archive.org/web/20220625173214/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.thrift%22%20AND%20a%3A%22libthrift%22)中找到最新版本。

## 3。界面描述语言

如前所述， [IDL](https://web.archive.org/web/20220625173214/https://thrift.apache.org/docs/idl) 允许用中性语言定义通信接口。下面你会发现目前支持的类型。

### 3.1。基本类型

*   `bool`–布尔值(真或假)
*   `byte`–一个 8 位有符号整数
*   `i16`–16 位有符号整数
*   `i32`–32 位有符号整数
*   `i64`–64 位有符号整数
*   `double`–64 位浮点数
*   `string`–使用 UTF-8 编码的文本字符串

### 3.2。特殊类型

*   `binary`–未编码的字节序列
*   `optional`–Java 8 的`Optional`类型

### 3.3。结构

Thrift `structs`相当于 OOP 语言中的类，但是没有继承。一个`struct`有一组强类型字段，每个字段都有一个唯一的名称作为标识符。字段可能有各种注释(数字字段 id、可选默认值等。).

### 3.4。集装箱

节俭容器是强类型容器:

*   `list`–元素的有序列表
*   `set`–一组无序的独特元素
*   `map<type1,type2>`–值的严格唯一键的映射

容器元素可以是任何有效的节俭类型。

### 3.5。例外情况

异常在功能上等同于`structs`，除了它们继承自本地异常。

### 3.6。服务

服务实际上是使用节俭类型定义的通信接口。它们由一组命名函数组成，每个函数都有一个参数列表和一个返回类型。

## 4。源代码生成

### 4.1。语言支持

目前支持的语言有一长串:

*   C++
*   C#
*   去
*   哈斯克尔
*   Java 语言(一种计算机语言，尤用于创建网站)
*   java 描述语言
*   节点. js
*   Perl 语言
*   服务器端编程语言（Professional Hypertext Preprocessor 的缩写）
*   计算机编程语言
*   红宝石

你可以点击查看完整列表[。](https://web.archive.org/web/20220625173214/https://thrift.apache.org/lib/)

### 4.2。使用库的可执行文件

只需下载[的最新版本](https://web.archive.org/web/20220625173214/https://thrift.apache.org/download)，如有必要，编译并安装它，并使用以下语法:

```
cd path/to/thrift
thrift -r --gen [LANGUAGE] [FILENAME]
```

在上面的命令集中，`[LANGUAGE]`是支持的语言之一，`[FILENAME`是一个带有 IDL 定义的文件。

注意`-r`标志。一旦发现给定的`.thrift`文件中包含 includes，它就告诉 Thrift 递归地生成代码。

### 4.3。使用 Maven 插件

将插件添加到您的`pom.xml`文件中:

```
<plugin>
   <groupId>org.apache.thrift.tools</groupId>
   <artifactId>maven-thrift-plugin</artifactId>
   <version>0.1.11</version>
   <configuration>
      <thriftExecutable>path/to/thrift</thriftExecutable>
   </configuration>
   <executions>
      <execution>
         <id>thrift-sources</id>
         <phase>generate-sources</phase>
         <goals>
            <goal>compile</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

之后，只需执行以下命令:

```
mvn clean install
```

请注意，这个插件将不再有任何进一步的维护。请访问[页面](https://web.archive.org/web/20220625173214/https://github.com/dtrott/maven-thrift-plugin)了解更多信息。

## 5。客户端-服务器应用示例

### 5.1。定义节俭文件

让我们编写一些带有异常和结构的简单服务:

```
namespace cpp com.baeldung.thrift.impl
namespace java com.baeldung.thrift.impl

exception InvalidOperationException {
    1: i32 code,
    2: string description
}

struct CrossPlatformResource {
    1: i32 id,
    2: string name,
    3: optional string salutation
}

service CrossPlatformService {

    CrossPlatformResource get(1:i32 id) throws (1:InvalidOperationException e),

    void save(1:CrossPlatformResource resource) throws (1:InvalidOperationException e),

    list <CrossPlatformResource> getList() throws (1:InvalidOperationException e),

    bool ping() throws (1:InvalidOperationException e)
}
```

如您所见，语法非常简单，一目了然。我们定义了一组名称空间(每种实现语言)，一个异常类型，一个结构，最后是一个服务接口，它将在不同的组件之间共享。

然后只需将其存储为一个`service.thrift`文件。

### 5.2。编译并生成代码

现在是运行编译器的时候了，它会为我们生成代码:

```
thrift -r -out generated --gen java /path/to/service.thrift
```

正如您可能看到的，我们添加了一个特殊的标志`-out`来指定生成文件的输出目录。如果您没有得到任何错误，`generated`目录将包含 3 个文件:

*   `CrossPlatformResource.java`
*   `CrossPlatformService.java`
*   `InvalidOperationException.java`

让我们通过运行以下命令来生成服务的 C++版本:

```
thrift -r -out generated --gen cpp /path/to/service.thrift
```

现在我们得到了同一个服务接口的两个不同的有效实现(Java 和 C++)。

### 5.3。添加服务实现

尽管 Thrift 已经为我们做了大部分工作，我们仍然需要编写自己的`CrossPlatformService`实现。为了做到这一点，我们只需要实现一个`CrossPlatformService.Iface`接口:

```
public class CrossPlatformServiceImpl implements CrossPlatformService.Iface {

    @Override
    public CrossPlatformResource get(int id) 
      throws InvalidOperationException, TException {
        return new CrossPlatformResource();
    }

    @Override
    public void save(CrossPlatformResource resource) 
      throws InvalidOperationException, TException {
        saveResource();
    }

    @Override
    public List<CrossPlatformResource> getList() 
      throws InvalidOperationException, TException {
        return Collections.emptyList();
    }

    @Override
    public boolean ping() throws InvalidOperationException, TException {
        return true;
    }
}
```

### 5.4。编写服务器

正如我们所说的，我们想要构建一个跨平台的客户机-服务器应用程序，所以我们需要一个服务器。Apache Thrift 的伟大之处在于它有自己的客户机-服务器通信框架，这使得通信变得轻而易举:

```
public class CrossPlatformServiceServer {
    public void start() throws TTransportException {
        TServerTransport serverTransport = new TServerSocket(9090);
        server = new TSimpleServer(new TServer.Args(serverTransport)
          .processor(new CrossPlatformService.Processor<>(new CrossPlatformServiceImpl())));

        System.out.print("Starting the server... ");

        server.serve();

        System.out.println("done.");
    }

    public void stop() {
        if (server != null && server.isServing()) {
            System.out.print("Stopping the server... ");

            server.stop();

            System.out.println("done.");
        }
    }
} 
```

首先是用`TServerTransport`接口的实现(或者更精确地说，抽象类)定义一个传输层。既然我们在谈论服务器，我们需要提供一个端口来监听。然后我们需要定义一个`TServer`实例，并选择一个可用的实现:

*   `TSimpleServer`–用于简单服务器
*   `TThreadPoolServer`–针对多线程服务器
*   `TNonblockingServer`–适用于非阻塞多线程服务器

最后，为所选的服务器提供一个处理器实现，该服务器已经由 Thrift 为我们生成，即`CrossPlatofformService.Processor`类。

### 5.5。编写客户端

这是客户端的实现:

```
TTransport transport = new TSocket("localhost", 9090);
transport.open();

TProtocol protocol = new TBinaryProtocol(transport);
CrossPlatformService.Client client = new CrossPlatformService.Client(protocol);

boolean result = client.ping();

transport.close();
```

从客户端的角度来看，这些操作非常相似。

首先，定义传输并将其指向我们的服务器实例，然后选择合适的协议。唯一的区别是，这里我们初始化了客户端实例，它也是由 Thrift 生成的，即`CrossPlatformService.Client`类。

因为它基于`.thrift`文件定义，我们可以直接调用那里描述的方法。在这个特殊的例子中，`client.ping()` 将远程调用服务器，服务器将使用`true`进行响应。

## 6。结论

在本文中，我们向您展示了使用 Apache Thrift 的基本概念和步骤，并展示了如何创建一个利用 Thrift 库的工作示例。

通常，所有的例子都可以在 GitHub 库的[中找到。](https://web.archive.org/web/20220625173214/https://github.com/eugenp/tutorials/tree/master/apache-thrift)