# Java 中的 BufferedReader vs 控制台 vs 扫描器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/bufferedreader-vs-console-vs-scanner-in-java>

## 1.概观

在这篇文章中，我们将通过 Java 中的`BufferedReader`、`Console`和`Scanner`类来了解它们之间的区别。

为了深入了解每个主题，我们建议看一下我们关于 [Java Scanner](/web/20220524031335/https://www.baeldung.com/java-scanner) 、[Java 控制台 I/O](/web/20220524031335/https://www.baeldung.com/java-console-input-output)和 [BufferedReader](/web/20220524031335/https://www.baeldung.com/java-buffered-reader) 的文章。

## 2.用户输入

给定传递给构造函数的底层流，**`BufferedReader`和`Scanner`类都能够处理更大范围的用户输入**，比如字符串、文件、系统控制台(通常连接到键盘)和套接字。

另一方面，`Console`类被设计成只访问与当前 Java 虚拟机相关联的基于字符的系统控制台(如果有的话)。

让我们看看`BufferedReader`构造函数，它们接受不同的输入:

```java
BufferedReader br = new BufferedReader(
  new StringReader("Bufferedreader vs Console vs Scanner in Java"));
BufferedReader br = new BufferedReader(
  new FileReader("file.txt"));
BufferedReader br = new BufferedReader(
  new InputStreamReader(System.in))

Socket socket = new Socket(hostName, portNumber);
BufferedReader br =  new BufferedReader(
  new InputStreamReader(socket.getInputStream())); 
```

类似地，`Scanner`类也可以在其构造函数中接受不同的输入:

```java
Scanner sc = new Scanner("Bufferedreader vs Console vs Scanner in Java")
Scanner sc = new Scanner(new File("file.txt"));
Scanner sc = new Scanner(System.in);

Socket socket = new Socket(hostName, portNumber);
Scanner sc =  new Scanner(socket.getInputStream());
```

`Console`类只能通过方法调用获得:

```java
Console console = System.console();
```

请记住，当我们使用`Console`类时，如果我们在诸如 Eclipse 或 IntelliJ IDEA 之类的 IDE 中运行代码，JVM 关联的系统控制台是不可用的。

## 3.用户输出

**与不向输出流写入任何内容的`BufferedReader`和`Scanner`类相反，`Console`类提供了一些方便的方法**，如`readPassword (String fmt, Object… args), readLine (String fmt, Object… args),` 和`printf (String format,Object… args)`、**来将提示写入系统控制台的输出流**:

```java
String firstName = console.readLine("Enter your first name please: ");
console.printf("Welcome " + firstName );
```

所以当我们编写程序与系统控制台交互时，`Console`类会通过删除不必要的`System.out.println`来简化代码。

## 4.解析输入

**`Scanner`类可以使用正则表达式**解析原始类型和字符串。

它使用自定义分隔符模式将其输入分解成标记，默认情况下匹配空格:

```java
String input = "Bufferedreader vs Console vs Scanner";
Scanner sc = new Scanner(input).useDelimiter("\\s*vs\\s*");
System.out.println(sc.next());
System.out.println(sc.next());
System.out.println(sc.next());
sc.close();
```

`BufferredReader`和`Console`类只是照原样读取输入流。

## 5.读取安全数据

`Console`类有方法`readPassword()`和`readPassword (String` fmt `, Object… args) `在回显被禁用的情况下读取安全数据，因此用户不会看到他们正在键入的内容:

```java
String password = String.valueOf(console.readPassword("Password :")); 
```

**`BufferedReader`和`Scanner`没有能力这样做。**

## 6.线程安全

`BufferedReader`中的读取方法和`Console`中的读写方法都是`synchronized`，而`Scanner`类中的则不是。如果我们在多线程程序中读取用户输入，那么`BufferedReader`或`Console`将是更好的选择。

## 7.缓冲区大小

**与`Scanner`类**中的 1 KB 相比，`BufferedReader`中的缓冲区大小为 8 KB。

另外，如果需要，我们可以在`the BufferedReader`类的构造函数中指定缓冲区大小。这将有助于从用户输入中读取长字符串。 **`Console`类从系统控制台**读取时没有缓冲区，但它有一个缓冲的输出流写入系统控制台。

## 8.多方面的

在各种情况下选择合适的类时，有一些差异不是我们考虑的主要因素。

### 8.1.关闭输入流

一旦我们创建了`BufferedReader`或`Scanner`的实例，我们需要**记得关闭它以避免内存泄漏**。但是这不会发生在`Console`类上——我们不需要在使用后关闭系统控制台。

### 8.2.异常处理

当`Scanner`和`Console`使用未检查异常方法时，`BufferedReader`中的方法抛出检查异常，这迫使我们编写样板 try-catch 语法来处理异常。

## 9.结论

既然我们已经说明了这些类别之间的差异，那么让我们来总结一些关于哪种类别最适合处理不同情况的经验法则:

*   如果我们需要从文件中读取长字符串，使用`BufferedReader`，因为它比`Scanner`有更好的性能
*   考虑一下`Console`如果我们从系统控制台读取安全数据，并且想要隐藏正在键入的内容
*   如果我们需要用定制的正则表达式解析输入流，使用`Scanner`
*   当我们与系统控制台交互时，`Scanner`将是首选，因为它提供了读取和解析输入流的细粒度方法。此外，性能缺陷并不是一个大问题，因为在大多数情况下，`nextXXX`方法会阻塞并等待手动输入
*   在线程安全的上下文中，考虑`BufferedReader`，除非我们必须使用特定于`Console`类的特性