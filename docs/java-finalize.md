# Java 中 finalize 方法的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-finalize>

## 1。概述

在本教程中，我们将关注 Java 语言的一个核心方面——由根`Object`类提供的`finalize`方法。

简单地说，这是在对特定对象进行垃圾收集之前调用的。

## 2。使用终结器

`finalize()`方法被称为终结器。

当 JVM 认为这个特定的实例应该被垃圾收集时，终结器被调用。这样的终结器可以执行任何操作，包括恢复对象的生命。

然而，终结器的主要目的是在对象从内存中移除之前释放它们所使用的资源。终结器可以作为清理操作的主要机制，或者在其他方法失败时作为安全网。

为了理解终结器是如何工作的，让我们看一下类声明:

```java
public class Finalizable {
    private BufferedReader reader;

    public Finalizable() {
        InputStream input = this.getClass()
          .getClassLoader()
          .getResourceAsStream("file.txt");
        this.reader = new BufferedReader(new InputStreamReader(input));
    }

    public String readFirstLine() throws IOException {
        String firstLine = reader.readLine();
        return firstLine;
    }

    // other class members
}
```

类`Finalizable`有一个字段`reader`，它引用了一个可关闭的资源。当从这个类创建一个对象时，它构造一个新的`BufferedReader`实例，从类路径中的文件读取。

在`readFirstLine`方法中使用这样的实例来提取给定文件中的第一行。**注意，在给定的代码中，阅读器没有关闭。**

我们可以使用终结器来实现:

```java
@Override
public void finalize() {
    try {
        reader.close();
        System.out.println("Closed BufferedReader in the finalizer");
    } catch (IOException e) {
        // ...
    }
}
```

很容易看出，终结器的声明就像任何普通的实例方法一样。

实际上，垃圾收集器调用终结器的时间取决于 JVM 的实现和系统的条件，这是我们无法控制的。

为了让垃圾收集就地发生，我们将利用`System.gc`方法。在现实世界的系统中，我们不应该显式地调用它，原因有很多:

1.  它是昂贵的
2.  它不会立即触发垃圾收集——它只是提示 JVM 启动 GC
3.  JVM 更清楚何时需要调用 GC

如果我们需要强制 GC，我们可以使用`jconsole`来实现。

下面是演示终结器操作的测试案例:

```java
@Test
public void whenGC_thenFinalizerExecuted() throws IOException {
    String firstLine = new Finalizable().readFirstLine();
    assertEquals("baeldung.com", firstLine);
    System.gc();
}
```

在第一条语句中，创建了一个`Finalizable`对象，然后调用它的`readFirstLine`方法。这个对象没有赋给任何变量，因此当调用`System.gc`方法时，它有资格进行垃圾收集。

测试中的断言验证输入文件的内容，并仅用于证明我们的自定义类按预期工作。

当我们运行所提供的测试时，控制台上会显示一条消息，提示缓冲读取器在终结器中被关闭。这意味着调用了`finalize`方法，它已经清理了资源。

到目前为止，终结器看起来是预销毁操作的一个很好的方式。然而，这并不完全正确。

在下一节中，我们将了解为什么应该避免使用它们。

## 3。避免终结器

尽管终结器带来了好处，但它也有许多缺点。

### 3.1.终结器的缺点

让我们看看在使用终结器执行关键操作时会遇到的几个问题。

第一个值得注意的问题是缺乏及时性。我们无法知道终结器何时运行，因为垃圾回收可能随时发生。

这本身不是问题，因为终结器迟早还是会执行。然而，系统资源不是无限的。因此，我们可能会在清理发生之前耗尽资源，这可能会导致系统崩溃。

终结器也对程序的可移植性有影响。由于垃圾收集算法是依赖于 JVM 实现的，所以一个程序可能在一个系统上运行得很好，而在另一个系统上表现不同。

性能成本是终结器带来的另一个重要问题。具体来说， **JVM 在构造和销毁包含非空终结器**的对象时，必须执行更多的操作。

我们要讨论的最后一个问题是在终结过程中缺少异常处理。如果一个终结器抛出一个异常，终结进程就会停止，使对象处于一个损坏的状态，而没有任何通知。

### 3.2.终结器效果的演示

是时候把理论放在一边，看看终结器在实践中的效果了。

让我们定义一个具有非空终结器的新类:

```java
public class CrashedFinalizable {
    public static void main(String[] args) throws ReflectiveOperationException {
        for (int i = 0; ; i++) {
            new CrashedFinalizable();
            // other code
        }
    }

    @Override
    protected void finalize() {
        System.out.print("");
    }
}
```

注意`finalize()`方法——它只是将一个空字符串打印到控制台。**如果这个方法完全为空，JVM 会认为这个对象没有终结器。**因此，我们需要为`finalize()`提供一个实现，它在这种情况下几乎什么都不做。

在`main`方法中，在`for`循环的每次迭代中都会创建一个新的`CrashedFinalizable`实例。**这个实例没有分配给任何变量，因此符合垃圾收集的条件。**

让我们在标有`// other code`的那一行添加一些语句，看看运行时内存中有多少对象:

