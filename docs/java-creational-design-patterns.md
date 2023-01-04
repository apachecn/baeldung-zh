# 核心 java 中的创造性设计模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-creational-design-patterns>

## 1。简介

**[设计模式](/web/20221208143921/https://www.baeldung.com/design-patterns-series)是我们在编写软件**时常用的模式。它们代表了长期以来形成的最佳实践。这些可以帮助我们确保我们的代码设计良好，构建良好。

**[创造模式](/web/20221208143921/https://www.baeldung.com/creational-design-patterns)是专注于我们如何获得对象实例**的设计模式。通常，这意味着我们如何构造一个类的新实例，但在某些情况下，这意味着获得一个已经构造好的实例供我们使用。

在本文中，我们将重温一些常见的创造性设计模式。我们将看到它们的样子，以及在 JVM 或其他核心库中的什么地方可以找到它们。

## 2.工厂方法

工厂方法模式是我们将实例的构造与我们正在构造的类分开的一种方式。这样，我们就可以抽象出确切的类型，允许我们的客户端代码以接口或抽象类的形式工作:

```java
class SomeImplementation implements SomeInterface {
    // ...
} 
```

```java
public class SomeInterfaceFactory {
    public SomeInterface newInstance() {
        return new SomeImplementation();
    }
}
```

在这里，我们的客户端代码永远不需要知道`SomeImplementation`，相反，它根据`SomeInterface`工作。更重要的是，**我们可以改变从我们工厂返回的类型，客户端代码不需要改变**。这甚至可以包括在运行时动态选择类型。

### 2.1.JVM 中的例子

这种模式最广为人知的例子可能是 JVM 中的`Collections`类的集合构建方法，比如 `singleton()`、 `singletonList()`和`singletonMap().`，这些都返回适当集合的实例——`Set`、 `List`或`Map –`，但是**确切的类型与**无关。此外，`Stream.of()`方法和新的`Set.of()`、`List.of()`和`Map.ofEntries()`方法允许我们对更大的集合做同样的事情。

还有很多这样的例子，包括`Charset.forName()`，它将根据请求的名称返回不同的`Charset`类实例，以及`ResourceBundle.getBundle()`，它将根据提供的名称加载不同的资源包。

也不是所有这些都需要提供不同的实例。有些只是隐藏内部工作的抽象概念。例如，`Calendar.getInstance()`和`NumberFormat.getInstance()`总是返回相同的实例，但是确切的细节与客户端代码无关。

## 3.抽象工厂

[抽象工厂](/web/20221208143921/https://www.baeldung.com/java-abstract-factory-pattern)模式比这更进了一步，其中使用的工厂也有一个抽象基础类型。然后，我们可以根据这些抽象类型编写代码，并在运行时以某种方式选择具体的工厂实例。

首先，我们有一个接口和一些我们实际想要使用的功能的具体实现:

```java
interface FileSystem {
    // ...
} 
```

```java
class LocalFileSystem implements FileSystem {
    // ...
} 
```

```java
class NetworkFileSystem implements FileSystem {
    // ...
} 
```

接下来，我们有一个接口和一些具体的实现供工厂获取上述内容:

```java
interface FileSystemFactory {
    FileSystem newInstance();
} 
```

```java
class LocalFileSystemFactory implements FileSystemFactory {
    // ...
} 
```

```java
class NetworkFileSystemFactory implements FileSystemFactory {
    // ...
} 
```

然后我们有另一个工厂方法来获得抽象工厂，通过它我们可以获得实际的实例:

```java
class Example {
    static FileSystemFactory getFactory(String fs) {
        FileSystemFactory factory;
        if ("local".equals(fs)) {
            factory = new LocalFileSystemFactory();
        else if ("network".equals(fs)) {
            factory = new NetworkFileSystemFactory();
        }
        return factory;
    }
}
```

这里，我们有一个`FileSystemFactory`接口，它有两个具体的实现。**我们在运行时选择确切的实现，但是使用它的代码不需要关心实际使用的是哪个实例**。然后，它们各自返回一个不同的`FileSystem`接口的具体实例，但是同样，我们的代码不需要关心我们拥有哪个实例。

通常，我们使用另一个工厂方法来获得工厂本身，如上所述。在我们的例子中，`getFactory()`方法本身是一个工厂方法，它返回一个抽象的`FileSystemFactory`，然后这个抽象的 T1 被用来构造一个`FileSystem`。

### 3.1.JVM 中的例子

在整个 JVM 中，有很多这种设计模式的例子。最常见的是 XML 包——例如，`DocumentBuilderFactory`、 `TransformerFactory,` 和`XPathFactory`。**这些都有一个特殊的`newInstance()`工厂方法，允许我们的代码获得抽象工厂**的一个实例。

在内部，这种方法使用许多不同的机制——系统属性、JVM 中的配置文件和[服务提供者接口](/web/20221208143921/https://www.baeldung.com/java-spi)——来尝试并决定使用哪个具体实例。如果我们愿意，这就允许我们在应用程序中安装替代的 XML 库，但是这对实际使用它们的任何代码都是透明的。

一旦我们的代码调用了`newInstance()`方法，它就会从适当的 XML 库中获得一个工厂实例。然后，这个工厂从同一个库中构造我们想要使用的实际类。

例如，如果我们使用 JVM 默认的 Xerces 实现，我们将得到一个`com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl`的实例，但是如果我们想使用不同的实现，那么调用`newInstance()`将透明地返回这个实例。

## 4.建设者

当我们想要以更灵活的方式构造一个复杂的对象时，构建器模式是很有用的。它的工作原理是有一个单独的类，我们用它来构建复杂的对象，并允许客户端用一个更简单的接口来创建这个对象:

```java
class CarBuilder {
    private String make = "Ford";
    private String model = "Fiesta";
    private int doors = 4;
    private String color = "White";

    public Car build() {
        return new Car(make, model, doors, color);
    }
}
```

这允许我们分别为`make`、`model`、`doors`和`color`提供值，然后当我们构建`Car`时，所有的构造函数参数都被解析为存储的值。

### 4.1.JVM 中的例子

JVM 中有一些这种模式的非常重要的例子。**`StringBuilder`和`StringBuffer`类是构建器，允许我们通过提供许多小部件**来构建一个长的`String`。最近的`Stream.Builder`类允许我们做完全相同的事情来建造一个`Stream`:

```java
Stream.Builder<Integer> builder = Stream.builder<Integer>();
builder.add(1);
builder.add(2);
if (condition) {
    builder.add(3);
    builder.add(4);
}
builder.add(5);
Stream<Integer> stream = builder.build();
```

## 5.惰性初始化

我们使用惰性初始化模式将某些值的计算推迟到需要的时候。有时，这可能涉及单个数据片段，而其他时候，这可能意味着整个对象。

这在许多情况下都很有用。例如，**如果完全构建一个对象需要数据库或网络访问，而我们可能永远都不需要使用它，那么执行这些调用可能会导致我们的应用程序性能不足**。或者，如果我们正在计算大量我们可能永远不需要的值，那么这会导致不必要的内存使用。

通常，这是通过让一个对象作为我们需要的数据的惰性包装器，并在通过 getter 方法访问时计算数据来实现的:

```java
class LazyPi {
    private Supplier<Double> calculator;
    private Double value;

    public synchronized Double getValue() {
        if (value == null) {
            value = calculator.get();
        }
        return value;
    }
}
```

计算圆周率是一个昂贵的操作，我们可能不需要执行。上面会在我们第一次调用`getValue()`的时候这样做，而不是之前。

### 5.1.JVM 中的例子

JVM 中这种例子相对较少。然而，Java 8 中引入的 [Streams API](/web/20221208143921/https://www.baeldung.com/java-streams) 就是一个很好的例子。**在流上执行的所有操作都是懒惰的**，所以我们可以在这里执行昂贵的计算，并且知道它们只在需要时才被调用。

然而，**流本身的实际生成也可以是懒惰的**。`Stream.generate()`需要下一个值时调用函数，并且只在需要时调用。我们可以用它来加载昂贵的值——例如，通过 HTTP API 调用——并且我们只在实际需要新元素时才支付成本:

```java
Stream.generate(new BaeldungArticlesLoader())
  .filter(article -> article.getTags().contains("java-streams"))
  .map(article -> article.getTitle())
  .findFirst();
```

这里，我们有一个`Supplier`,它将进行 HTTP 调用来加载文章，根据相关联的标签对它们进行过滤，然后返回第一个匹配的标题。如果加载的第一篇文章与这个过滤器匹配，那么不管实际上有多少篇文章，只需要进行一次网络调用。

## 6.对象池

当构造一个对象的新实例时，我们将使用对象池模式，创建这个实例的成本可能很高，但是重用一个现有的实例是一个可以接受的选择。我们不必每次都构造一个新的实例，而是可以预先构造一组实例，然后根据需要使用它们。

实际对象池的存在是为了管理这些共享对象。它还跟踪它们，以便每一个在同一时间只在一个地方使用。在某些情况下，整个对象集仅在开始时被构造。在其他情况下，如果有必要，池可以按需创建新的实例

### 6.1.JVM 中的例子

JVM 中这种模式的主要例子是线程池的使用。一个 [`ExecutorService`](/web/20221208143921/https://www.baeldung.com/java-executor-service-tutorial) 将管理一组线程，并允许我们在一个线程上执行任务时使用它们。使用这种方法意味着，每当我们需要生成一个异步任务时，我们不需要创建新的线程，而这又会带来所有的开销:

```java
ExecutorService pool = Executors.newFixedThreadPool(10);

pool.execute(new SomeTask()); // Runs on a thread from the pool
pool.execute(new AnotherTask()); // Runs on a thread from the pool
```

这两个任务从线程池中分配到一个线程来运行。它可能是同一个线程，也可能是完全不同的线程，对于我们的代码来说，使用哪个线程并不重要。

## 7.原型

当我们需要创建一个与原始对象相同的对象的新实例时，我们使用原型模式。原始实例充当我们的原型，并被用来构造新的实例，然后这些实例完全独立于原始实例。我们可以在必要的时候使用它们。

**通过实现`Cloneable`标记接口，然后使用`Object.clone()`** ，Java 对此有一定程度的支持。这将产生对象的浅层克隆，创建一个新的实例，并直接复制字段。

这样做更便宜，但缺点是我们的对象中任何构造了自己的字段都将是同一个实例。这意味着这些字段的更改也会在所有实例中发生。但是，如果有必要，我们总是可以自己覆盖它:

```java
public class Prototype implements Cloneable {
    private Map<String, String> contents = new HashMap<>();

    public void setValue(String key, String value) {
        // ...
    }
    public String getValue(String key) {
        // ...
    }

    @Override
    public Prototype clone() {
        Prototype result = new Prototype();
        this.contents.entrySet().forEach(entry -> result.setValue(entry.getKey(), entry.getValue()));
        return result;
    }
}
```

### 7.1.JVM 中的例子

JVM 有几个这样的例子。我们可以通过实现`Cloneable`接口的类来看到这些。比如`PKIXCertPathBuilderResult`、 `PKIXBuilderParameters`、 `PKIXParameters`、 `PKIXCertPathBuilderResult`、`PKIXCertPathValidatorResult`都是`Cloneable.`

再比如`java.util.Date`类。值得注意的是，**这个方法覆盖了`Object.` `clone()`方法，也可以跨一个额外的瞬态字段**进行复制。

## 8.一个

当我们有一个应该只有一个实例的类，并且这个实例应该可以从整个应用程序中访问时，通常使用 Singleton 模式。通常，我们通过静态方法访问一个静态实例来管理它:

```java
public class Singleton {
    private static Singleton instance = null;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

根据具体的需求，有几种不同的方式，例如，实例是在启动时创建还是在第一次使用时创建，访问它是否需要线程安全，以及每个线程是否需要不同的实例。

### 8.1.JVM 中的例子

**JVM 有一些这样的例子，用类表示 JVM 本身的核心部分** — `Runtime, Desktop,`和`SecurityManager`。这些都有访问器方法，返回各自类的单个实例。

**此外，大部分 Java 反射 API 都使用单例实例**。不管是使用`Class.forName()`、`String.class`还是通过其他反射方法访问，相同的实际类总是返回相同的`Class,`实例。

以类似的方式，我们可以认为代表当前线程的`Thread`实例是单例的。通常会有许多这样的实例，但是根据定义，每个线程只有一个实例。从同一个线程中执行的任何地方调用`Thread.currentThread()`将总是返回同一个实例。

## 9.摘要

在本文中，我们已经了解了用于创建和获取对象实例的各种不同的设计模式。我们还看了核心 JVM 中使用的这些模式的例子，因此我们可以看到许多应用程序已经从中受益。