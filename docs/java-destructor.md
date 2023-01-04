# Java 里有析构函数吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-destructor>

## 1.概观

In this short tutorial, we'll look at the possibility of destroying objects in Java.

## 2.Java 中的析构函数

每次我们创建一个对象，Java 都会自动分配堆上的内存。类似地，每当不再需要某个对象时，就会自动释放内存。

在像 C 这样的语言中，当我们用完内存中的一个对象时，我们必须手动释放它。不幸的是， **Java 不支持手动内存释放。此外，Java 编程语言的一个特点是自己处理对象销毁——使用一种叫做 [gar](/web/20221104133453/https://www.baeldung.com/jvm-garbage-collectors) [bage](/web/20221104133453/https://www.baeldung.com/jvm-garbage-collectors) [集合](/web/20221104133453/https://www.baeldung.com/jvm-garbage-collectors)的技术。**

## 3.碎片帐集

垃圾回收从堆上的内存中移除未使用的对象。它有助于防止内存泄漏。简单地说，当不再有对特定对象的引用并且该对象不再可访问时，垃圾收集器将该对象标记为不可访问，并回收其空间。

未能正确处理垃圾收集会导致性能问题，并最终导致应用程序耗尽内存。

当一个对象达到在程序中不再可访问的状态时，它可以被垃圾收集。当出现以下两种情况之一时，对象不再可访问:

*   该对象没有任何指向它的引用
*   对该对象的所有引用都超出了范围

Java 包含了 [`System.gc()`](/web/20221104133453/https://www.baeldung.com/java-system-gc) 方法来帮助支持垃圾收集。通过调用这个方法，我们可以建议 JVM 运行垃圾收集器。然而，我们不能保证 JVM 会真正调用它。JVM 可以忽略这个请求。

## 4.终结器

[对象](https://web.archive.org/web/20221104133453/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html)类提供了 [`finalize()`](/web/20221104133453/https://www.baeldung.com/java-finalize) 方法。在垃圾收集器从内存中移除对象之前，它会调用`finalize()`方法。该方法可以运行零次或一次。但是，它不能为同一个对象运行两次。

在`Object`类中定义的`finalize()`方法不执行任何特殊动作。

终结器的主要目标是在对象从内存中移除之前释放它所使用的资源。例如，我们可以覆盖方法来关闭数据库连接或其他资源。

让我们创建一个包含`BufferedReader`实例变量的类:

```java
class Resource {

    final BufferedReader reader;

    public Resource(String filename) throws FileNotFoundException {
        reader = new BufferedReader(new FileReader(filename));
    }

    public long getLineNumber() {
        return reader.lines().count();
    }
}
```

In our example, we didn't close our resources. We can close them inside the `finalize()` method:

```java
@Override
protected void finalize() {
    try {
        reader.close();
    } catch (IOException e) {
        // ...
    }
}
```

当 JVM 调用`finalize()`方法时，`BufferedReader`资源将被释放。由`finalize()`方法抛出的异常将停止对象终结。

**然而，从 Java 9 开始，`finalize()`方法已经被废弃了。**使用`finalize()`方法可能会令人困惑，难以正确使用。

如果我们想释放一个对象持有的资源，我们应该考虑实现`[AutoCloseable](/web/20221104133453/https://www.baeldung.com/java-try-with-resources)`接口。像`[Cleaner](https://web.archive.org/web/20221104133453/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/Cleaner.html)`和`[PhantomReference](/web/20221104133453/https://www.baeldung.com/java-phantom-reference)`这样的类提供了一种更灵活的方式来管理资源，一旦一个对象变得不可达。

### 4.1.实施`AutoCloseable`

`AutoCloseable`接口提供了`close()`方法，当退出`try-with-resources`程序块时会自动执行。在这个方法中，我们可以关闭一个对象使用的资源。

让我们修改我们的示例类来实现`AutoCloseable`接口:

```java
class Resource implements AutoCloseable {

    final BufferedReader reader;

    public Resource(String filename) throws FileNotFoundException {
        reader = new BufferedReader(new FileReader(filename));
    }

    public long getLineNumber() {
        return reader.lines().count();
    }

    @Override
    public void close() throws Exception {
        reader.close();
    }
}
```

我们可以使用`close()`方法来关闭我们的资源，而不是使用`finalize()`方法。

### 4.2.`Cleaner`阶级

如果我们想在一个对象变成幻影可达时执行特定的动作，我们可以使用`Cleaner`类。换句话说，当一个对象被终结并且它的内存准备被释放时。

现在，让我们看看如何使用`Cleaner`类。首先，我们来定义一下`Cleaner`:

```java
Cleaner cleaner = Cleaner.create();
```

接下来，我们将创建一个包含更干净的引用的类:

```java
class Order implements AutoCloseable {

    private final Cleaner cleaner;

    public Order(Cleaner cleaner) {
        this.cleaner = cleaner;
    }
}
```

其次，我们将定义一个在`Order`类中实现`Runnable`的静态内部类:

```java
static class CleaningAction implements Runnable {

    private final int id;

    public CleaningAction(int id) {
        this.id = id;
    }

    @Override
    public void run() {
        System.out.printf("Object with id %s is garbage collected. %n", id);
    }
}
```

我们内部类的实例将代表清理动作。我们应该注册每一个清理动作，以便它们在一个对象变成幻影可及之后运行。

我们应该考虑不使用λ进行清洁操作。通过使用 lambda，我们可以很容易地捕获对象引用，防止对象成为幻影可达对象。如上所述，使用静态嵌套类将避免保留对象引用。

让我们在`Order`类中添加`Cleanable`实例变量:

```java
private Cleaner.Cleanable cleanable;
```

`Cleanable`实例表示包含清理动作的清理对象。

接下来，让我们创建一个注册清理操作的方法:

```java
public void register(Product product, int id) {
    this.cleanable = cleaner.register(product, new CleaningAction(id));
}
```

最后，让我们实现`close()`方法:

```java
public void close() {
    cleanable.clean();
}
```

`clean()`方法取消注册 cleanable 并调用注册的清理动作。无论调用多少次 clean，此方法最多调用一次。

当我们在一个`try-with-resources`块中使用我们的`CleaningExample`实例时，`close()`方法调用清理动作:

```java
final Cleaner cleaner = Cleaner.create();
try (Order order = new Order(cleaner)) {
    for (int i = 0; i < 10; i++) {
        order.register(new Product(i), i);
    }
} catch (Exception e) {
    System.err.println("Error: " + e);
}
```

在其他情况下，当一个实例变成幻影可达时，清理器将调用`clean()`方法。

此外，在`System.exit()`期间，清理器的行为是特定于实现的。Java 不保证清理动作是否会被调用。

## 5.结论

In this short tutorial, we looked at the possibility of object destruction in Java. To sum up, Java doesn't support manual object destruction. However, we can use `finalize()` or `Cleaner` to free up the resources held by an object. As always, the source code for the examples is available [over on GitHub](https://web.archive.org/web/20221104133453/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9).