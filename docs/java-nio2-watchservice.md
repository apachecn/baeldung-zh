# Java NIO2 中的 WatchService 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio2-watchservice>

## 1。概述

在本文中，我们将探索 Java NIO.2 文件系统 API 的`WatchService`接口。这是 Java 7 中与`FileVisitor`接口一起引入的较新 IO APIs 的鲜为人知的特性之一。

要在您的应用程序中使用`WatchService`接口，您需要导入适当的类:

```java
import java.nio.file.*;
```

## 2。为什么用`WatchService`

理解服务功能的一个常见例子是 IDE。

您可能已经注意到，IDEs 总是**检测源代码文件**中发生的外部变化。有些 IDE 使用对话框通知您，以便您可以选择是否从文件系统重新加载文件，有些 IDE 只是在后台更新文件。

类似地，较新的框架，比如 Play，默认情况下也可以热重新加载应用程序代码——无论何时您在任何编辑器中执行编辑。

这些应用程序采用了一种称为 **`file change notification`** 的特性，这种特性在所有文件系统中都可用。

基本上，**我们可以编写代码来轮询文件系统中特定文件和目录的变化**。然而，这种解决方案是不可扩展的，尤其是当文件和目录达到成百上千时。

在 Java 7 NIO.2 中，`WatchService` API 提供了一个可伸缩的解决方案来监控目录的变化。它有一个干净的 API，性能优化得非常好，我们不需要实现自己的解决方案。

## 3。Watchservice 是如何工作的？

要使用`WatchService`特性，第一步是使用`java.nio.file.FileSystems`类创建一个`WatchService`实例:

```java
WatchService watchService = FileSystems.getDefault().newWatchService();
```

接下来，我们必须创建我们想要监视的目录的路径:

```java
Path path = Paths.get("pathToDir");
```

在这一步之后，我们必须向 watch 服务注册路径。在这个阶段，有两个重要的概念需要理解。`StandardWatchEventKinds`级和`WatchKey`级。看一下下面的注册码，就可以了解每个人的位置。我们将对此进行同样的解释:

```java
WatchKey watchKey = path.register(
  watchService, StandardWatchEventKinds...);
```

这里只注意两件重要的事情:首先，路径注册 API 调用将 watch 服务实例作为第一个参数，后面是变量参数`StandardWatchEventKinds`。其次，注册过程的返回类型是一个`WatchKey`实例。

### 3.1。 `**StandardWatchEventKinds**`

这是一个类，它的实例告诉观察服务在注册的目录上要观察的事件的种类。目前有四种可能的事件需要关注:

*   `**StandardWatchEventKinds.ENTRY_CREATE**`–在监视目录中创建新条目时触发。这可能是由于创建了新文件或重命名了现有文件。
*   `**StandardWatchEventKinds.ENTRY_MODIFY**`–当监视目录中的现有条目被修改时触发。所有文件编辑都会触发此事件。在某些平台上，甚至更改文件属性也会触发它。
*   `**StandardWatchEventKinds.ENTRY_DELETE**`–在监视目录中删除、移动或重命名条目时触发。
*   `**StandardWatchEventKinds.OVERFLOW**`–触发以指示丢失或丢弃的事件。我们不会太关注它

### 3.2。 `**WatchKey**`

此类别代表向 watch 服务注册目录。当我们注册一个目录时，以及当我们询问 watch 服务我们注册的事件是否已经发生时，它的实例被 watch 服务返回给我们。

Watch 服务不提供任何回调方法，只要有事件发生，就会调用这些方法。我们只能通过多种方式来查询这些信息。

我们可以使用`poll` API:

```java
WatchKey watchKey = watchService.poll();
```

这个 API 调用会立即返回。它返回下一个已发生事件的队列中的观察键，如果没有已注册的事件发生，则返回 null。

我们还可以使用一个带`timeout`参数的重载版本:

```java
WatchKey watchKey = watchService.poll(long timeout, TimeUnit units);
```

这个 API 调用在返回值方面类似于上一个。然而，它阻塞了`timeout`个时间单位，以给出更多的时间让事件发生，而不是立即返回 null。

最后，我们可以使用`take` API:

```java
WatchKey watchKey = watchService.take();
```

最后一种方法只是阻塞，直到事件发生。

这里我们必须注意一些非常重要的事情:**当`WatchKey`实例被`poll`或`take`API 返回时，如果它的重置 API 没有被调用，它将不会捕获更多的事件:**

```java
watchKey.reset();
```

这意味着每次轮询操作返回监视键实例时，都会将其从监视服务队列中删除。API 调用将它放回队列中等待更多的事件。

watcher 服务最实际的应用需要一个循环，在这个循环中，我们不断地检查被监视目录中的变化，并相应地进行处理。我们可以使用下面的习语来实现这一点:

```java
WatchKey key;
while ((key = watchService.take()) != null) {
    for (WatchEvent<?> event : key.pollEvents()) {
        //process
    }
    key.reset();
}
```

我们创建一个 watch 键来存储轮询操作的返回值。while 循环将一直阻塞，直到条件语句返回 watch 键或 null。

当我们得到一个手表键时，while 循环就执行其中的代码。我们使用`WatchKey.pollEvents` API 返回已经发生的事件列表。然后我们使用一个`for each`循环来逐个处理它们。

在处理完所有事件之后，我们必须调用`reset` API 来再次将 watch 键入队。

## 4。目录监视示例

由于我们已经在前一小节中介绍了`WatchService` API，以及它的内部工作方式和我们如何使用它，我们现在可以继续看一个完整的实用示例。

出于可移植性的原因，我们将观察用户主目录中的活动，这在所有现代操作系统上都应该是可用的。

该代码只包含几行代码，所以我们只将它保留在 main 方法中:

```java
public class DirectoryWatcherExample {

    public static void main(String[] args) {
        WatchService watchService
          = FileSystems.getDefault().newWatchService();

        Path path = Paths.get(System.getProperty("user.home"));

        path.register(
          watchService, 
            StandardWatchEventKinds.ENTRY_CREATE, 
              StandardWatchEventKinds.ENTRY_DELETE, 
                StandardWatchEventKinds.ENTRY_MODIFY);

        WatchKey key;
        while ((key = watchService.take()) != null) {
            for (WatchEvent<?> event : key.pollEvents()) {
                System.out.println(
                  "Event kind:" + event.kind() 
                    + ". File affected: " + event.context() + ".");
            }
            key.reset();
        }
    }
}
```

这就是我们真正要做的。现在，您可以运行该类来开始监视一个目录。

当您导航到用户主目录并执行任何文件操作活动(如创建文件或目录、更改文件内容甚至删除文件)时，这些都将记录在控制台中。

例如，假设您进入用户主页，右键单击空白处，选择``new – > file``创建一个新文件，然后将其命名为`testFile`。然后你添加一些内容并保存。控制台的输出如下所示:

```java
Event kind:ENTRY_CREATE. File affected: New Text Document.txt.
Event kind:ENTRY_DELETE. File affected: New Text Document.txt.
Event kind:ENTRY_CREATE. File affected: testFile.txt.
Event kind:ENTRY_MODIFY. File affected: testFile.txt.
Event kind:ENTRY_MODIFY. File affected: testFile.txt.
```

您可以随意编辑路径，使其指向您想要监视的任何目录。

## 5。结论

在本文中，我们探讨了 Java 7 NIO.2 中一些不常用的特性——文件系统 API，尤其是`WatchService`接口。

我们还设法完成了构建目录监视应用程序的步骤，以演示其功能。

和往常一样，本文中使用的示例的完整源代码可以在 Github 项目中获得。