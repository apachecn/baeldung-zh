# Java 9 中的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/new-java-9>

[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220824083946/https://www.baeldung.com/java-8-new-features)
• New Features in Java 9 (current article)[• New Features in Java 10](/web/20220824083946/https://www.baeldung.com/java-10-overview)
[• New Features in Java 11](/web/20220824083946/https://www.baeldung.com/java-11-new-features)
[• New Features in Java 12](/web/20220824083946/https://www.baeldung.com/java-12-new-features)
[• New Features in Java 13](/web/20220824083946/https://www.baeldung.com/java-13-new-features)
[• New Features in Java 14](/web/20220824083946/https://www.baeldung.com/java-14-new-features)
[• What's New in Java 15](/web/20220824083946/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20220824083946/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220824083946/https://www.baeldung.com/java-17-new-features)

## 1。概述

Java 9 具有丰富的特性集。虽然没有新的语言概念，但新的 API 和诊断命令肯定会引起开发人员的兴趣。

在这篇文章中，我们将快速、深入地了解一些新特性；新特性的完整列表可在[这里](https://web.archive.org/web/20220824083946/https://openjdk.java.net/projects/jdk9/)获得。

## 2。模块化系统——拼图项目

让我们从大的开始——将模块化引入 Java 平台。

模块化系统提供类似于 OSGi 框架系统的功能。模块有一个依赖的概念，可以导出一个公共 API，并保持实现细节的隐藏/私有。

这里的一个主要动机是提供模块化 JVM，它可以在可用内存少得多的设备上运行。JVM 可以只运行应用程序需要的那些模块和 API。查看[此链接](https://web.archive.org/web/20220824083946/http://cr.openjdk.java.net/~mr/jigsaw/ea/module-summary.html)了解这些模块的描述。

此外，像`com.sun.*`这样的 JVM 内部(实现)API 不再可以从应用程序代码中访问。

简单地说，模块将在位于 java 代码层次结构顶层的名为`module-info.java` 的文件中进行描述:

```
module com.baeldung.java9.modules.car {
    requires com.baeldung.java9.modules.engines;
    exports com.baeldung.java9.modules.car.handling;
} 
```

我们的模块`car`要求模块`engine`运行并为`handling`导出一个包。

有关更深入的示例，请查看 OpenJDK [Project Jigsaw:模块系统快速入门指南](https://web.archive.org/web/20220824083946/https://openjdk.java.net/projects/jigsaw/quick-start)。

## 3。新的 HTTP 客户端

期待已久的旧`HttpURLConnection`的替代品。

**新的 API 位于`java.net.http`包下。**

它应该同时支持 [HTTP/2 协议](https://web.archive.org/web/20220824083946/https://http2.github.io/)和 [WebSocket](https://web.archive.org/web/20220824083946/https://en.wikipedia.org/wiki/WebSocket) 握手，性能应该与 [Apache HttpClient](https://web.archive.org/web/20220824083946/https://hc.apache.org/httpcomponents-client-ga/) 、 [Netty](https://web.archive.org/web/20220824083946/http://netty.io/) 和 [Jetty](https://web.archive.org/web/20220824083946/https://eclipse.org/jetty) 相当。

让我们通过创建和发送一个简单的 HTTP 请求来看看这个新功能。

**更新: [HTTP 客户端 JEP](https://web.archive.org/web/20220824083946/https://openjdk.java.net/jeps/110) 正在被移动到孵化器模块，所以它不再出现在`java.net.http`包中，而是出现在`jdk.incubator.http.`T5 下**

### 3.1。快速获取请求

API 使用构建器模式，这使得快速使用变得非常容易:

```
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/get"))
  .GET()
  .build();

HttpResponse<String> response = HttpClient.newHttpClient()
  .send(request, HttpResponse.BodyHandler.asString()); 
```

## 4。流程 API

process API 已经过改进，可用于控制和管理操作系统进程。

### 4.1。过程信息

类别`java.lang.ProcessHandle` 包含大多数新功能:

```
ProcessHandle self = ProcessHandle.current();
long PID = self.getPid();
ProcessHandle.Info procInfo = self.info();

Optional<String[]> args = procInfo.arguments();
Optional<String> cmd =  procInfo.commandLine();
Optional<Instant> startTime = procInfo.startInstant();
Optional<Duration> cpuUsage = procInfo.totalCpuDuration();
```

`current` 方法返回一个代表当前运行的 JVM 进程的对象。`Info`子类提供了关于流程的细节。

### 4.2。销毁过程

现在，让我们使用`destroy()`停止所有正在运行的子进程:

```
childProc = ProcessHandle.current().children();
childProc.forEach(procHandle -> {
    assertTrue("Could not kill process " + procHandle.getPid(), procHandle.destroy());
});
```

## 5。小语种修改

### 5.1。尝试资源

在 Java 7 中，`try-with-resources`语法要求为语句管理的每个资源声明一个新变量。

在 Java 9 中，有一个额外的改进:如果资源被 final 或 effectively final 变量引用，try-with-resources 语句可以管理资源，而无需声明新的变量:

```
MyAutoCloseable mac = new MyAutoCloseable();
try (mac) {
    // do some stuff with mac
}

try (new MyAutoCloseable() { }.finalWrapper.finalCloseable) {
   // do some stuff with finalCloseable
} catch (Exception ex) { } 
```

### 5.2。菱形运算符扩展

现在我们可以将菱形运算符与匿名内部类结合使用:

```
FooClass<Integer> fc = new FooClass<>(1) { // anonymous inner class
};

FooClass<? extends Integer> fc0 = new FooClass<>(1) { 
    // anonymous inner class
};

FooClass<?> fc1 = new FooClass<>(1) { // anonymous inner class
}; 
```

### 5.3。接口私有方法

即将到来的 JVM 版本中的接口可以有`private`方法，这些方法可以用来拆分冗长的默认方法:

```
interface InterfaceWithPrivateMethods {

    private static String staticPrivate() {
        return "static private";
    }

    private String instancePrivate() {
        return "instance private";
    }

    default void check() {
        String result = staticPrivate();
        InterfaceWithPrivateMethods pvt = new InterfaceWithPrivateMethods() {
            // anonymous class
        };
        result = pvt.instancePrivate();
    }
}}
```

## 6。JShell 命令行工具

**JShell 简称为 read-eval-print loop-REPL。**

简单地说，它是一个交互式工具，可以评估 Java 的声明、语句和表达式，以及 API。这对于测试小代码片段非常方便，否则就需要用`main`方法创建一个新类。

`jshell`可执行文件本身可以在`<JAVA_HOME>/bin` 文件夹中找到:

```
jdk-9\bin>jshell.exe
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro
jshell> "This is my long string. I want a part of it".substring(8,19);
$5 ==> "my long string"
```

交互式 shell 带有历史和自动完成功能；它还提供保存和加载文件、所有或部分书面陈述等功能:

```
jshell> /save c:\develop\JShell_hello_world.txt
jshell> /open c:\develop\JShell_hello_world.txt
Hello JShell! 
```

代码片段在文件加载时执行。

## 7 .**。JCMD 子命令**

让我们探索一下`jcmd` 命令行实用程序中的一些新的子命令。我们将获得 JVM 中加载的所有类及其继承结构的列表。

在下面的例子中，我们可以看到运行 Eclipse Neon 的 JVM 中加载的`java.lang.Socket`的层次结构:

```
jdk-9\bin>jcmd 14056 VM.class_hierarchy -i -s java.net.Socket
14056:
java.lang.Object/null
|--java.net.Socket/null
|  implements java.io.Closeable/null (declared intf)
|  implements java.lang.AutoCloseable/null (inherited intf)
|  |--org.eclipse.ecf.internal.provider.filetransfer.httpclient4.CloseMonitoringSocket
|  |  implements java.lang.AutoCloseable/null (inherited intf)
|  |  implements java.io.Closeable/null (inherited intf)
|  |--javax.net.ssl.SSLSocket/null
|  |  implements java.lang.AutoCloseable/null (inherited intf)
|  |  implements java.io.Closeable/null (inherited intf) 
```

`jcmd` 命令的第一个参数是我们想要在其上运行命令的 JVM 的进程 id (PID)。

另一个有趣的子命令是`set_vmflag`。我们可以在线修改一些 JVM 参数，而不需要重启 JVM 进程和修改它的启动参数。

您可以使用子命令`jcmd 14056 VM.flags -all`找出所有可用的虚拟机标志

## 8。м多分辨率图像 API

接口`java.awt.image.MultiResolutionImage`将一组不同分辨率的图像封装成一个对象。我们可以基于给定的 DPI 度量和一组图像变换来检索特定分辨率的图像变体，或者检索图像中的所有变体。

`java.awt.Graphics`类基于当前显示 DPI 度量和任何应用的变换从多分辨率图像中获取变量。

类`java.awt.image.BaseMultiResolutionImage` 提供了基本的实现:

```
BufferedImage[] resolutionVariants = ....
MultiResolutionImage bmrImage
  = new BaseMultiResolutionImage(baseIndex, resolutionVariants);
Image testRVImage = bmrImage.getResolutionVariant(16, 16);
assertSame("Images should be the same", testRVImage, resolutionVariants[3]); 
```

## 9。可变手柄

API 位于`java.lang.invoke`下，由`VarHandle`和`MethodHandles`组成。它提供了对对象字段和数组元素的等效的`java.util.concurrent.atomic`和`sun.misc.Unsafe`操作，具有相似的性能。

**使用 Java 9 模块化系统，从应用程序代码访问`sun.misc.Unsafe`将是不可能的。**

## 10。发布-订阅框架

类`java.util.concurrent.Flow`提供了支持[反应流](https://web.archive.org/web/20220824083946/http://www.reactive-streams.org/)发布-订阅框架的接口。这些接口支持在 JVM 上运行的许多异步系统之间的互操作性。

我们可以使用实用程序类`SubmissionPublisher`来创建定制组件。

## 11。统一 JVM 日志记录

这个特性为 JVM 的所有组件引入了一个通用的日志系统。它提供了进行日志记录的基础设施，但是它没有添加来自所有 JVM 组件的实际日志记录调用。它也没有向 JDK 中的 Java 代码添加日志记录。

日志框架定义了一组`tags`——例如`gc`、`compiler`、`threads`等。我们可以使用命令行参数`-Xlog` 在启动时打开日志记录。

让我们使用“调试”级别将带有“gc”标记的消息记录到一个名为“gc.txt”的无修饰文件中:

```
java -Xlog:gc=debug:file=gc.txt:none ...
```

`-Xlog:help`将输出可能的选项和例子。运行时可以使用`jcmd`命令修改记录配置。我们将把 GC 日志设置为 info，并将它们重定向到一个文件——GC _ logs:

```
jcmd 9615 VM.log output=gc_logs what=gc
```

## 12。新的 API

### 12.1。不可变集合

`java.util.Set.of()`–创建给定元素的不可变集合。在 Java 8 中，创建一组元素需要几行代码。现在我们可以简单地做到:

```
Set<String> strKeySet = Set.of("key1", "key2", "key3");
```

该方法返回的`Set`是 JVM 内部类:`java.util.ImmutableCollections.SetN`，它扩展了 public `java.util.AbstractSet`。它是不可变的——如果我们试图添加或删除元素，就会抛出一个`UnsupportedOperationException`。

你也可以用同样的方法将整个数组转换成一个`Set`。

### 12.2。可选流

`java.util.Optional.stream()`为我们提供了一种简单的方法来在可选元素上使用 Streams 的功能:

```
List<String> filteredList = listOfOptionals.stream()
  .flatMap(Optional::stream)
  .collect(Collectors.toList()); 
```

## 13。结论

Java 9 将带有一个模块化的 JVM 和许多其他新的不同的改进和特性。

你可以在 GitHub 上找到示例[的源代码。](https://web.archive.org/web/20220824083946/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)

Next **»**[New Features in Java 10](/web/20220824083946/https://www.baeldung.com/java-10-overview)**«** Previous[New Features in Java 8](/web/20220824083946/https://www.baeldung.com/java-8-new-features)