```java
if ((i % 1_000_000) == 0) {
    Class<?> finalizerClass = Class.forName("java.lang.ref.Finalizer");
    Field queueStaticField = finalizerClass.getDeclaredField("queue");
    queueStaticField.setAccessible(true);
    ReferenceQueue<Object> referenceQueue = (ReferenceQueue) queueStaticField.get(null);

    Field queueLengthField = ReferenceQueue.class.getDeclaredField("queueLength");
    queueLengthField.setAccessible(true);
    long queueLength = (long) queueLengthField.get(referenceQueue);
    System.out.format("There are %d references in the queue%n", queueLength);
}
```

给定的语句访问内部 JVM 类中的一些字段，并在每一百万次迭代后打印出对象引用的数量。

让我们通过执行`main`方法来启动程序。**我们可能期望它无限期运行，但事实并非如此。**几分钟后，我们应该会看到系统崩溃，并显示如下错误:

```java
...
There are 21914844 references in the queue
There are 22858923 references in the queue
There are 24202629 references in the queue
There are 24621725 references in the queue
There are 25410983 references in the queue
There are 26231621 references in the queue
There are 26975913 references in the queue
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
    at java.lang.ref.Finalizer.register(Finalizer.java:91)
    at java.lang.Object.<init>(Object.java:37)
    at com.baeldung.finalize.CrashedFinalizable.<init>(CrashedFinalizable.java:6)
    at com.baeldung.finalize.CrashedFinalizable.main(CrashedFinalizable.java:9)

Process finished with exit code 1
```

看起来垃圾收集器没有做好它的工作——对象的数量一直在增加，直到系统崩溃。

如果我们移除终结器，引用的数量通常会是 0，程序会一直运行下去。

### 3.3.说明

为了理解为什么垃圾收集器没有像它应该的那样丢弃对象，我们需要看看 JVM 内部是如何工作的。

当创建一个有终结器的对象(也称为 referent)时，JVM 会创建一个类型为`java.lang.ref.Finalizer`的引用对象。**在 referent 准备好进行垃圾收集之后，JVM 将引用对象标记为准备好进行处理，并将其放入引用队列中。**

我们可以通过`java.lang.ref.Finalizer`类中的静态字段`queue`来访问这个队列。

同时，一个名为`Finalizer`的特殊守护线程保持运行，并在引用队列中寻找对象。当它找到一个时，它从队列中移除引用对象，并调用 referent 上的终结器。

在下一个垃圾收集周期，引用对象将被丢弃——当它不再被引用对象引用时。

如果一个线程一直高速产生对象，这就是我们的例子中发生的情况，`Finalizer`线程就跟不上了。最终，内存将无法存储所有的对象，我们以一个`OutOfMemoryError`结束。

请注意，像本节所示的以扭曲速度创建对象的情况在现实生活中并不经常发生。然而，它展示了重要的一点—**终结器非常昂贵。**

## 4。无终结器示例

让我们探索一个提供相同功能但不使用`finalize()`方法的解决方案。注意，下面的例子并不是替换终结器的唯一方法。

相反，它被用来演示一个重要的观点:总有一些选项可以帮助我们避免终结器。

这是我们新类的声明:

```java
public class CloseableResource implements AutoCloseable {
    private BufferedReader reader;

    public CloseableResource() {
        InputStream input = this.getClass()
          .getClassLoader()
          .getResourceAsStream("file.txt");
        reader = new BufferedReader(new InputStreamReader(input));
    }

    public String readFirstLine() throws IOException {
        String firstLine = reader.readLine();
        return firstLine;
    }

    @Override
    public void close() {
        try {
            reader.close();
            System.out.println("Closed BufferedReader in the close method");
        } catch (IOException e) {
            // handle exception
        }
    }
}
```

不难看出，新的`CloseableResource`类和我们以前的`Finalizable`类之间的唯一区别是实现了*自动关闭的*接口，而不是一个终结器定义。

请注意，`CloseableResource`的`close`方法的主体几乎与类`Finalizable`中的终结器的主体相同。

下面是一个测试方法，它读取一个输入文件，并在完成任务后释放资源:

```java
@Test
public void whenTryWResourcesExits_thenResourceClosed() throws IOException {
    try (CloseableResource resource = new CloseableResource()) {
        String firstLine = resource.readFirstLine();
        assertEquals("baeldung.com", firstLine);
    }
}
```

在上面的测试中，在 try-with-resources 语句的`try`块中创建了一个`CloseableResource`实例，因此当 try-with-resources 块完成执行时，该资源会自动关闭。

运行给定的测试方法，我们将看到从`CloseableResource`类的`close`方法打印出的消息。

## 5T2。结论

在本教程中，我们关注 Java 中的一个核心概念——`finalize`方法。这在理论上看起来很有用，但在运行时可能会有很糟糕的副作用。更重要的是，除了使用终结器之外，总有其他的解决方案。

**需要注意的一个关键点是`finalize`从 Java 9 开始就被弃用了——并且最终会被删除。**

一如既往，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206122811/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)