# java.lang.System 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lang-system>

## 1。概述

在本教程中，我们将快速浏览一下`java.lang.System`类及其特性和核心功能。

## 2。IO

`System`是`java.lang`的一部分，它的主要特性之一是让我们访问标准的 I/O 流。

简单地说，它公开了三个字段，每个流一个:

*   `out`
*   `err`
*   `in`

### 2.1。`System.out`

`System.out`指向标准输出流，将其公开为`PrintStream`，我们可以使用它将文本打印到控制台:

```java
System.out.print("some inline message");
```

`System`的一个高级用法是调用`System.setOut`，我们可以用它来定制`System.out`将写入的位置:

```java
// Redirect to a text file
System.setOut(new PrintStream("filename.txt"));
```

### 2.2。`System.err`

`System.err`很像`System.out`。这两个字段都是`PrintStream,` 的实例，都用于将消息打印到控制台。

但是`System.err`代表标准误差，我们专门用它来输出错误信息:

```java
System.err.print("some inline error message"); 
```

控制台呈现的错误流通常不同于输出流。

有关更多信息，请查看 [`PrintStream`](https://web.archive.org/web/20220628084400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/PrintStream.html) 文档。

### 2.3。`System.in`

`System.in`指向中的标准，将其公开为一个`InputStream,` ,我们可以使用它从控制台读取输入。

虽然参与程度有所提高，但我们仍然可以做到:

```java
public String readUsername(int length) throws IOException {
    byte[] name = new byte[length];
    System.in.read(name, 0, length); // by default, from the console
    return new String(name);
}
```

通过调用`System.in.read`，应用程序停止并等待标准输入。无论下一个`length` 字节是什么，都将从流中读取并存储在字节数组中。

**用户输入的任何其他东西都留在流**中，等待对`read.`的另一个调用

当然，在如此低的级别上操作可能具有挑战性并且容易出错，所以我们可以用`BufferedReader`来清理一下:

```java
public String readUsername() throws IOException {
    BufferedReader reader = new BufferedReader(
      new InputStreamReader(System.in));
    return reader.readLine();
}
```

有了上面的安排，`readLine`将从`System.in` 开始读取，直到用户点击 return，这与我们预期的有点接近。

注意，在这种情况下，我们故意不关闭流。**关闭标准`in`** **意味着在程序的生命周期内不能再次读取！**

最后，`System.in`的一个高级用法是调用`System.setIn`将其重定向到不同的`InputStream`。

## 3。实用方法

`System`为我们提供了许多帮助我们的方法，例如:

*   访问控制台
*   复制数组
*   观察日期和时间
*   退出 JRE
*   访问运行时属性
*   访问环境变量，以及
*   管理垃圾收集

### 3.1。访问控制台

Java 1.6 引入了另一种与控制台交互的方式，而不是简单地直接使用`System.out` 和`in` 。

我们可以通过调用`System.console`来访问它:

```java
public String readUsername() {
    Console console = System.console();	 	 

    return console == null ? null :	 
      console.readLine("%s", "Enter your name: ");	 	 
}
```

请注意，根据底层操作系统以及我们如何启动 Java 来运行当前程序， **`console`可能会返回`null,` ，因此在使用**之前，请务必进行检查。

查看[控制台](https://web.archive.org/web/20220628084400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/Console.html)文档了解更多使用方法。

### 3.2。复制数组

是一种古老的 C 风格的将一个数组复制到另一个数组的方法。

`arraycopy`主要是将一个完整的数组复制到另一个数组中:

```java
int[] a = {34, 22, 44, 2, 55, 3};
int[] b = new int[a.length];

System.arraycopy(a, 0, b, 0, a.length);
assertArrayEquals(a, b); 
```

但是，我们可以指定两个数组的起始位置，以及要复制多少个元素。

例如，假设我们想要从`a`复制 2 个元素，从`a[1]`开始到`b`，从`b[3]`开始:

```java
System.arraycopy(a, 1, b, 3, 2); 
assertArrayEquals(new int[] {0, 0, 0, 22, 44, 0}, b);
```

另外，记住`arraycopy` 会抛出:

*   `NullPointerException` 如果任一数组为`null`
*   `IndexOutOfBoundsException`如果副本引用了超出其范围的任一数组
*   如果复制导致类型不匹配

### 3.3。观察日期和时间

`System`中有两种与时间相关的方法。一个是`currentTimeMillis`，另一个是`nanoTime`。

`currentTimeMillis`返回自 Unix 纪元(1970 年 1 月 1 日 12:00 AM UTC:

```java
public long nowPlusOneHour() {
    return System.currentTimeMillis() + 3600 * 1000L;
}

public String nowPrettyPrinted() {
    return new Date(System.currentTimeMillis()).toString();
} 
```

`nanoTime`返回相对于 JVM 启动的时间。我们可以多次调用它来标记应用程序中的时间流逝:

```java
long startTime = System.nanoTime();
// ...
long endTime = System.nanoTime();

assertTrue(endTime - startTime < 10000); 
```

注意，由于`nanoTime` 是如此的细粒度，**由于数值溢出** 的[可能性，做`endTime – startTime < 10000` 比做`endTime < startTime`更安全。](https://web.archive.org/web/20220628084400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html#nanoTime())

### 3.4。退出程序

如果我们想以编程的方式退出当前正在执行的程序，`System.exit`将会做到这一点。

要调用`exit`，我们需要指定一个退出代码，它将被发送到启动程序的控制台或 shell。

按照 Unix 中的惯例，状态 0 表示正常退出，而非零表示发生了一些错误:

```java
if (error) {
    System.exit(1);
} else {
    System.exit(0);
}
```

注意，对于现在的大多数程序来说，需要调用这个函数是很奇怪的。例如，当在 web 服务器应用程序中被调用时，它可能会关闭整个网站！

### 3.5。访问运行时属性

`System`通过`getProperty`提供对运行时属性的访问。

我们可以用`setProperty` 和`clearProperty`来管理它们:

```java
public String getJavaVMVendor() {
    System.getProperty("java.vm.vendor");
}

System.setProperty("abckey", "abcvaluefoo");
assertEquals("abcvaluefoo", System.getProperty("abckey"));

System.clearProperty("abckey");
assertNull(System.getProperty("abckey"));
```

通过`-D`指定的属性可通过`getProperty`访问。

我们还可以提供一个默认值:

```java
System.clearProperty("dbHost");
String myKey = System.getProperty("dbHost", "db.host.com");
assertEquals("db.host.com", myKey);
```

并且`System.getProperties`提供了所有系统属性的集合:

```java
Properties properties = System.getProperties();
```

从中我们可以进行任何`Properties` 操作:

```java
public void clearAllProperties() {
    System.getProperties().clear();
}
```

### 3.6。访问环境变量

`System`还通过`getenv`提供对环境变量的只读访问。

例如，如果我们想访问`PATH`环境变量，我们可以这样做:

```java
public String getPath() {
    return System.getenv("PATH");
}
```

### 3.7。管理垃圾收集

通常，垃圾收集工作对我们的程序来说是不透明的。不过，有时我们可能希望向 JVM 提出直接的建议。

`System.runFinalization` 是一种允许我们建议 JVM 运行其 finalize 例程的方法。

`System.gc` 是一种允许我们建议 JVM 运行其垃圾收集例程的方法。

由于这两种方法的契约不能保证终结或垃圾收集会运行，所以它们的用处有限。

然而，它们可以作为优化来使用，比如当桌面应用程序最小化时调用`gc`:

```java
public void windowStateChanged(WindowEvent event) {
    if ( event == WindowEvent.WINDOW_DEACTIVATED ) {
        System.gc(); // if it ends up running, great!
    }
}
```

欲了解更多关于最终确定的信息，请查看我们的[最终确定指南](/web/20220628084400/https://www.baeldung.com/java-finalize)。

## 4。结论

在本文中，我们看到了`System`提供的一些字段和方法。完整的列表可以在[官方系统文档](https://web.archive.org/web/20220628084400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html)中找到。

另外，在 Github 上查看本文[中的所有例子。](https://web.archive.org/web/20220628084400/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)