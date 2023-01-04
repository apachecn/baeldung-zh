# 单例双重检查锁定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-singleton-double-checked-locking>

## 1。简介

在本教程中，我们将讨论双重检查锁定设计模式。这种模式通过简单地预先检查锁定条件来减少获取锁的次数。因此，通常会有性能提升。但是，需要注意的是 **[的双重锁定是一个破](https://web.archive.org/web/20220628152241/http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html) [声明](https://web.archive.org/web/20220628152241/http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)** 。

让我们更深入地看看它是如何工作的。

## 2。实施

首先，让我们考虑一个带有严格同步的简单单例:

```
public class DraconianSingleton {
    private static DraconianSingleton instance;
    public static synchronized DraconianSingleton getInstance() {
        if (instance == null) {
            instance = new DraconianSingleton();
        }
        return instance;
    }

    // private constructor and other methods ...
}
```

尽管这个类是线程安全的，但我们可以看到它有一个明显的性能缺陷:每次我们想获得 singleton 的实例时，我们都需要获得一个潜在的不必要的锁。

为了解决这个问题，我们可以先验证我们是否需要创建对象，只有在这种情况下我们才会获得锁。

更进一步，我们希望在进入 synchronized 块时再次执行相同的检查，以保持操作的原子性:

```
public class DclSingleton {
    private static volatile DclSingleton instance;
    public static DclSingleton getInstance() {
        if (instance == null) {
            synchronized (DclSingleton .class) {
                if (instance == null) {
                    instance = new DclSingleton();
                }
            }
        }
        return instance;
    }

    // private constructor and other methods...
}
```

这种模式需要记住的一点是，**字段需要是`volatile`** ，以防止缓存不一致的问题。事实上，Java 内存模型允许发布部分初始化的对象，这可能会导致一些细微的错误。

## 3。替代品

尽管双重检查锁定可能会加快速度，但它至少有两个问题:

*   因为它需要关键字`volatile`才能正常工作，所以它与 Java 1.4 和更低版本不兼容
*   它非常冗长，使得代码难以阅读

出于这些原因，让我们看看没有这些缺陷的其他选择。以下所有方法都将同步任务委托给 JVM。

### 3.1。早期初始化

实现线程安全的最简单方法是内联对象创建或使用等效的静态块。这利用了静态字段和块被相继初始化的事实( [Java 语言规范 12.4.2](https://web.archive.org/web/20220628152241/https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.4.2) ):

```
public class EarlyInitSingleton {
    private static final EarlyInitSingleton INSTANCE = new EarlyInitSingleton();
    public static EarlyInitSingleton getInstance() {
        return INSTANCE;
    }

     // private constructor and other methods...
}
```

### 3.2。按需初始化

此外，由于我们从上一段的 Java 语言规范参考中了解到，当我们第一次使用一个类的方法或字段时，就会发生类初始化，因此我们可以使用嵌套的静态类来实现惰性初始化:

```
public class InitOnDemandSingleton {
    private static class InstanceHolder {
        private static final InitOnDemandSingleton INSTANCE = new InitOnDemandSingleton();
    }
    public static InitOnDemandSingleton getInstance() {
        return InstanceHolder.INSTANCE;
    }

     // private constructor and other methods...
}
```

在这种情况下，`InstanceHolder`类将在我们第一次通过调用`getInstance.`访问该字段时分配它

### 3.3。枚举单例

最后一个解决方案来自约书亚·布洛克的`Effective Java`书(第 3 项)，使用了一个`enum`而不是一个`class`。在撰写本文时，这被认为是编写 singleton 的最简洁、最安全的方式:

```
public enum EnumSingleton {
    INSTANCE;

    // other methods...
}
```

## 4。结论

总之，这篇简短的文章介绍了双重检查锁定模式、它的限制和一些替代方案。

实际上，过多的冗长和缺乏向后兼容性使得这种模式容易出错，因此我们应该避免它。相反，我们应该使用让 JVM 进行同步的替代方法。

和往常一样，所有例子的代码都是在 GitHub 上可用的[。](https://web.archive.org/web/20220628152241/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-creational)