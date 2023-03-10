# Java 中的单例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-singleton>

## 1。简介

在这篇简短的文章中，我们将讨论在普通 Java 中实现单件的两种最流行的方法。

## 2。基于类的单例

最流行的方法是通过创建一个常规类来实现单例，并确保它具有:

*   私人构造函数
*   包含其唯一实例的静态字段
*   获取实例的静态工厂方法

我们还将添加一个 info 属性，仅供以后使用。因此，我们的实现将如下所示:

```java
public final class ClassSingleton {

    private static ClassSingleton INSTANCE;
    private String info = "Initial info class";

    private ClassSingleton() {        
    }

    public static ClassSingleton getInstance() {
        if(INSTANCE == null) {
            INSTANCE = new ClassSingleton();
        }

        return INSTANCE;
    }

    // getters and setters
}
```

虽然这是一种常见的方法，但重要的是要注意，它**在多线程场景**中可能会有问题，这是使用单例的主要原因。

简而言之，它会导致多个实例，破坏模式的核心原则。虽然这个问题有锁定解决方案，但是我们的下一个方法从根本上解决了这些问题。

## 3。枚举单例

接下来，我们不讨论另一个有趣的方法，那就是使用枚举:

```java
public enum EnumSingleton {

    INSTANCE("Initial class info"); 

    private String info;

    private EnumSingleton(String info) {
        this.info = info;
    }

    public EnumSingleton getInstance() {
        return INSTANCE;
    }

    // getters and setters
}
```

这种方法具有由 enum 实现本身保证的序列化和线程安全，这在内部确保了只有单个实例可用，从而纠正了基于类的实现中指出的问题。

## 4。用途

要使用我们的`ClassSingleton`，我们只需要静态地获取实例:

```java
ClassSingleton classSingleton1 = ClassSingleton.getInstance();

System.out.println(classSingleton1.getInfo()); //Initial class info

ClassSingleton classSingleton2 = ClassSingleton.getInstance();
classSingleton2.setInfo("New class info");

System.out.println(classSingleton1.getInfo()); //New class info
System.out.println(classSingleton2.getInfo()); //New class info
```

至于`EnumSingleton`，我们可以像使用任何其他 Java 枚举一样使用它:

```java
EnumSingleton enumSingleton1 = EnumSingleton.INSTANCE.getInstance();

System.out.println(enumSingleton1.getInfo()); //Initial enum info

EnumSingleton enumSingleton2 = EnumSingleton.INSTANCE.getInstance();
enumSingleton2.setInfo("New enum info");

System.out.println(enumSingleton1.getInfo()); // New enum info
System.out.println(enumSingleton2.getInfo()); // New enum info
```

## 5。常见陷阱

Singleton 是一种看似简单的设计模式，程序员在创建 singleton 时很少会犯一些常见的错误。

我们区分了两种类型的单例问题:

*   存在主义(我们需要独生子女吗？)
*   实施(我们正确地实施了吗？)

### 5.1.存在问题

从概念上讲，singleton 是一种全局变量。一般来说，我们知道应该避免使用全局变量——尤其是当它们的状态可变时。

我们并不是说我们不应该使用单件。然而，我们说可能有更有效的方法来组织我们的代码。

如果一个方法的实现依赖于单例对象，为什么不把它作为参数传递呢？在这种情况下，我们显式地展示了该方法所依赖的内容。因此，在执行测试时，我们可能会很容易地模仿这些依赖性(如果需要的话)。

例如，单例通常用于包含应用程序的配置数据(例如，到存储库的连接)。如果它们被用作全局对象，那么为测试环境选择配置就变得很困难。

因此，当我们运行测试时，生产数据库会被测试数据破坏，这是很难接受的。

如果我们需要一个 singleton，我们可以考虑将它的实例化委托给另一个类——一种工厂——它应该负责确保只有一个 singleton 实例在运行。

### 5.2.实施问题

尽管单例看起来很简单，但它们的实现可能会遇到各种问题。所有这些都导致了这样一个事实，即我们最终可能拥有不止一个类的实例。

**同步**
我们上面介绍的私有构造函数的实现不是线程安全的:它在单线程环境中工作得很好，但是在多线程环境中，我们应该使用同步技术来保证操作的原子性:

```java
public synchronized static ClassSingleton getInstance() {
    if (INSTANCE == null) {
        INSTANCE = new ClassSingleton();
    }
    return INSTANCE;
}
```

注意方法声明中的关键字`synchronized`。方法体有几个操作(比较、实例化和返回)。

在没有同步的情况下，有可能两个线程交叉执行，使得表达式`INSTANCE == null`对两个线程的计算结果都是`true `，结果，创建了`ClassSingleton`的两个实例。

`Synchronization`可能会显著影响性能。如果这段代码经常被调用，我们应该使用各种技术来加速它，比如`lazy initialization`或`double-checked locking`(请注意，由于编译器优化，这可能不会像预期的那样工作)。我们可以在我们的教程“[用单例](/web/20220627083712/https://www.baeldung.com/java-singleton-double-checked-locking)进行双重检查锁定”中看到更多细节。

多实例
还有几个与 JVM 本身相关的单例问题，可能会导致我们最终使用单例的多个实例。这些问题相当微妙，我们将对每个问题进行简要描述:

1.  单例在每个 JVM 中都应该是唯一的。对于分布式系统或内部基于分布式技术的系统来说，这可能是一个问题。
2.  每个类装入器都可能装入它的 singleton 版本。
3.  一旦没有人持有对单例的引用，它就可能被垃圾收集。这个问题不会导致一次出现多个单例实例，但是当重新创建时，该实例可能会与其以前的版本不同。

## 6。结论

在这个快速教程中，我们关注了如何只使用核心 Java 实现单例模式，如何确保它的一致性，以及如何利用这些实现。

这些例子的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220627083712/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-creational)